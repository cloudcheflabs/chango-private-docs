# Install Chango Components

There are several components supported by Chango.

See <a href="../../intro/intro">What is Chango?</a> for more details.

> **_NOTE:_** If Chango initialization failed or you want to reset Chango, then you need to move to 
> `http://[admin-host]:8123/cp-reset.html` to reset Chango.

## Initialize Chango

If you have not initialized Chango, you will get the following picture to initialze Chango.

<img width="900" src="../../images/init/init-chango.png" />

There are mandatory Chango Components like `MySQL`, `Object Storage`, `Chango Authorizer`, and 
`Chango REST Catalog` which must be installed when Chango is initialized. 
The other optional compoents can be installed after Chango initialization.

### Configure Hosts and SSH Private Key

All the `Chango Component Nodes` need to be registered, and SSH private key on `Chango Admin Node` need to be added to access `Chango Component Nodes` from 
the host of `Chango Admin Node`.

> **_NOTE:_** Take a note that you need to register all the nodes of `Chango Component Nodes` **except** `Chango Admin Node`.

Get SSH private key on `Chango Admin Node` with the following command, and paste it to the text box `SSH Private Key`.

```agsl
cat ~/.ssh/id_rsa
```

<img width="900" src="../../images/init/hosts-ssh-key.png" />


### Configure LVM

Raw disks attached to the `Chango Component Nodes` in comma separated list need to be added to mount as logical volume.

If the disks attached are same on all the nodes, use this.

<img width="900" src="../../images/init/lvm-all.png" />


If the disks attached are different on every node, use this.

<img width="900" src="../../images/init/lvm-indiv.png" />


### Configure MySQL

`MySQL` is used by open source components like `Apache Superset` and `Apache Ozone` in Chango.
Select the host on which `MySQL` will be installed.

<img width="900" src="../../images/init/mysql.png" />


### Configure Object Storage

Select the options for object storage. `Apache Ozone` is default object storage provided by Chango, 
which will be used in disconnected environment in most cases.
In public, you may select the external object storage like AWS S3, MinIO, OCI Object Storage.

If Apache Ozone is used as object storage, enter the values this below.
<img width="900" src="../../images/init/s3-ozone.png" />

If external object storage is used, enter values of S3 credentials.
<img width="900" src="../../images/init/s3-external.png" />


### Configure Chango Authorizer

`Chango Authorizer` is used to authenticate and authorize all the data access to Chango.

<img width="900" src="../../images/init/authorizer.png" />

### Configure Chango REST Catalog

`Chango REST Catalog` is used as data catalog in Chango.

<img width="900" src="../../images/init/rest-catalog.png" />


For now, the components you configured are mandatory.  After configuring mandatory components, you can skip configuration.

<img width="900" src="../../images/init/skip.png" />


You can install other optional components later after finishing Chango initialization.

### Install Configured Components

Install all the configured components.

<img width="900" src="../../images/init/install.png" />


When the installation is finished, press the button of `Installation Finished`.

<img width="900" src="../../images/init/install-finished.png" />


Then, you will move to the main page.

<img width="900" src="../../images/index/overview.png" />


### Show Log

You can see current log produced by installed components.

<img width="900" src="../../images/navi/metrics-status.png" />

Click the host name of components in `Status` to show log.

<img width="900" src="../../images/navi/show-log.png" />


## Chango Authorizer

### Scale Server

Chango Authorizer can be scaled out or unscaled.

<img width="900" src="../../images/authorizer/scale.png" />

First, select hosts for scaling out or unscaling Authorizer servers, and then, press the button of `Scale Out Server` to scale out Authorizer servers
or press the button of `Unscale Server` to unscale Authorizer servers.


## Chango REST Catalog

### Scale Server

Chango REST Catalog can be scaled out or unscaled.

<img width="900" src="../../images/rest-catalog/scale.png" />

First, select hosts for scaling out or unscaling REST Catalog servers, and then, press the button of `Scale Out Server` to scale out REST Catalog servers
or press the button of `Unscale Server` to unscale REST Catalog servers.



