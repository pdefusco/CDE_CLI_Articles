## Using Jars in CDE Spark Jobs

Cloudera Data Engineering (CDE) is a cloud-native service provided by Cloudera that is designed to simplify and enhance the development, deployment, and management of data engineering workloads at scale. It is part of the Cloudera Data Platform (CDP), which is a comprehensive, enterprise-grade platform for managing and analyzing data across hybrid and multi-cloud environments.

Among its many benefits, Cloudera Data Engineering allows you to build a "CDE Spark-Submit" with identical syntax to that of your Spark-Submit, or alternatively to declare your Spark-Submit as a "CDE Job of type Spark" with a reusable Job Definition for enhanced observability, troubleshooting, and dependency management.

These unique CDE capabilities are particularly advantageous for Spark Data Engineers who develop and deploy Spark Pipelines at scale, across multiple clusters, working with different Spark-Submit definitions and dynamic, complex dependencies.

For example, when packaging a JAR for a Spark-Submit, the JAR can include several types of dependencies that your Spark application needs to run properly. Your Jar can include application code (compiled Scala/Java code), third-party libraries (external dependencies), configuration and resource files (files for application configuration or runtime data) and custom JARs (any other internal or utility libraries your application requires).

In this article you will learn how to manage and simplify JAR dependency management with Cloudera Data Engineering in different scenarios.

### Example 1: CDE Job with Scala Application Code in Spark Jar

Scala Spark applications are typically developed and deployed in the following mannner:

1. Set Up Project in IDE: Use SBT to set up a Scala project in your IDE.
2. Write Code: Write your Scala application.
3. Compile & Package: Use sbt package to compile and package your code into a JAR.
4. Submit to Spark: Use spark-submit to run your JAR on a Spark cluster.

