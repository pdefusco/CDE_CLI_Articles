# CDE CLI Articles

This repository outlines some useful Cloudera Data Engineering (CDE) CLI commands that can help you accellerate your work with Spark and Airflow. It holds the code and artifacts related to the following Cloudera Community Articles:

* Working with CDE Resources:
  1. Cloudera Community Article
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDERESOURCES.md)

* Working with CDE Sessions:
  1. Cloudera Community Article
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDESESSIONS.md)

* Helpful Filtering Examples:
  1. Cloudera Community Article
  2. [Code](https://github.com/pdefusco/CDE_CLI_Articles/blob/main/code/CDELISTFILTERS.md)

## On CDE and the CLI

CDE is the Cloudera Data Engineering Service, a containerized managed service for Cloudera Data Platform designed for Large Scale Batch Pipelines with Spark, Airflow and Iceberg. It allows you to submit batch jobs to auto-scaling virtual clusters. As a Cloud-Native service, CDE enables you to spend more time on your applications, and less time on infrastructure.

CDE allows you to create, manage, and schedule Apache Spark jobs without the overhead of creating and maintaining Spark clusters. With CDE, you define virtual clusters with a range of CPU and memory resources, and the cluster scales up and down as needed to run your Spark workloads, helping to control your cloud costs.

Cloudera Data Engineering (CDE) provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, and so on.

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
