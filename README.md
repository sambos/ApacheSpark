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

* Recommended resources :
  * http://people.csail.mit.edu/matei/papers/2012/nsdi_spark.pdf
  * http://site.clairvoyantsoft.com/understanding-resource-allocation-configurations-spark-application/


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
* Spark automatically sets the number of partitions of an input file according to its size and for distributed shuffles. By default spark create one partition for each block of the file in HDFS it is 64MB by default.
  * Command to find block size : `hdfs getconf -confKey dfs.blocksize`

* Consider repartitioning carefully, depending on your data organization, it may or may not cause shuffle. The shuffle can happen at the begining of stage (is called Shuffle Read) and that can happen at the end of stage (is called Shuffle Write).

* The actual allocated memory by YARN for an executor = --executor-memory + spark.yarn.executor.memoryOverhead. The memoryOverhead is calculated as 7% of executor memory that you want to allocate. If you request 10GB, yarn will allocate total 10+0.7 = ~11GB
  * The calculation for that overhead is MAX(384, .07 * spark.executor.memory). If 7% of requsted is greater than 384MB, the greater overhead is used.
  * Without proper memory allocation, you will see errors like `Container killed by YARN for exceeding memory limits. 9.0 GB of 9 GB physical memory used`
  * Look for `yarn.scheduler.maximum-allocation-mb` to understand how much max memory can be allocated to a container. AM can only request this much max.
* Keep the number of cores per executor to  <=5 because more than 5 cores per executor may degrade HDFS I/O throughput.
* Keep aside few resources for Yarn processes
  * reserve one executor and 5 cores on each node other resources
  * reserve 1-2gb per node for additional overhead
  * remember driver is also an executor

### Understanding overall capacity:

|Type|Estimate|
|------------|-------------------|
|Cache Memory |	spark.executor.memory * spark.storage.memoryFraction|
|Container Memory	| spark.executor.memory + spark.yarn.executor.memoryOverhead|
|Heap Memory	| Container Memory - Cache Memory|
|Used Memory	| num-executors * Container Memory|
|Remaining Memory	| yarn.nodemanager.resource.memory-mb - Used Memory|

** yarn.nodemanager.resource.memory-mb => Amount of physical memory allocated to containers
** yarn.scheduler.maximum-allocation-mb => max memory RM can allocate to a container

### Tasks
* Per spark documentation, the recommended # of tasks in a cluster is 2 to 3 times CPU cores. You can specify the parallelism with `spark.default.parallelism` as part of spark-submit job.

### Data Skew
This could be a killer if data is not properly partitioned. If you see that 1% of the task is taking 99% of the time, then there is a probable indication that you have data skew problem. This is mostly caused when a large rdd is joined with a smaller rdd. Possible resolutions:
* Repartition the data based on the partitions required, recommended after joins or unions and after reading compressed data.
* By inroducing replication technique, see example: https://datarus.wordpress.com/2015/05/04/fighting-the-skew-in-spark

### Reducing Partitions using Coalese
Use Coalese with Shuffle as false instead of repartition when simply reducing the # of partitions, this will avoid full shuffle (minimize data movement). This is useful when you just want to write to disk with less number of partitions. Repartition internally calls coalese with shuffle as false.
