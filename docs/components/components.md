# Chango Private Components


## Component Version


| Component                    | Version                           | 
|:-----------------------------|:----------------------------------| 
| Apache Ozone                 | 1.3.0                             | 
| Kafka                        | 3.4.x                             | 
| Zookeeper                    | 3.6.4                             | 
| Schema Registry              | 7.4.0                             | 
| Spark                        | 3.4.3 / Scala 2.12                | 
| Trino                        | 435                               | 
| Iceberg                      | 1.4.2                             | 
| Open JDK                     | 8 / 11 / 17                       | 
| Azkaban                      | 3.69.0-8                          | 
| Azkaban CLI                  | 0.9.14                            |
| Superset                     | 0.38.1                            | 
| dbt                          | dbt-core: 1.2.1, dbt-trino: 1.2.2 |
| Chango Private Admin         | 2.3.0                             | 
| Chango Private Streaming     | 2.3.0                             | 
| Chango Private Streaming Tx  | 2.3.0                             | 
| Chango Spark Thrift Server   | 2.3.0                             | 
| Chango Spark SQL Runner      | 2.3.0                             | 
| Chango Private Data API      | 2.3.0                             | 
| Chango Private Trino Gateway | 2.3.0                             | 
| Chango Private Authorizer    | 2.3.0                             | 
| Chango Private REST Catalog  | 2.3.0                             |
| Chango Private Query Exec    | 2.3.0                             |

## Component Ports

### Azkaban
- 28081: Azkaban Web Server
- 28082: Azkaban Executor Server

### Kafka
- 9092: Kafka Server

### Schema Registry
- 8081: Registry Server

### MySQL
- 3306: MySQL Server

### Ozone
- 9862: OM RPC
- 9872: OM Ratis
- 9874: OM HTTP
- 9875: OM HTTPS
- 9861: SCM Data Node
- 9863: SCM Block Client
- 9860: SCM Client
- 9876: SCM HTTP
- 9877: SCM HTTPS
- 9894: SCM Ratis
- 9895: SCM GRPC
- 9961: SCM Security Service
- 9882: Data Node HTTP
- 9883: Data Node HTTPS
- 9858: Container Ratis IPC
- 9859: Container IPC
- 9864: Data Node Client
- 9855: Container Ratis Data Stream
- 9857: Container Ratis Admin
- 9856: Container Ratis Server
- 9878: S3G HTTP
- 9879: S3G HTTPS
- 9891: Recon RPC
- 9888: Recon HTTP
- 9889: Recon HTTPS
- 9884: Freon HTTP
- 9885: Freon HTTPS
- 88: KDC Server
- 749: KDC Admin Server
- 80: NGINX

### Spark
- 7777: Master Server
- 7778: Worker Server
- 8880: Master Web UI
- 8881: Worker Web UI
- 18882: History Server

### Livy
- 8998: Server

### Superset
- 38088: Superset Server

### Trino
- 18080: Coordinator 
- 18081: Worker

### Zookeeper
- 2181: Zookeeper Server 
- 2888: Zookeeper Peer Port 
- 3888: Zookeeper Leader Port

### PostgreSQL
- 5432: PostgreSQL Server

### Chango Private Admin
- 28095: Admin Server 
- 8080: NGINX

### Chango Private Admin UI
- 8123: Admin UI Server

### Chango Spark Thrift Server
- 10000: Server

### Chango Spark SQL Runner
- 29080: Server

### Chango Private Data API
- 28097: Server 
- 28098: Jetty Server 
- 27072: Management Server 
- 80: NGINX

### Chango Private Trino Gateway
- 28096: Server 
- 27081: Management Server 
- 27788: Jetty Proxy Server 
- 6379: Redis Master
- 6380: Redis Slave
- 16379: Redis Master Cluster
- 16380: Redis Slave Cluster
- 443: NGINX SSL

### Chango Private Authorizer
- 28196: Server 
- 27181: Management Server 
- 81: NGINX

### Chango Private REST Catalog
- 28197: Server 
- 27182: Management Server 
- 28297: Jetty Proxy Server 
- 8008: NGINX

### Chango Private Query Exec
- 28291: Server
- 27291: Management Server
- 8009: NGINX


