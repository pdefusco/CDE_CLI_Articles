## Working with CDE Spark Sessions

A Cloudera Data Engineering (CDE) Session is an interactive short-lived development environment for running Spark commands to help you iterate upon and build your Spark workloads.

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
from pyspark.sql.types import Row, StructField, StructType, StringType, IntegerType
rows = [Row(name="John", age=19), Row(name="Smith", age=23), Row(name="Sarah", age=18)]
some_df = spark.createDataFrame(rows)
some_df.printSchema()

>>> from pyspark.sql.types import Row, StructField, StructType, StringType, IntegerType

>>> rows = [Row(name="John", age=19), Row(name="Smith", age=23), Row(name="Sarah", age=18)]

>>> some_df = spark.createDataFrame(rows)

>>> some_df.printSchema()
root
 |-- name: string (nullable = true)
 |-- age: long (nullable = true)
>>>
```

List commands that were run in the session:

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
+--------------------+-----------+---------+-------------+----------------------+----------------------+-------------+
|        NAME        |   STATE   |  TYPE   | DESCRIPTION |       CREATED        |     LAST UPDATED     |   CREATOR   |
+--------------------+-----------+---------+-------------+----------------------+----------------------+-------------+
| francetemp         | killed    | pyspark |             | 2023-11-16T15:59:35Z | 2023-11-16T16:02:16Z | jmarchand   |
| interactiveSession | available | pyspark |             | 2023-11-28T22:00:47Z | 2023-11-28T22:01:16Z | pauldefusco |
| myNewSession       | killed    | pyspark |             | 2023-11-28T21:58:38Z | 2023-11-28T21:59:06Z | pauldefusco |
| mySparkSession     | killed    | pyspark |             | 2023-11-28T21:44:30Z | 2023-11-28T21:45:01Z | pauldefusco |
| TA-demo            | killed    | pyspark |             | 2023-11-13T10:12:12Z | 2023-11-13T10:13:41Z | glivni      |
+--------------------+-----------+---------+-------------+----------------------+----------------------+-------------+
```

Kill session:

```
% cde session kill --name interactiveSession
```
