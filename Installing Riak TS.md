# [Riak TS](README.md) - Installing Riak TS

Installing a single node of Riak TS locally for learning or development is a relatively simple task. If you haven't already done so you should download the latest version of TS from the official TS website: http://docs.basho.com/riak/ts/latest/downloads/.

The following instructions will help you get up and running with a single Riak TS node on a Mac running OS X:

1. Download and expand the tar.gz file to your local directory (and move it to your preferred location on your system);
1. Increase the open files limit on your machine (see http://docs.basho.com/riak/kv/2.1.4/using/performance/open-files-limit/ for more information on why and for instructions on increasing the open files limit on your machine)
1. Navigate to your Riak TS directory (For example: ``` > cd riak-ts-1.4.0```);
1. Start a Riak TS node: ``` > bin/riak start```

Once Riak TS has started you can further test that it is up and running using the following commands:

``` $ bin/riak ping```  

Riak should respond with:  

``` pong ```  

You can also use ``` riak-admin test ``` as illustrated below:

``` 
$ bin/riak-admin test
Successfully completed 1 read/write cycle to 'riak@127.0.0.1'
```
