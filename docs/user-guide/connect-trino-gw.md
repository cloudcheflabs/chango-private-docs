# Connect to Chango Trino Gateway with Trino Clients

Using Trino Clients like Trino CLI and Apache Superset, you can connect to `Chango Trino Gateway`.

Before proceeding to this example, you need to see <a href="../../install/run-first-ctas/" >Run First Trino Query</a> 
how to create `Cluster Group` in `Chango Trino Gateway` with storage security.

## Conect to Chango Trino Gateway with Apache Superset

You need to follow the instruction shown in <a href="../../install/run-first-ctas/" >Run First Trino Query</a> to 
connect to `Chango Trino Gateway` using Apache Superset.

## Connect to Chango Trino Gateway with Trino CLI

### Install Trino CLI

Run the following to install Trino CLI.

```agsl
mkdir -p ~/trino-cli;
cd ~/trino-cli;

export TRINO_VERSION=422
curl -L -O https://repo1.maven.org/maven2/io/trino/trino-cli/${TRINO_VERSION}/trino-cli-${TRINO_VERSION}-executable.jar;
mv trino-cli-${TRINO_VERSION}-executable.jar trino
chmod +x trino

./trino --version;
```

### Connect to Chango Trino Gateway

Let's say you have created trino user `trino` with proper password in the previous section, and the endpoint of 
`Chango Trino Gateway` is `https://chango-private-1.chango.private:443`. 
Run the following to connect to `Chango Trino Gateway`, and enter the password for the user `trino`.

```agsl
./trino --server https://chango-private-1.chango.private \
--user trino \
--insecure \
--password;
```

If there is no problem occurred to connect, then you will get the following with running `show catalogs`.

```agsl
trino> show catalogs;
 Catalog 
---------
 iceberg 
 jmx     
 system  
 tpcds   
 tpch    
(5 rows)

Query 20231108_052020_00744_pdmmv, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
0.04 [0 rows, 0B] [0 rows/s, 0B/s]
```