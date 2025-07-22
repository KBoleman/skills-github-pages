# AWS Projects: AWS 101 Workshop
### Setting up a basic Linux-Based Web Server
### Timothy Coleman
### 07/16/25
Level: 100\
Cost: ~ $0.44 per hour


Hello, my name is Timothy Coleman and I'm a UCLA undergrad working towards becoming a Solutions Architect. This is first of a series of cloud projects that I will document on GitHub. In making these posts, I aim to build cloud-engineering and architectural skills, improve my understanding of cloud principles and GitHub, and demonstrate commitment, learning, and growth.


The goal of this project is to develop a simple web application infrastructure

The following services / resouese are utilized:
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
7. Enter the code below into the User Data field to install the necessary PHP server components













 

