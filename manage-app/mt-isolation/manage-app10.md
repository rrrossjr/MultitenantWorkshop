#  Multitenant Tenant Isolation

## Lab Introduction

This is a series of hands-on labs designed to familiarize you with the  Oracle Multitenant and Network isolation feature. In these labs, We will dive into the concepts of Database Firewalls , Resource managements and Lockdown features.

### Setup

#### Lab Assumptions

- Each participant has been provided a username and password to the tenancy c4u03.
- Each participant has completed the Environment Setup lab.
- Each participant has created an OCI compute instance using the database template.

There are two container databases running:

- CDB1 running on port 1523
- CDB2 running on port 1524

#### Lab Setup

All the scripts for this lab are located in the /home/oracle/labs/multitenant/scripts folder.

- To access the scripts, secure shell into the OCI compute instance.

- Change to the ssh directory and ssh into your instance. The public IP address can be found by going to Compute -> Instance.

   ```
   cd /home/oracle/labs/multitenant
   ```

- Reset the container databases back to their original ports if they were changed in a previous lab. If any errors about dropping databases appear they can be ignored.

   ```
   ./resetCDB.sh
   ```

##  Database Service Firewall

Database Service Firewall is a feature of Oracle Access Control List (ACL) since 12.2.

 Service-Level ACLs allow you to control access to specific services, including those associated with individual pluggable databases (PDBs). This functionality is part of the Database Service Firewall, which isn't specifically a multitenant feature, but it is useful for controlling access to PDBs.

![](./images/MT3_DB_service_firewall.png)



### SETUP STEPS

   The steps include



   - Install the ACL package
   - Configure the listener
   - Add the IPADDRESS to the whitelist for each PDB.
   - Verify/test.



####    **Step 1.  Install ACL package**

   You need a package DBMS\_SFW\_ACL\_ADMIN package. This is installed by running as sysdba. This package is owned by the DBSFWUSER schema. The procedures in this package can be run only by the DBSFWUSER user.

   ```
   <copy>sudo su - oracle

   sqlplus sys/oracle@//localhost:1523/cdb1 as sysdba  @$ORACLE_HOME/rdbms/admin/dbmsaclsrv.sql </copy>


   ```



   ```
   [opc@mtv30 ~]$ sudo su - oracle
   Last login: Mon Apr  6 21:20:38 GMT 2020 on pts/0
   [oracle@mtv30 ~]$ . oraenv
   ORACLE_SID = [CDB1] ?
   The Oracle base remains unchanged with value /u01/app/oracle
   [oracle@mtv30 ~]$ sqlplus /nolog

   SQL*Plus: Release 19.0.0.0.0 - Production on Tue Apr 7 22:35:20 2020
   Version 19.5.0.0.0

   Copyright (c) 1982, 2019, Oracle.  All rights reserved.

   SQL> conn sys/oracle@//localhost:1523/cdb1 as sysdba
   Connected.
   SQL> @$ORACLE_HOME/rdbms/admin/dbmsaclsrv.sql

   Session altered.

   Grant succeeded.

   Grant succeeded.

   Grant succeeded.

   Grant succeeded.

   Grant succeeded.

   Grant succeeded.

   Grant succeeded.

   Grant succeeded.

   Package created.

   Session altered.

   SQL>
   ```


####    **Step 2.  Configure the listener.**

   The LOCAL\_REGISTRATION\_ADDRESS\_lsnr\_alias and FIREWALL setting must be added to the "listener.ora" file. The default listener name is LISTENER and listeners on default port 1521. However In our example the CDB1 DB is listening on listener LISTCDB1. Example setting below.

   ```
   # LOCAL_REGISTRATION_ADDRESS_lsnr_alias = ON
   #LOCAL_REGISTRATION_ADDRESS_LISTENER = ON
   LOCAL_REGISTRATION_ADDRESS_LISTCDB1 = ON
   ```



   The `FIREWALL` attribute can be added to the listener endpoint to control the action of the database firewall.

   - `FIREWALL=ON` : Only connections matching an ACL are considered valid. All other connections are rejected.
   - `FIREWALL=OFF` : The firewall functionality is disabled, so all connections are considered valid.

**Take a backup of current listener.**

   You could open another termimal to take a backup.

   ```
   sudo su - oracle
   cd $ORACLE_HOME/network/admin
   cp listener.ora listener.backup
   ```



**Edit Listener.ora**

  <pre>
  LISTCDB2 =
    (DESCRIPTION_LIST =
      (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = mtv28imp.vjvcn.oraclevcn.com)(PORT = 1524))
    )
  )
