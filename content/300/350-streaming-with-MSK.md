---
title: Clickstream Analytics using MSK
weight: 350
---

In this workshop, our overall goal is to visualize and analyze the performance of various products in an e-commerce site by ingesting, transforming and analyzing real time clickstream data using AWS services for Apache Kafka (Amazon MSK) and Apache Flink (Kinesis Data Analytics for Java Applications) and Elasticsearch (Amazon Elasticsearch).
[Clickstream Analytics using MSK](https://amazonmsk-labs.workshop.aws/en/mskkdaflinklab/overview.html) to learn more.

The high level architecture will look like this:

![](/static/300/images-MSK/MSKFlinkArch.png)

First we will use a data producer to produce Clickstream messages to a topic (ExampleTopic) in an Amazon MSK cluster hosting Apache kafka. We will then configure and start a Kinesis Data Analytics for Java App using the Apache Flink engine which is a managed serverless Apache Flink service from AWS which would read events from the ExampleTopic topic in Amazon MSK, process and aggregate it and then send the aggegated events (analytics) to both topics in Amazon MSK and Amazon Elasticsearch. We will then consume those messages from Amazon MSK to illustrate how consumers can get real-time clickstream analytics. Also, we will create Kibana visualizations and a Kibana dashboard to visualize the real-time clickstream analytics.

We will spin up the resources necessary for the lab via a CloudFormation template. The template can be downloaded here.

This template will spin up a stack that looks like this:

![](/static/300/images-MSK/KafkaFlinkNetwork.png)

The stack creates:

- A VPC with 1 Public subnet and 3 Private subnets and the required plumbing.
- 1 Public instance that hosts a schema registry service, a producer and consumer.
- An Amazon Elasticsearch cluster.
- An Amazon KDA for Java application.
- An Amazon MSK cluster.

In the labs, you will ssh into an EC2 instance that the stack creates.

__KafkaClientEC2Instance__

[Clickstream Analytics using MSK](https://amazonmsk-labs.workshop.aws/en/mskkdaflinklab/overview.html) to follow complete lab.