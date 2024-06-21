# Automatic Iceberg Table Maintenance

Everytime data committed to iceberg tables, many files will be created like data files, snapshots, metadata files which should be maintained.
`Chango REST Catalog` maintains iceberg tables automatically. That is, `Chango REST Catalog` does the followings for you automatically.

- Compacts small files.
- Expires snapshots.
- Remove old metadata files.
- Remove orphan files.
- Rewrite manifest files.
- Rewrite position delete files

If you don't want to let `Chango REST Catalog` maintain iceberg tables, you can disable maintenance for the individual iceberg tables.

## Move to Iceberg Maintenance Page

In order to move to iceberg maintenance page, click the button of `Go to Iceberg Maintenance` in the section of `action` in `Chango REST Catalog` page.
Then, you may see the following picture with selecting schema and table. You will see the current table size and record count like this.

<img width="900" src="../../images/user-guide/iceberg-maintenance.png" />

## Show Maintenance History

You may see the history of table maintenance by `Chango REST Catalog` automatically with clicking the tab of `Maintenance History`.

<img width="900" src="../../images/user-guide/maintenance-history.png" />

## Show Internal Iceberg Table Status and Statistics

You may also see all the internal table status and statistics with clicking another tabs, for example, click the tab of `History` to show table history.

<img width="900" src="../../images/user-guide/iceberg-history.png" />


## Simple Query Runner

There is a simple query runner in which you can run any spark sql queries. Click the tab of `Run Query`.

<img width="900" src="../../images/user-guide/iceberg-simple-query-runner.png" />





