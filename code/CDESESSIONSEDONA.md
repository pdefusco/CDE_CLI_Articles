## Working with Sedona in CDE Spark Sessions

A Cloudera Data Engineering (CDE) Session is an interactive short-lived development environment for running Spark commands to help you iterate upon and build your Spark workloads.

You can use CDE Sessions in CDE Virtual Clusters of type "All Purpose - Tier 2". The following commands illustrate a basic Geospatial example with Apache Sedona.

### Requirements

* A CDE 1.20 Service in Public or Private Cloud (AWS, Azure, OCP, Cloudera ECS OK)
* A CDE Virtual Cluster of type "All Purpose" with Spark 3.2 or above.
* A working installation of the CDE CLI on your local machine.

### Step by Step Guide

Create the CDE Custom Runtime with Apache Sedona. Notice this image has already been built for you. You can therefore skip this step if you'd like.

```
% docker build --network=host -t pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-002 . -f Dockerfile

% docker run -it pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-003 . -f Dockerfile /bin/bash

% docker push pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-003
```

Create CDE Docker Resource for the Custom Runtime in CDE:

```
% cde credential create --name docker-creds-pauldefusco --type docker-basic --docker-server hub.docker.com --docker-username pauldefusco

% cde resource create --name dex-spark-runtime-sedona-geospatial-pauldefusco --image pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-003 --image-engine spark3 --type custom-runtime-image --type pyspark
```

Create CDE Files Resource and load geospatial file:

```
% cde resource create --name geospatial-data --type files

% cde resource upload --name geospatial-data --local-path geospatial_data/usa-county.tsv
 4.2MB/4.2MB 100% [==============================================] usa-county.tsv
```

Create the Session:

```
% cde session create --name geospatial-analysis --runtime-image-resource-name dex-spark-runtime-sedona-geospatial-pauldefusco --type pyspark
{
  "name": "geospatial-analysis",
  "type": "pyspark",
  "creator": "pauldefusco",
  "created": "2024-03-25T20:52:18Z",
  "lastStateUpdated": "2024-03-25T20:52:18Z",
  "state": "starting",
  "interactiveSpark": {
    "id": 7,
    "driverCores": 1,
    "executorCores": 1,
    "driverMemory": "1g",
    "executorMemory": "1g",
    "numExecutors": 1
  },
  "runtimeImageResourceName": "dex-spark-runtime-sedona-geospatial-pauldefusco"
}
```

Now interact with the session from your local terminal:

```
% cde session interact --name geospatial-analysis
```

You are now running the Spark Shell. The rest of this example is based on the [Sedona Documentation Quickstart](https://sedona.apache.org/1.5.1/tutorial/sql/)

Load the data from the Files Resource:

```
>>> var rawDf = sedona.read.format("csv").option("delimiter", "\t").option("header", "false").load("/Download/usa-county.tsv")
>>> rawDf.createOrReplaceTempView("rawdf")
>>> rawDf.show()
```

Create a Geometry type column on a DataFrame:

```
>>> spatialDf = sedona.sql("SELECT ST_GeomFromWKT(_c0) AS countyshape, _c1, _c2 FROM rawdf")
>>> spatialDf.createOrReplaceTempView("spatialDf")
>>> spatialDf.printSchema()
```

Convert Geometry Column to Coordinate Reference System:

```
>>> spark.sql("SELECT ST_Transform(countyshape, "epsg:4326", "epsg:3857") AS newcountyshape, _c1, _c2, _c3, _c4, _c5, _c6, _c7 FROM spatialdf")
```

Run a Range Query:

```
>>> spark.sql("SELECT * FROM spatialdf \
                WHERE ST_Contains (ST_PolygonFromEnvelope(1.0,100.0,1000.0,1100.0), \
                newcountyshape")
```

Run a KNN Query:

```
>>> spark.sql("SELECT countyname, ST_Distance(ST_PolygonFromEnvelope(1.0,100.0,1000.0,1100.0), newcountyshape) AS distance \
                FROM spatialdf \
                ORDER BY distance DESC \
                LIMIT 5")
```

### Summary:

* You can use CDE Sessions to interact with your data at scale while leveraging the full power of Apache Spark. CDE Sessions allows you to deploy sessions using Scala and Python, and switch between different versions of Spark.
* CDE Resources allow you to load and reuse dependencies when using CDE Sessions. As of CDE 1.20, Resources can be File Archives, Python Environments, Docker Images, and Git Repositories. In this example we loaded data from a Files Resources and installed Apache Sedona in a Docker Image which we used to launch the CDE Session.

### References

[Apache Sedona Documantion](https://sedona.apache.org/1.5.1/)

[CDE CLI command syntax reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-reference.html)

[CDE CLI Spark Flag reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-spark-flag-reference.html)

[CDE Documentation](https://docs.cloudera.com/data-engineering/cloud/overview/topics/cde-service-overview.html)

[CDE Sessions Documentation](https://docs.cloudera.com/data-engineering/cloud/sessions/topics/cde-create-sessions.html)

[Spark Geospatial in CDE with Apache Sedona](https://community.cloudera.com/t5/Community-Articles/Spark-Geospatial-with-Apache-Sedona-in-Cloudera-Data/ta-p/378086)
