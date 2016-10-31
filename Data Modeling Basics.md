# [Riak TS](README.md) - Data Modeling Basics

**Riak TS** stores data in "**tables**", "**columns**", and "**rows**" similar to a traditional relational database however TS has some unique architectural features that make data modeling a different challenge. In this section we are going to introduce the basics of data modeling with Riak TS including:

* [How tables are created in Riak TS](#creating-a-table)
* [Table naming requirements](#table-names)
* [Columns and supported data types](#columns)
* [Primary Keys](#the-primary-key)
* [Reading and writing data with Riak TS](#reading-and-writing-data-with-riak-ts)
* [And a brief discussion of Riak TS's table architecture](#riak-ts-table-architecture)

## Creating a Table

Before writing data to Riak TS you need to create a table. Tables are created using a DDL (Data Definition Language - https://en.wikipedia.org/wiki/Data_definition_language) which is based on the standard SQL DDL. 

**Important Note About Table Creation**: Riak TS tables cannot be altered or deleted after they have been created so it is very important to carefully plan the table's schema before creating the table.

The following sample DDL creates a table called WeatherStationData that has six columns:

```
CREATE TABLE WeatherStationData 
(
	StationId			VARCHAR		NOT NULL,
	ReadingTimeStamp	TIMESTAMP	NOT NULL,
	Temperature			SINT64,
	Humidity			DOUBLE,
	WindSpeed			DOUBLE,
	WindDirection		DOUBLE,
	PRIMARY KEY (
		(StationId, QUANTUM(ReadingTimeStamp, 1, 'd') ),
		 StationId, ReadingTimeStamp
	)		
)
```

**Note**: In the DDL example above all of the keywords are written in UPPERCASE because I believe it makes the DDL more readable. In practice keywords can be in any case you wish to use as **they are not case sensitive**.

Once you have a DDL like the example above there are three ways that you can send the DDL to Riak TS to create your table:

* Use Riak TS's REPL (read-eval-print-loop) application **riak-shell** (http://docs.basho.com/riak/ts/latest/using/riakshell/)  to execute the DDL;
* Use one of Riak TS's client libraries (Java, Python, Erlang, Node.js, .Net, Go, Ruby, PHP - see the following page for more information on Riak TS's client libraries: http://docs.basho.com/riak/ts/latest/developing/) to execute the DDL;
* Or use the riak-admin command tool (see the following page for details on how to do this: http://docs.basho.com/riak/ts/latest/using/creating-activating/#riak-admin).

For most of this documentation we will use riak-shell to execute example code. If you have already installed Riak TS (and it is running) let's go ahead and create the WeatherStationData table using riak-shell.

From within your Riak TS root directory launch riak-shell using:

``` > bin/riak-shell ```

Riak-shell will output the following text when it launches:
```
Erlang R16B02_basho10 (erts-5.10.3) [source] [64-bit] [smp:4:4] [async-threads:10] [kernel-poll:false] [frame-pointer] [dtrace]

version "riak_shell 0.9/sql compiler 2", use 'quit;' or 'q;' to exit or 'help;' for help
Connected...
riak-shell(1)>
```

Copy and paste the following simplified, single line version of the CREATE TABLE statement (riak-shell currently doesn't support multi-line input) in to the shell and hitting enter:

```
CREATE TABLE WeatherStationData (StationId VARCHAR NOT NULL, ReadingTimeStamp TIMESTAMP NOT NULL, Temperature SINT64, Humidity DOUBLE, WindSpeed DOUBLE, WindDirection DOUBLE, PRIMARY KEY ((StationId, QUANTUM(ReadingTimeStamp, 1, 'd')), StationId, ReadingTimeStamp));
```

Riak TS doesn't send "success" messages when tables are created successfully. If Riak TS is able to add the new table successfully you should simply see a new command prompt upon completion. 

As noted above, once you have created a table you cannot alter or delete the table. If you run the table create statement a second time the database should output the following error message:

``` Error (1014): Failed to create table WeatherStationData: already_active ```

There are two ways within riak-shell to verify that your table has been created successfully (other than the absence of an error message). The first method is to use the ``` SHOW TABLES ``` command as illustrated below:

```
riak-shell(2)>SHOW TABLES;
+------------------+
|      Table       |
+------------------+
|WeatherStationData|
+------------------+
```

The second method is to use the ```DESCRIBE``` command to output your table's schema. Within riak-shell use the following command to output the schema for the WeatherStationData table:

``` DESCRIBE WeatherStationData; ```

The DESCRIBE command should return the following output for the WeatherStationData table:

```
+----------------+---------+-------+-----------+---------+--------+----+
|     Column     |  Type   |Is Null|Primary Key|Local Key|Interval|Unit|
+----------------+---------+-------+-----------+---------+--------+----+
|   StationId    | varchar | false |     1     |    1    |        |    |
|ReadingTimeStamp|timestamp| false |     2     |    2    |   1    | d  |
|  Temperature   | sint64  | true  |           |         |        |    |
|    Humidity    | double  | true  |           |         |        |    |
|   WindSpeed    | double  | true  |           |         |        |    |
| WindDirection  | double  | true  |           |         |        |    |
+----------------+---------+-------+-----------+---------+--------+----+
```

Now that we have created the table within Riak TS lets take a deeper look at the individual components of our DDL starting with the table name.

## Table Names

In our example DDL we specified the name of our new Riak TS table using the following line:

```CREATE TABLE WeatherStationData```

When creating a new table keep in mind the following rules for table names:

1. Table names must be unique
1. Table names can only contain ASCII characters
1. Table names can contain spaces and special characters (!, @, #, $, %, ^, &, etc) only **if** the name is double quoted in the DDL, e.g.: ```CREATE TABLE "Weather Station Data"``` and ```CREATE TABLE "Weather@Station$Data!"```

## Columns

Column definitions within DDLs contain three parts: the column's name, the data type, and an optional NULL value constraint. Rules for column names are essentially the same as rules for table names:

1. Column names must be unique within the table
1. Column names can only contain ASCII characters
1. Column names can contain spaces and special characters (!, @, #, $, %, ^, &, etc) only **if** the name is double quoted in the DDL, e.g.: ```"Column Name``` and ```"Column_Name_!"```

The five data types currently supported by Riak TS are:

| Data Type           | Description                                      |
|---------------------|--------------------------------------------------|
| VARCHAR             | String content including Unicode. Can only be compared using strict equality. Use single quotes to delimit varchar strings. |
| TIMESTAMP           | Timestamps are integer values expressing UNIX epoch time in UTC in milliseconds. Zero is not a valid timestamp. |
| SINT64              | Signed 64-bit integer |
| DOUBLE              | This type does not comply with its IEEE specification: NaN (not a number) and INF (infinity) cannot be used.|
| BOOLEAN             | True or False, values accepted in any case |

By default columns allow NULL values. If you need to prevent NULL values from being saved in a column you can apply the ```NOT NULL``` constraint to a column definition, e.g.:

```StationId			VARCHAR		NOT NULL,```

**Important Note**: Columns used in the primary key must be defined with the ```NOT NULL``` constraint. See **[The Primary Key](#the-primary-key)** section below for more information.

## The Primary Key

The primary key specification for Riak TS is one of the areas in which the Riak TS DDL diverges from the standard SQL DDL, e.g.

```
CREATE TABLE employees (
    id            INTEGER       PRIMARY KEY,
    first_name    VARCHAR(50)   not null,
    ...
);
```

Notice in the standard SQL DDL above the id column is set as the primary key directly within the column definition. In Riak TS the primary key is defined within the DDL after the column definitions as illustrated below:

```
CREATE TABLE WeatherStationData 
(
    StationId           VARCHAR     NOT NULL,
    ReadingTimeStamp    TIMESTAMP   NOT NULL,
    ...
    PRIMARY KEY (
        (StationId, QUANTUM(ReadingTimeStamp, 1, 'd') ),
         StationId, ReadingTimeStamp
    )       
)
```

The primary key in Riak TS consists of two parts: A **Partition Key** and a **Local Key**. The partition key is used by Riak TS to determine which partition, or virtual node, in the Riak TS cluster a row of data should be written to/read from. The local key determines how the row is written to disk on its partition. The choice of key has a big impact on the performance of queries and how efficiently your cluster hardware is utilized so we will explore key selection in great depth in [How Partition Keys Work](How Partition Keys Work.md). In the remainder of this section we will review the basic rules associated with creating partition and local keys.

### Partition Key

The partition key consists of a list of one or more column names (from the table's DDL) and an optional quantum function. 

**Important Note**: If you include the quantum function in your partition key it *must* be the last element of the key.

In our example DDL the partition key consists of the ```StationId``` column and a quantum function that takes the ```ReadingTimeStamp``` column as the first of three parameters, e.g.:

``` (StationId, QUANTUM(ReadingTimeStamp, 1, 'd') ), ``` 

The quantum function is designed to allow Riak TS to colocate data in a cluster based on a range of time. The three parameters that the function takes include:

* The name of the column to base the range of time on (must be of the TIMESTAMP data type)
* Units of time expressed as a positive integer (must be greater than zero)
* The type of unit of time expressed as one of the following string values:

| Unit | Definition |
|------|------------|
| d    | Days       |
| h    | Hours      |
| m    | Minutes    |
| s    | Seconds    |


### Local Key

Riak TS uses the local key when writing data to disk to determine how data is sorted within the partition. The local key has to (at a minimum) have the same columns, in the same order, that are in the partition key. In our example DDL the local key is specified as:

``` StationId, ReadingTimeStamp ```

You can add additional columns to the local key if for example additional columns are required to ensure key uniqueness, e.g.:

``` StationId, ReadingTimeStamp, Temperature ```

as long as the additional columns are added after the columns also contained within the partition key. 

## Reading and Writing Data with Riak TS

Now that we understand the basics of data modeling with Riak TS, and have added our first table to the database, we can start inserting and reading data. In this section we are going to briefly cover the basics of using the SQL ```INSERT``` and ```SELECT``` commands with riak-shell.

### Insert

Inserting data into Riak TS can be done using standard SQL within the riak-shell or via native methods in the client libraries. In the following example we are going to insert a row into our new table using SQL entered via riak-shell. The following example SQL INSERT statement will create our first record:

```
INSERT INTO WeatherStationData
	(StationId, ReadingTimeStamp, Temperature, Humidity, WindSpeed, WindDirection)
VALUES
	('Station-1001', '2014-07-22 16:28:57', 52, 43.2, 2.5, 290.0);
```

Remembering that the current version or riak-shell doesn't support multi-line statements you can use the following streamlined version (it leave off the column names) to create your first record:

```
INSERT INTO WeatherStationData VALUES ('Station-1001', '2014-07-22 16:28:57', 52, 43.2, 2.5, 290.0);
```

If you cut and paste the SQL ```INSERT``` command above into riak-shell a new record should be added to the ```WeatherStationData``` table.

**Important Note**:  As of Riak TS version 1.4 you can write dates using the ISO 8061 data format (http://www.iso.org/iso/home/standards/iso8601.htm) as demonstrated in the SQL insert statement above.


### Select

Now that we have added a row to our ```WeatherStationData``` table we can verify that it was written by using the SQL ```SELECT``` command to retrieve the new record. When querying your data the following rules apply to the partition portion of the record's primary key:

* All columns in the partition key must be in the query's ```WHERE``` clause
* The quantized field (if there is one, ```ReadingTimeStamp``` in our example) must be included as a bounded range (e.g. ```ReadingTimeStamp >= 1469204877 AND ReadingTimeStamp <= 1469204977``` or ```ReadingTimeStamp >= '2014-07-22 12:00:00' AND ReadingTimeStamp <= '2014-07-22 20:00:00'```)
* All other partition columns must be included as exact matches (e.g. ```StationId = 'Station-1001'```)

The following SQL ```SELECT``` will select our newly created records from the database:

```
SELECT * FROM WeatherStationData WHERE
     StationId = 'Station-1001' AND
     ReadingTimeStamp >= '2014-07-22 12:00:00' AND 
     ReadingTimeStamp <= '2014-07-22 20:00:00';
```

Cut and paste the single line version of the SQL statement into riak-shell to test:

```
SELECT * FROM WeatherStationData WHERE StationId = 'Station-1001' AND ReadingTimeStamp >= '2014-07-22 12:00:00' AND ReadingTimeStamp <= '2014-07-22 20:00:00';
```

When the command has executed riak-shell should return the following output:

```
+------------+--------------------+-----------+--------------------------+--------------------------+--------------------------+
| StationId  |  ReadingTimeStamp  |Temperature|         Humidity         |        WindSpeed         |      WindDirection       |
+------------+--------------------+-----------+--------------------------+--------------------------+--------------------------+
|Station-1001|2014-07-22T16:28:57Z|    53     |4.32000000000000028422e+01|2.50000000000000000000e+00|2.89000000000000000000e+02|
+------------+--------------------+-----------+--------------------------+--------------------------+--------------------------+
```

## Riak TS Table Architecture

Although Riak TS features tables, columns, and rows it is important to remember that it was developed on top of Riak KV, a key/value database. In this section we are going to briefly discuss how how Riak TS's features map to the underlying key/value architecture.

### Tables and Columns

Riak TS tables map one-to-one to Riak KV bucket types (see Riak KV's documentation for more information on bucket types: http://docs.basho.com/riak/kv/latest/using/reference/bucket-types/). Creating a new table creates a new bucket type and a bucket within that bucket type with the table's name. You can use the riak-admin tool to view the server's bucket types. From the command line type the following command (if you haven't quit out or riak-shell yet type ```q;``` to exit):

``` 
> bin/riak-admin bucket-type list ```
  default (active)
  WeatherStationData (active)
```

The riak-admin tool doesn't provide a list of buckets within each bucket type however you can use Riak TS's HTTP interface to view the bucket created under the WeatherStationData bucket by entering the following URL into your Web browser (if you are running Riak TS locally, otherwise you will need to type the URL of the remote TS node):

``` http://127.0.0.1:8098/types/WeatherStationData/buckets?buckets=true ```

```
{
   buckets: [
      "WeatherStationData"
   ]
}
```

**Important Note**: Listing buckets is an expensive operation and shouldn't be done on production clusters. See the following documentation for more information: http://docs.basho.com/riak/kv/latest/developing/api/http/list-buckets/.

The riak-admin tool allows you to list all of the properties associated with a bucket type including the table's schema and quantum information as illustrated below:


```
> bin/riak-admin bucket-type status WeatherStationData
WeatherStationData is active

young_vclock: 20
w: quorum
small_vclock: 50
rw: quorum
r: quorum
pw: 0
precommit: []
pr: 0
postcommit: []
old_vclock: 86400
notfound_ok: true
n_val: 3
linkfun: {modfun,riak_kv_wm_link_walker,mapreduce_linkfun}
last_write_wins: false
dw: quorum
dvv_enabled: true
ddl_compiler_version: 2
ddl: {ddl_v1,<<"WeatherStationData">>,
     [{riak_field_v1,<<"StationId">>,1,varchar,false},
      {riak_field_v1,<<"ReadingTimeStamp">>,2,timestamp,false},
      {riak_field_v1,<<"Temperature">>,3,sint64,true},
      {riak_field_v1,<<"Humidity">>,4,double,true},
      {riak_field_v1,<<"WindSpeed">>,5,double,true},
      {riak_field_v1,<<"WindDirection">>,6,double,true}],
      {key_v1,[{param_v1,[<<"StationId">>]},
          {hash_fn_v1,riak_ql_quanta,quantum,
           [{param_v1,[<<"ReadingTimeStamp">>]},1,d],
            timestamp}]},
      {key_v1,[{param_v1,[<<"StationId">>]},
      {param_v1,[<<"ReadingTimeStamp">>]}]}}
chash_keyfun: {riak_core_util,chash_std_keyfun}
big_vclock: 50
basic_quorum: false
allow_mult: true
write_once: true
active: true
claimant: 'riak@127.0.0.1'
```


### Rows

A Riak TS row maps to a key/value pair. The value stored in the database contains both the values associated with each of the row's columns and the column names. If we run the SQL query from above using the HTTP API and curl, e.g.:

``` 
> curl -XPOST http://127.0.0.1:8098/ts/v1/query --data "SELECT * FROM WeatherStationData WHERE StationId = 'Station-1001' AND ReadingTimeStamp >= '2014-07-22 12:00:00' AND ReadingTimeStamp <= '2014-07-22 20:00:00';"
```

Riak TS should return the following JSON (formatted here for readability):

```
{
  "columns":[
    "StationId",
    "ReadingTimeStamp",
    "Temperature",
    "Humidity",
    "WindSpeed",
    "WindDirection"
  ],
  "rows":[
    [
      "Station-1001",
      1406032137000,
      53,
      43.2,
      2.5,
      289.0
    ]
  ]
}
```

When Riak TS writes the (row) key/value pair it uses the two part key (partition and local) to determine where to write the data within the cluster and how to write the data on disk. In the next section we will go to more detail about how the keys work: [How Partition Keys Work](How Partition Keys Work.md).

---

 **Previous**: [ Data Modeling Home](README.md) | **Next**: [How Partition Keys Work](How Partition Keys Work.md)