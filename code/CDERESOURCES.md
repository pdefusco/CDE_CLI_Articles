## Working with CDE Resources

Create a Files Resource:

```
% cde resource create --name myScripts \
                      --type files
```

Uplaod multiple files to the same Files Resource:

```
% cde resource upload --name myScripts \
                      --local-path cde_jobs/spark_geospatial.py \
                      --local-path cde_jobs/utils.py
     3.5KB/3.5KB 100% [==============================================] spark_geospatial.py
     4.0KB/4.0KB 100% [==============================================] utils.py
```

Describe Files resource:

```
% cde resource describe --name myScripts
{
  "name": "myScripts",
  "type": "files",
  "status": "ready",
  "signature": "08531cbe538858eb20bda5ff1b7567ae4623d885",
  "created": "2023-11-28T23:31:31Z",
  "modified": "2023-11-28T23:33:32Z",
  "retentionPolicy": "keep_indefinitely",
  "files": [
    {
      "path": "spark_geospatial.py",
      "signature": "ec91fb5bddfcd16a0bcbe344f229b5e326b759c5",
      "sizeBytes": 3529,
      "created": "2023-11-28T23:33:32Z",
      "modified": "2023-11-28T23:33:32Z"
    },
    {
      "path": "utils.py",
      "signature": "aa5a8ea4b4f240183da8bd2d2b354eeaa58fd97a",
      "sizeBytes": 3996,
      "created": "2023-11-28T23:33:32Z",
      "modified": "2023-11-28T23:33:32Z"
    }
  ]
}
```

Create a Files Resource for data:

```
% cde resource create --name myData \
                      --type files
```

Upload an archive file to the resource:

```
% cde resource upload-archive --name myData \
                              --local-path data/ne_50m_admin_0_countries_lakes.zip
 817.5KB/817.5KB 100% [==============================================] ne_50m_admin_0_countries_lakes.zip
```

Describe resource metadata. Notice that the archive has been unarchived for you:

```
% cde resource describe --name myData
{
  "name": "myData",
  "type": "files",
  "status": "ready",
  "signature": "d552dff8fb80a0c7067afa1c4227b29010cce67b",
  "created": "2023-11-28T23:35:43Z",
  "modified": "2023-11-28T23:36:56Z",
  "retentionPolicy": "keep_indefinitely",
  "files": [
    {
      "path": "ne_50m_admin_0_countries_lakes.cpg",
      "signature": "663b90c899fa25a111067be0c22ffc64dcf581c2",
      "sizeBytes": 5,
      "created": "2023-11-28T23:36:55Z",
      "modified": "2023-11-28T23:36:55Z"
    },
    {
      "path": "ne_50m_admin_0_countries_lakes.dbf",
      "signature": "eec48a122399782bbef02aa8108e99aeaf52e506",
      "sizeBytes": 786828,
      "created": "2023-11-28T23:36:56Z",
      "modified": "2023-11-28T23:36:56Z"
    },
    {
      "path": "ne_50m_admin_0_countries_lakes.prj",
      "signature": "308d6355be935e0f7853161b1adda5bcd48188ff",
      "sizeBytes": 143,
      "created": "2023-11-28T23:36:56Z",
      "modified": "2023-11-28T23:36:56Z"
    },
    {
      "path": "ne_50m_admin_0_countries_lakes.README.html",
      "signature": "4bec87fbbe5f4e0e18edb3d6a4f10e9e2a705581",
      "sizeBytes": 38988,
      "created": "2023-11-28T23:36:55Z",
      "modified": "2023-11-28T23:36:55Z"
    },
    {
      "path": "ne_50m_admin_0_countries_lakes.shp",
      "signature": "57c38f48c5234db925a9fb1b31785250bd7c8d86",
      "sizeBytes": 1652200,
      "created": "2023-11-28T23:36:56Z",
      "modified": "2023-11-28T23:36:56Z"
    },
    {
      "path": "ne_50m_admin_0_countries_lakes.shx",
      "signature": "983eba7b34cf94b0cfe8bda8b2b7d533bd233c49",
      "sizeBytes": 2036,
      "created": "2023-11-28T23:36:56Z",
      "modified": "2023-11-28T23:36:56Z"
    },
    {
      "path": "ne_50m_admin_0_countries_lakes.VERSION.txt",
      "signature": "9e3d18e5216a7b4dfd402b00a25ee794842b481b",
      "sizeBytes": 7,
      "created": "2023-11-28T23:36:55Z",
      "modified": "2023-11-28T23:36:55Z"
    }
  ]
}
```

