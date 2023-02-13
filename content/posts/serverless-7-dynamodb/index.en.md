---
title: "Systematic Trading Done Serverless (7) - DynamoDB"
date: 2023-02-13
draft: false
description: "(STDS) 7 - DynamoDB"
summary: "(STDS) 7 DynamoDB"
images: []
tags: 
    - serverless
    - aws
    - systematic-trading
    - stds
    - dynamodb
categories: 
    - Software Design
lightgallery: true
resources:
- name: featured-image
  src: header.jpeg
- name: featured-image-preview
  src: header.jpeg
---

## What is DynamoDB

DynamoDB is THE serverless storage solution on aws, which is a fully managed,
serverless, key-value NoSQL database designed to run high-performance
applications at any scale.

It is:

- Distributed, meaning
    - if we have tables across multi region, we need to think about consistency
    - good horizontal scaling ability
- NoSQL, meaning
    - Better for fix pattern query instead of ad-hoc query
    - Schemaless
    - Keys and index are important
        - Primary Key
            - Partition Key
            - Sort Key
        - Secondary index

The purpose of DynamoDB is that no matter the size of your database or the
number of concurrent queries, DynamoDB aims to provide the same single-digit
millisecond response time for all operations.

## Core Concept of DynamoDB

The basic concept of DynamoDB are:

- Tables. A `table` is a collection of data (`items`)
    - table name
    - primary key, to uniquely identify an item
        - single partition key
        - composite key: partition key + sort key
- Items. Each `table` contains 0 or more `items`. An `item` is a group of `attributes` that is uniquely identifiable among all the other items.
- Attributes. Each `item` is composed of 1 or more `attribute`, which is fundamental, and cannot be broken down anymore.

### Primary Key

The `partition key` (PK) also called a hash key, as the name suggested, PK works
like a key in a dictionary, by which we can find the item directly. The PK determines
where our item are stored.

With the `sort key` (SK), we can carry out more flexible query operation instead
of just read PK. Essentially, with SK and PK, we can model a 1 to many relationship,
meaning several items can have same PK (so stored together) as long as they have
different SK assigned.

### Secondary Index

Apart from using PK and SK to query/fetch data, `Secondary Index` is another option
to query the table using an alternative key. In concept, secondary index creates
a new "primary key" (either single or composite) for the table:

- global secondary index can have different PK and SK than the original primary key
- local secondary index can only have different SK thant the original primary key

Note that we secondary index created in the table, every time the table changed,
either add, update, or delete, the index will be updated automatically. This implies
additional performance cost and of course $ cost. In exchange, we get flexible
query pattern and much better query performance.

### DynamoDB Stream

We mentioned DynamoDB stream in the post about [triggers of lambda](https://quant.funcoder.net/posts/serverless-x1-lambda-trigger/). 

It is an optional feature that capture data modification events in DynamoDB tables.
This could be very useful to build event driven applications.

Each event is represented by a _stream record_. If you enable a stream on a table, DynamoDB Streams writes a stream record whenever one of the following events occurs:

- A new item is added to the table: The stream captures an image of the entire item, including all of its attributes.
- An item is updated: The stream captures the "before" and "after" image of any attributes that were modified in the item.
- An item is deleted from the table: The stream captures an image of the entire item before it was deleted.

Each stream record also contains the name of the table, the event timestamp, and other metadata. Stream records have a lifetime of 24 hours; after that, they are automatically removed from the stream.

### Read/Write Capacity

Similar to other serverless service, DynamoDB has to mode to control scaling behavior (throughput and capacity):

- On-demand
- Provision ( Default )[^1]

I quote from aws documentation[^doc]: 

Amazon DynamoDB on-demand is a flexible billing option capable of serving thousands of requests per second without capacity planning. DynamoDB on-demand offers pay-per-request pricing for read and write requests so that you pay only for what you use.

On-demand mode is a good option if any of the following are true:

- You create new tables with unknown workloads.
- You have unpredictable application traffic.
- You prefer the ease of paying for only what you use.

If you choose provisioned mode, you specify the number of reads and writes per second that you require for your application. You can use auto scaling to adjust your table’s provisioned capacity automatically in response to traffic changes. This helps you govern your DynamoDB use to stay at or below a defined request rate in order to obtain cost predictability.

Provisioned mode is a good option if any of the following are true:

- You have predictable application traffic.
- You run applications whose traffic is consistent or ramps gradually.
- You can forecast capacity requirements to control costs.

Here is a concise video introduction to core concepts of DynamoDB: <https://www.youtube.com/watch?v=Mw8wCj0gkRc&t=231s>

## Setup DynamoDB Table

> Note that we can test dynamodb locally instead of on cloud. [^test]

The most basic table setup can be expressed as: 

```json
{
    TableName : "Music",
    // Note: DynamoDB is schemaless except for the keys! We need to define AttributeDefinitions for all the keys
    KeySchema: [
        {
            AttributeName: "Artist",
            KeyType: "HASH", //Partition key
        },
        {
            AttributeName: "SongTitle",
            KeyType: "RANGE" //Sort key
        }
    ],
    // An array of attributes that describe the key schema for the table and indexes.
    // Note: Do NOT including anything thing other than keys
    AttributeDefinitions: [
        {
            AttributeName: "Artist",
            AttributeType: "S"
        },
        {
            AttributeName: "SongTitle",
            AttributeType: "S"
        },
    ],
    ProvisionedThroughput: {       // Only specified if using provisioned mode
        ReadCapacityUnits: 1,
        WriteCapacityUnits: 1
    }
}
```

## DynamoDB Mindset

DynamoDB's API is somehow different from other database, especially SQL database.
DynamoDB does not have a build-in query planer or other optimization under the hood.
It exposes a direct access the raw data.

The only tooling we have to query is primary key ( PK and SK ) plus other secondary
indices, which essentially another set of primary keys. This mapped directly to the 
underlying data structure of dynamodb: partitions (PK) and B-Tree (SK).  
There is no join or aggregation operation in DynamoDB. The benefit is scalability.

This _direct API mindset_ affects the way we design DynamoDB tables and sometimes
makes me feel unconformable as I am used to SQL databases where I can join, aggragate
on the fly.

You can always get back to a full scan of the table to do any kind of query, 
although this is the last solution you would like to go to. It is always better
to understand the use case of the table and design the PK, SK, and secondary indices properly to 
avoid scanning, which can be super slow if the table grows.

Here is a very good article discussing the designing of DynamoDB table[^3].

The rule of thumb is that: any thing looks like a SQL table may be a bad design for DynamoDB table.
DynamoDB encourages to put related data together, not only logically but physically with PK, and 
do less query but fetch all relevant data back at one time.

There is a nice book discussing the DynamoDB design in details: <https://www.dynamodbbook.com/>

A more concise design process provided by aws: <https://docs.aws.amazon.com/prescriptive-guidance/latest/dynamodb-data-modeling/step3.html>

---

[^1]:DynamoDB provides 200 million read/write requests per month and 25GB space for FREE.
[^doc]: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html#HowItWorks.OnDemand
[^test]: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html
[^3]: https://aws.amazon.com/blogs/database/single-table-vs-multi-table-design-in-amazon-dynamodb/