---
title: "Lab: Real-Time Clickstream Anomaly Detection Kinesis Analytics"
weight: 320
---

## Introduction

This guide helps you complete Real-Time Clickstream Anomaly Detection using Amazon Kinesis Data Analytics.

Analyzing web log traffic to gain insights that drive business decisions has historically been performed using batch processing. Although effective, this approach results in delayed responses to emerging trends and user activities. There are solutions that process data in real time using streaming and micro-batching technologies, but they can be complex to set up and maintain. Amazon Kinesis Data Analytics is a managed service that makes it easy to identify and respond to changes in data behavior in real-time.

### Steps:

- Set up an Analytics Pipeline Application
- Connect Lambda as destination to Analytics Pipeline
- Environment Cleanup
- Appendix: Anomaly Detection Scripts

In the [Kinesis prelab setup](310-pre-lab.html), you fulfilled the prerequisites for this lab. In this lab, you will create the following data pipeline.
![](/static/300/images/16.png)

### Set up an Analytics Pipeline Application

::alert[Make sure you are in the appropriate AWS region.]{type="info"}

1. Navigate to the [Amazon Kinesis Analytics applications](https://console.aws.amazon.com/kinesisanalytics/home#/list/sql-applications-legacy) "SQL applications (legacy)" console.
2. Click "Create SQL application (legacy)" to create an application.
3. On the Create legacy SQL application page, fill the fields as follows:  
    a. For Application name, enter `anomaly-detection-application`.  
    b. For Description, type a description for your application.
    ![](/static/300/images/18.png)

4. Click **Create legacy SQL application**

5. On the application page, click **Configure** under Source tab.
   ![](/static/300/images/19.png)
6. Under Source configuration, and make the following selections:  
    a. For Source, choose Kinesis Firehose delivery stream.  
    b. For Kinesis Firehose delivery stream, choose **{stack-name}-FirehoseDeliveryStream-{some-random-string}** from the "Kinesis Firehose delivery stream" dropdown
    c. For Record preprocessing with AWS Lambda, leave it as **Off**.
    d. For IAM role for reading source stream, select **Choose from IAM roles that Kinesis Analytics can assume.**. Under the Service role dropdown, choose the role **{stack-name}-CSEKinesisAnalyticsRole-{random string}**.
    ![](/static/300/images/20.png)

::alert[Do not click “Discover schema” yet.]{type="warning"}

> You have set up the Kinesis Data Analytics application to receive data from a Kinesis Data Firehose and to use an IAM role from the pre-lab. However, you need to start sending some data to the Kinesis Data Firehose before you click Discover schema in your application.
> 
> Navigate to the Amazon Kinesis Data Generator (Amazon KDG) which you setup in prelab and start sending the Schema Discovery Payload at 1 record per second by click on Send data button. Make sure to select the appropriate AWS region.
> ![](/static/300/images/kdg.gif)
> Now that your Kinesis Data Firehose is receiving data, you can continue configuring the Kinesis Data Analytics Application.

7.  Go back to the Kinesis console, Now click **Discover Schema**. (Make sure your KDG is sending data to your Kinesis Data Firehose. If KDG was idle for more than 10-15 minutes or if the schema discovery fails, then refresh the KDG page and sign-in again.)  
  ![](/static/300/images/24.png)

8.   Click **Save changes**. Your Kinesis Data Analytics Application is created with an input stream.  
    
  ![](/static/300/images/25.png)
  Now, you can add some SQL queries to easily analyze the data that is being fed into the stream. 

9.  In the Real-time analytics section, click **Configure** to configure the SQL code.

  ![](/static/300/images/26.png)

10. Copy the contents of the file named :link[anomaly_detection.sql]{href="/static/300/scripts/Kinesis_Anlaytics_anomaly_detection.sql" action=download} from your lab package and paste it into the SQL editor. (You can also find code in Appendix)

  ![](/static/300/images/28.png)

11. Click **Save and run application**. The analytics application starts and runs your SQL query. (You can also find the SQL query in Appendix.)

> To learn more about the SQL logic, see the Analytics application section in the blog post titled [Real-time Clickstream Anomaly Detection with Amazon Kinesis Analytics](https://aws.amazon.com/blogs/big-data/real-time-clickstream-anomaly-detection-with-amazon-kinesis-analytics/).

12.  Once the application has started you can find Output & Input sections below the SQL Editor. On the Input tab, observe the input stream data named **SOURCE_SQL_STREAM_001**.
  ![](/static/300/images/30.png)
  If you choose the Output tab, you will notice multiple in-application streams You will populate data in these streams later in the lab.
  ![](/static/300/images/31.png)

## Connect Lambda as destination to Data Analytics Pipeline

Now that the logic to detect anomalies is in the Kinesis Data Analytics, you must connect it to a destination (AWS Lambda function) to notify you when there is an anomaly.

1. In the Output Streams section, choose **Connect to destination**.
![](/static/300/images/32.1.png)

2. For Destination configuration, choose the following.
  a. For destination, choose AWS Lambda function.  
  b. Under Lambda Function, browse and choose **CSEBeconAnomalyResponse** & for version, choose **$LATEST**.  
  c. In Access permissions for writing output stream, choose "Choose from IAM roles that Kinesis Data Analytics can assume" and select IAM role similar to **{stack-name}-CSEKinesisAnalyticsRole-{random-string}** from the dropdown.  

   ![](/static/300/images/32.png)

3. In the In-application stream section, make the following selections:  
    a. Select Choose an existing in-application stream.  
    b. For In-application stream name, choose **DESTINATION_SQL_STREAM**  
    c. For Output format, choose: **JSON** and click **Save changes**.  

    ![](/static/300/images/32_2.png)

This configuration allows your Kinesis Data Analytics Application to invoke your anomaly Lambda function and notify you when any anomalies are detected.

Now that all of the components are in place, you can test your analytics application.
For this part of the lab, you will need to use your Kinesis Data Generator in five separate browser windows. There will be one window sending normal impression payload, one window sending normal click payload, and three windows sending extra click payload.

1. Open your KDG in *five separate browser windows* and sign in as the same user.

::alert[Make sure to select the appropriate AWS region.]{type="warning"}

2. In one of your browser windows, start sending the Impression payload at a rate of 1 record per second (keep this running).

3. On another browser window, start sending the Click payload at a rate of 1 record per second (keep this running).

4. On your last three browser windows, start sending the Click payload at a rate of 1 record per second for a period of 20 seconds.
   \*\*If you did not receive an anomaly email, open another KDG window and send additional concurrent Click payloads. Make sure to not allow these functions to run for more than 10 to 20 seconds at a time. This could cause AWS Lambda to send you multiple emails due to the number of anomalies you are creating.

You can monitor anomalies on the **Real-time analytics tab** in the **DESTINATION_SQL_STREAM** table. If an anomaly is detected, it displays in that table.

![](/static/300/images/34.png)

::alert[Make sure to click other streams and review the data.]{type="info"}

Once an anomaly has been detected in your application and you will receive an email and text message to the specified accounts.

Email Snapshot:
![](/static/300/images/35.png)

SMS Snapshot:
![](/static/300/images/36.png)

After you have completed the lab, click **Actions → Stop Application** to stop your application and avoid flood of SMS and e-mails messages.
![](/static/300/images/37.png)

### Appendix

Anomaly Detection Scripts: 

::::expand{header="Content of the script anomaly_detection.sql"}
:::code{language=SQL showLineNumbers=false showCopyAction=true}
CREATE OR REPLACE STREAM "CLICKSTREAM" (
   "CLICKCOUNT" DOUBLE
);

CREATE OR REPLACE PUMP "CLICKPUMP" AS
INSERT INTO "CLICKSTREAM" ("CLICKCOUNT")
SELECT STREAM COUNT(*)
FROM "SOURCE_SQL_STREAM_001"
WHERE "browseraction" = 'Click'
GROUP BY FLOOR(
  ("SOURCE_SQL_STREAM_001".ROWTIME - TIMESTAMP '1970-01-01 00:00:00')
    SECOND / 10 TO SECOND
);

CREATE OR REPLACE STREAM "IMPRESSIONSTREAM" (
   "IMPRESSIONCOUNT" DOUBLE
);

CREATE OR REPLACE PUMP "IMPRESSIONPUMP" AS
INSERT INTO "IMPRESSIONSTREAM" ("IMPRESSIONCOUNT")
SELECT STREAM COUNT(*)
FROM "SOURCE_SQL_STREAM_001"
WHERE "browseraction" = 'Impression'
GROUP BY FLOOR(
  ("SOURCE_SQL_STREAM_001".ROWTIME - TIMESTAMP '1970-01-01 00:00:00')
    SECOND / 10 TO SECOND
);

CREATE OR REPLACE STREAM "CTRSTREAM" (
  "CTR" DOUBLE
);


CREATE OR REPLACE PUMP "CTRPUMP" AS
INSERT INTO "CTRSTREAM" ("CTR")
SELECT STREAM "CLICKCOUNT" / "IMPRESSIONCOUNT" * 100.000 as "CTR"
FROM "IMPRESSIONSTREAM",
  "CLICKSTREAM"
WHERE "IMPRESSIONSTREAM".ROWTIME = "CLICKSTREAM".ROWTIME;


CREATE OR REPLACE STREAM "DESTINATION_SQL_STREAM" (
    "CTRPERCENT" DOUBLE,
    "ANOMALY_SCORE" DOUBLE
);

CREATE OR REPLACE PUMP "OUTPUT_PUMP" AS
INSERT INTO "DESTINATION_SQL_STREAM"
SELECT STREAM * FROM
TABLE (RANDOM_CUT_FOREST(
             CURSOR(SELECT STREAM "CTR" FROM "CTRSTREAM"), --inputStream
             100, --numberOfTrees (default)
             12, --subSampleSize
             100000, --timeDecay (default)
             1) --shingleSize (default)
)
WHERE ANOMALY_SCORE > 2;
:::
::::
