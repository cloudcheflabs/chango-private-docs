# Send Streaming Events

`Chango Data API` server will collect all the incoming streaming events and produce them to kafka. 
`Chango Streaming` will consume events from kafka and save events to Iceberg table in Chango directly. 

That is, if you send streaming events to `Chango Data API` server, then all the streaming events will be inserted to Chango automatically.
`Chango Client` is used to send streaming events to `Chango Data API` server with ease.

> **_NOTE:_** Before sending streaming events, make sure that `Chango Data API` and `Chango Straming` are installed.

## Add Chango Client Library to Classpath

You can add the following maven dependency to your project.

```agsl
<dependency>
  <groupId>co.cloudcheflabs.chango</groupId>
  <artifactId>chango-client</artifactId>
  <version>2.0.5</version>
</dependency>
```

You can also download Chango Client jar file to add it to your application classpath.

```agsl
curl -L -O https://github.com/cloudcheflabs/chango-client/releases/download/2.0.5/chango-client-2.0.5-executable.jar;
```

## Create Iceberg Table before Sending JSON Events

Before sending json as streaming events to `Chango Data API` server, Iceberg table needs to be created beforehand with 
hive clients like Apache Superset which connects to `Chango Spark Thrift Server`.

It is good practice that Iceberg table for streaming events needs to be partitioned with timestamp.
Timestamp column  `ts` needs to be added to Iceberg table. 
Actually, with `ts` column, Chango will compact small data, manifest, and position delete files and expire snapshots to improve query performance.

For example, create `logs` table with hidden partitioning with the column of `ts`.

```
-- create iceberg schema.
CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;


-- create iceberg table.
CREATE TABLE iceberg.iceberg_db.logs (
    level string,
    message string,
    ts TIMESTAMP_LTZ
)
USING iceberg
;

-- add hidden partitions.
ALTER TABLE iceberg.iceberg_db.logs ADD PARTITION FIELD hour(ts);
```

> **_NOTE:_** The sequence  of table column names in **lower case** must be **alphanumeric in ascending order**.



## Send JSON Events with Chango Client

It is very simple to use Chango Client API in your code. 

You can construct `ChangoClient` instance like this.
```java
        ChangoClient changoClient = new ChangoClient(
                token,
                dataApiServer,
                schema,
                table,
                batchSize,
                interval
        );
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


And send JSON events.

```java
        // send json.
        changoClient.add(json);
```

Let's see the following example codes to send JSON events.

```java
import co.cloudcheflabs.chango.client.component.ChangoClient;
import co.cloudcheflabs.chango.client.util.JsonUtils;
import org.joda.time.DateTime;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.Map;

public class SendLogsToDataAPI {

    private static Logger LOG = LoggerFactory.getLogger(SendLogsToDataAPI.class);

    @Test
    public void sendLogs() throws Exception {
        String token = System.getProperty("token");
        String dataApiServer = System.getProperty("dataApiServer");
        String table = System.getProperty("table");

        int batchSize = 10000;
        long interval = 1000;
        String schema = "iceberg_db";

        ChangoClient changoClient = new ChangoClient(
                token,
                dataApiServer,
                schema,
                table,
                batchSize,
                interval
        );

        long count = 0;
        while (true) {
            int MAX = 50 * 1000;
            for (int i = 0; i < MAX; i++) {
                Map<String, Object> map = new HashMap<>();

                DateTime dt = DateTime.now();

                map.put("level", "INFO");
                map.put("message", "any log message ... [" + count + "]");
                map.put("ts", dt.toString());

                String json = JsonUtils.toJson(map);

                try {
                    // send json.
                    changoClient.add(json);

                    count++;
                } catch (Exception e) {
                    LOG.error(e.getMessage());

                    // reconstruct chango client.
                    changoClient = new ChangoClient(
                            token,
                            dataApiServer,
                            schema,
                            table,
                            batchSize,
                            interval
                    );
                    LOG.info("Chango client reconstructed.");
                    Thread.sleep(1000);
                }
            }
            Thread.sleep(10 * 1000);
            LOG.info("log [{}] sent...", count);
        }
    }

    private String padZero(int value) {
        String strValue = String.valueOf(value);
        if(strValue.length() == 1) {
            strValue = "0" + strValue;
        }
        return strValue;
    }
}
```

> Take a look at the value of `ts` must be timestamp type.


You can also use `add(List<String> jsonList)` method.
```agsl
            List<String> jsonList = ...;
            
            try {
                changoClient.add(jsonList);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
```



If you want to ingest events to Chango transactionally, you need to use transactional chango client like this.

```java
        ChangoClient changoClient = new ChangoClient(
                token,
                dataApiServer,
                schema,
                table,
                batchSize,
                interval,
                transactional
        );
```

> Before you use transactional chango client, you need to install <a href="../../install/install-component/#install-chango-streaming-tx">transactional chango streaming</a>.

