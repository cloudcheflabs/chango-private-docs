# Transactional Streaming

<a href="../../install/install-component/#install-chango-streaming-tx">Chango Streaming Tx</a> is a spark streaming job 
to support exactly-once delivery in contrast to 
<a href="../../install/install-component/#install-chango-streaming">Chango Streaming</a> which supports at-most once delivery.

`Chango Streaming Tx` like `Chango Streaming` is a streaming consumer and works with `Chango Data API` which is a streaming producer. So,
the combination of `Chango Streaming Tx` like `Chango Streaming` and `Chango Data API` is called `Chango Ingestion`. 

Chango provides the following tools as chango clients to ingest messages to Chango quickly and easily.

- Chango Client
- Chango CLI
- Chango Log
- Chango CDC

These chango clients can send messages which will be saved to iceberg tables transactionally in Chango.

## Use Transactional Chango Client API

You need to use the following Chango Client API in java codes to send messages transactionally.

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

## Configure Chango CLI

You need to add `--tx` argument to Chango CLI to upload JSON files transactionally.

```
chango upload json local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table test_iceberg \
--file /home/chango/multi-line-json.json \
--batch-size 150000 \
--tx \
;
```

## Configure Chango Log

Configure `conf/configuration.yml` with setting `tx` to `true` to send logs transactionally in `Chango Log`.

```agsl
chango:
  token: any-chango-credential
  dataApiUrl: http://any-data-api-endpoint
  schema: logs_db
  table: logs
  batchSize: 10000
  interval: 1000
  tx: true
```

## Configure Chango CDC

Configure `conf/configuration.yml` with setting `tx` to `true` to send CDC data transactionally in `Chango CDC`.

```agsl
chango:
  token: any-chango-credential
  dataApiUrl: http://any-data-api-endpoint
  schema: cdc_db
  table: student
  batchSize: 10000
  interval: 1000
  tx: true
```