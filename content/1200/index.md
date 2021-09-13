---
title: 'Lab: Data Lake Automation'
weight: 1200
---

### Introduction

This lab will give you an understanding of the AWS Lake Formation - a service that makes it easy to set up a secure data lake in days, as well as Athena for querying the data you import into your data lake.
![intro](/static/1200/images/1.png)

#### Prerequisites:

::alert[[Ingestion with DMS](/400) and [Transforming data with Glue ETL](/600) labs are prerequisites for this lab.]{type="warning"}

* Make sure you have the Postgres source database information from your Event Dashboard handy. If you are running the lab outside of AWS hosted event, please find the **DMSInstanceEndpoint** parameter value from **dmslab-instructor** <span style="color:blue">CloudFormation</span> **Outputs** tab.

#### Tasks Completed in this Lab:

In this lab you will be completing the following tasks:

1. Create a JDBC connection to RDS in AWS Glue
2. Lake Formation IAM Role
   ::alert[If your account is already setup with Glue Data catalog permissions then upgrade to Lake Formation Permission set by following [this documentation](https://docs.aws.amazon.com/lake-formation/latest/dg/upgrade-glue-lake-formation.html)]{type="info"}
3. Lake Formation - Add Administrator and start workflows using Blueprints
4. Explore the components of a Glue WorkFlow created by lake formation
5. Explore workflow results in Athena
6. Grant fine grain access controls to Data Lake user
7. Verify data permissions using Athena

### Check demo video below
::video{id=-SvLSuhbS1w }
