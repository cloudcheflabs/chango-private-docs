# Chango Data Ingestion

Chango Private provides components like `Chango Data API` and `Chango Streaming` to insert external data to Chango with ease.


## Upload Files


<img width="400" height="300" src="../../images/chango-ingestion.png" />


As data analytics engineer, you don’t have to struggle with long row Excel to analyze data. SQL is better to analyze data. 
External data like CSV, Excel, JSON can be inserted directly to iceberg table in chango, 
then you can explore and analyze iceberg tables with trino queries.




## Streaming


External event streaming application can insert streaming events into iceberg tables in chango without building streaming platform and writing streaming jobs.


### No Streaming Platform, No Streaming Jobs

<img width="700" src="../../images/chango-streaming.png" />

If you want to insert streaming events like user behavior events, logs, IoT events to data lakehouses, you need to build event streaming platform like kafka
and write streaming jobs like spark streaming jobs in most cases. But in chango, you don't have to do so.

<img width="400" src="../../images/chango-streaming2.png" />

External streaming application can insert streaming events to iceberg tables in chango directly without streaming platform and streaming jobs.

