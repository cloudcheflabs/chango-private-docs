# Data Transformation Using Standalone Chango Query Exec

`Chango Query Exec` is used to run trino ETL queries and spark SQL ETL queries through REST simply.

## Date Functions

Date Functions supported by `Chango Query Exec` can be found <a href="../../user-guide/query-exec/#date-functions">here</a>.


## Install Standalone Chango Query Exec

> You can install `Chango Query Exec` for yourself with downloading standalone package as below, 
> but it is easier to install `Chango Query Exec` using Admin UI.

Download `Chango Query Exec` distribution.
```agsl
curl -L -O https://github.com/cloudcheflabs/chango-libs/releases/download/chango-private-deps/chango-query-exec-2.7.0-linux-x64.tar.gz
```

Untar and move to query exec directory.
```agsl
tar zxvf chango-query-exec-2.7.0-linux-x64.tar.gz;
cd chango-query-exec-2.7.0-linux-x64/
```


Start `Chango Query Exec`.

```agsl
bin/start-chango-query-exec.sh
```

You can see the log file of `Chango Query Exec`.

```agsl
tail -f /tmp/chango-private-query-exec-logs/query-exec.log
```

To stop `Chango Query Exec`.

```agsl
bin/stop-chango-query-exec.sh
```

## Send Simple ETL Query

Create a file `exec-queries.sql`.

```agsl
-- create schema.
CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;

-- ctas.
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.metrics
AS
SELECT
    *
FROM postgresql.public.metrics 
where format_datetime(ts, 'YYYY-MM-dd HH:mm:ss.SSS') < '#{ nowMinusFormatted(0, 0, 0, 0, 10, 0, "YYYY-MM-dd HH:mm:ss.SSS") }'
limit 10000
;
```


Send queries to `Chango Query Exec`.

```agsl
curl -XPOST \
http://localhost:28291/v1/trino/exec-query \
-d "uri=chango-private-1.chango.private:18080" \
-d "user=trino" \
-d "ssl=false" \
-d "query=$(cat ./exec-queries.sql)" \
;
```

Parameters for this API.

- `uri`: Trino URI.
- `user`: Trino User.
- `password`: Trino Password. Optional.
- `query`: Trino Queries.
- `ssl`: SSL enabled or not. Default is `true`. Optional.
- `ssl_verification`: SSL Verification enabled or not. Default is `false`. Optional.



If `Chango Query Exec` is installed by `Chango Admin`, you need to add http header `Authorization: Bearer` with `Chango Credential` and 
parameter `password` like this.

```agsl
export ACCESS_TOKEN=<chango-credential>

curl -XPOST -H "Authorization: Bearer $ACCESS_TOKEN" \
http://<chango-query-exec-endpoint>/v1/trino/exec-query \
-d "uri=chango-private-1.chango.private:443" \
-d "user=trino" \
-d "password=<trino-password>" \
-d "query=$(cat ./exec-queries.sql)" \
;
```


## Send Query Flow

Let's create more complex queries like DAG, `exec-flow.yaml`.

```agsl
uri: chango-private-1.chango.private:18080
user: trino
ssl: false
sslVerification: false
queries:
  - id: query-0
    description: |-
      Query 0 description
    depends: NONE
    query: |-
      -- drop table.
      DROP TABLE iceberg.iceberg_db.metrics
  - id: query-1
    description: |-
      Query 1 description
    depends: query-0
    query: |-
      -- create schema.
      CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;
  - id: query-2
    description: |-
      Query 2 description
    depends: query-0
    query: |-
      -- create schema.
      CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;
  - id: query-3
    description: |-
      Query 3 description
    depends: query-1,query-2
    query: |-
      -- ctas.
      CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.metrics
      AS
      SELECT
      *
      FROM postgresql.public.metrics
      where format_datetime(ts, 'YYYY-MM-dd HH:mm:ss.SSS') < '#{ nowMinusFormatted(0, 0, 0, 0, 10, 0, "YYYY-MM-dd HH:mm:ss.SSS") }'
      limit 10000
      ;
  - id: query-4
    description: |-
      Query 4 description
    depends: query-3
    query: |-
      -- create schema.
      CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;

      -- ctas.
      CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.metrics
      AS
      SELECT
      *
      FROM postgresql.public.metrics
      where format_datetime(ts, 'YYYY-MM-dd HH:mm:ss.SSS') < '#{ nowMinusFormatted(0, 0, 0, 0, 50, 0, "YYYY-MM-dd HH:mm:ss.SSS") }'
      limit 10000
      ;
```

The individual queries used in this example are not useful. You should take a look at how to construct DAG like query flow in this example.

You can define the following properties except `queries` in the query flow.

- `uri`: Trino URI.
- `user`: Trino User.
- `password`: Trino Password. Optional.
- `ssl`: SSL enabled or not. Default is `true`. Optional.
- `sslVerification`: SSL Verification enabled or not. Default is `false`. Optional.


Send query flow to `Chango Query Exec`.

```agsl
curl -XPOST \
http://localhost:28291/v1/trino/exec-query-flow \
-d "flow=$(cat ./exec-flow.yaml)" \
;
```

If `Chango Query Exec` is installed by `Chango Admin`, you need to replace trino connection part in `exec-flow.yaml` with the following.

```agsl
uri: <trino-gateway-endpoint>
user: <trino-user>
password: <trino-password>
```

In addition, you need to add http header `Authorization: Bearer` with `Chango Credential`.

```agsl
export ACCESS_TOKEN=<chango-credential>

curl -XPOST -H "Authorization: Bearer $ACCESS_TOKEN" \
http://<chango-query-exec-endpoint>/v1/trino/exec-query-flow \
-d "flow=$(cat ./exec-flow.yaml)" \
;
```