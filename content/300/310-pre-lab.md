---
title: Streaming Data Analytics Prelab setup
weight: 310
---

## Streaming Data Analytics - Prelab setup

### Steps

- Introduction
- CloudFormation Stack Deployment
- Set up the Amazon Kinesis Data Generator
- Set up Email and SMS Subscription
- Observe AWS Lambda Anomaly function
- Appendix: CloudFormation Template

### Introduction

This guide will help you set up the pre-lab environment for the Real-Time Clickstream Anomaly Detection Amazon Kinesis Data Analytics lab.

The AWS CloudFormation template `Kinesis_PreLab.yaml` (optionally copy YAML code from end of this page) included with this lab deploys the following architecture without the highlighted components. You will set up the highlighted components manually.

![](/static/300/images/1.png)

After you deploy the CloudFormation template, sign into your account to view the following resources:

- Two Amazon Simple Storage Service (Amazon S3) buckets: You will use these buckets to persist raw and processed data.
- One AWS Lambda function: This Lambda function will be triggered once an anomaly has been detected.
- Amazon Simple Notification Service (Amazon SNS) topic with an email and phone number subscribed to it: The Lambda function will publish to this topic once an anomaly has been detected.
- Amazon Cognito User credentials: You will use these user credentials to log into the Kinesis Data Generator to send records to our Amazon Kinesis Data Firehose.

## CloudFormation Stack Deployment

::alert[If you are in AWS Event, your instructor would let you know which AWS region to use.]{type="info"}
::alert[If you are running it on your own account, you select an AWS region [e.g. US-EAST-1 (N. Virginia)] and stick to it for whole of the workshop.]{type="info"}

1. Click the **Deploy to AWS** button below to stand up the Kinesis pre-lab workshop infrastructure.
   
   :button[Deploy to AWS]{iconName="external" variant="primary" href="https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=kinesis-pre-lab&templateURL=https://s3.amazonaws.com/aws-dataengineering-day.workshop.aws/Kinesis_PreLab.yaml" target="_blank"}

The button above will open a Quick create stack form:
![](/static/300/images/2.1.png)

2. In the Parameters section, fill the following fields as shown in screenshot:

- **Username:** This is your username to login to the Kinesis Data Generator
- **Password:** This is your password for the Kinesis Data Generator. The password must be at least 6 alpha-numeric characters and contain at least one number and a capital letter.
- **Email:** Type an email address that you can access. The SNS topic sends a confirmation to this address.
- **SMS:** Type a phone number (+1XXXXXXXXX) where you can receive texts from the SNS topic.
- In the Capabilities section, select the check box marked "I acknowledge that AWS CloudFormation might create IAM resources".

3. Click Create. CloudFormation redirects you to your existing stacks. After a few minutes, the kinesis-pre-lab displays a CREATE_COMPLETE status.
   ![](/static/300/images/7.1.png)

4. While stack runs, watch out for email like:

![](/static/300/images/email.png)

5. Confirm the subscription
   ![](/static/300/images/subscription.png)

6. Once your stack is deployed, click the Outputs tab to view more information:

- **KinesisDataGeneratorUrl** - This value is the Kinesis Data Generator (KDG) URL.
- **RawBucketName** - Store raw data coming from KDG.
- **ProcessedBucketName** - Store transformed data
  ![](/static/300/images/8.1.png)

**Congratulations! You are all done with the CloudFormation deployment.**

## Set up the Amazon Kinesis Data Generator

On the Outputs tab, notice the Kinesis Data Generator URL. Navigate to this URL to login into the Amazon Kinesis Data Generator (Amazon KDG).

The KDG simplifies the task of generating data and sending it to Amazon Kinesis. The tool provides a user-friendly UI that runs directly in your browser. With the KDG, you can do the following tasks:

- Create templates that represent records for your specific use cases
- Populate the templates with fixed data or random data
- Save the templates for future use
- Continuously send thousands of records per second to your Amazon Kinesis stream or Firehose delivery stream

Let’s test your Cognito user in the Kinesis Data Generator.

