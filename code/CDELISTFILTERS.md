## Filtering Jobs, Runs and Resources with Filters

You can list all jobs and job runs.

```
% cde job list
% cde run list
```

However, that is often impossible if you have a large number of jobs/runs in your Virtual Cluster. Therefore, using filters can be very important.

### Setup

Prior to running the filtering commands you must set up some jobs and related dependencies. Run the following commands:

```
% cde resource create --name myScripts \
                      --type files

% cde resource upload --name myScripts \
                      --local-path cde_jobs/spark_geospatial.py \
                      --local-path cde_jobs/utils.py

% cde resource describe --name myScripts

% cde resource create --name myData \
                      --type files

% cde resource upload-archive --name myData \
                              --local-path data/ne_50m_admin_0_countries_lakes.zip

cde credential create --name my-docker-creds \
                      --type docker-basic \
                      --docker-server hub.docker.com \

cde resource create --name dex-spark-runtime-sedona-geospatial \
                    --image pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-003 \
                    --image-engine spark3 \
                    --type custom-runtime-image

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

cde job run --name geospatialRdd --executor-cores 4
```

### Filtering Examples

Filter all jobs by name where name equals "geospatialRdd"

```
% cde job list --filter 'name[eq]geospatialRdd'
[
  {
    "name": "geospatialRdd",
    "type": "spark",
    "created": "2023-11-29T00:59:11Z",
    "modified": "2023-11-29T00:59:11Z",
    "retentionPolicy": "keep_indefinitely",
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
    "spark": {
      "file": "code/spark_geospatial.py",
      "args": [
        "myArg"
      ],
      "driverMemory": "1g",
      "driverCores": 1,
      "executorMemory": "1g",
      "executorCores": 2,
      "conf": {
        "spark.dynamicAllocation.maxExecutors": "4",
        "spark.dynamicAllocation.minExecutors": "2",
        "spark.jars.packages": "org.apache.sedona:sedona-spark-shaded-3.0_2.12:1.5.0,org.datasyslab:geotools-wrapper:1.5.0-28.2"
      }
    },
    "schedule": {
      "enabled": false,
      "user": "pauldefusco"
    },
    "runtimeImageResourceName": "dex-spark-runtime-sedona-geospatial"
  }
]
```

You can nest filters. For example, filter all jobs where job application file equals "code/spark_geospatial.py":

```
% cde job list --filter 'spark.file[eq]code/spark_geospatial.py'
[
  {
    "name": "geospatialRdd",
    "type": "spark",
    "created": "2023-11-29T00:59:11Z",
    "modified": "2023-11-29T00:59:11Z",
    "retentionPolicy": "keep_indefinitely",
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
    "spark": {
      "file": "code/spark_geospatial.py",
      "args": [
        "myArg"
      ],
      "driverMemory": "1g",
      "driverCores": 1,
      "executorMemory": "1g",
      "executorCores": 2,
      "conf": {
        "spark.dynamicAllocation.maxExecutors": "4",
        "spark.dynamicAllocation.minExecutors": "2",
        "spark.jars.packages": "org.apache.sedona:sedona-spark-shaded-3.0_2.12:1.5.0,org.datasyslab:geotools-wrapper:1.5.0-28.2"
      }
    },
    "schedule": {
      "enabled": false,
      "user": "pauldefusco"
    },
    "runtimeImageResourceName": "dex-spark-runtime-sedona-geospatial"
  }
]
```

You can use different operators. For example, search all jobs whose name contains "spark":

```
% cde job list --filter 'name[rlike]spark'          
[
  {
    "name": "sparkxml",
    "type": "spark",
    "created": "2023-11-23T07:11:41Z",
    "modified": "2023-11-23T07:39:32Z",
    "retentionPolicy": "keep_indefinitely",
    "mounts": [
      {
        "dirPrefix": "/",
        "resourceName": "files"
      }
    ],
    "spark": {
      "file": "read_xml.py",
      "driverMemory": "1g",
      "driverCores": 1,
      "executorMemory": "1g",
      "executorCores": 1,
      "conf": {
        "dex.safariEnabled": "false",
        "spark.jars.packages": "com.databricks:spark-xml_2.12:0.16.0",
        "spark.pyspark.python": "python3"
      },
      "logLevel": "INFO"
    },
    "schedule": {
      "enabled": false,
      "user": "pauldefusco"
    }
  }
]
```

Search all jobs created on or after 11/23/23:

