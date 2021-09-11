---
title: "DMS Migration Lab"
weight: 430
---

### Steps

1. Introduction
2. Create the Subnet Group
3. Create the Replication Instance
4. Create the DMS Source Endpoint
5. Create the Target Endpoint
6. Create a task to perform the initial full copy
7. Create the CDC endpoint to replicate ongoing changes (Optional)
8. Create a task to perform the ongoing replication (Optional)

### Introduction

This lab will give you an understanding of the AWS Database Migration Service (AWS DMS). You will migrate data from an existing Amazon Relational Database Service (Amazon RDS) Postgres database to an Amazon Simple Storage Service (Amazon S3) bucket that you create.

![](/static/400/images/30.png)

In this lab you will complete the following tasks:

1. Create a subnet group within the DMS Lab VPC
2. Create a DMS replication instance
3. Create a source endpoint
4. Create a target endpoint
5. Create a task to perform the initial migration of the data.

Optionally, you can add ongoing replication of data changes on the source: (Only one of the DMS replication instances will enable this feature.)

6. Create target endpoint for CDC files to place these files in a separate location than the initial load files
7. Create a task to perform the ongoing replication of data changes

Your instructor has already created and populated the RDS Postgres database that you will use as your source endpoint in this lab. If you have deployed instructor lab, then get RDS endpoint from there.

Labs are also available in GitHub - https://github.com/aws-samples/data-engineering-for-aws-immersion-day

### Create the Subnet Group

