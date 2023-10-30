# Run dbt Model

This is an example to run `dbt` model in Chango Private.

## Configure Trino Connection

You need to configure trino connection in `profiles.yml` in dbt.
The following is a template to connect to `Chango Trino Gateway` from dbt.
```agsl
trino:
  target: dev
  outputs:
    dev:
      type: trino
      method: ldap
      user: trino
      password: xxx
      host: chango-private-5.chango.private
      port: 443
      database: iceberg
      schema: silver
      threads: 8
      http_scheme: https
      session_properties:
        query_max_run_time: 5d
        exchange_compression: True
```
The target catalog is `iceberg` and schema is `silver`. `host` is the host name of `Chango Trino Gateway` endpoint which 
can be found in `Endpoint` section in `Components` -> `Chango Trino Gateway`.

`user` and `password` can be created in `Settings` -> `Trino Gateway`. For more details, see <a href="../../install/run-first-ctas">here</a>.


## Run dbt Model using `git-sync`

`dbt` is installed as docker container in Chango Private. To see on which hosts `dbt` is installed, go to `Components` -> `dbt`.
First, you need to access `dbt` host to run dbt model. 

Assumed that your `dbt` model files are source controlled by git repo, you can run `dbt` model with this command.

```agsl
docker exec -it dbt \
/bin/sh -c '
/git-sync \
-repo [git-repo-url] \
-branch [branch] \
-username [user] \
-password [password] \
-root [root-dir] \
-one-time && \
cd [root-dir]/[git-repo-name].git/[dbt-model-dir] && \
dbt run \
--profiles-dir ../ \
--project-dir ./ \
-m models/[model-sql-file]
'
```

`git-sync` already installed in `dbt` docker image will be used to pull `dbt` model files from git repo with the following command.
```agsl
/git-sync \
-repo [git-repo-url] \
-branch [branch] \
-username [user] \
-password [password] \
-root [root-dir] \
-one-time && \
```

And run dbt model like this.
```agsl
dbt run \
--profiles-dir ../ \
--project-dir ./ \
-m models/[model-sql-file]
```