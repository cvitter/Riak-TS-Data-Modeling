# [Riak TS](README.md) - How Partition Keys Work

Riak TS is a distributed NoSQL database that is designed to scale to handle massive data sets and high volumes of read and write operations. Riak TS achieves scalability by being able to seamlessly distribute its data and workloads across many individual nodes (physical or virtual) in a cluster. Of course if data is distributed across many machines than the database needs to know how to address the data for reads and writes. In this section we are going to describe how Riak TS partitions data and how this influences the selection of partition keys.



## Partitions, AKA The Ring

Riak TS divides its dataset into partitions. By default Riak TS has 64 partitions however this number is configurable and can range from a minimum of 8 to a maximum 1024 (in powers of 2 e.g.: 8, 16, 32, 64, 128, 256, 512, 1024). The number of partition is specified in the ``` riak.conf ``` in the ``` etc ``` directory. The following lines from the ``` riak.conf ``` contain the ``` ring_size ``` parameter and basic information about setting the parameter:

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

**Note**: By default ``` ring_size ``` is commented out but defaults to 64. To set the value uncomment the parameter and update its value.




Riak TS attains scalability via its ability to span multiple physical (or virtual) machines referred to as nodes.