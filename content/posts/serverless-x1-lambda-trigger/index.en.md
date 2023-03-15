---
title: "STDS x1 - Triggers of AWS Lambda"
date: 2023-02-10
draft: true
description: "STDS X1 - Lambda is powerful, triggers are the handler to the power"
summary: "Systematic Trading Done Serverless (STDS) x1 - Introduces all kinds of lambda triggers, which is at the heart of serverless event driven design"
images: []
tags: 
    - lambda
    - serverless
    - aws
    - stds
categories: 
    - Software Design
lightgallery: true
resources:
- name: featured-image
  src: header.png
- name: featured-image-preview
  src: header.png
---

`lambda` is a piece of code ( or a function ) that is ready to run. 
However who triggers them? how to pass parameters to them?
This is the topic of this post.

## Triggers

There are few ways to trigger lambda in aws:

- API Gateway / Lambda URL
- Time based scheduler ( Implemented by EventBridge )
- Event Source Mapping
  - Queue or Message Bus
    - SQS
    - SNS
    - EventBridge
    - Kinesis
    - Kafka ( self managed or aws managed )
  - Other services
    - DynamoDB
    - S3
    - and almost every other aws services can trigger lambda
- Step Functions

Those triggers are very rich, up on which we can make the whole aws infrastructure
becomes our event driven architecture. 

The event triggers are bridges from serverless components to server based component,
which makes the system so plugable.

## API Gateway and Lambda URL

API Gateway is probably the most straightforward way to invoke a lambda function.

![20230210203849](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230210203849.png "API Gateway and Lambda")

User or other processes can simple by calling the REST API to pass payload to the 
lambda function and expect a result. The calling can either be sync or async depending
on the actual lambda function implementation.

However, API Gateway is much more powerful ( for good and for bad ), not only we 
can send HTTP request, but also Websocket! In addition, it come with more stuff
such as API key, caching, CORS, throttling, etc.

If you just want build a internal REST API, API Gateway probably a bit too much.
The other option is Lambda URL, which is a new service released 2022.

With [Lambda URL](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html), we will get a IAM auth based http endpoint for the lambda function.
We cannot setup a customized domain for it, neither caching, but it is cheaper as well
than API Gateway.

## Time Scheduler

This is basically put a crontab job for lambdas. For example, you can setup a 
daily job to run every weekday at 17:00.

![20230210205109](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230210205109.png)

Two things to mention:

- scheduler is actually based on EventBridge, which we will cover later.
- there is no user defined information passed to lambda function. ( lambda function always receive some system status information, by convention they are encapsulated in `Event` and `Context` objects. )

## Event Source Mapping

### SQS

[SQS](https://quant.funcoder.net/posts/serverless-4-queue/) is a queue service.
We can setup lambda functions to be triggered by the message in the queue. 

Technically, lambda is not triggered by the message in the queue, whereas it is 
aws will let the lambda polls the queue for us, if it find any message, the lambda
will process the message.

![20230210225912](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230210225912.png)

SQS will send message in batch to lambda, once sent to lambda, that batch will be 
marked with invisible, if the lambda successfully processed the message, those messages
are deleted, otherwise, failed batch (**note that whole batch is back!**) will be put back to the queue. 
After a predefined retry reached, failed message will be put into a `dead letter queue`.

### SNS

SNS, simple notification service, looks similar to SQS at first glance. 
However, they are different by nature. SNS is a event bus or event broker,
it bring messages under a topic to the topics subscribers. 
Plus, SNS has a much rich message filter functionality.
Most importantly, SNS won't persist any message. 

To use SNS, we need to register lambda functions to topics of SNS. Then if there
is a message in the topic, the lambda function will be trigger with the message.

### EventBridge, Kinesis, Kafka

Those services are event bus as well just like SNS, however, each of them a slightly
different purposes, hence different functionality and performance profiles.

![20230210225935](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230210225935.png)

I think the details worth another post. For this post, we just need know:

- like SNS, messages are sent to lambda ( no polling )

### DynamoDB

![20230210225955](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230210225955.png)

[DynamoDB](https://aws.amazon.com/dynamodb/) is AWS's answer to NoSQL. 
The trigger is called `DynamoDB Stream`.
Every time the a mutable change in the dynamodb table happens, a event is pushed
into the stream. We can setup lambda function to poll the stream **4 time per second**. 

In this way, a mutable change in the table can trigger lambda functions.

We can only setup 2 lambda functions to 1 stream.

### S3

[S3](https://aws.amazon.com/s3/) is AWS's object store. 

There are two ways S3 trigger lambda:

- Event Notification, which is similar to dynamodb stream
- S3 Object lambda

Object lambda is somehow unique, as you can add your own code to Amazon S3 GET, HEAD, and LIST requests to modify and process data before it is returned to an application.

## Step Functions

Technically speaking, [Step Functions](https://aws.amazon.com/step-functions/) is 
an orchestration tool to connect pretty much every services in aws including lambda.

With Step Function, not only we can chain lambda with events, but also we can chain 
lambdas with lambdas. 

In step function workflow, we can connect dynamodb, sns, sqs, lambda directly without
boilerplate code. I won't expand the details in this post as it is too big as a 
topic. 

For this post, it is good enough we know that we can use step functions as a tool
to trigger lambda functions!

## Summary

Understand triggers of lambda is very important to leverage lambda's power. Those
triggers actually changes the way we design our backend software and workflows

Most of the event/message based triggers have batching behavior, so we need to
implement the batch handling logic ourselves, including batch window and batch size.

Error and failure handling of each trigger is slightly as well.

Step functions is a powerful tooling for orchestration aws services.