1. On the Outputs tab, click the KinesisDataGeneratorUrl.
   ![](/static/300/images/8.2.png)
2. Sign in using the username and password you entered in the CloudFormation console.
   ![](/static/300/images/10.png)
3. After you sign in, you should see the KDG console. You need to set up some templates to mimic the clickstream web payload.

::alert[Note that your Kinesis Data Firehose has been deployed in the region where you deployed the CloudFormation stack.]{type="info"}

Create the following templates but don’t click on Send Data yet , we will do that during main lab. Copy the tab name highlight in bold letter and value as json string, refer screenshot:

::code[Schema Discovery Payload]{showCopyAction=true}

:::code{language=json showLineNumbers=false showCopyAction=true}
{"browseraction":"DiscoveryKinesisTest", "site": "yourwebsiteurl.domain.com"}
:::

::code[Click Payload]{showCopyAction=true}

:::code{language=json showLineNumbers=false showCopyAction=true}
{"browseraction":"Click", "site": "yourwebsiteurl.domain.com"}
:::

::code[Impression Payload]{showCopyAction=true}

:::code{language=json showLineNumbers=false showCopyAction=true}
{"browseraction":"Impression", "site": "yourwebsiteurl.domain.com"}
:::

Your Amazon Kinesis Data Generator console should look similar to this example.
![](/static/300/images/11.png)

### Verify Email and SMS Subscription

