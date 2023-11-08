# Create Azkaban Project using CLI

Azkaban Project file is a zip file in which workflow files are contained. 
Azkaban project file can be uploaded either via Azkaban Web UI or via Azkaban CLI to create workflow in Azkaban.

## Create Azkaban Project File

First, create flow file, for example, `spark-pi-client.flow`.

```agsl
---
config:
  failure.emails: admin@your-domain.com

nodes:
  - name: Start
    type: noop

  - name: SparkPi
    type: command
    config:
      command: ssh spark@chango-private-3.chango.private "/home/spark/spark-pi-client.flow"
    dependsOn:
      - Start

  - name: End
    type: noop
    dependsOn:
      - SparkPi
```

And create project file `flow20.project`.

```agsl
azkaban-flow-version: 2.0
```

Finally, package as zip file.
```agsl
zip spark-pi-client.zip spark-pi-client.flow flow20.project 
```

## Upload Azkaban Project File with Azkaban CLI

First, access to the host where `Azkaban CLI` is installed.

Login as azkaban cli user with activating python virtual environment.

```agsl
sudo su - azkabancli;
source venv/bin/activate;
azkaban --help;
```


Create azkaban project with the name of `spark-pi-client` with CLI.

```agsl
azkaban upload -c -p spark-pi-client -u azkaban@http://[azkaban-web-host]:28081 ./spark-pi-client.zip
```
Default password for the user `azkaban` is `azkaban`.

To update azkaban project, parameter `-c` needs to be removed from the above command.
