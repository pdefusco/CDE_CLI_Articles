# Spark Connect in CDE

1. Launch CDE Session and Download Tarballs
2. Create Virtual Env and Install Packages
3. Run Spark Connect commands


### 1. Launch CDE Session

Launch CDE Session of type SparkConnect

```
% cde session create \
  --name spark-connect-session \
  --type spark-connect
```

### 2. Create Virtual Env and Install Packages

Create and activate the Python Virtual Environment.

```
python3 -m venv cdeconnect
. cdeconnect/bin/activate
```

Next, install the CDESparkConnectSession package.

```
pip install [***CDECONNECT TARBALL***]
pip install [***PYSPARK TARBALL***]
```

Finally, make sure to do the following:

```
# Downgrade numpy version:
pip install numpy==1.26.4

# Upgrade cmake:
pip install --upgrade cmake

# Install pyarrow but not the latest:
pip install pyarrow==14.0.0
```

### 3. Run Spark Connect Commands

Launch the Python Shell

```
% python
```

Import the CDESparkConnectSession package.

```
>> from cde import CDESparkConnectSession
>> spark = CDESparkConnectSession.builder.sessionName('connect-session').get()
```

Create example Iceberg table.

```
>> spark.sql("""
CREATE TABLE default.example (
    id bigint,
    data string)
USING iceberg;
""")
```

Insert some values into the Iceberg table.

```
>> spark.sql("""
INSERT INTO default.example VALUES (1, 'a'), (2, 'b')
""")
```

Show Iceberg table contents.

```
spark.sql("SELECT * FROM default.example").show()
+---+----+
| id|data|
+---+----+
|  1|   a|
|  2|   b|
+---+----+
```
