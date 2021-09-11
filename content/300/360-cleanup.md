---
title: "Cleanup"
weight: 360
---

### Cleanup PreLab Environment

To save on cost, it is required to dispose your environment which you have created during this lab. 

1. In your AWS account, navigate to the CloudFormation console.  
2. On the CloudFormation console, select stack which you have created during pre-lab.  
3. Click on Action drop down and select delete stack as shown in below screenshot.  
![](/static/300/images/38.png)

4. As you created, Kinesis Analytics application manually, so need to delete it by selecting your analytics application . Click on Action drop down and select delete application  
![](/static/300/images/39.png)

5. Go the Cognito and delete the user pool that have been created.  

### Cleanup Streaming ETL Job

1. Navigate to [AWS Glue Console](https://console.aws.amazon.com/glue/home)  
2. On the AWS Glue menu, navigate to Jobs, select `TicketTransactionStreamingJob`, from the **Actions** drop down, select **Delete** and click Delete in the confirmation pop-up.
![](/static/300/images/cleanup-1.png)
3. On the AWS Glue menu, navigate to Crawler, select `TicketTransactionParquetDataCrawler`, from the **Actions** drop down, select **Delete crawler** and click Delete in the confirmation pop-up.
![](/static/300/images/cleanup-2.png)