```
% cde job list --filter 'created[gte]2023-11-23'
API User Password: [
  {
    "name": "asdfsdfsdfsdf",
    "type": "airflow",
    "created": "2023-11-23T06:56:45Z",
    "modified": "2023-11-23T06:56:45Z",
    "retentionPolicy": "keep_indefinitely",
    "mounts": [
      {
        "resourceName": "PipelineResource-asdfsdfsdfsdf-1700722602468"
      }
    ],
    "airflow": {
      "dagID": "asdfsdfsdfsdf",
      "dagFile": "dag.py"
    },
    "schedule": {
      "enabled": false,
      "user": "dschoberle",
      "start": "Thu, 23 Nov 2023 06:56:44 GMT",
      "catchup": true
    }
  },
  {
    "name": "geospatialRdd",
    "type": "spark",
    "created": "2023-11-29T00:59:11Z",
    "modified": "2023-11-29T00:59:11Z",
    "retentionPolicy": "keep_indefinitely",
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
    "spark": {
      "file": "code/spark_geospatial.py",
      "args": [
        "myArg"
      ],
      "driverMemory": "1g",
      "driverCores": 1,
      "executorMemory": "1g",
      "executorCores": 2,
      "conf": {
        "spark.dynamicAllocation.maxExecutors": "4",
        "spark.dynamicAllocation.minExecutors": "2",
        "spark.jars.packages": "org.apache.sedona:sedona-spark-shaded-3.0_2.12:1.5.0,org.datasyslab:geotools-wrapper:1.5.0-28.2"
      }
    },
    "schedule": {
      "enabled": false,
      "user": "pauldefusco"
    },
    "runtimeImageResourceName": "dex-spark-runtime-sedona-geospatial"
  }
]
```

Search all jobs with executorCores less than 2:

```
% cde job list --filter 'spark.executorCores[lt]2'          
API User Password: [
  {
    "name": "CDEPY_SPARK_JOB",
    "type": "spark",
    "created": "2023-11-14T23:02:48Z",
    "modified": "2023-11-14T23:02:48Z",
    "retentionPolicy": "keep_indefinitely",
    "mounts": [
      {
        "resourceName": "CDEPY_DEMO"
      }
    ],
    "spark": {
      "file": "pysparksql.py",
      "driverMemory": "1g",
      "driverCores": 1,
      "executorMemory": "4g",
      "executorCores": 1,
      "conf": {
        "spark.pyspark.python": "python3"
      },
      "logLevel": "INFO"
    },
    "schedule": {
      "enabled": false,
      "user": "pauldefusco"
    }
  },
  {
    "name": "CDEPY_SPARK_JOB_APAC",
    "type": "spark",
    "created": "2023-11-15T03:33:36Z",
    "modified": "2023-11-15T03:33:36Z",
    "retentionPolicy": "keep_indefinitely",
    "mounts": [
      {
        "resourceName": "CDEPY_DEMO_APAC"
      }
    ],
    "spark": {
      "file": "pysparksql.py",
      "driverMemory": "1g",
      "driverCores": 1,
      "executorMemory": "4g",
      "executorCores": 1,
      "conf": {
        "spark.pyspark.python": "python3"
      },
      "logLevel": "INFO"
    },
    "schedule": {
      "enabled": false,
      "user": "pauldefusco"
    }
  },
]
```

