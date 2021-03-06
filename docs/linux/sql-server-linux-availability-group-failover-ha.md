---
# required metadata

title: Operate availability group SQL Server on Linux | Microsoft Docs
description: 
author: MikeRayMSFT 
ms.author: mikeray 
manager: jhubbard
ms.date: 04/12/2017
ms.topic: article
ms.prod: sql-linux
ms.technology: database-engine
ms.assetid: 

# optional metadata
# keywords: ""
# ROBOTS: ""
# audience: ""
# ms.devlang: ""
# ms.reviewer: ""
# ms.suite: ""
# ms.tgt_pltfrm: ""
# ms.custom: ""

---

# Operate HA availability group for SQL Server on Linux

## <a name="failover"></a>Fail over availability group

Use the cluster management tools to failover an availability group managed by an external cluster manager. For example, if a solution uses Pacemaker to manage a Linux cluster, use `pcs` to perform manual failovers on RHEL or Ubuntu. On SLES use `crm`. 

> [!IMPORTANT]
> Under normal operations, do not fail over with Transact-SQL or SQL Server management tools like SSMS or PowerShell. When `CLUSTER_TYPE = EXTERNAL`, the only acceptable value for `FAILOVER_MODE` is `EXTERNAL`. With these settings, all manual or automatic failover actions are executed by the external cluster manager. 

### Manual failover examples

Manually fail over the availability group with the external cluster management tools. Under normal operations, do not initiate failover with Transact-SQL. If the external cluster management tools do not respond, you can force the availability group to failover. For instructions to force the manual failover, see [Manual move when cluster tools are not responsive](#forceManual).

Complete the manual failover in two steps. 

1. Move the availability group resource from the cluster node that owns the resources to a new node.

   The cluster manager moves the availability group resource and adds a location constraint. This constraint configures the resource to run on the new node. You must remove this constraint in order to move either manually or automatically failover in the future.

2. Remove the location constraint.

#### 1. Manually fail over

To manually failover an availability group resource named *ag_cluster* to cluster node named *nodeName2*, run appropriate command for your distribution:

- **RHEL/Ubuntu example**

   ```bash
   sudo pcs resource move ag_cluster-master nodeName2 --master
   ```

- **SLES example**

   ```bash
   crm resource migrate ag_cluster nodeName2
   ```



>[!IMPORTANT]
>After you manually failover a resource, you need to remove a location constraint that is automatically added during the move.

#### 2. Remove the location constraint

During a manual move, the `pcs` command `move` or `crm` command `migrate` adds a location constraint for the resource to be placed on the new target node. To see the new constraint, run the following command after manually moving the resource:

- **RHEL/Ubuntu example**

   ```bash
   sudo pcs constraint --full
   ```

- **SLES example**

   ```bash
   crm config show
   ```

You need to remove the location constraint so future moves - including automatic failover - succeed. 

To remove the constraint run the following command. 

- **RHEL/Ubuntu example**

   In this example `ag_cluster-master` is the name of the resource that was moved. 

   ```bash
   sudo pcs resource clear ag_cluster-master 
   ```

- **SLES example**

   In this example `ag_cluster` is the name of the resource that was moved. 

   ```bash
   crm resource clear ag_cluster
   ```

Alternatively, you can run the following command to remove the location constraint.  

- **RHEL/Ubuntu example**

   In the following command `cli-prefer-ag_cluster-master` is the ID of the constraint that needs to be removed. `sudo pcs constraint --full` returns this ID. 

   ```bash
   sudo pcs constraint remove cli-prefer-ag_cluster-master  
   ```
- **SLES example**

   In the following command `cli-prefer-ms-ag_cluster` is the ID of the constraint. `crm config show` returns this ID. 
   
   ```bash
   crm configure
   delete cli-prefer-ms-ag_cluster 
   commit
   ```

>[!NOTE]
>Automatic failover does not add a location constraint, so no cleanup is necessary. 

For more information:
- [Red Hat - Managing Cluster Resources](http://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/6/html/Configuring_the_Red_Hat_High_Availability_Add-On_with_Pacemaker/ch-manageresource-HAAR.html)
- [Pacemaker - Move Resources Manaually](http://clusterlabs.org/doc/en-US/Pacemaker/1.1-pcs/html/Clusters_from_Scratch/_move_resources_manually.html)
 [SLES Admininstration Guide - Resources](https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha/book_sleha.html#sec.ha.troubleshooting.resource) 
 

### <a name="forceManual"></a> Manual move when cluster tools are not responsive 

In extreme cases, if a user cannot use the cluster management tools for interacting with the cluster (i.e. the cluster is unresponsive, cluster management tools have a faulty behaviour), the user might have to perform a failover bypassing the external cluster manager. This is not recommended for regular operations, and should be used within cases cluster is failing to execute the failover action using the cluster management tools.

If you cannot failover the availability group with the cluster management tools, follow these steps to failover from SQL Server tools:

1. Verify that the availability group resource is not managed by the cluster any more. 

      - Attempt to set the resource to unmanaged mode. This signals the resource agent to stop resource monitoring and management. For example: 
      
      ```bash
      sudo pcs resource unmanage <**resourceName**>
      ```

      - If the attempt to set the resource mode to unmanaged mode fails, delete the resource. For example:

      ```bash
      sudo pcs resource delete <**resourceName**>
      ```

      >[!NOTE]
      >When you delete a resource it also deletes all of the associated constraints. 

1. Manually set the session context variable `external_cluster`.

   ```Transact-SQL
   EXEC sp_set_session_context @key = N'external_cluster', @value = N'yes';
   ```

1. Fail over the availability group with Transact-SQL. In the example below replace `<**MyAg**>` with the name of your availability group. Connect to the instance of SQL Server that hosts the target secondary replica and run the following command:

   ```Transact-SQL
   ALTER AVAILABILITY GROUP <**MyAg**> FAILOVER;
   ```

1. Restart cluster resource monitoring and management. Run the following command:

   ```bash
   sudo pcs resource manage <**resourceName**>
   sudo pcs resource cleanup <**resourceName**>
   ```

## Database level monitoring and failover trigger

For `CLUSTER_TYPE=EXTERNAL`, the  failover trigger semantics are different compared to WSFC. When the availability group is on an instance of SQL Server in a WSFC, transitioning out of `ONLINE` state for the database causes the availability group health to report a fault. This will signal the cluster manager to trigger a failover action. On Linux, the SQL Server instance cannot communicate with the cluster. Monitoring for database health is done "outside-in". If user opted in for database level failover monitoring and failover (by setting the option `DB_FAILOVER=ON` when creating the availability group), the cluster will check if the database state is `ONLINE` every time when it runs a monitoring action. The cluster queries the state in `sys.databases`. For any state different than `ONLINE`, it will trigger a failover automatically (if automatic failover conditions are met). The actual time of the failover depends on the frequency of the monitoring action as well as the database state being updated in sys.databases.

## Next steps

[Configure Red Hat Enterprise Linux Cluster for SQL Server Availability Group Cluster Resources](sql-server-linux-availability-group-cluster-rhel.md)

[Configure SUSE Linux Enterprise Server Cluster for SQL Server Availability Group Cluster Resources](sql-server-linux-availability-group-cluster-sles.md)

[Configure Ubuntu Cluster for SQL Server Availability Group Cluster Resources](sql-server-linux-availability-group-cluster-ubuntu.md)
