# Create Iceberg REST Catalog in Spark

This shows how to create Iceberg REST catalog in spark applications in Chango Private.


If spark applications need to access Iceberg tables using `Chango REST Catalog`, Chango credential and S3 credential are necessary.

Let's see the following codes to create Iceberg REST Catalog in Spark.
```java
        String s3AccessKey = "...";
        String s3SecretKey = "...";
        String s3Endpoint = "...";
        String s3Region = "...";
        String restUrl = "...";
        String warehouse = "...";
        String token = "...";

        SparkConf sparkConf = new SparkConf().setAppName("Run Spark with Iceberg REST Catalog");

        // set aws system properties.
        System.setProperty("aws.region", (s3Region != null) ? s3Region : "us-east-1");
        System.setProperty("aws.accessKeyId", s3AccessKey);
        System.setProperty("aws.secretAccessKey", s3SecretKey);

        // iceberg rest catalog.
        sparkConf.set("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions");
        sparkConf.set("spark.sql.catalog.rest", "org.apache.iceberg.spark.SparkCatalog");
        sparkConf.set("spark.sql.catalog.rest.catalog-impl", "org.apache.iceberg.rest.RESTCatalog");
        sparkConf.set("spark.sql.catalog.rest.io-impl", "org.apache.iceberg.aws.s3.S3FileIO");
        sparkConf.set("spark.sql.catalog.rest.uri", restUrl);
        sparkConf.set("spark.sql.catalog.rest.warehouse", warehouse);
        sparkConf.set("spark.sql.catalog.rest.token", token);
        sparkConf.set("spark.sql.catalog.rest.s3.endpoint", s3Endpoint);
        sparkConf.set("spark.sql.catalog.rest.s3.path-style-access", "true");
        sparkConf.set("spark.sql.defaultCatalog", "rest");

        SparkSession spark = SparkSession
                .builder()
                .config(sparkConf)
                .enableHiveSupport()
                .getOrCreate();

        Configuration hadoopConfiguration = spark.sparkContext().hadoopConfiguration();
        hadoopConfiguration.set("fs.s3a.endpoint", s3Endpoint);
        if(s3Region != null) {
            hadoopConfiguration.set("fs.s3a.endpoint.region", s3Region);
        }
        hadoopConfiguration.set("fs.s3a.access.key", s3AccessKey);
        hadoopConfiguration.set("fs.s3a.secret.key", s3SecretKey);
        hadoopConfiguration.set("fs.s3a.path.style.access", "true");
        hadoopConfiguration.set("fs.s3a.change.detection.mode", "warn");
        hadoopConfiguration.set("fs.s3a.change.detection.version.required", "false");
        hadoopConfiguration.set("fs.s3a.multiobjectdelete.enable", "true");
        hadoopConfiguration.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem");
        hadoopConfiguration.set("fs.s3a.aws.credentials.provider", "org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider");
```

- `s3AccessKey`: S3 Access Key.
- `s3SecretKey`: S3 Secret Key.
- `s3Endpoint`: S3 Endpoint.
- `s3Region`: S3 Region.
- `restUrl`: Endpoint of `Chango REST Catalog`.
- `warehouse`: Warehouse Path.
- `token`: Chango Credential. 

To get S3 Credential, see <a href="../../user-guide/s3-cred">Get S3 Credential</a>. 
For the value of `warehouse`, it can be `s3a://[bucket]/warehouse-rest` where `[bucket]` is found in your S3 credential list.

To get Endpoint URL of `Chango REST Catalog`, go to `Components` -> `Chango REST Catalog`. 
The URL will be shown in `Endpoint` section.

To get Chango Credential, see <a href="../../user-guide/cred">Get Chango Credential</a>.