## Apache Kafka

`Apache Kafka` is used as event streaming platform in Chango.
Multiple Kafka clusters are supported by Chango, that is, you can install kafka clusters as many as you want.

### Install Kafka

If you want to install `Apache Kafka`, press `Go to Install` button.

> **_NOTE:_** If you have not installed any kafka cluster, then, enter `default` for the cluster name.

<img width="900" src="../../images/kafka/install.png" />


After installing kafka, you will see kafka page like this.

<img width="900" src="../../images/kafka/after-install.png" />


Because Chango supports multiple kafka clusters, you can install another kafka cluster.

> **_NOTE:_** Because you have already installed default kafka cluster, you can enter anything for the cluster name.

<img width="900" src="../../images/kafka/install-another.png" />

After installing another kafka cluster, new created kafka cluster will be shown in tab list.

<img width="900" src="../../images/kafka/multi-clusters.png" />


### Scale Broker

Kafka Broker can be scaled out or unscaled.

<img width="900" src="../../images/kafka/scale-broker.png" />

First, select hosts for scaling out or unscaling kafka brokers, and then, press the button of `Scale Out Broker` to scale out brokers 
or press the button of `Unscale Broker` to unscale brokers.

### Configure Kafka

You can update kafka configurations like heap memory and `server.properties`.

First, select kafka cluster which you want to configure.

<img width="900" src="../../images/kafka/configure-select.png" />

After modifying configuration, press `Update` to update the selected kafka cluster.

<img width="900" src="../../images/kafka/update-conf.png" />


## Apache Spark

`Apache Spark` is used as computing engine to run batch and streaming jobs in Chango.

### Install Spark

Select hosts to install master, worker and history server of Spark.
<img width="900" src="../../images/spark/install.png" />

After installing Spark, you will see the spark page like this.
<img width="900" src="../../images/spark/after-install.png" />

### Scale Worker

You can scale out and unscale spark workers.

<img width="900" src="../../images/spark/scale-worker.png" />

### UI

There are URL links to get Spark Master UI and Spark History Server UI in `UI` of Spark Page.

Spark Master UI looks as below.

<img width="900" src="../../images/spark/master-ui.png" />

Spark History Server UI looks like this.

<img width="900" src="../../images/spark/history-ui.png" />


## Trino

`Trino` is used as query engine to run interactive and long running ETL query in Chango. 
Chango provides multiple trino clusters, so, you can install trino clusers as many as you want.

### Install Trino

Enter `default` for the cluser name if default trino cluster is not installed.
<img width="900" src="../../images/trino/install.png" />

After installing default trino cluster, trino page looks like this.
<img width="900" src="../../images/trino/multi-cluster-1.png" />

If you want to install another trino cluster, enter any name for cluster name.
<img width="900" src="../../images/trino/install-another.png" />

After installing another trino cluster, new created trino cluster will be shown in the cluster tab list.
<img width="900" src="../../images/trino/multi-cluster-2.png" />


### Scale Worker

You can scale out and unscale trino workers.

<img width="900" src="../../images/trino/scale-worker.png" />


### Configure Trino

You can update trino memory configurations and catalogs.

To update memory properties in trino, select trino cluster.
<img width="900" src="../../images/trino/configure.png" />

Update catalogs in trino.
<img width="900" src="../../images/trino/update-catalog.png" />

You can also add catalogs.
<img width="900" src="../../images/trino/add-catalog.png" />


### UI

You can get Trino UI clicking link in `UI` of the selected trino cluster.

<img width="900" src="../../images/trino/coordinator-ui.png" />

## Chango Trino Gateway

`Chango Trino Gateway` is used to route trino queries to the backend trino clusters in Chango.

In addition, `Chango Trino Gateway` also provides the following functions.
- authenticate trino users and authorize the queries run by trino users.
- activate and deactivate the backend trino clusters.

### Install Chango Trino Gateway

