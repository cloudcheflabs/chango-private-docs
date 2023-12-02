# Storage Security

Chango Private provides fine-grained data access control using RBAC to Chango storage.

## Data Access in Secure Way

<img width="700" src="../../images/security/storage-sec.png" />

Chango Authorizer controls all the data access to Chango Data Lakehouse. 
So all the chango components which want to access data in Chango Data Lakehouse need to be authenticated and authorized by Chango Authorizer.

All data accesses are controlled in the fine-grained manner like catalog, schema and table level. 


## Credential, Role and Privileges

A `Role` can have many `Credentials` and many `Privileges`. There are `READ` and `WRITE` type in privilege. 
Each privilege has storage access path with the convention of `<catalog>`.`<schema>`.`<table>`, for example.

- `iceberg.events.behavior` with `WRITE` : user / credential has the `WRITE` privilege to table behavior in `events` schema of `iceberg` catalog.
- `iceberg.events.*` with `READ`: user / credential has the `READ` privilege to all the tables in `events` schema of `iceberg` catalog.
- `mysql.*` with `READ`: user / credential has the `READ` privilege to all the tables in all schemas of `mysql` catalog.
- `*` with `WRITE`: user / credential has the `WRITE` privilege to all the tables in all schemas of all catalogs.


