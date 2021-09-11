---
title: "Instructor PreLab (not required in AWS event)"
weight: 410
---

::alert[The PreLab is only required if you are running the DMS lab outside of an AWS Event using Event Engine (EE), or in your own AWS Account.]{type="info"}

### Steps

1. Create the Instructor Environment, including a RDS Postgres database as the data source.
2. Changing RDS Security Group
3. Access Database from SQL Client (Optional)
4. Generate and Replicate the CDC Data (Optional)

### Limit Instruction:

This immersion day requires each student to have their own account. Sharing an account with multiple students by creating IAM users may result in hitting some of the dafult account limits listed below:

- VPC - VPCs per Region 5
- Glue - Number of crawlers per account 50
- Glue - Number of concurrent jobs runs per account 50
- Glue - Maximum DPUs used by a role at one time 300
- S3 - Number of buckets per account 100
- Athena - Number of DDL queries you can submit at the same time 20
- Athena - Number of DML queries you can submit at the same time 20
- RDS - Make sure you have enough disk space available in your RDS instance, if want to run DMS Change Data Capture (CDC) as generating large amount of data can exhaust RDS disk space.
- DMS - Make sure you have enough disk space available in your DMS replication instance, if want to run DMS Change Data Capture (CDC) as transferring large amount of CDC data can exhaust disk space.

### Introduction

The Database Migration Services (DMS) hands-on lab provide a scenario, where participant learns to hydrate Amazon S3 data lake with a relation database. To achieve that, participants need a source endpoint and this guide helps instructors set up a PostgreSQL database with public endpoint as the source database.

**In this lab, you will complete the following tasks using AWS CloudFormation template:**

1. Create the source database environment.
2. Hydrate the source database environment.
3. Update the source database environment to demonstrate CDC (Change Data Capture) replication within DMS.
4. Create Lambda function to trigger CDC data which will be replicated to Amazon S3 by DMS CDC endpoint.

**Relevant information about this lab:**

- Expected setup time | 15 minutes
- Source database name | sportstickets
- Source schema name | dms_sample

Instructor will provide source database details to participants during main lab to configure source endpoint.

### Create the Instructor Environment

In this section, you are going to create a PostgreSQL RDS instance as data source for AWS Data Migration Service to consume by lab attendees for data migration to Amazon S3 data lake.

::alert[Make sure you select the appropriate region and use the same region throughtout the workshop]{type="warning"}

1. Sign in to the Console where you will host the source database environment.

