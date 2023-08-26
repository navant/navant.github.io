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
    URL: "https://github.com/navant/navant.github.io/tree/main/content/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---
# Apache Iceberg — Solution Considerations

Apache Iceberg is been buzzing for a while this post is to deep dive on what that means and its solution considerations

### **What is it?**

> Its a table format to store and interact with the Data
> 

> From user perspective who are familiar in SQLs it is a way to answer the question of what data is in this table? , its not the data that's going to be in some database, but stored in a file that reside in a data lake (like s3) and then users can interact with that table with SQL. Some Data Warehouses support even loading this file as External files and enable Analytical query.
> 

Multiple users and tools can read, write or interact with the table at same time. All the corresponding data is written as table. You can modify the underlying schema on existing file without other rows and columns are impacted

![Example image](/iceberg.png)
### Solution Considerations:

- Are you going to load and store all data in a data lake and access? (Data Lake)
    - Ideal usecase  , esp If you want to store and query directly from data lake or adopting a lakehouse architecture

- Are you going to load and store all your data into a data warehouse?(Pure Data warehouse) such as AWS Redshift, Snowflake, Big Query, Synapse Analytics
    - Not Applicable  , you don't need this as cloud warehouse take care of this for you
- Are you going to load and maintain some data in data lake and some data in warehouse? (Delta Lake scenario)
    - **Partial**
        - **AWS** : Apache Iceberg supports popular data processing frameworks such as Apache Spark, Apache Flink, Apache Hive and Presto. AWS services such as [Amazon Athena](https://aws.amazon.com/athena/), [Amazon EMR](https://aws.amazon.com/pm/emr/), and [AWS Glue](https://aws.amazon.com/glue/), include native support Apache Iceberg for transactional data lake, often based on storage in [S3](https://aws.amazon.com/s3/).
        - 
        - **Snowflake** - can query the data directly in Iceberg and treat as external table
        - Google Big Query - Can support load and query from  iceberg table. [https://cloud.google.com/bigquery/docs/query-iceberg-data](https://cloud.google.com/bigquery/docs/query-iceberg-data)
        
        <aside>
        💡 **Gotcha:**
        
        - AWS Redshift still dont support reading Iceberg format (atleast in Aug 2023, strongly awaited feature in the roadmap)
        - Azure also cannot support read and write in Iceberg format natively (atleast in aug 2023) so need work around
        </aside>
        
- Do you want to maintain data in Open Table format
    - **yes , but even parquet can also support this**

<aside>
💡 **Final call outs:**

- Not easy or seamless yet but way to go in the future
- Do you deal with high volume of data and manage the data (query, compute) from data lake directly - Iceberg format is way to go
- Do you willing to migrate from delta tables or cloud native format to iceberg - yes
- If you are using AWS Redshift or Azure - its not yet seamless to interact with iceberg
- If you are storing data directly in AWS Redshift or Big Query or Azure, then you dont need to consider.
- How ever if you how ever if you are exporting data  then store data in iceberg in external table is good option

</aside>

If you want to deep dive more , here are some additional information

### How it has emerged?

Netflix created it to solve the issues they faced on the big data format on performance, atomicity and reliability.

Some history: True heavy weight in the big data arena was Apache Parquet , which has been the go-to choice for the columnar storage file format .

Apache Iceberg has been designed to address some of the issues that is faced with Parquet file format.

### Different characteristics

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

Additional Cost consideration:

1. **Storage Efficiency**: Iceberg has certain optimizations, like column pruning and partition pruning, that can reduce the amount of data read, potentially saving costs on storage. However, schema evolution might lead to duplicate data in the short term until older versions of data are garbage collected.
2. **Compute Costs**: Efficient data retrieval can save on compute costs, as tasks complete faster. This means less CPU and memory usage in cloud environments or distributed systems.
3. **Integration Costs**: While Iceberg integrates with popular computation frameworks like Spark, Flink, and Presto, there might be initial costs related to setting it up, especially if migrating from another table format or storage solution.
4. **Migration Costs**: If you're transitioning from another storage layer to Iceberg, there might be costs associated with the migration process, which includes time, resources, and potential cloud or infrastructure charges.
5. **Maintenance Costs**: Like any technology, there are costs associated with maintaining and managing the infrastructure. This includes managing snapshots, performing garbage collection, and ensuring schema compatibility. There may also be costs related to training your team on the new system.
6. **Versioning Costs**: One of Iceberg's features is its ability to handle versioning and snapshots. While this is a beneficial feature, maintaining multiple versions of data can increase storage costs until older snapshots are deleted.
7. **Performance Tuning**: Optimizing Iceberg for your specific workload can involve time and expertise, leading to associated costs. For instance, choosing the right partitioning strategy is essential for performance but might require experimentation.
8. **Vendor Lock-in**: If you're using a particular cloud provider's storage solution and are considering moving to a more open format like Iceberg to avoid vendor lock-in, consider potential costs of transferring data out of your current cloud storage, as well as potential benefits in flexibility and potential cost savings in the future.
9. **Backup and Disaster Recovery**: Backup strategies might differ with Iceberg, potentially affecting costs, depending on your chosen storage backend and associated charges for data retrieval or redundancy.
10. **Open Source**: While Apache Iceberg is open source and free to use, implementing, customizing, and maintaining the software might require expertise. This could involve hiring or training staff familiar with the system.

In summary, while Apache Iceberg offers many benefits in terms of performance, flexibility, and scalability, there are costs to consider when implementing and maintaining it. It's essential to weigh these costs against the potential benefits for your specific use case and infrastructure.

###