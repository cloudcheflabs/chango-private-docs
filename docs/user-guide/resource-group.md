# Configure Resource Groups

`Resource Group` is used to limit resources in trino.
`Chango Trino Gateway` supports `Resource Group` to control resources of the backend trino clusters in which trino 
queries will be run.

In `Chango Trino Gateway`, there is `Global Resource Group` which is root resource group to which all subgroups belong which are equivalent to `Cluster Group` of `Chango Trino Gateway`.
For example, if users in `BI Cluster Group` send queries, the queries will be run in the backend trino clusters according to `BI Resource Group` configuration.

## Create Resource Group

The default configuration of `Global Resource Group` looks like this. It is root resource group of `Chango Trino Gateway`.

<img width="900" src="../../images/user-guide/global-resource-group.png" />

All the sub resource groups which are equivalent to `Cluster Group` of `Chango Trino Gateway` are dependent on this `Global Resource Group` configuration.
For example, if `schedulingPolicy` is either `weighted` or `weighted_fair` in `Global Resource Group`, `schedulingWeight` needs to be configured in all the subgroups(that is, in all cluster groups).


Let's create a resource group. Go to `Trino Gateway` -> `Cluster Groups`, and click `Create Cluster Group`.

<img width="900" src="../../images/user-guide/create-resource-group.png" />

There are several properties which can be configured for the `Resource Group` with the name of `Cluster Group`. Click the button of `Create`.