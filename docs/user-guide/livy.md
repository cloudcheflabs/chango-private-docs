# Run Spark Applications using Livy

Livy exposes REST API to run spark applications remotely. 

## Run Spark Batch Application

You need to know Livy endpoint to which spark job configuration should be sent in the menu of `Spark` beforehand.

Let's run spark example PI application using Livy.

```agsl
cat <<EOF > /home/opc/run-livy-batch.json
{
   "file":"file:///usr/lib/spark/examples/jars/spark-examples_2.12-3.4.3.jar",
   "className":"org.apache.spark.examples.SparkPi",
   "args": ["100"],
   "driverMemory":"1G",
   "driverCores": 1,
   "executorMemory":"1G",
   "executorCores":1,
   "numExecutors":1,
   "name":"PI Example New Named 3",
   "conf":{
      "spark.master":"spark://chango-private-1.chango.private:7777"
   }
}
EOF

# convert pretty json to oneline json.
cat /home/opc/run-livy-batch.json | jq -c > /home/opc/run-livy-batch-oneline.json

# call livy api.
curl -H "Content-Type: application/json" \
-XPOST \
"http://chango-private-1.chango.private:8998/batches" \
-d @/home/opc/run-livy-batch-oneline.json \
;
```

The livy endpoint above needs to be replaced with your current livy endpoint shown in `Spark` UI page.


## Get Batch State

You can get batch state of spark applications run by Livy.

```agsl
curl -H "Content-Type: application/json" \
-XGET \
"http://chango-private-1.chango.private:8998/batches/0/state" \
;
```

## Get Batch Log

If you want to see the log of spark application, call the following.

```agsl
curl -H "Content-Type: application/json" \
-XGET \
"http://chango-private-1.chango.private:8998/batches/0/log" \
;

```

## Delete Batch

In order to delete spark application job, run the following rest call.


```agsl
curl -H "Content-Type: application/json" \
-XDELETE \
"http://chango-private-1.chango.private:8998/batches/0" \
;
```

