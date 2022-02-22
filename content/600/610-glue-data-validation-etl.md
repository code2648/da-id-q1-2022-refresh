---
title: "Data Validation and ETL"
weight: 610
---

::::expand{header="Expand to follow the instructions if you are starting the workshop in this module"}
Today, you are attending a formal event and you will have been sent your
access details beforehand. If in the future you might want to perform
these labs in your own AWS environment by yourself.

A 12-character access code (or ‘hash’) is the access code that grants
you permission to use a dedicated AWS account for the purposes of this
workshop.

1.  Go to <https://dashboard.eventengine.run/>, enter the access code
    and click Proceed:

![](/static/600/media/image3.png)

2.  On the Team Dashboard web page you will see a set of parameters that
    you will need during the labs. Best to save them to a text file
    locally, alternatively you can always go to this page to review
    them. Replace the parameters with the corresponding values from here
    where indicated in subsequent labs:

Because you’re at a formal event, some AWS resources have been
pre-deployed for your convenience, for example:

-   The source database connection in RDS DB Info module

![](/static/600/media/image4.png)

-   S3 Bucket, IAM role for the Glue lab etc

![](/static/600/media/image5.png)

1.  On the Team Dashboard, please click AWS Console to log into the AWS
    Management Console:

![](/static/600/media/image6.png)

4.  Click Open Console. For the purposes of this workshop, you will not
    need to use command line and API access credentials:

![](/static/600/media/image7.png)

Once you have completed these steps, you can continue with the rest of
this lab.
::::

#### PART A: Data Validation and ETL

##### Create Glue Crawler for initial full load data

