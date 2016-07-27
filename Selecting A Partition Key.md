# [Riak TS](README.md) - Selecting A Partition Key

Choosing the right partition key for your table is an important piece of ensuring both the performance of the cluster and queriability of your data. In [Data Modeling Basics](Data Modeling Basics.md) and [How Partition Keys Work](How Partition Keys Work.md) we introduced you to the basics of Riak TS table schemas and how the partition key shapes the distribution of data within a cluster. In this section we are going expand upon the theoretical knowledge with practical advice on choosing partition keys including:

* [The Importance of Partition Keys in a Query](#the-importance-of-partition-keys-in-a-query) 

## The Importance of Partition Keys in a Query

When Riak TS executes a query it splits the query into sub-queries along quantum boundaries. Each sub-query is then sent to the corresponding virtual node that handles that quantum's matching partition. At the partition level the virtual node performs a range scan based on the partition keys in the ``` WHERE ``` clause and then applies a secondary filter on non partition key fields.



As noted in the [How Partition Keys Work](How Partition Keys Work.md) section querying across fewer quanta is better in terms for performance.




---

 **Previous**: [How Partition Keys Work](How Partition Keys Work.md) | **Next**: 