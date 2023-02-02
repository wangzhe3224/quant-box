---
title: "Systematic Trading Done Serverless - Part 1"
date: 2023-02-01
draft: true
description: "Part 1 - Introduction to Serverless"
images: []
tags: 
    - serverless
    - aws
categories: 
    - Design
lightgallery: true
---

> This series is about building a systematic trading system prototype using serverless computing.
Although the implementation and some explanations are using a specified provider AWS, 
most of the contents are applied to any serverless providers who provide similar services.

----


{{< admonition note "To Compute or not to compute?" >}} 
Systematic trading is all about computation.
_whereas_   
Serverless shires when computation is not happening.

(I will explain) :(far fa-grin-squint fa-fw): 
{{< /admonition >}}

Let me walk you through how these two seemingly contradictory things work together VERY well.

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
__However, we are still coding against operating systems, our runtime__.
For example, we need to setup a message broker cluster, say Kafka, to flow events
around, to setup a PostgreSQL server, to think about the hard drive spaces on the box,
setup JVM on the box to run 2 lines of Java code (sorry, this is a joke, you probably cannot create a 2 line java binary you need 3.), setup Python version to run another python process, etc.

Serverless bring us further: __it removes the operating system layer from us__.
Technically speaking, serverless is a set of unlimited services instead of boxes ready to use.

In the end, Serverless is not really without server, someone has to do heavy lifting. 
It is more about thinking without server(s).

This leads us to next section: the building blocks.

## Serverless building blocks

What are we thinking when we create a software or an application or even trying to do anything with a computer?

- Computation
- Communication
- Storage

For example, if I would like to trade Bitcoin on a 1 min bar data. 

First, I will download some data from a provider. This process involves:

- Call some APIs

## Whats serverless used for?

This figure gives you an idea the scale of serverless usage these days[^1]:

![AWS lambda usage](https://raw.githubusercontent.com/wangzhe3224/pic_repo/master/images/20230201231216.png "AWS lambda usage")

There are 10 trillion lambda invocations (the computation service of serverless)!

[^1]: https://www.youtube.com/watch?v=0_jfH6qijVY