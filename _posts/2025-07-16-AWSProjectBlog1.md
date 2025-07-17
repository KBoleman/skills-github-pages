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
## Setup Networking
First things first, what even is networking? Truthfully, I do not know for sure. Based on what I do know, I presume that networking refers to the way that computers connect and share data.

AWS Definition of Computer Networking: "Computer networking refers to interconnected computing devices that can exchange data and share resources with each other." I was pretty close.

My goal here is to use the VPC wizard to create a virtual network for my web server, including subnets, routing, and more

Quick Notes:
- ***Virtual Networks*** provide secure, isolated environments for AWS resources to be launced and shared
- ***Subnets***: a range of IP addresses in a VPC
  - Private Subnet: cannot access the internet directly, must use Network Address Translation (NAT) device
  - Public Subnet: able to access the internet directly through an internet gateway
  
Subnets are bound to a single Availability Zone. Launching AWS resources in multiple Availability Zones protects applications from failure of a single Availability Zone. Each subnet is connecteed to a ***route table***, which directs network traffic. The public subnets are then routed to an ***internet gateway***, a horizontally scaled, redundant, highly available VPC component that allows the VPC to communicate with the internet. The private subnets are linked to a ***VPC Endpoint*** that provides connectivity to other AWS resources instead of the internet. The web server will be hosted in private subnet and connected to a storage service for security.

###Steps
The only change I will make to the default VPC setup is the addition of two NAT gateways, one for each private subnet. These gateways enable the web server to connect to the internet and are paid services.

Takeaways:
- VPCs are enviroments that securely host and route AWS services
- The core features of a virtual network include:
  - subnets: a range of ip addresses
  - route tables: directs network traffic
  - VPC Endpoints / Internet Gateways: provide internet connectivity for public and private subnets respectively
-----
##Resource Security
***Security Groups*** control inbound and outbound traffic for resources like servers. VPCs come with a default security group, however, 





 

