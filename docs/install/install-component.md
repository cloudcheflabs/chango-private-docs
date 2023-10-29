# Install Chango Components

There are several components supported by Chango Private. Let's see the following Data Lakehouse Platform built by Chango Private.

<img width="900" src="../../images/architecture/lakehouse-platform.png" />

In `Ingestion` layer:

- `Spark` and `Trino` with `dbt` will be used as data integration tool.
- `Kafka` is used as event streaming platform to handle streaming events.
- `Chango Data API` and `Chango Streaming` will be used to insert incoming streaming events to Chango directly.

In `Storage` layer:

- Chango Private supports Apache Ozone as object storage by default and external S3 compatible object storage like AWS S3, MinIO, OCI Object Storage.
- Default storage format is `Iceberg` table format used for data warehousing in Chango.

In `Transformation` layer:

- `Spark` and `Trino` with `dbt` will be used to run ETL jobs.

In `Analytics` layer:

- `Trino` will be used as query engine to explore all the data in Chango.
- `BI` tools like `Apache Superset` will connect to `Trino` to run queries through `Chango Trino Gateway`.

In `Management` layer:

- `Azkaban` will be used as workflow. All the batch jobs like ETL will be integrated with `Azkaban`.
- `Chango REST Catalog` will be used as data catalog in Chango.
- Chango Private supports storage security to control data access based on RBAC in Chango. `Chango Authorizer` will be used for it.
- `Chango Trino Gateway` is an implementation of Trino Gateway concept. `Chango Trino Gateway` provides several features like authentication, authorization, smart query routing(routing to less exhausted trino clusters), trino cluster activation/deactivation. For more details, see [TODO: Chango Trino Gateway].

