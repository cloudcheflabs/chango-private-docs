# Data Catalog

`Chango REST Catalog` is iceberg REST Catalog used as data catalog in Chango.


## Security-first Data Catalog

Storage Security is a first-class mandatory in modern data lakehouses.
`Chango REST Catalog` works with `Chango Authorizer` tightly which controls all the data access with strong storage security of catalog, schema and table level in Chango.
That is, multiple iceberg supported engines like `spark`, `trino` can work with `Chango REST Catalog` seamlessly with the support of strong storage security to iceberg in Chango.

## Automatic Iceberg Table Maintenance

Everytime data committed to iceberg tables, many files will be created like data files, snapshots, metadata files which should be maintained manually later.
`Chango REST Catalog` maintains iceberg tables automatically for you. That is, `Chango REST Catalog` does the followings for you automatically.

- Compacts small files.
- Expires snapshots.
- Remove old metadata files.
- Remove orphan files.
- Rewrite manifest files.
- Rewrite position delete files
