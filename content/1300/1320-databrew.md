---
title: Data preparation with Glue DataBrew
weight: 1320
---

In this lab you will be completing the following tasks.

### Tasks Completed in this Lab:

- Create a Glue DataBrew project to explore a dataset
- Connect a sample dataset from S3
- Explore the dataset in Glue DataBrew  
- Generate a rich data profile for the dataset
- Create a recipe and job to clean and normalize data in the dataset

<!-- Labs are also available at https://aws-dataengineering-day.workshop.aws/ -->

### Creating a project

1. Navigate to the AWS Glue DataBrew service
   ![](/static/1300/images/1.png)

2. On the DataBrew console, select [Projects](https://console.aws.amazon.com/databrew/home#projects)
   ![](/static/1300/images/2.png)

3. Click **Create project**   
4. In the **Project details** section, enter `covid-states-daily` as the project name
   ![](/static/1300/images/3.png)
   
5. In the **Select a dataset** section, select **New dataset** and enter `covid-states-daily-stats`
   ![](/static/1300/images/4.png)
   
6. In the **Connect to a new dataset** section, select **Amazon S3** under "Data lake/data store"  

   Enter the **DatasetS3Path** that is available in Event Engine Team Dashboard or outputs section of your CloudFormation stack 
   ![](/static/1300/images/5.png)

7. In the **Sampling** section, leave the default configuration values
   ![](/static/1300/images/6.png)

8. In the **Permissions** section, select the role `databrew-lab-DataBrewLabRole-xxxxx` from the drop-down list  
   ![](/static/1300/images/7.png)

9. Click **Create project**   
   Glue DataBrew will create the project, this may take a few minutes.

   ![](/static/1300/images/8.png)

### Exploring the dataset

1. When the project has been created, you will be presented with the **Grid** view. This is the default view, where a sample of the data is shown in tabular format.  
![](/static/1300/images/40.PNG)
    The Grid view shows
    - Columns in the dataset 
    - Data type of each column
    - Summary of the range of values that have been found
    - Statistical distribution for numerical columns

2. Click on the **Schema** tab  
    The Schema view shows the schema that has been inferred from the dataset. In schema view, you can see statistics about the data values in each column.  

    In the Schema view, you can
    - Select the checkbox next to a column to view the summary of statistics for the column values
    - Show/Hide columns
    - Rename columns
    - Change the data type of columns
    - Rearrange the column order by dragging and dropping the columns

3. Click on the **Profile** tab

    In the Profile view, you can run a data profile job to examine and collect statistical summaries about the data. A data profile is an assessment in terms of structure, content, relationships, and derivation.

    Click on **Run data profile** 

    In the **job details** and **job run sample** panels, leave the default values.

    ![](/static/1300/images/12.png)

    In the **Job output settings** section, select the S3 bucket with the name `databrew-lab-databrewoutputs3bucket-xxxxx` and a folder name (eg. data-profile)

    ![](/static/1300/images/13.png)

    In the **Permissions** section, select the IAM role with the name `databrew-lab-DataBrewLabRole-xxxxx`

    Leave all other settings as the default values

    Click **Create and run job**
    
    The data profile job takes approximately 5 minutes complete. 

4.  Click on **Jobs** from the menu on the left hand side of the DataBrew console.

    Click on **Profile jobs** tab to view a list of profile jobs.

    You can see the status of your profile job on this screen.

    ![](/static/1300/images/14.png)

    When the profile job has successfully completed, click on **View data profile**

    ![](/static/1300/images/15.png)

    You can also access the data profile from the **Profile** tab in the project.

    The data profile shows a summary of the rows and columns in the dataset, how many columns and rows are valid, and correlations between columns.

5.  Click on the **Column statistics** tab to view a column-by-column breakdown of the data values.

    ![](/static/1300/images/16.png)

### Preparing the dataset

In this section, we will apply the following transformations to the dataset.
- Convert the `date` column to from integer to string
- Split the `date` column into three new columns (year, month, day) to partition the data by these columns
- Fill the missing values in the `probableCases` column with 0
- Map the values of the `dataQualityGrade` column to a numerical value

1. Navigate back to the `covid-states-daily` project grid view.

2. DataBrew has inferred data type of the `date` column as integer. We will convert the data type of the `date` column to string.

    Click on the `#` icon next to the `date` column name and select **string**

    ![](/static/1300/images/17.png)

    Note that the transformation is added to the recipe at the right.

    ![](/static/1300/images/18.png)

3. We will duplicate the `date` column first before splitting it into `year`, `month`, `day` columns, as the original column will be deleted by this transformation.

    Select the `...` at the top of the `date` column.

    From the pop-up menu, scroll to the bottom and select **Duplicate**

    ![](/static/1300/images/41.png)
    
    Leave the default settings in the **Duplicate column** dialog

    ![](/static/1300/images/19.png)

    Click **Apply**

    A copy of the `date` column is created with the name `date_copy`. Note that the **duplicate column** transformation is added as a step to the recipe at the right.

4. Let's split the `date_copy` column into `year`, `month`, `day` columns.

    Select the `...` at the top of the `date_copy` column.

    Select **Split column** / **At positions from beginning**

    ![](/static/1300/images/20.png)

    In the **Split column** dialog, enter `4` for **Position from the beginning** to split out the year. Leave all other default settings.

    ![](/static/1300/images/21.png)

    In the **Split column** dialog, scroll down and click **Preview changes** to see how the column is split. Note that the `date_copy` column is marked for deletion.

    ![](/static/1300/images/22.png)

    Click **Apply**

5. Next, split the `date_copy_2` column into month and day.

    The result should look like the screenshot below.

    ![](/static/1300/images/24.png)

6. Let's rename the new columns to `year`, `month`, `day`

    Click on the `date_copy_1` column and select **Rename** from the menu. Enter `year` as the new column name, and click **Apply**

    ![](/static/1300/images/25.png)

    Rename the other two new columns - `date_copy_2_1` and `date_copy_2_2` - to `month` and `day` respectively.

    The result should look like the following.

    ![](/static/1300/images/38.png)

7. The `probableCases` column has some missing values. We will set these missing values to 0.

    To navigate to the `probableCases` column, click on the **columns** drop-down list at the top, enter `probableCases` in the search field and click **View**.

    ![](/static/1300/images/42.png)
    
    Click on the `probableCases` column and select **Remove or fill missing values** / **Fill with custom value**

    ![](/static/1300/images/26.png)

    Enter `0` as the **Custom value** and click **Apply**

    ![](/static/1300/images/27.png)

8. Map the values of the `dataQualityGrade` column to numerical values.

    To navigate to the `dataQualityGrade` column, click on the **columns** drop-down list at the top, enter `dataQualityGrade` in the search field and click **View**.

    ![](/static/1300/images/43.png)
    
    Click on the `dataQualityGrade` column and select **Categorical mapping**

    ![](/static/1300/images/28.png)
    
    In the **Categorically map column** dialog
    - Select the option **Map all values**
    - Enable **Map values to numeric values**
    - Map the current `dataQualityGrade` value to the new value as follows
    
      |dataQualityGrade|New value|
      |---|---|
      |N/A|0|
      |A+|1|
      |A|2|
      |B|3|
      |C|4|
      |D|5|

    ![](/static/1300/images/29.png)

    Leave all other settings as default. Click **Apply**

    After this transform, the new column `dataQualityGrade_mapped` is of type double, convert this column to integer.

9. You are now ready to publish the recipe so that it can be used in DataBrew jobs. The final recipe looks like the following.

    ![](/static/1300/images/31.png)

    Click on the **Publish** button at the top of the recipe.
    
    Optionally enter a version description, and click **Publish**

    ![](/static/1300/images/32.png)

    The recipe is published as Version 1.0. DataBrew applies a version number when a recipe is published.

### Creating a DataBrew job

1. Click on **Jobs** from the menu on the left hand side of the DataBrew console.

    On the **Recipe jobs** tab, click on **Create job**

    Enter `covid-states-daily-prep` for the job name

    Select **Create a recipe job**

    Select the `covid-states-daily` dataset

    Select the 'covid-states-daily-recipe'

    ![](/static/1300/images/33.png)

    In the **Job output settings** section, enter the S3 location `s3://databrew-lab-databrewoutputs3bucket-xxxxx/job-outputs/`. 

    Expand the **Additional Configuration - optional** panel.
    
    Under **Custom partition by column values** add `year`, `month` and `day` columns. This will partition the data in the output folder by year, month and day. 

    ![](/static/1300/images/34.png)

    In the **Permissions** section, select the role `databrew-lab-DataBrewLabRole-xxxxx`

    ![](/static/1300/images/35.png)

    Click **Create and run job**

2. The DataBrew job is created and the job status is `Running`

    ![](/static/1300/images/36.png)

    Wait until the job has completed successfully (approx. 4 minutes)

    ![](/static/1300/images/37.png)

    Click on the link to the job output, and verify that the output files are partitioned in the S3 bucket

### Viewing data lineage

1. In DataBrew, navigate back to the `covid-states-daily` project
    
    Click on **Lineage** at the top right

    This view shows the origin of the data and the transformation steps that the data has been through.

    ![](/static/1300/images/30.png)

**Congratulations, you have completed the DataBrew lab. If you haven't already done so, you can return to step 13 to examine the data profile.**