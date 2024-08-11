# Data Transformation

In most of cases, trino ETL queries and spark SQL ETL queries will be used to transform data in Chango, 
and all the ETL jobs of trino and spark SQL queries need to be integrated with workflow like `Azkaban`.


## Chango Query Exec

`Chango Query Exec` is a REST application to execute trino ETL queries and spark SQL ETL queries to transform data in Chango. 

<img width="900" src="../../images/user-guide/query-exec-trino-spark.png" />

You just send trino and spark SQL ETL queries to `Chango Query Exec` through REST using such as `curl`. 

`Chango Query Exec` has several advantages, for example.

- send trino and spark SQL ETL queries via REST simply without additional tool and library installation.
- just use the same trino and spark SQL ETL queries which you already used to explore data with your BI tools.
- easy way to integrate with workflow engine.

See <a href="../../user-guide/query-exec">Data Transformation Using Chango Query Exec</a>.
