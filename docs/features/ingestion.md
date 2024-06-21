# Data Ingestion

In order to ingest data to iceberg tables in Chango, Chango provides the followings.

- Load Files to Iceberg Tables using `Chango SQL Procedure`.
- Send Streaming Events / Log / CDC through `Chango Streaming`.


## Load Files to Iceberg Tables using SQL

`Chango SQL Procedure` is an easy way to load external files like `CSV`, `JSON`, `Parquet` and `ORC` located in s3 compatible object storage 
to iceberg tables in Chango.

Take a look at <a href="../../user-guide/load-files">Load Files to Iceberg Tables using SQL</a> for more details.



## Streaming


External event streaming application can insert streaming events into iceberg tables in chango without building streaming platform and writing streaming jobs.


### No Streaming Platform, No Streaming Jobs

<img width="700" src="../../images/chango-streaming.png" />

If you want to insert streaming events like user behavior events, logs, IoT events to data lakehouses, you need to build event streaming platform like kafka
and write streaming jobs like spark streaming jobs in most cases. But in chango, you don't have to do so.

<img width="400" src="../../images/chango-streaming2.png" />

External streaming application can insert streaming events to iceberg tables in chango directly without streaming platform and streaming jobs.

## Aggregate Logs

[Chango Log](https://github.com/cloudcheflabs/chango-log) is a log agent to read local log files and send logs
to Chango to analyze logs. 
Using `Chango Log`, you can analyze logs from all your distributed logs joining different databases in richer manner realtimely in Chango.
See <a href="../../user-guide/chango-log/">Reading Log Files Using Chango Log</a> for more details.


## Change Data Capture

[Chango CDC](https://github.com/cloudcheflabs/chango-cdc) is Change Data Capture application to catch CDC data of database
and send CDC data to Chango.
You don't need such as Kafka and Kafka Connect cluster to accomplish CDC. See <a href="../../user-guide/chango-cdc/">Change Data Capture Using Chango CDC</a> for more details.


## Upload Files

<img width="400" height="300" src="../../images/chango-ingestion.png" />


As data analytics engineer, you don’t have to struggle with long row Excel to analyze data. SQL is better to analyze data.
External data like CSV, Excel, JSON can be inserted directly to iceberg table in chango,
then you can explore and analyze iceberg tables with trino queries.


