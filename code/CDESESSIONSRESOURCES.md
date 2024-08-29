## CDE RESOURCES IN SESSIONS

As of version 1.20 CDE introduces the ability to leverage CDE Resources when running CDE Interactive Sessions.

This means that you can leverage files and Python environments when working with your data interactively.

In this example we will leverage both CDE Files and Python resources in order to parse a Yaml file.


Create the CDE Files resource and load a yaml file to it.

```
%cde resource create --type files --name myYamlFiles

%cde resource upload --name myYamlFiles --local-path cdesessionsresources/sample.yaml
```

Create a CDE Python resource and activate the environment.

```
%cde resource create --type python-env --name py_yaml_resource

%cde resource upload --name py_yaml_resource --local-path cdesessionsresources/requirements.txt
```

Allow a moment for the Python resource to build. Check build status in the UI.

Next, launch the CDE Session:

```
%cde session create --name pyyamlSession --type pyspark --python-env-resource-name py_yaml_resource --mount-1-resource myYamlFiles

{
  "name": "pyyamlSession",
  "type": "pyspark",
  "creator": "pauldefusco",
  "created": "2024-05-13T23:47:29Z",
  "mounts": [
    {
      "dirPrefix": "/",
      "resourceName": "myYamlFiles"
    }
  ],
  "lastStateUpdated": "2024-05-13T23:47:29Z",
  "state": "starting",
  "interactiveSpark": {
    "id": 4,
    "driverCores": 1,
    "executorCores": 1,
    "driverMemory": "1g",
    "executorMemory": "1g",
    "numExecutors": 1,
    "pythonEnvResourceName": "py_yaml_resource"
  }
}
```

Open the PySpark shell and execute the rest of the following code from there:

```
%cde session interact --name pyyamlSession

Starting REPL...
Waiting for the session to go into an available state...
Connected to Cloudera Data Engineering...
Press Ctrl+D (i.e. EOF) to exit
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\
      /_/

Type in expressions to have them evaluated.

>>>
```

Import the PyYaml Python module:

```
>>> import yaml
>>> from yaml import load, dump
>>> from yaml import Loader, Dumper
```

Load the data from the files resource:

```
>>> myYamlDocument = "/app/mount/sample.yaml"
```

Parse the file with PyYaml:

```
>>> with open(myYamlDocument) as stream:
      data = yaml.load(stream, Loader=Loader)
      print(data)
{'name': "Martin D'vloper", 'job': 'Developer', 'skill': 'Elite', 'employed': True, 'foods': ['Apple', 'Orange', 'Strawberry', 'Mango'], 'languages': {'perl': 'Elite', 'python': 'Elite', 'pascal': 'Lame'}, 'education': '4 GCSEs\n3 A-Levels\nBSc in the Internet of Things\n'}
```
