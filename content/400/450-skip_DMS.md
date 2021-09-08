---
title: 'Option 3: Skip DMS Lab'
weight: 450
---

### Steps

1. ABOUT THE LAB SETUP
2. SETUP CLOUD9 IDE FOR DATA COPY
3. CREATE AN EC2 ENVIRONMENT WITH THE CONSOLE
4. COPY DATA ACROSS FROM STAGING AMAZON S3 BUCKET TO YOUR S3 BUCKET
5. VERIFY THE DATA
- NEXT STEPS
- APPENDIX A: SELF-PACED DATA LAKE LAB

### About the lab setup:

![](/static/400/images/image92.png)

RDS Postgres Database is used as a source of ticket sales system for sporting events. It stores transaction information about ticket sales price to selected people and ticket ownership transfer with additional tables for event details. AWS Database Migration Service (DMS) is used for a full data load from the Amazon RDS source to Amazon S3 bucket.

Before the Glue lab starts, you might choose to skip the DMS data migration, instead copy the source data to your S3 bucket directly.

In today's lab, you will copy the data from a centralized S3 bucket to your AWS account, crawl the dataset with AWS Glue crawler for metadata creation and transform the data with AWS Glue to Query data and create a View with Athena and Build a dashboard with Amazon QuickSight.


### Setup Cloud9 IDE for Data Copy

In AWS Cloud9, a development environment (or just environment) is a
place where you store your development project's files and where you
run the tools to develop your applications. In this tutorial, you
create a special kind of environment called an EC2 environment and
then work with the files and tools in that environment.

#### Create an EC2 Environment with the Console

1.  Sign in to the AWS Cloud9 console as follows:[https://console.aws.amazon.com/cloud9/](https://console.aws.amazon.com/cloud9/).

2.  After you sign in to the AWS Cloud9 console, in the top navigation
    bar, choose the AWS Region where your workshop is running. For a list of available AWS Regions, see [AWS
    Cloud9](https://docs.aws.amazon.com/general/latest/gr/rande.html#cloud9_region)
    in the *AWS General Reference*.

![](/static/400/images/image93.png)

3.  If a welcome page is displayed, for **New AWS Cloud9 environment**, choose **Create environment**. 

![](/static/400/images/image94.png)

Or:

![](/static/400/images/image95.png)

4.  On the **Name environment** page, for **Name**, type a name for your
    environment. For this tutorial, use **my-demo-environment**.

5.  For **Description**, type something about your environment. For this
    tutorial, use This environment is for the AWS Cloud9 tutorial.

6.  Choose **Next step**.

7.  On the **Configure settings** page, for **Environment type**, choose
    **Create a new instance for environment (EC2)**.

:::alert{type="warning"}
Choosing **Create a new instance for environment (EC2)** might result
in possible charges to your AWS account for Amazon EC2.
:::

8.  For **Instance type**, leave the default choice. This choice has
    relatively low RAM and vCPUs, which is sufficient for this
    tutorial.


:::alert{type="info"}
Choosing instance types with more RAM and vCPUs might result in
additional charges to your AWS account for Amazon EC2.
:::

9.  For **Platform**, choose the type of Amazon EC2 instance that AWS
    Cloud9 will create and then connect to this environment: **Amazon
    Linux** or **Ubuntu**.

10.  For **Cost-saving setting**, choose the amount of time until AWS Cloud9 shuts down the Amazon EC2 instance for the environment after all web browser instances that are connected to the IDE for the environment have been closed. Or **leave** the default choice.


:::alert{type="info"}
Choosing a longer time period might result in more charges to your AWS
account.
:::

11.  Leave the default settings for **Network settings (advanced)**.

12.  Choose **Next step**.

13.  On the **Review** page, choose **Create environment**. Wait while AWS Cloud9 creates your environment. This can take several minutes.

After AWS Cloud9 creates your environment, it displays the AWS Cloud9
IDE for the environment.

If AWS Cloud9 doesn't display the IDE after at least five minutes,
there might be a problem with your web browser, your AWS access
permissions, the instance, or the associated virtual private cloud
(VPC). For possible fixes, see [Cannot Open an
Environment](https://docs.aws.amazon.com/cloud9/latest/user-guide/troubleshooting.html#troubleshooting-env-loading)
in *Troubleshooting*.

### Copy Data across from staging Amazon S3 bucket to your S3 bucket

Open Cloud9 Console from AWS and you will see the terminal screen in the bottom:

![](/static/400/images/image96.jpeg)

1.  Generate a key pair by issuing the command `ssh-keygen`

2.  Press enter 3 times to take the default choices.

3.  Upload the public key to your EC2 region:
```
aws ec2 import-key-pair --key-name "lfworkshop" --public-key-material file://~/.ssh/id_rsa.pub
```

4.  Issue the following command in the terminal, and replace the bucket
    name with your own one.
```
aws s3 cp --recursive s3://aws-dataengineering-day.workshop.aws/data/ s3://<YourBucketName>/tickets/
```

#### The data will be copied to your S3 Bucket and you will see the following:

![](/static/400/images/image97.jpeg)

### Verify the Data

1.  Open the S3 console and view the data that was copied from Cloud9
    terminal.

2.  Your S3 bucket name will look like below :
    BucketName/bucket_folder_name/schema_name/table_name/objects/

3.  In our lab example this becomes: [/**BucketName**/tickets/dms_sample](https://s3.console.aws.amazon.com/s3/home?region=us-east-1)
    with a separate path for each table_name

![](/static/400/images/image98.png)

4.  Download one of the files:

    - Select the **check box** next to the object name and click **Download** in the pop-up window.

    - Click **Save File**.

    - Open the file.

![](/static/400/images/image99.jpeg)

5.  Note that column names are included in the file in the first row. Explore the objects in the S3 directory further.

![](/static/400/images/image100.png)


### Next Steps

In the next part of this lab, we will complete the following tasks:

-   Extract, Transform and Load Data Lake with AWS Glue

### Appendix A: Self-Paced Data Lake Lab

If you If want to re-run the lab by yourself, please follow the lab instruction published in the GitHub:

[https://github.com/aws-samples/data-engineering-for-aws-immersion-day](https://github.com/aws-samples/data-engineering-for-aws-immersion-day)