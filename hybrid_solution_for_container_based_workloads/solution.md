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