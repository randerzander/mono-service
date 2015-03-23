An Ambari Stack service package for the Mono ".NET" runtime.

**Note:** Deploying this stack will automatically install the [EPEL](https://fedoraproject.org/wiki/EPEL) repository if it isn't in /etc/yum.repos.d/ already.

To deploy, copy the entire directory into your Ambari stacks folder and restart Ambari:

```
git clone https://github.com/randerzander/mono-stack
sudo mv mono-stack /var/lib/ambari-server/resources/stacks/HDP/2.2/services/
sudo service ambari-server restart
```

Then you can click on 'Add Service' from the 'Actions' dropdown menu in the bottom left of the Ambari dashboard. When you've completed the install process, the Mono Runtime will be available for use on all selected nodes.

The below examples use a C# application compiled with Visual Studio 2012, but will also work with C# code compiled via Mono's "mcs" tool. The C# application appends '!!!' to each line read from standard input, and prints it back to standard output.

Example execution as a "Hive UDF":
```
[dev@n2 example]$ hadoop fs -put test_data/ .
[dev@n2 example]$ hive
15/02/18 19:47:56 WARN conf.HiveConf: HiveConf of name hive.optimize.mapjoin.mapreduce does not exist
15/02/18 19:47:56 WARN conf.HiveConf: HiveConf of name hive.heapsize does not exist
15/02/18 19:47:56 WARN conf.HiveConf: HiveConf of name hive.server2.enable.impersonation does not exist
15/02/18 19:47:56 WARN conf.HiveConf: HiveConf of name hive.auto.convert.sortmerge.join.noconditionaltask does not exist

Logging initialized using configuration in file:/etc/hive/conf/hive-log4j.properties
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/hdp/2.2.0.0-2041/hadoop/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/hdp/2.2.0.0-2041/hive/lib/hive-jdbc-0.14.0.2.2.0.0-2041-standalone.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
hive> create external table test(line string) location '/user/dev/test_data/';   
OK
Time taken: 0.589 seconds
hive> select * from test;
OK
Hello
This
is a test
Time taken: 0.879 seconds, Fetched: 3 row(s)
hive> add file ConsoleApplication.exe;
Added resources: [ConsoleApplication.exe]
hive> select transform(line) using 'mono ConsoleApplication.exe' as output from test;
Query ID = dev_20150218194848_e6576a80-3340-40d4-8030-34ec3057cfac
Total jobs = 1
Launching Job 1 out of 1


Status: Running (Executing on YARN cluster with App id application_1424306012395_0006)

--------------------------------------------------------------------------------
        VERTICES      STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED
--------------------------------------------------------------------------------
Map 1 ..........   SUCCEEDED      1          1        0        0       0       0
--------------------------------------------------------------------------------
VERTICES: 01/01  [==========================>>] 100%  ELAPSED TIME: 4.70 s     
--------------------------------------------------------------------------------
OK
Hello!!!
This!!!
is a test!!!
Time taken: 9.37 seconds, Fetched: 3 row(s)
```

Example of running the same C# application over files in HDFS via Streaming MapReduce:
```
[dev@n2 example]$ hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar -file ConsoleApplication.exe -mapper 'mono ConsoleApplication.exe' -input /user/dev/test_data -output /user/dev/output
15/02/18 20:03:36 WARN streaming.StreamJob: -file option is deprecated, please use generic option -files instead.
packageJobJar: [ConsoleApplication.exe] [/usr/hdp/2.2.0.0-2041/hadoop-mapreduce/hadoop-streaming-2.6.0.2.2.0.0-2041.jar] /tmp/streamjob4542833215895771050.jar tmpDir=null
15/02/18 20:03:38 INFO impl.TimelineClientImpl: Timeline service address: http://n2.dev:8188/ws/v1/timeline/
15/02/18 20:03:38 INFO client.RMProxy: Connecting to ResourceManager at n2.dev/192.168.1.127:8050
15/02/18 20:03:39 INFO impl.TimelineClientImpl: Timeline service address: http://n2.dev:8188/ws/v1/timeline/
15/02/18 20:03:39 INFO client.RMProxy: Connecting to ResourceManager at n2.dev/192.168.1.127:8050
15/02/18 20:03:39 INFO mapred.FileInputFormat: Total input paths to process : 1
15/02/18 20:03:39 INFO mapreduce.JobSubmitter: number of splits:2
15/02/18 20:03:39 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1424306012395_0008
15/02/18 20:03:40 INFO impl.YarnClientImpl: Submitted application application_1424306012395_0008
15/02/18 20:03:40 INFO mapreduce.Job: The url to track the job: http://n2.dev:8088/proxy/application_1424306012395_0008/
15/02/18 20:03:40 INFO mapreduce.Job: Running job: job_1424306012395_0008
15/02/18 20:03:46 INFO mapreduce.Job: Job job_1424306012395_0008 running in uber mode : false
15/02/18 20:03:46 INFO mapreduce.Job:  map 0% reduce 0%
15/02/18 20:03:53 INFO mapreduce.Job:  map 100% reduce 0%
15/02/18 20:03:59 INFO mapreduce.Job:  map 100% reduce 100%
15/02/18 20:04:00 INFO mapreduce.Job: Job job_1424306012395_0008 completed successfully
15/02/18 20:04:00 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=45
		FILE: Number of bytes written=347480
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=228
		HDFS: Number of bytes written=33
		HDFS: Number of read operations=9
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Launched map tasks=2
		Launched reduce tasks=1
		Data-local map tasks=2
		Total time spent by all maps in occupied slots (ms)=9266
		Total time spent by all reduces in occupied slots (ms)=4128
		Total time spent by all map tasks (ms)=9266
		Total time spent by all reduce tasks (ms)=4128
		Total vcore-seconds taken by all map tasks=9266
		Total vcore-seconds taken by all reduce tasks=4128
		Total megabyte-seconds taken by all map tasks=9488384
		Total megabyte-seconds taken by all reduce tasks=4227072
	Map-Reduce Framework
		Map input records=3
		Map output records=3
		Map output bytes=33
		Map output materialized bytes=51
		Input split bytes=196
		Combine input records=0
		Combine output records=0
		Reduce input groups=3
		Reduce shuffle bytes=51
		Reduce input records=3
		Reduce output records=3
		Spilled Records=6
		Shuffled Maps =2
		Failed Shuffles=0
		Merged Map outputs=2
		GC time elapsed (ms)=488
		CPU time spent (ms)=3610
		Physical memory (bytes) snapshot=852717568
		Virtual memory (bytes) snapshot=4731486208
		Total committed heap usage (bytes)=985661440
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=32
	File Output Format Counters 
		Bytes Written=33
15/02/18 20:04:00 INFO streaming.StreamJob: Output directory: /user/dev/output
[dev@n2 example]$ hadoop fs -cat /user/dev/output/*
Hello!!!	
This!!!	
is a test!!!
```

If you want to delete the Mono client from the Ambari services list (mono-devel will remain installed on selected nodes):
```
curl -u $user:$pass -i -H 'X-Requested-By: ambari' -X DELETE http://$host:8080/api/v1/clusters/$cluster/services/MONO
```
