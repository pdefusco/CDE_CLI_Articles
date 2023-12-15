# Working with CDE Spark Job Parameters

The CDE CLI doesn't just allow you to create CDE Spark Jobs with syntax that is nearly identical to that of a Spark Submit.

The CDE CLI also offers you invaluable advantages as part of your daily Spark Pipeline development and management activities including enhancing observability, reusability, and overall organization of Spark dependencies and configurations.

Simply put, if you're creating and managing many Spark Submits, the CDE CLI will save you time and dramatically increase your productivity.

In this example you will learn about different options for parameterizing a CDE Spark Job i.e. to pass Spark Configurations, Options, Files and Arguments.

## CDE Spark Jobs and Spark Configurations

The CDE Spark Job is a reusable definition of a Spark Application consisting of its code, and file, Docker, and Python dependencies, and Spark configurations and options.

Once a CDE Spark Job is created, its definition is accessible in the CDE UI or via the CDE CLI.

```
cde job create --name myJob\
               --type Spark\
               --application-file myScript.py\
               --mount-1-resource myFilesResource
```

More importantly, it is not yet run. The Application is actually executed with a separate command.

```
cde job run --name myJob
```

Generally, parameterizing a CDE Spark Jobs allows you to dynamically set important configurations at runtime i.e. override default parameters based on some changing external input.

Before moving on, let us review the different types of parameters that can be set in a CDE Spark Job:

1) Spark Application File: this is the PySpark script or Jar containing the Application code. In CDE you must use the ```--application-file``` flag and you can only set this at CDE Spark Job creation time.

Examples:

```
cde job create --name myPySparkJob\
               --type spark\
               --application-file myScript.py\
               --mount-1-resource myFilesResource
```

```
cde job create --name myScalaJob \
               --type spark \
               --application-file myJar.jar \
               --mount-1-resource myFilesResource
```

The files can only be found by CDE if they are present in a Files resource. As shown above, the Files resource is then set with the ```--mount-N-resource``` flag. More on that in the end to end example section.


2) Spark Resources: these include basic job resources such as executor and driver memory and cores, initial number of executors, min and max executors for Spark Dynamic Allocation.

They can be set both at CDE Spark Job creation as well as runtime. Each of these has a default value which is chosen unless specified by your CLI command.

Examples:

```
cde job create --name myPySparkJob\
               --type spark\
               --application-file myScript.py\
               --mount-1-resource myFilesResource\
               --executor-cores 4\
               --executor-memory "2g"
```

```
cde job run --name myPySparkJob \
            --executor-cores 4 \
            --driver-cores 2
```

The above job will run with the following resource configurations:

```
--executor-cores 4
--executor-memory "2g"
--driver-cores 2
```


3) Spark Configurations: Spark offers a wide range of configurations ranging from Python version to memory options, join and dynamic allocation thresholds, and much more.

These can be set via the ```--conf``` flag e.g. ```--conf spark.executor.memoryOverhead=6g``` or ```--conf spark.pyspark.python=python3``` or ```--conf spark.yarn.maxAppAttempts=1```.

These can also be ovverriden at runtime.

Examples:

```
cde job create --name myPySparkJob\
               --type spark\
               --application-file myScript.py\
               --mount-1-resource myFilesResource\
               --conf spark.executor.memoryOverhead=6g
```

```
cde job create --name myPySparkJob\
               --type spark\
               --application-file myScript.py\
               --mount-1-resource myFilesResource\
               --conf spark.executor.memoryOverhead=6g
```

```
cde job run --name myPySparkJob\
            --type spark\
            --application-file myScript.py\
            --mount-1-resource myFilesResource\
            --conf spark.executor.memoryOverhead=2g
```

In the above example, the "memoryOverhead" setting is overriden to "2g".

4) Command Line Arguments: these are specific variable inputs that lend themselves particularly well to being overridden at runtime.

They are defined with the ```--arg``` flag in the CDE Spark Job definition, and require being read in Spark Application code via the Python ```sys.argv``` module.

For example, a CDE Spark Job will include the ```--arg 10``` argument from the CLI in order for the value of "10" to be utilized as part of the Spark Application code.

Example CDE CLI command to create the CDE Spark Job:

```
cde job create --name myPySparkJob\
               --type spark\
               --application-file myScript.py\
               --mount-1-resource myFilesResource\
               --conf spark.executor.memoryOverhead=6g\
               --arg 10
```

Example CDE Spark Job application code rerefencing the argument:

```
import sys
from pyspark.sql import SparkSession

spark = SparkSession\
    .builder\
    .appName("CLIArguments")\
    .getOrCreate()

print("JOB ARGUMENT")
print(sys.argv[1])
```

You can ovverride CLI arguments at runtime. The following example sets the input to 5 rather than 10:

```
cde job run --name myPySparkJob --arg 5
```

5) Files: these can be set via the ```--file``` or ```--py-files``` options and allow you to specify file dependencies at CDE Spark Job creation.

