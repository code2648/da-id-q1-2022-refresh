---
title: Lake Formation Lab
weight: 1210
---

## Steps

- Setup Network Configuration    
- Create an IAM role to use with Lake Formation
- Create Glue JDBC connection for RDS
- Lake Formation – Add Administrator and start workflows using Blueprints.
- Explore the Underlying Components of a Blueprint    
- Explore workflow results in Athena    
- [Optional] Grant fine grain access controls to Data Lake user
- [Optional] Verify data permissions using Athena


## Setup Network Configuration for AWS Glue (additional read)

If you use Amazon Virtual Private Cloud (Amazon VPC) to host your AWS
resources, you can establish a private connection between your VPC and
AWS Glue. You use this connection to enable AWS Glue to communicate with
the resources in your VPC without going through the public internet.

Amazon VPC is an AWS service that you can use to launch AWS resources in
a virtual network that you define. With a VPC, you have control over
your network settings, such the IP address range, subnets, route tables,
and network gateways. To connect your VPC to AWS Glue, you define an
interface VPC endpoint for AWS Glue. When you use a VPC interface
endpoint, communication between your VPC and AWS Glue is conducted
entirely and securely within the AWS network.

## Create an IAM role to use with Lake Formation: (additional read)

With AWS Lake Formation, you can import your data using *workflows*. A
workflow defines the data source and schedule to import data into your
data lake. You can easily define workflows using *blueprints*, or
templates, that Lake Formation provides.

When you create a workflow, you must assign it an AWS Identity and
Access Management (IAM) role that enables Lake Formation to set up the
necessary resources on your behalf to ingest the data. In this lab,
we’ve pre-created an IAM role for you, called <br />
**random-LakeFormationWorkflowRolerandom**

## Create Glue JDBC connection for RDS 

1.  Navigate to the AWS Glue console:
    <u>https://console.aws.amazon.com/glue/home?region=us-east-1</u>

2.  On the AWS Glue menu (left hand menu), select **Connections**.

![](/static/1200/media/image8.jpg)

3.  Click **Add Connection**.

4.  Enter **glue-rds-connection** as the connection name.

5.  Choose **JDBC** for connection type.

6.  Optionally, enter the description. This should also be descriptive and easily recognizable. Click **Next**.

![](/static/1200/media/image9.jpg)

7. Input JDBC URL with the format of **jdbc\:postgresql://<RDS_Server_Name>:5432/sportstickets**
    - a. If you are running the lab in an AWS hosted event, get the RDS_Server_Name from your instructor.
    - b. If you are running the lab outside of AWS event, find the **DMSInstanceEndpoint** value on the Cloudformation stack dmslab-instructor Outputs tab.
    
8.  Enter **master** as username, **master123** as **Password**

9.  For **VPC**, select the pre-created VPC ending with **dmslstudv1**

10. For **Subnet,** choose one of the existing **private_subnet**

11. Select the **security group** with **sgdefault** in the name.

![](/static/1200/media/image11.jpg)

12. Click **Next to** complete the **glue-rds-connection** setup. To
    test it, select the connection, and choose **Test connection**.

![](/static/1200/media/image12.jpg)

13. Choose the pre-created **IAM role** (e.g., [random_string]-**LakeFormationWorkflowRole**-[random_string]),then click **Test Connection.**

![](/static/1200/media/image13.jpg)
 

## Lake Formation – Add Administrator and start workflows using Blueprints. 

