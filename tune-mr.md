## Tuning MapReduce Job

### Tuning map and reduce tasks for Java Heap
mapreduce.map.memory.mb is the upper memory limit that Hadoop allows to be allocated to a mapper, in megabytes. 
The default is 512. If this limit is exceeded, Hadoop will kill the mapper

* Look at total memory allocated for each node `yarn.nodemanager.resource.memory-mb`
* Look for CPUs per node `yarn.nodemanager.resource.cpu-vcores` 
* Your total # of possible MR tasks would be `( yarn.nodemanager.resource.memory-mb x yarn.nodemanager.resource.cpu-vcores )`

* Find the total memory and cores allocated for MR jobs in /etc/hadoop/conf/mapred-site.xml
`mapreduce.[map|reduce].memory.mb`   
`mapreduce.[map|reduce].cpu.vcores`

Allocate Jvm Heap size for each map and reduce task. Generally they should be kept at 75-80% of the total allocated MB mapreduce.[map|reduce].memory.mb
Also the reduce taks would normally require more memory than map tasks.

`mapreduce.map.java.opts = 0.8 * mapreduce.map.memory.mb`  
`mapreduce.reduce.java.opts = 0.8 * 2 * mapreduce.map.memory.mb` 


For Hadoop v1 use the following conifgs (these are deprecated as of v2, but check your cluster vendor for specific changes):   
`mapred.job.(map|reduce).memory.mb`   
`mapred.(map|reduce).child.java.opts` 

``` shell
         #HADOOP_OPTS="-Djava.security.auth.login.config=jaas-client.conf" hadoop \
          HADOOP_OPTS="-Djava.security.auth.login.config=jaas-client.conf -Dmapreduce.map.memory.mb=4096 -Dmapreduce.reduce.memory.mb=8192 -Dmapreduce.map.java.opts=-Xmx2048m -Dmapreduce.reduce.java.opts=-Xmx4096m -Dmapreduce.cluster.acls.enabled=true -Dmapreduce.job.acl-view-job=user -Dmapreduce.reduce.memory.totalbytes=10" hadoop \
         --config /etc/hadoop/conf \
         jar $CDH_PARCEL/jars/search-mr-*-job.jar \
         org.apache.solr.hadoop.MapReduceIndexerTool \
         --log4j $LOG4J_PATH \
         --morphline-file $2 \
         --output-dir $HDFS_OUT_DIR \
         --verbose \
         --zk-host $SOLR_ZK_ENSEMBLE \
         --collection $1 \
         --go-live "${@:3}"

```

