---
title: "Glue Workflows (Optional)"
weight: 630
---

#### PART C: Glue Workflows (Optional, self-paced)

::alert[Pre-requisite before creating workflow - completed Glue Job Bookmark lab]{type="warning"}

##### Overview:

In AWS Glue, you can use workflows to create and visualize complex extract, transform, and load (ETL) activities involving multiple
crawlers, jobs, and triggers. Each workflow manages the execution and monitoring of all its components. As a workflow runs each component, it records execution progress and status, providing you with an overview of the larger task and the details of each step. The AWS Glue console provides a visual representation of a workflow as a graph.

##### Creating and Running Workflows:

Above mentioned Part A (ETL with Glue) and Part B (Glue Job Bookmarks) can be created and executed using workflows. Complex ETL jobs involving multiple crawlers and jobs can also be created and executed using workflows in an automated fashion. Below is a simple example to demonstrate how to create and run workflows.

Try creating a new Glue Workflow to string together the two Crawlers and one Job from part B as follows:

On-demand trigger → glue-lab-cdc-crawler → Glue-Lab-TicketHistory-Parquet-with-bookmark → glue_lab_cdc_bookmark_crawler

![](/static/600/media/image77.png)

**To create a workflow:**

1.  Navigate to **AWS Glue Console** and under **ETL**, click on **Workflows**. Then Click on **Add Workflow**.
    ![](/static/600/media/image78.png)
2.  Give the workflow name as `Workflow_tickethistory`. Provide a description (optional) and click on **Add Workflow** to create it.
3.  Click on the **workflow** and scroll to the bottom of the page. You will see an option **Add Trigger**. Click on that button.
    ![](/static/600/media/image79.png)
4.  In **Add Trigger** window, From Clone Existing and Add New options, click on **Add New**.
    - Provide **Name** as `Trigger1`
    - Provide a **description**: `Trigger to start workflow`
    - **Trigger type**: **On-demand**.
    - Click on **Add**
    Triggers are used to initiate the workflow and there are multiple ways the trigger which in turn starts the workflow
    ![](/static/600/media/image81.png)
5.  Click on **Trigger1** to add a **new node**. New Node can be a crawler or job, depending upon the workflow you want to build.
    ![](/static/600/media/image82.png)
6.  Click on **Add node,** a new window to add jobs or crawlers will open. Select the Crawler `glue-lab-cdc-crawler`, then **Add.**
7.  Click on the crawler and **Add Trigger** provide the following:
    - **Name**: `Trigger2`
    - **Description**: `Trigger to execute job`
    - **Trigger** **type**: **Event**
    - **Trigger logic**: **Start after ALL watched event.** This will make sure that job starts once Glue Crawler finishes.
    - Click **Add**
    ![](/static/600/media/image83.png)
8.  After **Trigger2** is added to workflow, Click on **Add node,** select job `Glue-Lab-TicketHistory-Parquet-with-bookmark`, click
    **Add.**
9.  Click on the job and **Add Trigger** provide the following:
    - **Name**: `Trigger3`
    - **Description**: `Trigger to execute crawler`
    - **Trigger** **type**: **Event**
    - **Trigger logic**: **Start after ANY watched event.** This will make sure that crawler starts once Glue job finishes processing of ALL data.
    - Click **Add**
10. Click on **Add node,** Select the Crawler `glue_lab_cdc_bookmark_crawler`, then **Add.**
11. Select your workflow, click on **Actions → Run** and this will start the first trigger **Trigger1**
    ![](/static/600/media/image84.png)
12. Once the workflow is completed, you will observe that glue job and crawlers have been successfully executed.

#### Congratulations!! You have successfully completed this lab