# AWS Projects: AWS 101 Workshop
### Setting up a basic Linux-Based Web Server
### Timothy Coleman
### 07/16/25
Level: 100  
Cost: ~ $0.44 per hour


Hello, my name is Timothy Coleman and I'm a UCLA undergrad working towards becoming a Solutions Architect. This is first of a series of cloud projects that I will document on GitHub. In making these posts, I aim to build cloud-engineering and architectural skills, improve my understanding of cloud principles and GitHub, and demonstrate commitment, learning, and growth.


The goal of this project is to develop a simple web application infrastructure

The following services / resources are utilized:
- Virtual Private Cloud (VPC)
- Security Groups (SGs)
- Identity and Access Management (IAM)
- Amazon Elastic Compute Cloud (EC2)
- AWS Systems Manager (SSM
- Application Load Balancer
- Amazon Simple Storage Service (S3)
- Autoscaling Groups (ASGs)
-----
## Setup Networking (VPC)
First things first, what even is networking? Truthfully, I do not know for sure. Based on what I do know, I presume that networking refers to the way that computers connect and share data.

AWS Definition of Computer Networking: "Computer networking refers to interconnected computing devices that can exchange data and share resources with each other." I was pretty close.

My goal here is to use the VPC wizard to create a virtual network for my web server, including subnets, routing, and more

### Quick Notes:
- ***Virtual Networks*** provide secure, isolated environments for AWS resources to be launced and shared
- ***Subnets***: a range of IP addresses in a VPC
  - Private Subnet: cannot access the internet directly, must use Network Address Translation (NAT) device
  - Public Subnet: able to access the internet directly through an internet gateway
  
Subnets are bound to a single Availability Zone. Launching AWS resources in multiple Availability Zones protects applications from failure of a single Availability Zone. Each subnet is connecteed to a ***route table***, which directs network traffic. The public subnets are then routed to an ***internet gateway***, a horizontally scaled, redundant, highly available VPC component that allows the VPC to communicate with the internet. The private subnets are linked to a ***VPC Endpoint*** that provides connectivity to other AWS resources instead of the internet. The web server will be hosted in private subnet and connected to a storage service for security.

### Steps
The only change I will make to the default VPC setup is the addition of two NAT gateways, one for each private subnet. These gateways enable the web server to connect to the internet and are paid services.

### Takeaways:
- VPCs are enviroments that securely host and route AWS services
- The core features of a virtual network include:
  - subnets: a range of ip addresses
  - route tables: directs network traffic
  - VPC Endpoints / Internet Gateways: provide internet connectivity for public and private subnets respectively
-----
## Resource Security (SGs)
***Security Groups*** control inbound and outbound traffic for resources like servers. VPCs come with a default security group, however, additional groups can be configured with custom inbound and outbound rules.

My goal here is to manage the incoming traffic for the web server and the resource within the public subnets.

### Steps
1. I start a new security group
2. I set the security group name to "Load Balancer Security Group" and the description to "External Access" (for clarification this VPC should end with (project-vpc))
3. I set the following inbound rules:
   - Type: HTTP
   - Protocol: TCP
   - Port Range: 80
   - Source: Anywhere-IPv4
   - Description: "Allow HTTP inbound from Internet"
4. I apply a tag for organization. I set the key to "Name" and the value to "LoadBalancerSecurityGroup"
5. I create the security group
6. I start another security group.
7. I set the security group name to "Web Server Security Group" and the description to "Web Server Access" (same VPC)
8. I repeat step 3, however the following rules are different:
   - Source: Load Balancer Security Group
   - Description: "Allow HTTP inbound from Load Balancer"
9. I repeat step 4, however the value for this new tag is "WebServerSecurityGroup"
10. I create the second security group

To my understanding, the first security group named "Load Balancer Security Group" manages in incoming traffic from the internet and the second security group named "Web Server Security Group" manages incoming traffic from the initial load balancer to the web server. I'm curious as to why two load balancers are placed inbetween the server and the internet. I presume that the Web Server Security Group is added in case the intial load balancer fails, essentially serving as a last line of defense.

### Takeaways
- Security Groups control the incoming and outgoing traffic for resources
-----
## Access Management (IAM)

AWS Identity and Access Management securely controls access to AWS services. It serves as the hub for managing users, security credentials, and permissions to users and apps.

My goal here is to designate which AWS resources the web server can use.

### Quick Notes:
- ***IAM Polices***: a set of permissions
- Policies are assigned to ***IAM Roles***, which may be a user or service such as EC2
- ***EC2 Instance Profile***: a container that holds an IAM role and connects it to an EC2 instance
- ***AWS Systems Manager***: a service that allows EC2 instances to be administered and managed without accessing them over the public internet

### Steps
1. Create a new IAM role and attach it to the EC2 service
2. For the use case, select "EC2 Role for AWS Systems Manager" (policy name will be "AmazonSSMManagedInstanceCore")
3. Lastly, name the role "WebServerInstanceProfile"

~~This last step confuses me. The name of the role I just created indicates that it is an Instance Profile, which is a container for IAM roles. But how can a role hold another role within it?~~

Okay, I think I understand the difference between an AWS Systems Manager, a role, and an Instance Profile
1. AWS Systems Manager is the job specific to an EC2 with specific permissions and abilities. However, AWS Systems Manager is NOT an actual role, more like a type of EC2.
2. For comparison, think about the difference between a job and a job title. A job comes with general obligations, responsibilities, and power, comparable to a role in AWS. The term "manager" dilineates specific tasks and responsibilities, denoting a _type_ of job.
3. By creating a role that provides permissions (in this case administrative and managerial authority) to any EC2 that it connects to, an Instance Profile is formed. To further the analogy in #2, think of an Instance Profile as job for EC2. Instances may be replaced in the same manner that an employee is and, similarly, any new instance that assumes that job (instance profile) will have the same tasks, expectations, and reach.
-----
## Deploy Compute (EC2)

AWS Elastic Compute Cloud provides scalable, on-demand computing capacity to AWS cloud infrastructure. EC2 can be used to create virual servers, configure security and networking, and manage storage.

My goal here is to deploy a web server using EC2

### Steps
1. Launch an instance named "mywebserver"
2. Select the Amazon Linux 2023 Amazon Machine Image (AMI) and choose 64-bit (x86) architecture
3. Choose the t2.micro instance type, which provides 1 vCPU and 1 GiB of memory, then proceed without a key pair (because connections to the EC2 are made through instance profiles and not direct SSH access)
4. Under network settings, associate this server with the VPC and one of the ***private*** subnets set up earlier
5. Under firewall, select the "Web Server Security Group" created earlier to manage traffic from the public subnets
6. Under advanced details, choose the "WebServerInstanceProfile". This will enable private connection to the web server
7. Enter the code below into the User Data field to install the necessary PHP server components and finalize the instance
-----
    #!/bin/bash
    yum update -y
    # Install Session Manager agent
    yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
    systemctl enable amazon-ssm-agent
    # Install and start the php web server
    dnf install -y httpd wget php-json php
    chkconfig httpd on
    systemctl start httpd
    systemctl enable httpd
    
    # Install AWS SDK for PHP
    wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.zip
    unzip aws.zip -d /var/www/html/sdk
    rm aws.zip
    
    #Install the web pages for our lab
    if [ ! -f /var/www/html/index.html ]; then
    rm index.html
    fi
    cd /var/www/html
    wget https://ws-assets-prod-iad-r-iad-ed304a55c2ca1aee.s3.us-east-1.amazonaws.com/2aa53d6e-6814-4705-ba90-04dfa93fc4a3/index.php
    
    # Update existing packages
    dnf update -y
-----

### Takeaway
- EC2 is one of the essential services of AWS and its complexity matches its importance. There are so many different ways to configure EC2 instances and many applications as well.
-----
## Administer Web Server (SSM)

Session Manager is a fully managed AWS Systems Manager capability that allows users to manager their EC2 instances, virtual machines, edge devices, and on-premises servers.

My goal here is to securely access the web server for administrative purposes.

### Steps
1. Connect to the web server using Session Manager
2. Copy and paste the following command lines into the shell:

       echo -n 'Private IPv4 Address: ' && ifconfig enX0 | grep -i mask | awk '{print $2}'| cut -f2 -d: && \
       echo -n 'Public IPv4 Address: ' && curl checkip.amazonaws.com

   This will produce two IP addresses to the screen
   - Private IP Address - evidence of successful connection without connection to the internet
   - Public IP Address - the Elastic IP address allocated to the NAT gateway, allowing private subnet resources (the web server) to communicate with the internet

### Troubleshooting
I intially experience trouble connecting to the instance via Sessions Manager. I kept running into the following error message:

"**SSM Agent is not online**
The SSM Agent was unable to connect to a Systems Manager endpoint to register itself with the service."

This was very confusing because I could see that I chose the proper IAM role to enable connection. The instance's subnet id read the id to a public subnet, so I considered if I misconfigured my network. Because the instance only displayed a private IPv4 address, I ruled out misconfiguration to a public subnet. Then I considered if Security Groups played a role in this problem. Creating a new instance from scratch was my absolute last option, so I looked for ways to edit the security groups. I found just that under the Actions button. Finally, I reached the page to manange security groups, and lo and behold:

I chose the wrong security group. I most defintely skipped over the firewall process while configuring the instance. Just goes to show the importance of the little things and attention to detail when completing projects and running complex systems.

Okay, so I changed the security group and I still can't connect. Awesome. I suspect that the issue is another skipped step.

I decided to delete the instance and create a new one so I can keep the same name. However, I recognize that in a professional context new resources must be configured and deployed before old resources are removed. In the future, I will create before deleting. While setting up this new instance, I realized that I left the default public subnet instead of choosing a private subnet. Yeah, I'm never working on AWS in a lecture ever again. After completing the rest of the steps, I can finally connect. Hallelujah.

### Takeaways
- Session Manager _requires_ an IAM instance profile to connect with an instance
- Without a Key Pair, instance require a private subnet to be connected to using Sessions Manager
- Projects go a whole lot smoother when you dedicate time to focus on them exclusive
-----

## Load Balancing (ALB)

Application Load Balancers distribute traffic across your infrastructure.

My goal here is to route incoming web traffic to the web server instance. The load balancer will handle network configuration and securty policies to enable secure communication between clients and the web server

### Steps
1. Create an Application Load Balancer
2. Configure the load balancer with the following settings:
   - Load balance name: WebServerLoadBalancer
   - Scheme: internet-facing
   - Load balancer IP address type: IPv4
   - VPC: project-vpc
   - Availability Zone 1: project-subnet-public1
   - Availability Zone 2: project-subnet-public2
3. Replace the default security group with the Load Balancer Securty Group created earlier
4. Leave the protocol and port on their defaults of HTTP & 80, then click "create a target group" to define which instances the load balancer will route traffic to. Do not delete the original tab after a new tab pops up.
5. Configure the target group with the following settings:
  - Target Type: Instances
  - Target Group Name: WebServerTargetGroup
  - Protocol: HTTP
  - Port: 80
  - IP Address Type: IPv4
  - VPC: project-vpc
  - Protcol Version: HTTP1
  - Health Check Protocol: HTTP
  - Health Checks Path: /
  - Key: Name
  - Valie: WebServerTargetGroup
4. After registering, select the instance labeled "webserver" and click the button "Include as pending below". This will configure the load balancer to route traffic from the internet to the EC2 web server instance
5. Finalize the target group, then refresh the original tab. If the previous configuration disappear, simply readd them to their respective forms.
6. Under Listening and Routers, select the newly created target group and hit "Create load balancer" to finish.
-----
## Testing the Web Server

The web server must be fully provised before and the target group must be shown as healthy before accessing the web server.

My goal here is to connect to the web server via browser

### Steps
1. Check that the load balancer state is active
2. Check that a healthy target is listed after clicking the WebServerTargetGroup link under the Listening and Rules tab
3. Find the load balancer's public URL under the DNS name and search it in the browser


### Troubleshooting

The server failed to load after searching the URL and a 504 error was displayed instead. I returned to my registered targets and found that they had failed the health checks. After doing some digging, I realized that I created my load balancer security group with no outbound rules. I presume that means the page was empty because the browser was not able to recieve the data from the load balancer.

### Takeaways
- Load balancers are responsible for the communication between cloud resources and internet resources
- Security groups manage how data is recieved and transferred via load balancer
-----
## Storage (S3)

Amazon Simple Storage Service is an object storage service offering scalability, data availability, performance, and security. This service enables data management, analysis, and protection for a variety of use cases.

My goal here is to upload files to S3 and view them using the web server

### Steps
1. Create and name a bucket. Leave the settings on their defaults.
2. Download the required files, unarchive them, then upload them to the bucket (files found here:https://catalog.workshops.aws/aws101/en-US/2-build-web-server/08-storage)
3. Add the "AmazonS3ReadOnlyAccess" permission to the Instance Profile through IAM. This will give the web server permission to list the contents of the bucket
4. Search the name of the S3 bucket using the web server

## Takeaways
- S3 is used to store and use data
- Buckets are very easy to set up
- Instances, by default, need permission to access buckets
-----
## Scaling (ASG)

Auto Scaling Groups are collections of EC2 instances, logically grouped for automatic scaling and management.

My goals here is to create an ASG to circumvent my single point of failure.

### Steps
1. Create a new ASG
2. Create a launch template
3. Choose the same VPC but a _different_ AZ from the one you choose for the active web server
4. Configure the rest of the ASG. Most of the options can be left to their defaults.

### Takeways
- ASGs can be used in cases of failure but also in cases of scaling to accomodate growth.




