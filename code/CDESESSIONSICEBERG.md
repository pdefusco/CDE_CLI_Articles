## Working with Iceberg in CDE Spark Sessions

A Cloudera Data Engineering (CDE) Session is an interactive short-lived development environment for running Spark commands to help you iterate upon and build your Spark workloads.

You can use CDE Sessions in CDE Virtual Clusters of type "All Purpose - Tier 2". The following commands illustrate a basic Iceberg Time Travel Example.

Create the Session:

```
% cde session create --name interactiveSession \
                      --type pyspark \
                      --executor-cores 2 \
                      --executor-memory "2g"
{
  "name": "interactiveSession",
  "creator": "pauldefusco",
  "created": "2023-11-28T22:00:47Z",
  "type": "pyspark",
  "lastStateUpdated": "2023-11-28T22:00:47Z",
  "state": "starting",
  "interactiveSpark": {
    "id": 5,
    "driverCores": 1,
    "executorCores": 2,
    "driverMemory": "1g",
    "executorMemory": "2g",
    "numExecutors": 1
  }
}
```

Show session metadata:

```
% cde session describe --name interactiveSession
{
  "name": "interactiveSession",
  "creator": "pauldefusco",
  "created": "2023-11-28T22:00:47Z",
  "type": "pyspark",
  "lastStateUpdated": "2023-11-28T22:01:16Z",
  "state": "available",
  "interactiveSpark": {
    "id": 5,
    "appId": "spark-3fe3bd8905a04eef8805e6b973ec4289",
    "driverCores": 1,
    "executorCores": 2,
    "driverMemory": "1g",
    "executorMemory": "2g",
    "numExecutors": 1
  }
}
```

Interact via the PySpark Shell from your terminal (the session is running in CDE):

```
% cde session interact --name interactiveSession
Starting REPL...
Waiting for the session to go into an available state...
Connected to Cloudera Data Engineering...
Press Ctrl+D (i.e. EOF) to exit
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\
      /_/

Type in expressions to have them evaluated.

>>>
```

Run some basic Spark SQL operations:

```
>>> from pyspark.sql.types import Row, StructField, StructType, StringType, IntegerType

>>> rows = [Row(name="John", age=19), Row(name="Smith", age=23), Row(name="Sarah", age=18)]

>>> some_df = spark.createDataFrame(rows)

>>> some_df.printSchema()
root
 |-- name: string (nullable = true)
 |-- age: long (nullable = true)
>>>
```

Notice that we didn't need to create a Spark Session. The Spark Context is already running inside the CDE Session.

Here are all the confs the session is running with. Notice that Iceberg dependencies have already been accounted for:

```
>>> def printConfs(confs):
      for ele1,ele2 in confs:
        print("{:<14}{:<11}".format(ele1,ele2))

>>> printConfs(confs)
spark.eventLog.enabledtrue       
spark.driver.hostinteractivesession-b7d65d8c1d6005a9-driver-svc.dex-app-58kqsms2.svc
spark.kubernetes.executor.annotation.created-bylivy
spark.kubernetes.memoryOverheadFactor0.1        
spark.sql.catalog.spark_catalogorg.apache.iceberg.spark.SparkSessionCatalog
spark.kubernetes.container.imagecontainer.repository.cloudera.com/cloudera/dex/dex-livy-runtime-3.3.0-7.2.16.3:1.19.3-b29
spark.kubernetes.executor.label.nameexecutor   
spark.kubernetes.driver.connectionTimeout60000      
spark.hadoop.yarn.resourcemanager.principalpauldefusco       
...
spark.yarn.isPythontrue       
spark.kubernetes.submission.connectionTimeout60000      
spark.kryo.registrationRequiredfalse      
spark.sql.catalog.spark_catalog.typehive       
spark.kubernetes.driver.pod.nameinteractivesession-b7d65d8c1d6005a9-driver
```

Load a CSV file from Cloud Storage:

```
>>> cloudPath = "s3a://go01-demo/datalake/pdefusco/cde119_workshop"

>>> car_installs = spark.read.csv(cloudPath + "/car_installs_119.csv", header=True, inferSchema=True)

>>> car_installs.show()
+-----+-----+----------------+--------------------+
|   id|model|             VIN|           serial_no|
+-----+-----+----------------+--------------------+
|16413|    D|433248UCGTTV245J|5600942CL3R015666...|
|16414|    D|404328UCGTTV965J|204542CL4R0156661...|
|16415|    B|647168UCGTTV8Z5J|6302942CL2R015666...|
|16416|    B|454608UCGTTV7H5J|4853942CL1R015666...|
|16417|    D|529408UCGTTV6R5J|2428342CL9R015666...|
|16418|    B|362858UCGTTV7A5J|903142CL2R0156661...|
|16419|    E|609158UCGTTV245J|3804142CL7R015666...|
|16420|    D|  8478UCGTTV825J|6135442CL7R015666...|
|16421|    B|539488UCGTTV4R5J|306642CL6R0156661...|
|16422|    B|190928UCGTTV6A5J|5466242CL1R015666...|
|16423|    B|316268UCGTTV4M5J|4244342CL5R015666...|
|16424|    B|298898UCGTTV3Y5J|3865742CL4R015666...|
|16425|    B| 28688UCGTTV9T5J|6328542CL5R015666...|
|16426|    D|494858UCGTTV295J|463642CL5R0156661...|
|16427|    D|503338UCGTTV5Y5J|4358642CL2R015666...|
|16428|    D|167128UCGTTV2H5J|3809342CL1R015666...|
|16429|    D|547178UCGTTV7M5J|2768042CL3R015666...|
|16430|    B|503998UCGTTV4Q5J|2568142CL6R015666...|
|16431|    D|433998UCGTTV9Y5J|6338642CL6R015666...|
|16432|    B|378548UCGTTV7V5J|2648942CL1R015666...|
+-----+-----+----------------+--------------------+
```

