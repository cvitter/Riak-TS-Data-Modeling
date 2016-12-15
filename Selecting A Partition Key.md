# [Riak TS](README.md) - Selecting A Partition Key

In [Data Modeling Basics](Data Modeling Basics.md) and [How Partition Keys Work](How Partition Keys Work.md) we introduced you to the basics of Riak TS table schemas and how the partition key shapes the distribution of data within a cluster. In this section we are going to cover how the choice of partition keys affects the performance of your Riak TS cluster (both in terms of writes and reads per second) and the queriability of the data stored in your cluster.  

* [Optimizing Partition Keys for Write Performance](#optimizing-partition-keys-for-write-performance) 
* [How Riak TS Executes Queries](#how-riak-ts-executes-queries) 
* [Why Use Quantums At All](#why-use-quantums-at-all)

## Optimizing Partition Keys for Write Performance

Write performance in a Riak TS is affected by a number of factors including:

* The number of nodes in a cluster
* The hardware specifications of physical nodes including CPU number and speed, RAM, drive performance, network card speed, etc.
* The load balancer directing write requests to nodes (and the algorithm used by the load balancer to direct traffic)
* The size of the object being written
* And the partition key

The key to maximizing performance in a cluster is to ensure that all of the nodes in the cluster are able to equally share in the write workload. From a schema design perspective your choice of partition key is the number one thing affecting write performance. The goal is to create a partition key that avoids performance issues. In this section we are going to talk about things to consider when designing partition keys that will help you limit performance bottle necks. Let's start by taking another look at the example table that we created in the [Data Modeling Basics](Data Modeling Basics.md) section with the following primary key:

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

Based on the above theoretical conditions our partition key design is ideal from a perspective of evenly distributiing the write work load around our cluster in terms of both performance and data storage.

Now it might be tempting to create a primary key that looks like the following:

```
	PRIMARY KEY (
		(QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 ReadingTimeStamp, StationId
	)
```

In this primary key the partition key consists of only the quantum function (we retain the StationId in the local key to maintain record uniqueness). All of the records written to the table using this partition key will hash to the same partition for a one day time period (144,000,000 records per day per partition vs 1,440 per partition in the first example). Using this quantum only partition key gives you the ability to query across weather stations (more on querying later on in this document) but will have a negative impact on performance as writes for the day won't be distributed equally across all five nodes. Based on our previous example numbers above we would expect to see writes to only 3 out of 5 nodes in the cluster (with the standard replication factor of 3) with each of those nodes handling 1,667 writes per second vs 1000 writes per second (an increase of 67% on each of those three nodes).

In review, the key difference in the design pattern of the two partition key examples above is that:

* The first example writes data to 100,000 partitions in parallel that are equally distributed around the cluster
* The second example writes data to 1 partition and writes are not equally distributed around the cluster

These two examples are very black and white but they give you the basic tools to reason about how the choice of partition key will affect write performance. In the next section we will describe how Riak TS executes queries and how the execution path affects read performance.


## How Riak TS Executes Queries

In the previous section we outlined the key factors behind maximizing write performance in a Riak TS cluster and how the choice of partition keys can make a significant impact on the number of writes per second a cluster can sustain. In this section we are going to outline how Riak TS executes queries and how the choice of partition keys affects read latency.

* The Riak TS client sends a query to the Riak TS cluster (via a load balancer ideally)

* The Riak TS node that handles the query (or coordinating node) splits the query into sub-queries along quantum boundaries

* Each sub-query is then sent to the corresponding virtual node that handles that quantum's matching partition (or partition key's where no quantum is specified)

* The virtual node **first** performs a range scan based on the partition keys in the ``` WHERE ``` clause

* The virtual node **then** applies a secondary filter on any additional fields included in the ``` WHERE ``` clause

* Each virtual node queried returns its piece of the query result set to the coordinating node

* The coordinating node assembles the separate result sets into one unified product and returns the results to the client that sent the query


Based on this query execution pattern you should design your partition key keeping in mind the following rules of thumb:

* Querying across fewer quanta is better in terms of performance. When possible you should use partition keys that limit the number of quanta you need to span in queries.

* Queries that **only** require partition keys in their ``` WHERE ``` clauses will be faster than queries that add non-partition-key columns to the ``` WHERE ``` clause since non key fields require a second level of filtering **after** the virtual node perfoms the intitial range scan on a partition.


## Why Use Quantums At All

As we noted in the [Partition Key](Data Modeling Basics.md#partition-key) section of the [Data Modeling Basics](Data Modeling Basics.md) document the quantum function is an optional part of the partition key.



---

 **Previous**: [How Partition Keys Work](How Partition Keys Work.md) | **Next**: 
 