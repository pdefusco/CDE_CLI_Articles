## Creating a CDE Job with a Spark Application Code located in S3

#### Objective

In this brief example you will learn how to use the CDE CLI to create a CDE Spark Job with PySpark and Scala app code located in an S3 bucket.

#### Requirements

In order to reproduce these examples in your CDE Virtual Cluster you will need:

* A Spark application in the form of a PySpark script or Scala jar, located in an S3 bucket.
* A working installation of the CDE CLI.
* A CDE 1.23 Service and a Spark 3 Virtual Cluster running in Cloudera on Cloud or Cloudera on Prem.

#### Using Dependencies located in S3 with the Spark-Submit

CDE provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, etc.

Apache Spark Spark-Submit allows you to run a Spark job with application code located in an S3 bucket. The CDE CLI also provides this functionality.

For example, in PySpark:

```
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files s3://your-bucket/path/to/dependency_one.zip, s3://your-bucket/path/to/dependency_two.py \
  --jars s3://your-bucket/path/to/dependency_one.jar,s3://your-bucket/path/to/dependency_two.jar \
  s3://your-bucket/path/to/pyspark_app.py \
  --arg1 value_one --arg2 value_two
```

Or with a Jar compiled from Scala application code:

```
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --py-files s3://your-bucket/path/to/dependency_one.zip, s3://your-bucket/path/to/dependency_two.py \
  --jars s3://your-bucket/path/to/dependency_one.jar,s3://your-bucket/path/to/dependency_two.jar \
  s3://your-bucket/path/to/spark_app.jar \
  --arg1 value_one --arg2 value_two
```

#### Using Dependencies located in S3 with the CDE CLI

You can accomplish the same with the CDE CLI, either by creating a CDE CLI Spark Submit or a CDE Job.

* When using the CDE Spark Submit the syntax is nearly identical to that of a Spark Submit.
* When creating the CDE Job the syntax is also similar but the application file flag is needed for specifying the app py or jar file.
* In both cases the Hadoop FileSystem API Spark configurations are needed. These are specified with the conf flag.  

CDE Spark Submit with PySpark application:

```
cde spark submit \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g \
  --conf spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  s3://default-cdp-bucket/data-eng-artifacts/cde_spark_jobs/simple-pyspark-sql.py
```

CDE Job with PySpark application:

```
cde job create \
  --application-file s3://your-bucket/path/to/pyspark_app.py \
  --py-files s3://your-bucket/path/to/dependency_one.zip, s3://your-bucket/path/to/dependency_two.py \
  --jars s3://default-cdp-bucket/path/to/dependency_one.jar,s3://your-bucket/path/to/dependency_two.jar \
  --arg1 value_one
```

CDE Spark Submit with Scala application:

```
cde spark submit \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g \
  --conf spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  s3://default-cdp-bucket/data-eng-artifacts/cde_spark_jobs/simple-pyspark-sql.py
```

CDE Job with Scala application:

```
cde job create \
  --application-file s3://your-bucket/path/to/spark_app.jar \
  --py-files s3://your-bucket/path/to/dependency_one.zip, s3://your-bucket/path/to/dependency_two.py \
  --jars s3://default-cdp-bucket/path/to/dependency_one.jar,s3://your-bucket/path/to/dependency_two.jar \
  --arg1 value_one
```

For example, in the case of a sample PySpark application:

CDE Spark Submit:

```
cde spark submit \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g \
  --conf spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  s3://pdf-3425-buk-c59557bd/data-eng-artifacts/cde_spark_jobs/simple-pyspark-sql.py
```

CDE Job:

```
cde job create \
  --name my-cde-job-from-s3-pyspark \
  --type spark \
  --application-file s3://pdf-3425-buk-c59557bd/data-eng-artifacts/cde_spark_jobs/simple-pyspark-sql.py \
  --conf spark.sql.shuffle.partitions=10 \
  --conf spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --executor-cores 2 \
  --executor-memory 2g
```

```
cde job run \
  --name my-cde-job-from-s3-pyspark
```

Or with a Scala Jar.

CDE Spark Submit:

```
cde spark submit \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g \
  --conf spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  s3://pdf-3425-buk-c59557bd/data-eng-artifacts/cde_spark_jobs/cdejobjar_2.12-1.0.jar
```

```
cde job create \
  --name my-cde-job-from-s3-scalajar \
  --type spark \
  --conf spark.hadoop.fs.s3.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
  --application-file s3://data-eng-artifacts/cde_spark_jobs/cdejobjar_2.12-1.0.jar \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g
```

```
cde job run \
  --name my-cde-job-from-s3-scalajar
```

#### Using CDE Files Resources

As an alternative to hosting your application code and file dependencies in S3, you can leverage CDE Files Resources.

Files Resources are arbitrary collections of files that a job can reference where application code and any necessary configuration files or supporting libraries can be stored. Files can be uploaded to and removed from a resource as needed.

CDE Files Resources offer a few key advantages:

* They are located inside a CDE Virtual Cluster where they can be easily monitored.
* You can easily work with them through the CDE CLI and perform actions such as updating their content or adding them to CDE job definitions.
* Once the job has run, resources applied for a specific run are easily traceable in the UI or via the CLI and API. In other words, if you want to dynamically apply file dependencies across multiple runs, the CDE Files Resources can be swapped modularly and are shown in the Job Runs UI post-execution.

You can create a CDE Files reosource with the CLI.

```
cde resource create \
  --name my-files-resource \
  --type files
```

You can upload files to the Resource:

```
cde resource upload \
  --name my-files-resource \
  --local-path simple-pyspark-sql.py
```

And finally you can mount the Files Resource when creating the CDE Job Definition:

```
cde job create \
  --type spark \
  --name my-job-with-resource \
  --mount-1-resource my-files-resource \
  --application-file simple-pyspark-sql.py
```

And finally run it with:

```
cde job run \
  --name my-job-with-resource \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g
```

For more in-depth information on using CDE Resources please visit the following publications:
* [Working with CDE Files Resources](https://community.cloudera.com/t5/Community-Articles/Working-with-CDE-Files-Resources/ta-p/379891)
* [Using CDE Resources in CDE Sessions](https://community.cloudera.com/t5/Community-Articles/Using-CDE-Resources-in-CDE-Sessions/ta-p/387834)

## Summary and Next Steps

Cloudera Data Engineering (CDE) provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, and so on.

In this article we have reviewed some advanced use cases for the CLI. If you are using the CDE CLI you might also find the following articles and demos interesting:

* [Installing the CDE CLI](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html)
* [Simple Introduction to the CDE CLI](https://github.com/pdefusco/CDE_CLI_Simple)
* [CDE Concepts](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-concepts.html)
* [CDE CLI Command Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-reference.html)
* [CDE CLI Spark Flag Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-spark-flag-reference.html)
* [CDE CLI Airflow Flag Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-airflow-flag-reference.html)
* [CDE CLI list command syntax reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-list-flag-reference.html)
* [CDE Jobs API Reference](https://docs.cloudera.com/data-engineering/cloud/jobs-rest-api-reference/index.html)
