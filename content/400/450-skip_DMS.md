---
title: "Option 3: Skip DMS Lab"
weight: 450
---

### Steps

1. Open AWS CloudShell
2. Copy the data from staging Amazon S3 bucket to your S3 bucket
3. Verify the data
- Next Steps
- Appendix A: Self-Paced Data Lake Lab

### About the lab setup:

![](/static/400/images/image92.png)

RDS Postgres Database is used as a source of ticket sales system for sporting events. It stores transaction information about ticket sales price to selected people and ticket ownership transfer with additional tables for event details. AWS Database Migration Service (DMS) is used for a full data load from the Amazon RDS source to Amazon S3 bucket.

Before the Glue lab starts, you might choose to skip the DMS data migration, instead copy the source data to your S3 bucket directly.

In today's lab, you will copy the data from a centralized S3 bucket to your AWS account, crawl the dataset with AWS Glue crawler for metadata creation and transform the data with AWS Glue to Query data and create a View with Athena and Build a dashboard with Amazon QuickSight.


#### Open AWS CloudShell

Open [AWS CloudShell](https://console.aws.amazon.com/cloudshell/home?region=us-east-1). It will open a terminal window in the browser. (If there is a pop-up, close it)

::alert[CloudShell is available in specific regions and you shall find the supported regions in [AWS Regional Services List](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/). By executing the following command you will be copying the data to the correct S3 bucket in whatever region it belongs (It can also be across region)]{type="info"}

#### Copy Data across from staging Amazon S3 bucket to your S3 bucket

Issue the following command in the terminal, and replace the bucket name with your own one.
```bash
aws s3 cp --recursive --copy-props none s3://aws-dataengineering-day.workshop.aws/data/ s3://<YourBucketName>/tickets/
```

#### The data will be copied to your S3 Bucket and you will see the following:

![](/static/400/images/image97.jpeg)

#### Verify the Data

1.  Open the S3 console and view the data that was copied through CloudShell
    terminal.

2.  Your S3 bucket name will look like below:
    BucketName/bucket_folder_name/schema_name/table_name/objects/

3.  In our lab example this becomes:
    [/&lt;**BucketName**&gt;/tickets/dms_sample](https://s3.console.aws.amazon.com/s3/home)
    with a separate path for each table_name

    ![](/static/400/images/53.png)

4.  Navigate to one of the files and review it using [S3 Select](https://docs.aws.amazon.com/AmazonS3/latest/userguide/selecting-content-from-objects.html):
   a. Navigate in to the directory named **player** and select the check box next to the file name.  
   b. Click the **Actions** dropdown button and choose **Query with S3 Select**.  
   ![](/static/400/images/53-1.png)
   c. In the Query with S3 Select page, leave the default value for *Input Settings* and *SQL Query* and click **Run SQL query**.   
   ![](/static/400/images/53-2.png) 
   d. It will execute the specified SQL query and return the first 5 lines from the CSV file.  
   ![](/static/400/images/53-3.png)
   
> Note that column names are included in the file in the first row.

Explore the objects in the S3 directory further.

### Next Steps

In the next part of this lab, we will complete the following tasks:

-   [Extract, Transform and Load Data Lake with AWS Glue](/600.html)

### Appendix A: Self-Paced Data Lake Lab

If you If want to re-run the lab by yourself, please follow the lab instruction published in the GitHub:

[https://github.com/aws-samples/data-engineering-for-aws-immersion-day](https://github.com/aws-samples/data-engineering-for-aws-immersion-day)
