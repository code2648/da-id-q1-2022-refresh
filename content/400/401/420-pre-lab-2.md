---
title: "Student PreLab (not required in AWS event)"
weight: 420
---

::alert[The PreLab is only required if you are running the DMS lab outside of an AWS Event using Event Engine (EE), or in your own AWS Account.]{type="info"}

### Steps

1. Introduction
2. Create the Student Environment

### Introduction

This guide helps students set up the pre-environment for the [AWS Database Migration Service (AWS DMS) Main Lab](/400/401/430-main-lab.html). It is a compulsory step for the DMS Main Lab, but not required for the [AutoComplate DMS Lab](/400/440-auto-complete-lab.html).

AWS DMS required source and destination as shown below:

![](/static/400/images/19.png)

Your instructor will provide you source database details during main lab to configure source endpoint. If you ran instructor lab to setup your own instance of Postgres database then use instance endpoint from instructor lab.

In this lab, you will complete the following pre-requisite using AWS CloudFormation template deployment:

1. Create required VPC setup for AWS DMS instance.
2. Create Amazon S3 bucket for destination end point configuration.
3. Create Amazon S3 buckets for Amazon Athena query result storage.
4. Create required Amazon S3 bucket policy to put data by AWS DMS service.
5. Create AWS Glue Service Role to use in later hands-on workshop.
6. Create Amazon Athena workgroup users to use in Athena workshop.
7. Create Amazon Lake formation users to use in Lake formation workshop.

Labs are also available in GitHub - https://github.com/aws-samples/data-engineering-for-aws-immersion-day

### Create the Student Environment

::alert[Make sure you use the same region as where you deployed your instructor CloudFormation stack]{type="info"}

1. Sign in to the AWS console where you will host the student environment.

2. Go to the [IAM console](https://console.aws.amazon.com/iam/home#/roles), search the role `dms-cloudwatch-logs-role` in the search box. Note whether the role is present or not. In this example screenshot the role doesn't exist.
   ![](/static/400/images/79.png)
3. Click the **Deploy to AWS** button below to stand up the environment setup.

:button[Deploy to AWS]{iconName="external" variant="primary" href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=dmslab-student&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/DMSlab_student_CFN.yaml" target="_blank"}

4. As the role doesn't exists we choose **false** as the value for Parameter. Check the box "I acknowledge that ...", then click on "Create Stack" to create the stack

![](/static/400/images/student-stack.png)

In case you aren't able to launch the quick create stack, you can download the [template file](https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/DMSlab_student_CFN.yaml) and then follow the steps to [create stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) manually.

You have successfully completed the student prelab and setup all pre-requisite required to run the rest of the workshop.
Please proceed to the next lab [Data ingestion with DMS](/static/400/430-main-lab.html).
