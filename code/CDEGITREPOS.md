## Working with Git Repos in CDE

You can now use Git repositories to collaborate, manage project artifacts, and promote applications from lower to higher environments. Cloudera currently supports Git providers such as GitHub, GitLab, and Bitbucket.

Repository files can be accessed when you create a Spark or Airflow job. You can then deploy the job and use CDE's centralized monitoring and troubleshooting capabilities to tune and adjust your workloads.

### Step by Step Instructions

Create a git repository with basic code artifacts. A git repository to test this code was already created for this article and is located [here](https://github.com/pdefusco/cde_git_repo). In order to complete these steps you can either create your own or a fork and then clone it locally.

Using the CDE CLI, run the following commands.

Create a CDE repository:

```
cde repository create --name myArtifacts --url https://github.com/pdefusco/cde_git_repo --branch main
```

Validate repository in CDE repositories page.

![alt text](img/cderepos1.png)

Navigate to your git repository (or your fork) and make a modification to the PySpark script. Remove lines 59-62:

```
# A list of Rows. Infer schema from the first row, create a DataFrame and print the schema
rows = [Row(name="John", age=19), Row(name="Smith", age=23), Row(name="Sarah", age=18)]
some_df = spark.createDataFrame(rows)
some_df.printSchema()
```

Then commit your changes via git commands:

```
git add simple-pyspark-sql.py
git commit -m "removed lines 59-62"
git push
```

Using the CDE CLI, commit the new changes:

```
cde repository sync --name myArtifacts
```
