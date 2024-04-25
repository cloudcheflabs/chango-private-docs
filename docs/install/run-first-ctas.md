# Run First Trino Query

This is an example to run queries to Trino through `Chango Trino Gateway`.

## Create Cluster Group

First, you need to create `Cluster Group` in `Trino Gateway`.
Go to `Settings` -> `Trino Gateway`. Click `Create Cluster Group` in `Cluster Groups` section.

<img width="900" src="../../images/run-first/create-cg.png" />

And, enter cluster group name `bi`.
<img width="900" src="../../images/run-first/create-cg-2.png" />

## Create Trino User and Register Trino Cluster

In order to create trino user and register trino cluster, first select cluster group created before.
<img width="900" src="../../images/run-first/select-cg.png" />

Click `Create Trino User` in `Trino Users` section, and enter user name with password.

<img width="900" src="../../images/run-first/create-user.png" />

To register trino cluster, click `Register Trino Cluster` in `Trino Clusters` section. 
Select trino cluster which trino queries will be routed to.

<img width="900" src="../../images/run-first/reg-cluster.png" />


## Create Privileges

To add privileges to the cluster group, go to `Settings` -> `Security`. And select role in `Roles` section.

<img width="900" src="../../images/run-first/select-role.png" />

Add privilege of `*` for READ type.
<img width="900" src="../../images/run-first/create-priv-read.png" />

Add privilege of `*` for WRITE type.
<img width="900" src="../../images/run-first/create-priv-write.png" />

`*` means all data access to Chango is allowed.

But, note that all data access above is not secure, so you need to set the values in fine-grained manner in your production environment, for example.

```agsl
information_schema.*	   READ
tpch.*                     READ
iceberg.*                  READ
iceberg.*                  WRITE
```

> **_NOTE:_** `information_schema.*` needs to be added for trino connector of superset.


## Get Endpoint of Chango Trino Gateway

You need endpoint of `Chango Trino Gateway` to which clients will connect to run queries.

Go to `Components` -> `Chango Trino Gateway`. Get the endpoint in `Endpoint` section.
<img width="900" src="../../images/run-first/trino-gw-endpoint.png" />


## Run First CTAS Query with Superset

To run queries to trino, Superset will be used.

### Login to Superset

To move to `Superset` UI, go to `Components` -> `Apache Superset`. Click UI URL to move to Superset UI.

First, login as `admin` with default password `SupersetPass1#`.

### Add Trino Database

You need to add trino database in superset.

You need to enter trino url in `SQLAlchemy URI *` with the following convention.
```agsl
trino://[user]:[password]@[trino-gateway-endpoint-without-scheme]
```
For example, you can enter like this.
```agsl
trino://trino:trino123@chango-private-3.chango.private:443
```
Replace `chango-private-3.chango.private:443` with your trino gateway endpoint without `https://`.

<img width="900" src="../../images/run-first/superset-trino.png" />


And you need to add the following in `Extra` to disable TLS validation.
```agsl
{
    "metadata_params": {},
    "engine_params": {
          "connect_args":{
              "http_scheme": "https",
              "verify": false
        }
     },
    "metadata_cache_timeout": {},
    "schemas_allowed_for_csv_upload": []
}
```
<img width="900" src="../../images/run-first/superset-extra.png" />

Check the options of `Allow CREATE TABLE AS`, `Allow CREATE VIEW AS`, `Allow DML`.

Finally, press `Save`.

### Run CTAS Query


Run CTAS query which selects rows from `tpch.sf1000.lineitem` table and insert them to new created iceberg table `iceberg.iceberg_db.test_ctas`.
```agsl
-- create schema.
CREATE SCHEMA IF NOT EXISTS iceberg.iceberg_db;


-- ctas.
CREATE TABLE IF NOT EXISTS iceberg.iceberg_db.test_ctas 
AS
SELECT
	*
FROM tpch.sf1000.lineitem limit 1000
;
```
<img width="900" src="../../images/run-first/ctas.png" />


And select rows from created iceberg table.
```agsl
-- select.
select * from iceberg.iceberg_db.test_ctas limit 1000;
```
<img width="900" src="../../images/run-first/ctas-2.png" />


Congratulations!


