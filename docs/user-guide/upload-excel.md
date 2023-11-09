# Upload Excel

You can upload Excel files with either Chango CLI or Admin UI.

This example will show how to upload Excel files with Admin UI.

> **_NOTE:_** Before uploading Excel, make sure that `Chango Data API` and `Chango Straming` are installed.


## Get Chango Data API Endpoint

You need to get the endpoint of `Chango Data API` in `Components` -> `Chango Data API`. 

<img width="900" src="../../images/user-guide/data-api-endpoint.png" />

## Get Chango Credential

You need to get Chango Credential to have privileges to upload Excel file. Go to `Settings` -> `Security`.

First, create a role `upload-excel`.

<img width="900" src="../../images/user-guide/role-upload-excel.png" />

Crate a credential for the role `upload-excel`.

<img width="900" src="../../images/user-guide/cred-upload-excel.png" />

Create privileges with storage path `iceberg.excel_db.*` for the role `upload-excel`.

<img width="900" src="../../images/user-guide/read-priv-upload-excel.png" />
<img width="900" src="../../images/user-guide/write-priv-upload-excel.png" />

## Upload Excel File

Before uploading Excel file, make sure that the header columns exist in the Excel file. Go to `Upload Files` -> `Upload Excel`.

Assumed that you want to insert uploaded Excel rows into the table `test_excel` in schema `excel_db` of catalog `iceberg`, 
Enter the values as follows.

<img width="900" src="../../images/user-guide/upload-excel.png" />

To select rows of uploaded Excel, run the query with Superset like this.

```agsl
-- select rows in uploaded excel.
select * from iceberg.excel_db.test_excel;
```

<img width="900" src="../../images/user-guide/select-upload-excel.png" />

