# Run Spark SQL ETL Jobs with Azkaban

Spark SQL ETL queries can be sent to `Chango Query Exec` through REST to run spark SQL ETL query jobs in `Chango Spark Thrift Server` with ease.

<img width="900" src="../../images/user-guide/query-exec-trino-spark.png" />

Here, we will learn how to integrate Spark SQL ETL jobs with Azkaban workflow in Chango.

## Passwordless SSH Connection from Azkaban Executor Nodes to Flow Query Execution Node

`Flow Query Execution Node` is where all the files like flow query files, azkaban project files and query runner shell files in this example below will be created. We are going to use SSH to run remote shell files to run Spark SQL ETL jobs.
In order to access `Flow Query Execution Node` from azkaban executor nodes without password prompt, follow <a href="../../user-guide/spark-azkaban/#password-less-ssh-access-to-spark-node-from-azkaban-executor-nodes" >this instruction</a>.


## Create Flow of Chango Query Exec

The following is an example flow file called `load-parquet-to-iceberg.yaml` which will be sent to `Chango Query Exec` to run Saprk SQL queries through `Chango Spark Thrift Server`.

```agsl
uri: <sts-uri>
user: <sts-token>
queries:
  - id: query-0
    description: |-
      Create iceberg table if not exists.
    depends: NONE
    query: |-
      CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.test_proc_import_parquet_mergeinto (
          orderkey BIGINT,
          partkey BIGINT,
          suppkey BIGINT,
          linenumber INT,
          quantity DOUBLE,
          extendedprice DOUBLE,
          discount DOUBLE,
          tax DOUBLE,
          returnflag STRING,
          linestatus STRING,
          shipdate DATE,
          commitdate DATE,
          receiptdate DATE,
          shipinstruct STRING,
          shipmode STRING,
          comment STRING)
      USING iceberg
      TBLPROPERTIES (
          'format' = 'iceberg/PARQUET',
          'format-version' = '2',
          'write.format.default' = 'PARQUET',
          'write.metadata.delete-after-commit.enabled' = 'true',
          'write.metadata.previous-versions-max' = '100',
          'write.parquet.compression-codec' = 'zstd')
  - id: query-1
    description: |-
      Load parquet files in external s3 into iceberg table.
    depends: query-0
    query: |-
      -- import parquet file in external s3 with the mode of merge.
      PROC iceberg.system.import (
          source => 's3a://mykidong/temp-external-mergeinto-parquet-path/#{ nowMinusFormatted(0, 0, 1, 0, 0, 0, "YYYY-MM-dd") }',
          s3_access_key => 'any-access-key',
          s3_secret_key => 'any-secret-key',
          s3_endpoint => 'any-endpoint',
          s3_region => 'any-region',
          file_format => 'parquet',
          action => 'MERGE',
          id_columns => 'orderkey,partkey,suppkey',
          target_table => 'iceberg.iceberg_db.test_proc_import_parquet_mergeinto'
      )
```

- `<sts-uri>`: URI of chango spark thrift server.
- `<sts-token>`: Chango credential to connect Chango Spark Thrift Server.

Table `iceberg.iceberg_db.test_proc_import_parquet_mergeinto` will be created if not exists like this.

```agsl
      CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.test_proc_import_parquet_mergeinto (
          orderkey BIGINT,
          partkey BIGINT,
          suppkey BIGINT,
          linenumber INT,
          quantity DOUBLE,
          extendedprice DOUBLE,
          discount DOUBLE,
          tax DOUBLE,
          returnflag STRING,
          linestatus STRING,
          shipdate DATE,
          commitdate DATE,
          receiptdate DATE,
          shipinstruct STRING,
          shipmode STRING,
          comment STRING)
      USING iceberg
      TBLPROPERTIES (
          'format' = 'iceberg/PARQUET',
          'format-version' = '2',
          'write.format.default' = 'PARQUET',
          'write.metadata.delete-after-commit.enabled' = 'true',
          'write.metadata.previous-versions-max' = '100',
          'write.parquet.compression-codec' = 'zstd')
```

After that, parquet files in external s3 will be loaded to iceberg table in Chango incrementally using `Chango SQL Procedure`.

```agsl
      PROC iceberg.system.import (
          source => 's3a://mykidong/temp-external-mergeinto-parquet-path/#{ nowMinusFormatted(0, 0, 1, 0, 0, 0, "YYYY-MM-dd") }',
          s3_access_key => 'any-access-key',
          s3_secret_key => 'any-secret-key',
          s3_endpoint => 'any-endpoint',
          s3_region => 'any-region',
          file_format => 'parquet',
          action => 'MERGE',
          id_columns => 'orderkey,partkey,suppkey',
          target_table => 'iceberg.iceberg_db.test_proc_import_parquet_mergeinto'
      )
```

