# Change Data Capture Using Chango CDC

[Chango CDC](https://github.com/cloudcheflabs/chango-cdc) is Change Data Capture application to catch CDC data of database 
and send CDC data to Chango.

## Build and Package Chango CDC

Embedded Debezium is used by `Chango CDC`. So you need to choose which Debezium version will be used for your database, see the [release notice of Debezium](https://debezium.io/releases/).
After that, choose the concrete maven dependency version of Debezium connector.
For example, maven dependency version of Debezium connector for PostgreSQL can be found [here](https://mvnrepository.com/artifact/io.debezium/debezium-connector-postgres).

Package `Chango CDC` distribution with Debezium maven dependency version. Please note that Java 11 and Maven 3 are required to build `Chango CDC`.
```agsl
git clone -b branch-1.1.0 https://github.com/cloudcheflabs/chango-cdc.git
cd chango-cdc;

export CHANGO_CDC_VERSION=1.1.0
export DEBEZIUM_VERSION=1.9.7.Final

./package-dist.sh \
--version=${CHANGO_CDC_VERSION} \
--debezium.version=${DEBEZIUM_VERSION} \
;
```

## Install Chango CDC

You may use `Chango CDC` package which was built for yourself previously. 
For this example, the pre-built `Chango CDC` will be used. Download Chango CDC.

```agsl
curl -L -O https://github.com/cloudcheflabs/chango-cdc/releases/download/1.1.0/chango-cdc-1.1.0-debezium-1.9.7.Final-linux-x64.tar.gz
```

Untar and move to Chango CDC directory.

```agsl
tar zxvf chango-cdc-1.1.0-debezium-1.9.7.Final-linux-x64.tar.gz
cd chango-cdc-1.1.0-debezium-1.9.7.Final-linux-x64/
```

## Configure Chango CDC

Modify `conf/configuation.yml`. For example, it looks like this.

```agsl
chango:
  token: eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJjZGM0MTFkYzIzMzg2NjU0ZGZkYTdjYjk4OTMzNjA1NWNiNyIsImV4cCI6MTcwNjY1OTE5OSwiaWF0IjoxNzAxMzU2NDEyfQ.-WjO6mpNV5QM5t1jwLmBD8tBuRNOxrzcREU6RqLJtHGD0u_TGi28NWG9lFYA-ZKQ-nDwGbr6Nf_MXaUeeO2VAw
  dataApiUrl: http://chango-private-1.chango.private:80
  schema: cdc_db
  table: student
  batchSize: 10000
  interval: 1000
  tx: true

debezium:
  connector: |-
    name=postgres-connector
    connector.class=io.debezium.connector.postgresql.PostgresConnector
    offset.storage=org.apache.kafka.connect.storage.FileOffsetBackingStore
    offset.storage.file.filename=/tmp/chango-cdc/offset-student.dat
    offset.flush.interval.ms=60000
    topic.prefix=cdc
    schema.history.internal=io.debezium.storage.file.history.FileSchemaHistory
    schema.history.internal.file.filename=/tmp/chango-cdc/schemahistory-student.dat
    database.server.name=postgresql-server
    database.server.id=pg-1
    database.hostname=localhost
    database.port=5432
    database.user=anyuser
    database.password=anypassword
    database.dbname=studentdb
    table.include.list=public.student
```

- `chango.token`: Chango Credential which is necessary to access `Chango Data API`. See <a href="../../user-guide/cred">Get Chango Credential</a>.
- `chango.dataApiUrl`: `Chango Data API` endpoint URL.
- `chango.schema`: Schema of Iceberg catalog in Chango.
- `chango.table`: Iceberg Table in Chango.

In this example, we will use PostgreSQL from which CDC data will be caught and sent to Chango.

- `debezium.connector`: Properties of Debezium connector for PostgreSQL.

If you want to use another database, consult [Debezium Connectors](https://debezium.io/documentation/reference/stable/connectors/index.html).


> **_NOTE:_** If `chango.tx` is set to `true`, then you need to install <a href="../../install/install-component/#install-chango-streaming-tx">transactional chango streaming</a> beforehand.



## Install PostgreSQL

PostgreSQL will be installed as docker container.

Assumed that docker and docker compose are installed on your machine, create docker compose file.

```agsl
cat <<EOF > docker-compose.yml
version: "3.5"

services:
  postgres:
    container_name: postgres
    image: debezium/postgres:9.6
    ports:
      - 5432:5432
    environment:
      - POSTGRES_DB=studentdb
      - POSTGRES_USER=anyuser
      - POSTGRES_PASSWORD=anypassword
EOF
```

Run docker compose.

```agsl
docker-compose up -d;
```

And then, enter the docker container of PostgreSQL to create a table.

```agsl
docker exec -it postgres sh;
```

Connect PostgreSQL database in it.

```agsl
psql -U anyuser -d studentdb;
```

Create table `student`.

```agsl
CREATE TABLE public.student
(
    id integer NOT NULL,
    address character varying(255),
    email character varying(255),
    name character varying(255),
    CONSTRAINT student_pkey PRIMARY KEY (id)
);
```

## Create Iceberg Table

You need to create Iceberg table in Chango using hive clients like `Superset` which connects to `Chango Spark Thrift Server`.

```agsl
-- create iceberg schema.
CREATE SCHEMA IF NOT EXISTS iceberg.cdc_db;


-- create iceberg table.
CREATE TABLE iceberg.cdc_db.student (
    address string,
    email string,
    id long,
    name string,
    op string,
    ts TIMESTAMP_LTZ
)
USING iceberg
;

-- add hidden partitions.
ALTER TABLE iceberg.cdc_db.student ADD PARTITION FIELD year(ts);
ALTER TABLE iceberg.cdc_db.student ADD PARTITION FIELD month(ts);
ALTER TABLE iceberg.cdc_db.student ADD PARTITION FIELD day(ts);
```

In addition to the original fields of PostgreSQL table, fields `ts` and `op` are required for partitioning and small files compaction.
If fields `ts` and `op` exist in the original PostgreSQL table, then `_` will be appended to the original fields of PostgreSQL table.

Take a note that the type of field `id` in original PostgreSQL table is `integer`, but the type of field `id` in Iceberg table in Chango is `long`.

> **_NOTE:_** The sequence  of table column names in **lower case** must be **alphanumeric in ascending order**.


## Run Chango CDC

> **_NOTE:_** Before running Chango CDC, make sure that Chango Data API and Chango Streaming Tx are installed in Chango.

Move to Chango CDC directory, and run Chango CDC.

```agsl
bin/start-chango-cdc.sh
```

You can check the log file `/tmp/chango-cdc/chango-cdc.log`.


If you want to stop Chango CDC, run the following.

```agsl
bin/stop-chango-cdc.sh
```

## Run PostgreSQL CUD Queries

In PostgreSQL docker container, run the CUD queries.

```agsl
INSERT INTO STUDENT(ID, NAME, ADDRESS, EMAIL) VALUES('1','Kidong Lee','Seoul','kidong@example.com');

UPDATE STUDENT SET EMAIL='kidong2@example.com', NAME='Kidong2 Lee' WHERE ID = 1; 

DELETE FROM STUDENT WHERE ID = 1;
```

Check if CDC data has been saved in Iceberg table in Chango with running the following query in `Superset`.

```agsl
select * from iceberg.cdc_db.student order by ts desc;
```




