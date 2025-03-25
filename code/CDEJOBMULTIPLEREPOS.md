## CDE Jobs with Multiple CDE Repository Dependencies

## Objective

In this article you will create a CDE Spark Job with dependencies located in multiple CDE Repositories.

## On CDE and the CLI

CDE is the Cloudera Data Engineering Service, a containerized managed service for Cloudera Data Platform designed for Large Scale Batch and Streaming Pipelines with Spark, Airflow and Iceberg. It allows you to submit batch jobs to auto-scaling virtual clusters. As a Cloud-Native service, CDE enables you to spend more time on your applications, and less time on infrastructure.

CDE allows you to create, manage, and schedule Apache Spark jobs without the overhead of creating and maintaining Spark clusters. With CDE, you define virtual clusters with a range of CPU and memory resources, and the cluster scales up and down as needed to run your Spark workloads, helping to control your cloud costs.

CDE provides a command line interface (CLI) client. You can use the CLI to create and update jobs, view job details, manage job resources, run jobs, etc.

## Requirements

The following are required in order to reproduce these commands in your CDE environment:

* A CDE Service on version 1.23.
* A working installation of the CDE CLI. Please follow [these instructions](https://docs.cloudera.com/data-engineering/cloud/cli-access/topics/cde-cli.html) to install the CLI.

## Code

The application code and util are located in two different repositories. Create two CDE Repositories by cloning git repositories.

```
cde repository create \
  --name function_repo \
  --url https://github.com/pdefusco/cde_git_repo_2.git \
  --branch main
```

```
cde repository create \
  --name app_code_repo \
  --url https://github.com/pdefusco/cde_git_repo.git \
  --branch main
```

Optionally sync the repositories as needed:

```
cde repository sync \
  --name function_repo
```

```
cde repository sync \
  --name app_code_repo
```

Create the job by referencing the repositories as if they were CDE Files Resources. In other words, use the `--mount-n-resource` for each repository, as needed:

```
cde job create \
  --name from_multiple_repos \
  --type spark \
  --mount-1-resource app_code_repo \
  --mount-2-resource function_repo \
  --application-file app_code.py \
  --max-executors 4 \
  --executor-cores 2
```

Run the Job:

```
cde job run \
  --name from_multiple_repos
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
* [CDE CLI Articles](https://github.com/pdefusco/CDE_CLI_Articles)
