# Home

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Code Blocks

### Code Blocks Example


```java
     public String callGetRestAPI(String url) throws IOException {
        LOG.info("ready to call [{}]", url);
        Request request = new Request.Builder()
                .url(url)
                .addHeader("Authorization", "Bearer " + accessToken)
                .build();
        RestResponse restResponse = ResponseHandler.doCall(simpleHttpClient.getClient(), request);
        if(restResponse.getStatusCode() == STATUS_OK) {
            return restResponse.getSuccessMessage();
        } else {
            LOG.error("response for [{}]: {}", url, restResponse.getErrorMessage());
            throw new RuntimeException(restResponse.getErrorMessage());
        }
    }
```


### Code Blocks Example2


```java title="AnyClass.java" linenums="1" hl_lines="2-3"
     private static SparkStreamingInstance streamingContext(String metastoreUrl, String bucket, String accessKey, String secretKey, String endpoint, String region) {
        SparkConf sparkConf = new SparkConf().setAppName("disruptor streaming context");
        sparkConf.setMaster("local[2]");

        SparkSession sparkSession = (spark == null) ? SparkSession
        .builder()
        .config(sparkConf)
        .enableHiveSupport()
        .getOrCreate() : spark;

        spark = sparkSession;

        // iceberg catalog from hive metastore.
        sparkConf = sparkSession.sparkContext().conf();
        sparkConf.set("spark.sql.catalog.hive_prod", "org.apache.iceberg.spark.SparkCatalog");
        sparkConf.set("spark.sql.catalog.hive_prod.type", "hive");
        sparkConf.set("spark.sql.catalog.hive_prod.uri", "thrift://" + metastoreUrl);
        sparkConf.set("spark.sql.catalog.hive_prod.warehouse", "s3a://" + bucket + "/warehouse");


        JavaStreamingContext scc = new JavaStreamingContext(JavaSparkContext.fromSparkContext(spark.sparkContext()), new Duration(1000));

        org.apache.hadoop.conf.Configuration hadoopConfiguration = scc.sparkContext().hadoopConfiguration();
        hadoopConfiguration.set("fs.defaultFS", "s3a://" + bucket);
        hadoopConfiguration.set("fs.s3a.endpoint.region", region);
        hadoopConfiguration.set("fs.s3a.endpoint", endpoint);
        hadoopConfiguration.set("fs.s3a.access.key", accessKey);
        hadoopConfiguration.set("fs.s3a.secret.key", secretKey);
        hadoopConfiguration.set("fs.s3a.path.style.access", "true");
        hadoopConfiguration.set("fs.s3a.change.detection.mode", "warn");
        hadoopConfiguration.set("fs.s3a.change.detection.version.required", "false");
        hadoopConfiguration.set("fs.s3a.multiobjectdelete.enable", "true");
        hadoopConfiguration.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem");
        hadoopConfiguration.set("fs.s3a.aws.credentials.provider", "org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider");
        hadoopConfiguration.set("hive.metastore.uris", "thrift://" + metastoreUrl);
        hadoopConfiguration.set("hive.server2.enable.doAs", "false");
        hadoopConfiguration.set("hive.metastore.client.socket.timeout", "1800");
        hadoopConfiguration.set("hive.execution.engine", "spark");

        return new SparkStreamingInstance(scc, sparkSession);
        }
    }
```

### Code Blocks Example3


```java hl_lines="2-5"
     public String callGetRestAPI(String url) throws IOException {
        LOG.info("ready to call [{}]", url);
        Request request = new Request.Builder()
                .url(url)
                .addHeader("Authorization", "Bearer " + accessToken)
                .build();
        RestResponse restResponse = ResponseHandler.doCall(simpleHttpClient.getClient(), request);
        if(restResponse.getStatusCode() == STATUS_OK) {
            return restResponse.getSuccessMessage();
        } else {
            LOG.error("response for [{}]: {}", url, restResponse.getErrorMessage());
            throw new RuntimeException(restResponse.getErrorMessage());
        }
    }
```


