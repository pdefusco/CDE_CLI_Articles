## Using Jars in CDE Spark Jobs & Sessions

### Example 1: Using Packages Argument

```
cde resource create --name files --type files

cde resource upload --name files --local-path read_xml.py --local-path books.xml

cde job create --name sparkxml --application-file read_xml.py --mount-1-resource files --type spark --packages com.databricks:spark-xml_2.12:0.16.0

cde job run --name sparkxml
```

### Example 2: Using Packages and Repository Arguments

### Example 3: Load in the UI



### Example 4: Load from CDE Resource

```
cde resource create --name files --type files

cde resource upload --name files --local-path jars/read_xml.py --local-path jars/books.xml --local-path jars/spark-xml_2.12-0.16.0.jar

cde job create --name sparkxml --application-file read_xml.py --mount-1-resource files --type spark

cde job run --name sparkxml --jar /app/mount/spark-xml_2.12-0.16.0.jar
```


### References

* [CDE Concepts](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-concepts.html)
* [Using the CDE CLI](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html)
* [CDE CLI Command Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-reference.html)
