# [Riak TS](README.md) - Selecting A Partition Key

In [Data Modeling Basics](Data Modeling Basics.md) and [How Partition Keys Work](How Partition Keys Work.md) we introduced you to the basics of Riak TS table schemas and how the partition key shapes the distribution of data within a cluster. In this section we are going to cover how the choice of partition keys affects the performance of your Riak TS cluster (both in terms of writes and reads per second) and the queriability of the data stored in your cluster.  

* [Optimizing Partition Keys for Write Performance](#optimizing-partition-keys-for-write-performance) 
* [How Riak TS Executes Queries](#how-riak-ts-executes-queries) 
* [Why Use Quantums At All](#why-use-quantums-at-all)

## Optimizing Partition Keys for Write Performance

The choice of partition key affects Riak TS's write performance in the same way that the key affects the distribution of data around a cluster. Riak TS distributes the data and the write workload around a cluster using the partition key. In a high write workload environment the choice of partition key can be a significant factor in determining how many writes per second a cluster can sustain. Consider again the example table that we created in the [Data Modeling Basics](Data Modeling Basics.md) section with the following primary key:

```
	PRIMARY KEY (
		(StationId, QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 StationId, ReadingTimeStamp
	)
```

The partition key in this example specifies that the combination of the StationId column and ReadingTimeStamp column will hash to one partition (quantum) for a twenty-four hour period. That is an average of 1440 writes per day per weather station.

What if we were collecting data for 100,000 weather stations?


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
 