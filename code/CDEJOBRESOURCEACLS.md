# CDE Job and Resource ACLs

User Access Management allows you to assign the roles and define who can access and manage the Cloudera Data Engineering environment, Virtual Clusters, and the artifacts by defining the access levels and permissions for a particular user.

Among other benefits, this brings CDE Admins and Users granular ACL's at the CDE Job and Resource level. CDE Users can leverage the CDE CLI to assign jobs and roles to specific roles in the cluster, thus preventing others from unexpectedly deleting or modifying spark pipelines unwantedly.

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