List all runs for job "geospatialRdd":

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
  },
  {
    "id": 21825,
    "job": "geospatialRdd",
    "type": "spark",
    "status": "failed",
    "user": "pauldefusco",
    "started": "2023-11-29T00:48:29Z",
    "ended": "2023-11-29T00:49:01Z",
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
      "sparkAppID": "spark-e5460856fb3a459ba7ee2c748c802d07",
      "sparkAppURL": "https://58kqsms2.cde-g6hpr9f8.go01-dem.ylcu-atmi.cloudera.site/hs/history/spark-e5460856fb3a459ba7ee2c748c802d07/jobs/",
      "spec": {
        "file": "myScripts/geospatial_rdd.py",
        "args": [
          "myArg"
        ],
        "driverMemory": "1g",
        "driverCores": 1,
        "executorMemory": "1g",
        "executorCores": 2,
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

You can combine multiple filters. Return all job runs from today (11/29/23) i.e. where the start date is greater than or equal to 11/29 and the end date is less than or equal to 11/30. Notice all times default to +00 UTC timezone.

```
% cde run list --filter 'started[gte]2023-11-29' --filter 'ended[lte]2023-11-30'
[
  {
    "id": 21907,
    "job": "ge_data_quality-pauldefusco-banking",
    "type": "spark",
    "status": "succeeded",
    "user": "pauldefusco",
    "started": "2023-11-29T02:56:44Z",
    "ended": "2023-11-29T02:57:46Z",
    "mounts": [
      {
        "dirPrefix": "/",
        "resourceName": "cde_demo_files-pauldefusco-banking"
      }
    ],
    "runtimeImageResourceName": "dex-spark-runtime-ge-data-quality-pauldefusco-banking",
    "spark": {
      "sparkAppID": "spark-8f9d7999056f4b53a01cc2afc5304cca",
      "sparkAppURL": "https://58kqsms2.cde-g6hpr9f8.go01-dem.ylcu-atmi.cloudera.site/hs/history/spark-8f9d7999056f4b53a01cc2afc5304cca/jobs/",
      "spec": {
        "file": "ge_data_quality.py",
        "driverMemory": "1g",
        "driverCores": 1,
        "executorMemory": "1g",
        "executorCores": 1
      }
    },
    "identity": {
      "disableRoleProxy": true,
      "role": "instance"
    }
  },
  {
    "id": 21908,
    "job": "data_quality_orchestration-pauldefusco-banking",
    "type": "airflow",
    "status": "running",
    "user": "pauldefusco",
    "started": "2023-11-29T03:00:01Z",
    "ended": "0001-01-01T00:00:00Z",
    "airflow": {
      "dagID": "CDE_Demo_pauldefusco-banking",
      "dagRunID": "scheduled__2023-11-29T02:55:00+00:00",
      "dagFile": "airflow.py",
      "executionDate": "2023-11-29T02:55:00Z"
    }
  },
  {
    "id": 21909,
    "job": "batch_load-pauldefusco-banking",
    "type": "spark",
    "status": "running",
    "user": "pauldefusco",
    "started": "2023-11-29T03:00:14Z",
    "ended": "0001-01-01T00:00:00Z",
    "mounts": [
      {
        "dirPrefix": "jobCode/",
        "resourceName": "cde_demo_files-pauldefusco-banking"
      }
    ],
    "runtimeImageResourceName": "dex-spark-runtime-ge-data-quality-pauldefusco-banking",
    "spark": {
      "sparkAppID": "spark-3d8a4704418841929d325af0e0190a20",
      "sparkAppURL": "https://58kqsms2.cde-g6hpr9f8.go01-dem.ylcu-atmi.cloudera.site/livy-batch-14907-dyL7LLeM",
      "spec": {
        "file": "jobCode/batch_load.py",
        "driverMemory": "1g",
        "driverCores": 1,
        "executorMemory": "1g",
        "executorCores": 1
      }
    },
    "identity": {
      "disableRoleProxy": true,
      "role": "instance"
    }
  }
]
```

List all successful airflow jobs created by user pauldefusco that started after 3 am UTC on 11/29/23:

```
% cde run list --filter 'type[eq]airflow' --filter 'status[eq]succeeded' --filter 'user[eq]pauldefusco' --filter 'started[gte]2023-11-29T03'
[
  {
    "id": 21908,
    "job": "data_quality_orchestration-pauldefusco-banking",
    "type": "airflow",
    "status": "succeeded",
    "user": "pauldefusco",
    "started": "2023-11-29T03:00:01Z",
    "ended": "2023-11-29T03:03:01Z",
    "airflow": {
      "dagID": "CDE_Demo_pauldefusco-banking",
      "dagRunID": "scheduled__2023-11-29T02:55:00+00:00",
      "dagFile": "airflow.py",
      "executionDate": "2023-11-29T02:55:00Z"
    }
  }
]
```

List all CDE Resources will return all types ("python-env", "files", "custom-runtime-image"):

```
% cde resource list
[
  {
    "name": "BankingPyEnv",
    "type": "python-env",
    "status": "pending-build",
    "created": "2023-11-07T21:27:16Z",
    "modified": "2023-11-07T21:27:16Z",
    "retentionPolicy": "keep_indefinitely",
    "pythonEnv": {
      "pythonVersion": "python3",
      "type": "python-env"
    }
  },
  {
    "name": "CDEPY_DEMO_APAC",
    "type": "files",
    "status": "ready",
    "signature": "5d216f3c4a10578ffadba415b13022d9e383bc22",
    "created": "2023-11-15T03:33:36Z",
    "modified": "2023-11-15T03:33:36Z",
    "retentionPolicy": "keep_indefinitely"
  },
  {
    "name": "dex-spark-runtime-sedona-geospatial",
    "type": "custom-runtime-image",
    "status": "ready",
    "created": "2023-11-28T23:51:11Z",
    "modified": "2023-11-28T23:51:11Z",
    "retentionPolicy": "keep_indefinitely",
    "customRuntimeImage": {
      "engine": "spark3",
      "image": "pauldefusco/dex-spark-runtime-3.2.3-7.2.15.8:1.20.0-b15-sedona-geospatial-003"
    }
  }
]
```

List all CDE Resources named "myScripts":

```
% cde resource list --filter 'name[eq]myScripts'
[
  {
    "name": "myScripts",
    "type": "files",
    "status": "ready",
    "signature": "17f820aacdad9bbd17a24d78a5b93cd0ec9e467b",
    "created": "2023-11-28T23:31:31Z",
    "modified": "2023-11-29T01:48:12Z",
    "retentionPolicy": "keep_indefinitely"
  }
]
```

List all CDE Resources of type Python Environment:

```
% cde resource list --filter 'type[eq]python-env'
[
  {
    "name": "BankingPyEnv",
    "type": "python-env",
    "status": "pending-build",
    "created": "2023-11-07T21:27:16Z",
    "modified": "2023-11-07T21:27:16Z",
    "retentionPolicy": "keep_indefinitely",
    "pythonEnv": {
      "pythonVersion": "python3",
      "type": "python-env"
    }
  }
]
```

### References:

[CDE CLI list command syntax reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-list-flag-reference.html)
