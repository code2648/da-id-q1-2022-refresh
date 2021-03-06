---
title: "Glue Job Bookmark (Optional)"
weight: 620
---

#### PART B: Glue Job Bookmark (Optional):

::alert[Pre-requisite: Completion of CDC part of DMS Migration Lab]{type="warning"}

##### Step 1: Create Glue Crawler for ongoing replication (CDC Data)

Now, let’s repeat this process to load the data from change data
capture.

1. On the AWS Glue menu, select **Crawlers**.
   ![](/static/600/media/image9.png)

2. Click **Add crawler**.

3. Enter the crawler name for ongoing replication. This name should be descriptive and easily recognized (e.g. `glue-lab-cdc-crawler`).

4. Optionally, enter the description. This should also be descriptive and easily recognized and Click **Next**.
   ![](/static/600/media/image47.png)

5. Choose **Data Stores** as Crawler Source Type, **Crawl all folders**
    and Click **Next**
   ![](/static/600/media/image11.png)

6. On the Add a data store page, make the following selections:

    - For **Choose a data store**, click the drop-down box and select **S3**.  
    - For **Crawl data in**, select **Specified path in my account**.  
    - For **Include path**, enter the **target folder** for your DMS ongoing replication, e.g., “s3://xxx-dmslabs3bucket-xxx/**cdc/dms_sample**”  

7. Click **Next**.
   ![](/static/600/media/image48.png)

8. On the **Add another data store page**, select **No** and Click **Next**.
   ![](/static/600/media/image49.png)

9. On the **Choose an IAM role** page, make the following selections:
    - Select **Choose an existing IAM role**.  
    - **For IAM role**, select **xxx-GlueLabRole-xxx**. E.g. “dmslab-student-GlueLabRole-ZOQDII7JTBUM”  

10. Click **Next**.
   ![](/static/600/media/image50.png)

11. On the Create a schedule for this crawler page, for Frequency, select **Run on demand** and Click **Next**.
   ![](/static/600/media/image51.png)

12. On the Configure the crawler’s output page, select the existing **Database** for crawler output (e.g., **ticketdata**).

13. For **Prefix added to tables,** specify `cdc_`

14. For Configuration options (optional), keep the **default** selections and click **Next**.
   ![](/static/600/media/image52.png)

15. Review the summary page noting the Include path and Database target and Click **Finish**. The crawler is now ready to run.
   ![](/static/600/media/image53.png)

16. Tick the crawler name **glue-lab-cdc-crawler**, click **Run crawler** button.

17. When the crawler is completed, you can see it has “Status” as **Ready,** Crawler will change status from starting to stopping, wait until crawler comes back to ready state, you can see that it has created **2 tables**.
   ![](/static/600/media/image54.png)

18. Click the database name (e.g., `ticketdata`) to browse the tables. Specify `cdc` as the filter to list only newly imported tables.
   ![](/static/600/media/image55.png)

##### Step 2: Create a Glue Job with Bookmark Enabled

1. On the left-hand side of Glue Console, click on **Jobs** and then click on **Create**
   ![](/static/600/media/image56.png)

2. Create a **Glue-Lab-TicketHistory-Parquet-with-bookmark** job with the following attributes:

| **Task / Action**               | **Attribute**          | **Values**                            |
|---------------------------------|------------------------|---------------------------------------|
| “Data source - S3 bucket” node  | Database               | `ticketdata`                            |
|                                 | Table                  | `cdc_ticket_purchase_hist`                       |
| “Transform - ApplyMapping” node | Schema transformations | None                                  |
| “Data target - S3 bucket” node  | Format                 | Parquet                               |
|                                 | Compression Type       | Uncompressed                          |
|                                 | S3 target path         | `s3://xxx-dmslabs3bucket-xxx/cdc_bookmark/ticket_purchase_history/data/` |
| “Job details tab”               | Job Name               | `Glue-Lab-TicketHistory-Parquet-with-bookmark`  |
|                                 | IAM Role               | xxx-GlueLabRole-xxx                   |
|                                 | Job bookmark           | Enable                               |

![](/static/600/media/image86.png)

![](/static/600/media/image87.png)

![](/static/600/media/image88.png)

![](/static/600/media/image89.png)

![](/static/600/media/image90.png)

3. Click the **Save** button in the top right-hand corner to create the job.

4. Once you see the **Successfully created job** message in the banner, click the **Run** button to start the job.

