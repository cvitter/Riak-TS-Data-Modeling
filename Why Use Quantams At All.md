# [Riak TS](README.md) - Why Use Quantums At All

As we noted in the [Partition Key](Data Modeling Basics.md#partition-key) section of the [Data Modeling Basics](Data Modeling Basics.md) document the quantum function is an optional part of the partition key. So far all of the examples we have explored have used the quantum function to distribute and query data based on a range of time but there are a subset of non-time based use cases that are also ideal candidates for Riak TS. One simple example that demonstrates the feasibility of non-time based use cases in Riak TS is the ecommerce shopping cart. In the following section we will talk through the implementation of a simple shopping cart example in Riak TS.


## The Riak TS Shopping Cart



```
CREATE TABLE ShoppingCartItem 
(
	CartId				VARCHAR		NOT NULL,
	ItemId				VARCHAR		NOT NULL,
	ItemAdded			TIMESTAMP	NOT NULL,
	UnitCost			DOUBLE,
	PRIMARY KEY (
		(CartId),
		 CartId, ItemId
	)		
);
```



---

 **Previous**: [Selecting A Partition Key](Selecting A Partition Key.md) | **Next**: