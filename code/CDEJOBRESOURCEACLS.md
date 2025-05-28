# CDE Job and Resource ACLs

A resource in Cloudera Data Engineering is a named collection of files used by a job or a session. Resources can include application code, configuration files, custom Docker images, and Python virtual environment specifications (requirements.txt).

CDE 1.24 introduces granular ACL's to prevent developers from utilizing and modifying resources in use by other users' Spark and Airflow jobs. The attached examples rely on the CDE CLI.

### Background Info on CDE and the CLI

CDE is the Cloudera Data Engineering Service, a containerized managed service for Cloudera Data Platform designed for Large Scale Batch and Streaming Pipelines with Spark, Airflow and Iceberg. It allows you to submit batch jobs to auto-scaling virtual clusters. As a Cloud-Native service, CDE enables you to spend more time on your applications, and less time on infrastructure.

CDE allows you to create, manage, and schedule Apache Spark jobs without the overhead of creating and maintaining Spark clusters. With CDE, you define virtual clusters with a range of CPU and memory resources, and the cluster scales up and down as needed to run your Spark workloads, helping to control your cloud costs.

CDE provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, etc.

### Background Info on CDE Resources

The resource types supported by Cloudera Data Engineering are files, python-env, and custom-runtime-image.

***files***
An arbitrary collection of files that a job can reference. The application code for the job, including any necessary configuration files or supporting libraries, can be stored in a files resource. Files can be uploaded to and removed from a resource as needed.

***python-env***
A defined virtual Python environment that a job runs in. The only file that can be uploaded to a python-env resource is a requirements.txt file. When you associate a python-env resource with a job, the job runs within a Python virtual environment built according to the requirements.txt specification.

***custom-runtime-image***
A Docker container image. When you run a job using a custom-runtime-image resource, the executors that are launched use your custom image.

### Background Info on User Access Management

User Access Management allows you to assign the roles and define who can access and manage the Cloudera Data Engineering environment, Virtual Clusters, and the artifacts by defining the access levels and permissions for a particular user.

Among other benefits, this brings CDE Admins and Users granular ACL's at the CDE Job and Resource level. CDE Users can leverage the CDE CLI to assign jobs and roles to specific roles in the cluster, thus preventing others from unexpectedly deleting or modifying spark pipelines unwantedly.

### Requirements

The following are required in order to reproduce these commands in your CDE environment:

* A CDE Service on version 1.24 or above.
* A working installation of the CDE CLI. Please follow [these instructions](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html) to install the CLI.

### Example

Create a CDE Files Resource with ACL

```
cde resource create \
  --name filesResource \
  --type files \
  --acl-full-access-group group1 \            
  --acl-full-access-user user1 \    
  --acl-view-only-group group2 \               
  --acl-view-only-user user2
```

```
cde resource upload \
  --name myProperties \
  --local-path cde_jobs/propertiesFile_1.conf \
  --local-path cde_jobs/propertiesFile_2.conf \
  --local-path cde_jobs/sparkJob.py
```

Create a CDE Job with ACL

```
cde job create \
  --name sampleJob \
  --type spark \
  --mount-1-resource myProperties \
  --application-file sparkJob.py \
  --executor-cores 2 \
  --executor-memory "2g"
```

```
cde job run --name myPySparkJob\
--arg MY_DB\
--arg CUSTOMER_TABLE\
--arg propertiesFile_1.conf
```

## Summary and Next Steps

Cloudera Data Engineering (CDE) provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, and so on.

In this article we have reviewed some advanced use cases for the CLI. If you are using the CDE CLI you might also find the following articles and demos interesting:

* [Installing the CDE CLI](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html)
* [Simple Introduction to the CDE CLI](https://github.com/pdefusco/CDE_CLI_Simple)
* [CDE CLI Demo](https://github.com/pdefusco/CDE_CLI_demo)
* [CDE Concepts](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-concepts.html)
* [CDE CLI Command Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-reference.html)
* [CDE CLI Spark Flag Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-spark-flag-reference.html)
* [CDE CLI Airflow Flag Reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-airflow-flag-reference.html)
* [CDE CLI list command syntax reference](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli-list-flag-reference.html)
* [CDE Jobs API Reference](https://docs.cloudera.com/data-engineering/cloud/jobs-rest-api-reference/index.html)