In this example you will build a CDE Spark Job with a Scala application that has already been compiled into a JAR. To learn how to complete these steps, please visit [this tutorial](https://xd-deng.com/render_html/step_by_step_to_package_spark_app_scala.html).

```
cde resource create --name cde_scala_job_files

cde resource upload --name cde_scala_job_files --local-path jars/cde-scala-example_2.12-0.1.jar

cde job create \
  --name cde-scala-job \
  --type spark \
  --mount-1-resource cde_scala_job_files \
  --application-file cde-scala-example_2.12-0.1.jar \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g

cde job run --name cde-scala-job
```

You can also add further JAR dependencies with the ```--jar``` or ```--jars``` options. In this case we'll add the Spark XML library from the same CDE Files Resource:

```
cde resource upload --name cde_scala_job_files --local-path jars/spark-xml_2.12-0.16.0.jar

cde job create \
  --name cde-scala-job-jar-dependency \
  --type spark \
  --mount-1-resource cde_scala_job_files \
  --application-file cdejobjar_2.12-1.0.jar \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g \
  --jar spark-xml_2.12-0.16.0.jar

cde job run --name cde-scala-job-jar-dependency
```

Notice you could have also done the above with two CDE Files Resources, each containing one of the JARs. You have the option to create as many CDE Files Resources as needed for each JAR file.

In the following example you will reference the application code JAR located in the ```cde_scala_job_files``` CDE Files Resource which you created earlier, and an additional JAR for the Spark-XML package from a new CDE Files Resource which you will create as ```cde_spark_xml_jar```.

Notice the new ```--mount-N-prefix``` option is used below. When you have more than one CDE Resource being utilized by the same ```CDE Job Create``` command you have to apply an alias to each Files Resource so each command option can reference the files correctly.

```
cde resource create --name cde_spark_xml_jar

cde resource upload --name cde_spark_xml_jar --local-path jars/spark-xml_2.12-0.16.0.jar

cde job create \
  --name cde-scala-job-multiple-jar-resources \
  --type spark \
  --mount-1-prefix scala_app_code \
  --mount-1-resource cde_scala_job_files \
  --mount-2-prefix spark_xml_jar \
  --mount-2-resource cde_spark_xml_jar \
  --application-file scala_app_code/cdejobjar_2.12-1.0.jar \
  --conf spark.sql.shuffle.partitions=10 \
  --executor-cores 2 \
  --executor-memory 2g \
  --jar spark_xml_jar/spark-xml_2.12-0.16.0.jar

cde job run --name cde-scala-job-multiple-jar-resources
```

### Example 2: CDE Job with PySpark Application Code and Jar Dependency from Maven

For Maven dependencies, you can use the ```--packages``` option to automatically download and include dependencies. This is often more convenient than manually managing JAR files. In the following example the ```--packages``` option replaces the ```--jars``` option.

In this specific example you will reference the Spark-XML package from Maven, so you can use it to parse the sample ```books.xml``` file from the same CDE Files Resource.

```
cde resource create --name spark_files --type files

cde resource upload --name spark_files --local-path read_xml.py --local-path books.xml

cde job create --name sparkxml \
  --application-file read_xml.py \
  --mount-1-resource spark_files \
  --type spark \
  --packages com.databricks:spark-xml_2.12:0.16.0

cde job run --name sparkxml
```

As in the previous example, multiple CDE Files Resources can be used to manage PySpark Application code and the sample XML file. Notice that the application code in ```read_xml_multi_resource.py``` is different. At line 67, the ```sample_xml_file``` Files Resource is referenced directly in the application code with its alias ```xml_data```.

```
cde resource create --name sample_xml_file --type files
cde resource create --name pyspark_script --type files

cde resource upload --name pyspark_script --local-path read_xml_multi_resource.py
cde resource upload --name sample_xml_file --local-path books.xml

cde job create --name sparkxml-multi-deps \
  --application-file code/read_xml_multi_resource.py \
  --mount-1-prefix code \
  --mount-1-resource pyspark_script \
  --mount-2-prefix xml_data \
  --mount-2-resource sample_xml_file \
  --type spark \
  --packages com.databricks:spark-xml_2.12:0.16.0

cde job run --name sparkxml-multi-deps
```

### Example 3: CDE Job with PySpark Application Code and Jar Dependency from CDE Files Resource

Similar to example 1, you can reference JARs directly uploaded into CDE Files Resources instead of using Maven as in example 2.  

The following commands pick up from example 2 but replace the ```packages``` option with the ```jars``` option.

Notice that the ```--jars``` option is used in the ```cde job run``` command rather than the ```cde job create```. The ```---jars``` option can either be set at CDE Job creation or runtime.

```
cde resource create --name spark_xml_jar --type files

cde resource upload --name spark_xml_jar --local-path jars/spark-xml_2.12-0.16.0.jar

cde job create --name sparkxml-multi-deps-jar-from-res \
  --application-file code/read_xml_multi_resource.py \
  --mount-1-prefix code \
  --mount-1-resource pyspark_script \
  --mount-2-prefix xml_data \
  --mount-2-resource sample_xml_file \
  --mount-3-prefix deps \
  --mount-2-resource spark_xml_jar \
  --type spark \

cde job run --name sparkxml-multi-deps-jar-from-res \
  --jar deps/spark-xml_2.12-0.16.0.jar
```

### Summary

In this article you used the CDE CLI to simplify Spark JAR management with Cloudera Data Engineering.

* You can use the CDE CLI to create CDE Job Definitions using Spark JAR dependencies, and to create CDE Files Resources to store and reference one or multiple JARs. Cloudera Data Engineering provides dramatic enhancements to Spark Dependency Management compared to traditional Spark-Submits outside of CDE.

* You can use the Job Runs page in the CDE UI to track JAR dependencies applied to each job execution. Cloudera Data Engineering provides dramatic enhancements to Spark Observability and Troubleshooting compared to traditional Spark-Submits outside of CDE.

### References

* [CDE Concepts](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-concepts.html)
* [Using the CDE CLI](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html)
* [CDE CLI Command Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-reference.html)
