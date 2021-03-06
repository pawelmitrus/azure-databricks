# azure-databricks

1. [ Description. ](#desc)
2. [ Pricing. ](#pricing)
    1. [ Pricing Tiers. ](#pricing-tiers)
    2. [ Workloads. ](#pricing-workloads)
    3. [ Analyze costs. ](#pricing-costs)
  
<a name="desc"></a>
## 1. Description


<a name="pricing"></a>
## 2. Pricing
All details with regards to pricing could be found here: https://azure.microsoft.com/en-us/pricing/details/databricks/
Below article explains some of the details and summarizes the most important differences.

Cost related to Databricks usage depends on:
- DBUs (Databricks Units) - Each running cluster node generates DBUs according to its size. Available in *pay-as-you-go* and *pre-purchased* offerings.
- VMs (Virtual Machines, Disks, Network, Public IP Addresses)

Overall cost of single job is calculated with formula:

`cost = TW * DBU * s / 3600`

where:
- TW - rate for type of *workload* and *tier* being used
- DBU - amount of DBUs generated by all nodes
- s* - time of the job execution (including cluster warmp-up if it was not running)

(*) Applies only for automated-cluster (single cluster dedicated for single job). For interactive-clusters (multiple jobs can be submitted to cluster) the cost is calculated for the time cluster is running (its every node).

<a name="pricing-tiers"></a>
### 2.1 Pricing Tiers
Price of DBU depends on pricing tier of Databricks workspace. There are 2 different pricing tiers offered by Azure Databricks:
* Standard
* Premium

The main difference is that with *Standard* tier you will not get:
* RBAC for any of the objects (everyone is an admin)
* Credential Passthrough for clusters is not available (ACLs on ADLS mean nothing)
* JDBC/ODBC Endpoint Authentication (no authentication to HIVE)

You may downgrade/upgrade pricing tier by redeploying Databricks Workspace with the same name (using the same ARM template) but with another pricing tier.

<a name="pricing-workloads"></a>
### 2.2 Workloads
Price of DBU depends on workload executed on Databricks workspace. Workloads cannot be pre-provisioned or set. They are set depending on the actions taken. There are 3 different workloads offered by Azure Databricks:
* Engineering Light
* Engineering
* Analytics

The main differences for given workloads are:
* Engineering Light - only when cluster is running with Databricks Light runtime (very close to pure Apache Spark, does not support features like autoscaling, delta lake, high concurrent cluster, notebooks and collaborative features, submiting notebook as a job etc. 
* Engineering - only when jobs are run with automated clusters. Well suited for scenarios when Databricks are used as ETL engine and we do not plan to plenty of small jobs (cost overhead of spinning up new automated cluster for every single small job would be significant)
* Analytics - any other scenario - using interactive clusters, using notebooks, collaboration, history version, integration with R studio, using high concurrency clusters, BI integration through JDBC/ODBC endpoint

<a name="pricing-costs"></a>
### 2.3 Analyze costs

As mentioned above Databricks cost can be divided into Databricks generated costs (DBUs) and Cluster costs (VMs). **By default Azure doesn't provide provide both in one place**. You will find Databricks costs related to the Resource Group where the workspace is hosted. However Cluster costs are available with managed Resource Group, where VMs are hosted (Portal -> Databricks resource -> Overview -> Managed Resource Group).

#### DBUs & Workloads
Cost analytics provides split into Workspace / Pricing Tier / Workload Type granularity. **There is no straightforward information which cluster or job generated specific amount of DBUs.**

Example cost summary:

Workspace  | Meter  | Cost
------------- | ------------- | -------------
databricks-workspace-name  | standard data analytics dbu | $14.51
databricks-workspace-name  | standard data engineering dbu | $31.21

#### Clusters

Managed resource group offers detailed break down of generated cost. Each node (VM, Disk etc.) contains set of *default_tags* that indicate which cluster or instance of this cluster (same interactive cluster terminated and started later) was used. 

Example cost summary:

ResourceId  | Meter  | Cost | Tags
------------- | ------------- | ------------- | -------------
.../databricks-rg-fszfgsdf/.../virtualmachines/ejifs9u38u829  | d3 v2/ds3 v2 | $2.51 | clusterid:3452-4262775-honey2415
.../databricks-rg-fszfgsdf/.../disks/293ut9fds9idf  | p15 disks | $0.51 | clusterid:3452-4262775-honey2415

#### Cost summary
Due to above, you can easily chargeback costs for you automated-clusters (engineering workload) for any job. However this data is not sufficient to chargeback costs to the job level for interactive-clusters.
