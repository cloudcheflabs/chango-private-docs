# Data Tranformation Using Chango Query Exec

`Chango Query Exec` is a REST application to execute trino ETL queries requested through REST call to transform 
data in Chango. 

`dbt` is a popular tool to transform data with running trino ETL queries. But `Chango Query Exec` is a more simpler way 
to transform data in Chango. You just send trino queries for ETL jobs to `Chango Query Exec` explicitly through REST using, for example `curl`.
Such ETL jobs using `Chango Query Exec` can be integrated with Workflow like `Azkaban` with ease.



## Date Functions 

There are date functions supported by `Chango Query Exec`.


### nowInMillis()
Current Time in Millis.

```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.ctas
AS
SELECT
*
FROM anycatalog.anyschema.anytable
WHERE ts < #{ nowInMillis() }
;
```

### nowFormatted(String format)
Current date with date format.

```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.ctas
AS
SELECT
*
FROM anycatalog.anyschema.anytable
WHERE dateAt < '#{ nowFormatted("YYYY-MM-dd") }'
;
```


### nowPlusInMillis(int years, int months, int days, int hours, int minutes, int weeks)
Add time amount to current date.


```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.ctas
AS
SELECT
*
FROM anycatalog.anyschema.anytable
WHERE where ts < #{ nowPlusInMillis(0, 0, 1, 0, 0, 0) }
;
```


### nowPlusFormatted(int years, int months, int days, int hours, int minutes, int weeks, String format)

Formatted plus date time.

```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.ctas
AS
SELECT
*
FROM anycatalog.anyschema.anytable
WHERE where dateAt >= '#{ nowPlusFormatted(0, 0, 0, 0, 0, 0, "YYYY-MM-dd") }' and dateAt < '#{ nowPlusFormatted(0, 0, 1, 0, 0, 0, "YYYY-MM-dd") }'
;
```

### nowMinusInMillis(int years, int months, int days, int hours, int minutes, int weeks)


Substract time amount from current time.

```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.ctas
AS
SELECT
*
FROM anycatalog.anyschema.anytable
WHERE where ts >= #{ nowMinusInMillis(0, 0, 1, 0, 0, 0) }
;
```


### nowMinusFormatted(int years, int months, int days, int hours, int minutes, int weeks, String format)

Formatted minus date time.

```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.ctas
AS
SELECT
*
FROM anycatalog.anyschema.anytable
WHERE where dateAt >= '#{ nowMinusFormatted(0, 0, 1, 0, 0, 0, "YYYY-MM-dd") }' and dateAt < '#{ nowMinusFormatted(0, 0, 0, 0, 0, 0, "YYYY-MM-dd") }'
;
```

## Send Simple ETL Query

This is a simple ETL query file called `exec-queries.sql`.

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

It will create iceberg schema and CTAS table. 

Take a look at the function of `nowMinusFormatted(0, 0, 0, 0, 10, 0, "YYYY-MM-dd HH:mm:ss.SSS")` 
which will be replaced with the date before 10 minutes of current time by `Chango Query Exec`.


Send this simple trino query to `Chango Query Exec` using `curl`.

```agsl
export ACCESS_TOKEN=<access-token>

curl -XPOST -H "Authorization: Bearer $ACCESS_TOKEN" \
http://<chango-query-exec-endpoint>/v1/trino/exec-query \
-d "uri=<trino-gateway-endpoint>" \
-d "user=<trino-user>" \
-d "password=<trino-password>" \
-d "query=$(base64 -w 0 ./exec-queries.sql)" \
;
```

- `<access-token>`: Chango Credential. See <a href="../../user-guide/cred">Get Chango Credential</a>.
- `<trino-gateway-endpoint>`: Chango Trino Gateway Endpoint without https scheme.
- `<trino-user>`: Trino User.
- `<trino-password>`: Trino Password.
- `<chango-query-exec-endpoint>`: Chango Query Exec Endpoint without http scheme.


## Send Query Flow

You may send DAG like query flow to `Chango Query Exec`.

Let's create query flow file `exec-flow.yaml`.

```agsl
uri: <trino-gateway-endpoint>
user: <trino-user>
password: <trino-password>
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

This is a query flow example. You don't have to see the details of the individual queries.

You need define unique ids for `queries[*].id`, and you may add dependencies with `queries[*].depends`. 


Send query flow to `Chango Query Exec`.

```agsl
curl -XPOST -H "Authorization: Bearer $ACCESS_TOKEN" \
http://<chango-query-exec-endpoint>/v1/trino/exec-query-flow \
-d "flow=$(base64 -w 0 ./exec-flow.sql)" \
;
```




