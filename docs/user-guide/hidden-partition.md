# Create Hidden Partitioned Iceberg Table

There are ways to create hidden partitioned iceberg tables in Chango. You can create them with trino, but trino does not 
support all the features of iceberg at the moment. 
Chango provides `Chango Spark SQL Runner` and `Chango Spark Thrift Server` to execute spark sql which can be iceberg specific queries.
`Chango Spark SQL Runner` and `Chango Spark Thrift Server` provide full features of iceberg, so they are suitable to run iceberg 
specific queries like DDL and iceberg maintenance queries.


This shows how to create hidden partitioned iceberg table using `Chango Spark Thrift Server`.

## Connect Chango Spark Thrift Server using Superset

After installing `Chango Spark Thrift Server`, you can connect `Chango Spark Thrift Server` with Superset.
See <a href="../../user-guide/maintain-iceberg/#connect-chango-spark-thrift-server-using-superset">Connect Chango Spark Thrift Server using Superset</a>.


## Hidden Partitioned Iceberg Table

You can create hidden partitioned iceberg table like this.

```agsl
--hidden partitioned table.
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.hidden_partitioning (
    id long,
    log string,
    ts TIMESTAMP_LTZ
)
USING iceberg
;
-- add hidden partitions.
ALTER TABLE iceberg.iceberg_db.hidden_partitioning ADD PARTITION FIELD day(ts);
```

Column `ts` must be timestamp type.
After creating iceberg table, you need to add partitions using iceberg functions like one of `year()`, `month()`, `day()` or `hour()`.

## Create Hidden Partitioned Iceberg Table using Superset

Run the above iceberg DDL queries using Superset like this.

<img width="900" src="../../images/user-guide/hidden-partition.png" />

Then, you will get table description with the following query.

```agsl
-- describe table.
describe iceberg.iceberg_db.hidden_partitioning;
```

<img width="900" src="../../images/user-guide/describe-hidden-partition.png" />
