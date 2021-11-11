---
title: Athena Federated query
weight: 820
---

[Federated query](https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source.html) is a new Amazon Athena feature that enables data analysts, engineers, and data scientists to execute SQL queries across data stored in relational, non-relational, object, and custom data sources. With Athena federated query, customers can submit a single SQL query and analyze data from multiple sources running on-premises or hosted on the cloud. Athena executes federated queries using Data Source Connectors that run on [AWS Lambda](http://aws.amazon.com/lambda).

As part of this lab, we will explore how to query different data sources from Athena. We will be using existing RDS database which was created as part of the initial setup as datasource-1 and data lake data (stored in S3) as datasource-2.

![](/static/800/820media/image1.png)

#### Prerequisites

:::alert{type="warning"}
[Ingestion with DMS](/400) and [Transforming data with Glue ETL](/600) labs are prerequisites for this lab.
:::
If you are in an AWS Event use Event Engine Team Dashboard, if you are in your own account check Outputs section of the CloudFormation stack to get the following values.
*  Security Group Id for Athena Federated Query Connector Check the Security Group Name with string sgdefault from Amazon EC2 Security Group console (e.g. sg-006190abcdc987641).
* SubnetId for Athena Federated Query Connector [Private Subnet ID from Amazon VPC Subnet console (e.g. subnet-07edabc12335e98bd).

### Steps:

1. Create an S3 endpoint to allow S3 access to Athena connecter Lambda, follow steps mentioned [here](https://docs.aws.amazon.com/glue/latest/dg/vpc-endpoints-s3.html) to create S3 endpoint. Make sure to use the same subnet which will be used for Lambda function in next step. This will put a route entry in the subnet routing table to reach to S3 via AWS backbone.

2. In the Athena console, click the  hamburger icon (3 horizontal lines) on the top left corner and choose **"Data sources"**.

![](/static/800/820media/image2-1.png)
![](/static/800/820media/image2-2.png)

3. In Data Sources page, choose **Connect data source** button

![](/static/800/820media/image2-3.png)

4. Select the data source as **PostgreSQL** to which you want to connect, as shown in the following screenshot.
![](/static/800/820media/image3.png)

5. Under *Data source details*: for **Data source name**, enter `Postgres_DB`
![](/static/800/820media/image3-1.png)

6. Under *Lambda function*, Choose ‘**Create a new function**’ and click **Create a new Lambda function in AWS Lambda button** which will open a new tab/window to AWS Lambda Console.

![](/static/800/820media/image4.png)

7. Fill the application setting fields as per below snapshot and click deploy.

|Field|Value|
|---|---|
|Application Name|`AthenaJdbcConnector`|
|SecretNamePrefix|`AthenaFed_`|
|SpillBucket|Get the **BucketName** from EventEngine Team Dashboard|
|DefaultConnectionString|`postgres://jdbc:postgresql://<DATABASE_ENDPOINT>:5432/sportstickets?user=adminuser&password=admin123` **→** replace **\<DATABASE_EDNPOINT\>** with the correct database endpoint (e.g. dmslabinstance.abcdshic87yz.eu-west-1.rds.amazonaws.com)|
|DisableSpillEncryption|`false`|
|LambdaFunctionName|`postgresqlconnector`|
|LambdaMemory|`3008`|
|LambdaTimeout|`900`|
|SecurityGroupIds|Use the **SecurityGroupId** noted in prerequisites|
|SpillPrefix|`athena-spill-bucket`|
|SubnetIds|Use the **SubnetId** noted in prerequisites| 

![](/static/800/820media/image5.png)

8. Wait for the function to deploy. Go back to the Athena window, click the refresh button next to **Choose or enter a Lambda function**. Select the newly deployed Lambda function and choose ‘Connect data source’
![](/static/800/820media/image6.png)

9. Once the function is deployed. Select the function and change handler string in runtime setting as below:

:::code{language=java showLineNumbers=false showCopyAction=true}
com.amazonaws.athena.connectors.jdbc.postgresql.PostGreSqlCompositeHandler
:::

![](/static/800/820media/image7.png)
![](/static/800/820media/image8.png)

10. Verify that the new data source is visible

![](/static/800/820media/image9.png)

11.  Navigate back to the [query editor](https://console.aws.amazon.com/athena/home#/query-editor) and choose the new datasource and review whether it's displaying the corresponding database (dms_sample).

![](/static/800/820media/image10.png)

12. For this example, we will be using “**sport_location**” table from Postgres data source and “**parquet_sporting_event**” table from data lake.

Copy below query and paste in the query editor:

:::code{language=sql showLineNumbers=false showCopyAction=true}
SELECT loc.city, count(distinct evt.id) AS events
FROM "Postgres_DB"."dms_sample"."sport_location" AS loc
JOIN "AwsDataCatalog"."ticketdata"."parquet_sporting_event" AS evt
ON loc.id = evt.location_id
GROUP BY loc.city
ORDER BY loc.city ASC;
:::

![](/static/800/820media/image11.png)