Take a look at the function of `nowMinusFormatted(0, 0, 1, 0, 0, 0, "YYYY-MM-dd")` 
which will be interpreted as yesterday, for example, `2024-08-06` in `Chango Query Exec`. See <a href="../../user-guide/query-exec/">Date Functions</a> for more details.
If parquet files are saved in the path of `s3a://mykidong/temp-external-mergeinto-parquet-path/<yyyy-MM-dd>` everyday, the above import procedure will load parquet files from the appropriate date path.


## Create Shell File to Send Flow Queries to Chango Query Exec

You can send flow queries to `Chango Query Exec` simply using `curl`. 
The following is an example shell file called `hive-query-exec.sh` to send flow queries to `Chango Query Exec`.

```agsl
#!/bin/sh

set -e

export CONFIG_PATH=NA
export FLOW_PATH=NA

for i in "$@"
do
case $i in
    --config-path=*)
    CONFIG_PATH="${i#*=}"
    shift
    ;;
    --flow-path=*)
    FLOW_PATH="${i#*=}"
    shift
    ;;
    *)
          # unknown option
    ;;
esac
done

echo "CONFIG_PATH: $CONFIG_PATH"
echo "FLOW_PATH: $FLOW_PATH"

export ACCESS_TOKEN=NA
export CHANGO_QUERY_EXEC_URL=NA

# get properties file.
while IFS='=' read -r key value
do
    key=$(echo $key | tr '.' '_')
    eval ${key}=\${value}
done < "$CONFIG_PATH"

ACCESS_TOKEN=${accessToken}
CHANGO_QUERY_EXEC_URL=${changoQueryExecUrl}

#echo "ACCESS_TOKEN: $ACCESS_TOKEN"
echo "CHANGO_QUERY_EXEC_URL: $CHANGO_QUERY_EXEC_URL"

# request.
http_response=$(curl -sS -o response.txt -w "%{response_code}" -XPOST -H "Authorization: Bearer $ACCESS_TOKEN" \
$CHANGO_QUERY_EXEC_URL/v1/hive/exec-query-flow \
--data-urlencode "flow=$(cat $FLOW_PATH)" \
)
if [ $http_response != "200" ]; then
    echo "Failed!"
    cat response.txt
    exit 1;
else
    echo "Succeeded."
    cat response.txt
    exit 0;
fi
```

In addition to shell file, you can create a configuration file called `config.properties` like this.

```agsl
changoQueryExecUrl=http://cp-3.chango.private:8009
accessToken=<chango-credential>

```

Then, you can run query exec runner shell file as follows, for example.

```agsl
/home/spark/chango-etl-example/runner/hive-query-exec.sh \
--config-path=/home/spark/chango-etl-example/configure/config.properties \
--flow-path=/home/spark/chango-etl-example/etl/import/load-parquet-to-iceberg.yaml \
;
```

## Create Azkaban Project 

You need to create flow file called `metrics.flow` of Azkaban like this.

```agsl
---
config:
  failure.emails: admin@your-domain.com

nodes:
  - name: start
    type: noop

  - name: "load-parquet-to-iceberg"
    type: command
    config:
      command: |-
        ssh spark@cp-1.chango.private " \
        /home/spark/chango-etl-example/runner/hive-query-exec.sh \
        --config-path=/home/spark/chango-etl-example/configure/config.properties \
        --flow-path=/home/spark/chango-etl-example/etl/import/load-parquet-to-iceberg.yaml \
        ;
        "
    dependsOn:
      - start

  - name: end
    type: noop
    dependsOn:
      - "load-parquet-to-iceberg"

```

This azkaban flow file shows DAG to run remote shell files which send query flow to `Chango Query Exec`.

Create a metadata file called `flow20.project` for azkaban project.

```agsl
azkaban-flow-version: 2.0
```

> All the files here created for now should be managed by source control system like git.


Create Azkaban project file as zip.

```agsl
# package azkaban project file.
zip load.zip load.flow flow20.project 

# move to /tmp.
mv load.zip /tmp
```

Upload azkaban project file on `Flow Query Execution Node` where `Azkaban CLI` needs to be installed.

```agsl
sudo su - azkabancli;
source venv/bin/activate;
azkaban upload \
-c \
-p load \
-u azkaban:<azkban-password>@http://cp-1.chango.private:28081 \
/tmp/load.zip \
;
```

- `<azkban-password>` is azkaban password. Default password is `azkaban`.
