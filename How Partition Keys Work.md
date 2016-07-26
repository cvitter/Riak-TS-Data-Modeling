# [Riak TS](README.md) - How Partition Keys Work

Riak TS is a distributed NoSQL database that is designed to scale to handle massive data sets and high volumes of read and write operations. Riak TS achieves this scalability by being able to seamlessly distribute its data and workloads across many individual nodes (physical or virtual) in a cluster. Of course if data is distributed across many machines than the database needs to know how to address the data for reads and writes. In this section we are going to describe how Riak TS partitions data and how this influences the selection of partition keys:

* [Partitions](#partitions)
* [The Ring](#the-ring)
* [Consistant Hashing](#consistent-hashing)
* [The Partition Key](#the-partition-key)


## Partitions

Riak TS divides its dataset into partitions. By default Riak TS has 64 partitions however this number is configurable and can range from a minimum of 8 partitions up to a maximum 1024 in powers of 2 (e.g.: 8, 16, 32, 64, 128, 256, 512, 1024). The number of partitions is specified in the ``` riak.conf ``` file in the ``` etc ``` directory. The following lines from the ``` riak.conf ``` contain the ``` ring_size ``` parameter and basic information about setting the parameter:

```
## Number of partitions in the cluster (only valid when first
## creating the cluster). Must be a power of 2, minimum 8 and maximum
## 1024.
## 
## Default: 64
## 
## Acceptable values:
##   - an integer
## ring_size = 64
```

By default ``` ring_size ``` is commented out but defaults to 64. To set the value uncomment the parameter and update its value.

If you have a single Riak TS nodes all of the partitions will live on that single node. If you have a cluster of Riak TS nodes than the partitions are distributed equally around the cluster. For example, if you have 8 nodes and 64 partitions then each node will have 8 partitions.

**Important Note:** The number of partitions has a direct impact on the scalability and performance of your cluster. Before setting up a production cluster please review the ring size planning guidance found here: http://docs.basho.com/riak/kv/2.1.4/setup/planning/cluster-capacity/#ring-size-number-of-partitions.


## The Ring

In order to write data to partitions in Riak TS those partitions need to have addresses. Riak TS's "Ring" is an a 160-bit integer address space. This means that each Riak TS cluster can have:

``` 0 - 2^160 ```

or

``` 0 - 1,461,501,637,330,902,918,203,684,832,716,283,019,655,932,542,976 ```

 keys that are shared equally across each partition in the cluster. 

 Each partition will have the following number of keys assigned to it (where N equals the number of partitions):

``` 2^160 / N ```


The following graphic provides a simplified illustration of how Riak TS assigns key ranges to partitions:

![alt text](Images/riak-ring.png)



## Consistent Hashing

How does Riak TS know which partition a key/value pair should be written too or read from? It is quite simple really. Riak TS uses a consistent hashing (https://en.wikipedia.org/wiki/Consistent_hashing) function based on the SHA-1 algorith (https://en.wikipedia.org/wiki/SHA-1) to convert the key into a number. Since every node in a Riak TS cluster has an updated copy of "the ring" (kept up to date via gossip protocol) every node in the cluster is capable of serving reads and writes using this simple consistent hashing mechanism.


## The Partition Key

