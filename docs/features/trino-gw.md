# Chango Private Trino Gateway

Trino queries will be routed to the backend trino clusters by Chango Private Trino Gateway dynamically.


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