More importantly, the funtionality provided by these flags is enhanced by CDE Files Resources and we using them instead is generally recommended.

CDE Files Resources are unique to CDE and allow you to pass and manage entire files as CDE Spark Job dependencies.

You can reference these files within Spark Application code via the ```/app/mount``` prefix.

Example:

```
cde job create --name penv-udf-test --type spark \
               --mount-1-prefix appFiles/ \
               --mount-1-resource cde-cli-files-resource \
               --python-env-resource-name cde-cli-penv-resource \
               --application-file /app/mount/appFiles/deps_udf_dependency.py \
               --conf spark.executorEnv.PYTHONPATH=/app/mount/appFiles/deps_udf.zip \
               --py-file /app/mount/appFiles/deps_udf.zip
```

Here, the "--py-file" flag is used to set in order to distribute dependencies that have been compressed and then loaded in a CDE Files Resource.

Notice that unlike in a Spark Submit, you cannot use ```--file``` to specify the application code and you must use the ```--application-file``` flag as shown in example 1.

CDE Files Resources are mounted on the Spark Application pod at CDE Job runtime. Thanks to this, you can also reference a file from within your Spark Application code by using the ConfigParser module and referencing the "/app/mount" directory.

Example properties file "parameters.conf" that has been loaded in the CDE Files Resource referenced by the CDE Spark Job:

```
[general]
property_1: s3a://go01-demo/
property_2: datalake/
```

Example PySpark application code referencing the "parameters.conf" file:

```
from pyspark.sql import SparkSession
import configparser

config = configparser.ConfigParser()
config.read('/app/mount/parameters.conf')
data_lake_name=config.get("general","property_1")
file_path=config.get("general","property_2")

spark = SparkSession.\
                    builder.\
                    appName('INGEST').\
                    config("spark.kubernetes.access.hadoopFileSystems", data_lake_name).getOrCreate()

cloudPath=data_lake_name+file_path

myDf  = spark.read.csv(cloudPath + "/myCsv.csv", header=True, inferSchema=True)
myDf.show()
```

## End to End CDE Spark Job Workflow Example

Now that you have gained exposure to the different options we will utilize these in the context of a simplified CDE Spark Job creation workflow that will show how to actually implement the above:

1) Create a CDE Files Resource in order to host all Application code and file dependencies.
2) Upload Application code and file dependencies to the CDE Files Resource in order to make those accessible to CDE Spark Jobs in the future.
3) Create the CDE Spark Job by referencing Application code, file dependencies and other Spark configurations in order to then:
4) Run the CDE Spark Job with either no additional configurations or by overriding configurations in order to execute your Spark Application.

#### 1. Create a CDE Files Resource

```
cde resource create --name myProperties
```

#### 2. Upload Application Code and File Dependencies

```
cde resource upload --name myProperties\
                    --local-path cde_jobs/propertiesFile_1.conf\
                    --local-path cde_jobs/propertiesFile_2.conf\
                    --local-path cde_jobs/sparkJob.py
```

#### 3. Create CDE Spark Job

```
cde job create --name myPySparkJob\
               --type spark\
               --application-file sparkJob.py\
               --mount-1-resource myProperties\
               --executor-cores 2\
               --executor-memory "2g"
```

#### 4. Run the CDE Spark Job with Different Options

Example 1: Run the job with two CLI arguments and read properties file 1

```
cde job run --name myPySparkJob\
            --arg MY_DB\
            --arg CUSTOMER_TABLE\
            --arg propertiesFile_1.conf
```

Example 2: Run the job with two CLI arguments and read properties file 2

```
cde job run --name myPySparkJob\
            --arg MY_DB\
            --arg SALES_TABLE\
            --arg propertiesFile_2.conf
```

Application code in sparkJob.py:

```
import sys
from pyspark.sql import SparkSession
import configparser

spark = SparkSession\
    .builder\
    .appName("PythonSQL")\
    .getOrCreate()

print("JOB ARGUMENTS")
print(sys.argv)
print(sys.argv[0])
print(sys.argv[1])
print(sys.argv[2])
print(sys.argv[3])

dbname = sys.argv[1]
tablename = sys.argv[2]

config = configparser.ConfigParser()
config.read('/app/mount/{}.properties'.format(sys.argv[3]))
property_1=config.get("general","property_1")
property_2=config.get("general","property_2")

def myFunction(dbname, tablename, property_1, property_2):
    print("DBNAME\n")
    print(dbname)
    print("TABLNAME\n")
    print(tablename)
    print("PROPERTY1\n")
    print(property_1)
    print("PROPERTY2\n")
    print(property_2)
    print("COMPLETE!\n")

# A list of Rows. Infer schema from the first row, create a DataFrame and print the schema

myFunction(dbname, tablename, property_1, property_2)
```

### References:

[CDE CLI command syntax reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-reference.html)

[CDE CLI Spark Flag reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-spark-flag-reference.html)
