# Migration Strategy

## Introduction

In this lab, you will have a look at the necessary steps to move a database running on your local environment to ADB.

Estimated Time: 5 Minutes

### Objectives

In this lab, you will:

* Familiarize yourself with ADB migration steps

### Prerequisites

This lab assumes:

- You have completed Lab 1: Initialize Environment

This is an optional lab. You can skip it if you are already familiar with ADB migration steps.

## Task 1: Strategy

For moving a database to ADB, we need to perform basically 4 steps:

### 1 - Checking source database for readiness.

To evaluate the compatibility of the source database before you migrate to an Oracle Cloud database, use the Cloud Premigration Advisor Tool (CPAT).

The purpose of the Cloud Premigration Advisor Tool (CPAT) is to help plan successful migrations to Oracle Databases in the Oracle Cloud or on-premises. It analyzes the compatibility of the source database with your database target and chosen migration method, and suggests a course of action for potential incompatibilities. CPAT provides you with information to consider for different migration tools.

### 2 - Evaluating the best Migration Method

There are multiple ways of migrating a database. You could ether use Data Pump, Database Links, Golden Gate, OCI DMS or ZDM. In this lab, we will check how Oracle Cloud Migration Advisor brings you the expert technical knowledge of Oracle Database upgrade and migration to give you the best possible migration advice.

### 3 - Performing the Migration

In this lab, we will move:

- The *RED* local PDB to *RUBY* running on ADB.
- The *BLUE* local PDB to *SAPPHIRE* running on ADB.

![Migration Strategy](./images/migration-strategy.png)  

We will explore using both a dump file in a shared NFS, as using the database link to move the data.

### 4 - Post steps

After migration has finished, on this lab you will check how to perform some maintainance tasks using the "Database Actions" page.

## Task 2: Test connection on the ADB Sapphire instance

To connect on the ADB instance, you must use a ADB Wallet, which is already uncompressed and available at _/home/oracle/adb_tls_wallet_. 

1. Use the *yellow* terminal 🟨. Set the environment to *SAPPHIRE* and connect.

    ```
    <copy>
    . sapphire
    sqlplus admin/Welcome_1234@sapphire_tp
    </copy>

    -- Be sure to hit RETURN
    ```
2. Check all the non-internal users already created on the database.

    ```
    <copy>
    select username
      from dba_users
     where oracle_maintained='N'
       and cloud_maintained='NO'
     order by 1
    /
    </copy>
    ```

    * `ADMIN` is the default DBA user on ADB.
    * `CMA` is the schema where CMA tool was deployed.
    * `MPACK_OEE` is a pre-created schema available in ADB Free Container with the State Explorer tool.
    * `ORDS_PLSQL_GATEWAY2` and `ORDS_PUBLIC_USER2` are schemas created to handle ORDS access.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    USERNAME
    --------------------------------------------------------------------------------
    ADMIN
    CMA
    MPACK_OEE
    ORDS_PLSQL_GATEWAY2
    ORDS_PUBLIC_USER2
    ```
    </details>    

3. Describe the package `DBMS_METADATA`.

    ```
    <copy>
    desc dbms_metadata
    </copy>
    ```

    * Data Pump uses `DBMS_METADATA` to extract the definition of objects during an export.
    * Data Pump stores the metadata information in an XML format in the dump file.
    * While importing metadata, Data Pump reads the XML from the dump file and translates that into DDL calls.
    * In a later lab, you'll try to extract metadata from the database.


    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> desc dbms_metadata
    FUNCTION ADD_TRANSFORM RETURNS NUMBER
     Argument Name                  Type                    In/Out Default?
     ------------------------------ ----------------------- ------ ----------
     HANDLE                         NUMBER                  IN
     NAME                           VARCHAR2                IN
     ENCODING                       VARCHAR2                IN     DEFAULT
     OBJECT_TYPE                    VARCHAR2                IN     DEFAULT
    FUNCTION CHECK_CONSTRAINT RETURNS NUMBER
     Argument Name                  Type                    In/Out Default?
     ------------------------------ ----------------------- ------ --------
     OBJ_NUM                        NUMBER                  IN

    (output truncated)

    PROCEDURE SET_XMLFORMAT
     Argument Name                  Type                    In/Out Default?
     ------------------------------ ----------------------- ------ --------
     HANDLE                         NUMBER                  IN
     NAME                           VARCHAR2                IN
     VALUE                          BOOLEAN                 IN     DEFAULT
    PROCEDURE TRANSFORM_STRM
     Argument Name                  Type                    In/Out Default?
     ------------------------------ ----------------------- ------ --------
     INDOC                           CLOB                   IN
     OUTDOC                          CLOB                   IN/OUT
     MDVERSION                       VARCHAR2               IN     DEFAULT
    ```
    </details>  

4. Examine Data Pump processes.

    ```
    <copy>
    select pname from v$process where pname='DM00' or pname like 'DW%';
    </copy>
    ```

    * The query returns no rows because no Data Pump job is running.
    * *DM00* is the control process that coordinates the Data Pump jobs. 
    * The control process can start one or more worker processes. Those are named *DW00*, *DW01*, and so forth.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> select pname from v$process where pname='DM00' or pname like 'DW%';
    
    no rows selected
    ```
    </details> 

5. Exit SQL*Plus.

    ```
    <copy>
    exit
    </copy>
    ```

## Task 2: Data Pump client

Normally, you start a Data Pump job using the command-line clients.

1. Find the Data Pump clients.

    ```
    <copy>
    ll $ORACLE_HOME/bin/*dp
    </copy>
    ```

    * There are two clients. `expdp` for exports, and `impdp` for imports.
    * You can also find the clients in a client installation. This allows you to start a Data Pump job remotely by just having a local client installation.
    
    <details>
    <summary>*click to see the output*</summary>
    ``` text
    -rwxr-x--x. 1 oracle oinstall 235128 May  2 19:09 /u01/app/oracle/product/19/bin/expdp
    -rwxr-x--x. 1 oracle oinstall 242992 May  2 19:09 /u01/app/oracle/product/19/bin/impdp
    ```
    </details> 


2. Examine the command line help.

    ```
    <copy>
    expdp -help
    </copy>
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    Export: Release 19.0.0.0.0 - Production on Fri Apr 25 07:17:10 2025
    Version 19.27.0.0.0
    
    Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.
    
    
    The Data Pump export utility provides a mechanism for transferring data objects
    between Oracle databases. The utility is invoked with the following command:
    
    (output truncated)
    
    STOP_WORKER
    Stops a hung or stuck worker.
    
    TRACE
    Set trace/debug flags for the current job.
    ```
    </details>     


You may now *proceed to the next lab*.

## Acknowledgments

* **Author** - Rodrigo Jorge
* **Contributors** - William Beauregard, Daniel Overby Hansen, Mike Dietrich, Klaus Gronau, Alex Zaballa
* **Last Updated By/Date** - Rodrigo Jorge, May 2025