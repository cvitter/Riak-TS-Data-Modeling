# [Riak TS](README.md) - Why Use Quantums At All

As we noted in the [Partition Key](Data Modeling Basics.md#partition-key) section of the [Data Modeling Basics](Data Modeling Basics.md) document the quantum function is an optional part of the partition key. So far all of the examples we have explored have used the quantum function to distribute and query data based on a range of time but there are a subset of non-time based use cases that are also ideal candidates for Riak TS. One simple example that demonstrates the feasibility of non-time based use cases in Riak TS is the ecommerce shopping cart. In the following section we will talk through the implementation of a simple shopping cart example in Riak TS.


## The Riak TS Shopping Cart

In the following example we are going to create a table in Riak TS that supports very basic ecommerce shopping cart functionality (I am sure a real world shopping cart is more complex) including capturing:

* The ID of the item added to the cart
* The quantity of the item added
* The unit cost of the item (at the time it was added to the cart)
* And the time that the item was added to the cart

The DDL we will use is:

```
CREATE TABLE ShoppingCartItem 
(
	CartId				VARCHAR		NOT NULL,
	ItemId				VARCHAR		NOT NULL,
	ItemQuantity		SINT64		NOT NULL,
	UnitCost			DOUBLE		NOT NULL,
	ItemAdded			TIMESTAMP	NOT NULL,
	PRIMARY KEY (
		(CartId),
		 CartId, ItemId
	)		
);
```

The following single line version of the DDL above can be copied and pasted directly into ``` riak-shell ```:

```
CREATE TABLE ShoppingCartItem (CartId VARCHAR NOT NULL, ItemId VARCHAR NOT NULL, ItemQuantity SINT64 NOT NULL, UnitCost DOUBLE NOT NULL, ItemAdded TIMESTAMP NOT NULL, PRIMARY KEY ((CartId), CartId, ItemId));
```

Once the ``` CREATE TABLE ``` statement has executed you can use the ``` DESCRIBE ``` command to review the table that we created:

```
riak-shell(2)>DESCRIBE ShoppingCartItem;
+------------+---------+-------+-----------+---------+--------+----+
|   Column   |  Type   |Is Null|Primary Key|Local Key|Interval|Unit|
+------------+---------+-------+-----------+---------+--------+----+
|   CartId   | varchar | false |     1     |    1    |        |    |
|   ItemId   | varchar | false |           |    2    |        |    |
|ItemQuantity| sint64  | false |           |         |        |    |
|  UnitCost  | double  | false |           |         |        |    |
| ItemAdded  |timestamp| false |           |         |        |    |
+------------+---------+-------+-----------+---------+--------+----+
```


```
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Shirt0001', 1, 12.25, '2016-12-16 15:43:22');
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Socks0001', 4, 3.75, '2016-12-16 15:47:02');
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Underwear0001', 4, 5.25, '2016-12-16 15:52:31');
```



```
SELECT * FROM ShoppingCartItem WHERE CartId = 'ShoppingCart0001';
```


```
+----------------+-------------+------------+--------------------------+--------------------+
|     CartId     |   ItemId    |ItemQuantity|         UnitCost         |     ItemAdded      |
+----------------+-------------+------------+--------------------------+--------------------+
|ShoppingCart0001|  Shirt0001  |     1      |1.22500000000000000000e+01|2016-12-16T15:43:22Z|
|ShoppingCart0001|  Socks0001  |     4      |3.75000000000000000000e+00|2016-12-16T15:47:02Z|
|ShoppingCart0001|Underwear0001|     4      |5.25000000000000000000e+00|2016-12-16T15:52:31Z|
+----------------+-------------+------------+--------------------------+--------------------+
```


---

 **Previous**: [Selecting A Partition Key](Selecting A Partition Key.md) | **Next**: