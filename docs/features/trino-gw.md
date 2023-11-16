# Chango Trino Gateway

`Chango Trino Gateway` is an implementation of trino gateway concept.


## Understand Trino Gateway Concept

Chango has the concept of trino gateway which routes trino queries to upstream backend trino clusters dynamically. 
If one of the backend trino clusters has exhausted, then trino gateway will route queries to the trino cluster 
which is executing less requested queries.
Trino does not support HA because trino coordinator has single point failure.  In order to support HA of trino, we need to use trino gateway.

Let’s say, there is only one large trino clusters(100 ~ 1000 workers) in the company. Many people like BI experts, data scientists, 
and data engineers are running trino queries on this large cluster intensively. 
These trino queries can be interactive or ETL. For example, because long running ETL queries have occupied most of the resources 
which trino cluster needs to execute another queries, there can be little resources remaining trino cluster can use 
to execute another interactive queries. The people who are running the interactive queries need to wait 
until the long running ETL queries will finish. Such conflict problems can also happen in reverse case.

Such monolithic approach with large trino workers can lead to be problematic. We need to separate a large trino cluster 
to small trino clusters for the groups like BI team, data scientist team, and data engineer team individually.

Let’s say, BI team has 3 backeend trino clusters. If one of trino clusters needs to be scaled out, and trino catalogs needs 
to be added to this trino cluster for new external data sources, or the trino cluster needs to be reinstalled with new trino version, 
then  first just deactivate this trino cluster to which the queries will not be routed without down time problem of trino query execution. 
After finishing scaling workers ,updating catalogs or reinstalling the trino cluster, activate this trino cluster to which the queries 
will be routed again. With trino gateway, the activation and deactivation of backend trino clusters can be done with ease. 

## Chango Trino Gateway Features

`Chango Trino Gateway` provides several critical functions of trino gateway.

### Trino User Authentication and Authorization

If trino user sends trino queries to `Chango Trino Gateway`, `Chango Trino Gateway` authenticates trino user and 
authorize trino queries if trino user is allowed to access data in Chango or not. 

For authorization, `Chango Trino Gateway` works with `Chango Authorizer` which is central to authorize all credentials used by all chango components.
So, `Chango Trino Gateway` controls the data access of trino user to Chango with RBAC in the fine-grained manner like catalog, schema and table level.

Especially, RBAC to `Cluster Group` can be configured without restarting backend trino clusters.


### Route Trino Queries to Trino Clusters

Routing trino queries to the backend trino clusters is fundamental function of `Chango Trino Gateway`.
Let's see the following picture how `Chango Trino Gateway` will route.

<img width="900" src="../../images/trino-gw-concept/chango-trino-gw-1.png" />

Trino queries which trino users run who belong to the `Cluster Group` will be routed to the backend trino clusters which belong to the `Cluster Group`.
`Chango Trino Gateway` detects exhausted trino clusters, and will route trino queries to less exhausted trino clusters in smart way.

From the point of the storage security in Chango, `Cluster Group` is equivalent to `Role` in `Chango Security` model. 
`Chango Trino Gateway` will check if trino queries are allowed to run according to privileges of `Cluster Group` or not.

Even if you have just one trino cluster as backend trino cluster, you can create several `Cluster Groups` 
as many as you need, and you can control data access of `Cluster Groups` as usual in `Chango Security` model.
<img width="900" src="../../images/trino-gw-concept/chango-trino-gw-2.png" />

### Activate and Deactivate Trino Clusters

`Chango Trino Gateway` provides the feature of activation and deactivation of trino clusters. If one of the backend trino clusters 
is deactivated, trino queries will not be routed to this trino cluster. If the trino cluster is activated, then trino queries will be routed to this 
trino cluster.

If the configuration of backend trino clusters is updated, the backend trino clusters need to be restarted.
In order to provide `Zero Downtime to Run Trino Queries`, `Chanog Trino Gateway` provides such feature of activation and deactivation of trino clusters.

Do the following steps for every trino clusters in order to accomplish `Zero Downtime to Run Trino Queries`.

- deactivate the trino cluster whose configuration needs to be updated.
- update configuration.
- restart that trino cluster. 
- activate that trino cluster again. 