1. On the Amazon [SNS](https://console.aws.amazon.com/sns/v3/home#/topics) navigation menu, select **Topics**. An SNS topic named **ClickStreamEvent** appears in the display.:
   ![](/static/300/images/12.png)
2. Click the topic. The Topic details screen appears listing the e-mail/SMS subscription as pending or confirmed.
   ![](/static/300/images/13.png)

::alert[Select corresponding subscription endpoint and Click **Request confirmations** to confirm your subscription for e-mail/SMS. Make sure to check your email junk folder for the request confirmation link.]{type="info"}

### Observe AWS Lambda Anomaly function:

CloudFormation template already deployed this Lambda function. You
Just need to spend few minutes to observe code and understand action behind the seen trigger by lambda

1. In the console, navigate to [AWS Lambda](https://console.aws.amazon.com/lambda/home#/functions).
2. In the AWS Lambda navigation pane, select Functions **CSEBeconAnomalyResponse**.
   ![](/static/300/images/14.png)
3. Click the function to scroll down to code section.
   ![](/static/300/images/15.png)
4. Go through the code in the Lambda code editor. Notice **TopicArn** value your recorded in Email/SMS subscription step. Lambda will send message to this topic and notify.

At this point you have all the necessary components to work on the lab.

**This is the end of the pre-lab, congratulations!**

## Appendix

Pre-Lab CloudFormation Code for your reference

::::expand{header="Content of the CloudFormation template Kinesis_PreLab.yaml"}
:::code{language=yaml showLineNumbers=false showCopyAction=true}
---
AWSTemplateFormatVersion: 2010-09-09
Description: Supporting elements for the Kinesis Analytics click stream lab
Parameters:
  email:
    Description: Email address to send anomaly detection events.
    Type: String
  SMS:
    Description: Mobile Phone number to send SMS anomaly detection events. +1XXXXXXXXXX
    Type: String
  Username:
    Description: The username of the user you want to create in Amazon Cognito.
    Type: String
    AllowedPattern: "^(?=\\s*\\S).*$"
    ConstraintDescription: Cannot be empty
  Password:
    Description: The password of the user you want to create in Amazon Cognito. Must be at least 6 alpha-numeric characters, and contain at least one number
    Type: String
    NoEcho: true
    AllowedPattern: "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{6,}$"
    ConstraintDescription: Must be at least 6 alpha-numeric characters, and contain at least one number
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      -
        Label:
          default: "Kinesis Pre Lab set up"
        Parameters:
          - "Username"
          - "Password"
          - "email"
          - "SMS"
Resources:
  RawS3Bucket:
    Type: AWS::S3::Bucket
  ProcessedS3Bucket:
    Type: AWS::S3::Bucket
  FirehoseDeliveryStream:
    Type: AWS::KinesisFirehose::DeliveryStream
    Properties:
      S3DestinationConfiguration:
        BucketARN: !Sub arn:aws:s3:::${RawS3Bucket}
        BufferingHints:
          IntervalInSeconds: 60
          SizeInMBs: 50
        CompressionFormat: GZIP
        Prefix: weblogs/raw/
        RoleARN: !GetAtt S3Role.Arn
  S3Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Service: firehose.amazonaws.com
            Action: sts:AssumeRole
            Condition:
              StringEquals:
                sts:ExternalId: !Ref AWS::AccountId
      Policies:
        -
          PolicyName: S3Access
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Effect: Allow
                Action:
                  - "s3:*"
                Resource:
                  - !Sub arn:aws:s3:::${RawS3Bucket}
                  - !Sub arn:aws:s3:::${RawS3Bucket}/*
        -
          PolicyName: CloudWatch
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Effect: Allow
                Action:
                  - "cloudwatch:*"
                  - "cloudwatchlogs:*"
                Resource:
                  - "*"
  DataGenCognitoSetupLambdaFunc:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        S3Bucket: !Sub aws-kdg-tools-${AWS::Region}
        S3Key: datagen-cognito-setup.zip
      Description: Creates a Cognito User Pool, Identity Pool, and a User.  Returns IDs to be used in the Kinesis Data Generator.
      FunctionName: KinesisDataGeneratorCognitoSetup
      Handler: createCognitoPool.createPoolAndUser
      Role: !GetAtt CognitoLambdaExecutionRole.Arn
      Runtime: nodejs12.x
      Timeout: 60
  CognitoLambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        -
          PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Effect: Allow
                Action:
                  - logs:*
                Resource: !Sub arn:aws:logs:${AWS::Region}:${AWS::AccountId}:*
              -
                Effect: Allow
                Action:
                  - cognito-idp:AdminConfirmSignUp
                  - cognito-idp:CreateUserPoolClient
                  - cognito-idp:AdminCreateUser
                Resource:
                  - !Sub arn:aws:cognito-idp:${AWS::Region}:${AWS::AccountId}:userpool/*
              -
                Effect: Allow
                Action:
                  - cognito-idp:CreateUserPool
                  - cognito-identity:*
                Resource: "*"
              -
                Effect: Allow
                Action:
                  - iam:UpdateAssumeRolePolicy
                Resource:
                  - !GetAtt AuthenticatedUserRole.Arn
                  - !GetAtt UnauthenticatedUserRole.Arn
              -
                Effect: Allow
                Action:
                  - iam:PassRole
                Resource:
                  - !GetAtt AuthenticatedUserRole.Arn
                  - !GetAtt UnauthenticatedUserRole.Arn
  SetupCognitoCustom:
    Type: Custom::DataGenCognitoSetupLambdaFunc
    Properties:
      ServiceToken: !GetAtt DataGenCognitoSetupLambdaFunc.Arn
      Region: !Ref AWS::Region
      Username: !Ref Username
      Password: !Ref Password
      AuthRoleName: !Ref AuthenticatedUserRole
      AuthRoleArn: !GetAtt AuthenticatedUserRole.Arn
      UnauthRoleName: !Ref UnauthenticatedUserRole
      UnauthRoleArn: !GetAtt UnauthenticatedUserRole.Arn
  AuthenticatedUserRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Federated:
                - cognito-identity.amazonaws.com
            Action:
              - sts:AssumeRoleWithWebIdentity
      Path: /
      Policies:
        -
          PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Action:
                  - kinesis:DescribeStream
                  - kinesis:PutRecord
                  - kinesis:PutRecords
                Resource:
                  - !Sub arn:aws:kinesis:${AWS::Region}:${AWS::AccountId}:stream/*
                Effect: Allow
              -
                Action:
                  - firehose:DescribeDeliveryStream
                  - firehose:PutRecord
                  - firehose:PutRecordBatch
                Resource:
                  - !Sub arn:aws:firehose:${AWS::Region}:${AWS::AccountId}:deliverystream/*
                Effect: Allow
              -
                Action:
                  - mobileanalytics:PutEvents
                  - cognito-sync:*
                  - cognito-identity:*
                  - ec2:DescribeRegions
                  - firehose:ListDeliveryStreams
                  - kinesis:ListStreams
                Resource:
                  - "*"
                Effect: Allow
  UnauthenticatedUserRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Federated:
                - cognito-identity.amazonaws.com
            Action:
              - sts:AssumeRoleWithWebIdentity
      Path: /
      Policies:
        -
          PolicyName: root
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Effect: Allow
                Action:
                  - mobileanalytics:PutEvents
                  - cognito-sync:*
                Resource:
                  - "*"
  CSELambdaSNSPublishRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      Policies:
        -
          PolicyName: lambda_sns
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Effect: Allow
                Action:
                  - sns:Publish
                Resource: !Ref CSEClickStreamEvent
  CSEClickStreamEvent:
    Type: AWS::SNS::Topic
    Properties:
      DisplayName: ClkStrEv2
      Subscription:
        -
          Endpoint: !Ref email
          Protocol: email
        -
          Endpoint: !Ref SMS
          Protocol: sms
      TopicName: ClickStreamEvent2
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      Policies:
        -
          PolicyName: CSELambdaExecutionRole
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Sid: publish2sns
                Effect: Allow
                Action:
                  - sns:Publish
                Resource:
                  - !Ref CSEClickStreamEvent
              -
                Sid: writelogs
                Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: !Sub arn:aws:logs:${AWS::Region}:${AWS::AccountId}:*
              -
                Sid: readkinesis
                Effect: Allow
                Action:
                  - kinesis:DescribeStream
                  - kinesis:GetRecords
                  - kinesis:GetShardIterator
                  - kinesis:ListStreams
                Resource: "*"
              -
                Action:
                  - s3:PutObject
                Resource: !Sub arn:aws:s3:::${ProcessedS3Bucket}/*
                Effect: Allow
                Sid: writeToS3
  CSEBeconAnomalyResponse:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        ZipFile: !Sub |
          var AWS = require('aws-sdk');
          var sns = new AWS.SNS( { region: '${AWS::Region}' });
          var s3 = new AWS.S3();

          exports.handler = function(event, context) {
            console.log(JSON.stringify(event, null, 3));
            event.records.forEach(function(record) {
              var payload = new Buffer(record.data, 'base64').toString('ascii');
              var rec = payload.split(',');
              var ctr = rec[0];
              var anomaly_score = rec[1];
              var detail = 'Anomaly detected with a click through rate of ' + ctr + '% and an anomaly score of ' + anomaly_score;
              var subject = 'Anomaly Detected';
              var SNSparams = {
                Message: detail,
                MessageStructure: 'String',
                Subject: subject,
                TopicArn: '${CSEClickStreamEvent}'
              };
              sns.publish(SNSparams, function(err, data) {
                if (err) context.fail(err.stack);
                else{
                  var anomaly = [{
                    'date': Date.now(),
                    'ctr': ctr,
                    'anomaly_score': anomaly_score
                  }]

                  convertArrayOfObjectsToCSV({ data: anomaly }, function(err1, data1){
                    if(err1) context.fail(err1.stack); // an error occurred
                    else{
                      var today = new Date();
                      var S3params = {
                        'Bucket': '${ProcessedS3Bucket}',
                        'Key': `weblogs/processed/${!today.getFullYear()}/${!today.getMonth()}/${!today.getDate()}/${!record.recordId}.csv`,
                        'Body': data1,
                        'ContentType': 'text/csv'
                      }
                      s3.putObject(S3params, function(err2, data2) {
                        if (err2) context.fail(err2.stack); // an error occurred
                        else {
                          context.succeed('Published Notification'); // successful response
                        }
                      })
                    }
                  })
                }
              });
            });
          };

          function convertArrayOfObjectsToCSV(args, callback) {
            var result, ctr, keys, columnDelimiter, lineDelimiter, data;
            data = args.data || null;
            if (data == null || !data.length) {
              callback(new Error('data is null'));
            }
            columnDelimiter = args.columnDelimiter || ',';
            lineDelimiter = args.lineDelimiter || ',';
            keys = Object.keys(data[0]);
            result = '';
            result += keys.join(columnDelimiter);
            result += lineDelimiter;
            data.forEach(function(item) {
              ctr = 0;
              keys.forEach(function(key) {
                  if (ctr > 0) result += columnDelimiter;
                  result += item[key];
                      ctr++;
                  });
                  result += lineDelimiter;
            });
            callback(null, result);
          }
      Description: Click Stream Example Lambda Function
      FunctionName: CSEBeconAnomalyResponse
      Handler: index.handler
      MemorySize: 128
      Role: !GetAtt LambdaExecutionRole.Arn
      Runtime: nodejs12.x
      Timeout: 5
  CSEKinesisAnalyticsRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Service:
                - kinesisanalytics.amazonaws.com
            Action:
              - sts:AssumeRole
      Policies:
        -
          PolicyName: firehose
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              -
                Sid: ReadInputFirehose
                Effect: Allow
                Action:
                  - firehose:DescribeDeliveryStream
                  - firehose:Get*
                Resource:
                  - !GetAtt FirehoseDeliveryStream.Arn
              -
                Sid: UseLambdaFunction
                Effect: Allow
                Action:
                  - lambda:InvokeFunction
                  - lambda:GetFunctionConfiguration
                Resource:
                  - !Sub ${CSEBeconAnomalyResponse.Arn}:$LATEST
  LambdaRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          -
            Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        - PolicyName: LambdaRolePolicy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action: s3:DeleteObject
                Resource:
                  - !Sub arn:aws:s3:::${RawS3Bucket}/*
                  - !Sub arn:aws:s3:::${ProcessedS3Bucket}/*
              - Effect: Allow
                Action: s3:ListBucket
                Resource:
                  - !Sub arn:aws:s3:::${RawS3Bucket}
                  - !Sub arn:aws:s3:::${ProcessedS3Bucket}
              - Effect: Allow
                Action: logs:CreateLogGroup
                Resource: !Sub arn:aws:logs:${AWS::Region}:${AWS::AccountId}:*
              - Effect: Allow
                Action:
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: !Sub arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:*
  S3BucketHandler:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      Role: !GetAtt LambdaRole.Arn
      Code:
        ZipFile: |
          import os
          import json
          import cfnresponse
          import boto3
          from botocore.exceptions import ClientError

          s3 = boto3.resource('s3')


          def handler(event, context):
              print("Received event: %s" % json.dumps(event))
              s3_bucket = s3.Bucket(event['ResourceProperties']['Bucket'])

              try:
                  if event['RequestType'] == 'Create' or event['RequestType'] == 'Update':
                      result = cfnresponse.SUCCESS
                  elif event['RequestType'] == 'Delete':
                      s3_bucket.objects.delete()
                      result = cfnresponse.SUCCESS
              except ClientError as e:
                  print('Error: %s', e)
                  result = cfnresponse.FAILED

              cfnresponse.send(event, context, result, {})

      Runtime: python3.8
      Timeout: 300

  EmptyRawS3Bucket:
    Type: "Custom::EmptyS3Bucket"
    Properties:
      ServiceToken: !GetAtt S3BucketHandler.Arn
      Bucket: !Ref RawS3Bucket

  EmptyProcessedS3Bucket:
    Type: "Custom::EmptyS3Bucket"
    Properties:
      ServiceToken: !GetAtt S3BucketHandler.Arn
      Bucket: !Ref ProcessedS3Bucket
Outputs:
  KinesisDataGeneratorUrl:
    Description: The URL for your Kinesis Data Generator.
    Value: !Sub https://awslabs.github.io/amazon-kinesis-data-generator/web/producer.html?${SetupCognitoCustom.Querystring}
  RawBucketName:
    Description: This the bucket name of where your Raw data will be store at
    Value: !Ref RawS3Bucket
  ProcessedBucketName:
    Description: This the bucket name of where your Processed data will be store at
    Value: !Ref ProcessedS3Bucket
:::
::::