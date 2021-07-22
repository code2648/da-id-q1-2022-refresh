---
title: Transforming data with Glue
weight: 610
---

In this lab you will be completing the following tasks. You can choose to complete only Part-(A) to move to next lab where tables can be queried using Amazon Athena and Visualize with Amazon Quciksight


### Tasks Completed in this Lab:

- PART-(A): Data Validation and ETL
  - Create Glue Crawler for initial full load data
  - Data ETL Exercise
  - Create Glue Crawler for Parquet Files
- PART-(B): Glue Job Bookmark (Optional)
  - Step 1: Create Glue Crawler for ongoing replication (CDC Data)
  - Step 2: Create a Glue Job with Bookmark Enabled
  - Step 3: Create Glue crawler for Parquet data in S3
  - Step 4: Generate CDC data and to observe bookmark functionality
- PART-(C): Glue Workflows (Optional)
  - Overview
  - Creating and Running Workflows

Labs are also available at https://aws-dataengineering-day.workshop.aws/


### Get Started Using the Lab Environment

Please skip this section if you are running the lab on your own AWS
account.

Today, you are attending a formal event and you will have been sent your
access details beforehand. If in the future you might want to perform
these labs in your own AWS environment by yourself, you can follow
instructions on GitHub -
<https://github.com/aws-samples/data-engineering-for-aws-immersion-day>.

A 12-character access code (or ‘hash’) is the access code that grants
you permission to use a dedicated AWS account for the purposes of this
workshop.

1.  Go to <https://dashboard.eventengine.run/>, enter the access code
    and click Proceed:

<img src="/static/600/media/image3.png" alt="EventEngine Login" />

2.  On the Team Dashboard web page you will see a set of parameters that
    you will need during the labs. Best to save them to a text file
    locally, alternatively you can always go to this page to review
    them. Replace the parameters with the corresponding values from here
    where indicated in subsequent labs:

Because you’re at a formal event, some AWS resources have been
pre-deployed for your convenience, for example:

-   The source database connection in RDS DB Info module

<img src="/static/600/media/image4.png" />

-   S3 Bucket, IAM role for the Glue lab etc

<img src="/static/600/media/image5.png"  />

3.  On the Team Dashboard, please click AWS Console to log into the AWS
    Management Console:

<img src="/static/600/media/image6.png"  />

4.  Click Open Console. For the purposes of this workshop, you will not
    need to use command line and API access credentials:

<img src="/static/600/media/image7.png"  alt="EventEngine Open Console" />

Once you have completed these steps, you can continue with the rest of
this lab.

<span id="_PART-(A)" class="anchor"></span>

#### PART A: Data Validation and ETL

Create Glue Crawler for initial full load data
----------------------------------------------

