# [Riak TS](README.md) - Selecting A Partition Key

In [Data Modeling Basics](Data Modeling Basics.md) and [How Partition Keys Work](How Partition Keys Work.md) we introduced you to the basics of Riak TS table schemas and how the partition key shapes the distribution of data within a cluster. In this section we are going to cover how the choice of partition keys affects the performance of your Riak TS cluster (both in terms of writes and reads per second) and the queriability of the data stored in your cluster.  

* [Optimizing Partition Keys for Write Performance](#optimizing-partition-keys-for-write-performance) 
* [How Riak TS Executes Queries](#how-riak-ts-executes-queries) 
* [How Big Should I Make My Quanta?](#how-big-should-i-make-my-quanta)
* [Should I Just Use a Quantum Function?](#should-i-just-use-a-quantum-function)
* [What If I Don't Care What Time It Is?](#what-if-i-dont-care-what-time-it-is)

**Important Note**: Every use case is different. In this document I am going to give a lot of general advice on how to design your Riak TS tables keeping read and write performance in mind _however_ I highly recommend that you benchmark your table designs for suitability before putting them into production.


## Optimizing Partition Keys for Write Performance

Write performance in a Riak TS is affected by a number of factors including:

* The number of nodes in a cluster
* The hardware specifications of physical nodes including CPU number and speed, RAM, drive performance, network card speed, etc.
* The load balancer directing write requests to nodes (and the algorithm used by the load balancer to direct traffic)
* The size of the object being written
* And the partition key

The key to maximizing performance in a cluster is to ensure that all of the nodes in the cluster are able to equally share in the write workload. From a schema design perspective your choice of partition key is the number one thing affecting write performance. In this section we are going to talk about things to consider when designing partition keys that will help you limit performance bottle necks. Let's start by taking another look at the example table that we created in the [Data Modeling Basics](Data Modeling Basics.md) section with the following primary key:

```
	PRIMARY KEY (
		(StationId, QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 StationId, ReadingTimeStamp
	)
```

The partition key in this example specifies that the combination of the StationId column and ReadingTimeStamp column will hash to one partition (quantum) for a twenty-four hour period. To understand how this affects performance lets consider the following hypothetical conditions:

* There are 100,000 weather stations reporting data
* Each weather station reports once a minute (1440 times a day)
* In a 5 node cluster each node would average 1000 writes per second (see Simple Math below)

**Simple Math**:
```
	1. 100,000 updates a minute / 60 seconds = 1666.6...
	2. 1666.6 * 3 (the default n_val or Riak TS replication factor) = 5000
	3. 5000 / 5 (the number of nodes in our theoretical cluster)  = 1000 writes per second, per node
```

Based on the above theoretical conditions our partition key design is ideal from a perspective of evenly distributing the write work load around our cluster in terms of both performance and data storage.

Now it might be tempting to create a primary key that looks like the following:

```
	PRIMARY KEY (
		(QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 ReadingTimeStamp, StationId
	)
```

In this primary key the partition key contains only the quantum function (we retain the StationId in the local key to maintain record uniqueness). All of the records written to the table using this partition key will hash to the same partition for a one day time period (144,000,000 records per day per partition vs 1,440 per partition in the first example). Using this quantum only partition key gives you the ability to query across weather stations (more on querying later on in this document) but will have a negative impact on performance as writes for the day won't be distributed equally across all five nodes. Based on our previous example numbers above we would expect to see writes to only 3 out of 5 nodes in the cluster (with the standard replication factor of 3) with each of those nodes handling 1,667 writes per second vs 1000 writes per second (an increase of 67% on each of those three nodes).

In review, the key difference in the design pattern of the two partition key examples above is that:

* The first example writes data to 100,000 partitions in parallel that are equally distributed around the cluster
* The second example writes data to 1 partition and writes are not equally distributed around the cluster

These two examples are very black and white but they give you the basic tools to reason about how the choice of partition key will affect write performance. In the next section we will describe how Riak TS executes queries and how the execution path affects read performance.


## How Riak TS Executes Queries

In the previous section we outlined the key factors behind maximizing write performance in a Riak TS cluster and how the choice of partition keys can make a significant impact on the number of writes per second a cluster can sustain. In this section we are going to outline how Riak TS executes queries and how the choice of partition keys affects read latency.

At a high level queries are executed as follows:

* The Riak TS client sends a query to the Riak TS cluster (via a load balancer ideally)

* The Riak TS node that handles the query (or coordinating node) splits the query into sub-queries along quantum boundaries

* Each sub-query is then sent to the corresponding virtual node that handles that quantum's matching partition (or partition key's where no quantum is specified)

* The virtual node **first** performs a range scan based on the partition keys in the ``` WHERE ``` clause

* The virtual node **then** applies a secondary filter on any additional fields included in the ``` WHERE ``` clause

* Each virtual node queried returns its piece of the query result set to the coordinating node

* The coordinating node assembles the separate result sets into one unified product and returns the results to the client that sent the query


Based on this query execution pattern you should design your partition key keeping in mind the following rules of thumb:

* Querying across fewer quanta is better in terms of performance. When possible you should use partition keys that limit the number of quanta you need to span in queries.

* Queries that **only** require partition keys in their ``` WHERE ``` clauses will be faster than queries that add non-partition-key columns to the ``` WHERE ``` clause since non key fields require a second level of filtering **after** the virtual node performs the initial range scan on a partition.


## How Big Should I Make My Quanta?

At this point you are probably asking yourself, "How big should I make my quanta?" While there isn't one simple rule of thumb for choosing the "perfect" quantum size the main consideration to keep in mind is the standard time range that your application will be querying. If your application typically requests all of the weather station updates for the last two weeks than the one day quantum we created in our ``` WeatherStationData ``` table example will be too small. In fact the quantum size we created in our sample table of one day is pretty small. Increasing the size of the quantum, even significantly, shouldn't make a difference in write performance since we have have also partitioned our data on ``` StationId ``` ensuring that our writes and data will still be distributed evenly around the cluster.

## Should I Just Use a Quantum Function?

In [Optimizing Partition Keys for Write Performance](#optimizing-partition-keys-for-write-performance) we provided an example of a partition key that dropped the ``` StationId ``` and used only a quantum function:

```
	PRIMARY KEY (
		(QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 ReadingTimeStamp, StationId
	)
```

The advantage to creating a quantum only partition key is that it allows you to issue queries that return data for more than one weather station at a time. At first blush the temptation to follow this design pattern will be strong but let's consider the type of queries that you want to run against your data set. The following is a list of common questions you might want to ask of your weather station data:

* What is the current temperature at weather station X?

* What was the high/low temperature at weather station X yesterday?

* What was the average temperature at weather station X on July 15th?

* Which weather station had the highest/lowest temperature on December 25th?

While there are questions that you will want to ask that require a review of all of the data for all of the stations, generally speaking in this use case the questions are weather station specific. Let's consider a simple query that returns the average temperature for one weather station for one day:

```
SELECT AVG(Temperature) FROM WeatherStationData WHERE
     ReadingTimeStamp >= '2016-05-11 00:00:00' AND 
     ReadingTimeStamp <= '2014-05-11 11:59:59' AND
     StationId = 'Station-1001';
```

When the query is executed Riak TS will:

1. Perform a range scan on the partition (or partitions) storing the data we a requesting

1. Filter the result set on ``` StationId ```

1. ``` AVG ``` the ``` Temperature ``` value

Remember that each partition in our example will contain 144,000,000 records. Filtering down from 140,000,000 records to 1440 records for the one weather station we are interested before we even average the data could cause a significant performance issue. Querying across multiple quanta will increase the potential performance problems. That said, if your testing shows acceptable performance at scale feel free to ignore my advice not to go this route.


## What If I Don't Care What Time It Is?

Up until now we have focused our data modeling discussion on data sets where time is an important factor. In the next section [Why Use Quanta At All?](Why Use Quanta At All.md) we are going to step away the quantum function and look at how Riak TS can be used to solve non-time based data problems. If the you don't care about non-time based use cases please feel free to stop here!

---

 **Previous**: [How Partition Keys Work](How Partition Keys Work.md) | **Next**: [Why Use Quanta At All?](Why Use Quanta At All.md)
 