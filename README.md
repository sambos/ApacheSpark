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

[Spark Performance Tuning from Spark Summit 2013](https://www.youtube.com/watch?v=NXp3oJHNM7E)
