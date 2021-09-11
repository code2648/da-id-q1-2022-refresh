---
title: "Athena and SageMaker (Optional)"
weight: 830
hidden: false
---

### Introduction

Amazon SageMaker is an end-to-end machine learning platform that lets you build, train, and deploy machine learning models in AWS. It is a highly modular service that lets you use each of these components independently of each other.

You will Learn:
* How to use the Jupyter notebook component of Sagemaker to integrate with the data lake using Athena
* Populate data frames for data manipulation.

![](/static/800/images/ml1.png)

This process to prepare the data to satisfy the needs of ML algorithms is iterative. To prepare the data, we will make the table definitions in Athena available in a Jupyter notebook instance of SageMaker.

Jupyter notebooks are popular among data scientists and used to visualize data, perform statistical analysis, complete data manipulations, and make the data ready for machine learning work.

#### Prerequisites

::alert[[Ingestion with DMS](/400) and [Transforming data with Glue ETL](/600) labs are prerequisites for this lab.]{type="warning"}

#### Create Amazon SageMaker Notebook Instance

1. Go to the [Amazon Sagemaker](https://console.aws.amazon.com/sagemaker/home) from AWS console
![](/static/800/images/ml2.png)
2. In the Amazon SageMaker navigation pane, click Notebook instances and Click Create notebook instance.
![](/static/800/images/ml3.png)
3. Put following values to create instance  
   * Give your choice of name for Notebook instance name e.g., `datalake-Sagemaker`
   * You can leave Notebook instance type as default value **ml.t2.medium** for this lab
   * Leave Elastic Inference as null. This is to add extra resources.
   * Choose a role for the notebook instances in Amazon SageMaker to interact with Amazon S3. As role doesn’t exist select the **Create new role** option.
   * In Create an IAM Role pop up window, choose the specific S3 bucket you will grant access to. For this lab, choose Any S3bucket as shown below and click Create Role:
   ![](/static/800/images/ml4.png)
   * You will see a Role got create, as you are going to Access Athena from SageMaker, so SageMaker execution role must have thenecessary permission to access Athena. To achieve that click the link for newly created IAM Role and in the new window
   ![](/static/800/images/ml5.png)
   * There is no Athena permission available in your SageMaker execution role. In this case, it is “AmazonSageMaker-ExecutionRole-20190531T143193”.
   ![](/static/800/images/ml6.png)
   Click Attach Policies button in Permissions tab.
   * Filter policies by “Athena”, check AmazonAthenaFullAccess managed policy and click Attach Policy button at the bottom of the screen. Close the new window/tab.
   ![](/static/800/images/ml7.png)
   After Athena policy to IAM Role, you can close this window and go back SageMaker browser window.
   * Leave all other options default. Click Create notebook instance.
   ![](/static/800/images/ml8.png)
4. Wait for the notebook instances to be created and the Status to change to Inservice and Click Open Jupyter in from “Actions” column.
![](/static/800/images/ml9.png)
5. The notebook interface opens in a separate browser window.
![](/static/800/images/ml10.png)

#### Connect the SageMaker Jupyter notebook to Athena

1. In the Jupyter notebook interface, click New. For the kernel, choose conda_python3
![](/static/800/images/ml11.png)
:::alert{type="info"}
Amazon SageMaker provides several kernels for Jupyter, including support for Python 2 and Python 3, MXNet, TensorFlow, and PySpark. This exercise uses Python because it includes the Pandas library.
:::
2. Within the notebook, execute the following commands to install the Athena JDBC driver and in the top toolbar, click Run. (PyAthena is a Python [DB API 2.0 (PEP 249)](https://www.python.org/dev/peps/pep-0249/) compliant client for the [Amazon Athena JDBC driver](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html).)
:::code{language=python showLineNumbers=false showCopyAction=true}
import sys
!{sys.executable} -m pip install PyAthena
:::

![](/static/800/images/ml12.png)
Observe success message in log:
![](/static/800/images/ml13.png)

:::alert{type="info"}
If an error occurs, you may be required to set a file system path for an application that helps build the driver software. If this is the case, expand **Troubleshoot error** below for the instructions.
:::

::::expand{header="Troubleshoot error"}
a. In your home notebook window, select New → Terminal.
![](/static/800/images/ml14.png)
b. Execute the following command in the Terminal.
:::code{language=bash showLineNumbers=false showCopyAction=true}
sudo ln -s /usr/libexec/gcc/x86_64-amazon-linux/4.8.5/cc1plus /usr/local/bin/
:::
![](/static/800/images/ml15.png)
c. Execute the original paragraph code from Step 2 again to build the **AthenaJDBC driver**.
::::

#### Working in Pandas
After the Athena driver is installed, you can use the JDBC connection to connect to Athena and populate the Pandas data frames. For data scientists, working with data is typically divided into multiple stages: ingesting and cleaning data, analyzing and modeling data, then organizing the results of the analysis into a form suitable for plotting or tabular display. Pandas is the ideal tool for all of these tasks.

1. You can load Athena table data from data lake to Pandas data frame and apply machine learning. Copy following code in your notebook cell and replace following:  
    - Your region: Run the following command in the notebook cell and get the current AWS region.  
    :::code{language=bash showLineNumbers=false showCopyAction=true}
    !aws configure get region
    :::
    ![](/static/800/images/ml16.png)
    - Your Athena Database name, this is the Glue Database name which you created during previous lab e.g., `ticketdata`. Update the S3 bucket location whose name contains string **dmslabs3bucket** followed by `athenaquery/`, so that it looks like **s3://xxx-dmslabs3bucket-xxx/athenaquery/**.

    :::code{language=python showLineNumbers=false showCopyAction=true}
    from pyathena import connect
    import pandas as pd
    conn = connect(s3_staging_dir='s3://xxx-dmslabs3bucket-xxx/athenaquery/',
    region_name='<yourregion>')

    df = pd.read_sql('SELECT * FROM "ticketdata"."nfl_stadium_data" order by stadium limit 10;', conn)
    df
    :::

2. Click Run and data frame will display query output.
   ![](/static/800/images/ml17.png)
   In this query, you are loading all nfl statdium information to panda dataframe from table nfl_stadium_data.

:::alert{type="info"}
If you get a SageMaker does not have Athena execution permissions error issue. You need to add Athena Access to the Sagemaker role as steps provide in previous section.
:::

3. You can use apply different ML algorithm in populated Pandas data frames. For example, draw a plot. Copy following code in your notebook and replace Your Athena Database name; this is the Glue Database name which you created during previous lab e.g., `ticketdata`.
   In this query, you are loading all even ticket information to panda dataframe from table sporting_event_ticket_info.

:::code{language=python showLineNumbers=false showCopyAction=true}
df = pd.read_sql('SELECT sport, \
       event_date_time, \
       home_team,away_team, \
       city, \
       count(*) as tickets, \
       sum(ticket_price) as total_tickets_amt, \
       avg(ticket_price) as avg_ticket_price, \
       max(ticket_price) as max_ticket_price, \
       min(ticket_price) as min_ticket_price  \
       FROM "<yourathenadatabase>"."sporting_event_ticket_info" \
       group by 1,2,3,4,5 \
       order by 1,2,3,4,5  limit 1000;', conn)
df
:::

4. Click Run and data frame will display query output
   ![](/static/800/images/ml18.png)
5. In new execution line copy following code
    :::code{language=python showLineNumbers=false showCopyAction=true}
    import matplotlib.pyplot as plt 
    df.plot(x='event_date_time',y='avg_ticket_price')
    :::
6. Click Run and you will see data plot which got draw using matplotlib library.
   ![](/static/800/images/ml19.png)

#### Cleanup
1. Go to [SageMaker Notebook Instances console](https://console.aws.amazon.com/sagemaker/home#/notebook-instances)
2. Select the notebook instance that you created e.g. `datalake-Sagemaker`
3. Choose **Actions → Stop**, it will take few minutes for the notebook instance to stop.
4. Once the notebook instance is stopped, choose **Actions → Delete** and choose **Delete** in the confirmation pop-up to delete the instance.