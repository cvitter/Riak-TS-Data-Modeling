# [Riak TS](README.md) - Installing Riak TS

Installing a single node of Riak TS locally for learning or development is a relatively simple task. If you haven't already done so you should download the latest version of TS from the official TS website: http://docs.basho.com/riak/ts/latest/downloads/.

The following instructions will help you get up and running on Mac OS X:

1. Download and expand the tar.gz file to your local directory (and move it to your preferred location on your system);
1. Increase the open files limit on your machine (see http://docs.basho.com/riak/kv/2.1.4/using/performance/open-files-limit/ for more information on why and for instructions on increasing the open files limit on your machine)
1. Navigate to your Riak TS directory (For example: ``` > cd riak-ts-1.3.1```);
1. Launch Riak TS: ``` > bin/riak start```

Once Riak TS has started you can further test that it is up and running using the following commands:

``` > bin/riak ping```  
Riak should respond with:  
``` > pong```  

``` > bin/riak-admin test```  
Riak should respond with:  
``` > Successfully completed 1 read/write cycle to 'riak@127.0.0.1'```
