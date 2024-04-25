# Maintain Iceberg Tables

Everytime committing data to iceberg tables, many files will be created like metadata, 
snapshots which need to be maintained, for example.

- remove old metadata files.
- expire snapshots.
- compact small files.

Chango provides ways to maintain iceberg tables with ease.

## Iceberg Maintenance Overview

<img width="700" src="../../images/user-guide/maintain-iceberg.png" />

`Chango Spark SQL Runner` and `Chango Spark Thrift Server` are spark streaming jobs running on spark cluster 
and will execute spark sql queries sent by clients through REST or JDBC/Thrift.
In particular, iceberg maintenance spark sql queries need to be sent to either `Chango Spark SQL Runner` or `Chango Spark Thrift Server` 
to maintain iceberg tables in Chango.

`Chango Spark SQL Runner` exposes REST API to which clients can send spark sql queries through REST, 
`Chango Spark Thrift Server` exposes JDBC/Thrift interface to which clients can send spark sql queries through JDBC/Thrift.

All the incoming queries in `Chango Spark SQL Runner` and `Chango Spark Thrift Server` will be checked by `Chango Authorizer` 
if the queries is allowed to be run. This access control provided by Chango is RBAC with fain-grained access control like 
catalog, schema and table level to Iceberg.

Take a note that `Chango Spark SQL Runner` and `Chango Thrift Server` will connect to `Chango REST Catalog` as data catalog.



## Create Chango Credential

Before sending spark sql queries, you need to create Chango credential.
First, create a role like `spark-sql` and credential and privileges.

<img width="900" src="../../images/user-guide/sql-rbac.png" />

Privileges need to be added like this.

```agsl
iceberg.*               READ
iceberg.*               WRITE
```


## Send Spark SQL Queries to Chango Spark SQL Runner

Clients like `curl` can send iceberg maintenance spark queries to `Chango Spark SQL Runner` through REST.
For example, the spark sql queries below are small files compaction, snapshot expiration and old metadata files removal 
will be sent.

```agsl
### request.
# access token for authorizer.
export ACCESS_TOKEN=eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJzcGFyay1zcWxkN2JkMDllNmM3Y2Q0ZDQ5OTI2NDRhYTViYzc4ZTExNCIsImV4cCI6MTcyMTg2NTU5OSwiaWF0IjoxNzE0MDAyMzY1fQ.y-JvgmeKkUcE6V538B9O2989fE-8IauYLNAQC4eOmziW7B_taeN9OdHGXZzTwrWYEd4vC3lo2R0EAjSh9IDp_w

# spark queries.
cat <<EOF > queries.sql
-- compact small files.
CALL iceberg.system.rewrite_data_files(table => 'iceberg.iceberg_db.test_ctas',strategy => 'binpack', options => map('min-input-files','2'));

-- expire snapshot.
CALL iceberg.system.expire_snapshots('iceberg.iceberg_db.test_ctas');

-- remove old metadata files.
ALTER TABLE iceberg.iceberg_db.test_ctas SET TBLPROPERTIES ('write.metadata.delete-after-commit.enabled'='true','write.metadata.previous-versions-max'='100');
EOF

# request.
curl -XPOST \
-H "Authorization: Bearer $ACCESS_TOKEN" \
-H "Content-Type: application/x-www-form-urlencoded" \
http://<spark-sql-runner-host>:29080/v1/spark-sql/execute \
-d "query=$(cat ./queries.sql)" \
;
```

`<spark-sql-runner-host>` needs to be replaced with the current driver host of Chango Spqrk SQL Runner.


## Send Spark SQL Queries to Chango Spark Thrift Server

Chango Spark Thrift Server exposes JDBC/Thrift interface to run spark sql queries. 
For example, superset can connect using Thrift, and java applications can connect using JDBC.

### Connect Chango Spark Thrift Server using Superset

First, create Hive Connector in Superset.

<img width="900" src="../../images/user-guide/superset-hive-conn.png" />

Add the following URL as `SQLAlchemy URI` in superset ui.
```agsl
hive://<sts-host>:10000/iceberg
```

`<sts-host>` needs to be replaced with the current driver host of spark thrift server.

Check `Allow CREATE TABLE AS`, `Allow CREATE VIEW AS` and `Allow DML`.

And you need to enter the following json value to `Extra`.

```agsl
{
   "metadata_params":{
   },
   "engine_params":{
      "connect_args":{
         "username":"<chango-credential>",
         "auth": "NONE"
      }
   },
   "metadata_cache_timeout":{  
   },
   "schemas_allowed_for_csv_upload":[
   ]
}
```

`<chango-credential>` needs to be set with the chango credential created previously.

And, run iceberg maintenance spark sql queries as follows.

```agsl
-- compact small files.
CALL iceberg.system.rewrite_data_files(table => 'iceberg.iceberg_db.test_ctas',strategy => 'binpack', options => map('min-input-files','2'));

-- expire snapshot.
CALL iceberg.system.expire_snapshots('iceberg.iceberg_db.test_ctas');

-- remove old metadata files.
ALTER TABLE iceberg.iceberg_db.test_ctas SET TBLPROPERTIES ('write.metadata.delete-after-commit.enabled'='true','write.metadata.previous-versions-max'='100');
```

<img width="900" src="../../images/user-guide/superset-hive-sql.png" />


### Connect Chango Spark Thrift Server using Beeline

If spark with spark thrift server support is installed, beeline can be used to connect Chango Spark Thrift Server.

```agsl
export ACCESS_TOKEN=<chango-credential>

$SPARK_HOME/bin/beeline -u jdbc:hive2://<sts-host>:10000/iceberg -n $ACCESS_TOKEN
```

`<chango-credential>` and `<sts-host>` need to be replaced.



### Connect Chango Spark Thrift Server using JDBC

Java applications can send iceberg maintenance spark sql queries to Chango Spark Thrift Server using JDBC.

First, add hive jdbc dependency.

```agsl
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>2.3.9</version>
</dependency>
```
The following java codes shows to send iceberg snapshot expiration query.

```java
        String url = "jdbc:hive2://<sts-host>:10000/iceberg";
        String token = "<chango-credential>";
        String query = "CALL iceberg.system.expire_snapshots('iceberg.iceberg_db.test_ctas')";

        // properties
        Properties properties = new Properties();
        properties.setProperty("user", token);

        Connection con = DriverManager.getConnection(url, properties);
        Statement stmt = con.createStatement();
        boolean ret = stmt.execute(query);
```

`<chango-credential>` and `<sts-host>` need to be replaced.
