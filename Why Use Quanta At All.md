# [Riak TS](README.md) - Why Use Quanta At All?

As we noted in the [Partition Key](Data%20Modeling%20Basics.md#partition-key) section of the [Data Modeling Basics](Data%20Modeling%20Basics.md) document the quantum function is an optional part of the partition key. So far all of the examples we have explored have used the quantum function to distribute and query data based on a range of time but there are a subset of non-time based use cases that are also ideal candidates for Riak TS. One simple example that demonstrates the feasibility of non-time based use cases in Riak TS is the e-commerce shopping cart. In the following section we will walk through the implementation of a simple shopping cart example in Riak TS.


## The Riak TS Shopping Cart

In the following example we are going to create a table in Riak TS that supports very basic e-commerce shopping cart functionality including capturing:

* The ID of the item added to the cart
* The quantity of the item added
* The unit cost of the item (at the time it was added to the cart)
* And the time that the item was added to the cart

The DDL we will use for our shopping cart table is:

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

Once the ``` CREATE TABLE ``` statement has executed you can use the ``` DESCRIBE ``` command to review the table that you created:

```
riak-shell(2)>DESCRIBE ShoppingCartItem;
+------------+---------+--------+-------------+---------+--------+----+----------+
|   Column   |  Type   |Nullable|Partition Key|Local Key|Interval|Unit|Sort Order|
+------------+---------+--------+-------------+---------+--------+----+----------+
|   CartId   | varchar | false  |      1      |    1    |        |    |          |
|   ItemId   | varchar | false  |             |    2    |        |    |          |
|ItemQuantity| sint64  | false  |             |         |        |    |          |
|  UnitCost  | double  | false  |             |         |        |    |          |
| ItemAdded  |timestamp| false  |             |         |        |    |          |
+------------+---------+--------+-------------+---------+--------+----+----------+
```

In our example table the partition key consists of the ``` CartId ``` column alone. This means that every record we write to the table that shares the same ``` CartId ``` will be written to the same partition in our cluster (**Note**: Record uniqueness is established at the local key level via the combination of ``` CartId ``` and ``` ItemId ``` columns). Although different carts can have widely different number of line items associated with them the writes and reads will be distributed evenly around the cluster as the number of line items should average out over a large number of shopping carts.

With our example table created we can now add a few records to the table. Use the following SQL ``` INSERT ``` statements to insert three line items into a shopping cart:

```
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Shirt0001', 1, 12.25, '2016-12-16 15:43:22');
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Socks0001', 4, 3.75, '2016-12-16 15:47:02');
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Underwear0001', 4, 5.25, '2016-12-16 15:52:31');
```

The three line items that were inserted above all share the same ``` CartId ``` value meaning that we can use the following ``` SELECT ``` statement to retrieve the three records:

```
riak-shell(2)>SELECT * FROM ShoppingCartItem WHERE CartId = 'ShoppingCart0001';
+----------------+-------------+------------+--------+--------------------+
|     CartId     |   ItemId    |ItemQuantity|UnitCost|     ItemAdded      |
+----------------+-------------+------------+--------+--------------------+
|ShoppingCart0001|  Shirt0001  |     1      | 12.25  |2016-12-16T15:43:22Z|
|ShoppingCart0001|  Socks0001  |     4      |  3.75  |2016-12-16T15:47:02Z|
|ShoppingCart0001|Underwear0001|     4      |  5.25  |2016-12-16T15:52:31Z|
+----------------+-------------+------------+--------+--------------------+
```

Let's add three more items to our shopping cart:

```
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'Apple0001', 8, 2.50, '2016-12-16 15:53:37');
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'ZebraHoodie0001', 1, 25.55, '2016-12-16 15:58:04');
INSERT INTO ShoppingCartItem VALUES('ShoppingCart0001', 'TennisRacket0001', 1, 65.95, '2016-12-16 16:05:54');
```

and then execute our ``` SELECT ``` statement again:

```
riak-shell(17)>SELECT * FROM ShoppingCartItem WHERE CartId = 'ShoppingCart0001';                                            
+----------------+----------------+------------+--------+--------------------+
|     CartId     |     ItemId     |ItemQuantity|UnitCost|     ItemAdded      |
+----------------+----------------+------------+--------+--------------------+
|ShoppingCart0001|   Apple0001    |     8      |  2.5   |2016-12-16T15:53:37Z|
|ShoppingCart0001|   Shirt0001    |     1      | 12.25  |2016-12-16T15:43:22Z|
|ShoppingCart0001|   Socks0001    |     4      |  3.75  |2016-12-16T15:47:02Z|
|ShoppingCart0001|TennisRacket0001|     1      | 65.95  |2016-12-16T16:05:54Z|
|ShoppingCart0001| Underwear0001  |     4      |  5.25  |2016-12-16T15:52:31Z|
|ShoppingCart0001|ZebraHoodie0001 |     1      | 25.55  |2016-12-16T15:58:04Z|
+----------------+----------------+------------+--------+--------------------+
```

Notice that the output of the select statement is ordered by ``` CartId ``` and ``` ItemId ``` columns (the local key) which is how the records are physically sorted on disk. If we want to change the order of items in our result set we can use ``` ORDER BY ``` as demonstrated in the query below where we sort the result set to show our items in descending order of when they were added:

```
riak-shell(18)>SELECT * FROM ShoppingCartItem WHERE CartId = 'ShoppingCart0001' ORDER BY ItemAdded DESC;
+----------------+----------------+------------+--------+--------------------+
|     CartId     |     ItemId     |ItemQuantity|UnitCost|     ItemAdded      |
+----------------+----------------+------------+--------+--------------------+
|ShoppingCart0001|TennisRacket0001|     1      | 65.95  |2016-12-16T16:05:54Z|
|ShoppingCart0001|ZebraHoodie0001 |     1      | 25.55  |2016-12-16T15:58:04Z|
|ShoppingCart0001|   Apple0001    |     8      |  2.5   |2016-12-16T15:53:37Z|
|ShoppingCart0001| Underwear0001  |     4      |  5.25  |2016-12-16T15:52:31Z|
|ShoppingCart0001|   Socks0001    |     4      |  3.75  |2016-12-16T15:47:02Z|
|ShoppingCart0001|   Shirt0001    |     1      | 12.25  |2016-12-16T15:43:22Z|
+----------------+----------------+------------+--------+--------------------+
```

In addition to the basic ``` SELECT ``` statement you can use aggregates like ``` COUNT ``` to count the total number of line items in the shopping cart or ``` SUM ``` to count the total number of items in the cart. For example:

```
riak-shell(20)>SELECT COUNT(*) FROM ShoppingCartItem WHERE CartId = 'ShoppingCart0001';
+--------+    
|COUNT(*)|
+--------+
|   6    |
+--------+

riak-shell(21)>SELECT SUM(ItemQuantity) FROM ShoppingCartItem WHERE CartId = 'ShoppingCart0001';
+-----------------+
|SUM(ItemQuantity)|
+-----------------+
|       19        |
+-----------------+
```

If we had added a column to our table that stored the total cost of the items added (e.g. ItemQuantity * UnitCost) it would be very easy to get the total dollar value of the shopping cart using ``` SUM ```.


## Other Non-Time Based Use Cases for Riak TS

In the shopping cart example above we demonstrated how easy it would be to use Riak TS for a one non-time based use case. The basic requirements of the use case include:

* Data grouped together by a simple identifier for easy access -  the ability to create and query an unbounded set of objects **easily**

* High availability - people won't buy anything if their shopping carts vanish (**Note***: The enterprise version of Riak TS adds support for multi-cluster and multi-data center replication further improving Riak TS's high availability profile)

* Low latency - people tend to abandon slow web sites

* Even distribution of data - evenly utilizes the cluster hardware in terms of both storage and performance

Some similar use cases that share the same requirements include:

* Testing - Capture and store test answers

* Pick/part lists - A list of items to pack to fulfill an order

* To Do lists - A list of tasks to complete

This grouping of use case types share a common key pattern illustrated in the DDL below:

```
CREATE TABLE YourCoolUseCase 
(
	GroupId				VARCHAR		NOT NULL,
	ItemId				VARCHAR		NOT NULL,
	...
	PRIMARY KEY (
		(GroupId),
		 GroupId, ItemId
	)		
);
```

Don't forget that the columns used for the partition and local keys don't have to be of the VARCHAR data type and that the values use in the local key affect how the the data written is sorted (and returned in queries).

If you find any creative non-time based use cases you would like to share please feel free to create an issue: https://github.com/cvitter/Riak-TS-Data-Modeling/issues.


---

 **Previous**: [Selecting A Partition Key](Selecting%20A%20Partition%20Key.md) | **Next**: