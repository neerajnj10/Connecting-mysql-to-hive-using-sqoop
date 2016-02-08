
### how to use sqoop (informally database connector)


Step 1: Get Data

In the first step we will grab some data from an existing database, created when doing a demo of cascalog, and load it 
into Hive using a tool called sqoop. Sqoop is a tool designed to move large quantities of data between hdfs and structured 
datastores.

Download sqoop and cd into bin/

```
./sqoop import --connect jdbc:mysql://localhost/cascalog  --username dan  --table unique_user_counts -m 1 
--hive-import
```

This will import data from the cascalog MySQL database, specifically the unique_user_counts database. The –hive-import switch will place data into Hive (I installed Hive using Homebrew). The -m toggle defines the number of mappers the job will use, in this instance just 1.

You can take a peek at the data using the Hive repl.

```
λ bin  hive
hive> show databases;
OK
default
Time taken: 2.48 seconds
hive> use default;
OK
Time taken: 0.014 seconds
hive> show tables;
OK
unique_user_counts
Time taken: 0.286 seconds
hive> select * from unique_user_counts;
```


This dumps out the data. If you want to know more about your table then you can use the describe ‘table_name’ command.

To get data into R Hive must be started in server mode, this can be achieved by

```
hive --service hiveserver
```

The final step is R …

```
library(RJDBC)
 
hive_jars <-list.files("/usr/local/Cellar/hadoop/1.1.1/libexec", pattern="jar$", full.names=T)
hadoop_jars <- list.files("/usr/local/Cellar/hive/0.9.0/libexec/lib", pattern="jar$", full.names=T)
 
hive_driver <- JDBC("org.apache.hadoop.hive.jdbc.HiveDriver", c(hive_jars, hadoop_jars))
 
hive_conn <- dbConnect(hive_driver, "jdbc:hive://localhost:10000/default")
 
rs <- dbGetQuery(hive_conn,"select client, count from unique_user_counts")
 
x <- 1:length(rs$count)
plot(x, rs$count)
```


The *interesting* part here is the definition of the jars required to connect to Hive.

@biomunky.

