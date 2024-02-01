# Using Pandas UDFs in Cloudera Data Engineering

### Create the CDE Python Resource

You can create CDE Resources of type Python in order to associate a CDE Spark Job with a set of Python libraries. In this example we will create a Python Environment in order to use Pandas and Scipy in our CDE Spark Job.

### Step by Step Instructions

##### 0. Review Spark Application Code

Pandas UDF leverage Apache Arrow in order to accellerate the distribution of Python code to Spark Executors. Pandas UDF can be created inside a PySpark application script, for example in a CDE Spark Job application code, and often require running your code in an environment where Python libraries have been installed.

Open the "cde_udf_job.py" script located in the "cde_jobs" folder and review the code. Notice the following syntax:

Lines 86-94: A PySpark UDF is created as a function. Notice the use of the "@udf()" decorator.

```
from pyspark.sql.functions import udf

# Use udf to define a row-at-a-time udf
@udf('double')
# Input/output are both a single double value
def plus_one(v):
      return v + 1

df = df.withColumn('ID2', plus_one(df.ID))
```

Lines 100-110: A Pandas UDF is also created as a function with a decorator. Unlike the previous example, Pandas, SciPy and any other Python libraries are imported.  

```
import pandas as pd
from scipy import stats
from pyspark.sql.functions import pandas_udf, PandasUDFType

@pandas_udf('double')
def cdf(v):
    return pd.Series(stats.norm.cdf(v))

df = df.withColumn('cumulative_probability', cdf(df.ID))

df.show()
```

In the next steps we will build the CDE Spark Job along with the required Python environment in order for these Python libraries to be available to the CDE Pod where the job is run.

##### 1. Create CDE Python Resource

```
cde resource create --name myPyEnv --type python-env
```

```
cde resource upload --name myPyEnv --local-path cde_resources/requirements.txt
```

You must wait for the environment build to complete before moving on to the next step. Sometimes the build can take up to a couple of minutes.

##### 2. Create CDE Spark Job

```
cde resource create --name myScripts --type files
```

```
cde resource upload --name myScripts --local-path cde_jobs/cde_udf_job.py
```

```
cde job create --name myUdfJob --type spark --application-file cde_udf_job.py --mount-1-resource myScripts --mount-2-resource myPyEnv
```

```
cde job run --name myUdfJob
```


# References

[CDE Python Resources](https://docs.cloudera.com/data-engineering/1.5.0/use-resources/topics/cde-python-virtual-env.html)
