## `SparkSession` vs `SparkContext`
The SparkSession is created when you build your session, as in the following:
```java
SparkSession spark = SparkSession.builder()
.appName("An app")
.master("local[*]")
.getOrCreate();
```
As part of your session, you will also get a context: SparkContext. The context was your only way to deal with Spark prior to v2. You mostly do not need to interact with the SparkContext, but when you do (accessing infrastructure information, creating accumulators, and more, described in chapter 17), this is how you access it:
```java
SparkContext sc = spark.sparkContext();
System.out.println("Running Spark v" + sc.version());
```

## Executors
Executors are JVM processes that run computations and store data for your application.

## Task distribution
The `sparkSession` which resides in master, sends tasks to the available nodes.   

## Mounting data
As with the shared drive, I highly recommend that the mount point is the same on every worker.

## Code deploy to cluster
* Build an uber JAR with your code and all your dependencies.
* Build a JAR with your app and make sure all the dependencies are on every worker node (not recommended).
* Clone/pull from your source control repository.

## Building fat/uber jar
Exclusions are key; you do not want to carry all your dependencies. If you do not have exclusions, all your dependent classes will be transferred in your uber JAR. This includes all the Spark classes and artifacts. They are indeed needed, but because they are included with Spark, they will be available in your target system. If you bundle them in your uber JAR, that uber JAR will become really big, and you may encounter conflicts between libraries.

Because Spark comes with more than 220 libraries, you don’t need to bring, in your uber JAR, the dependencies that are already available on the target system.

`minimizeJa` Removes all classes that are not used by the project, reducing the size of jar.
```xml
<configuration>
  <minimizeJAR>true</minimizeJAR>
  ...
</configuration>
```


- [ ] is it enough to make the dependencies provided or we should exclude in maven shade plugin?

✅ Answer: spark master makes jar available for download by the workers. consider that for high volume files and large number of workers they may push the master to edge. example of log spark-submit doing it.
```text
2018-08-20 11:52:14 INFO SparkContext:54 - Added JAR file:/.../app.JAR at
spark://un.oplo.io:42805/JARs/app.JAR with timestamp 1534780334746
```

- [ ] why our `stdout` is always empty?