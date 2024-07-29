# Reading Log Files Using Chango Log

[Chango Log](https://github.com/cloudcheflabs/chango-log) is a log agent to read local log files and send logs 
to Chango to analyze logs.

## Create Iceberg Table

Before sending logs to Chango, you need to create Iceberg table for logs with hive clients like `Superset` which connects to `Chango Spark Thrift Server`.

```agsl
-- create schema.
CREATE SCHEMA IF NOT EXISTS iceberg.logs_db;

-- create iceberg table.
CREATE TABLE IF NOT EXISTS iceberg.logs_db.logs (
    fileName string,
    filePath string,
    hostAddress string,
    hostName string,
    lineNumber long,
    message string,
    ts TIMESTAMP_LTZ
)
USING iceberg
;

-- add hidden partitions.
ALTER TABLE iceberg.logs_db.logs ADD PARTITION FIELD hour(ts);
```

## Install Chango Log

Download `Chango Log` distribution package.

```agsl
curl -L -O https://github.com/cloudcheflabs/chango-log/releases/download/1.1.0/chango-log-1.1.0-linux-x64.tar.gz;
```

Untar Chango Log file and move to Chango Log directory.

```agsl
tar zxvf chango-log-1.1.0-linux-x64.tar.gz;
cd chango-log-1.1.0-linux-x64;
```

## Configure Chango Log

Modify `conf/configuration.yml`.

```agsl
chango:
  token: any-chango-credential
  dataApiUrl: http://any-data-api-endpoint
  schema: logs_db
  table: logs
  batchSize: 10000
  interval: 1000
  tx: false

logs:
  - path: /log-dir-1
    file: rest*.log
  - path: /log-dir-2
    file: admin*.log
  - path: /log-dir-3

task:
  log:
    interval: 20000
    threads: 3

rocksdb:
  directory: /tmp/rocksdb
```

You need to configure the following properties to send logs to Chango.

- `chango.token`: Chango Credential which is necessary to access `Chango Data API`. See <a href="../../user-guide/cred">Get Chango Credential</a>.
- `chango.dataApiUrl`: `Chango Data API` endpoint URL.

You can configure multiple log directories in which all the log files will be read.

- `logs[*].path`: Log directory path.
- `logs[*].file`: Log file name pattern. All the log files matched with this pattern will be read in the log directory. If this property is omitted, all the log files will be read without pattern matching.


## Run Chango Log

> **_NOTE:_** Before running Chango Log, make sure that Chango Data API and Chango Streaming are installed in Chango.

Start `Chango Log` to read local log files and send logs to Chango.

```agsl
bin/start-chango-log.sh
```

You can see the logs of `Chango Log` in the path `/tmp/chango-log/chango-log.log`.


To stop `Chango Log`, run the following.

```agsl
bin/stop-chango-log.sh
```

## Run Queries to Analyze Logs

Run queries to analyze logs saved in Iceberg table `logs`.

For example.

```agsl
select * from iceberg.logs_db.logs where fileName = 'admin.log' order by lineNumber asc limit 100000;
```













