## Working with the Hadoop API in CDE

The Hadoop API was born as a python library to manage files in HDFS.
Although CDP Data Services no longer use Hadoop, the API provides capabilities to interact with S3 and ADLS.

You can use the Hadoop API in CDE Sessions or Jobs in order to create, move, remove files and directories in Cloud Storage, and more.
This is a quickstart with a few basic commands to help you get started.

Copying the following commands in a CDE Session is recommended, although you can also run the same code in a CDE Job of type Spark.

### Step by Step Instructions

Preload some data in a cloud storage location of your choice.
Then, specify the path as a Python variable.
In this example we use two locations to read and write data with AWS S3.

```
>>> readlocation = "s3a://go01-demo/mycsvfile.csv"
>>> writelocation = "s3a://go01-demo/newdir/myfile"
```

Read the file in a Spark Dataframe:

```
>>> df=spark.read.csv(readlocation)
```

Obtain the configured FileSystem implementation:

```
>>> myPath = "s3a://go01-demo/newdir"

>>> sc=df.rdd.context
>>> hadoop = sc._jvm.org.apache.hadoop
>>> hdfs = hadoop.fs.FileSystem.get(sc._jvm.java.net.URI.create(myPath), sc._jsc.hadoopConfiguration())
```

Write the dataframe as a file in Cloud Storage:

```
>>> df.write.mode("overwrite").parquet(writelocation)
```

List the contents of the directory:

```
>>> status = hdfs.listStatus(hadoop.fs.Path("s3a://go01-demo/newdir"))

>>> for fileStatus in status:
      print(fileStatus.getPath())

s3a://go01-demo/newdir/myfile    
```

List the contents of a directory with filter pattern:

```
>>> status = hdfs.globStatus(hadoop.fs.Path("s3a://go01-demo/*.csv"))
>>> for fileStatus in status:
        print(fileStatus.getPath())

s3a://go01-demo/test_file.csv
```

Rename file:

```
>>> hdfs.rename(hadoop.fs.Path("s3a://go01-demo/test_file.csv"), hadoop.fs.Path("s3a://go01-demo/test_file_2.csv"))
True

>>> status = hdfs.globStatus(hadoop.fs.Path("s3a://go01-demo/*.csv"))
>>> for fileStatus in status:
        print(fileStatus.getPath())

s3a://go01-demo/test_file_2.csv
```

Delete file:

```
>>> hdfs.delete(hadoop.fs.Path("s3a://go01-demo/test_file_2.csv"), True)
True

>>> status = hdfs.globStatus(hadoop.fs.Path("s3a://go01-demo/*.csv"))
>>> for fileStatus in status:
        print(fileStatus.getPath())
```

Create a subdirectory:

```
>>> hdfs.mkdirs(hadoop.fs.Path("s3a://go01-demo/newdir/newsubdir"))
True
```

### References

The full API Method list is located at [this link](https://hadoop.apache.org/docs/current/api/org/apache/hadoop/fs/FileSystem.html)