Create a Hive Managed Table with Spark:

```
>>> username = "pauldefusco"

>>> spark.sql("DROP DATABASE IF EXISTS MYDB_{} CASCADE".format(username))

>>> spark.sql("CREATE DATABASE IF NOT EXISTS MYDB_{}".format(username))

>>> car_installs.write.mode("overwrite").saveAsTable('MYDB_{0}.CAR_INSTALLS_{0}'.format(username), format="parquet")
```

Migrate the table to Iceberg Table Format:

```
spark.sql("ALTER TABLE MYDB_{0}.CAR_INSTALLS_{0} UNSET TBLPROPERTIES ('TRANSLATED_TO_EXTERNAL')".format(username))

spark.sql("CALL spark_catalog.system.migrate('MYDB_{0}.CAR_INSTALLS_{0}')".format(username))
```

You can query Iceberg Metadata tables to track Iceberg Snapshots, History, Partitions, etc:

```
>>> spark.read.format("iceberg").load("spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}.history".format(username)).show(20, False)
+-----------------------+-------------------+---------+-------------------+
|made_current_at        |snapshot_id        |parent_id|is_current_ancestor|
+-----------------------+-------------------+---------+-------------------+
|2023-11-29 23:58:43.427|6191572403226489858|null     |true               |
+-----------------------+-------------------+---------+-------------------+

>>> spark.read.format("iceberg").load("spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}.snapshots".format(username)).show(20, False)
+-----------------------+-------------------+---------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|committed_at           |snapshot_id        |parent_id|operation|manifest_list                                                                                                                                                                |summary                                                                                                                                                                                                                                                                  |
+-----------------------+-------------------+---------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|2023-11-29 23:58:43.427|6191572403226489858|null     |append   |s3a://go01-demo/warehouse/tablespace/external/hive/mydb_pauldefusco.db/car_installs_pauldefusco/metadata/snap-6191572403226489858-1-bf191e06-38cd-4d6e-9757-b8762c999177.avro|{added-data-files -> 2, added-records -> 82066, added-files-size -> 1825400, changed-partition-count -> 1, total-records -> 82066, total-files-size -> 1825400, total-data-files -> 2, total-delete-files -> 0, total-position-deletes -> 0, total-equality-deletes -> 0}|
+-----------------------+-------------------+---------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Insert some data. Notice that Iceberg provides a PySpark API to create, append, and overwrite data in an Iceberg table from a Spark Dataframe. In this case we will append some data that we sample from the same table:

```
# PRE-INSERT TIMESTAMP
>>> from datetime import datetime

>>> now = datetime.now()

>>> timestamp = datetime.timestamp(now)

>>> print("PRE-INSERT TIMESTAMP: ", timestamp)
PRE-INSERT TIMESTAMP:  1701302029.338524

# PRE-INSERT COUNT
>>> spark.sql("SELECT COUNT(*) FROM spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}".format(username)).show()
+--------+
|count(1)|
+--------+
|   82066|
+--------+

>>> temp_df = spark.sql("SELECT * FROM spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}".format(username)).sample(fraction=0.1, seed=3)

>>> temp_df.writeTo("spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}".format(username)).append()
```

Check the new count Post insert:

```
# POST-INSERT COUNT
>>> spark.sql("SELECT COUNT(*) FROM spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}".format(username)).show()
+--------+
|count(1)|
+--------+
|   90276|
+--------+
```

Notice that the table history and snapshots have been updated:

```
>>> spark.sql("SELECT * FROM spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}.history".format(username)).show(20, False)
+-----------------------+-------------------+-------------------+-------------------+
|made_current_at        |snapshot_id        |parent_id          |is_current_ancestor|
+-----------------------+-------------------+-------------------+-------------------+
|2023-11-29 23:58:43.427|6191572403226489858|null               |true               |
|2023-11-30 00:00:15.263|1032812961485886468|6191572403226489858|true               |
+-----------------------+-------------------+-------------------+-------------------+

