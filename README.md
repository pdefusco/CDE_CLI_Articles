# CDE CLI Articles

This repository outlines some useful Cloudera Data Engineering (CDE) CLI commands that can help you accellerate your work with Spark and Airflow. It holds the code and artifacts related to the following Cloudera Community Articles:

* Working with CDE Resources:
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Working-with-CDE-Files-Resources/ta-p/379891)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDERESOURCES.md)

* Working with Apache Iceberg in CDE Sessions:
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Working-with-Iceberg-in-CDE-Spark-Sessions/ta-p/379892)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDESESSIONSICEBERG.md)

* Helpful Monitoring Examples:
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Efficiently-Monitoring-Jobs-Runs-and-Resources-with-the-CDE/ta-p/379893)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDELISTFILTERS.md)

* Working with Spark Job Parameters with the CDE CLI
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Working-with-CDE-Spark-Job-Parameters-in-Cloudera-Data/ta-p/380792)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDESPARKJOBPARAMETERS.md)

* Working with the Hadoop API in CDE
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Cloud-Storage-File-System-Operations-with-the-Hadoop-API-in/ta-p/384213)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDEHADOOPAPI.md)

* Working with Apache Sedona in CDE Sessions:
  1. [Cloudera Community Article](WIP)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDESESSIONSEDONA.md)

* Working with Pandas UDFs in Cloudera Data Engineering:
  1. [Cloudera Community Article] Coming Soon...
  2. [Code](WIP)

* Using CDE Resources in CDE Sessions:
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Using-CDE-Resources-in-CDE-Sessions/ta-p/387834)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDESESSIONSRESOURCES.md)

* Using Jars in CDE Jobs:
  1. [Cloudera Community Article](https://community.cloudera.com/t5/Community-Articles/Simplify-Spark-Submit-JAR-Dependency-Management-with/ta-p/393014)
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDEUSINGJARS.md)

* Working with Spark NLP in CDE Sessions:
  1. [Cloudera Community Article]()
  2. [Code]()

* Using Spark-Connect in CDE Sessions:
  1. [Cloudera Community Article]()
  2. [Code]()

## On CDE and the CLI

CDE is the Cloudera Data Engineering Service, a containerized managed service for Cloudera Data Platform designed for Large Scale Batch and Streaming Pipelines with Spark, Airflow and Iceberg. It allows you to submit batch jobs to auto-scaling virtual clusters. As a Cloud-Native service, CDE enables you to spend more time on your applications, and less time on infrastructure.

CDE allows you to create, manage, and schedule Apache Spark jobs without the overhead of creating and maintaining Spark clusters. With CDE, you define virtual clusters with a range of CPU and memory resources, and the cluster scales up and down as needed to run your Spark workloads, helping to control your cloud costs.

CDE provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, etc.

## Requirements

The following are required in order to reproduce these commands in your CDE environment:

* A CDE Service on version 1.19.0 or above.
* A working installation of the CDE CLI. Please follow [these instructions](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html) to install the CLI.

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
