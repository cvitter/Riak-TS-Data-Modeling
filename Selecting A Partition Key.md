# [Riak TS](README.md) - Selecting A Partition Key

In [Data Modeling Basics](Data Modeling Basics.md) and [How Partition Keys Work](How Partition Keys Work.md) we introduced you to the basics of Riak TS table schemas and how the partition key shapes the distribution of data within a cluster. In this section we are going to cover how the choice of partition keys affects the performance of your Riak TS cluster (both in terms of writes and reads per second) and the queriability of the data stored in your cluster.  

* [Optimizing Partition Keys for Write Performance](#optimizing-partition-keys-for-write-performance) 
* [How Riak TS Executes Queries](#how-riak-ts-executes-queries) 
* [Why Use Quantums At All](#why-use-quantums-at-all)

## Optimizing Partition Keys for Write Performance

The choice of partition key directly affects Riak TS's write performance and it is important to design partition keys that allow writes to be distributed evenly across all of the cluster's nodes to maximize performance. In this section we are going to talk about things to consider when designing partition keys that will help you limit performance bottle necks. Let's start by taking another look at the example table that we created in the [Data Modeling Basics](Data Modeling Basics.md) section with the following primary key:

```
	PRIMARY KEY (
		(StationId, QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 StationId, ReadingTimeStamp
	)
```

The partition key in this example specifies that the combination of the StationId column and ReadingTimeStamp column will hash to one partition (quantum) for a twenty-four hour period. To understand how this affects performance lets consider the following hypothetical conditions:

* There are 100,000 weather stations reporting data
* Each weather station reports once a minute (1440 writes per day)
* There are an average of 5,001 writes per second across the whole cluster (100,000 updates a minute / 60 seconds = 1,667 updates per second * 3 for standard Riak TS replication factor)
* In a 5 node cluster each node would handle an average of 1000 writes per second

Based on the theoretical conditions our partition key design is ideal from a perspective of distributiing the write work load around our cluster optimizing both performance and data storage.

Now it might be tempting to create a primary key that looks like the following:

```
	PRIMARY KEY (
		(QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 ReadingTimeStamp, StationId
	)
```

In this primary key the partition key consists of only the quantum function meaning that all records written to the table will hash to the same partition for a one day time period (144,000,000 records per day vs 1,440 in the first example). Using this quantum only partition key gives you the ability to query across weather stations (more on querying later on in this document) but will have a negative impact on performance as writes for the day won't be distributed equally across all five nodes. Based on our previous example numbers above we would expect to see writes to only 3 out of 5 nodes in the cluster (with the standard replication factor of 3) with each of those nodes handling 1,667 writes per second vs 1000 writes per second.


## How Riak TS Executes Queries

When Riak TS executes a query it splits the query into sub-queries along quantum boundaries. Each sub-query is then sent to the corresponding virtual node that handles that quantum's matching partition (or partition key's where no quantum is specified). At the partition level the virtual node **first** performs a range scan based on the partition keys in the ``` WHERE ``` clause **and then** applies a secondary filter on any additional fields included in the where clause.

Based on this query execution pattern you should design your partition key keeping in mind the following rules of thumb:

* Querying across fewer quanta is better in terms of performance. When possible you should use partition keys that limit the number of quanta you need to span in queries.

* Queries that **only** require partition keys in their ``` WHERE ``` clauses will be faster than queries that add non-partition-key columns to the ``` WHERE ``` clause since non key fields require a second level of filtering **after** the virtual node perfoms the intitial range scan on a partition.

## Why Use Quantums At All

The design of your partition keys is heavily influenced by how you plan to query your data.

Selecting partition keys

Although a quantum function is not required in a table's partition key (see: [Data Modeling Basics](Data Modeling Basics.md#partition-key)) Riak TS is optimized for their use. 


---

 **Previous**: [How Partition Keys Work](How Partition Keys Work.md) | **Next**: 
 