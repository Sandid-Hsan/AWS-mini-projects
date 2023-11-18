# Architecting the solution:
In this solution we will provide an insurance company with a hybrid solution for contaienr based workloads.
We will be taking in consideration the customer requirements and the problems that they are
trying to solve with this migration.
Then we will design this solution for this use case by implementing AWS services.


The company has been migrating workloads to AWS as their data center contracts expire. They have already moved some things to AWS, but now they are reaching a point where they want to migrate their workload, where half of the servers have expired contracts, and half do not. So essentially, they want to move half of the workloads to AWS and leave the other half on prem for a few years before they can move the whole thing to AWS.

The company will need to maintain network between AWS and their on-prem servers, but it is really important to them that we have the lowest amount of latency and consistent throughput possible for the network between the environments.

The part of the workload running in the data center will create files that need to be accessed by applications in the cloud. And they would like to ideally store that data in AWS, since eventually everything will be moved over anyways. They need to make sure that the communication between the two environments is reliable, and to think about how to store those files in AWS.

The company uses containers right now for all of their applications, except for databases. So they plan on using containers in AWS, as well. Also, these applications are all internal. So, they need to make sure that there is no access from the internet to the containers they moved to AWS.

They really wants to try to use the same management tools across the environments wherever possible. Same thing for containers orchestration. They would prefer not using two completely different orchestrators from on prem and AWS.

To summarize the company needs dedicated connectivity to AWS for communication, the container workload should be private, not public, and need a database in AWS that's compatible, PostgreSQL, and then, they want to maximize resilience and fault tolerance, when possible. And they want to try to use similar tooling across environments, when possible, with embedded resiliency and fault tolerance.

### Customer requirements:

1. Run container on AWS.
2. Host a PostgreSQL database on AWS. 
3. Use common tooling across environments when possible.
4. Store data generated on premises in AWS with minimal refactoring.
5. Optimize for resilience.

### Attacking the problem:

#### Hybrid Networking and Conenctivity:

**AWS Direct Connect** is the shortest path to your AWS resources. While your network traffic is in transit, it remains on the AWS global network and never touches the public internet. This isolation reduces the chance of encountering bottlenecks or unexpected increases in latency. 

**AWS Managed VPN**

We have both AWS Site-to-Site VPN and AWS client VPN.

1. AWS Site-to-Site VPN would give you the permission to access the VPC from your remote network. Site-to-Site VPN supports IPsec VPN connections.

2. AWS Client VPN is a fully managed, remote-access VPN solution that your remote workforce can use to securely access resources within both AWS and your on-premises network. It’s fully elastic, so it automatically scales up or down, based on demand.

**AWS Transit Gateway** 
AWS Transit Gateway connects your VPCs and on-premises networks through a central hub. This arrangement simplifies your network and minimizes complex peering relationships. Transit Gateway acts as a cloud router—each new connection is made only once.

As you expand globally, inter-Region peering connects transit gateways together through the AWS global network. Your data is automatically encrypted and never travels over the public internet.

#### Running Containers on AWS and NAT Gateways:

The customer needed a solution to host containers on AWS. These containers will be running internal applications that don’t require inbound communication from the internet. However, the applications might require outbound communication to the internet so they can do tasks such as download updates from internet sources. The customer must use their own custom Amazon Machine Image (AMI) for the cluster that hosts the containers, and they must also have SSH access to underlying instances.

For these reasons, we chose Amazon Elastic Container Service **Amazon ECS** as the container orchestration tool and Amazon Elastic Compute Cloud **Amazon EC2** as the launch type.

We also included NAT gateway in the architecture so that private instances could download information from the internet. 

####  Amazon Relational Database Service:

We recommend  that they migrate their on-premises database to Amazon Relational Database Service **Amazon RDS**, and to use a Multi-AZ deployment for high availability. In a Multi-AZ deployment, Amazon RDS automatically creates a primary database (DB) instance and synchronously replicates the data to an instance in a different Availability Zone. When it detects a failure, Amazon RDS automatically fails over to a standby instance without manual intervention. This failover mechanism meets the customer’s need to have a highly available database.

We also suggest that thecustomer use AWS Database Migration Service **Amazon DMS** to migrate data from their on-premises database to Amazon RDS.
AWS DMS helps you migrate databases to AWS quickly and securely. The source database remains fully operational during the migration, which minimizes the downtime to applications that rely on the database. The AWS DMS can migrate your data to and from widely used commercial and open-source databases.