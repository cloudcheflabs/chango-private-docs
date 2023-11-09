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
  <version>2.0.0</version>
</dependency>
```

You can also download Chango Client jar file to add it to your application classpath.

```agsl
curl -L -O https://github.com/cloudcheflabs/chango-client/releases/download/2.0.0/chango-client-2.0.0-executable.jar;
```

## Create Iceberg Table before Sending JSON Events

Before sending json as streaming events to `Chango Data API` server, Iceberg table needs to be created beforehand with 
trino clients like Trino CLI and Apache Superset.

It is good practice that Iceberg table for streaming events needs to be partitioned with date, for example, `year`, `month` and `day`.
In addition, timestamp column like `ts` also needs to be added to Iceberg table for compacting small files.

Especially, in order to compact small files and add partitioning in Iceberg table in Chango, you need to follow the rules.

> The name of timestamp column must be `ts` whose type is `bigint` which is equivalent to `long` in java. 
> Column names `year`, `month`, `day` will be used as partitioning columns.

- `ts`:  the number of **milliseconds** since 1970-01-01 00:00:00.
- `year`: year with the format of `yyyy` which is necessary for partitioning.
- `month`: month of the year with the format of `MM` which is necessary for partitioning.
- `day`: day of the month with the format of `dd` which is necessary for partitioning.

For example, create `logs` table with partitioning and timestamp.

```
-- create iceberg schema.
CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;


-- create iceberg table.
CREATE TABLE iceberg.iceberg_db.logs (
    day varchar,
    level varchar,
    message varchar,
    month varchar,
    ts bigint,
    year varchar 
)
WITH (
    partitioning=ARRAY['year', 'month', 'day'],
    format = 'PARQUET'
);
```

> **_NOTE:_** The sequence  of table column names in **lower case** must be **alphanumeric in ascending order**.

You can create Iceberg table with Superset provided by Chango Private like this.

<img width="900" src="../../images/user-guide/create-table.png" />




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

Let's see the full codes to send JSON events.

```java
import co.cloudcheflabs.chango.client.component.ChangoClient;
import com.cloudcheflabs.changoprivate.example.util.JsonUtils;
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
            for(int i = 0; i < MAX; i++) {
                Map<String, Object> map = new HashMap<>();

                DateTime dt = DateTime.now();

                String year = String.valueOf(dt.getYear());
                String month = padZero(dt.getMonthOfYear());
                String day = padZero(dt.getDayOfMonth());
                long ts = dt.getMillis(); // in milliseconds.

                map.put("level", "INFO");
                map.put("message", "any log message ... [" + count + "]");
                map.put("ts", ts);
                map.put("year", year);
                map.put("month", month);
                map.put("day", day);

                String json = JsonUtils.toJson(map);

                try {
                    // send json.
                    changoClient.add(json);
                } catch (Exception e) {
                    e.printStackTrace();
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
                }

                count++;
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

> Take a look at the value of `ts` must be the number of **milliseconds** since 1970-01-01 00:00:00.

## Run Query in Iceberg Table for Streaming Events

You can run queries in iceberg table `logs` to which streaming events are inserted.

```agsl
-- select with partitioning columns.
select *, from_unixtime(ts/1000) from iceberg.iceberg_db.logs where year = '2023' and month = '11' and day = '07' limit 1000;
```

<img width="900" src="../../images/user-guide/select-logs.png" />