2. Click the **Deploy to AWS** button below to stand up the DMS pre-lab 1 workshop infrastructure.

    :button[Deploy to AWS]{iconName="external" variant="primary" href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=dmslab-instructor&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/DMSLab_instructor_CFN.yaml" target="_blank"}

3. Leave the default values for the Parameters and Check the box "I acknowledge that ...", then click on "Create Stack" to create the stack

![](/static/400/images/CloudFormation-Stack.png)

> In case you aren't able to launch the quick create stack, you can download the [template file](https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/DMSLab_instructor_CFN.yaml) and then follow the steps to [create stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) manually.

::alert[Please make sure the Postgres database is fully populated before proceed with the DMS lab. It takes 15 to 20 minutes to finish, after the stack is launched.]{type="info"}

You can see all resources listed below:

![](/static/400/images/5.png)

Go to the Outputs tabs of AWS CloudFormation stack and note down the instance Endpoint information for your RDS endpoint, which will be similar to information shown in below screenshot

![](/static/400/images/6.png)

### Changing RDS Security Group

Currently your RDS source end point is not open to connect to outside world for security reason. You need to open RDS security group to accept traffic from intended range of IP address. As it is difficult to determine range of IP address of workshop environment, so to have smooth experience of running lab you can temporally allow inbound traffic from all IP address (0.0.0.0/0 CIDR range).

::alert[It is not best practice to allow ALL CIDR range in your database security group. You should never apply open to all IP CIDR range while working on actual workload. If you are in a **self-paced workshop**, the better secure way is to whitelist an IP address from DMS lab, ie. add an Elastic IP address of a NAT Gateway to the RDS security group.]{type="warning"}

Follow below steps to open security group for students to connect with source RDS database for DMS full data and CDC data dump:

1.  Go to the [RDS](https://console.aws.amazon.com/rds/home#databases:) and double click on [“dmslabinstance”](https://console.aws.amazon.com/rds/home#database:id=dmslabinstance;is-cluster=false) DB identifier as shown below:
    ![](/static/400/images/7.png)

2.  Click VPC security groups under Connectivity & security tab as shown below:
    ![](/static/400/images/8.png)
3.  In Security group screen, Go to Inbound tab and click on Edit as shown below
    ![](/static/400/images/9.png)
4.  Update Inbound rule to “Anywhere” from hard coded value “72.21.196.67/32” , as shown in below screen. Make sure to remove the “Anywhere” inbound rule from security group, as soon as you are done with DMS lab.
    ![](/static/400/images/10.png)

5.  If you are running both instructor and student labs in a single AWS account, for example in a self-paced environment, replace the "Anywhere" by an IP address from student lab instead. In this example screenshot, we will allow AutoComplete DMS lab to access the RDS.
    ![](/static/400/images/86.png)

6.  Go to [VPC NAT gateways Console](https://console.aws.amazon.com/vpc/home#NatGateways:), and look for the IP address you need to add to the RDS security group.

    - If you are running the [Student PreLab](/static/400/420-pre-lab-2.html), note down the IP address tagged with "dmslab-student".

    ![](/static/400/images/84.png)

    - Or if you are running the [AutoComplete DMS Lab](/static/400/440-auto-complete-lab.html), copy the IP address tagged with "auto-dmslab".

    ![](/static/400/images/85.png)

7.  Click on Save. Now everyone will be able to connect to source RDS instance for lab purpose to ingest data using DMS endpoint.

::alert[Make sure to remove “Anywhere” inbound rule from security group as soon as you are done with DMS main lab. Optionally, You can read through the documentation to better understand the source database environment. The GitHub repository for aws-database-migration-samples is located here: https://github.com/aws-samples/aws-database-migration-samples/tree/master/PostgreSQL/sampledb/v1]{type="warning"}

### Access Database from SQL Client (Optional)

You can follow below instruction to setup SQL Workbench to access your Postgres Database from SQL client:
https://aws.amazon.com/getting-started/tutorials/create-connect-postgresql-db/

Database Name: `sportstickets`
Database credentials: master/master123

In SQL Workbench:
Run following query to find out all Schema and table created.
```sql
SELECT * FROM pg_catalog.pg_tables;
```

Ensure the following 2 functions exists. If anything is missing, check the solution at Troubleshoot section.
```sql
SELECT * FROM pg_stat_user_functions WHERE funcname in ('generateticketactivity','generatetransferactivity')
```
Use following query to analyze a table
`select * from schemaname.tablename;`

**For example:**
```sql
select * from pg_catalog.pg_tables;
```
![](/static/400/images/12.png)
**For example:**
```sql
select * from dms_sample.player;
```
![](/static/400/images/13.png)

Following sections are optional, you only need to execute if you want to show change data capture replication with DMS.

### Generate and Replicate the CDC Data (Optional)

::alert[This step is not required at your initial lab environment setup. Once the full table load of DMS lab is completed, you can start to generate extra transactions in source database to demonstrate DMS CDC (Change Data Capture) functionality.]{type="warning"}

::alert[If you are an instructor at AWS Event expand the following section to find the instructions]{type="info"}

::::expand{header="Expand to find the instructions to Send CDC data in AWS Event"}
`GenerateCDCData` lambda function is deployed in the event central account and not team account. Hence you need to federate in to the event central account by navigating **Actions → Federated Login** as shown below.
![](/static/400/images/ee-federated-login.png)
::::

Navigate to [Lambda console](https://console.aws.amazon.com/lambda/home#/functions) and you will see a pre-built Lambda function named **GenerateCDCData**.

![](/static/400/images/14.png)

1. Click on the function and scroll down. You will see the code for this function. Copy the below query and paste it in the placeholder (value) of this code line:
   “ var query_cmd= ”<insert-SQL-query-here>” ”

2. Run this query first: `select dms_sample.generateticketactivity(10)`;
    ![](/static/400/images/15.png)

    This query will generate 10 ticket sales in batches of 1-6 tickets to randomly selected people for a random price (within a range.) A record of each transaction is recorded in the ticket_purchase_hist table.

3. Click on Save and then click on Test to run the function. You can create an empty event as shown here:
    ![](/static/400/images/16.png)
    You will see no error in lambda log
    ![](/static/400/images/17.png)

4. Once you've sold some tickets you can run the generateTransferActivity procedure. The following will transfer tickets from the owner to another person. The whole "batch" of tickets purchased is transferred 80% of the time and 20% of the time an individual ticket is transferred.

    Run this query next in the lambda function: `select dms_sample.generatetransferactivity(10);`

    Click on **Save** and then click on **Test** to run the function.

    ![](/static/400/images/18.png)

::alert[When enabling CDC functionality in DMS, only one DMS instance/task should activate “Ongoing replication” to avoid conflicts. When replicating to multiple targets, the processing to fan out the updates should begin with the Amazon S3 bucket, that is the target of the DMS task responsible for Ongoing replication. The process should not begin with the source database, as only one CDC process should be tracking and setting the last committed transaction that was replicated.]{type="info"}

### Troubleshooting

::::expand{header="Failed to run Lambda function 'GenerateCDCData’"}

![](/static/400/images/71.png)

**Cause**

The source DB **sportstickets** setup is interrupted. Some database objects, such as the function **generateticketactivity()** will be missing.

**Resolution**

Go to [EC2 console](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:), reboot the instance `DMSLabEC2`. It will reload the DB and create any objects that were missing. Due to the re-run issue, the table sporting_event_ticket will be doubled in size at each reboot. You can manually drop the table via the following query, before the reboot. Then wait for 20 minutes before checking the missing DB object again.

:::code{language=sql showLineNumbers=false showCopyAction=true}
DROP TABLE dms_sample.sporting_event_ticket CASCADE
:::
::::

::::expand{header="RDS source database is out of storage space"}

Or you may see ‘No Space left on device’ error from DMSLabEC2 system log
![](/static/400/images/72.png)
![](/static/400/images/73.png)
![](/static/400/images/74.png)

**Cause**
Check the knowledge center [here](https://aws.amazon.com/premiumsupport/knowledge-center/diskfull-error-rds-postgresql/)

**Resolution**
Increase the RDS instance disk size, as a quick fix.
![](/static/400/images/75.png)
![](/static/400/images/76.png)
![](/static/400/images/77.png)
::::