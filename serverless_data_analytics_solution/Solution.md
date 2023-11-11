# Architecting the solution:

In this solution we will provide a software company with a serverless data analytics solution on AWS. We will be taking in consideration the 
customer requirements and the problems that they are trying to solve with this migration. Then we will design this solution for this use case by 
implementing AWS services.

The customer company currently offer a solution that uses an S3 static website to host an HTML webpage containing restaurant menus.
Also developed a solution, where restaurant admins can log in to a system of ours and update that object in the bucket.
They do that when they want to edit the menu. Furthermore  have a system to generate a QR code pointing to the bucket, so restaurants use that 
to print the QR code and add it to their restaurant tables. All of that is already done and maintained. 
And also have a third-party payment-processing service that they use for other solutions, and they would like to add an order this item option
to the static HTML websites, that calls an API that they already have.

The customer would like to have data analytics that will help restaurants gain insights into their menus.
So for example, the customer wants to be able to see what dishes in the menu people are viewing. Insights can show us if people are just scrolling through 
the whole menu, or if they stop after appetizers, and they don't get to entrees, things like that. We also want to help his customers optimize 
their menu design through analytics.
Alsp Cost is a priority for us, so we would prefer to use AWS services that bill per refined usage, and not per time. We also would prefer 
using managed services, when possible, also prefer to have the data ingested and available as a backup in another AWS Region with encryption.

## Customer Requirements:

+ Provide HTTPS endpoint for data collection
+ Prefer using AWS managed services
+ Prefer services with pre-usage billing, not pre-time billing
+ Have cross-region data replication and encryption
+ Use different storage classes to save on cost and encryption

####  AWS Data Services:
###### Data lakes and data storage
+ Amazon S3:
	Amazon Simple Storage Service (Amazon S3) is an object storage service that offers scalability, data availability, security, and 
	performance.
+ Amazon S3 Glacier:
	Amazon S3 storage classes will have their own dedicated reading later in this week.The Amazon S3 Glacier storage classes are 
	purpose-built for data archiving. They are designed to provide you with high performance, retrieval flexibility, and low-cost archive 
	storage in the cloud.
+ AWS Lake Formation:
	AWS Lake Formation is a service that you can use to set up a secure data lake in days. A data lake is a centralized, curated, and 
	secured repository that stores all your data, both in its original form and prepared for analysis.
###### Data analytics:
+ Amazon Athena:
	Amazon Athena is an interactive query service that you can use to analyze data in Amazon S3 by using standard Structured Query Language 
	(SQL). **Athena is serverless** , so you don’t need to manage infrastructure, and you pay only for the queries that you run.
+ Amazon EMR:
	Amazon EMR is a big data solution for petabyte-scale data processing, interactive analytics, and machine learning that use open-source 
	frameworks, such as Apache Spark, Apache Hive, and Presto.
+ Amazon OpenSearch Service:
	You can use Amazon OpenSearch Service to perform interactive log analytics, real-time application monitoring, website search, and more. 
###### Data movement:
+ Amazon Kinesis:
	With Amazon Kinesis, you can collect, process, and analyze real-time, streaming data so that you can get timely insights and react 
	quickly to new information.
+ AWS Glue: 
	AWS Glue is a serverless data integration service that you can use to discover, prepare, and combine data for analytics, machine 
	learning, and application development
+ AWS DMS:
	AWS Database Migration Service (AWS DMS) helps you migrate databases to AWS quickly and securely.
###### Predictive analytics and machine learning:
+ Amazon SageMaker:
	SageMaker can be used for any generic ML solution. You can use it to build, train, and deploy ML models for virtually any use case 
	with fully managed infrastructure, tools, and workflows.
+Amazon Rekognition:
	Amazon Rekognition is one of Raf’s favorite ML services from the entire list of AWS services! It is easy to use, serverless, and 
	abstracted, in the sense that you interact with it by doing API calls.
#### Why choose S3 over EBS and EFS?

