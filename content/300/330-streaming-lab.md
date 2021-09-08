---
title: 'Lab: Streaming ETL with Glue'
weight: 330
---

## Streaming ETL Lab
![](/static/300/images-streamingETL/1.png)

### Set up Environment:
Use the [DMS Lab Student PreLab CloudFormation](https://aws-dataengineering-day.workshop.aws/400/420-pre-lab-2.html) to setup your core workshop infrastructure environment. Skip the same PreLab in the DMS section.
Click the **Deploy to AWS** icons below: 

| Region | Launch Template |
| ------------ | ------------- | 
**N.Virginia** (us-east-1) | [![Launch CloudFormation](/static/300/https:/aws-dataengineering-day.workshop.aws/images/00-deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateUrl=https%3A%2F%2Fs3.us-east-1.amazonaws.com%2Faws-dataengineering-day.workshop.aws%2FDMSlab_student_CFN.json&stackName=dmslab-student&param_DMSCWRoleCreated=no)

### Set up the kinesis stream 
1.    Navigate to AWS Kinesis console by using this link 
https://console.aws.amazon.com/kinesis/home?region=us-east-1
2.    Click “Create data stream” 
![](/static/300/images-streamingETL/2.png)
 
3.    Put `TicketTransactionStreamingData` as data stream name and put number of open shards as `2`, then click “Create data stream”.
![](/static/300/images-streamingETL/3.png)

Lab Instructions

Create Table for Kinesis Stream Source in Glue Data Catalog
1.    Navigate to AWS Glue console by using this link 
https://console.aws.amazon.com/glue/home?region=us-east-1
2.    On the AWS Glue menu, select Tables
 ![](/static/300/images-streamingETL/4.png)
3.    Enter `TicketTransactionStreamData` as the table name
4.    Click Add database and enter `tickettransactiondatabase` as the database name, and click create. 
 ![](/static/300/images-streamingETL/5.png)
5.    Using drop down to select the database we just created, and click Next
 ![](/static/300/images-streamingETL/6.png)
6.    Select Kinesis as the source, enter `TicketTransactionStreamingData` as the stream name and for the kinesis source URL, enter in `https://kinesis.us-east-1.amazonaws.com`, click Next
 ![](/static/300/images-streamingETL/7.png)
7.    Choose JSON as the incoming data format, as we will trigger JSON payload from Kinesis Data Generator in following steps. Click Next.
 ![](/static/300/images-streamingETL/8.png)
8.    Leave the schema as empty, as we will enable schema detection feature when defining glue stream job. Click Next.
 ![](/static/300/images-streamingETL/9.png)

9.    Review all the details and click Finish.
 ![](/static/300/images-streamingETL/10.png)


### Create and trigger Glue Stream job 

1.    Navigate to AWS Glue console by using this link 
https://console.aws.amazon.com/glue/home?region=us-east-1
2.    On the AWS Glue menu, select Jobs and then click Add job
 ![](/static/300/images-streamingETL/11.png)
3.    Enter `TicketTransactionStreamingJob` as the job name, select the IAM role with “GlueLabRole” in the name.
For job type, use dropdown list, select Spark Streaming; 
 ![](/static/300/images-streamingETL/12.png)

leave the rest configurations as-is and click Next.
 ![](/static/300/images-streamingETL/13.png)

4.    For Data source, select the data source named tickettransactionstreamdata we just created. Click Next.
 ![](/static/300/images-streamingETL/14.png)
5.    For Data target, select Create tables in your data target, for Data store, using dropdown list and select Amazon S3, for Format, using dropdown list and select Parquet.
 ![](/static/300/images-streamingETL/15.png)
Click the button next to Target path to select the S3 bucket. From the pop up window, select the S3 bucket with dmslabs3bucket in the name.
 ![](/static/300/images-streamingETL/16.png)
Make sure you add a path at the end `/TicketTransactionStreamData` 
 ![](/static/300/images-streamingETL/17.png)
6.    Make sure you select Automatically detect schema of each record, then click Save job and edit script.
 ![](/static/300/images-streamingETL/18.png)
7.    Review the generated script, click Save and then quit the editor.
 ![](/static/300/images-streamingETL/19.png)
8.    Select the TicketTransactionStreamingJob we just created, from the Action dropdown list, select Run job.
 ![](/static/300/images-streamingETL/20.png)

Just leave the optional parameters as default and click Run job to trigger the Glue Stream Job
 ![](/static/300/images-streamingETL/21.png)

### Trigger stream data from KDG 

1.    Launch KDG using the url you bookmarked from the lab setup, login using the user and password you provided when deploying the Cloudformation stack.
2.    Make sure you select us-east-1 region, from the dropdown list, select the TicketTransactionStreamingData as the target Kinesis stream, leave Records per second as default (100 records per second); for the record template, type in NormalTransaction as the payload name, and copy the template payload as below:
```
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
```

 ![](/static/300/images-streamingETL/22.png)

To learn more about what the payload will look like when sending from KDG simulator, refer to the document as this link,
https://awslabs.github.io/amazon-kinesis-data-generator/web/help.html

3.    Click Send data to trigger the simulated ticketed purchasing transaction streaming data.
 ![](/static/300/images-streamingETL/23.png)

### Verify the Glue stream job
1.     Navigate to Amazon S3 console by using this link
https://s3.console.aws.amazon.com/s3/home?region=us-east-1
2.    Navigate to the S3 bucket path we’ve set as Glue Stream Job target, note the folder structure of the processed data.
 ![](/static/300/images-streamingETL/24.png)
3.    Check the folder content using current date and time as the folder name. Verify that the streaming data has been transformed into parquet format and persisted into corresponding folders.
 ![](/static/300/images-streamingETL/25.png)

### Create Glue Crawler for the transformed data

1.    Navigate to AWS Glue console by using this link 
https://console.aws.amazon.com/glue/home?region=us-east-1
2.    On the AWS Glue menu, select Crawlers and click Add crawler.
 ![](/static/300/images-streamingETL/26.png)
3.    Enter `TicketTransactionParquetDataCrawler` as the name of the crawler, click Next.
 ![](/static/300/images-streamingETL/27.png)
4.    Leave the default to specify Data stores as Crawler source type and Crawl all folders, click Next.
 ![](/static/300/images-streamingETL/28.png)
5.    Choose S3 as data store and choose Specified path in my account.
 ![](/static/300/images-streamingETL/29.png)
Click the icon next to Include path input to select the S3 bucket. Make sure you select the folder TicketTransactionStreamingData. Click Select.
 ![](/static/300/images-streamingETL/30.png)
Expand the Exclude patterns, enter `checkpoint`/** to exclude the data in checkpoint folder. Review the current input and click Next.
 ![](/static/300/images-streamingETL/31.png)
6.    Select No to indicate no other data store needed, then click Next.
 ![](/static/300/images-streamingETL/32.png)
7.    Choose an existing IAM role, using the dropdown list to select the role with GlueLabRole in the name, click Next.
 ![](/static/300/images-streamingETL/33.png)
8.    As the data is partitioned to hour, so we set the crawler to run every hour to make sure newly added partition is added. Click Next.
 ![](/static/300/images-streamingETL/34.png)
9.    Using the dropdown list to select tickettransactiondatabase as the output database, enter `parquet_` as the prefix for the table, click Next.
 ![](/static/300/images-streamingETL/35.png)
10.    Review the crawler configuration and click Finish to create the crawler.
 ![](/static/300/images-streamingETL/36.png)
11.    Once the crawler is created, select the crawler and click Run crawler to trigger the first run.
 ![](/static/300/images-streamingETL/37.png)
12.    When crawler job stopped, go to Glue Data catalog, under Tables, verify that parquet_tickettransactionstreamingdata table is listed.
 ![](/static/300/images-streamingETL/38.png)
13.    Click the parquet_tickettransactionstreamingdata table, verify that Glue has correctly identified the streaming data format while transforming source data from Json format to Parquet.
 ![](/static/300/images-streamingETL/139.png)
 
### Trigger abnormal transaction data from KDG 
1.    Keep the KDG streaming data running, open another browser and launch KDG using the url you bookmarked from the lab setup, login using the user and password you provided when deploying the Cloudformation stack.

2.    Make sure you select us-east-1 region, from the dropdown list, select the TicketTransactionStreamingData as the target Kinesis stream, put Records per second as 1; click Template 2, and prepare to copy abnormal transaction data,
 ![](/static/300/images-streamingETL/40.png)


3.    For the record template, type in `AbnormalTransaction` as the payload name, and copy the template payload as below,

```
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
```
![](/static/300/images-streamingETL/41.png)
 
Click Send data to simulate abnormal transactions (1 transaction per second all from the same source IP address).

### Detect Abnormal Transactions using Ad-Hoc query from Athena
1.    Navigate to AWS Athena console by using this link
https://console.aws.amazon.com/athena/home?region=us-east-1

If it’s the first time you are using Athena in your AWS Account, click
**Get Started**

![](/static/300/images-streamingETL/image73.png)

Then click **set up a query result location in Amazon S3** at the top

![](/static/300/images-streamingETL/image74.png)

choose the **dmslab-student-dmslabs3bucket** ( e.g: <dmslab-student-dmslabs3bucket-xg1hdyq60ibs>). then click on “Select” button.

2.    Make sure you select AwsDataCatalog as Data source and tickettransactiondatabase as the database, refresh to make sure the parquet_tickettransactionstreamingdata is showing in the table list.
![](/static/300/images-streamingETL/42.png)
 
3.    Copy query as below, this is to query last hour the number of transactions by sourceip. You should see there’s large amount of transactions from the same sourceip. (if this is the first time you launch Athena, you might need to go to Athena Settings and set Query result location, please set the path to s3 bucket with "dmslab-student-dmslabs3bucket" in the name and under "athenaquery" folder)
```
SELECT count(*) as numberOfTransactions, sourceip
FROM "tickettransactiondatabase"."parquet_tickettransactionstreamingdata" 
WHERE ingest_year='2021'
AND cast(ingest_year as bigint)=year(now())
AND cast(ingest_month as bigint)=month(now())
AND cast(ingest_day as bigint)=day_of_month(now())
AND cast(ingest_hour as bigint)=hour(now())
GROUP BY sourceip
Order by numberOfTransactions DESC;
```
![](/static/300/images-streamingETL/43.png)
 
4.    Copy query as below, this is to further check if the transaction details from the same source IP.  The query verified that the request are coming from same ip but with different customer id, so it’s verified as abnormal transactions.
```
SELECT *
FROM "tickettransactiondatabase"."parquet_tickettransactionstreamingdata" 
WHERE ingest_year='2021'
AND cast(ingest_year as bigint)=year(now())
AND cast(ingest_month as bigint)=month(now())
AND cast(ingest_day as bigint)=day_of_month(now())
AND cast(ingest_hour as bigint)=hour(now())
AND sourceip='221.233.116.256'
limit 100;
```
![](/static/300/images-streamingETL/44.png)