Navigate to the AWS Lake Formation service: [https://console.aws.amazon.com/lakeformation/](https://console.aws.amazon.com/lakeformation/home?region=us-east-1#databases)

![](/static/1200/media/image14.png)

1. If you are logging into the lake formation console for the first time then you must add administrators first. In order to do that follow Steps 2 and 3. Else skip to Step 4.

2. Click **Add administrators**

![](/static/1200/media/image15.jpg)

3.  Add **TeamRole** Role as the Lake Formation Administrator and Click **Save**

![](/static/1200/media/image16.jpg)

4.  Navigate to **Databases** on left pane. Select **ticketdata** and
    click on **Actions**, select **Grant** to grant permissions. If you
    can’t see any databases, make sure to complete **Part A of Lab 2.
    ETL with AWS Glue**

![](/static/1200/media/image17.png)

5.  Under “**IAM Users and Roles**”, select two roles: the Lake
    Formation role that was precreated:
    **random-LakeFormationWorkflowRole-random** and
    **TeamRole**. Grant **super** permissions for **Database permissions
    and Grantable permissions.**

![](/static/1200/media/image18.jpg)

6.  Select **Actions-Edit** on the **ticketdata** database

![](/static/1200/media/image19.png)

7.  Clear the checkbox **Use only IAM access control** and click
    **Save**. Changing the default security setting so that access to
    Data Catalog resources (databases and tables) is managed by Lake
    Formation permissions.

![](/static/1200/media/image20_1.JPG)

8.  On the left pane navigate to **Blueprints** and click **Use blueprints**.
![](/static/1200/media/image21_1.JPG)

    a.  For Blueprint Type, select Database snapshot.

    b.  Under Import Source:
        -  For Database Connection, choose glue-rds-connection
        -  For Source Data Path, enter sportstickets/dms_sample/player

![](/static/1200/media/image22.png)

    c.  Under Import Target:
        -  For Target Database, choose ticketdata
        -  For Target storage location browse and select the **xxx-dmslabS3bucket-xxx** created in the previous lab.

![](/static/1200/media/image23.jpg)

        -  Add /lakeformation at the end of the bucket url path, e.g.s3://mod-08b80667356c4f8a-dmslabs3bucketnh54wqg771lk/lakeformation
        -  For Data Format, choose Parquet

![](/static/1200/media/image24.jpg)

    d.  For Import Frequency, Select Run On Demand

    e.  For Import Options:
        -  Give a Workflow Name RDS-S3-Glue-Workflow
        -  For the IAM 
        -  For Table prefix, type lakeformation

![](/static/1200/media/image25.jpg)

9.  Leave other options as default, click **Create**, and wait for the
    console to report that the workflow was successfully created.

10. Once the blueprint gets created, select it and click **Action -
    Start.** There may be a delay of 5-10 seconds for the blueprint
    showing up. You may have to **hit** **refresh** button.

11. Once the workflow starts executing, you will see the status changes
    from **running** -> **discovering** -> **Completed**

![](/static/1200/media/image26.png)

## Explore the Underlying Components of a Blueprint 

The Lake Formation blueprint creates a Glue Workflow, which orchestrates Glue ETL jobs (both 
python shell and pyspark), Glue crawlers, and triggers.  It will take somewhere between 20-30 mins to
finish its first execution. In the meantime, let us drill down to see what it creates for us.

1.  On the **Lake Formation console**, in the navigation pane, choose
    **Blueprints**

2.  In the **Workflow section**, click on the **Workflow name**. This
    will direct you to the Workflow run page. Click on the **Run Id.**

![](/static/1200/media/image27.png)

3.  Here you can see the graphical representation of the Glue workflow
    built by Lake Formation blueprint. Highlighting and clicking on
    individual components will display the details of those components
    (name, description, job run id, start time, execution time)

4.  To understand what all Glue Jobs got created as a part of this
    workflow, in the navigation pane, click on **Jobs**.

5.  Every job comes with history, details, script and metrics tab.
    Review each of these tabs for any of the python shell or pyspark
    jobs.

![](/static/1200/media/image28.jpg)

## Explore workflow results in Athena 

1.  Navigate to the **Lake Formation** Console. <u>https://console.aws.amazon.com/lakeformation/home?region=us-east-1#databases</u>

2.  Navigate to **Databases** on the left panel and select **ticketdata**

3.  Click on **View tables**

![](/static/1200/media/image29.png)

4. Select table **lakeformation_sportstickets_dms_sample_player**. As per our configuration above, Lake Formation tables were prefixed with **lakeformation**

5. Click **Action -> View Data**

![](/static/1200/media/image30.png)

This will now take you to **Athena** console.

If you see a “Get Started” page, it’s because it’s the first time we’re using Athena in this AWS Account. To proceed, click **Get Started**.

![](/static/1200/media/image31.png)

Then click **set up a query result location in Amazon S3** at the top

![](/static/1200/media/image32.png)

In the pop-up window in the **Query result location** field, enter your s3 bucket location followed by /, so that it looks like **s3://xxx-dmslabs3bucket-xxx/** and click **Save**

![](/static/1200/media/image33.png)

On Athena Console, you can run some queries using query editor:

![](/static/1200/media/image34.png)

To select some rows from the table, try running:

```
SELECT * FROM "ticketdata"."lakeformation_sportstickets_dms_sample_player" limit 10;
```

To get a row count, run:

```
SELECT count(*) as recordcount FROM "ticketdata"."lakeformation_sportstickets_dms_sample_player" limit 10;
```

**Congratulations!!! You have completed lake formation lab. To explore more fine grain data lake security feature, continue to next section.**

## \[Optional\] Grant fine grain access controls to Data Lake user 

Before we start querying the data, let us create an IAM User
**datalake_user** and grant column level access on the table created by
the Lake formation workflow above, to **datalake_user.**

1.  Navigate to **IAM Console** and click on **Add** **User**.

![](/static/1200/media/image35.png)

2.  Create a user named **datalake_user** and give it a password:
    **Master123!**

![](/static/1200/media/image36.jpg)

3.  Next click on **Permissions**

4.  Choose **Attach existing policies directly** and search for
    **AthenaFullAccess**

![](/static/1200/media/image37.jpg)

5.  Keep navigating to the next steps until reached the end. Review the
    details and click on “**Create User**”.

6.  On the final screen, write down the sign-in link and hit **Close**

![](/static/1200/media/image38.jpg)

7. Click on the **datalake_user** user, select **add inline policy**, and switch to the **JSON** tab.
![](/static/1200/media/image39.jpg)

![](/static/1200/media/image40_1.JPG)

Use the following json snippet replacing <mark>your_dmslabs3bucket_unique_name</mark> with the name of your dmslabs3bucket, e.g. **arn\:aws\:s3:::mod-08b80667356c4f8a-dmslabs3bucketnh54wqg771lk**

```
{
    "Version": "2012-10-17", 
    "Statement": [ 
        { 
        "Effect": "Allow", 
        "Action": [ 
        "s3\:Put*", 
        "s3\:Get*", 
        "s3\:List*" 
        ], 
        "Resource": [ 
"arn\:aws\:s3:::<your_dmslabs3bucket_unique_name>/*" ] 
        } 
    ] 
}
```

8.  Give a name to the policy, such as **athena_access**. Then **Create Policy**. IAM user with required policies have been created.

9.  Next, Navigate to **Lake Formation console**, under **Permissions**
    choose **Data permissions**.

10. Choose **Grant**. 

![](/static/1200/media/image41.png)

11. Once the **Grant permissions** window opens up:

    1.  For **IAM user and roles**, choose **datalake_user.**

    2.  For **Database**, choose **ticketdata**

    3.  The **Table** list populates.

    4.  For **Table**, choose
        **lakeformation_sportstickets_dms_sample_player**.

    5.  For Columns, select **Include Columns** and choose **id**,
        **first_name**

    6.  For **Table permissions**, uncheck **Super** and choose
        **Select**.

    7.  Click **Grant**

![](/static/1200/media/image42.png)

## \[Optional\] Verify data permissions using Athena 

Using Athena, let us now explore the data set as the **datalake_user**.

1.  Write down your **Account ID** and then sign out of your AWS Account. 

![](/static/1200/media/image43_1.JPG)

2.  On the same web page, sign back in as the IAM user
    **datalake_user**, using **Master123!** as password. Note: remove
    *hyphens* ‘-‘ from your Account ID
![](/static/1200/media/image45_1.JPG)

    - a. Make sure to change the region to **us-east-1** (N. Virginia)
![](/static/1200/media/image50.JPG)

    - b.  Navigate to **Athena console**

![](/static/1200/media/image51.png)

3.  If you see a “Get Started” page, it is the first time using Athena in this AWS Account. To proceed, click **Get Started**. 

![](/static/1200/media/image52.png)

Then click **set up a query result location in Amazon S3** at the top.

![](/static/1200/media/image32.png)

In the pop-up window, enter your s3 bucket location followed by "**/**" in the **Query result location** box. It looks like **s3://xxx-dmslabs3bucket-xxx/** and click **Save**

![](/static/1200/media/image33.png)

4.  Next, ensure database **ticketdata** is selected on right hand panel.

5.  Now run a select query:

```
SELECT * FROM "ticketdata"."lakeformation_sportstickets_dms_sample_player" limit 10;
```

6.  You will notice that the **datalake_user** can **only see** the
    columns **id, first_name** in the ‘select all‘ query result. The
    datalake_user cannot see **last_name**, **sports_team_id**,
    **full_name** columns in the table.

![](/static/1200/media/image53.jpg)

This illustrates that AWS Lake Formation can provide granular access at table and column level to IAM users.

**Congratulations!! You have successfully completed this lab!**