---
title: DataBrew Pre-Lab Setup
weight: 1310
---

## DataBrew - Pre-Lab Setup

:::alert{type="warning"}
If you are part of an AWS event, you don't need to run this lab. Please check the necessary details in your Event Engine Team Dashboard under **5. Glue DataBrew** section
:::

### Steps

- Introduction
- CloudFormation Stack Deployment

### Introduction

In this lab, we will be using AWS Glue DataBrew to explore a dataset in S3, and to clean and prepare the data.

To do this, we will first set up an IAM role to use in DataBrew, and an S3 bucket for the results from the DataBrew jobs.

## CloudFormation Stack Deployment

1. Click the **Deploy to AWS** button below to create the AWS resources for the DataBrew lab.

:button[Deploy to AWS]{iconName="external" variant="primary" href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=databrew-lab&templateURL=https://s3.us-east-1.amazonaws.com/aws-dataengineering-day.workshop.aws/DataBrew_PreLab_CFN.yaml" target="_blank"}

2. Check the box "I acknowledge that ...", then click on "Create Stack" to create the stack  
![](/static/1300/images/databrew-stack.png)

> In case you aren't able to launch the quick create stack, you can download the [template file](https://s3.us-east-1.amazonaws.com/aws-dataengineering-day.workshop.aws/DataBrew_PreLab_CFN.yaml) and then follow the steps to [create stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) manually.

3. Once your stack is deployed, click the **Outputs** tab to view more information  
![](/static/1300/images/cfn6.png)

Note the values for **DatasetS3Path**, **DataBrewLabRole** and **DataBrewOutputS3Bucket** which will be used in the lab.

**Congratulations! You are all done with the CloudFormation deployment.**