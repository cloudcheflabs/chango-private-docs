# Upload Files

Using `Chango CLI`, CSV, JSON, and Excel Files can be uploaded whose data will be inserted to Iceberg table in Chango automatically.

> **_NOTE:_** Before uploading files, make sure that `Chango Data API` and `Chango Straming` are installed.

## Install Chango CLI

Download chango client jar and install it as Chango CLI.
```agsl
curl -L -O https://github.com/cloudcheflabs/chango-client/releases/download/2.0.1/chango-client-2.0.1-executable.jar;
cp chango-client-2.0.1-executable.jar ~/bin/chango;
chmod +x ~/bin/chango;
```

## Initialize Chango CLI

Credential needs to be entered when Chango CLI is initialized.
To Get Chango Credential, see <a href="../../user-guide/cred">Get Chango Credential</a>.
```agsl
chango init
```

After credential entered, output looks like this.
```agsl
chango init

Enter Token: xxxxxxxx
Initialization success!
```

## Upload JSON

You can upload individual JSON file or all the JSON files in directory.

> **_NOTE:_** Before uploading JSON files, target Iceberg table needs to be created beforehand. 
> See <a href="../streaming/#create-iceberg-table-before-sending-json-events">here</a> for more details.

Upload JSON file.

```
chango upload json local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table test_iceberg \
--file /home/chango/multi-line-json.json \
--batch-size 150000 \
;
```

- `schema` : iceberg schema created before.
- `table`: iceberg table where json data will be ingested in chango.
- `file` : local json file path.
- `batch-size` : list of json in gzip will be sent in batch mode. The size of json list.


Upload all JSON files in directory.

```
chango upload json local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table test_iceberg \
--directory /home/chango/json-files \
--batch-size 150000 \
;
```

- `directory`: the directory where json files are located which will be sent to chango.

`[endpoint-of-data-api]` is the endpoint of `Chango Data API`, go to `Components` -> `Chango Data API`, and get URL in `Endpoint` section.


## Upload Excel

You don't have to create Iceberg table before uploading Excel to chango. Chango will create Iceberg table automatically.

> **_NOTE:_** Take a note that the header of Excel must exist.

You can also create Iceberg table(for example, Iceberg table with the definition of partition columns) before uploading Excel to chango.

```
chango upload excel local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table excel_to_json \
--file /home/chango/data/excel-to-json.xlsx \
;
```


## Upload CSV

As like Excel, you don’t have to create Iceberg table beforehand. Chango will create Iceberg table automatically.

> **_NOTE:_** Take a note that the header of CSV must exist.

You can also define a Iceberg table(for example, Iceberg table with the definition of partition columns) before uploading CSV to chango.


The following example is to upload CSV with the separator of comma.

```
chango upload csv local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table csv_to_json_comma \
--separator "," \
--is-single-quote false \
--file /home/chango/data/csv-to-json-comma.csv \
;
```

- `separator` is the separator of csv data. If csv data is tab separated, the value is `TAB` . But for another separators, you need to type a separator value, for instance,  `,` , `|` , or something else.
- `is-single-quote` is if the escaped value of csv data is single quoted or not. Default is `false`.

For tab separated CSV, use the following.

```
chango upload csv local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table csv_to_json_tab \
--separator TAB \
--is-single-quote false \
--file /home/chango/data/csv-to-json-tab.csv \
;
```

- parameter `separator` value must be `TAB`.

You can upload multiple CSV files located in the directory.

```
chango upload csv local \
--data-api-server [endpoint-of-data-api] \
--schema iceberg_db \
--table csv_to_json_comma \
--separator "," \
--is-single-quote false \
--directory /home/chango/local-csvs \
;
```