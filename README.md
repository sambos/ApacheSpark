# ApacheSpark
Apache Spark is designed for interactive batch processing and somewhat near realtime streaming (micro batch) processing. It will still be in the market and is well fit for batch processing. For real time streaming i would recommend to look at Apache Flink project.

## Sharing state across Spark Jobs
Global variable in spark   
Broadcast variables   
Stats Counter   
Partial aggregation (optimization that reduces data movement in shuffle stage)   
How does spark shuffle actually works ? - does it write to disk (2 stage process ?)   

 
> Apache Ignite for sharing state across spark applications

## Spark Performance Tuning
Beware, Spark has a lot more performance tuning parameters and requires a good knowledge of GC/Memory handling apart from understanding its core architecture. I will share with you few articles and tips on how to performance tune few areas, obviously this is not the place to provide all the performance tuning information..


## Things to Keep in Mind
Having too many small files on HDFS is not good for performance. First of all. Each time you read file, NameNode queries for block locations, then connect to the DataNode with stored file. The Overhead of this connections and responses is really huge.

* Understanding the  default Spark settings will help in diagnosing the problem, so make sure you understand the default settings. 
* By default Spark starts on YARN with 2 executors (--num-executors) with 1 thread each (--executor-cores) and 512m of RAM (--executor-cores), giving you only 2 threads with 512MB RAM each, which is really small for the real-world tasks.
* You can change this settings with --num-executors 4 --executor-memory 12g --executor-cores 4 which would give you more parallelism - 16 threads in this particular case, which means 16 tasks running in parallel. You can also check default parallelism from sc SparkConfig.
* For fast reading and writing use SequenceFile (that uses binary compressed format)

* Spark shuffles data when using joins or aggregations, you can override the shuffle partitions that spark uses by setting spark.sql.shuffle.partitions (default is 200) see [Spark SQL Programming Guide](http://spark.apache.org/docs/latest/sql-programming-guide.html#other-configuration-options)

```
sqlContext.setConf("spark.sql.shuffle.partitions", "400")
sqlContext.setConf("spark.default.parallelism", "300")

or using 
./bin/spark-submit --conf spark.sql.shuffle.partitions=400 --conf spark.default.parallelism=300
```