1. On the [DMS console](https://console.aws.amazon.com/dms/v2/home#createSubnetGroup), select Subnet Groups and Create subnet group.
   ![](/static/400/images/31.png)
   - In the Identifier box, type a descriptive name that you will easily recognize (e.g., `dms-lab-subnet-grp`).
   - In the Description box, type an easily recognizable description (e.g., `Replication instance for production data system`).
   - For VPC, select the name of the VPC that you created earlier with AWS CloudFormation template. VPC name ending with **dmslstudv1**.
  The subnet list populates in the Available Subnets pane.
   - Select as many subnets as you want and click Add. The selected subnets move to the Subnet Group pane.

   ::alert[DMS requires at least two separate availability zones to be selected.]{type="info"}

   ![](/static/400/images/32.png)

2. Click Create subnet group

3. On the DMS console, the subnet group status displays Complete.

   ![](/static/400/images/33.png)

### Create the Replication Instance

1. On the DMS console, select [Replication instances](https://console.aws.amazon.com/dms/v2/home#createReplicationInstance) to create a new replication instance.
   ![](/static/400/images/34.png)

   - For Name, type a name for the replication instance that you will easily recognize. (e.g., `DMS-Replication-Instance`).
   - For Description, type a description you will easily recognize. (e.g., `DMS Replication Instance`).
   - For Instance class, choose **dms.t3.medium**
   - Select the latest engine version
   - For VPC, select the name of the VPC that you created earlier with AWS CloudFormation template or preprovisoned for you. VPC name ending with **dmslstudv1**.
   - For Multi AZ, from the dropdown select **Dev or test workload (Single-AZ)**
   ![](/static/400/images/35.png)

   - Click Advanced to expand the section.
   - Select the security group with **sgdefault** in the name.
   ![](/static/400/images/36.png)

2. Leave other fields at their default values, then click **Create**.
3. The DMS console displays creating for the instance status. When the replication instance is ready, the status changes to available. While replication instance is spinning up, you can proceed to next step for DMS endpoint creation.
   ![](/static/400/images/37.png)

### Create the DMS Source Endpoint

Please proceed to create your endpoints, without waiting for the step above.

1. On the DMS console, select Endpoints to create one source [Endpoint](https://console.aws.amazon.com/dms/v2/home#createNewEndpoint)
   ![](/static/400/images/38.1.png)
   - select **Source Endpoint** type.
   - For Endpoint identifier, select your easily recognized name (e.g. `rds-source-endpoint`)
   - For Source engine, select `PostgreSQL`.
   - For "Access to Endpoint database", select "Provide access information manually"
   - Enter the Server name, get the **Database Endpoint** from Environment Setup module on your event engine dashboard. 
   ![](/static/400/images/db_endpoint.png)
      Or if you ran instructor lab then take recorded endpoint from the instructor [pre-lab](https://console.aws.amazon.com/cloudformation/home#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false) (e.g. “dmslabinstance.ccla1oozkrry.us-east-1.rds.amazonaws.com”).
   - For Port, enter `5432`.
   - For SSL mode, choose `none`.
   - For User name, type `master`.
   - For Password, type `master123`.
   - For Database name, type `sportstickets`.

_(These credentials were defined directly in the CFN template.)_

2. Accept other defaults and then click Create endpoint to create the endpoint. When available, the endpoint status changes to active.
3. Check the replication instance created previously. Make sure the status is available.
   ![](/static/400/images/40.png)
4. Select your newly created source endpoint, and choose Test connection on the Actions drop-down list.
   ![](/static/400/images/41.png)
   > If the test fails then ensure that 1/ you have followed the steps to allow inbound access to the RDS database instance security group, and 2/ your database credentials are correct
5. Click Run test. This step tests connectivity to the source database system. If successful, the message “Connection tested successfully” appears.
   ![](/static/400/images/42.png)

::alert[Note for instructor: In case the test fail, two possible reasons: a) RDS security group restriction. To resolve this, revisit instructor lab and follow section “Changing RDS security Group” to troubleshoot. b) RDS is not accessible because its disk is full. The solution is VACUUM FULL on the DB, or free up some disk space by deleting unnecessary auto-snapshots.]{type="info"}

### Create the Target Endpoint

Before start, make sure you have the following values handy (on your event dashboard).  
* ARN of DMSLabRole to access S3 - It looks like "arn:aws:iam::\<account number\>\:role/xxx-DMSLabRoleS3-xxxx"
* S3 Bucket name - It looks like “xxx-dmslabs3bucket-xxxx”
![](/static/400/images/43.1.png)

1. On the DMS console, select Endpoint to create a target [Endpoint](https://console.aws.amazon.com/dms/v2/home#createNewEndpoint).
   ![](/static/400/images/43.png)

   - For Endpoint type, select **Target endpoint**.
   - For Endpoint identifier, type an easily recognized name such as `s3-target-endpoint`
   - For Target engine, choose **s3**.
   - For Service access role ARN, paste the DMSLabRoleS3 ARN number noted earlier
   - For Bucket name, paste the S3 Bucket Name noted earlier
   - For Bucket folder, type `tickets`.
     ![](/static/400/images/44.png)
   - Expand the Endpoint settings section.
   - Click on the __Use endpoint connection__ attributes checkbox, type in `addColumnName=true` in the Extra connection attributes box”
   ![](/static/400/images/dms_settings.png)
   - Expand the Test endpoint connection (optional) section, and choose your “VPC name with dmslstudv1” on the VPC drop-down list.
   - Click Run test. This step tests connectivity to the source database system. If successful, the message “Connection tested successfully” appears.
     ![](/static/400/images/45.png)

2. Click Create Endpoint. When available, the endpoint status changes to active.
   ![](/static/400/images/46.png)

### Create a task to perform the initial full copy

1. On the DMS console, select **Database Migration Tasks**.
2. Click Create Task.
   - Type an easily recognized Task name e.g. `dms-full-dump-task`.
   - Select your Replication instance from drop down.
   - Select your Source endpoint from drop down.
   - Select your Target endpoint from drop down.
   - For Migration type choose **Migrate existing data**.
     ![](/static/400/images/47.png)
   - Expand Task Settings.
   - Select the **Enable CloudWatch logs** check box.  
     ![](/static/400/images/48.png)
   - Go to **Table Mappings**.
   - Click on Add new selection rule and select “Enter a Schema” in Schema field.
   - For Schema name, select `dms_sample` . Keep the settings for the remaining fields
     ![](/static/400/images/49.png)

3. Click **Create task**. Your task is created and starts automatically.
   ::alert[The complete creation and data extraction process takes 5 to 15 minutes.]{type="info"}

4. Once complete, the console displays **100%** progress.
   ![](/static/400/images/50.png)

5. Select your task and explore the summary. Scroll down and you can observe all table information loaded in S3 from RDS by DMS
   ![](/static/400/images/51.png)
   ![](/static/400/images/52.png)

6. Open the S3 console and view the data that was copied by DMS.

Your S3 bucket name will look like below : **BucketName/bucket_folder_name/schema_name/table_name/objects/**

In our lab example this becomes: */dmslab-student-dmslabs3bucket-woti4bf73cw3/tickets/dms_sample* with a separate path for each table_name)
![](/static/400/images/53.png)

7. Download one of the files:  
   a. Select the check box next to the file name and click **Download** in the pop-up window.  
   b. Click **Save File**.  
   c. Open the file.  
   
You will notice that the file contains the column headers in the first row as requested by the **“addColumnName=true”** connection attribute we included when we created the s3 target endpoint. Note that column names are included in the file in the first row.
![](/static/400/images/54.png)
Explore the objects in the S3 directory further.

### Create the CDC endpoint to replicate ongoing changes (Optional)

As of now we are enabling only one schema replication for CDC. Copy the following values from the Event Engine Team Dashboard.
* ARN of DMSLabRole to access S3 - It looks like “arn:aws:iam::\<account number\>\:role/xxx- DMSLabRoleS3-xxxx“
* S3 Bucket name - It looks like “xxx-dmslabs3bucket-xxxx”
![](/static/400/images/43.1.png)

1. On the DMS console, select Endpoints.
   ![](/static/400/images/55.png)

2. Click Create endpoint.
   - For Endpoint type, select **Target**.
   - For Endpoint identifier, type an easily recognized name e.g. `rds-cdc-endpoint`
   - For Target engine, choose **s3**.
   - For Service Access Role ARN, paste the ARN value that you noted earlier
   - For Bucket name, type the name of the s3 bucket you noted down earlier
   - For Bucket folder, type **cdc**.
     ![](/static/400/images/56.png)
   - Click Endpoint-specific settings to expand the section.
   - In the Extra connection attributes box, type `addColumnName=true`. This attribute includes the column names in the files in the S3 bucket.
   - Expand the Test endpoint connection (optional) section, and choose your **dmslstudv1** name on the VPC drop-down list.
   - Click Run test. This step tests connectivity to the source database system. If successful, the message “Connection tested successfully” appears.
     ![](/static/400/images/57.png)

3. Click Create endpoint.
4. When available, the endpoint status changes to active.
   ![](/static/400/images/58.png)

### Create a task to perform the ongoing replication (Optional)

1. On the DMS console, select Database Migration Tasks.
   ![](/static/400/images/59.png)

2. Click Create Task.
   - Type an easily recognized Task Identifier. For example `cdctask`.
   - Select your Replication instance.
   - Select your Source endpoint.
   - Select your Target endpoint as cdc endpoint created in the previous section.
   - For Migration type, choose **Replicate data changes only**.
   - Select the Start task on create check box.
     ![](/static/400/images/60.png)
   - In Task Settings, Select the Enable CloudWatch logs check box. Do not enable the validation.
   ![](/static/400/images/61.1.png)
   - Go to Table Mappings.
   - Click on Add new selection rule and select “Enter a Schema” in Schema field.
   - For Schema name, select `dms_sample` . Keep the settings for the remaining fields
   ![](/static/400/images/62.png)

3. Click Create task. Your task is created and starts automatically. You can see status as ongoing replication, after couple of minutes.
   ![](/static/400/images/63.png)

Once complete, the console displays 100% complete.

4. Your instructor will generate CDC activity which above migration task will capture. If you ran the instructor setup on your own, then make sure to follow **[Generate the CDC Data](/static/400/410-pre-lab-1.html#generate-and-replicate-the-cdc-data-optional)** section from instructor lab.

You may need to wait 5 to 10 minutes for CDC data to first reflect in your RDS postgres database and then picked up by DMS CDC migration task.

5. Once the CDC Data gets replicated, you can navigate to CDC task details, see **Table statistics** section and verify it, as shown below:

   ::alert[In case you see DMS CDC task in fail/error status. Make sure your replication instance version is greater than 3.3.2 and it is large enough (dms.t2.medium or above) to run CDC replication task]{type="info"}

   ![](/static/400/images/64.png)

6. Open the S3 console and view the CDC data that was copied by DMS.

   Your S3 bucket name will look like below : BucketName/bucket_folder_name/schema_name/table_name/objects/

   In our lab example this becomes: */dmslab-student-dmslabs3bucket-woti4bf73cw3/cdc/dms_sample* with a separate path for each table_name

   ![](/static/400/images/65.png)

7. Download one of the files:

   - Select the check box next to the object name and click Download in the pop-up window.
   - Click Save File.
   - Open the file.  
  You will notice that the file contains the column headers in the first row as requested by the “addColumnName=true” connection attribute we included when we created the s3 target endpoint.

   ![](/static/400/images/67.png)

Note that file name has date time - 20190529-230604667.csv

You can see the header is included and the operation column is added at the beginning of each row. The file below shows updates (U) to the table along with the values after the update. Inserts (I) show data after the insert and Deletes (D) show data before the delete.
![](/static/400/images/68.png)

Explore the objects in the S3 directory further.
