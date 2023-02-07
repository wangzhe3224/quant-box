---
title: "Systematic Trading Done Serverless (3) - Fargate"
date: 2023-02-07
draft: false
description: "(STDS) Part 3 - Compute with Fargate. What is Fargate? Why using Fargate? Compare with Lambda?"
summary: "(STDS) Part 3 - Compute with Fargate. What is Fargate? Why using Fargate? Compare with Lambda?"
tags: 
    - serverless
    - aws
    - fargate
    - building-block
categories: 
    - Software Design
lightgallery: true
resources:
- name: featured-image
  src: fargate.png
- name: featured-image-preview
  src: fargate.png
---

## What is `Fargate`

AWS `fargate` is serverless compute for containers. In concept, `fargate`
provide a service that user only need to provide a image containing business
logic, and AWS will handle the container management. 

If we put the other serverless computation engine `lambda` with `fargate`[^1]:

![20230206211222](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230206211222.png
"EC2 vs Lambda vs Fargate")

We can see that `fargate` is just between a server solution (ec2) and `lambda`
in terms of flexibility and operation burden.

As this series is about serverless let's compare lambda and fargate, which are
the only computation engine in our serverless toolbox. (This is not to say we
cannot use server based services in a serverless architecture, in fact, once we
follow serverless design patterns ie micro-service and event based, it is very
easy to plug any server based services.)

## `Lambda` vs `Fargate`

### Start Time

Lambda job starts much faster than Fargate tasks. In lambda even the slow cold
start is counting in milliseconds, whereas with Fargate, we are talking about
minutes.

The long start time makes fargate more suitable for long lasting sessions so
that the cost of startup is meaningful, whereas lambda is for those jobs short
lived.

### Computation Power

In terms of computation power, fargate is much more powerful than lambda (with
trade off the start time).

![20230206234224](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230206234224.png
'Fargate Configs')

With fargate, you can get a container with 16 vCores and 120 GB memory to run
your image, whereas in lambda the memory is capped with 10 GB (as of 2023, aws
seems increasing the limit during the past years). And in lambda the CPU is
allocated proportion to memory sizing with the cap of 6 vCores.

In systematic trading land, there are definitely some tasks that requires a
beefy container to run.

### Duration

Fargate jobs can run forever just like a normal server, whereas lambda has a
duration cap at 15 min.

Haven said that, if you have a task that need to run 24*7, probably `EC2` is
what you need, because running same size fargate container 24*7 is more
expensive that a same size EC2 box.

In systematic trading land, if you have some latency sensitive task that need a
much longer session, say during US market open hours, fargate is the best
choice.

### Operation and Scaling

Fargate is more involved in terms of operation, for example, the auto-scaling of
fargate is not as straightforward as lambda. With lambda there is very little
operation to do and auto scaling is really, say, auto.

On the other side of the coin, more operation means more control. Fargate
proviode more control to the runtime.

## When to choose `Fargate`

Choose `Fargate`:

- you want to migrate your docker container service to serverless
- any job you want to run more than 15 min
- long session jobs
- low latency jobs
- computation intensive jobs, for example requirement more than 10 GB memory

Choose `Lambda`:

- short session jobs
- latency is not a big deal
- computation less intensive

Although before deicide lambda and fargate, it is always better to think about
the structure of the task.

Can we simply break down a big task that need big fargate container to several
lambda functions? From my experience, in some cases use `Step Function` with
lambda can simple break down the big fargate jobs.

We also need to think about the cost. The worst thing we can do is that we use
fargate and it turns out it is more expensive than a EC2 solution.

## Final Thought

Comparing to `Lambda`, `Fargate` is more like traditional server host solution.
It does not affect design architecture as much as lambda does.

However, we should never stick to a toolbox and ignore all other tools, in the end
our target is to build a software, trading system in our case, we should use
more tooling to help us as long as it fit in the system.

## References

- [Introducing AWS Fargate - AWS Online Tech
  Talks](https://www.youtube.com/watch?v=wrZvlJlcZio&t=66s)

[^1]: https://medium.com/thundra/getting-it-right-between-ec2-fargate-and-lambda-bb42220b8c79