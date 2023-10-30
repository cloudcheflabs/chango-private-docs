# Send Streaming Events

`Chango Data API` server will collect all the incoming streaming events and produce them to kafka. 
`Chango Streaming` will consume events from kafka and save events to Iceberg table in Chango directly. 

That is, if you send streaming events to `Chango Data API` server, then all the streaming events will be inserted to Chango automatically.
`Chango Client` is used to send streaming events to `Chango Data API` server with ease.

## Add Chango Client Library to Classpath

You can add the following maven dependency to your project.

```agsl
<dependency>
  <groupId>co.cloudcheflabs.chango</groupId>
  <artifactId>chango-client</artifactId>
  <version>2.0.0</version>
</dependency>
```

You can also download Chango Client jar file to add it to your application classpath.

```agsl
curl -L -O https://github.com/cloudcheflabs/chango-client/releases/download/2.0.0/chango-client-2.0.0-executable.jar;
```

## Create Iceberg Table before Sending JSON Events

Before sending json as streaming events to `Chango Data API` server, Iceberg table needs to be created beforehand, for example.

```
-- create iceberg schema.
CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;

-- create iceberg table.
CREATE TABLE iceberg.iceberg_db.test_iceberg (
    baseproperties ROW(eventtype varchar, 
                       ts bigint, 
                       uid varchar, 
                       version varchar), 
    itemid varchar, 
    price bigint, 
    quantity bigint 
)
WITH (
    partitioning = ARRAY['itemid'],
    format = 'PARQUET'
);
```

> **_NOTE:_** The sequence  of table column names in lower case must be alphanumeric in ascending order.


## Send JSON Events to Chango Data API Server

It is very simple to use Chango Client API in your code. Just add the following to send json events.

```java
        String token = "...";
        String dataApiServer = "...";
        int batchSize = 10000; 
        long interval = 1000;
        String schema = "iceberg_db";
        String table = "test_iceberg";
        
        ChangoClient changoClient = new ChangoClient(
                token,
                dataApiServer,
                schema,
                table,
                batchSize,
                interval
        );
        
        String json = "...";
        
        changoClient.add(json);
```

- `token` : Data access credential issued by `Chango Authorizer`.
- `dataApiServer` : Endpoint URL of `Chango Data API`.
- `schema`: Target Iceberg schema which needs to be created before sending json data to chango.
- `table`: Target Iceberg table which also needs to be created beforehand.
- `batchSize` : The size of json list which will be sent to chango in batch mode and in gzip format.
- `interval` : Json data will be queued internally in chango client. The queued json list will be sent in this period whose unit is milliseconds.

In order to get `token`, see <a href="../../user-guide/cred">Get Chango Credential</a>.

To get the endpoint of `Chango Data API`, go to `Components` -> `Chango Data API`.

- Get URL in `Endpoint` section.