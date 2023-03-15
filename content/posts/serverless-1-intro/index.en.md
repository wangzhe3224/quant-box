---
title: "Systematic Trading Done Serverless (1) - Introduction"
date: 2023-02-01
draft: true
description: "(STDS) Part 1 - Introduction to Serverless."
summary: "(STDS) Part 1 What is serverless? Why it is a good fit for systematic trading system?"
images: []
tags: 
    - serverless
    - aws
    - systematic-trading
    - stds
categories: 
    - Software Design
lightgallery: true
resources:
- name: featured-image
  src: header.jpg
- name: featured-image-preview
  src: header.jpg
---


> This series is about building a systematic trading system prototype using
>   serverless computing. Although the implementation and some explanations are
>   using a specified provider AWS, most of the contents are applied to any
>   serverless providers who provide similar services.  
>   I am not sure how many posts are there to complete this series because the
>   prototype is a building in progress as well. Haven't said that, here is my
>   plan:  
> 
>   1. [Introduction to Serverless](https://quant.funcoder.net/posts/serverless-1-intro/)
>   2. [Computation for short- Lambda](https://quant.funcoder.net/posts/serverless-2-lambda/)
>   3. [Computation for long - Fargate](https://quant.funcoder.net/posts/serverless-3-fargate/)
>   4. [Communication - Queue](https://quant.funcoder.net/posts/serverless-4-queue/)
>   5. Communication - Message Bus
>   6. Storage for objects - S3
>   7. [Storage for semi-structured data - DynamoDB](https://quant.funcoder.net/posts/serverless-7-dynamodb/)
>   8. API design and implementation - API Gateway
>   8. Orchestration with workflow - Step Functions
>   9. [Serverless systematic trading - Daily Data](https://quant.funcoder.net/posts/insider_trade_pipeline/)
>   9. Serverless systematic trading - Intraday Data
>   10. Serverless systematic trading - Research
>   11. Serverless systematic trading - Batched Trading Strategy
>   12. Serverless systematic trading - Event-driven Trading Strategy
>   13. Serverless testing - Unit, Integration and Local test
>   13. Serverless dev-ops - Code Pipeline

<!-- Most of the contents come and will come from my dissertation for [Masters'
degree (MSc) in Software Engineering from the University of
Oxford](https://www.cs.ox.ac.uk/softeng/courses/subjects.html). -->

----

{{< admonition note "To Compute or not to compute?" >}} Systematic trading is
all about computation. _whereas_   
Serverless shires when computation is not happening.

(I will explain) :(far fa-grin-squint fa-fw): {{< /admonition >}}

Let me walk you through how these two seemingly contradictory things work
together VERY well.

This post expands this way: first, introduce what is serverless and its building 
blocks. Then we explore why serverless may or may not fit in systematic trading 
system design and implementation.

## What is Serverless?

Serverless is the new ( about 8 years old as of 2023 ) cool kid in cloud
computing domain.

In concept, Serverless gives developer a new set of tools to build
application/software. And new tooling often changes the way we design,
implement, and deploy our software. Hence the new serverless tooling results a
new architecture, namely `Serverless Architecture`.

A discussion of serverless without cloud computing is not completed. Serverless
born from Cloud Computing. In cloud computing, comparing to non-cloud computing,
the hosting and managing of the infrastructures are off loaded to cloud
providers from us ( developers ). However, the tooling is not really changed
much, we still think about some linux server/boxes hosted somewhere, and we use
container tools to orchestrate our services we coded up. time to time, we need
to think about how to scale the servers, how to migrate to a bigger box, how to
do load-balancing, etc.

To put it simple, with cloud computing, we always have unlimited linux boxes
ready to use without thing maintain the hardwares (as long as we pay for it).
__However, we are still coding against operating systems, our runtime__. For
example, we need to setup a message broker cluster, say Kafka, to flow events
around, to setup a PostgreSQL server, to think about the hard drive spaces on
the box, setup JVM on the box to run 2 lines of Java code (sorry, this is a
joke, you probably cannot create a 2 line java binary you need 3.), setup Python
version to run another python process, etc.

Serverless bring us further: __it removes the operating system layer from us__.
Technically speaking, serverless is a set of unlimited services instead of boxes
ready to use.

In the end, Serverless is not really without server, someone has to do heavy
lifting. It is more about thinking without server(s).

This leads us to next section: the building blocks.

## Serverless Building Blocks

![20230202205426](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230202205426.png)

What are we thinking when we create a software or an application or even trying
to do anything with a computer?

- Computation
- Communication
- Storage

For example, if I would like to trade Bitcoin on a 1 min bar data. I may do
following:

- Call some APIs or websocket to stream data (Computation and Communication)
- Once bar data ticks, I compute my target position based on historical
  information ( Computation and Communication)
- To do that, I need to access disk/database to load long history into memory (
  Storage )
- Once I have target position, orders are computed and submitted to broker (
  Computation and Communication )
- The process waiting for other events, such as order filles, rejected or next
  bar ticks

As you found may find as well, we are not thinking about where to put the
processing in which operating system. We are thinking about three different
services: computation, communication, and storage.

![20230202225223](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230202225223.png)

Please let me use AWS serverless as example to expand each components.

### Computation

We have 2 tools in computation: `AWS Lambda` and `AWS Fargate`.

Lambda is an event-driven, pay-as-you-go compute service that let you run code
without provisioning or managing servers. You can think Lambda as a ready to use
container with several runtime time already setup for you.

There are only two things we plug into Lambda: our business logic/computation
code and resources and runtime we want. Out of box, Lambda support mainstream
runtimes such as python, nodejs, go, rust, Java, C#.

*Lambda is suitable for short lived (less than 15 minutes) and small (less than
10gb memory) computation.*

What about long lived and big computation that Lambda cannot handle? We have
`Fargate`.

Fargate is basically a container runtime managed by aws. There are also two
things we plugin: our code image and resource limit.

In short, serverless computation is about runtime + code and resource we need.

### Storage

I view storage as three levels:

- long term with limited query ability, like a file system
- long term with reach query ability, like a database
- short cache, like Redis

AWS gives us more than those categories, but let us focus on those:

- S3: long term object store, where we can put anything, text, csv, binary
- DynamoDB: long term semi-structured store, where we put data that we want to
  query against
- ElasticCache: low latency cache. ( AWS claims it has microsecond level
  latency.. )

All above storage service we can assume:

- no storage limit
- no concurrency limit
- auto scaling

Sounds good right? Yeah, but we need to pay the bill.

### Communication and Integration

Communication services are the key to hook everything together. 

We have to patterns of communication:

- Queue based
    - AWS SQS
- Message Bus based ( Pub/Sub )
    - AWS SNS
    - AWS EventBridge
- API pulling
    - AWS API Gateway
- Workflow
    - AWS Step Function

SQS is a fully managed message queuing for microservices, distributed systems,
and serverless applications.

SNS is fully managed Pub/Sub service for A2A and A2P messaging.

EventBridge is build event-driven applications at scale across AWS, existing
systems, or SaaS applications

API Gateway  is a fully managed service that makes it easy for developers to
create, publish, maintain, monitor, and secure APIs at any scale.

Step Function is a new service which is a visual workflow service that helps
developers use AWS services to build distributed applications, automate
processes, orchestrate microservices, and create data and machine learning (ML)
pipelines. 

### Summary

Those three types of services are exactly our new serverless building blocks.
And they are always ready for us to use, to scale up and down.

Put it in this way, there will always unlimited CPU and memory, unlimited scaled
message broker, and unlimited storage readly for us to deploy our logical code
and integration logic.

We can build pretty much anything with those components, of course including `a
systematic trading system`.

This leads us to the next section: systematic trading software.
But before that, let's briefly discussion the serverless architecture.

## Serverless Architecture

![20230202205622](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230202205622.png)

Architecture is too big as topic for a section in a blog. But I would like
mention a few thing that I found interesting when it comes to serverless
architecture.

> The tools we use have a profound (and devious) influence on our thinking
> habits, and, therefore, on our thinking abilities.  
> 
> By Edsger W. Dijkstra 

Serverless dynamically changed our tool box, hence changed the way we design,
implement, and deploy our software. One of the biggest change is that serverless
encourages Event-driven design and fearless scaling.

The other thing is that we need to think more about the cost of running the
software. Since almost all the services we have is pay as you go model, our cost
is not constant any more, like the good old days we have some boxes running for
us all day long even doing nothing. Because the cost is dynamic now, we need to
think about how to optimize it.

(this is why I was saying serverless shires when computation not happening. We
only pay when when compute.)

The last thing is scale of the project. Serverless Architecture is really
fantastic in the sense that it fits from very small ad-hoc project to large
enterprise projects. And as soon I adapt serverless, the design for smaller
project can move to large scale stepup incrementally.

For a startup type product without much traffic, aws's free tier plan will give
us nearly free ride of the whole infrastructure. When the project get more
attention, the same code scales automatically. This is simply amazing.

## Systematic Trading Software

Let's analyses what properties a systematic trading system should have:

- [p1] handle the different velocities of data, from ms to days or even months.
- [p2] handle different resource limitations. The resources here are CPU cores,
  memory capacity, disk space, and most importantly the time to finish the
  computation.
- [p3] handle ever-increasing new predictors, new datasets, and new models that
  are added into the system.
- [p4] system should be available all the time ( especially in the crypto world,
  market opens 24 * 7)
- [p5] reduce costs of all kinds. This includes capital to buy and maintain the
  infrastructure, and human resources to manage the infrastructure.
- [p6] be reactive. The system should be reactive to different events such as
  order filled, market data, or human intervention.
- [p7] be flexible. Handle new forms of data, trading rules, etc.
- [p8] time to market should be as short as possible.
- [p9] some systems need low latency. For example, the time between a market
  event to order execution can be limited to sub-milliseconds.
- [p10] can be debugged and fixed quickly. If the systems show abnormal
  behavior, we should be able to identify and fix the problem quickly.

## Serverless: the Good

- Scalability
- High Availability
- Reducing Cost
- Flexibility
- Low Operation Overhead

### Scalability

Serverless computing automatically scales resources based on demand, allowing
applications to handle sudden spikes in traffic without any additional effort or
cost.

This fits p1, p2, and p3:

- p1: The ability to scale up and down based on which part of the system is
  running is a huge win in terms of infrastructure admin.
- p2: The ability to provide different resources given the limitations on the
  fly is very beneficial.
- p3: serverless services normally can scale without any overhead.

### High availability

Most serverless providers offer built-in high availability, which helps ensure
  that your applications are available even in the case of infrastructure
  failures.

This maps to p4.

### Reducing cost

This fits p5.

Cost is important in trading in general, as cost in trading may turn a
profitable trade into a loss. However, since we invest so much in the
infrastructure to support our trading system, the cost of the infrastructure and
operation becomes increasingly large.

With serverless, you only pay for the specific resources you use, rather than
  paying for a fixed amount of resources upfront. This can lead to significant
  cost savings, especially for applications with variable or unpredictable
  workloads.

Although it is difficult to know the details of the design of different
companies (it is still a bit secret industry even in an open Internet age ).
However, we do know companies spend a huge amount of money and resource to build
their in-house infrastructure to support their systematic trading business.

Some companies are adapting to outsource their infrastructure to cloud
providers, ie moving to cloud computing. Serverless is a natural fit, as
serverless is by nature cloud computing at its core.

Any performance optimizations you make to your code will not only increase the
speed of your app, but they’ll have a direct and immediate link to a reduction
in operational costs,

### Flexibility

Serverless architecture forces to decouple the system component due to the way
it manages the computation resources, leveraging microservices and
message-passing patterns. 

Another aspect is that most of the serverless providers, Amazon AWS, provide a
different runtime environment for a different types of languages. This means we
could introduce different languages in different components. For example, we
could implement an order management system by compiled languages such as Rust
and Java; and we could realise a predictor component by Python leveraging the
powerful data analysis ecosystem of Python.

This maps to p7.

### Reducing operation overhead and better integration

Serverless providers offer a wide range of services that can be easily
  integrated with your applications, such as databases, message queues, and
  storage.

Serverless allows you to build event-driven applications, which can
  automatically trigger functions in response to specific events, such as
  changes to a database or the arrival of new data.

This fits in p6, p7, and p8.

## Serverless: the Bad

- Cold Start
- Unstable Latency
- Vendor Lock-in
- Hard to Debug
- Limited State Management
- Complex Choreography 

Noticed that p9 and p10 are not mentioned in the previous section.

### Cold start

This may be harmful to p9.

### Unstable Latency

This may be harmful to p9.

### Vendor lock-in

This may increase the cost of switching vendors, hence p5.

### Limited debug and monitoring options

This may be harmful to p10.

### Limited on state management and duration

This may be harmful to p7.

### Complex choreography

This may be harmful to p8.

## The End

In this series, we will explore different serverless patterns that can be used
to build robust systematic trading/research system, and how can we reduce the
downside of serverless within systematic trading software.

One thing I would like to mention, the native serverless tool set is __NOT__
suitable for trading system requires low latency. By low latency, I mean any
latency under 100ms. The reason is that all the messaging services (SNS, SQS,
EventBridge) has a median latency ranging from 16ms to 400ms, and it's p99
latency can reach 1500 ms for step function![^2]

Haven said that, it does not mean serverless cannot help with low latency
trading. We all know that even for a low latency trading system, not all the
parts requires the same latency limit. The strong benefit for serverless is that
you can migrate part of the system and integrate it into original software
easily and incrementally.

All right, I hope this part makes you feel excited to build trading system with serverless.
With these new cool tools in the box, let's start the journey.

## Additional Fun Facts

This figure gives you an idea the scale of serverless usage these days[^1]:

![AWS lambda
usage](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230201231216.png
"AWS lambda usage")

There are 10 trillion lambda invocations (the computation service of
serverless)!

[^1]: https://www.youtube.com/watch?v=0_jfH6qijVY
[^2]: https://bitesizedserverless.com/bite/serverless-messaging-latency-compared/
