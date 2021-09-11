---
title: "Option 2: AutoComplete DMS Lab"
weight: 440
---

### Introduction

Labs in the Data Engineering workshop are to be completed in sequence. This lab is designed to automate the Data Lake hydration with AWS Database Migration Service (AWS DMS), so we can fast forward to the following Glue lab. 

If you prefer to get hands-on with AWS DMS service, please choose [Option 1: DMS Main Lab](/static/400/401.html). 


### Pre-requisite

::alert[**Student PreLab Setup** is not required. Otherwise, duplicate S3 buckets and IAM roles will appear in your lab environment. If that happens, please use the resources with a name prefix ***auto-dmslab-***.]{type="warning"}

- [**Instructor Prelab Setup**](/400/401/410-pre-lab-1) - is completed and the source RDS database is fully populated.
- **RDS Database Server Name** - If you are in an AWS hosted event, you can get this value from Event Engine Team Dashboard. Otherwise, check the **Outputs** tab on your [CloudFormation](https://console.aws.amazon.com/cloudformation/home) Console, note down the RDS Server value named `DMSInstanceEndpoint`.
![](/static/400/images/78.png)

- **`dms-cloudwatch-logs-role`** & **`dms-vpc-role`** -  Check if the Identity and Access Managment (IAM) roles exist in your workshop AWS account. Go to the [IAM console](https://console.aws.amazon.com/iam/home#/roles), copy & paste the names in the search box respectively. 

Note whether these role are present or not. In this example screenshot the cloudwatch role is absent. 
![](/static/400/images/79.png)


### AutoComplete DMS

::alert[Make sure you select the appropriate AWS region]{type="info"}

1. Click the "Deploy to AWS" icon and open the link in a new web browser tab. It will load the CloudFormation dashboard to start the DMS automation process:
   
   :button[Deploy to AWS]{iconName="external" variant="primary" href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=auto-dmslab&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/SkipDMSlab_student_CFN.yaml" target="_blank"}

   Select correct choice for the parameters **DMSCWRoleCreated** and **DMSVPCRoleCreated**. For the **ServerName** parameter enter the database endpoint that you obtained from the pre-requisite. Under Capabilities, Check the box "I acknowledge that ...", then click on "Create Stack" to create the stack.

	It completes the following tasks on your behalf:
   - Set up the workshop enviroment to auto complete the DMS lab	
   - Create a DMS subnet group within the VPC
   - Create a DMS replication instance
   - Create a source endpoint for RDS source database
   - Create a target endpoint for full data load
   - Create a target endpoint for CDC
   - Create a task to perform the initial full data migration
   - Create a task to support the ongoing replication of data changes(CDC)


2. Select the Parameters as explained below: 
    - **DMSCWRoleCreated**: - If you have this role in your account, keep to **true**. If you do NOT have the role then choose **false**.
	- **DMSVPCRoleCreated**: - **false** if the role doesn't exist. Otherwise, change to **true**
	- **ServerName**: - Enter the RDS Database Server Name.  It likes this: dmslabinstance.xxxx<region>.rds.amazonaws.com
	![](/static/400/images/80.png)

3. Under Capabilities, check the box to acknowledge the policy and then click on Create Stack. 
	![](/static/400/images/23.png) 

4. The stack launch may take 5-6 minutes. Wait until your stack status advances to "CREATE_COMPLETE".
	![](/static/400/images/81.png) 

5. At this point, the source data has been fully loaded from RDS database to your S3 bucket via DMS. Go to [AWS DMS console](https://console.aws.amazon.com/dms/v2/home#tasks), you should see two **Database migration tasks** are 100% completed. If not, please wait until they are finished, then proceed to the [Glue lab](../600.html)
	![](/static/400/images/82.png) 
