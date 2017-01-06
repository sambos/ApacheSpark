# ApacheSpark
Apache Spark is designed for interactive batch processing and somewhat near realtime streaming (micro batch) processing. It will still be in the market and is well fit for batch processing. For real time streaming i would recommend to look at Apache Flink project.

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
* This flag (spark.sql.tungsten.enabled) is default enabled and is removed since [Spark 1.6](https://spark.apache.org/releases/spark-release-1-6-0.html)