>>> spark.sql("SELECT * FROM spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}.snapshots".format(username)).show(20, False)
+-----------------------+-------------------+-------------------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|committed_at           |snapshot_id        |parent_id          |operation|manifest_list                                                                                                                                                                |summary                                                                                                                                                                                                                                                                                                                        |
+-----------------------+-------------------+-------------------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|2023-11-29 23:58:43.427|6191572403226489858|null               |append   |s3a://go01-demo/warehouse/tablespace/external/hive/mydb_pauldefusco.db/car_installs_pauldefusco/metadata/snap-6191572403226489858-1-bf191e06-38cd-4d6e-9757-b8762c999177.avro|{added-data-files -> 2, added-records -> 82066, added-files-size -> 1825400, changed-partition-count -> 1, total-records -> 82066, total-files-size -> 1825400, total-data-files -> 2, total-delete-files -> 0, total-position-deletes -> 0, total-equality-deletes -> 0}                                                      |
|2023-11-30 00:00:15.263|1032812961485886468|6191572403226489858|append   |s3a://go01-demo/warehouse/tablespace/external/hive/mydb_pauldefusco.db/car_installs_pauldefusco/metadata/snap-1032812961485886468-1-142965b8-67ea-4b53-b76d-558ab5e74e1f.avro|{spark.app.id -> spark-93d1909a680948fea5303b55986704ac, added-data-files -> 1, added-records -> 8210, added-files-size -> 183954, changed-partition-count -> 1, total-records -> 90276, total-files-size -> 2009354, total-data-files -> 3, total-delete-files -> 0, total-position-deletes -> 0, total-equality-deletes -> 0}|
+-----------------------+-------------------+-------------------+---------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Time travel to pre-insert table state:

```
# TIME TRAVEL AS OF PREVIOUS TIMESTAMP
>>> df = spark.read.option("as-of-timestamp", int(timestamp*1000)).format("iceberg").load("spark_catalog.MYDB_{0}.CAR_INSTALLS_{0}".format(username))

# POST TIME TRAVEL COUNT
>>> print(df.count())
82066
```

Finally, drop the database:

```
>>> spark.sql("DROP DATABASE IF EXISTS MYDB_{} CASCADE".format(username))
```

Exit the Spark Shell (Ctrl+D). List commands that were run in the session. Notice that this could be a lot, so the example below only includes a few initial commands.

```
% cde session statements --name interactiveSession
+--------------------------------+------------------------------------------+
|              CODE              |                  OUTPUT                  |
+--------------------------------+------------------------------------------+
| print("hello Spark")           | hello Spark                              |
+--------------------------------+------------------------------------------+
| from pyspark.sql.types import  |                                          |
| Row, StructField, StructType,  |                                          |
| StringType, IntegerType        |                                          |
+--------------------------------+------------------------------------------+
| rows = [Row(name="John",       |                                          |
| age=19), Row(name="Smith",     |                                          |
| age=23), Row(name="Sarah",     |                                          |
| age=18)]                       |                                          |
+--------------------------------+------------------------------------------+
| some_df =                      |                                          |
| spark.createDataFrame(rows)    |                                          |
+--------------------------------+------------------------------------------+
| some_df.printSchema()          | root  |-- name: string                   |
|                                | (nullable = true)  |-- age:              |
|                                | long (nullable = true)                   |
+--------------------------------+------------------------------------------+
```

List all sessions:

```
% cde session list
+---------------------------+-----------+---------+-------------+----------------------+----------------------+-------------+
|           NAME            |   STATE   |  TYPE   | DESCRIPTION |       CREATED        |     LAST UPDATED     |   CREATOR   |
+---------------------------+-----------+---------+-------------+----------------------+----------------------+-------------+
| francetemp                | killed    | pyspark |             | 2023-11-16T15:59:35Z | 2023-11-16T16:02:16Z | jmarchand   |
| IcebergSession            | available | pyspark |             | 2023-11-29T21:24:27Z | 2023-11-29T21:56:56Z | pauldefusco |
| interactiveSession        | killed    | pyspark |             | 2023-11-28T22:00:47Z | 2023-11-28T22:01:16Z | pauldefusco |
| interactiveSessionIceberg | available | pyspark |             | 2023-11-29T23:17:58Z | 2023-11-29T23:56:06Z | pauldefusco |
| myNewSession              | killed    | pyspark |             | 2023-11-28T21:58:38Z | 2023-11-28T21:59:06Z | pauldefusco |
| mySparkSession            | killed    | pyspark |             | 2023-11-28T21:44:30Z | 2023-11-28T21:45:01Z | pauldefusco |
| TA-demo                   | killed    | pyspark |             | 2023-11-13T10:12:12Z | 2023-11-13T10:13:41Z | glivni      |
+---------------------------+-----------+---------+-------------+----------------------+----------------------+-------------+
```

Kill session:

```
% cde session kill --name interactiveSession
```
