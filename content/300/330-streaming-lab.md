---
title: "Lab: Streaming ETL with Glue"
weight: 330
---

## Streaming ETL Lab
![](/static/300/images-streamingETL/1.png)

### Set up environment
::alert[You need to set this environment only if you are running this workshop on your own account. If you are on an AWS event this has already been done for you.]{type="warning"}

Use the [DMS Lab Student PreLab CloudFormation](/400/420-pre-lab-2.html) to setup your core workshop infrastructure environment. Skip the same PreLab in the DMS section.
Click the **Deploy to AWS** icon below: 

::alert[Make sure you are in the appropriate AWS region.]{type="warning"}

:button[Deploy to AWS]{iconName="external" variant="primary" href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=dmslab-student&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/DMSlab_student_CFN.yaml" target="_blank"}

### Set up the kinesis stream 
1. Navigate to [AWS Kinesis console](https://console.aws.amazon.com/kinesis/home) 

2. Click “Create data stream” 
![](/static/300/images-streamingETL/2.png)
 
3. Enter the following details

- **Data stream name:** `TicketTransactionStreamingData`
- **Capacity mode:** `Provisioned`
- **Provisioned shards:** `2`

  ![](/static/300/images-streamingETL/3.png)

4. Click **Create data stream**


### Lab Instructions

#### Create Table for Kinesis Stream Source in Glue Data Catalog
1. Navigate to [AWS Glue console](https://console.aws.amazon.com/glue/home)  
2. On the AWS Glue menu, select Tables, then select **Add table manually** from the drop down list.
 ![](/static/300/images-streamingETL/4.png)
3. Enter `TicketTransactionStreamData` as the table name

4. Click Add database and enter `tickettransactiondatabase` as the database name, and click create. 
 ![](/static/300/images-streamingETL/5.png)
5. Using drop down to select the database we just created, and click Next
 ![](/static/300/images-streamingETL/6.png)
6. Select **Kinesis** as the source, select Stream in my account for Select a kinesis data stream, select the appropriate AWS region where you have created the stream, select the stream name as `TicketTransactionStreamingData` from the dropdown, and click Next.
 ![](/static/300/images-streamingETL/7.png)
7. Choose JSON as the incoming data format, as we will trigger JSON payload from Kinesis Data Generator in following steps. Click Next.
 ![](/static/300/images-streamingETL/8.png)
8. Leave the schema as empty, as we will enable schema detection feature when defining glue stream job. Click Next.
 ![](/static/300/images-streamingETL/9.png)
9. Leave partition indices empty. Click Next.
 ![](/static/300/images-streamingETL/45.png)
10. Review all the details and click Finish.
 ![](/static/300/images-streamingETL/10.png)

### Create and trigger Glue Streaming job

1. On the left-hand side of Glue Console, click on **Jobs**, this will launch Glue Studio.
   ![](/static/300/images-streamingETL/46.png)

2. Select the **Visual with a blank canvas** option. Click **Create**
   ![](/static/300/images-streamingETL/47.png)

3. Select **Amazon Kinesis** from the Source drop down list.
   ![](/static/300/images-streamingETL/48.png)

4. In the panel on the right under “Data source properties - Kinesis Stream”, configure as follows:

- **Amazon Kinesis Source:** `Data Catalog table`
- **Database:** `tickettransactiondatabase`
- **Table:** `tickettransactionstreamdata`
- Make sure that **Detect schema** is selected
- Leave all other fields as default

  ![](/static/300/images-streamingETL/49.png)

5. Select **Amazon S3** from the Target drop down list.
   ![](/static/300/images-streamingETL/50.png)

6. Select the **Data target - S3 bucket** node at the bottom of the graph, and configure as follows:

   - **Format:** `Parquet`
   - **Compression Type:** `None`
   - **S3 Target Location:** Select **Browse S3** and select the “mod-xxx-dmslabs3bucket-xxx” bucket
   
   Append `TicketTransactionStreamingData/` to the S3 URL. The path should look similar to s3://mod-xxx-dmslabs3bucket-xxx/<strong>TicketTransactionStreamingData/</strong> - don’t forget the **/** at the end. The job will automatically create the folder.
    ![](/static/300/images-streamingETL/51.png)

7. Finally, select the **Job details** tab at the top and configure as follows:

   - **Name:** `TicketTransactionStreamingJob`
   - **IAM Role:** Select the `xxx-GlueLabRole-xxx` from the drop down list
   - **Type:** `Spark Streaming`

    ![](/static/300/images-streamingETL/52.png)

8. Press the **Save** button in the top right-hand corner to create the job.

9. Once you see the **Successfully created job** message in the banner, click the **Run** button to start the job.

### Trigger stream data from KDG 

1.	Launch KDG using the url you bookmarked from the lab setup, login using the user and password you provided when deploying the Cloudformation stack.
2.	Make sure you select the appropriate region, from the dropdown list, select the TicketTransactionStreamingData as the target Kinesis stream, leave Records per second as default (100 records per second); for the record template, type in NormalTransaction as the payload name, and copy the template payload as below:
:::code{language=json showLineNumbers=false showCopyAction=true}
{
    "customerId": "{{random.number(50)}}",
    "transactionAmount": {{random.number(
        {
            "min":10,
            "max":150
        }
    )}},
    "sourceIp" : "{{internet.ip}}",
    "status": "{{random.weightedArrayElement({
        "weights" : [0.8,0.1,0.1],
        "data": ["OK","FAIL","PENDING"]
        }        
    )}}",
   "transactionTime": "{{date.now}}"      
}
:::

 ![](/static/300/images-streamingETL/22.png)

