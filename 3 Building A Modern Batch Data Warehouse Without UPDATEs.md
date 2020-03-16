# Building A Modern Batch Data Warehouse Without UPDATEs

By Daniel Mateus Pires

Last Updated: 1/31/20

Source: https://towardsdatascience.com/building-a-modern-batch-data-warehouse-without-updates-7819bfa3c1ee

### Intro

How to design Dimensions, Facts and processes that feed them without performing mutable changes

First, assume the data is on a cheap cloud storage, processed by a computer engine that operates directly on the files (Spark/ Hive), and written in immutable blocks (Parquet / Avro)  

Know that there are options for transactionality, updates and locking in Big Data Stacks.
Big data stacks like databricks and snowflakes built very sucessful businesses around providing those services. databricks even open-sourced "delta lake", bringing some of those features to Apache Spark.  

However, Updates are complex operations in Big Data stack. Author believes that keeps most of your data processing immutable is simpler to maintain and reason about.  

Updates introduce unwanted flexibility to a data pipeline. He has seen a team extend tables owned by another team only through UPDATES instead of collaborating and extending common scripts. Immutability forces a data pipeline to have certain structure.  

### Star Schema

The Star Schema is a type of relational modeling technique that seperates measures or events(facts) relating to a business process from their contex(dimensions).  

it is more denormalized than relational database schemas (application databases), and was designed to be efficient for analytic queries (OLAP workloads).

### star schema example

In an e-commerce website, an example of a business process is a transaction on the website. Some example measures, in this scenario, are:
- amount paid during a transaction
- quantity of a product bought

The context is made of:
- the user that made the transaction
- the date of that transaction
- the product bought

