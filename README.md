# ApacheSpark

## SparkSQL vs HiveSQL

## Sharing state across Spark Jobs
Global variabl in spark
Broadcast variables
Stats Counter
Partial aggregation (optimization that reduces data movement in shuffle stage)
How does spark shuffle actually works ? - does it write to disk (2 stage process ?)
 - as of 1.1, its pull based (not push based) - but its pluggable
 - 
 
> Apache Ignite for sharing state across spark applications


## Spark with Tungsten
* Improves memory and CPU efficiency of Spark Applications (optimizes Memory+CPU)
* Pushes performances closer to the limits of the modern hardware

* spark.sql.tungsten.enabled flag is true by default in Spark 1.5
* try disabling and see how it performs
* 