LISTCDB1 =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = mtv28imp.vjvcn.oraclevcn.com)(PORT = 1523) <b><font color="red">(FIREWALL=ON)</font>) </b>
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1523))
    )
  )

<b><font color="red">LOCAL_REGISTRATION_ADDRESS_LISTCDB1 = ON</font></b>

</pre>


##### **Restart listener and verify FIREWALL= ON.**

```
lsnrctl stop listcdb1
lsnrctl start listener
lsnrctl status listcdb1
```
```
[oracle@mtv30 admin]$ lsnrctl stop listcdb1

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 09-APR-2020 18:49:57

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=mtv30.sub04061927430.mtw                                                                                                             orkshop.oraclevcn.com)(PORT=1523)(FIREWALL=ON)))
The command completed successfully
[oracle@mtv30 admin]$ lsnrctl start listcdb1

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 09-APR-2020 18:50:06

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Starting /u01/app/oracle/product/19c/dbhome_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 19.0.0.0.0 - Production
System parameter file is /u01/app/oracle/product/19c/dbhome_1/network/admin/list                                                                                                             ener.ora
Log messages written to /u01/app/oracle/diag/tnslsnr/mtv30/listcdb1/alert/log.xm                                                                                                             l
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=mtv30)(PORT=1523)(FIREWA                                                                                                             LL=ON)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1523)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=mtv30.sub04061927430.mtw                                                                                                             orkshop.oraclevcn.com)(PORT=1523)(FIREWALL=ON)))
STATUS of the LISTENER
------------------------
Alias                     listcdb1
Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
Start Date                09-APR-2020 18:50:06
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/19c/dbhome_1/network/admin/lis                                                                                                             tener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/mtv30/listcdb1/alert/log.                                                                                                             xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=mtv30)(PORT=1523)(FIREWALL=ON)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1523)))
The listener supports no services
The command completed successfully


[oracle@mtv30 admin]$ lsnrctl stat listcdb1

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 09-APR-2020 18:57:12

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=mtv30.sub04061927430.mtworkshop.oraclevcn.com)(PORT=1523)(FIREWALL=ON)))
STATUS of the LISTENER
------------------------
Alias                     listcdb1
Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
Start Date                09-APR-2020 18:50:06
Uptime                    0 days 0 hr. 7 min. 6 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/19c/dbhome_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/mtv30/listcdb1/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=mtv30)(PORT=1523)(FIREWALL=ON)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1523)))
Services Summary...
Service "CDB1" has 1 instance(s).
  Instance "CDB1", status READY, has 1 handler(s) for this service...
Service "CDB1XDB" has 1 instance(s).
  Instance "CDB1", status READY, has 1 handler(s) for this service...
Service "a206980f93d62602e0530200000ab752" has 1 instance(s).
  Instance "CDB1", status READY, has 1 handler(s) for this service...
Service "pdb1" has 1 instance(s).
  Instance "CDB1", status READY, has 1 handler(s) for this service...
The command completed successfully

````
It can take up to 5 minutes before all services have been registered again. If you want to speed this up, login to the CDB1 using SQL*Plus and execute the command 'alter system register'.
Once all services are available, specifically the PDB1 services, we can continue with the lab.


#### **Step 3: Add IP address to PDB whitelist.**

  Create a policy whitelist in access control list (ACL) containing hosts that are allowed access to a specific database service. Local listeners and server processes validate all inbound client connections against the ACL.

   Once the firewall is set and listener is restarted, We will need to add the IP address of every connection that can be accepted per PDB. We are creating a whitelist of all IP address that can connect to a service. In our multitenant environment, CDB1 and PDB1 are both services. We can add additional user defined service and add whitelist to them as well.
   Try to login to PDB1 in your localhost without any ipaddress in the whitelist.


```
 sqlplus sys/oracle@//localhost:1523/pdb1 as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Thu Apr 9 19:03:48 2020
Version 19.5.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