5. Once the job finishes its run, check the **S3 bucket** for the parquet partitioned data.
   ![](/static/600/media/61.png)

##### Step 3: Create Glue crawler for Parquet data in S3 

1. Once you have the data in S3 bucket, navigate to **Glue Console** and now we will crawl the parquet data in S3 to create data catalog.
2. Click on **Add crawler**
   ![](/static/600/media/image64.png)
3. In crawler configuration window, provide crawler name as `glue_lab_cdc_bookmark_crawler` and Click **Next**.
   ![](/static/600/media/image65.png)
4. In **Specify** **crawler source type**, select **Data stores** and **Crawl all folders**. Click **Next**
   ![](/static/600/media/image11.png)
5. In **Add a data store**:
    - For **Choose a data store**, select **S3**
    - For the **Include path**, click the folder icon and choose your target S3 bucket, then append `/cdc_bookmark/ticket_purchase_history `, e.g., “s3://xxx-dmslabs3bucket-xxx/cdc_bookmark/ticket_purchase_history”
6. Click on **Next**
   ![](/static/600/media/image67.png)
7. For **Add another data** store, select **No** and click **Next**.
   ![](/static/600/media/image68.png)
8. In **Choose an IAM role**, select an existing IAM role contains **GlueLabRole** text. Something looks like this: xxx-**GlueLabRole**-xxx![](/static/600/media/image69.png)
9. For setting the **frequency** in create a schedule for this crawler, select “**Run on demand**”. Click **Next**
10. For the crawler’s output:  
    - For Database, select “**ticketdata**” database.  
    - Optionally, add prefix to the newly created tables for easy dentification. Provide the prefix as `bookmark_parquet_`  
    - Click **Next**  
   ![](/static/600/media/image70.png)

11. Review all the details and click on **Finish**. Then **Run crawler**.
   ![](/static/600/media/image71.png)

12. After the crawler finishes running, click on Databases, select **ticketdata** and view tables in this database. You will find the newly created table as **bookmark_parquet_ticket_purchase_history**
   ![](/static/600/media/image72.png)

13. Select the checkbox next to the **bookmark_parquet_ticket_purchase_history** table, click on **Action** and from **dropdown** select **View Data**

This will launch the Athena query editor.

If it’s the first time you are using Athena in your AWS Account, expand **Get Started using Athena...** below.

::::expand{header="Get Started using Athena"}
Click **View settings** to set up a query result location in Amazon S3.

![](/static/600/media/image91.png)

Then click **Manage**

![](/static/600/media/image92.png)

In the pop-up window in the **Location of query result** field, click **Browse**. Choose the bucket with the string  **dmslabs3bucket** (e.g: *mod-3fccddd609114925-dmslabs3bucket-1wrxdypwl36xo*), then click **Save**.

Append `athenaquery/` at the end of the S3 location, don't forget the **/** at the end. Click on **Save**.

![](/static/600/media/image93.png)
::::

To select some rows from the table, try running:
:::code{language=sql showLineNumbers=false showCopyAction=true}
SELECT * FROM
"ticketdata"."bookmark_parquet_ticket_purchase_history" limit 10;
:::

To get a row count, run:
:::code{language=sql showLineNumbers=false showCopyAction=true}
SELECT count(*) as recordcount FROM
"ticketdata"."bookmark_parquet_ticket_purchase_history";
:::
Before moving on to next step, note the rowcount.

##### Step 4: Generate CDC data and to observe bookmark functionality

Ask your instructor generate more CDC data at source database, if you ran the instructor setup on your own, then make sure to follow **Generate the CDC Data** section from instructor prelab.

1.  To make sure the new data has been successfully generated, check the S3 bucket for cdc data, you will see new files generated. Note the time when the files were generated.
   ![](/static/600/media/image76-1.png)
2.  Rerun the Glue job **Glue-Lab-TicketHistory-Parquet-with-bookmark** you created in Step 2
3.  Go to the Athena Console, and rerun the following query to notice the increase in row count:
 
   :::code{language=sql showLineNumbers=false showCopyAction=true}
   SELECT count(*) as recordcount FROM
   "ticketdata"."bookmark_parquet_ticket_purchase_history";
   :::

   To review the latest transactions, run:  
   :::code{language=sql showLineNumbers=false showCopyAction=true}
   SELECT * FROM
   "ticketdata"."bookmark_parquet_ticket_purchase_history" order by
   transaction_date_time desc limit 100;
   :::