Select hosts for `Chango Trino Gateway` servers, host for NGINX proxy, and host for Redis cache.
<img width="900" src="../../images/trino-gw/install.png" />


It looks as below after installing it.
<img width="900" src="../../images/trino-gw/after-install.png" />

### Scale Server

Chango Trino Gateway can be scaled out or unscaled.

<img width="900" src="../../images/trino-gw/scale.png" />

First, select hosts for scaling out or unscaling Trino Gateway servers, and then, press the button of `Scale Out Server` to scale out Trino Gateway servers
or press the button of `Unscale Server` to unscale Trino Gateway servers.


## Apache Superset

`Apache Superset` is used as BI tool in Chango.

### Install Superset

Select host for superset server.
<img width="900" src="../../images/superset/install.png" />

### UI

You can get Superset UI clicking link in `UI` of superset page.
<img width="900" src="../../images/superset/ui.png" />


## Azkaban

`Azkaban` is used as workflow to integrate all the batch jobs like spark ETL jobs and trino ETL jobs in Chango.

### Install Azkaban

Select host for web and hosts for executors.
<img width="900" src="../../images/azkaban/install.png" />

### UI

You can get Azkaban UI clicking link in `UI` of azkaban page.
<img width="900" src="../../images/azkaban/ui.png" />


## Azkaban CLI

`Azkaban CLI` is CLI to create and update azkaban project on Azkaban.

### Install Azkaban CLI

Select hosts for azkaban CLI.
<img width="900" src="../../images/azkaban-cli/install.png" />


## Chango Data API

Chango provides data ingestion especially for streaming events.
`Chango Data API` is used to collect streaming events and produce them to kafka.

### Install Chango Data API

Select hosts for Chango Data API servers and host for NGINX proxy.
<img width="900" src="../../images/data-api/install.png" />

### Scale Server

You can scale out and unscale `Chango Data API` servers.
<img width="900" src="../../images/data-api/scale-server.png" />

## Chango Streaming

`Chango Streaming` is a spark streaming job and used to consume streaming events from kafka and save them to Iceberg table in Chango.

### Install Chango Streaming

Enter spark configurations for `Chango Streaming` job.
<img width="900" src="../../images/streaming/install.png" />

After installation, you will see the driver host of `Chango Streaming` spark job.
<img width="900" src="../../images/streaming/driver-host.png" />

In spark master UI, `Chango Streaming` job will be found.
<img width="900" src="../../images/streaming/in-spark-ui.png" />


### Install Chango Streaming Tx

This is transactional spark streaming jobs. Enter spark configurations for `Chango Streaming Tx` job.

> You need to enter iceberg `schema` and `table` to which streaming messages will be saved.

<img width="900" src="../../images/streaming/install-tx.png" />

After installation, you will see chango streaming tx applications in running status in spark ui.


## Chango Spark Thrift Server

`Chango Spark Thrift Server` is a spark streaming job to run spark sql queries. Clients can connect to spark thrift server, 
for example through JDBC/Thrift.

### Install Chango Spark Thrift Server
Enter spark configurations for `Chango Spark Thrift Server` job.
<img width="900" src="../../images/streaming/install-sts.png" />

After installation, you will see spark thrift server in running status in spark ui.


## Chango Spark SQL Runner

`Chango Spark SQL Runner` is a spark streaming job which exposes REST API to run spark sql queries requested by clients through REST.

### Install Chango Spark SQL Runner
Enter spark configurations for `Chango Spark SQL Runner` job.
<img width="900" src="../../images/streaming/install-sql.png" />

After installation, you will see spark sql runner in running status in spark ui.


## Chango Query Exec

`Chango Query Exec` is a REST application to execute trino ETL queries to transform data in Chango. 
It may be used as alternative to dbt to transform data in Chango.

### Install Chango Query Exec

Select hosts for Chango Query Exec servers and host for NGINX proxy.
<img width="900" src="../../images/query-exec/install.png" />

### Scale Server

You can scale out and unscale `Chango Query Exec` servers.
<img width="900" src="../../images/query-exec/scale-server.png" />

