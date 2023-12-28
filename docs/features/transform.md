# Data Transformation

In most of cases, trino ETL queries will be used to transform data in Chango, 
and all the ETL jobs of trino queries need to be integrated with workflow like `Azkaban`.


## Chango Query Exec

`Chango Query Exec` is a REST application to execute trino ETL queries to transform data in Chango. 
You just send trino ETL queries to `Chango Query Exec` through REST using such as `curl`. 

All the ETL jobs need to be integrated with workflow engine like `Azkaban`. Let's see the following pictures to run trino ETL queries integrated with workflow `Azkaban` using `dbt` and `Chango Query Exec`.


<img width="700" src="../../images/query-exec/dbt-etl.png" />

A workflow DAG task runs dbt model to run trino ETL queries.


<img width="700" src="../../images/query-exec/query-exec-etl.png" />

In similar way, a workflow DAG task sends trino ETL queries to `Chango Query Exec` through REST to run trino ETL queries.


As seen, both of them have similar approach to run trino ETL queries, but `Chango Query Exec` has several advantages, for example.

- send trino ETL queries via REST simply without additional tool and library installation.
- just use the same trino ETL queries which you already used to explore data with your BI tools.
- easy way to integrate with workflow engine.

See <a href="../../user-guide/query-exec">Data Transformation Using Chango Query Exec</a>.