To learn more about what the payload will look like when sending from KDG simulator, refer to the document at this link,
https://awslabs.github.io/amazon-kinesis-data-generator/web/help.html

3.	Click Send data to trigger the simulated ticketed purchasing transaction streaming data.
 ![](/static/300/images-streamingETL/23.png)

### Verify the Glue stream job
1.	Navigate to [Amazon S3 console](https://s3.console.aws.amazon.com/s3/home) 
2.	Navigate to the S3 bucket path we’ve set as Glue Stream Job target, note the folder structure of the processed data.
 ![](/static/300/images-streamingETL/24.png)
3.	Check the folder content using current date and time as the folder name. Verify that the streaming data has been transformed into parquet format and persisted into corresponding folders.
 ![](/static/300/images-streamingETL/25.png)

### Create Glue Crawler for the transformed data

1.	Navigate to [AWS Glue console](https://console.aws.amazon.com/glue/home)  
2.	On the AWS Glue menu, select Crawlers and click Add crawler.
 ![](/static/300/images-streamingETL/26.png)
3.	Enter `TicketTransactionParquetDataCrawler` as the name of the crawler, click Next.
 ![](/static/300/images-streamingETL/27.png)
4.	Leave the default to specify Data stores as Crawler source type and Crawl all folders, click Next.
 ![](/static/300/images-streamingETL/28.png)
5.	Choose S3 as data store and choose Specified path in my account.
 ![](/static/300/images-streamingETL/29.png)
Click the icon next to Include path input to select the S3 bucket. Make sure you select the folder TicketTransactionStreamingData. Click Select.
 ![](/static/300/images-streamingETL/30.png)
6.	Select No to indicate no other data store needed, then click Next.
 ![](/static/300/images-streamingETL/32.png)
7.	Choose an existing IAM role, using the dropdown list to select the role with GlueLabRole in the name, click Next.
 ![](/static/300/images-streamingETL/33.png)
8.	As the data is partitioned to hour, so we set the crawler to run every hour to make sure newly added partition is added. Click Next.
 ![](/static/300/images-streamingETL/34.png)
9.	Using the dropdown list to select tickettransactiondatabase as the output database, enter `parquet_` as the prefix for the table, click Next.
 ![](/static/300/images-streamingETL/35.png)
10.	Review the crawler configuration and click Finish to create the crawler.
 ![](/static/300/images-streamingETL/36.png)
11.	Once the crawler is created, select the crawler and click Run crawler to trigger the first run.
 ![](/static/300/images-streamingETL/37.png)
12.	When crawler job stopped, go to Glue Data catalog, under Tables, verify that parquet_tickettransactionstreamingdata table is listed.
 ![](/static/300/images-streamingETL/38.png)
13.	Click the parquet_tickettransactionstreamingdata table, verify that Glue has correctly identified the streaming data format while transforming source data from Json format to Parquet.
 ![](/static/300/images-streamingETL/139.png)
 
### Trigger abnormal transaction data from KDG 
1.	Keep the KDG streaming data running, open another browser and launch KDG using the url you bookmarked from the lab setup, login using the user and password you provided when deploying the **kinesis-prelab** Cloudformation stack.

2.	Make sure you select the appropriate region, from the dropdown list, select the TicketTransactionStreamingData as the target Kinesis stream, put Records per second as 1; click Template 2, and prepare to copy abnormal transaction data,
 ![](/static/300/images-streamingETL/40.png)

3.	For the record template, type in `AbnormalTransaction` as the payload name, and copy the template payload as below,

:::code{language=json showLineNumbers=false showCopyAction=true}
{
    "customerId": "{{random.number(50)}}",
    "transactionAmount": {{random.number(
        {
            "min":10,
            "max":150
        }
    )}},
    "sourceIp" : "221.233.116.256",
    "status": "{{random.weightedArrayElement({
        "weights" : [0.8,0.1,0.1],
        "data": ["OK","FAIL","PENDING"]
        }        
    )}}",
   "transactionTime": "{{date.now}}"      
}
:::
![](/static/300/images-streamingETL/41.png)
 
