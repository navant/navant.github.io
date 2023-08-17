---
title: "Apache Iceberg-Considerations"
date: 2023-08-16T11:30:03+00:00
draft: false
# weight: 1
# aliases: ["/first"]
tags: ["data"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "Apache Iceberg is been buzzing for a while this post is to deep dive on what that means and its solution considerations"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page/
editPost:
    URL: "https://github.com/navant/navant.github.io/tree/main/content/posts"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---


### **What is it?**

> Its a table format to store and interact with the Data
> 

> From user perspective who are familiar in SQLs it is a way to answer the question of what data is in this table? , its not the data that's going to be in some database, but stored in a file that reside in a data lake (like s3) and then users can interact with that table with SQL. Some Data Warehouses support even loading this file as External files and enable Analytical query.
> 

Multiple users and tools can read, write or interact with the table at same time. All the corresponding data is written as table. You can modify the underlying schema on existing file without other rows and columns are impacted

### Solution Considerations:

- Are you going to load and store all your data into a data warehouse?(Pure Data warehouse) such as AWS Redshift, Snowflake, Big Query, Synapse Analytics
- Are you going to load and store all data in a data lake and access? (Data Lake)
- Are you going to load and maintain some data in data lake and some data in warehouse? (Delta Lake scenario)
- Do you want to maintain data in Open Table format
- What other considerations:
- Who Supports it:
    
    Cost
    
    Performance
    
    Scalability
    
    Flexibility
    

If you want to deep dive more , here are some additional information

### How it has emerged?

Netflix created it to solve the issues they faced on the big data format on performance, atomicity and reliability.

Some history: True heavy weight in the big data arena was Apache Parquet , which has been the go-to choice for the columnar storage file format .

Apache Iceberg has been designed to address some of the issues that is faced with Parquet file format.

### Dif0ferent characteristics

Below provides a high level comparison with Apache Parquet

![https://cdn-images-1.medium.com/max/800/0*HQWKOEt3AkLutpj4.png](https://cdn-images-1.medium.com/max/800/0*HQWKOEt3AkLutpj4.png)

### Schema Evolution-

Imagine you want to modify the schema while maintaining compatibility with older data — can we not able to do that in Parquet?. Parquet supports schema evolution to a certain extent, but it’s not as robust as Iceberg. For instance, Parquet allows you to add new columns or remove existing ones, but it doesn’t support evolving nested structures or updating partitioning specifications.

More details on these can be found on the [Iceberg docs site](https://iceberg.apache.org/evolution/).

### Atomic Transactions-

Data remain consistent even during concurrent writes and reads , which is achieved by snapshot isolation and optimistic concurrency control .

### Performance and Efficiency

**Predicate Pushdown** is supported which means -> filtering data at the storage level than at processing level. Even though Iceberg and Parquet support thism ,Because data is tracked at the file level in Iceberg format, smaller updates can be made much more efficiently.

In [Ryan Blue’s presentation](https://youtu.be/nWwQMlrjhy0?t=138) , he shares the results of an example use case at Netflix:

- For a query on a Hive table, it took 9.6 minutes just to *plan* the query
- For the same query on an Iceberg table, it only took 42 seconds to plan and execute the query

### Storage

Parquet is a Columnar format BUT as mentioned earlier Iceberg on the other hand is table format than file format. It can work with various file formats including Parquet, ORC, Avro — so more flexibility

Iceberg takes partitioning to next level with dynamic partitioning and partioning spec evolution. This feature allows you to change partition schemas without rewriting the entire table

### **Event listeners**

Event listeners in Apache Iceberg are a way to be notified of changes to an Iceberg table. They are implemented as a simple interface, `Listener<E>`, where `E` is the type of event that the listener is interested in.

Event listeners can be used to implement a variety of functionality, such as:

- Monitoring changes to an Iceberg table
- Updating downstream systems when an Iceberg table changes
- Ensuring that data is consistent across multiple systems

### **Logical view**

- Iceberg provides the ability to continually expose a logical view to your users, decoupling the logical interaction point from the physical layout of the data.
- It is much easier to experiment with different table layouts behind the scenes. Once committed, the changes will take effect without users having to change their application code or queries. If an experiment turns out to make things worse, the transaction can be rolled back and users are returned to the previous experience.

### File Structure

![https://cdn-images-1.medium.com/max/800/1*WCDa16CT2blmbHY17cLutQ.png](https://cdn-images-1.medium.com/max/800/1*WCDa16CT2blmbHY17cLutQ.png)

Link: [https://www.youtube.com/watch?v=iGvj1gjbwl0](https://www.youtube.com/watch?v=iGvj1gjbwl0)

Courtesy:[https://github.com/johnny-chivers/iceberg-athena-aws#main-tutorial](https://github.com/johnny-chivers/iceberg-athena-aws#main-tutorial)

### How it actually looks

use S3 example:

their will be two objects— data and metadata

![https://cdn-images-1.medium.com/max/800/1*_8VOqn6rZIQHKLdl0yPdAw.png](https://cdn-images-1.medium.com/max/800/1*_8VOqn6rZIQHKLdl0yPdAw.png)

### Step by Step:

What happens after a create table( Iceberg is SQL like)

It creates a entry into a catalog which denotes a table is created. (still no data file )

![https://cdn-images-1.medium.com/max/1200/1*79UqUFwj-7Own-yYnIgETw.png](https://cdn-images-1.medium.com/max/1200/1*79UqUFwj-7Own-yYnIgETw.png)

Now insert a record to the table

![https://cdn-images-1.medium.com/max/1200/1*bbO_Tvj4MUBi7zz9sv5MiQ.png](https://cdn-images-1.medium.com/max/1200/1*bbO_Tvj4MUBi7zz9sv5MiQ.png)

now if u update the record

![https://cdn-images-1.medium.com/max/1200/1*CKmV9Sqvr8x8WdOYOidq3A.png](https://cdn-images-1.medium.com/max/1200/1*CKmV9Sqvr8x8WdOYOidq3A.png)

How it will actually look like

![https://cdn-images-1.medium.com/max/800/1*2F2fBJVIu2pHWn2dxHtaMw.png](https://cdn-images-1.medium.com/max/800/1*2F2fBJVIu2pHWn2dxHtaMw.png)

You need a catalogue and query engine to connect to navigate the data. In this case its AWS Glue and AWS Athena

###