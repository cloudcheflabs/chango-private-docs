# Realtime Analytics

<img width="700" src="../../images/realtime/realtime-analytics.png" />

Chango Private provides several solutions for realtime analytics. 
Streaming events from external applications, distributed logs and CDC(Change Data Capture) data will be inserted to Iceberg tables 
in Chango and will be analyzed in realtime for example using `Trino` provided by Chango.



## Streaming

<img width="400" src="../../images/chango-streaming2.png" />

External streaming application can insert streaming events to Chango using [Chango Client](https://github.com/cloudcheflabs/chango-client).
See <a href="../../user-guide/streaming/">Send Streaming Events</a> for more details.


## Aggregate Logs

<img width="700" src="../../images/realtime/log.png" />

[Chango Log](https://github.com/cloudcheflabs/chango-log) is a log agent to read local log files and send logs
to Chango to analyze logs. 
See <a href="../../user-guide/chango-log/">Reading Log Files Using Chango Log</a> for more details.


## Change Data Capture

<img width="700" src="../../images/realtime/cdc.png" />

[Chango CDC](https://github.com/cloudcheflabs/chango-cdc) is Change Data Capture application to catch CDC data of database
and send CDC data to Chango. See <a href="../../user-guide/chango-cdc/">Change Data Capture Using Chango CDC</a> for more details.




