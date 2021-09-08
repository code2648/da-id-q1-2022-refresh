---
title: Athena Federated query
weight: 820
---

[Federated query](https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source.html)
is a new Amazon Athena feature that enables data analysts, engineers,
and data scientists to execute SQL queries across data stored in
relational, non-relational, object, and custom data sources. With Athena
federated query, customers can submit a single SQL query and analyze
data from multiple sources running on-premises or hosted on the cloud.
Athena executes federated queries using Data Source Connectors that run
on [<u>AWS Lambda</u>](http://aws.amazon.com/lambda).

As part of this lab, we will explore how to query different data sources
from Athena. We will be using existing RDS database which was created as
part of the initial setup as datasource-1 and data lake data (stored in
S3) as datasource-2.

![](/static/800/820media/image1.png)

### Steps:

1.  Create an S3 endpoint to allow S3 access to Athena connecter Lambda, follow steps mentioned [here](https://docs.aws.amazon.com/glue/latest/dg/vpc-endpoints-s3.html) to create S3 endpoint. Make sure to use the same subnet which will be used for Lambda function in next step. This will put a route entry in the subnet routing table to reach to S3 via AWS backbone.

2.  Choose **"Connect data source"** on the Query Editor

![](/static/800/820media/image2.png)

3.  Select the data source as ‘PostgreSQL’ to which you want to connect,
    as shown in the following screenshot. Click ‘Next’  
    ![](/static/800/820media/image3.png)

4.  Click on **Configure new function** and give a catalog name. Then select **Configure new lambda function**.

![](/static/800/820media/image4.png)

5.  Fill the application setting fields as per below snapshot and click
    deploy.

![](/static/800/820media/image5.png)

6.  Wait for the function to deploy. Go back to the Athena window and
    select the newly deployed Lambda function and click ‘Connect’

![](/static/800/820media/image6.png)

7.  Once the function is deployed. Select the function and change
    handler string in runtime setting as below:

"**com.amazonaws.connectors.athena.jdbc.postgresql.PostGreSqlCompositeHandler**"

![](/static/800/820media/image7.png)


![](/static/800/820media/image8.png)

8.  Verify that the new data source is visible

![](/static/800/820media/image9.png)

9.  Refresh the query editor and check the new datasource is visible
    with corresponding database.

![](/static/800/820media/image10.png)

10. For this example, we will be using “**sport_location**” table from Postgres data source and “**parquet_sporting_event**” table from data lake.

Copy below query and paste in the query editor:
```
SELECT loc.city, count(distinct evt.id) AS events
FROM "Postgres_DB"."dms_sample"."sport_location" AS loc
JOIN "AwsDataCatalog"."ticketdata"."parquet_sporting_event" AS evt
ON loc.id = evt.location_id
GROUP BY loc.city
ORDER BY loc.city ASC;
```
![](/static/800/820media/image11.png)