![example](https://miro.medium.com/max/960/1*PG8JSNSROK-S0oyc9WvuKg.jpeg)

in reality, each transaction could contain different productsL this is a simplified model

### Snowflake schema

The dimensions can be further normalized: for example the 'brands' of products could be kept in a seperate table with a foreign key relationship to "products", creating a dimension to dimension relationship.  

The further normalization of the dimensions of a star schema results in a snowflake schema and, historically is main objective was to reduce the amount of redundant data (save storage)

the downside is that snowflake schema introduces more complexity through more joins. As storage became less of a concern a snowflake schema approach is not appropriate for the majority of cases.

### Star Schema relevance in Big Data

The star schema was created in a time where storage and compute where expensive. Because storage was expensive and limited, reducing data redundancy was a main concern of the data warehousing team.  

It was also an efficient way to support data warehousing queries as large amounts of data could be skipped on fact tables through JOINs and filters on dimension tables. The predictable accesss patterns allowed for simple optimizations, such as the creation of indexes on the foreign keys of fact tables.  

> short and sweet- all foreign key columns should have a no-clustered non unique index

https://www.datavail.com/blog/how-to-index-a-fact-table-a-best-practice/

Today, storage is cheap and it is (relatively) easy to provision compute power as needed. We could measure the engineering cost of designing and implementing those models against the amount saved on hardware, and ask outselves -- is it still worth doing?  

## Why is star schema still relevant?

### Diverse sources

Companies collect more and more data from an increasing number of sources, and the resulting datasets need to be reconciled for analytics.  

For example, a cable network might have very different systems to shot information about their TV subscribers and the subscribers of their newly launched streaming services. Similarly, their 'customer support' analytics has to integrate data from Twitter, their 3rd party call center and their support emails. 

The normalization effort has the objective of bringing together the different definitions of "subscribers", "customer feedback" and other logical entities, removing the analytics complexity that would be otherwise introduced by the heterogenous sorce systems.  

### The standard

The Data Warehouse Toolkit (1996) written by Ralph Kimball, and the Kimball Group website define concepts widely understood in the industry.  

New hires can quickly get a grasp of the Data Warehouse structure without being familiar with the specifics of the organization.  

Data engineers, data scientists and analysts have common terminology (fact, dimensions, grain), facilitating collaboration.  

### Extensibility 

- A newly added fact can re-use the existing dimensions.
- New dimensions can be added to facts by adding more foreign keys to the fact table
- As a result, new datasets can be integrated without introducing major changes to the schema

### performance

A star schema can be populated purely through inserts and computations easily parallelizeable within a Map-Reduced framework(see how in next section).  

### Query performance tuning:

Although foreign key indexes are typically not an option because they are not supported in many modern Data Warehousing frameworks -- we have other options to increase performance.  

Dimensions are typically small, and can sometimes fit in memory, enabling Map-side join optimizations(see Spark;s BroadCastJoin)  
https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-joins-broadcast.html

We can use bucketing optimization on dimension tables.  
https://kb.databricks.com/data/bucketing.html

Spark has support for star schema detection to re-order JOINs and efficiently skip data 
https://developer.ibm.com/code/2018/04/16/star-schema-enhancements-in-apache-spark/

### Adaptable to Big Data

Many of the practices from 1996 are still relevant today, but some methodologies need to be redefined. Companies such as Lyft have been sucessful at updating those Data Warehousing practices to adapt to the new technology landscape. Maxime Beauchemin has a great talk and the "managing Dimensions" section of this post is heavily inspired from it 

## Foreign keys

### Natural keys

The Data Warehousing books warned against re-using natural keys (unique IDs from the production system) as foreign keys in fact tables.  

Natural keys in the production system could potentially re-use IDs if a table is partially emptied (for performance reasons). Because the Data Warehouse keeps historic data, the re-use of IDs create clashes complicated the resolve.

### Sequentially Generated ID's

The best practice for creation of "surrogtate keys" was to use integer IDs sequentially generated by the data processing system, and detached from the production systems' natural keys.  

Integers allowed aving storage and creating smaller and efficient indexes.  

Indexes are not used in modern datawarehouses. Hive 3.0 removed indexes, that are replaced by statistics / metadata compiled in the files (ORC, Parquet), physical partitions and bucketing which, similarly, enable skipping large protions of data.  

### performance hit

Generating surrogate keys is a complex operation to paralllelize. 

The BigQuery (Google) blog post describes the limits of adding a generated sequence using the common approach (Row_number function over the new rows):

> Unfortunately, this approach is limited. To implement Row_number(), BigQuery needs to sort values at the root node of the execution tree, which is limited by the amount of memory in one execution node.

### The solution

UUIDs and hashes are easier to parallelize, making it a more scalable approach to generating IDs for big datasets.  

For 'dimensional snapshots' we will prefer hashes over UUIDs, we go through the motivation in the below sub-section: 'Managing Dimensions" / "Surrogate Keys"

https://cloud.google.com/blog/products/data-analytics/bigquery-and-surrogate-keys-practical-approach

### Managing Dimensions

### Static Dimensions

Some special dimensions such as the 'data' dimension (a static dimension), are generated, rather than being the result of data processing. Those tables can be generated in advance, and no history of their changes is kept.

When a dimension is static, we can simply overwrite teh entire data every time we need to make modifications to it.  


### Slowly and rapidly changing dimensions

For dimensions requireing changes over-time, we need a strategy to mutate the data.  

A user cam change its home address, a product can change names and a brand can change owners.  

Over the years, many "types" of slowly and rapidly changing dimension mangament strategies were formalized. Some of them keep history and most of them use "UPDATEs" to add or modify information in-place.  

There was a seperation between dimensions that have rapidly changing attributes, and the ones that have slowly changing attributes.  

In a dimension that keeps history, the entire row of data needs to be duplicated when any attribute changes in the record -- when the attribute change often, more storage is used.  

Those techniques are complex because they were designed under strict storage constraints.  

Dimensions snapshots use more storage but they are simpler to create and query. 

### Dimension snapshots

Dimensions should be much smaller than facts. The transactions, orders, and customer reviews (facts) of an e-commerce might be in the millions, but the number of unique customers (dimension) will be much smaller.  

Dimensions also differ from facts due to the "state" they carry. A record in a dimension will have an "identity" that needs to be preserved as other attributes change.  

Dimension snapshots are simple to manage.  

Every day we re-write the entire dimension table in a version snapshot

> s3://my_data_warehouse/dim_user/ds=2020-01-01
> s3://my_data_warehouse/dim_user/ds=2020-01-02

Because the entire table is re-written no-matter how many changes happen, there is no advantage in having different strategies for Rapidly and Slowly changing dimensions.  

### History

with this model, we can answer historical questions easily by joining with a filter on a specific day.  

The history is also not slowing down the queries because passing a specific date effectively skips files for any other date.  

The structure of the table makes time-series on dimension tables easy. We could, for example, compute the count of users per country of residence over time.  

### Surogate keys

UUIDs introduce randomness. To keep new surrogate keys aligned with the previous snapshot's surrogate keys we need to query that snapshot.  

Conversely, the hash is based on concatenation of keys that make up 'identity' of a record (excluding the natural key), so we can re-compute it from the current date only. This is why we prefer hashing over UUIDs for dimension snapshots.  

Notice that a daily snapshot removes the need for a second key. Some types of slowly changing dimensions implement 2 surrogate keysL one point to the 'latest' state of the dimension record, and another point to the historical snapshot of the record at the time where the specific fact was ingested. Because each snapshot contains all the data that point in time, we can use the data of the snapshot to get the latest state, or a state at a given point in time.

Won't that create a lot of data?

Data duplication is the main downside of dimensional snapshots.  

ANother downside is that the dimensions need to be created from one process only, as multiple snapshots a day would aggravate the duplication.  

Requirements to refresh dimensions on a more frequent cadence than daily will also amplify duplication.  


The gains are simplicity, easy access to history, time-series on dimensions, and performance in both writing and querying.  

However, for the cases where data duplication is not an option, we can fall back to another type of slowly changing dimension management. Using a project like Delta Lake to support UPDATEs.

https://delta.io/


### Managing facts

> Fact tables are the foundation of the data warehouse. They contain the fundamental measurements of the enterprise, and they are the ultimate target of most data warehouse queries

There are multiple types of fact models to cover the diffeerent measures and events that a fact table aims to capture. 

https://en.wikipedia.org/wiki/Fact_table#Types_of_fact_tables


### The grain

When designing our fact, we need to decide the level of detail of each row in the table.  

Pre-aggregating facts could save storage, but it is a risky bet to make, as analytic requirements from the business can require more details in the future. For most cases today, capturing fact information at the lowest level of detail is desirable.  

### Insert only

The vast majority of facts should not require UPDATEs. Even in the specific case of Accumulating snapshots, we design the model in a way that removes the need for mutability. 

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/accumulating-snapshot-fact-table/

For an e-commerce order for example, if we have the states:
- order_being_prepared
- order_shipped
- order_in_transit

Instead of going to the fact table and updating the row with an end_date every time the action finishes, we can create new states that symbolize the end of previous states:
- order_being_prepared_end
- order_shipped_end
- order_in_transit_end

Now each change of state is a sperate event that can be inserted

### Physical partitions

In order to create immutable partitions that can be overwritten in case of upstream errors or change in business requirements, we need to partition our data by the 'extraction date' -- the date when the data was extracted from the source system.  

This first partitioning key is typically not useful for query performance, but it is important in order to implement idempotent batch processing jobs. 

https://medium.com/@maximebeauchemin/functional-data-engineering-a-modern-paradigm-for-batch-data-processing-2327ec32c42a

Additionally, partitioning by the 'event date' is recommended as queries will often use it as filter (WHERE clauses).  

Other partitioning keys can be added to additionally increase query speed, but we need to make sure they are used in filters, and the resulting files are not to small.  

https://blog.cloudera.com/the-small-files-problem/

example table structure:
> under s3://my_data_warehouse/fact_transactions:

> /event_date=2020-02-05/extraction_date=2020-02-05/event_date=2020-02-05/extraction_date=2020-02-06
/event_date=2020-02-06/extraction_date=2020-02-06
...

Our data processing job created 1 partition on the 5th: the '/event_date=2020–02–05/extraction_date=2020–02–05' partition.  

On the 6th however: some transactions that took place on the previous day were captured late, so 2 partitions were created: /event_date=2020–02–05/extraction_date=2020–02–06 and /event_date=2020–02–06/extraction_date=2020–02–06, this is commonly known as "late arriving facts", and is supported by this table structure

https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/late-arriving-fact/

### In conclusion

We can avoid UPDATEs, locking and the need for ACID compliance in Data Warehousing by adapting the "Star Schema" approach to leverage cheap cloud storage.

- Foreign keys are hashes instead of integer sequences
- Dimensions use daily snapshots
- Facts are partitions by 'extraction date' to support idempotent processing and the potential overwriting of partitions

