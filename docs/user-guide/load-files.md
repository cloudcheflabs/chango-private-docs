# Load Files to Iceberg Tables using SQL

You may load files like `CSV`, `JSON`, `Parquet` and `ORC` in S3 compatible object storages to iceberg tables in Chango using SQL, 
for which Chango provides `Chango SQL Procedure`.

`Chango Spark Thrift Server` to which hive clients like superset can connect through JDBC/Thrift 
and `Chango Spark SQL Runner` which exposes REST API can be used to run `Chango SQL Procedure`.


## Load Files to Iceberg Tables using Import Procedure

With `import` procedure, you may load files to iceberg tables in Chango.

```agsl
PROC iceberg.system.import (
    source => 's3a://any-bucket/any-path',
    s3_access_key => 'any access key',
    s3_secret_key => 'any secret key',
    s3_endpoint => 'any endpoint',
    s3_region => 'any region',
    file_format => 'json',
    id_columns => 'id_1, id_2',
    action => 'MERGE',
    target_table => 'iceberg.test_db.test'
)
```

- `s3_access_key`: s3 access key.
- `s3_secret_key`: s3 secret key.
- `s3_endpoint`: s3 endpoint.
- `s3_region`: s3 region.
- `file_format`: one of `csv`, `json`, `parquet`, `orc`.
- `source`: file source path. currently, only `s3a` scheme is supported. It must be started with `s3a`.
- `action`: one of `MERGE`, `APPEND`, `OVERWRITE`.
- `id_columns`: comma separated id columns of iceberg table. If action is `MERGE`, it is mandatory.
- `target_table`: target iceberg table with fully qualified name. The convention is `iceberg.<schema>.<table>`.

With the action value of `MERGE` which needs id_columns for merging, the external file will be merged into iceberg table. 
In addition, in order to append to existing iceberg table, the action value of `APPEND` needs to be used, 
and the action value of `OVERWRITE` needs to be used to replace iceberg table dataset with new loaded file data.

For example, to load the files in S3 to iceberg table in Chango using SQL, first you need to create iceberg table `iceberg.iceberg_db.test_proc_import_parquet` using for example, 
superset which connects to `Chango Spark Thrift Server` in Chango like this.

```agsl
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.test_proc_import_parquet (
    baseproperties STRUCT<eventtype: string,
                       ts: long,
                       uid: string,
                       version: string>,
    itemid string,
    price long,
    quantity long
)
USING iceberg
PARTITIONED BY (itemid);
```

After that, run the following import procedure with, for example superset.

```agsl
PROC iceberg.system.import (
    source => 's3a://mykidong/temp-external-parquet-path',
    s3_access_key => 'xxx',
    s3_secret_key => 'xxx',
    s3_endpoint => 'https://xxx.compat.objectstorage.ap-singapore-1.oraclecloud.com',
    s3_region => 'ap-singapore-1',
    file_format => 'parquet',
    action => 'APPEND',
    target_table => 'iceberg.iceberg_db.test_proc_import_parquet'
)
```

The parquet file located in `s3a://mykidong/temp-external-parquet-path` will be loaded into iceberg table `iceberg.iceberg_db.test_proc_import_parquet` in Chango.


## Export Iceberg Table to File using Export Procedure

If you want to export iceberg table dataset to files, use the following `export` procedure in Chango.

```agsl
PROC iceberg.system.export (
    source_table => 'iceberg.test_db.test',
    file_format => 'json',
    target_path => 's3a://any-bucket/any-path',
    s3_access_key => 'any access key',
    s3_secret_key => 'any secret key',
    s3_endpoint => 'any endpoint',
    s3_region => 'any region'
)
```

- `source_table`: source iceberg table with fully qualified name. The convention is `iceberg.<schema>.<table>`.
- `target_path`: target file path. currently, only `s3a` scheme is supported. It must be started with `s3a`.

For example, run the following `export` procedure in order to export iceberg table dataset to parquet file in S3.

```agsl
-- export iceberg table to parquet file in external s3.
PROC iceberg.system.export (
    source_table => 'iceberg.iceberg_db.test_proc_import_parquet',
    file_format => 'parquet',
    target_path => 's3a://chango-bucket/temp-proc-export-another-s3-parquet',
    s3_access_key => 'xxx',
    s3_secret_key => 'xxx',
    s3_endpoint => 'https://s3.ap-northeast-2.amazonaws.com',
    s3_region => 'ap-northeast-2'
)
```

The dataset of iceberg table `iceberg.iceberg_db.test_proc_import_parquet` in Chango will be 
exported to `s3a://chango-bucket/temp-proc-export-another-s3-parquet` as parquet file.