Click Send data to simulate abnormal transactions (1 transaction per second all from the same source IP address).

### Detect Abnormal Transactions using Ad-Hoc query from Athena
1.	Navigate to [AWS Athena console](https://console.aws.amazon.com/athena/home)

If it’s the first time you are using Athena in your AWS Account, expand **Get Started using Athena...** below.

::::expand{header="Get Started using Athena"}
Click **View settings** to set up a query result location in Amazon S3.

![](/static/300/images-streamingETL/image91.png)

Then click **Manage**

![](/static/300/images-streamingETL/image92.png)

In the pop-up window in the **Location of query result** field, click **Browse**. Choose the bucket with the string  **dmslabs3bucket** (e.g: *mod-3fccddd609114925-dmslabs3bucket-1wrxdypwl36xo*), then click **Save**.

Append `athenaquery/` at the end of the S3 location, don't forget the **/** at the end. Click on **Save**.

![](/static/300/images-streamingETL/image93.png)
::::

2.	Make sure you select AwsDataCatalog as Data source and tickettransactiondatabase as the database, refresh to make sure the parquet_tickettransactionstreamingdata is showing in the table list.
![](/static/300/images-streamingETL/42.png)
 
3.	Copy query as below, this is to query last hour the number of transactions by sourceip. You should see there’s large amount of transactions from the same sourceip.
    :::code{language=sql showLineNumbers=false showCopyAction=true}
    SELECT count(*) as numberOfTransactions, sourceip
    FROM "tickettransactiondatabase"."parquet_tickettransactionstreamingdata" 
    WHERE ingest_year='2022'
    AND cast(ingest_year as bigint)=year(now())
    AND cast(ingest_month as bigint)=month(now())
    AND cast(ingest_day as bigint)=day_of_month(now())
    AND cast(ingest_hour as bigint)=hour(now())
    GROUP BY sourceip
    Order by numberOfTransactions DESC;
    :::
    ![](/static/300/images-streamingETL/43.png)
 
4.	Copy query as below, this is to further check if the transaction details from the same source IP. The query verified that the request are coming from same ip but with different customer id, so it’s verified as abnormal transactions.
    :::code{language=sql showLineNumbers=false showCopyAction=true}
    SELECT *
    FROM "tickettransactiondatabase"."parquet_tickettransactionstreamingdata" 
    WHERE ingest_year='2022'
    AND cast(ingest_year as bigint)=year(now())
    AND cast(ingest_month as bigint)=month(now())
    AND cast(ingest_day as bigint)=day_of_month(now())
    AND cast(ingest_hour as bigint)=hour(now())
    AND sourceip='221.233.116.256'
    limit 100;
    :::
    ![](/static/300/images-streamingETL/44.png)
 
