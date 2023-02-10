---
title: "Systematic Trading Done Serverless (4) - Queue"
date: 2023-02-08
draft: false
description: "(STDS) Part 4 - Simple Queue Service (SQS)" 
summary: "(STDS) Part 4 - What is SQS? How to use SQS? Concrete use case of SQS"
tags: 
    - serverless
    - aws
    - sqs
    - stds
categories: 
    - Software Design
lightgallery: true
resources:
- name: featured-image
  src: sqs.png
- name: featured-image-preview
  src: sqs.png
---

> Some of the content in this post is written by `ChatGPT3`. Guess which parts? :)

In the previous posts, we've met two computation tools in serverless: [lambda](https://quant.funcoder.net/posts/serverless-2-lambda/) and [fargate](https://quant.funcoder.net/posts/serverless-3-fargate/).

From this post, we will explore the tools that integrate computation together. 
Today's topic is queue. 

## Quick Review of Queue

Queue is a simple yet powerful data structure. There are so many type of queues,
such as First-in-First-out (FIFO), priority queue, unordered queue, bounded, unbounded, circular, etc.

Some of the important uses of queues include:

1. Task scheduling: Queues are used to schedule tasks in a computer system, such as printing, background processing, and other types of batch processing.
2. Resource allocation: Queues can be used to manage resource allocation, such as managing requests for disk I/O or network access.
3. Event management: Queues can be used to manage events that need to be processed in a specific order, such as user interactions or network packets.
4. Load balancing: Queues can be used to distribute workloads across multiple processing units, helping to ensure that resources are used efficiently and avoid overloading individual processors.

Overall, queues are a useful data structure for managing processes and resources in computer systems, and they help to ensure that tasks are performed in the desired order and with the desired level of efficiency.

## SQS - Serverless Queue

In the previous section, we talked more about the abstraction of queue, 
now let's zoom in to AWS serverless and have a look at the oldest serverless
service in AWS - `Simple Queue Service (SQS)`.

> Fun Fact: AWS always use first letters as the abbreviate of the service with number for duplication.
> For example, EC2 - Elastic Compute Cloud, S3 - Simple Storage Service

Amazon Simple Queue Service (SQS) is a fully managed message queue service
provided by Amazon Web Services (AWS). It provides a simple way to decouple and
scale microservices, distributed systems, and serverless applications.

With SQS, you can send, store, and receive messages between software components
at any scale, without losing messages or requiring other services to be
available. The service enables you to transmit any volume of data, at any level
of throughput, without losing messages or requiring other services to be
available.

Here is the important part, there are two type queues SQS provides: `FIFO` and `Standard`.

- FIFO
    - exact once delivery
    - ordered
    - lower throughput
    - higher latency[^1]
- Standard
    - at least once delivery
    - best effort ordering
    - higher throughput
    - lower latency[^1]

## Queue vs Message Broker

It would be helpful here to discuss an very close related topic to queue: message broker.
These two words have some overlap in my mind. 

If I ask: is Kafka[^2] a message queue or a message broker? It is a bit of both.

When it comes to AWS, we immediately think of SNS (Simple Notification System) and EventBridge.
SNS and EventBridge are more like a message broker rather than a queue. 
(hence they are separate services, although SNS is very similar to EventBridge)

> I have to say AWS some time is very confusing.. SQS, SNS, EventBridge, Kinesis? 
> I as asking why can't we just have a aws "kafka"? 
> And guess what, I found a aws managed kafka cluster service...
> which make things even more complicated...

So here I give my rule to differentiate queue and broker:

- Broker
    - more about sub/pub pattern
    - usually don't persist message ( not true, at least we don't expect it to persist )
    - has topics
- Queue
    - more about communication and polling
    - usually persist message for later consumption
    - has message types

## Use Cases

SQS is very useful when it comes to serverless design:

- Decoupling Microservices: SQS can be used to decouple microservices, allowing
  different parts of a system to run independently and asynchronously. This
  helps to ensure that one part of the system can continue to operate even if
  another part fails.
- Serverless Applications: SQS can be used as the backbone for serverless
  architectures, providing a simple and scalable way to transmit messages
  between serverless functions.
- Distributed Systems: SQS can be used to build and operate distributed systems,
  providing a way to transmit messages between multiple systems and components.
- Background Jobs: SQS can be used to manage background jobs, such as image
  processing, data analysis, and other long-running tasks.
- Workflow Processing: SQS can be used to manage workflows, providing a way to
  transmit messages between multiple steps in a workflow.
- Event-Driven Architecture: SQS can be used as the backbone for event-driven
  architectures, providing a way to transmit events between components and
  trigger subsequent actions.

> (^^ a very typical ChatGPT answer: right but somehow useless... )

Let's see if I can do better thant ChatGPT!

### Priority Queue


We can use two separate SQS queue to construct a priority queue pattern together
with `lambda`.

Basically, we can set two event triggers lambda function, q1 and 12. Then in the 
lambda handler, we build following logic:

- Poll q1, the priority queue, if has item, process it
- if not, poll q2

![20230208155041](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230208155041.png "Priority Queue")

In this way, we can push important item into priority queue q1 and less important
items to q2. 

### Async Task Processing

This is a typical queue use case. Say we have some function to do backtesting 
given some parameters. The backtesting takes very long time to finish. To avoid
timeout of the API call, we could connect job lambda function to a working queue
and polling for the next job to do if any.

A sample design could be like this:

![20230208155405](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230208155405.png "Async Task Queue")

### Auto-Scaling

Give a job queue with a lot of jobs to do, we could auto scaling our workers number
based on how much jobs in the queue![^3]

![20230208155629](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230208155629.png)

[^1]: https://bitesizedserverless.com/bite/serverless-messaging-latency-compared/
[^2]: https://kafka.apache.org/
[^3]: https://jayendrapatil.com/aws-sqs-simple-queue-service/