1.  Navigate to the [AWS Glue
    service](https://console.aws.amazon.com/glue/home?region=us-east-1)

<img src="/static/600/media/image8.png" />

2.  On the AWS Glue menu, select **Crawlers**.

<img src="/static/600/media/image9.png"  />

3.  Click **Add crawler**.

4.  Enter **glue-lab-crawler** as the crawler name for initial data
    load.

5.  Optionally, enter the description. This should also be descriptive
    and easily recognized and Click **Next**.

<img src="/static/600/media/image10.png" />

6.  Choose **Data stores**, **Crawl all folders** and **Click Next**

<img src="/static/600/media/image11.png" />

7.  On the **Add a data store** page, make the following selections:

    1.  For Choose a data store, click the drop-down box and select
        **S3**.

    2.  For Crawl data in, select **Specified path in my account**.

    3.  For Include path, browse to the target folder stored CSV files,
        e.g., **s3://xxx-dmslabs3bucket-xxx/tickets**

8.  Click **Next**.

<img src="/static/600/media/image12.png" />

9.  On the **Add another data store page**, select **No**. and Click
    **Next**.

<img src="/static/600/media/image13.png" />

10.  On the **Choose an IAM role** page, make the following selections:

    1.  Select **Choose an existing IAM role**.

    2.  **For IAM role**, select
        &lt;stackname&gt;-**GlueLabRole**-&lt;RandomString&gt;
        pre-created for you. For example
        “dmslab-student-GlueLabRole-ZOQDII7JTBUM”

11.  Click **Next**.

<img src="/static/600/media/image14.png"  />

12.  On the Create a schedule for this crawler page, for Frequency,
    select **Run on demand** and Click **Next**.

<img src="/static/600/media/image15.png"  />

13.  On the Configure the crawler’s output page, click **Add database**
    to create a new database for our Glue Catalogue.

<img src="/static/600/media/image16.png"  />

14.  Enter **ticketdata** as your database name and click **create**

<img src="/static/600/media/image17.png" />

15.  For **Prefix added to tables (optional)**, leave the field empty.

16.  For **Configuration options (optional)**, select **Add new columns
    only** and keep the remaining default configuration options and
    Click **Next**.

<img src="/static/600/media/image18.png" />

17.  Review the summary page noting the Include path and Database output
    and Click **Finish**. The crawler is now ready to run.

<img src="/static/600/media/image19.png" />

18.  Tick the crawler name, click **Run crawler** button.

<img src="/static/600/media/image20.png"  />

Crawler will change status from starting to stopping, wait until crawler
comes back to ready state (the process will take a few minutes), you can
see that it has created 15 tables.

<img src="/static/600/media/image21.png"  />

19.  In the AWS Glue navigation pane, click **Databases** &gt;
    **Tables**. You can also click the **ticketdata** database to browse
    the tables.

<img src="/static/600/media/image22.png"  />

Data Validation Exercise 
------------------------

1.  Within the Tables section of your **ticketdata** database, click the
    person table.

<img src="/static/600/media/image23.png"  />

You may have noticed that some tables (such as person) have column
headers such as col0,col1,col2,col3. In absence of headers or when the
crawler cannot determine the header type, default column headers are
specified.
>
This exercise uses the person table in an example of how to resolve
this issue.

2.  Click **Edit Schema** on the top right side.

<img src="/static/600/media/image24.png" />

3.  In the Edit Schema section, double-click **col0** (column name) to
    open edit mode. Type “id” as the column name.

Repeat the preceding step to change the remaining column names to match
those shown in the following figure: full_name, last_name and
first_name

<img src="/static/600/media/19.png" />

4.  Click **Save**.

Data ETL Exercise 
-----------------

**Pre-requisite:** To store processed data in parquet format, we need a
new folder location for each table, eg. the full path for sport_team
table look like this – `s3://<s3_bucket_name>/tickets/dms_parquet/sport_team`

Glue will create the new folder automatically, based on your input of
the full file path, such as the example above. Please refer to the [user
guide](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-folder.html)
in terms of how to manually create a folder in S3 bucket.

1.  In the left navigation pane, under ETL, click **AWS Glue Studio**.

<img src="/static/600/media/image26.png" />

2.  Choose “**Create and Manage Jobs**”

<img src="/static/600/media/image27.png" />

3.  Leave the “Source and target added to the graph” option selected,
    and press “**Create**”

<img src="/static/600/media/image28.png" />

4.  Select the “Data source - S3 bucket” at the top of the graph.

5.  In the panel on the right under “Data source properties - S3”,
    choose the “**ticketdata**” database from the drop down.

6.  For Table, select the **sport_team** table.

<img src="/static/600/media/image29.png" />

7.  Select the “ApplyMapping” node. In the Transform panel on the right
    and change the data type of “id” column to double in the dropdown.

<img src="/static/600/media/image30.png" />

8.  Select the “Data target - S3 bucket” node at the bottom of the
    graph, and change the Format to **Parquet** in the dropdown.

9.  Under “S3 Target Location”, select “**Browse S3**” browse to the
    “mod-xxx-dmslabs3bucket-xxx” bucket, select “**tickets**” item and
    press
    “**Choose**”.<img src="/static/600/media/image31.png" />

10.  In the textbox, append **dms_parquet/sport_team/** to the S3 url.
    The path should look similar to
    s3://mod-xxx-dmslabs3bucket-xxx/tickets**/dms_parquet/sport_team/** -
    don’t forget the “/” at the end. The job will automatically create
    the folder.

<img src="/static/600/media/image32.png" />

11.  Finally, select the **Job details** tab at the top. Enter
    **Glue-Lab-SportTeamParquet** under Name.

12.  For “**IAM Role**”, select the role named similar to
    mod-xxx**-GlueLabRole-**xxx.

13.  Scroll down the page and under “**Job bookmark**”, select
    “**Disable**” in the drop down. You can try out the bookmark
    functionality later in this lab.

<img src="/static/600/media/image33.png" />

14.  Press the “**Save**” button in the top right-hand corner to create
    the job.

15.  Once you see the “**Successfully created job**” message in the
    banner, click the “**Run**” button to start the job.

16.  Select “**Jobs**” from the navigation panel on the left-hand side to
    see a list of your jobs.

17.  Select “**Monitoring**” from the navigation panel on the left-hand
    side to view your running jobs, success/failure rates and various
    other statistics.

<img src="/static/600/media/image34.png" />

18.  Scroll down to the “**Job runs**” list to verify that the ETL job
    has completed successfully. This should take about 1 minute to
    complete.<img src="/static/600/media/image35.png" />

19.  We need to repeat this process for an additional 4 jobs, to
    transform the **sport_location, sporting_event,
    sporting_event_ticket** and **person** tables.

During this process, we will need to modify different column data
types. We can either repeat the process above for each table, or we
can clone the first job and update the details. The steps below
describe how to clone the job - if creating manually each time, follow
the above steps but make sure you use the updated values from the
tables below.

20.  Return to the “**Jobs**” menu, and select the
    “**Glue-Lab-SportsTeamParquet**” job by clicking the small circle
    next to the name.

<img src="/static/600/media/image36.png" />

21.  Under the “**Actions**” dropdown, select “**Clone job**”. Update the
    job as per the following tables, then “**Save**” and “**Run**”.

#### **1. Sport_Location:** 

Create a **Glue-Lab-SportLocationParquet** job with the following
attributes:

| **Task / Action**               | **Attribute**          | **Values**                            |
|---------------------------------|------------------------|---------------------------------------|
| “Data source - S3 bucket” node  | Database               | ticketdata                            |
|                                 | Table                  | sport_location                       |
| “Transform - ApplyMapping” node | Schema transformations | None                                  |
| “Data target - S3 bucket” node  | Format                 | Parquet                               |
|                                 | S3 target path         | tickets/dms_parquet/sport_location/ |
| “Job details tab”               | Job Name               | Glue-Lab-SportLocationParquet         |
|                                 | IAM Role               | xxx-GlueLabRole-xxx                   |
|                                 | Job bookmark           | Disable                               |

***2. Sporting_Event:***

Create a **Glue-Lab-SportingEventParquet** job with the following
attributes:

| **Task / Action**               | **Attribute**         | **Values**                                 |
|---------------------------------|-----------------------|--------------------------------------------|
| “Data source - S3 bucket” node  | Database              | ticketdata                                 |
|                                 | Table                 | sporting_event                            |
| “Transform - ApplyMapping” node | Schema tranformations | column “start_date_time” =&gt; TIMESTAMP |
|                                 |                       | column “start_date” =&gt; DATE            |
| “Data target - S3 bucket” node  | Format                | Parquet                                    |
|                                 | S3 target path        | tickets/dms_parquet/sporting_event/      |
| “Job details tab”               | Job Name              | Glue-Lab-SportingEventParquet              |
|                                 | IAM Role              | xxx-GlueLabRole-xxx                        |
|                                 | Job bookmark          | Disable                                    |

***3. Sporting_Event_Ticket:***

Create a **Glue-Lab-SportingEventTicketParquet** job with the following
attributes:

| **Task / Action**               | **Attribute**         | **Values**                                    |
|---------------------------------|-----------------------|-----------------------------------------------|
| “Data source - S3 bucket” node  | Database              | ticketdata                                    |
|                                 | Table                 | sporting_event_ticket                       |
| “Transform - ApplyMapping” node | Schema tranformations | column “id” =&gt; DOUBLE                      |
|                                 |                       | column “sporting_event_id” =&gt; DOUBLE     |
|                                 |                       | column “ticketholder_id” =&gt; DOUBLE        |
| “Data target - S3 bucket” node  | Format                | Parquet                                       |
|                                 | S3 target path        | tickets/dms_parquet/sporting_event_ticket/ |
| “Job details tab”               | Job Name              | Glue-Lab-SportingEventTicketParquet           |
|                                 | IAM Role              | xxx-GlueLabRole-xxx                           |
|                                 | Job bookmark          | Disable                                       |

***4. Person:***

Create a **Glue-Lab-PersonParquet** job with the following attributes:

| **Task / Action**               | **Attribute**         | **Values**                   |
|---------------------------------|-----------------------|------------------------------|
| “Data source - S3 bucket” node  | Database              | ticketdata                   |
|                                 | Table                 | person                       |
| “Transform - ApplyMapping” node | Schema tranformations | column “id” =&gt; DOUBLE     |
| “Data target - S3 bucket” node  | Format                | Parquet                      |
|                                 | S3 target path        | tickets/dms_parquet/person/ |
| “Job details tab”               | Job Name              | Glue-Lab-PersonParquet       |
|                                 | IAM Role              | xxx-GlueLabRole-xxx          |
|                                 | Job bookmark          | Disable                      |

Create Glue Crawler for Parquet Files 
-------------------------------------

1.  In the Glue Studio naviation menu, select **Crawlers** to open the
    Glue Crawlers page in a new tab. Click **Add crawler**.

<img src="/static/600/media/image37.png" />

2.  For **Crawler name**, type **glue-lab-parquet-crawler** and Click
    **Next**.

<img src="/static/600/media/33.png" />

3.  In next screen **Specify crawler source type,** select **Data
    Stores** as choice **for Crawler source type** and click **Next.**

4.  In Add a data store screen

    1.  For **Choose a data store**, select “S3”.

    2.  For **Crawl data** in, select “**Specified path in my
        account**”.

    3.  For Include path, specify the S3 Path (Parent Parquet folder)
        that contains the nested parquet files e.g.,
        s3://xxx-dmslabs3bucket-xxx/tickets/dms_parquet

    4.  Click **Next**.

<img src="/static/600/media/image39.png" />

<img src="/static/600/media/image40.png" />

5.  For Add another data store, select **No** and Click **Next**.

<img src="/static/600/media/image41.png" />

6.  On the Choose an IAM role page, select **Choose an existing IAM
    role**.

For IAM role, select the existing role “xxx-**GlueLabRole**-xxx” and
Click **Next**.

<img src="/static/600/media/image42.png" />

7.  For **Frequency**, select “Run On Demand” and Click **Next**.

<img src="/static/600/media/image43.png" />

8.  For the crawler’s output database, choose your existing database
    which you created earlier e.g. “**ticketdata**”

9.  For the **Prefix added to tables** (optional), type "**parquet_**"

<img src="/static/600/media/image44.png" />

10.  Review the summary page and click **Finish**.

11.  Click **Run Crawler**. Once your crawler has finished running, you
    should report that tables were added from 1 to 5, depending on how
    many parquet ETL conversions you set up in the previous section.

<img src="/static/600/media/image45.png" />

Confirm you can see the tables:

12.  In the left navigation pane, click **Tables**.
13.  Add the filter "parquet" to return the newly created tables.

<img src="/static/600/media/image46.png" /><span
id="_Glue_Job_Bookmark:" class="anchor"></span>

#### PART B: Glue Job Bookmark (Optional):

****Pre-requisite: Completion of CDC part of DMS Lab ****

Step 1: Create Glue Crawler for ongoing replication (CDC Data)
--------------------------------------------------------------

Now, let’s repeat this process to load the data from change data
capture.

1.  On the AWS Glue menu, select Crawlers.

<img src="/static/600/media/image9.png" />

2.  Click **Add crawler**.

3.  Enter the crawler name for ongoing replication. This name should be
    descriptive and easily recognized (e.g.,
    "**glue-lab-cdc-crawler**").

4.  Optionally, enter the description. This should also be descriptive
    and easily recognized and Click
    **Next**.<img src="/static/600/media/image47.png" />

5.  Choose **Data Stores** as Crawler Source Type**, Crawl all folders**
    and Click **Next**

<img src="/static/600/media/image11.png" />

6.  On the Add a data store page, make the following selections:

    1.  For **Choose a data store**, click the drop-down box and select
        **S3**.

    2.  For **Crawl data in**, select **Specified path in my account**.

    3.  For **Include path**, enter the **target folder** for your DMS
        ongoing replication, e.g.,
        “s3://xxx-dmslabs3bucket-xxx/**cdc/dms_sample**”

7.  Click **Next**.

<img src="/static/600/media/image48.png" />

8.  On the **Add another data store page**, select **No** and Click
    **Next**.

<img src="/static/600/media/image49.png" />

9.  On the **Choose an IAM role** page, make the following selections:

    1.  Select **Choose an existing IAM role**.

    2.  **For IAM role**, select **xxx-GlueLabRole-xxx**. E.g.
        “dmslab-student-GlueLabRole-ZOQDII7JTBUM”

10.  Click **Next**.

<img src="/static/600/media/image50.png" />

11.  On the Create a schedule for this crawler page, for Frequency,
    select **Run on demand** and Click **Next**.

<img src="/static/600/media/image51.png" />

12.  On the Configure the crawler’s output page, select the existing
    **Database** for crawler output (e.g., "**ticketdata**").

13.  For **Prefix added to tables,** specify "**cdc_**"

14.  For Configuration options (optional), keep the **default**
    selections and click **Next**.

<img src="/static/600/media/image52.png"  />

15.  Review the summary page noting the Include path and Database target
    and Click **Finish**. The crawler is now ready to run.

<img src="/static/600/media/image53.png" />

16.  Tick the crawler name “**glue-lab-cdc-crawler”**, click **Run
    crawler** button.

17.  When the crawler is completed, you can see it has “Status” as
    **Ready,** Crawler will change status from starting to stopping,
    wait until crawler comes back to ready state, you can see that it
    has created **2 tables**.

<img src="/static/600/media/image54.png" />

18.  Click the database name (e.g., "**ticketdata**") to browse the
    tables. Specify "**cdc**" as the filter to list only newly imported
    tables.

<img src="/static/600/media/image55.png" />

Step 2: Create a Glue Job with Bookmark Enabled
-----------------------------------------------

1.  On the left-hand side of Glue Console, click on **Jobs** and then
    Click on **Add Job.**

<img src="/static/600/media/image56.png" />

2.  On the Job properties page, make the following selections:

    1.  For **Name**,
        type **Glue-Lab-TicketHistory-Parquet-with-bookmark**

    2.  For **IAM role**, choose existing role “xxx-**GlueLabRole**-xxx”

    3.  For **Type**, Select **Spark**

    4.  For **Glue Version**, select **Spark 2.4, Python 3 (Glue version
        2.0)** or whichever is the latest version

    5.  For **This job runs**, select **A proposed script generated by
        AWS Glue**.

    6.  For **Script file name**, use the **default**.

    7.  For **S3 path where the script is stored**, provide a unique
        Amazon S3 path to store the scripts. (You can keep the
        **default** for this lab.)

    8.  For **Temporary directory**, provide a unique Amazon S3
        directory for a temporary directory. (You can keep the
        **default** for this lab.)

3.  Expand the **Advanced properties** section. For Job bookmark, select
    **Enable** from the drop-down option.

4.  Expand on the **Monitoring** options, enable **Job metrics**.

5.  Click **Next**

<img src="/static/600/media/image57.png" />

6.  In **Choose a data source**, select **cdc_ticket_purchase_hist**
    as we are generating new data entries for **ticket_purchase_hist**
    table. Click **Next**

<img src="/static/600/media/56.png" />

7.  In **Choose a transform type**, select **Change Schema** and Click
    **Next**

<img src="/static/600/media/image59.png" />

8.  In Choose a data target:

    1.  Create tables in your data target

    2.  For **Data store**: select **Amazon** **S3**

    3.  Format: **parquet**

    4.  **Target path**:
        s3://xxx-dmslabs3bucket-xxx/**cdc_bookmark/ticket_purchase_history/data/**

    5.  Click **Next**

<img src="/static/600/media/image60.png" />

9.  In map the source columns to target columns window, leave everything
    as **default** and Click on **Save job and edit script**.

<img src="/static/600/media/59.png" />

10.  In the next window, review the job script and click on **Run job**,
    then click on **close** **mark** on the top right of the window to
    close the screen.

<img src="/static/600/media/60.png" />

11.  Once the job finishes its run, check the **S3 bucket** for the
    parquet partitioned data.

<img src="/static/600/media/61.png" />

#### Step 3: Create Glue crawler for Parquet data in S3 


1.  Once you have the data in S3 bucket, navigate to **Glue Console**
    and now we will crawl the parquet data in S3 to create data catalog.

2.  Click on **Add crawler**

<img src="/static/600/media/image64.png" />

3.  In crawler configuration window, provide crawler name as
    **glue_lab_cdc_bookmark_crawler** and Click **Next**.

<img src="/static/600/media/image65.png" />

4.  In **Specify** **crawler source type**, select **Data stores** and
    **Crawl all folders**. Click **Next**

<img src="/static/600/media/image66.png" />

5.  In **Add a data store**:

    1.  For **Choose a data store**, select **S3**

    2.  For the **Include path**, click the folder icon and choose your
        target S3 bucket, then append
        **/cdc_bookmark/ticket_purchase_history** , e.g.,
        “s3://xxx-dmslabs3bucket-xxx/cdc_bookmark/ticket_purchase_history”

6.  Click on **Next**

<img src="/static/600/media/image67.png" />

7.  For **Add another data** store, select **No** and click **Next**.

<img src="/static/600/media/image68.png" />

8.  In **Choose an IAM role**, select an existing IAM role contains
    **GlueLabRole** text. Something looks like this:
    xxx-**GlueLabRole**-xxx<img src="/static/600/media/image69.png" />

9.  For setting the **frequency** in create a schedule for this crawler,
    select “**Run on demand**”. Click **Next**

10.  For the crawler’s output:

    1.  For Database, select “**ticketdata**” database.

    2.  Optionally, add prefix to the newly created tables for easy
        identification. Provide the prefix as **bookmark_parquet_**

    3.  Click **Next**

<img src="/static/600/media/image70.png"  />

11.  Review all the details and click on **Finish**. Then **Run**
    **crawler**.
    <img src="/static/600/media/image71.png" />

12.  After the crawler finishes running, click on Databases, select
    “**ticketdata**” and view tables in this database. You will find the
    newly created table as
    “**bookmark_parquet_ticket_purchase_history**”

<img src="/static/600/media/image72.png" />

13.  Once the table is created, click on **Action** and from **dropdown**
    select **View Data.**

If it’s the first time you are using Athena in your AWS Account, click
**Get Started**

<img src="/static/600/media/image73.png" />

Then click **set up a query result location in Amazon S3** at the top

<img src="/static/600/media/image74.png" />

In the pop-up window in the **Query result location** field, enter
your s3 bucket location followed by **/**, so that it looks like
**s3://xxx-dmslabs3bucket-xxx/** and click
**Save**<img src="/static/600/media/image75.png" />

To select some rows from the table, try running:

```
SELECT * FROM
"ticketdata"."bookmark_parquet_ticket_purchase_history" limit 10;
```

To get a row count, run:

```
SELECT count(*) as recordcount FROM
"ticketdata"."bookmark_parquet_ticket_purchase_history";
```
Before moving on to next step, note the rowcount.

#### Step 4: Generate CDC data and to observe bookmark functionality

Ask your instructor generate more CDC data at source database, if you
ran the instructor setup on your own, then make sure to follow
“**Generate the CDC Data”** section from instructor prelab.

1.  To make sure the new data has been successfully generated, check the
    S3 bucket for cdc data, you will see new files generated. Note the
    time when the files were generated.

<img src="/static/600/media/image76.png" />

2.  Rerun the Glue job **Glue-Lab-TicketHistory-Parquet-with-bookmark**
    you created in Step 2

3.  Go to the Athena Console, and rerun the following query to notice
    the increase in row count:
```
SELECT count(*) as recordcount FROM
"ticketdata"."bookmark_parquet_ticket_purchase_history";
```
To review the latest transactions, run:
```
SELECT * FROM
"ticketdata"."bookmark_parquet_ticket_purchase_history" order by
transaction_date_time desc limit 100;
```

#### PART C: Glue Workflows (Optional, self-paced)

****Pre-requisite before creating workflow** - completed Part B**

Overview:
---------

In AWS Glue, you can use workflows to create and visualize complex
extract, transform, and load (ETL) activities involving multiple
crawlers, jobs, and triggers. Each workflow manages the execution and
monitoring of all its components. As a workflow runs each component, it
records execution progress and status, providing you with an overview of
the larger task and the details of each step. The AWS Glue console
provides a visual representation of a workflow as a graph.

Creating and Running Workflows:
-------------------------------

Above mentioned Part A (ETL with Glue) and Part B (Glue Job Bookmarks)
can be created and executed using workflows. Complex ETL jobs involving
multiple crawlers and jobs can also be created and executed using
workflows in an automated fashion. Below is a simple example to
demonstrate how to create and run workflows.

Try creating a new Glue Workflow to string together the two Crawlers and
one Job from part B as follows:

On-demand trigger -&gt; glue-lab-cdc-crawler -&gt;
Glue-Lab-TicketHistory-Parquet-with-bookmark -&gt;
glue_lab_cdc_bookmark_crawler

<img src="/static/600/media/image77.png" />

**To create a workflow:**

1.  Navigate to **AWS Glue Console** and under **ETL**, click on
    **Workflows**. Then Click on **Add Workflow**.

<img src="/static/600/media/image78.png" />

2.  Give the workflow name as “**Workflow_tickethistory**”. Provide a
    description (optional) and click on **Add Workflow** to create it.

3.  Click on the **workflow** and scroll to the bottom of the page. You
    will see an option **Add Trigger**. Click on that button.
    <img src="/static/600/media/image79.png" />

<img src="/static/600/media/image80.png" />

4.  In **Add Trigger** window, From Clone Existing and Add New options,
    click on **Add New**.

    1.  Provide **Name** as “**trigger1**”

    2.  Provide a **description**: Trigger to start workflow

    3.  **Trigger type**: **On-demand**.

    4.  Click on **Add**

Triggers are used to initiate the workflow and there are multiple ways
to invoke the trigger. Any scheduled operation or any event can activate
the trigger which in turn starts the workflow

<img src="/static/600/media/image81.png" />

5.  Click on **trigger1** to add a **new node**. New Node can be a
    crawler or job, depending upon the workflow you want to build.

<img src="/static/600/media/image82.png" />

6.  Click on **Add node,** a new window to add jobs or crawlers will
    open. Select the Crawler **glue-lab-cdc-crawler**, then **Add.**

7.  Click on the crawler and **Add Trigger** provide the following:

    1.  **Name**: **trigger2**

    2.  **Description**: Trigger to execute job

    3.  **Trigger** **type**: **Event**

    4.  **Trigger logic**: **Start after ALL watched event.** This will
        make sure that job starts once Glue Crawler finishes.

    5.  Click **Add**

<img src="/static/600/media/image83.png" />

8.  After **trigger2** is added to workflow, Click on **Add node,**
    select job **Glue-Lab-TicketHistory-Parquet-with-bookmark,** click
    **Add.**

9.  Click on the job and **Add Trigger** provide the following:

    1.  **Name**: **trigger3**

    2.  **Description**: Trigger to execute crawler

    3.  **Trigger** **type**: **Event**

    4.  **Trigger logic**: **Start after ANY watched event.** This will
        make sure that crawler starts once Glue job finishes processing
        of ALL data.

    5.  Click **Add**

10.  Click on **Add node,** Select the Crawler
    **glue_lab_cdc_bookmark_crawler**, then **Add.**

11.  Select your workflow, click on **Actions-&gt;Run** and this will
    start the first trigger “trigger1”

<img src="/static/600/media/image84.png" />

12.  Once the workflow is completed, you will observe that glue job and
    crawlers have been successfully executed.

#### Congratulations!! You have successfully completed this lab