ERROR:
ORA-12506: TNS:listener rejected connection based on service ACL filtering
```

You will see that the connection fails with error ORA-12506. Since even the local host IP address is not included in the whitelist, the listener does not allow connections.
We are clearly not allowed to access this database based on the ACL filtering available. So now we are going to add our IP address to the whitelist for the (default) service PDB1.


````
<copy>
sqlplus / as sysdba
show pdbs
exec  dbsfwuser.DBMS_SFW_ACL_ADMIN.ip_add_ace('pdb1','localhost');
exec  dbsfwuser.DBMS_SFW_ACL_ADMIN.commit_acl;
</copy>

````
````
SQL>  exec dbsfwuser.DBMS_SFW_ACL_ADMIN.ip_add_ace('pdb1','localhost');
PL/SQL procedure successfully completed.

SQL> exec  dbsfwuser.DBMS_SFW_ACL_ADMIN.commit_acl;

PL/SQL procedure successfully completed.
````
Now connect using username/password from localhost to verify.

````
<copy>
conn sys/oracle@//localhost:1523/pdb1 as sysdba </copy>

SQL> conn sys/oracle@//localhost:1523/pdb1 as sysdba
Connected.
````
You can now connect to PDB1 successfully from localhost since we added localhost to the whitelist. If you need to test IP address or external hosts in this environment, we will have to open port 1523 in Oracle Virtual Network Connection (VNC) and add port 1523 in the linux firewall.  

The V$IP_ACL view lists the active ACLs.

````
- Display the saved ACLs.
COLUMN service_name FORMAT A30
COLUMN host FORMAT A30

-- Display the active ACLs.
SELECT service_name,
       host,
       con_id
FROM   v$ip_acl
ORDER BY 1, 2;


SERVICE_NAME                   HOST                               CON_ID
------------------------------ ------------------------------ ----------
PDB1                           10.0.0.2                                3
PDB1                           LOCALHOST                               3
PDB1                           MTV30.SUB04061927430.MTWORKSHO          3
                               P.ORACLEVCN.COM

````
This is the end of LAB. Please clean your environment so that it is ready for the next lab
- restore the original listener  and restart the listener.
````
cd $ORACLE_HOME/network/admin
cp  listener.backup listener.ora
lsnrctl stop LISTCDB1
lsnrctl start LISTCDB1
````

## Multitenant Lockdown

Tenant isolation is a key requirement for security in a multitenant environment. A PDB lockdown profile allows you to restrict the operations and functionality available from within a PDB. This can be very useful from a security perspective, giving the PDBs a greater degree of separation and allowing different people to manage each PDB, without compromising the security of other PDBs with the same instance.

A lockdown profile is a mechanism to restrict certain operations or functionalities in a PDB. This new Multitenant feature is managed by a CDB administrator and can be used to restrict user access in a particular PDB. A lockdown profile can prevent PDB users from:

Executing certain SQL statements, such as ALTER SYSTEM and ALTER SESSION,

Running procedures that access the network (e.g. UTL\_SMTP, UTL\_HTTP),
Accessing a common user’s objects,
Interacting with the OS (In addition to the capabilities covered by PDB\_OS\_CREDENTIAL),
Making unrestricted cross-PDB connections in a CDB,
Taking AWR snapshots,
Using JAVA partially or as a whole,
Using certain database options such as Advanced Queueing and Partitioning.

We can fulfill these requirements by creating a lockdown profile in our CDB Root and adding these restrictions to it. Before we move onto the “How?” part of this discussion, it’s worth mentioning a couple of important details about lockdown profiles:
We can fulfill these requirements by creating a lockdown profile in our CDB Root and adding these restrictions to it. Before we move onto the “How?” part of this discussion, it’s worth mentioning a couple of important details about lockdown profiles:

In order to be able to create a lockdown profile, you have to be a common user with CREATE LOCKDOWN PROFILE privilege and in order to enable a lockdown profile (either at the CDB or PDB level), you have to be common user with common ALTER SYSTEM or common SYSDBA privilege.

A single lockdown profile can have several rules defined in it. In other words, you don’t have to create a lockdown profile for every restriction you want to implement.

A PDB can have only one lockdown profile active at a time.

The restrictions enforced by a lockdown profile are PDB-wide, they affect every single user including the SYS and SYSTEM.

If you enable a lockdown profile in CDB Root, it affects all PDBs in the CDB. If you enable it in an Application Root (App Root), it affects all Application PDBs (App PDBs) under that App Root. If you enable it within a PDB, it only affects that PDB.




The steps are
-  Create lockdown profile
-  add statements to the lockdown profile which are disabled.
- set PDB_LOCKDOWN parameter


#### create a lockdown profile.

````
conn / as sysdba
show con_name
show pdbs

create lockdown profile TENANT_LOCK;
````

##### Add restrictions to profile

You can lockdown Oracle options such as partitions option or lockdown statements like alter system.
Eg alter lockdown profile sec_profile disable option=('Partitioning');

You can restrict all statements by using the "ALL" clause.
eg.  alter lockdown profile sec_profile disable statement=('alter system') clause=('set') option all;.

 The scope of the restriction can be reduced using the  CLAUSE, OPTION, MINVALUE, MAXVALUE options and values.
 <pre>
 eg. ALTER LOCKDOWN PROFILE hr_prof
     DISABLE STATEMENT = ('ALTER SYSTEM')
          CLAUSE = ('SET')
          OPTION = ('CPU_COUNT')
          MINVALUE = '2'
          MAXVALUE = '6';
</pre>

As a example, we will prevent a PDB to change the settings for the parameter CURSOR_SHARING. Changing this parameter could cause changes in performance and behavior and affect other tenants in the CDB. Adding a rule to the newly created TENANT\_LOCK lockdown profile is done with an ALTER LOCKDOWN PROFILE command. We will also lockdown the use of partitioning option into the lockdown profile.

````
<copy>
ALTER LOCKDOWN PROFILE TENANT_LOCK DISABLE STATEMENT=('alter system') CLAUSE=('set') OPTION=('cursor_sharing');

ALTER LOCKDOWN PROFILE TENANT_LOCK DISABLE OPTION=('Partitioning');
</copy>
````
````
SQL> ALTER LOCKDOWN PROFILE TENANT_LOCK DISABLE STATEMENT=('alter system') CLAUSE=('set') OPTION=('cursor_sharing');
Lockdown Profile altered.

SQL> ALTER LOCKDOWN PROFILE TENANT_LOCK DISABLE OPTION=('Partitioning');
Lockdown Profile altered.

````
Information about PDB lockdown profiles can be displayed using the DBA\_LOCKDOWN\_PROFILES view.
````
<copy>
SET LINESIZE 200
COLUMN profile_name format a20
COLUMN rule_type FORMAT A20
COLUMN rule_type FORMAT A10
COLUMN rule format a13
COLUMN clause FORMAT A5
COLUMN clause_option FORMAT A15

select profile_name, rule_type, rule, clause, clause_option, status, users from DBA_LOCKDOWN_PROFILES;
</copy>
````
````
SQL> select profile_name, rule_type, rule, clause, clause_option, status, users from DBA_LOCKDOWN_PROFILES;

PROFILE_NAME         RULE_TYPE  RULE          CLAUS CLAUSE_OPTION   STATUS  USERS
-------------------- ---------- ------------- ----- --------------- ------- ------
PRIVATE_DBAAS                                                       EMPTY
PUBLIC_DBAAS                                                        EMPTY
SAAS                                                                EMPTY
TENANT_LOCK          OPTION     PARTITIONING                        DISABLE ALL
TENANT_LOCK          STATEMENT  ALTER SYSTEM  SET   CURSOR_SHARING  DISABLE ALL
````

Once the restrictions are added to the lockdown profile, we need to set it. Let us verify you can change CURSOR_SHARING and create a partitioned table before the lockdown profile TENANT_LOCK is set.
Connect to container PDB1 and display the value of CURSOR_SHARING
````
<copy>alter session set container=PDB1;
show parameter cursor_sharing</copy>
````
````
SQL>  alter session set container=PDB1;
Session altered.

SQL> show parameter cursor_sharing
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
cursor_sharing                       string      EXACT
SQL>
````
Change the value oF CURSOR_SHARING to FORCE
````
SQL> <copy> alter system set cursor_sharing = FORCE; </copy>

System altered.

````
We have set the TENANT\_LOCK lockdown profile to prevent creation tables with partitions.
create a partitioned table before the profile is enabled.
````
SQL><copy> create table MyTable01 (id number) partition by hash (id) partitions 2;</copy>

Table created.
````
lockdown profile has not been set for this PDB, We saw that the DBA can create partitioned tables and alter initialization Parameters

Set the LOCKDOWN profile TENANT_LOCK

````
SQL> <copy>alter system set PDB_LOCKDOWN=TENANT_LOCK;</copy>
System altered.
SQL> show parameter pdb_lockdown

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
pdb_lockdown                         string      TENANT_LOCK

````
Now, try to change the value of  CURSOR_SHARING back to EXACT

````
SQL> <copy>alter system set cursor_sharing=EXACT;<copy>
alter system set cursor_sharing=EXACT
*
ERROR at line 1:
ORA-01031: insufficient privileges
````
Test to see if you can create a partitioned table when the profile is enabled.
````
SQL> <copy>create table MyTable02 (id number) partition by hash (id) partitions 2;</copy>
create table MyTable02 (id number) partition by hash (id) partitions 2
*
ERROR at line 1:
ORA-00439: feature not enabled: Partitioning

````
As you can see , We are not able to create a partitioned table or alter initialization parameters from PDB1 even with SYS privileges.
Now we will drop the lockdown profile and unset the parameter.

#### Drop LOCKDOWN profile

execute the drop command to drop the LOCKDOWN profile
````
<copy>Alter system set pdb_lockdown='';
conn / as SYSDBA
DROP LOCKDOWN PROFILE TENANT_LOCK;
 </copy>
````
This is the end of the lackdown lab.

## Resource Management
In a CDB, workloads within multiple PDBs can compete for system and CDB resources. Resource plans solve this problem.

In a multitenant environment, Resource Manager operates on two levels:
- CDB level :
Resource Manager can manage the workloads for multiple PDBs that are contending for system and CDB resources. You can specify how resources are allocated to PDBs, and you can limit the resource utilization of specific PDBs. The principal tool is a CDB resource plan. For Tenant isolation we need to implement at CDB level.

- PDB level :
Resource Manager can manage the workloads within each PDB. The principal tool is a PDB resource plan.

In a CDB with multiple PDBs, The Resource Manager enables you to prioritize and limit the resource usage of specific PDBs. With the Resource Manager, you can:

- Limit the CPU usage of a particular PDB

- Limit the number of parallel execution servers that a particular PDB can use

- Limit the memory usage of a particular PDB

- Specify the amount of memory guaranteed for a particular PDB

- Specify the maximum amount of memory a particular PDB can use

- Limit the I/O generated by specific PDBs

### Memory Resource Management

  The advantage of having multitenant is that, all the memory in a server can be provided to a single CDB and memory can be optimally utilized. However, in case one of the tenants use overly

PDB Memory Parameters
The following parameters can be set at the PDB level.

- DB\_CACHE\_SIZE : The minimum buffer cache size for the PDB.
- SHARED\_POOL\_SIZE : The minimum shared pool size for the PDB.
- PGA\_AGGREGATE\_LIMIT : The maximum PGA size for the PDB.
- PGA\_AGGREGATE\_TARGET : The target PGA size for the PDB.
- SGA\_MIN\_SIZE : The minimum SGA size for the PDB.
- SGA_TARGET : The maximum SGA size for the PDB.

ensure that the CDB and the other PDBs have sufficient memory for their operations. The initialization parameters control the memory usage of PDBs only if the following conditions are met:

The NONCDB_COMPATIBLE initialization parameter is set to false in the CDB root.

The MEMORY_TARGET initialization parameter is not set or is set to 0 (zero) in the CDB root.

Let us set SGA_TARGET for PDB1 to 1G. First verify the default settings to enable Memory management.

````
<copy>conn / as SYSDBA
show parameter  NONCDB_COMPATIBLE
show parameter MEMORY_TARGET </copy>
````

````
SQL> show parameter NONCDB_COMPATIBLE
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
noncdb_compatible                    boolean     FALSE

SQL> show parameter MEMORY_TARGET
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
memory_target                        big integer 0
````
Next check the value of SGA_TARGET in CDB1 and PDB1.

````
<copy>show parameter sga_target
ALTER SESSION SET CONTAINER=pdb1;
show parameter SGA_TARGET</copy>
````
````
SQL> show parameter sga_target
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
sga_target                           big integer 4432M

SQL> ALTER SESSION SET CONTAINER=pdb1;
Session altered.

SQL> SHOW PARAMETER sga_target;
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
sga_target                           big integer 0
````

Set SGA_TARGET for pdb1
````
ALTER SYSTEM SET sga_target=1G SCOPE=BOTH;
SHOW PARAMETER sga_target;
````
````
SQL> ALTER SYSTEM SET sga_target=1G SCOPE=BOTH;
System altered.
SQL> SHOW PARAMETER sga_target;
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
sga_target                           big integer 1G
````
You have successfully set a limit of memory PDB1 can use.

#### Monitoring Memory Usage for PDBs
Oracle now provides views to monitor the resource (CPU, I/O, parallel execution, memory) usage of PDBs. Each view contains similar information, but for different retention periods.

- V$RSRCPDBMETRIC : A single row per PDB, holding the last of the 1 minute samples.
- V$RSRCPDBMETRIC_HISTORY : 61 rows per PDB, holding the last 60 minutes worth of samples from the V$RSRCPDBMETRIC view.
- V$RSRC_PDB : Cumulative statistics since the CDB resource plan ws set.
- DBA\_HIST\_RSRC\_PDB\_METRIC : AWR snaphots, retained based on the AWR retention period.

### CPU Resources Management

Two ways to limit CPU resources
- Instance Caging
- Resource Manager


Instance caging is a technique that uses an initialization parameter to limit the number of CPUs that an instance can use simultaneously. You can set CPU\_COUNT at the PDB level. If Resource Manager is enabled, then the PDB is “caged” (restricted) to the number of CPUs specified by CPU\_COUNT. This count can be altered dynamically altered. In Oracle database hosting environment, This is useful to scale cpus up and down PDBs.  Another advantage is oracle can monitor resource usage. The resource limits can trigger to dynamically change the  CPU_COUNT. The CPUs will be available in all the session in the PDB immediately. This way, we can auto scale the CPUs for PDB tenant.

Resource Manager allows one Oracle process per CPU to run at a given time. All other processes wait on an internal Resource Manager run queue, under the wait event "resmgr:cpu quantum". Resource Manager allows an Oracle process to run for a small quantum of time (100 milliseconds). At the end of this quantum or when the Oracle process starts a wait (e.g. for a lock or I/O), Resource Manager selects a new Oracle process to run. Resource Manager uses a round-robin algorithm , and has priority Queues to choose between all runnable processes.

As mentioned above, to setup Instance Caging, we need to enable Resource Manager and set CPU_COUNT at PDB level.
````
<copy>conn / as SYSDBA
show parameter CPU_COUNT
show parameter resource_</copy>
````
set default resource manager plan
````
SQL> <copy> alter system set resource_manager_plan='DEFAULT_CDB_PLAN' ; </copy>

System altered.

````
set CPU count in pdb1
````
alter session set container=pdb1;
alter system set cpu_count=1;
show parameter CPU_COUNT
````
````
SQL> alter session set container=pdb1;
Session altered.

SQL> alter system set cpu_count=1;
System altered.

SQL> show parameter cpu_count
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
cpu_count                            integer     1
````
That is it. By setting the default resouce plan in CDB and CPU count in PDB , we have setup instance caging.
To test this, you can run the sample workload.

````
 SQL> <copy> host /home/oracle/labs/multitenant/cpu_test.sh</copy>
````
 open another session and run top command and observe the cpu busy percentage. It will be limited to the percentgate equal to one cpu. In our cace, Total cpus were 2, so 1 cpu would be 50% cpu utilization.

In a production system with many CPUs and PDB consolidation, it is possible to over provision. i.e The total of CPU_COUNT at PDB level is more than allocated at CDB level. This is a recommended configuration if we want better CPU utilization of the system.

Now unset the CPU caging and rerun the workload. In our test, the cpu utilization peaks to 100% consuming the 2 CPUs available.
````
 <copy>conn / as SYSDBA
 alter system set resource_manager_plan='';
 alter session set container=PDB1;
 alter system set cpu_count=0;
 host /home/oracle/labs/multitenant/cpu_test.sh</copy>
 ````
The workload without he resources runs faster as they make use of all the CPUs.

Resource Manager CPU.

Another way of managing CPU resources is through a defining a percentage or share to each PDB.
To allocate resources among PDBs, assign a share value to each PDB or performance profile. A higher share value results in more guaranteed resources for a PDB or the PDBs that use the performance profile.

A CDB resource plan is made up of CDB resource plan directives. The plan directives allocate shares, which define the proportion of the CDB resources available to the PDB, and specific utilization percentages, that give a finer level of control. CDB resource plans are managed using the DBMS_RESOURCE_MANAGER package. Each plan directive is made up of the following elements:

- pluggable\_database : The PDB the directive relates to.
- shares : The proportion of the CDB resources available to the PDB.
- utilization\_limit : A utilization limit restrains the system resource usage of a specific PDB or a specific PDB performance profile.
- parallel\_server\_limit : The percentage of the CDBs available parallel servers (PARALLEL_SERVERS_TARGET initialization parameter) that are available to the PDB.
- PDBs without a specific plan directive use the default PDB directive.

CPU_COUNT works the same way as the utilization_limit directive in the CDB plan. However, the CPU_COUNT limit is expressed in terms of number of CPUs rather than utilization percentage. If both the utilization_limit and CPU_COUNT are specified, then the lower limit is enforced.

CPU_COUNT is advantageous because when the PDB is plugged into a new container, the CPU_COUNT setting remains with the plugged-in PDB. Also, Oracle Database uses the CPU_COUNT setting for a PDB to derive many other PDB parameters, such as those for parallel execution.


The following figure shows an example of three PDBs with share values specified for them in a CDB resource plan.


<style>
	.demo {
		border:1px solid #C0C0C0;
		border-collapse:collapse;
		padding:5px;
	}
	.demo th {
		border:1px solid #C0C0C0;
		padding:5px;
		background:#F0F0F0;
	}
	.demo td {
		border:1px solid #C0C0C0;
		padding:5px;
	}
</style>
<table class="demo">
	<caption>Table 1</caption>
	<thead>
	<tr>
		<th>&nbsp;</th>
		<th>Shares</th>
		<th>Utilization Limit %</th>
		<th>Parallel Server Limit %</th>
	</tr>
	</thead>
	<tbody>
	<tr>
		<td>&nbsp;Default per PDB </td>
		<td>&nbsp;1</td>
		<td>&nbsp;100</td>
		<td>&nbsp;100</td>
	</tr>
	<tr>
		<td>&nbsp;PDB1</td>
		<td>&nbsp;2</td>
		<td>&nbsp;50</td>
		<td>&nbsp;20</td>
	</tr>
	<tr>
		<td>&nbsp;PDB2</td>
		<td>&nbsp;4</td>
		<td>&nbsp;75</td>
		<td>&nbsp;20</td>
	</tr>
	<tr>
		<td>&nbsp;PDB3</td>
		<td>&nbsp;14</td>
		<td>&nbsp;100</td>
		<td>&nbsp;100</td>
	</tr>
	</tbody>
</table>

PDB1 gets guaranteed 2 share of total 20 (2+4+14), so 10% of system resources (CPU, Exadata I/O
Bandwidth, queued parallel statements) –
PDB2  is allocated 4/20 or 20% and PDB3 is allocated 14/20 or 70%.

The tasks you will accomplish in this lab are:

Create a pluggable database OE in the container database CDB1
Create a resource profile to manage PDB1 and OE PDBs
run sample workloads.

Connect to CDB1
````
<copy>sqlplus /nolog
connect sys/oracle@localhost:1523/cdb1 as sysdba </copy>
````
Create a pluggable database PDB3_RM with an admin user of SOE
````
<copy>
create pluggable database PDB3_RM admin user soe identified by soe roles=(dba);
alter pluggable database PDB3_RM open;
alter session set container = oe;
grant create session, create table to soe;
alter user soe quota unlimited on system;
</copy>
````
Create a resource profile. Allocate 3/10 shares to pdb1 and 7/10 shares to PDB3_RM.
````
BEGIN
DBMS_RESOURCE_MANAGER.CREATE_PENDING_AREA();
DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN( plan => 'newcdb_plan',comment => 'CDB resource plan for newcdb');
DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN_DIRECTIVE(
    plan                  => 'newcdb_plan',
    pluggable_database    => 'PDB1',
    shares                => 3,
    utilization_limit     => 30,
    parallel_server_limit => 100);

    DBMS_RESOURCE_MANAGER.CREATE_CDB_PLAN_DIRECTIVE(
       plan                  => 'newcdb_plan',
       pluggable_database    => 'PDB3_RM',
       shares                => 7,
       utilization_limit     => 70,
       parallel_server_limit => 100);


DBMS_RESOURCE_MANAGER.VALIDATE_PENDING_AREA();
DBMS_RESOURCE_MANAGER.SUBMIT_PENDING_AREA();

END;
/
````

To enable a CDB resource plan:
````
ALTER SYSTEM SET RESOURCE_MANAGER_PLAN = 'newcdb_plan';
````

Now run out CPU test againt PDB1 and verify in another session that the CPU limit is at 30%.
alter session set container=pdb1;
host /home/oracle/labs/multitenant/cpu_test.sh

BEGIN
  DBMS_RESOURCE_MANAGER.UPDATE_CDB_PLAN_DIRECTIVE(
    plan                      => 'newcdb_plan',
    pluggable_database        => 'pdb1',
    new_shares                => 3,
    new_utilization_limit     => 11,
    new_parallel_server_limit => 20);
END;
/

### I/O Resource Management

I/O Resource Management for Non-Exadata Platforms (PDB only)
The MAX\_IOPS and MAX\_MBPS initialization parameters limit the disk I/O generated by a PDB.

A large amount of disk I/O can cause poor performance. Several factors can result in excess disk I/O, such as poorly designed SQL or index and table scans in high-volume transactions. If one PDB is generating a large amount of disk I/O, then it can degrade the performance of other PDBs in the same CDB.

Use one or both of the following initialization parameters to limit the I/O generated by a specific PDB:

- The ***MAX_IOPS*** initialization parameter limits the number of I/O operations for each second.

- The ***MAX_MBPS*** initialization parameter limits the megabytes for I/O operations for each second.

If you set both preceding initialization parameters for a single PDB, then Oracle Database enforces both limits. Note that these limits are not enforced for Oracle Exadata, which uses I/O Resource Management (IORM) to manage I/Os between PDBs.

If these initialization parameters are set with the CDB root as the current container, then the values become the default values for all containers in the CDB. If they are set with an application root as the current container, then the values become the default values for all application PDBs in the application container. When they are set with a PDB or application PDB as the current container, then the settings take precedence over the default settings in the CDB root or the application root. These parameters cannot be set in a non-CDB.

The default for both initialization parameters is 0 (zero). If these initialization parameters are set to 0 (zero) in a PDB, and the CDB root is set to 0, then there is no I/O limit for the PDB. If these initialization parameters are set to 0 (zero) in an application PDB, and its application root is set to 0, then there is no I/O limit for the application PDB.

Critical I/O operations, such as ones for the control file and password file, are exempted from the limit and continue to run even if the limit is reached. However, all I/O operations, including critical I/O operations, are counted when the number of I/O operations and the megabytes for I/O operations are calculated.

You can use the DBA\_HIST\_RSRC\_PDB\_METRIC view to calculate a reasonable I/O limit for a PDB. Consider the values in the following columns when calculating a limit: IOPS, IOMBPS, IOPS\_THROTTLE\_EXEMPT.
 The ***rsmgr:io rate limit*** wait event indicates that a limit was reached.

Example Limiting the I/O Generated by a PDB

With the PDB as the current container, run the following SQL statement to set the MAX_IOPS initialization parameter both in memory and in the SPFILE to a limit of 1,000 I/O operations for each second:

````ALTER SYSTEM SET MAX_IOPS = 1000 SCOPE = BOTH;````

Example 22-4 Limiting the Megabytes of I/O Generated by a PDB

With the PDB as the current container, run the following SQL statement to set the MAX_MBPS initialization parameter both in memory and in the SPFILE to a limit of 200 MB of I/O for each second:
````
ALTER SYSTEM SET MAX_MBPS = 200 SCOPE = BOTH;
````


The tasks you will accomplish in this lab are:

- Create a pluggable database OE in the container database CDB1
- Create a load against the pluggable database OE
- Compare  wait events and timing of workloads .

Connect to CDB1
````
<copy>sqlplus /nolog
connect sys/oracle@localhost:1523/cdb1 as sysdba </copy>
````
Create a pluggable database OE with an admin user of SOE
````
<copy>
create pluggable database oe admin user soe identified by soe roles=(dba);
alter pluggable database oe open;
alter session set container = oe;
grant create session, create table to soe;
alter user soe quota unlimited on system;
</copy>
````

Unset  resource\_manager\_plan in CDB  and  connect to OE and run workload.
````
alter system set resource_manager_plan='';
alter session set container=OE;
````
run a workload with MAX_IOPS=0.

````
<copy>
set timing on
-- unset any IOPS RM
alter system set  MAX_IOPS=0;

BEGIN
  EXECUTE IMMEDIATE ' create table test as select * from dba_objects';
  FOR i in 1..6  LOOP
    execute immediate 'insert into test select * from test';
    commit;
  END LOOP;
  execute immediate 'drop table test purge';
END;
/</copy>
PL/SQL procedure successfully completed.

Elapsed: 00:00:18.48

````
The workload runs without any resource manager in under 20 seconds.

Now set MAX_IOPS=1 and rerun the load.

````
<copy>alter system set  MAX_IOPS=1;
BEGIN
  EXECUTE IMMEDIATE ' create table test as select * from dba_objects';
  FOR i in 1..6  LOOP
    execute immediate 'insert into test select * from test';
    commit;
  END LOOP;
  execute immediate 'drop table test purge';
END;
/
</copy>
PL/SQL procedure successfully completed.

Elapsed: 00:01:14.17

````
Now the same workload takes about 250 to 300 seconds to run.
Open a new terminal window, sudo to the oracle user and query v$session_event to see IO resource wait event.
````
sudo su - oracle
sqlplus / as SYSDBA
select se.con_id,event,time_waited from v$session_event se,v$pdbs pdb where event like 'resmgr: %' and pdb.name='OE' and pdb.con_id=se.con_id ;

   CON_ID EVENT                          TIME_WAITED
---------- ------------------------------ -----------
        4 resmgr: I/O rate limit              185428
        4 resmgr: I/O rate limit                9016
````
