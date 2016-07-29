# [Riak TS](README.md) - Selecting A Partition Key

Choosing the right partition key for your table is an important piece of ensuring both the performance of the cluster and queriability of your data. In [Data Modeling Basics](Data Modeling Basics.md) and [How Partition Keys Work](How Partition Keys Work.md) we introduced you to the basics of Riak TS table schemas and how the partition key shapes the distribution of data within a cluster. In this section we are going expand upon the theoretical knowledge with practical advice on choosing partition keys including:

* [How Riak TS Executes Queries](#how-riak-ts-executes-queries) 
* [Why Use Quantums At All](#why-use-quantums-at-all)

## How Riak TS Executes Queries

When Riak TS executes a query it splits the query into sub-queries along quantum boundaries. Each sub-query is then sent to the corresponding virtual node that handles that quantum's matching partition. At the partition level the virtual node first performs a range scan based on the partition keys in the ``` WHERE ``` clause _and then_ applies a secondary filter on non-partition key fields.

Based on this query execution pattern you should design your partition key keeping in mind the following rules of thumb:

* Querying across fewer quanta is better in terms of performance so, when possilbe, you should use partition keys that limit the number of quanta you need to span in queries.

* Queries that _only_ require partition keys in their ``` WHERE ``` clauses will be faster than queries that add non-partition keys to the ``` WHERE ``` clause since non-partition keys require a second level of filtering _after_ the virtual node perfoms the intitial range scan on a partition.

## Why Use Quantums At All



---

 **Previous**: [How Partition Keys Work](How Partition Keys Work.md) | **Next**: 