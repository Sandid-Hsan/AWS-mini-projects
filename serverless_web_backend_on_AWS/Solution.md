# Architecting the solution steps:
In this solution we will provide an ecommerce company with a backend web service on AWS.
We will be taking in consideration the customer requirements and the problems that they are
trying to solve with this migration.
Then we will design this solution for this use case by implementing AWS services.

The customer have a company that sell cleaning products all over the world and he have a
website and an app open to the public, where customers shop and place orders.
Also he have a website for buyers to purchase their products wholesale, for resale in
their own shops.
All of the orders from across these various frontends come in to the orders service, 
hosted on premises, that validates, authenticates, accepts, orders, and processes, and 
stores the order in a database, and then also makes calls downstream to other services, like
the inventory fulfillment and accounting service, to reflect the confirmed order.
The inventory fulfillment and accounting services are already hosted in the AWS Cloud, 
and the orders service is the last major piece that needs to be moved over. 
They are using a web server that is also hosted on the same server with the application,
to route requests to the backend service.
The customer is currently using MySQL database on-prem for this service to store the order
data. 
The other downstream services will work with their own data store. The customer is open to 
changing databases, but it's very important that to be highly available and durable. 
Orders are only stored in one table, so maintaining a whole MySQL instance for this has
been a bit of a time-consuming task for the minimal data stored in the database.
Orders are slow to get accepted, and sometimes fail when the application becomes overwhelmed.
The customer have a hard time scaling on premises to meet his demands, and he is looking for
a solution with automatic scaling that doesn't require time to set up or operate. 
The components are too tightly coupled. So if one API call fails, then the next few ones 
don't get completed. This causes inconsistency with the orders data and causes 
headaches for the customers.
The app has pretty spiky demand. When launching online sales or sending out coupon codes,
there is like, huge traffic. But then, at other times, there is essentially zero demand
For the monitoring and logging to be easy to put in place. And ideally, everything have
to use the same logging system.
And finally, the customer wants to optimize the solution for cost and performance as much
as possible.
### Customer requirements:

1. Managed scaling for compute and database components.
2. Decoupling components to maximize resilience.
3. Centralized monitoring and logging
4. Optimizing for cost, performance efficiency and operational overhead.

### Attacking the problem:

#### Compute requirement:
To have a managed and scaled compute service we have to use serverless services.
We have at our hand multiple compute services categories like **instances**, 
**containers**, **Edge and  hyprid** and **Serverless**.
As the customer required managed scaling for compute then we will be proceeding with
the serverless category and our service will be **AWS Lambda** for the compute backend.

Now we have to expose the backend lambda function and for taht we will integrate 
**Amazon API Gateway**, thus providing a way to expose the backend servie without exposing 
to the open internet intecepating if the customer decieds to add authentication to 
API Gateway to secure it further.
	
#### Database component:
For the database component, the customer is storing  the data in a MySQL database so we should go with **AuroraDB** option 
for its efficiency with relationaldatabases. But since there is no complex queries and jointed tables we will choose the 
noSQL option **DynamoDB** as we will also be using **AWS Lambda**.

####Decoupling components:
Our customer currently uses a synchronous web application to host the orders service, which is causing various issues—for 
example, the code is too tightly coupled with downstream API calls. So we suggest that they move to an **event-driven architecture** 
to solve this problem.

**Event-driven architectures** have three key components: **event producers** , **event routers** , and **event consumers**. 
A producer publishes an event to the router, which filters and pushes the events to consumers. 
Producer services and consumer services are decoupled, which means that they can be scaled, updated, and deployed 
independently. 
As event-driven architectures we have two choices. To either go with **Amazon EventBridge** solution or **Amazon SNS**.
To satisfy our customer need in cost optimization we will choose Amazon SNS since it is more suitable for our architecture 
and because it’s simple to use and supports a straightforward way to send messages between application components. 

Also we will be implementing **Amazon SQS** in our architecture since in the online sales period the system gets overwhelmed 
with requests which can lead to loosing requests.For that we have to add a messaging queue between **Amazon API Gateway** and 
**Amazon Lambda**. 

###Centralized monitoring and logging:

We won't have to add another service for monitoring or logging since our serverless solutions all tend to be integrated 
with **Amazon CloudWatch** and **Amazon CloudWatch Logs** since they meet the need for centralized application logging.