**Amazon EBS** or Amazon Elastic Block Store is a service that provides block-level access when you need a file system. It provides high-end 
performance and can handle workloads at virtually any scale. The reason why we won't choose EBS volumes in this architecture is because it 
does not align well with the requirements and needs since they need an EC2 instance to be attached and their billing is not granular as
Amazon S3. Also if you allocate 40-gigabyte volume, you'll have to pay for 50 gigabytes regardless of what you use.
Also the durability standrads of amazon S3 are higher than Amazon EBS since S3 replicates data across an entire AWS Region.

**Amazon EFS** or Amazon Elastic File System. It can replicate data across an AWS Region. But it needs to be attached to resources such EC2 
instances, containers or servers. It doesn't address data analytics architectures.

**Amazon S3** let you interact with it by doing API calls from everywhere. S3 is ideal for separating storage from data processing and that is
very important to have in data analytics. Also detaching storage from processing gives me the flexibility of using specific data-processing 
services that focus on the right thing. That's why Amazon S3 is also commonly used as a data lake. 
Furthermore, Amazon S3 is known for providing high data durability standards. It is designed to provide 11 nines, or 99.999999999 percent 
durability for objects, over in a given year. S3 only charges for defined usage which means it charges for only the amount stored small or 
big. For large amounts of data S3 provides a feature called **S3 Intelligent-Tiering** which transition objects from one storage class to 
another automatically.

In a nutshell, Amazon S3 charges per usage, refined usage. It easily enables cross-Region replication. It encrypts data, both in transit and 
at rest, at no additional cost. And let me use multiple data ingestion and processing systems, by being a managed service that does not need 
instances or servers to operate. This is perfect for our use case.

#### Choosing a service for data ingestion:

**Amazon EMR** is used in doing clickstream ingestion with a managed hadoop cluster which although it's a managed service, charges per time 
and requires some big data knowledge to operate. We won't  proceed with amazon EMR.

**AWS DMS** or AWS dtabase Migration Service. It is suitable when you want to bring a database to the cloud. In our scenario there are no
databases to be migradetd. So for this reason AWS DMS is not needed

**AWS Data Exchange** is a data analytics service that provides data catalogs, and is more suitable for integrating third-party data into a 
data lake for further analysis. also not required for our architecture.

**Amazon Kinesis Data Analytics** is suitable for real-time data processing of data that is being ingested.  Since our customer will have 
control over how the data is being sent, not requiring any transformation at all, We won't be going to use Kinesis Data Analytics for now.

**Amazon Kinesis Firehose** will be used in our scenarion since our customer would not appreciate to write additional code also he does not
have low latency requirements. Firehose is designed to reliably load streaming data. It can capture, transform and deliver Amazon S3 or generic
HTTP endpoints. It is a fully managed service that automatically scales.

In order to interact with Amazon kinesis we need to perform Kinesis API calls such as PutRecord. Our customer has a JavaScript library that 
they want to use and the JS library is looking to make HTTPS POST. And for this we should use **Amazon API Gateway** to send data to our Kinesis
stream. 

#### Accessing the ingested data:

**Amazon S3 Select** could be used to access data. It perform SQL queries to filter the contents of Amazon S3 objects. The issue here is that 
S3 Select can only query one file per query, one file at a time, and not lots of small files. Our solution is very likely to produce multiple 
objects, unless we are running something else to aggregate the data, which we are not doing for now.

**Amazon GLue** and **Amazon EMR** are strong candidates for data processing but they really shine when it comes to processing unstructured 
data using big data frameworks. And our customer asked to keep the architecture the most minimal possible because of reduced staff, cost 
and maintenance. That's why we won't proceed with these two services.

**Amazon Athena** is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL. Athena is serverless, so 
there is no infrastructure to manage, and you pay only for the queries that you run. The only problem that we can face here is that Athena 
needs a serializer/deserializer on the table since the customer is using a Javascript library. The developers are  only using the JSON format for
the data which makes Athena the right choice.


