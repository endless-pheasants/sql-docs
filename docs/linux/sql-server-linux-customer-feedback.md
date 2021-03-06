---
# required metadata

title: Customer Feedback for SQL Server on Linux | Microsoft Docs
description: 
author: annashres 
ms.author: anshrest 
manager: jhubbard
ms.date: 06/26/2017
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
# Customer Feedback for SQL Server on Linux

By default, Microsoft SQL Server collects information about how its customers are using the application. Specifically, SQL Server collects information about the installation experience, usage, and performance. This information helps Microsoft improve the product to better meet customer needs. For example, Microsoft collects information about what kinds of error codes customers encounter so that we can fix related bugs, improve our documentation about how to use SQL Server, and determine whether features should be added to the product to better serve customers.

This document provides details about what kinds of information are collected and about how to configure Microsoft SQL Server on Linux to send that collected information to Microsoft. SQL Server 2017 includes a privacy statement that explains what information we do and do not collect from users. Please read the privacy statement.

Specifically, Microsoft does not send any of the following types of information through this mechanism:
- Any values from inside user tables
- Any logon credentials or other authentication information
- Personally Identifiable Information (PII)

SQL Server 2017 always collects and sends information about the installation experience from the setup process so that we can quickly find and fix any installation problems that the customer is experiencing. SQL Server 2017 can be configured not to send information (on a per-server instance basis) to Microsoft through **mssql-conf**. mssql-conf is a configuration script that installs with SQL Server 2017 for Red Hat Enterprise Linux, SUSE Linux Enterprise Server, and Ubuntu. 

> [!NOTE]
>  You can disable the sending of information to Microsoft only in paid versions of SQL Server.

## Disable Customer Feedback
This option lets you change if SQL Server sends feedback to Microsoft or not. By default, this value is set to true. To change the value, run the following commands:

1. Run the mssql-conf script as root with the "set" command for "telemetry.customerfeedback":

   ```bash
   sudo /opt/mssql/bin/mssql-conf set telemetry.customerfeedback false
   ```
2. Restart the SQL Server service:

   ```bash
   sudo systemctl restart mssql-server
   ```

## Local Audit for SQL Server on Linux Usage Feedback Collection

Microsoft SQL Server 2017 contains Internet-enabled features that can collect and send information about your computer or device ("standard computer information") to Microsoft. The Local Audit component of SQL Server Usage Feedback collection can write data collected by the service to a designated folder, representing the data (logs) that will be sent to Microsoft. The purpose of the Local Audit is to allow customers to see all data Microsoft collects with this feature, for compliance, regulatory or privacy validation reasons.

In SQL Server on Linux, Local Audit is configurable at instance level for SQL Server Database Engine. Other SQL Server components and SQL Server Tools do not have Local Audit capability for usage feedback collection.

### Enable Local Audit
This option enables Local Audit and lets you set the directory where the Local Audit logs are created.

1. Create the directory where the Local Audit logs will reside. For example, we will use /tmp/audit:

   ```bash
   sudo mkdir /tmp/audit
   ```

2. Change the owner and group of the directory to the "mssql" user:

   ```bash
   sudo chown mssql /tmp/audit
   sudo chgrp mssql /tmp/audit
   ```

3. Run the mssql-conf script as root with the "set" command for "telemetry.userrequestedlocalauditdirectory":

   ```bash
   sudo /opt/mssql/bin/mssql-conf set userrequestedlocalauditdirectory /tmp/audit
   ```
4. Restart the SQL Server service:

   ```bash
   sudo systemctl restart mssql-server
   ```
