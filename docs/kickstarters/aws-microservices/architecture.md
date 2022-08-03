
# Microservice Architecture

### What type of microservice architecture did we choose?

We mainly explored 2 options to build the Microservice architecture,

1. Kubernetes Microservice - Microservice built using Kubernetes and docker. (EKS, EC2).

2. Serverless Microservice - Microservice built using serverless services such as Lambda and API Gateway.

Given the 2 options, we decided to go with the first one due to a couple of reasons, listed below are some of the main points.

1. Kubernetes/ Docker is cloud agnostic, therefore it's very flexible language wise and platform wise.

2. Serverless requires application logic to be written in a format that AWS lambda understands, this is not the case with K8s.

3. Have more control over the deployment environment and configurations, less prone to unknown errors due to platform updates.

4. AWS knowledge is required to maintain the deployment environment if we are going with Serverless Architecture. That is to signify that environment management is more flexible with Kubernetes.

5. If you want to do long running tasks/processes then Lambda is not suitable. Lambda has hard limits in certain environment constraints. (Memory, execution time, storage)

Thats not to say the Serverless Approach doesn't hold its merits, we just felt that Kubernetes approach is more flexible and well-known and thereby would be more widely applicable. A serverless architecture would be generally cheaper than what we are building here, since there you only pay for what you use but the cloud-agnostic nature of Kubernetes made it a compelling choice to go ahead with. A serverless microservice template could be something we can work on in the future.

## Architecture Diagram

![Architecture Diagram](https://raw.githubusercontent.com/keshav1002/architecture.99x.io/aws-microservice-architecture/docs/kickstarters/aws-microservices/SolutionArchitectureDiagram.png)

The diagram above describes a possible architecture to implement the proposed system. The following services from AWS are utilized to build this solution.

- [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks) is a managed Kubernetes service that makes it easy for you to run Kubernetes on AWS and on-premises. This will be the main orchestration service that will automatically spin up and maintain the Kubernetes cluster.

- [Amazon Virtual Private Cloud (VPC)](https://aws.amazon.com/vpc/) is a service that lets you launch AWS resources in a logically isolated virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. EKS requires the deployment of a VPC in order to function correctly. It also ensures that the built solution is more secure and resilient.

- [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2) is a web service that provides secure, resizable compute capacity in the cloud. Our EKS cluster will spin up worker nodes using EC2. (There is another option of using Fargate, but we went with EC2 to keep things simple and utilization of t2.micro instances can keep us in the free tier)

- [Amazon RDS](https://aws.amazon.com/rds/) is a managed relational database service that provides you six familiar database engines to choose from, including Amazon Aurora, MySQL, MariaDB, PostgreSQL, Oracle, and Microsoft SQL Server. We decided to use RDS instead of Aurora or DynamoDB due to its simplicity and the fact that it can host pretty much any generally used SQL database services.

- [Amazon Cognito](https://aws.amazon.com/cognito/) lets you add user sign-up, sign-in, and access control to your web and mobile apps quickly and easily. Cognito will be mainly used for Authentication and Authorization purposes.

## Workflows

Here's the traffic flow and basic configuration of the architecture:

1. Requests route from the internet to the orders frontend app (frontend architecture not mentioned in the diagram, but it will be a simple static website hosted using S3 + Cloudfront)

2. Amazon Cognito is utilized for Authentication and Authorization.

3. Any request to the backend is first picked up by the load balancer which sits in the public subnet. The load balancer will automatically register the ENI (Elastic Network Interface) of worker nodes inside the private subnet so that it can route the requests to the correct worker nodes.

4. Worker nodes (EC2) will remain in the private subnet and will be managed by EKS. 2 private subnets and public subnets are utilized for higher availability.

5. Any outbound requests from the worker nodes to the outside internet (Ex:- 3rd party API request) will be routed through the NAT gateway existing in the public subnet. 2 NAT gateways across 2 AZs will further increase availability in case one of them goes down.

6. The worker nodes will also communicate directly with the RDS database instance hosted within the same private subnet. The RDS instance thereby will have no direct access through the internet, it can only be accessed by the worker nodes. (SSH tunnels can be placed in the public subnet if we ever need to directly connect to the database)

7. The VPC will internally use a Router to navigate traffic within the subnet. An internet gateway is also provisioned to enable internet access to the entire network. Requests coming through the NAT gateway will also eventually flow through the internet gateway.

## Pricing

1. EKS - 0.10$ per hour per EKS cluster. [Fixed cost for all regions](https://aws.amazon.com/eks/pricing/)

2. VPC - free

1. NAT Gateway - ~0.45$ per hour per NAT gateway, [cost changes based on region](https://aws.amazon.com/vpc/pricing/).

3. EC2 - If t2.micro instances are used, can be covered under free tier. Other instances [cost based on the type chosen](https://aws.amazon.com/ec2/pricing/).

4. RDS - No additional cost for RDS except for the [cost of the chosen instance type](https://aws.amazon.com/rds/pricing/). Can stay under free tier if t2.micro is chosen.

5. Cognito - free up to 50,000 monthly active users. [Cost based on region after that limit](https://aws.amazon.com/cognito/pricing/).

## Flexibility

Aspects of the architecture can be plugged in and plugged out based on your requirement. Listed below are some possible options

1. Using Fargate instead of EC2 or going for more powerful EC2 instances for the EKS worker nodes.

2. Using Aurora/DynamoDB or any other database service instead of RDS.

3. Reducing/Increasing the number of AZs depending on need and availability of AZs in certain regions.

4. Reducing/Increasing the number of NAT gateways provisioned.

5. Keeping all the worker nodes in a public subnet instead of private subnets, thereby eliminating the need for NAT gateways and reducing costs.

6. Using a different OAuth authentication provider instead of Cognito.

7. Use ECS instead of EKS if Kubernetes is not needed.

Keep in mind that these are just a few of the possible modifications that can be made. Some of the options mentioned above can also affect the availability and security of your application. These need to be considered when taking decisions to modify the template.