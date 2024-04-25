# Reading Log Files Using Chango Log

[Chango Log](https://github.com/cloudcheflabs/chango-log) is a log agent to read local log files and send logs 
to Chango to analyze logs.

## Create Iceberg Table

Before sending logs to Chango, you need to create Iceberg table for logs with trino clients like `Superset`.

```agsl
-- create schema.
CREATE SCHEMA IF NOT EXISTS iceberg.logs_db;


-- create iceberg table.
CREATE TABLE iceberg.logs_db.logs (
    day varchar,
    fileName varchar,
    filePath varchar,
    hostAddress varchar,
    hostName varchar,
    lineNumber bigint,
    message varchar,
    month varchar,
    readableTs varchar,
    ts bigint,
    year varchar 
)
WITH (
    partitioning=ARRAY['year', 'month', 'day'],
    format = 'PARQUET'
);
```

## Install Chango Log

Download `Chango Log` distribution package.

```agsl
curl -L -O https://github.com/cloudcheflabs/chango-log/releases/download/1.0.1/chango-log-1.0.1-linux-x64.tar.gz;
```

Untar Chango Log file and move to Chango Log directory.

```agsl
tar zxvf chango-log-1.0.1-linux-x64.tar.gz;
cd chango-log-1.0.1-linux-x64;
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


If the query is run with `Superset`, then it looks like this.

<img width="900" src="../../images/user-guide/select-logs.png" />