1. Navigate to the [AWS Glue Console](https://console.aws.amazon.com/glue/home)

    ![](/static/600/media/image8.png)

2. On the AWS Glue menu, select **Crawlers**.

    ![](/static/600/media/image9.png)

3. Click **Add crawler**.

4. Enter `glue-lab-crawler` as the crawler name for initial data load.

5. Optionally, enter the description. This should also be descriptive and easily recognized and Click **Next**.

    ![](/static/600/media/image10.png)

6. Choose **Data stores**, **Crawl all folders** and **Click Next**

    ![](/static/600/media/image11.png)

7. On the **Add a data store** page, make the following selections:
    - For Choose a data store, click the drop-down box and select **S3**.
    - For Crawl data in, select **Specified path in my account**.
    - For Include path, browse to the target folder stored CSV files, e.g., **s3://xxx-dmslabs3bucket-xxx/tickets**

8. Click **Next**.

    ![](/static/600/media/image12.png)

9. On the **Add another data store page**, select **No**. and Click **Next**.

    ![](/static/600/media/image13.png)

10. On the **Choose an IAM role** page, make the following selections:  
    - Select **Choose an existing IAM role**.  
    - **For IAM role**, select &lt;stackname&gt;-**GlueLabRole**-&lt;RandomString&gt; pre-created for you. For example “dmslab-student-GlueLabRole-ZOQDII7JTBUM”

11. Click **Next**.  
    ![](/static/600/media/image14.png)

12. On the Create a schedule for this crawler page, for Frequency, select **Run on demand** and Click **Next**.
    ![](/static/600/media/image15.png)

13. On the Configure the crawler’s output page, click **Add database** to create a new database for our Glue Catalogue.
    ![](/static/600/media/image16.png)

14. Enter `ticketdata` as your database name and click **create**
    ![](/static/600/media/image17.png)

15. For **Prefix added to tables (optional)**, leave the field empty.

16. For **Configuration options (optional)**, select **Add new columns only** and keep the remaining default configuration options and Click **Next**.
    ![](/static/600/media/image18.png)

17. Review the summary page noting the Include path and Database output and Click **Finish**. The crawler is now ready to run.
    ![](/static/600/media/image19.png)

18. Tick the crawler name, click **Run crawler** button.
    ![](/static/600/media/image20.png)
    Crawler will change status from starting to stopping, wait until crawler comes back to ready state (the process will take a few minutes), you can see that it has created 15 tables.
    ![](/static/600/media/image21.png)

19.  In the AWS Glue navigation pane, click **Databases → Tables**. You can also click the **ticketdata** database to browse the tables.
  ![](/static/600/media/image22.png)

##### Data Validation Exercise 

1.  Within the Tables section of your **ticketdata** database, click the person table.
    ![](/static/600/media/image23.png)
    You may have noticed that some tables (such as person) have column headers such as col0,col1,col2,col3. In absence of headers or when the crawler cannot determine the header type, default column headers are specified.

    This exercise uses the person table in an example of how to resolve this issue.

2.  Click **Edit Schema** on the top right side.
    ![](/static/600/media/image24.png)

3.  In the Edit Schema section, double-click **col0** (column name) to open edit mode. Type `id` as the column name.

    Repeat the preceding step to change the remaining column names to match those shown in the following figure: `full_name`, `last_name` and `first_name`
    ![](/static/600/media/19.png)

4.  Click **Save**.

##### Data ETL Exercise 

**Pre-requisite:** To store processed data in parquet format, we need a new folder location for each table, eg. the full path for sport_team table look like this - `s3://<s3_bucket_name>/tickets/dms_parquet/sport_team`

Glue will create the new folder automatically, based on your input of the full file path, such as the example above. Please refer to the [user guide](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-folder.html) in terms of how to manually create a folder in S3 bucket.

1. In the left navigation pane, under ETL, click **AWS Glue Studio**.
    ![](/static/600/media/image26.png)

2. Choose “**View jobs**”
    ![](/static/600/media/image27.png)

3. Leave the “Visual with a source and target” option selected, and click “**Create**”
    ![](/static/600/media/image28.png)

4. Select the **Data source - S3 bucket** at the top of the graph.

5. In the panel on the right under “Data source properties - S3”, choose the `ticketdata` database from the drop down.

6. For Table, select the `sport_team` table.
    ![](/static/600/media/image29.png)

7. Select the **ApplyMapping** node. In the Transform panel on the right and change the data type of **id** column to `double` in the dropdown.
    ![](/static/600/media/image30.png)

8. Select the **Data target - S3 bucket** node at the bottom of the graph, and change the Format to **Parquet** in the dropdown. Under *Compression Type*, select **Uncompressed** from the dropdown.

9. Under “S3 Target Location”, select “**Browse S3**” browse to the “mod-xxx-dmslabs3bucket-xxx” bucket, select “**tickets**” item and press
    “**Choose**”.![](/static/600/media/image31.png)

10. In the textbox, append `dms_parquet/sport_team/` to the S3 url. The path should look similar to s3://mod-xxx-dmslabs3bucket-xxx/tickets<strong>/dms_parquet/sport_team/</strong> - don’t forget the **/** at the end. The job will automatically create the folder.
    ![](/static/600/media/image32.png)

11. Finally, select the **Job details** tab at the top. Enter `Glue-Lab-SportTeamParquet` under Name.

12. For **IAM Role**, select the role named similar to mod-xxx<strong>-GlueLabRole-</strong>xxx.

13. Scroll down the page and under **Job bookmark**, select **Disable** in the drop down. You can try out the bookmark functionality later in this lab.
    ![](/static/600/media/image33.png)

14. Press the **Save** button in the top right-hand corner to create the job.

15. Once you see the **Successfully created job** message in the banner, click the **Run** button to start the job.

16. Select **Jobs** from the navigation panel on the left-hand side to see a list of your jobs.

17. Select **Monitoring** from the navigation panel on the left-hand side to view your running jobs, success/failure rates and various other statistics.
    ![](/static/600/media/image34.png)

18. Scroll down to the **Job runs** list to verify that the ETL job has completed successfully. This should take about 1 minute to complete.
    ![](/static/600/media/image35.png)

19. We need to repeat this process for an additional 4 jobs, to transform the **sport_location, sporting_event, sporting_event_ticket** and **person** tables.

During this process, we will need to modify different column data types. We can either repeat the process above for each table, or we can clone the first job and update the details. The steps below describe how to clone the job - if creating manually each time, follow the above steps but make sure you use the updated values from the tables below.

1. Return to the **Jobs** menu, and select the **Glue-Lab-SportsTeamParquet** job by clicking the checkbox next to the name.
    ![](/static/600/media/image36.png)

2. Under the **Actions** dropdown, select **Clone job**. Update the job as per the following tables, then **Save** and **Run**.

**1. Sport_Location:** 

Create a **Glue-Lab-SportLocationParquet** job with the following attributes:

| **Task / Action**               | **Attribute**          | **Values**                            |
|---------------------------------|------------------------|---------------------------------------|
| “Data source - S3 bucket” node  | Database               | `ticketdata`                            |
|                                 | Table                  | `sport_location`                       |
| “Transform - ApplyMapping” node | Schema transformations | None                                  |
| “Data target - S3 bucket” node  | Format                 | Parquet                               |
|                                 | Compression Type       | Uncompressed                          |
|                                 | S3 target path         | `tickets/dms_parquet/sport_location/` |
| “Job details tab”               | Job Name   Compression Type            | `Glue-Lab-SportLocationParquet`  |
|                                 | IAM Role               | xxx-GlueLabRole-xxx                   |
|                                 | Job bookmark           | Disable                               |

***2. Sporting_Event:***

Create a **Glue-Lab-SportingEventParquet** job with the following attributes:

| **Task / Action**               | **Attribute**         | **Values**                                 |
|---------------------------------|-----------------------|--------------------------------------------|
| “Data source - S3 bucket” node  | Database              | `ticketdata`                               |
|                                 | Table                 | `sporting_event`                            |
| “Transform - ApplyMapping” node | Schema tranformations | column “start_date_time” =&gt; TIMESTAMP |
|                                 |                       | column “start_date” =&gt; DATE            |
| “Data target - S3 bucket” node  | Format                | Parquet                                    |
|                                 | Compression Type       | Uncompressed                          |
|                                 | S3 target path        | `tickets/dms_parquet/sporting_event/`      |
| “Job details tab”               | Job Name              | `Glue-Lab-SportingEventParquet`             |
|                                 | IAM Role              | xxx-GlueLabRole-xxx                        |
|                                 | Job bookmark          | Disable                                    |

***3. Sporting_Event_Ticket:***

Create a **Glue-Lab-SportingEventTicketParquet** job with the following attributes:

| **Task / Action**               | **Attribute**         | **Values**                                    |
|---------------------------------|-----------------------|-----------------------------------------------|
| “Data source - S3 bucket” node  | Database              | `ticketdata`                                    |
|                                 | Table                 | `sporting_event_ticket`                       |
| “Transform - ApplyMapping” node | Schema tranformations | column “id” =&gt; DOUBLE                      |
|                                 |                       | column “sporting_event_id” =&gt; DOUBLE     |
|                                 |                       | column “ticketholder_id” =&gt; DOUBLE        |
| “Data target - S3 bucket” node  | Format                | Parquet                                       |
|                                 | Compression Type       | Uncompressed                          |
|                                 | S3 target path        | `tickets/dms_parquet/sporting_event_ticket/` |
| “Job details tab”               | Job Name              | `Glue-Lab-SportingEventTicketParquet`           |
|                                 | IAM Role              | xxx-GlueLabRole-xxx                           |
|                                 | Job bookmark          | Disable                                       |

::::expand{header="Troubleshoot: Glue-Lab-SportingEventTicketParquet job failing with error 'Unsupported case of DataType'"}
Are you getting the error similar to below.
:::code{language=java showLineNumbers=false}
An error occurred while calling o115.pyWriteDynamicFrame. Unsupported case of DataType: com.amazonaws.services.glue.schema.types.LongType@4f722276 and DynamicNode: stringnode.
:::

This might be mostly due to not having CDC data written to different S3 folder/prefix in your CDC DMS task. Check Step 2 in [Create the CDC endpoint to replicate ongoing changes (Optional)](/400/401/430-main-lab#create-the-cdc-endpoint-to-replicate-ongoing-changes-(optional)) section of DMS Lab and make sure you are using correct folder name in your CDC Target endpoint (`rds-cdc-endpoint`).

::::

***4. Person:***

Create a **Glue-Lab-PersonParquet** job with the following attributes:

| **Task / Action**               | **Attribute**         | **Values**                   |
|---------------------------------|-----------------------|------------------------------|
| “Data source - S3 bucket” node  | Database              | `ticketdata`                   |
|                                 | Table                 | `person`                       |
| “Transform - ApplyMapping” node | Schema tranformations | column “id” =&gt; DOUBLE     |
| “Data target - S3 bucket” node  | Format                | Parquet                      |
|                                 | Compression Type       | Uncompressed                          |
|                                 | S3 target path        | `tickets/dms_parquet/person/` |
| “Job details tab”               | Job Name              | `Glue-Lab-PersonParquet`       |
|                                 | IAM Role              | xxx-GlueLabRole-xxx          |
|                                 | Job bookmark          | Disable                      |

##### Create Glue Crawler for Parquet Files 

1. In the Glue Studio navigation menu, select **Crawlers** to open the Glue Crawlers page in a new tab. Click **Add crawler**.
    ![](/static/600/media/image37.png)

2. For **Crawler name**, type `glue-lab-parquet-crawler` and Click **Next**.
    ![](/static/600/media/33.png)

3. In next screen **Specify crawler source type,** select **Data Stores** as choice **for Crawler source type** and click **Next.**

4. In Add a data store screen 
    - For **Choose a data store**, select “S3”.  
    - For **Crawl data** in, select “**Specified path in my account**”.  
    - For Include path, specify the S3 Path (Parent Parquet folder) that contains the nested parquet files e.g., s3://xxx-dmslabs3bucket-xxx/tickets/dms_parquet  
    - Click **Next**.   
    ![](/static/600/media/image39.png)
    ![](/static/600/media/image40.png)

5.  For Add another data store, select **No** and Click **Next**.
    ![](/static/600/media/image41.png)

6.  On the Choose an IAM role page, select **Choose an existing IAM role**.
    * For IAM role, select the existing role “xxx-**GlueLabRole**-xxx” and
    * Click **Next**.
    ![](/static/600/media/image42.png)

7.  For **Frequency**, select “Run On Demand” and Click **Next**.
    ![](/static/600/media/image43.png)

8.  For the crawler’s output database, choose your existing database which you created earlier e.g. **ticketdata**

9.  For the **Prefix added to tables** (optional), type `parquet_`
    ![](/static/600/media/image44.png)

10.  Review the summary page and click **Finish**.

11.  Click **Run Crawler**. Once your crawler has finished running, you should report that tables were added from 1 to 5, depending on how many parquet ETL conversions you set up in the previous section.
  ![](/static/600/media/image45.png)

**Confirm you can see the tables:**

1.   In the left navigation pane, click **Tables**.
2.   Add the filter `parquet` to return the newly created tables.
  ![](/static/600/media/image46.png)