Create CDE Credentials for accessing your Docker repository:

```
cde credential create --name my-docker-creds \
                      --type docker-basic \
                      --docker-server hub.docker.com \
                      --docker-username pauldefusco
```

Create a CDE Custom Docker Runtime Resource:

```
cde resource create --name dex-spark-runtime-sedona-geospatial \
                    --image pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-003 \
                    --image-engine spark3 \
                    --type custom-runtime-image
```

Notice that the custom image has already been created. If you want to learn more about how to create Custom Docker Resources please visit this [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Creating-Custom-Runtimes-with-Spark3-Python-3-9-on-Cloudera/ta-p/368867)

Finally, create a job leveraging all three resources above:

```
cde job create --name geospatialRdd \
                --type spark \
                --mount-1-prefix code/ --mount-1-resource myScripts \
                --mount-2-prefix data/ --mount-2-resource myData \
                --runtime-image-resource-name dex-spark-runtime-sedona-geospatial \
                --packages org.apache.sedona:sedona-spark-shaded-3.0_2.12:1.5.0,org.datasyslab:geotools-wrapper:1.5.0-28.2 \
                --application-file code/spark_geospatial.py \
                --arg myArg \
                --max-executors 4 \
                --min-executors 2 \
                --executor-cores 2
```

Notice the following:

* When using multiple Files resources you should prefix each i.e. "data" and "code". The "data" prefix is used at line 82 ("/app/mount/data") in the "spark_geospatial.py" script in order to access the data from the resource. The "code" prefix is usded in the same CLI command in the application file argument.
* If you are using any Spark packages you can set these directly at job creation.
* You can pass one or multiple arguments to the Python script via the --arg argument. The arguments are referenced in the script with the "sys.argv" syntax e.g. line 60 in "geospatial_rdd.py".

Run the job:

```
cde job run --name geospatialRdd --executor-cores 4
```

Notice that at runtime, you can override spark configs that were set at job creation. For example, the "--executor-cores" was originally set to 2 and is now overridden to 4.

List job runs filtering by job name:

```
% cde run list --filter 'job[eq]geospatialRdd'
[
  {
    "id": 21815,
    "job": "geospatialRdd",
    "type": "spark",
    "status": "failed",
    "user": "pauldefusco",
    "started": "2023-11-29T00:32:02Z",
    "ended": "2023-11-29T00:32:36Z",
    "mounts": [
      {
        "dirPrefix": "data/",
        "resourceName": "myData"
      },
      {
        "dirPrefix": "code/",
        "resourceName": "myScripts"
      }
    ],
    "runtimeImageResourceName": "dex-spark-runtime-sedona-geospatial",
    "spark": {
      "sparkAppID": "spark-f542530da24f485da4993338dca81d3c",
      "sparkAppURL": "https://58kqsms2.cde-g6hpr9f8.go01-dem.ylcu-atmi.cloudera.site/hs/history/spark-f542530da24f485da4993338dca81d3c/jobs/",
      "spec": {
        "file": "code/geospatial_rdd.py",
        "args": [
          "myArg"
        ],
        "driverMemory": "1g",
        "driverCores": 1,
        "executorMemory": "1g",
        "executorCores": 4,
        "conf": {
          "spark.dynamicAllocation.maxExecutors": "4",
          "spark.dynamicAllocation.minExecutors": "2",
          "spark.jars.packages": "org.apache.sedona:sedona-spark-shaded-3.0_2.12:1.5.0,org.datasyslab:geotools-wrapper:1.5.0-28.2"
        }
      }
    },
    "identity": {
      "disableRoleProxy": true,
      "role": "instance"
    }
  }
]

```

Open Spark UI for respective run:

```
% cde run ui --id 21815  
```
