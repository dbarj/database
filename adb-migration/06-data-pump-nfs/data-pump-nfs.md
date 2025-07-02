# Migration using Data Pump with NFS

## Introduction

Finally, after performing all the analysis, it's time to start our migration.

In this lab, we will migrate the *BLUE* PDB to the *SAPPHIRE* ADB using Data Pump.

Estimated Time: 10 Minutes

### Objectives

In this lab, you will:

* Setup NFS on both source env and target ADB to share the Data Pump dump file.
* Generate a full schema export on the source database.
* Import the generated dump on the target database. 

### Prerequisites

This lab assumes:

- You have completed Lab 3: Evaluate Database Compatibility

## Task 1: Setup NFS server

There are multiple ways you can import a dump file in ADB. You could use Object Storage, move it to an Oracle Directory with UTL_FILE, or setup a NFS Server.

In this lab, we will setup a NFS Server that is going to be visible by both our source and target databases and use it to move and load our Data Pump dump file.

1. Use the *yellow* terminal 🟨. Ensure NFS Server is running.

    ``` shell
    <copy>
    sudo podman restart nfs-server
    </copy>

    # Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [ADB:oracle@holserv1:~]$ sudo podman restart nfs-server
    nfs-server
    ```
    </details>

2. Mount the NFS share available at server *nfs-server:/exports* on the localhost.

    ``` shell
    <copy>
    sudo mkdir -p /nfs_mount
    sudo mount -t nfs nfs-server:/exports /nfs_mount
    ls -l /nfs_mount
    </copy>

    # Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [ADB:oracle@holserv1:~]$ sudo mkdir -p /nfs_mount
    [ADB:oracle@holserv1:~]$ sudo mount -t nfs nfs-server:/exports /nfs_mount
    [ADB:oracle@holserv1:~]$ ls -l /nfs_mount
    total 0
    -rw-r--r--. 1 root root 0 Jul  1 19:13 WORKING
    ```
    </details>

## Task 2: Export the *BLUE* PDB

1. Connect on the *BLUE* PDB to create a directory. 

    ``` shell
    <copy>
    . cdb23
    sqlplus sys/oracle@//localhost:1521/blue as sysdba
    </copy>

    -- Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [CDB23:oracle@holserv1:~]$ . cdb23
    [CDB23:oracle@holserv1:~]$ sqlplus sys/oracle@//localhost:1521/blue as sysdba

    SQL*Plus: Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems on Tue Jul 1 19:21:07 2025
    Version 23.8.0.25.04

    Copyright (c) 1982, 2025, Oracle.  All rights reserved.

    Last Successful login time: Tue Jul 01 2025 19:20:34 +00:00

    Connected to:
    Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
    Version 23.8.0.25.04

    SQL>
    ```
    </details>

2. Create a directory pointing to */nfs_mount*. 

    ``` shell
    <copy>
    create directory nfs_dir as '/nfs_mount';
    exit;
    </copy>

    -- Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> create directory nfs_dir as '/nfs_mount';

    Directory created.

    SQL> exit;
    Disconnected from Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
    Version 23.8.0.25.04
    ```
    </details>

3. Export all the schemas of *BLUE* PDB. 

    ``` shell
    <copy>
    expdp userid=system/oracle@//localhost:1521/blue \
    schemas=HR,PM,IX,SH,BI \
    logtime=all \
    metrics=true \
    directory=nfs_dir \
    dumpfile=schemas_export.dmp \
    logfile=schemas_export.log
    </copy>

    # Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    Export: Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems on Tue Jul 1 20:11:39 2025
    Version 23.8.0.25.04

    Copyright (c) 1982, 2025, Oracle and/or its affiliates.  All rights reserved.

    Connected to: Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
    01-JUL-25 20:11:44.291: Starting "SYSTEM"."SYS_EXPORT_SCHEMA_01":  userid=system/********@//localhost:1521/blue schemas=HR,PM,IX,SH,BI logtime=all metrics=true directory=nfs_dir dumpfile=schemas_export.dmp logfile=schemas_export.log
    01-JUL-25 20:11:44.959: W-1 Startup on instance 1 took 0 seconds
    01-JUL-25 20:11:47.746: W-1 Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
    01-JUL-25 20:11:47.968: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
    01-JUL-25 20:11:48.022: W-1      Completed 54 INDEX_STATISTICS objects in 1 seconds
    01-JUL-25 20:11:48.153: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/BITMAP_INDEX/INDEX_STATISTICS
    01-JUL-25 20:11:48.161: W-1      Completed 15 INDEX_STATISTICS objects in 0 seconds
    01-JUL-25 20:11:48.228: W-1 Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
    01-JUL-25 20:11:48.240: W-1      Completed 38 TABLE_STATISTICS objects in 0 seconds
    01-JUL-25 20:11:53.739: W-1      Completed 1 [internal] STATISTICS objects in 5 seconds
    01-JUL-25 20:11:53.825: W-1 Processing object type SCHEMA_EXPORT/USER
    01-JUL-25 20:11:53.836: W-1      Completed 5 USER objects in 0 seconds
    01-JUL-25 20:11:53.866: W-1 Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
    01-JUL-25 20:11:53.880: W-1      Completed 44 SYSTEM_GRANT objects in 0 seconds
    01-JUL-25 20:11:53.971: W-1 Processing object type SCHEMA_EXPORT/ROLE_GRANT
    01-JUL-25 20:11:53.978: W-1      Completed 11 ROLE_GRANT objects in 0 seconds
    01-JUL-25 20:11:53.997: W-1 Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
    01-JUL-25 20:11:54.003: W-1      Completed 5 DEFAULT_ROLE objects in 1 seconds
    01-JUL-25 20:11:54.041: W-1 Processing object type SCHEMA_EXPORT/TABLESPACE_QUOTA
    01-JUL-25 20:11:54.047: W-1      Completed 5 TABLESPACE_QUOTA objects in 0 seconds
    01-JUL-25 20:11:54.148: W-1 Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA/AQ
    01-JUL-25 20:11:54.419: W-1      Completed 2 AQ objects in 0 seconds
    01-JUL-25 20:11:54.421: W-1 Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA/LOGREP
    01-JUL-25 20:11:54.423: W-1      Completed 10 LOGREP objects in 0 seconds
    01-JUL-25 20:11:54.627: W-1 Processing object type SCHEMA_EXPORT/SYNONYM/SYNONYM
    01-JUL-25 20:11:54.634: W-1      Completed 8 SYNONYM objects in 0 seconds
    01-JUL-25 20:11:54.661: W-1 Processing object type SCHEMA_EXPORT/DB_LINK
    01-JUL-25 20:11:54.665: W-1      Completed 1 DB_LINK objects in 0 seconds
    01-JUL-25 20:11:55.510: W-1 Processing object type SCHEMA_EXPORT/TYPE/TYPE_SPEC
    01-JUL-25 20:11:55.516: W-1      Completed 4 TYPE objects in 0 seconds
    01-JUL-25 20:11:55.752: W-1 Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
    01-JUL-25 20:11:55.758: W-1      Completed 5 SEQUENCE objects in 0 seconds
    01-JUL-25 20:12:00.674: W-1 Processing object type SCHEMA_EXPORT/TABLE/TABLE
    01-JUL-25 20:12:26.367: W-1      Completed 35 TABLE objects in 28 seconds
    01-JUL-25 20:12:26.733: W-1 Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
    01-JUL-25 20:12:26.739: W-1      Completed 10 OBJECT_GRANT objects in 0 seconds
    01-JUL-25 20:12:26.872: W-1 Processing object type SCHEMA_EXPORT/TABLE/COMMENT
    01-JUL-25 20:12:26.914: W-1      Completed 127 COMMENT objects in 0 seconds
    01-JUL-25 20:12:28.189: W-1 Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
    01-JUL-25 20:12:28.196: W-1      Completed 2 PROCEDURE objects in 0 seconds
    01-JUL-25 20:12:28.524: W-1 Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
    01-JUL-25 20:12:28.529: W-1      Completed 2 ALTER_PROCEDURE objects in 0 seconds
    01-JUL-25 20:12:30.576: W-1 Processing object type SCHEMA_EXPORT/VIEW/VIEW
    01-JUL-25 20:12:30.584: W-1      Completed 8 VIEW objects in 2 seconds
    01-JUL-25 20:12:32.017: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
    01-JUL-25 20:12:32.068: W-1      Completed 22 INDEX objects in 2 seconds
    01-JUL-25 20:12:34.649: W-1 Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
    01-JUL-25 20:12:34.662: W-1      Completed 21 CONSTRAINT objects in 2 seconds
    01-JUL-25 20:12:40.698: W-1 Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
    01-JUL-25 20:12:40.708: W-1      Completed 20 REF_CONSTRAINT objects in 0 seconds
    01-JUL-25 20:12:41.188: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/BITMAP_INDEX/INDEX
    01-JUL-25 20:12:41.485: W-1      Completed 15 INDEX objects in 1 seconds
    01-JUL-25 20:12:41.595: W-1 Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
    01-JUL-25 20:12:41.600: W-1      Completed 2 TRIGGER objects in 0 seconds
    01-JUL-25 20:12:58.954: W-1 Processing object type SCHEMA_EXPORT/MATERIALIZED_VIEW
    01-JUL-25 20:12:58.960: W-1      Completed 2 MATERIALIZED_VIEW objects in 17 seconds
    01-JUL-25 20:13:17.352: W-1 Processing object type SCHEMA_EXPORT/DIMENSION
    01-JUL-25 20:13:17.359: W-1      Completed 5 DIMENSION objects in 0 seconds
    01-JUL-25 20:13:17.940: W-1 Processing object type SCHEMA_EXPORT/TABLE/POST_INSTANCE/PROCACT_INSTANCE/AQ
    01-JUL-25 20:13:17.945: W-1      Completed 30 AQ objects in 0 seconds
    01-JUL-25 20:13:18.142: W-1 Processing object type SCHEMA_EXPORT/TABLE/POST_INSTANCE/PROCDEPOBJ/RULE
    01-JUL-25 20:13:18.145: W-1      Completed 12 RULE objects in 0 seconds
    01-JUL-25 20:13:18.148: W-1 Processing object type SCHEMA_EXPORT/TABLE/POST_INSTANCE/PROCDEPOBJ/AQ
    01-JUL-25 20:13:18.150: W-1      Completed 8 AQ objects in 0 seconds
    01-JUL-25 20:13:18.436: W-1 Processing object type SCHEMA_EXPORT/POST_SCHEMA/PROCOBJ/RULE
    01-JUL-25 20:13:18.440: W-1      Completed 12 RULE objects in 0 seconds
    01-JUL-25 20:13:18.600: W-1 Processing object type SCHEMA_EXPORT/POST_SCHEMA/PROCACT_SCHEMA/AQ
    01-JUL-25 20:13:19.298: W-1      Completed 2 AQ objects in 1 seconds
    01-JUL-25 20:13:19.599: W-1 . . exported "SH"."CUSTOMERS"                             10.3 MB   55500 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.159: W-1 . . exported "PM"."PRINT_MEDIA"                          191.3 KB       4 rows in 1 seconds using external_table
    01-JUL-25 20:13:20.205: W-1 . . exported "PM"."TEXTDOCS_NESTEDTAB"                      88 KB      12 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.300: W-1 . . exported "SH"."SALES":"SALES_Q4_2001"                  2.3 MB   69749 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.362: W-1 . . exported "SH"."SALES":"SALES_Q3_1999"                  2.2 MB   67138 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.424: W-1 . . exported "SH"."SALES":"SALES_Q3_2001"                  2.1 MB   65769 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.483: W-1 . . exported "SH"."SALES":"SALES_Q2_2001"                  2.1 MB   63292 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.551: W-1 . . exported "SH"."SALES":"SALES_Q1_1999"                  2.1 MB   64186 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.609: W-1 . . exported "SH"."SALES":"SALES_Q1_2001"                    2 MB   60608 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.670: W-1 . . exported "SH"."SALES":"SALES_Q4_1999"                    2 MB   62388 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.732: W-1 . . exported "SH"."SALES":"SALES_Q1_2000"                    2 MB   62197 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.792: W-1 . . exported "SH"."SALES":"SALES_Q3_2000"                  1.9 MB   58950 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.857: W-1 . . exported "SH"."SALES":"SALES_Q4_2000"                  1.8 MB   55984 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.915: W-1 . . exported "SH"."SALES":"SALES_Q2_2000"                  1.8 MB   55515 rows in 0 seconds using direct_path
    01-JUL-25 20:13:20.974: W-1 . . exported "SH"."SALES":"SALES_Q2_1999"                  1.8 MB   54233 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.029: W-1 . . exported "SH"."SALES":"SALES_Q3_1998"                  1.6 MB   50515 rows in 1 seconds using direct_path
    01-JUL-25 20:13:21.084: W-1 . . exported "SH"."SALES":"SALES_Q4_1998"                  1.6 MB   48874 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.137: W-1 . . exported "SH"."SALES":"SALES_Q1_1998"                  1.4 MB   43687 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.184: W-1 . . exported "SH"."SALES":"SALES_Q2_1998"                  1.2 MB   35758 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.242: W-1 . . exported "SH"."SUPPLEMENTARY_DEMOGRAPHICS"           698.2 KB    4500 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.253: W-1 . . exported "IX"."STREAMS_QUEUE_TABLE"                      0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:21.328: W-1 . . exported "SH"."TIMES"                                383.3 KB    1826 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.365: W-1 . . exported "SH"."FWEEK_PSCAT_SALES_MV"                 420.2 KB   11266 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.376: W-1 . . exported "IX"."ORDERS_QUEUETABLE"                        0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:21.444: W-1 . . exported "SH"."COSTS":"COSTS_Q4_2001"                278.8 KB    9011 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.497: W-1 . . exported "SH"."COSTS":"COSTS_Q3_2001"                234.9 KB    7545 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.555: W-1 . . exported "SH"."COSTS":"COSTS_Q1_2001"                228.3 KB    7328 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.612: W-1 . . exported "SH"."COSTS":"COSTS_Q2_2001"                  185 KB    5882 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.666: W-1 . . exported "SH"."COSTS":"COSTS_Q1_1999"                  184 KB    5884 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.720: W-1 . . exported "SH"."COSTS":"COSTS_Q4_2000"                160.7 KB    5088 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.775: W-1 . . exported "SH"."COSTS":"COSTS_Q4_1999"                159.5 KB    5060 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.829: W-1 . . exported "SH"."COSTS":"COSTS_Q3_2000"                151.9 KB    4798 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.890: W-1 . . exported "SH"."COSTS":"COSTS_Q4_1998"                145.1 KB    4577 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.939: W-1 . . exported "SH"."COSTS":"COSTS_Q1_1998"                139.9 KB    4411 rows in 0 seconds using direct_path
    01-JUL-25 20:13:21.981: W-1 . . exported "SH"."COSTS":"COSTS_Q3_1999"                137.8 KB    4336 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.023: W-1 . . exported "SH"."COSTS":"COSTS_Q2_1999"                  133 KB    4179 rows in 1 seconds using direct_path
    01-JUL-25 20:13:22.065: W-1 . . exported "SH"."COSTS":"COSTS_Q3_1998"                131.5 KB    4129 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.106: W-1 . . exported "SH"."COSTS":"COSTS_Q2_2000"                119.4 KB    3715 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.144: W-1 . . exported "SH"."COSTS":"COSTS_Q1_2000"                  121 KB    3772 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.183: W-1 . . exported "SH"."COSTS":"COSTS_Q2_1998"                 79.9 KB    2397 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.217: W-1 . . exported "SH"."PROMOTIONS"                            59.6 KB     503 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.248: W-1 . . exported "SH"."PRODUCTS"                              27.6 KB      72 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.280: W-1 . . exported "HR"."EMPLOYEES"                             17.5 KB     107 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.309: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_S"             12.3 KB       1 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.338: W-1 . . exported "IX"."AQ$_ORDERS_QUEUETABLE_S"               11.9 KB       4 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.368: W-1 . . exported "SH"."COUNTRIES"                             10.9 KB      23 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.378: W-1 . . exported "IX"."AQ$_ORDERS_QUEUETABLE_H"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.381: W-1 . . exported "IX"."AQ$_ORDERS_QUEUETABLE_I"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.417: W-1 . . exported "HR"."LOCATIONS"                              8.7 KB      23 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.420: W-1 . . exported "IX"."AQ$_ORDERS_QUEUETABLE_L"                  0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.450: W-1 . . exported "SH"."CHANNELS"                               7.7 KB       5 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.480: W-1 . . exported "HR"."JOB_HISTORY"                            7.4 KB      10 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.514: W-1 . . exported "HR"."JOBS"                                   7.3 KB      19 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.545: W-1 . . exported "HR"."DEPARTMENTS"                            7.3 KB      27 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.696: W-1 . . exported "HR"."COUNTRIES"                              6.5 KB      25 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.731: W-1 . . exported "SH"."CAL_MONTH_SALES_MV"                     6.5 KB      48 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.760: W-1 . . exported "HR"."REGIONS"                                5.6 KB       4 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.771: W-1 . . exported "IX"."AQ$_ORDERS_QUEUETABLE_G"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.774: W-1 . . exported "IX"."AQ$_ORDERS_QUEUETABLE_T"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.777: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_C"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.781: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_G"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.784: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_H"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.787: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_I"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.795: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_L"                0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.804: W-1 . . exported "IX"."AQ$_STREAMS_QUEUE_TABLE_T"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:13:22.812: W-1 . . exported "SH"."COSTS":"COSTS_1995"                       0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.814: W-1 . . exported "SH"."COSTS":"COSTS_1996"                       0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.817: W-1 . . exported "SH"."COSTS":"COSTS_H1_1997"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.819: W-1 . . exported "SH"."COSTS":"COSTS_H2_1997"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.821: W-1 . . exported "SH"."COSTS":"COSTS_Q1_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.824: W-1 . . exported "SH"."COSTS":"COSTS_Q1_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.826: W-1 . . exported "SH"."COSTS":"COSTS_Q2_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.828: W-1 . . exported "SH"."COSTS":"COSTS_Q2_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.831: W-1 . . exported "SH"."COSTS":"COSTS_Q3_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.833: W-1 . . exported "SH"."COSTS":"COSTS_Q3_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.835: W-1 . . exported "SH"."COSTS":"COSTS_Q4_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.838: W-1 . . exported "SH"."COSTS":"COSTS_Q4_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.841: W-1 . . exported "SH"."SALES":"SALES_1995"                       0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.843: W-1 . . exported "SH"."SALES":"SALES_1996"                       0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.845: W-1 . . exported "SH"."SALES":"SALES_H1_1997"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.848: W-1 . . exported "SH"."SALES":"SALES_H2_1997"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.850: W-1 . . exported "SH"."SALES":"SALES_Q1_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.853: W-1 . . exported "SH"."SALES":"SALES_Q1_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.855: W-1 . . exported "SH"."SALES":"SALES_Q2_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.857: W-1 . . exported "SH"."SALES":"SALES_Q2_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.860: W-1 . . exported "SH"."SALES":"SALES_Q3_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.862: W-1 . . exported "SH"."SALES":"SALES_Q3_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.865: W-1 . . exported "SH"."SALES":"SALES_Q4_2002"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:22.867: W-1 . . exported "SH"."SALES":"SALES_Q4_2003"                    0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:13:24.306: W-1      Completed 89 SCHEMA_EXPORT/TABLE/TABLE_DATA objects in 3 seconds
    01-JUL-25 20:13:24.751: W-1 Master table "SYSTEM"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
    01-JUL-25 20:13:24.812: ******************************************************************************
    01-JUL-25 20:13:24.812: Dump file set for SYSTEM.SYS_EXPORT_SCHEMA_01 is:
    01-JUL-25 20:13:24.812:   /nfs_mount/schemas_export.dmp
    01-JUL-25 20:13:24.823: Job "SYSTEM"."SYS_EXPORT_SCHEMA_01" successfully completed at Tue Jul 1 20:13:24 2025 elapsed 0 00:01:44
    ```
    </details>

4. Verify that the dump file was generated on the NFS folder. 

    ``` shell
    <copy>
    ls -l /nfs_mount
    </copy>

    # Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [CDB23:oracle@holserv1:~]$ ls -l /nfs_mount
    total 52088
    -rw-r-----. 1 oracle oinstall 53313536 Jul  1 19:37 schemas_export.dmp
    -rw-r--r--. 1 oracle oinstall    22570 Jul  1 19:37 schemas_export.log
    -rw-r--r--. 1 root   root            0 Jul  1 19:13 WORKING
    ```
    </details>

## Task 3: Share NFS with ADB

1. Connect on the *SAPPHIRE* ADB to create a directory. 

    ``` shell
    <copy>
    . adb
    sqlplus admin/Welcome_1234@sapphire_tp
    </copy>

    -- Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [CDB23:oracle@holserv1:~]$ . adb
    [ADB:oracle@holserv1:~]$ sql admin/Welcome_1234@sapphire_tp

    SQL*Plus: Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems on Tue Jul 1 19:42:28 2025
    Version 23.8.0.25.04

    Copyright (c) 1982, 2025, Oracle.  All rights reserved.

    Last Successful login time: Tue Jul 01 2025 17:37:59 +00:00

    Connected to:
    Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
    Version 23.8.0.25.05

    SQL>
    ```
    </details>

2. Create a directory pointing to *nfs-server:/exports*.

    ``` shell
    <copy>
    create directory nfs_dir as 'nfs';
    begin
      dbms_cloud_admin.attach_file_system (
          file_system_name      => 'nfs',
          file_system_location  => 'nfs-server:/exports',
          directory_name        => 'nfs_dir',
          description           => 'Source NFS for data'
      );
    end;
    /
    select * from dbms_cloud.list_files('nfs_dir');
    </copy>

    -- Be sure to hit RETURN
    ```

    * Note that the *nfs_dir* directory was created and can read the contents of the NFS share.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> create directory nfs_dir as 'nfs';

    Directory NFS_DIR created.

    SQL> begin
      2    dbms_cloud_admin.attach_file_system (
      3        file_system_name      => 'nfs',
      4        file_system_location  => 'nfs-server:/exports',
      5        directory_name        => 'nfs_dir',
      6        description           => 'Source NFS for data'
      7    );
      8  end;
      9* /

    PL/SQL procedure successfully completed.

    SQL> select * from dbms_cloud.list_files('nfs_dir');

    OBJECT_NAME                 BYTES CHECKSUM    CREATED    LAST_MODIFIED
    _____________________ ___________ ___________ __________ ______________________________________
    WORKING                         0                        01-JUL-25 07.13.46.691421000 PM GMT
    schemas_export.log          22570                        01-JUL-25 07.37.18.375691000 PM GMT
    schemas_export.dmp       53313536                        01-JUL-25 07.37.16.256638000 PM GMT

    SQL>
    ```
    </details>

## Task 4: Import schemas in ADB

1. Import all the 5 schemas on *SAPPHIRE* ADB. 

    ``` shell
    <copy>
    . adb
    impdp userid=admin/Welcome_1234@sapphire_tpurgent \
    schemas=HR,PM,IX,SH,BI \
    logtime=all \
    metrics=true \
    directory=nfs_dir \
    dumpfile=schemas_export.dmp \
    logfile=schemas_import.log
    </copy>

    # Be sure to hit RETURN
    ```

    * Note that the only error reported was with the DB Link that points to F1 schema on *RED* PDB. We will handle that later.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    Import: Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems on Tue Jul 1 20:24:48 2025
    Version 23.8.0.25.04

    Copyright (c) 1982, 2025, Oracle and/or its affiliates.  All rights reserved.

    Connected to: Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
    01-JUL-25 20:24:54.295: W-1 Startup on instance 1 took 1 seconds
    01-JUL-25 20:24:56.001: W-1 Master table "ADMIN"."SYS_IMPORT_SCHEMA_01" successfully loaded/unloaded
    01-JUL-25 20:24:56.515: Starting "ADMIN"."SYS_IMPORT_SCHEMA_01":  userid=admin/********@sapphire_tpurgent schemas=HR,PM,IX,SH,BI logtime=all metrics=true directory=nfs_dir dumpfile=schemas_export.dmp logfile=schemas_import.log
    01-JUL-25 20:24:56.583: W-1 Processing object type SCHEMA_EXPORT/USER
    01-JUL-25 20:24:56.879: W-1      Completed 5 USER objects in 0 seconds
    01-JUL-25 20:24:56.879: W-1 Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
    01-JUL-25 20:24:56.984: W-1      Completed 44 SYSTEM_GRANT objects in 0 seconds
    01-JUL-25 20:24:56.984: W-1 Processing object type SCHEMA_EXPORT/ROLE_GRANT
    01-JUL-25 20:24:57.083: W-1      Completed 11 ROLE_GRANT objects in 1 seconds
    01-JUL-25 20:24:57.083: W-1 Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
    01-JUL-25 20:24:57.157: W-1      Completed 5 DEFAULT_ROLE objects in 0 seconds
    01-JUL-25 20:24:57.157: W-1 Processing object type SCHEMA_EXPORT/TABLESPACE_QUOTA
    01-JUL-25 20:24:57.231: W-1      Completed 5 TABLESPACE_QUOTA objects in 0 seconds
    01-JUL-25 20:24:57.265: W-1 Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA/AQ
    01-JUL-25 20:24:57.403: W-1      Completed 1 AQ objects in 0 seconds
    01-JUL-25 20:24:57.406: W-1 Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA/LOGREP
    01-JUL-25 20:24:57.445: W-1      Completed 5 LOGREP objects in 0 seconds
    01-JUL-25 20:24:57.455: W-1 Processing object type SCHEMA_EXPORT/SYNONYM/SYNONYM
    01-JUL-25 20:24:57.535: W-1      Completed 8 SYNONYM objects in 0 seconds
    01-JUL-25 20:24:57.535: W-1 Processing object type SCHEMA_EXPORT/DB_LINK
    01-JUL-25 20:24:57.604: ORA-31685: Object type DB_LINK:"SH"."F1" failed due to insufficient privileges. Failing sql is:
    CREATE DATABASE LINK "F1"  CONNECT TO "F1" IDENTIFIED BY VALUES ':1'  USING '//localhost:1521/red'

    01-JUL-25 20:24:57.628: W-1      Completed 1 DB_LINK objects in 0 seconds
    01-JUL-25 20:24:57.628: W-1 Processing object type SCHEMA_EXPORT/TYPE/TYPE_SPEC
    01-JUL-25 20:24:57.863: W-1      Completed 4 TYPE objects in 0 seconds
    01-JUL-25 20:24:57.863: W-1 Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
    01-JUL-25 20:24:57.963: W-1      Completed 5 SEQUENCE objects in 0 seconds
    01-JUL-25 20:24:57.963: W-1 Processing object type SCHEMA_EXPORT/TABLE/TABLE
    01-JUL-25 20:25:06.542: W-1      Completed 35 TABLE objects in 9 seconds
    01-JUL-25 20:25:06.563: W-1 Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
    01-JUL-25 20:25:06.733: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_C"                0 KB       0 rows in 0 seconds using direct_path
    01-JUL-25 20:25:07.791: W-1 . . imported "HR"."REGIONS"                                5.6 KB       4 rows in 1 seconds using external_table
    01-JUL-25 20:25:07.803: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_G"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:07.805: W-1 . . imported "SH"."SALES":"SALES_1996"                       0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:07.808: W-1 . . imported "SH"."SALES":"SALES_H1_1997"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:07.810: W-1 . . imported "SH"."SALES":"SALES_H2_1997"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:08.101: W-1 . . imported "SH"."SALES":"SALES_Q1_1998"                  1.4 MB   43687 rows in 1 seconds using external_table
    01-JUL-25 20:25:08.376: W-1 . . imported "SH"."SALES":"SALES_Q2_1998"                  1.2 MB   35758 rows in 0 seconds using external_table
    01-JUL-25 20:25:08.672: W-1 . . imported "SH"."SALES":"SALES_Q3_1998"                  1.6 MB   50515 rows in 0 seconds using external_table
    01-JUL-25 20:25:08.925: W-1 . . imported "SH"."SALES":"SALES_Q4_1998"                  1.6 MB   48874 rows in 0 seconds using external_table
    01-JUL-25 20:25:09.205: W-1 . . imported "SH"."SALES":"SALES_Q1_1999"                  2.1 MB   64186 rows in 1 seconds using external_table
    01-JUL-25 20:25:09.470: W-1 . . imported "SH"."SALES":"SALES_Q2_1999"                  1.8 MB   54233 rows in 0 seconds using external_table
    01-JUL-25 20:25:09.760: W-1 . . imported "SH"."SALES":"SALES_Q3_1999"                  2.2 MB   67138 rows in 0 seconds using external_table
    01-JUL-25 20:25:10.035: W-1 . . imported "SH"."SALES":"SALES_Q4_1999"                    2 MB   62388 rows in 1 seconds using external_table
    01-JUL-25 20:25:10.309: W-1 . . imported "SH"."SALES":"SALES_Q1_2000"                    2 MB   62197 rows in 0 seconds using external_table
    01-JUL-25 20:25:10.580: W-1 . . imported "SH"."SALES":"SALES_Q2_2000"                  1.8 MB   55515 rows in 0 seconds using external_table
    01-JUL-25 20:25:10.887: W-1 . . imported "SH"."SALES":"SALES_Q3_2000"                  1.9 MB   58950 rows in 0 seconds using external_table
    01-JUL-25 20:25:11.153: W-1 . . imported "SH"."SALES":"SALES_Q4_2000"                  1.8 MB   55984 rows in 1 seconds using external_table
    01-JUL-25 20:25:11.424: W-1 . . imported "SH"."SALES":"SALES_Q1_2001"                    2 MB   60608 rows in 0 seconds using external_table
    01-JUL-25 20:25:11.703: W-1 . . imported "SH"."SALES":"SALES_Q2_2001"                  2.1 MB   63292 rows in 0 seconds using external_table
    01-JUL-25 20:25:11.975: W-1 . . imported "SH"."SALES":"SALES_Q3_2001"                  2.1 MB   65769 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.281: W-1 . . imported "SH"."SALES":"SALES_Q4_2001"                  2.3 MB   69749 rows in 1 seconds using external_table
    01-JUL-25 20:25:12.289: W-1 . . imported "SH"."SALES":"SALES_Q1_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.292: W-1 . . imported "SH"."SALES":"SALES_Q2_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.295: W-1 . . imported "SH"."SALES":"SALES_Q3_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.298: W-1 . . imported "SH"."SALES":"SALES_Q4_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.300: W-1 . . imported "SH"."SALES":"SALES_Q1_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.303: W-1 . . imported "SH"."SALES":"SALES_Q2_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.306: W-1 . . imported "SH"."SALES":"SALES_Q3_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.308: W-1 . . imported "SH"."SALES":"SALES_Q4_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.310: W-1 . . imported "IX"."AQ$_ORDERS_QUEUETABLE_G"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.313: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_L"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.524: W-1 . . imported "HR"."COUNTRIES"                              6.5 KB      25 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.680: W-1 . . imported "HR"."LOCATIONS"                              8.7 KB      23 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.828: W-1 . . imported "HR"."JOB_HISTORY"                            7.4 KB      10 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.835: W-1 . . imported "IX"."AQ$_ORDERS_QUEUETABLE_H"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.838: W-1 . . imported "IX"."AQ$_ORDERS_QUEUETABLE_L"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:12.840: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_I"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.058: W-1 . . imported "SH"."SUPPLEMENTARY_DEMOGRAPHICS"           698.2 KB    4500 rows in 1 seconds using external_table
    01-JUL-25 20:25:13.066: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_T"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.068: W-1 . . imported "SH"."COSTS":"COSTS_1995"                       0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.071: W-1 . . imported "SH"."COSTS":"COSTS_1996"                       0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.073: W-1 . . imported "SH"."COSTS":"COSTS_H1_1997"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.075: W-1 . . imported "SH"."COSTS":"COSTS_H2_1997"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.280: W-1 . . imported "SH"."COSTS":"COSTS_Q1_1998"                139.9 KB    4411 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.457: W-1 . . imported "SH"."COSTS":"COSTS_Q2_1998"                 79.9 KB    2397 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.649: W-1 . . imported "SH"."COSTS":"COSTS_Q3_1998"                131.5 KB    4129 rows in 0 seconds using external_table
    01-JUL-25 20:25:13.834: W-1 . . imported "SH"."COSTS":"COSTS_Q4_1998"                145.1 KB    4577 rows in 0 seconds using external_table
    01-JUL-25 20:25:14.026: W-1 . . imported "SH"."COSTS":"COSTS_Q1_1999"                  184 KB    5884 rows in 1 seconds using external_table
    01-JUL-25 20:25:14.321: W-1 . . imported "SH"."COSTS":"COSTS_Q2_1999"                  133 KB    4179 rows in 0 seconds using external_table
    01-JUL-25 20:25:14.506: W-1 . . imported "SH"."COSTS":"COSTS_Q3_1999"                137.8 KB    4336 rows in 0 seconds using external_table
    01-JUL-25 20:25:14.695: W-1 . . imported "SH"."COSTS":"COSTS_Q4_1999"                159.5 KB    5060 rows in 0 seconds using external_table
    01-JUL-25 20:25:14.879: W-1 . . imported "SH"."COSTS":"COSTS_Q1_2000"                  121 KB    3772 rows in 0 seconds using external_table
    01-JUL-25 20:25:15.055: W-1 . . imported "SH"."COSTS":"COSTS_Q2_2000"                119.4 KB    3715 rows in 1 seconds using external_table
    01-JUL-25 20:25:15.244: W-1 . . imported "SH"."COSTS":"COSTS_Q3_2000"                151.9 KB    4798 rows in 0 seconds using external_table
    01-JUL-25 20:25:15.437: W-1 . . imported "SH"."COSTS":"COSTS_Q4_2000"                160.7 KB    5088 rows in 0 seconds using external_table
    01-JUL-25 20:25:15.634: W-1 . . imported "SH"."COSTS":"COSTS_Q1_2001"                228.3 KB    7328 rows in 0 seconds using external_table
    01-JUL-25 20:25:15.828: W-1 . . imported "SH"."COSTS":"COSTS_Q2_2001"                  185 KB    5882 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.022: W-1 . . imported "SH"."COSTS":"COSTS_Q3_2001"                234.9 KB    7545 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.335: W-1 . . imported "SH"."COSTS":"COSTS_Q4_2001"                278.8 KB    9011 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.342: W-1 . . imported "SH"."COSTS":"COSTS_Q1_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.345: W-1 . . imported "SH"."COSTS":"COSTS_Q2_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.347: W-1 . . imported "SH"."COSTS":"COSTS_Q3_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.350: W-1 . . imported "SH"."COSTS":"COSTS_Q4_2002"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.352: W-1 . . imported "SH"."COSTS":"COSTS_Q1_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.354: W-1 . . imported "SH"."COSTS":"COSTS_Q2_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.357: W-1 . . imported "SH"."COSTS":"COSTS_Q3_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.359: W-1 . . imported "SH"."COSTS":"COSTS_Q4_2003"                    0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.881: W-1 . . imported "SH"."CUSTOMERS"                             10.3 MB   55500 rows in 0 seconds using external_table
    01-JUL-25 20:25:16.888: W-1 . . imported "IX"."AQ$_ORDERS_QUEUETABLE_I"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:17.086: W-1 . . imported "SH"."FWEEK_PSCAT_SALES_MV"                 420.2 KB   11266 rows in 1 seconds using external_table
    01-JUL-25 20:25:17.250: W-1 . . imported "IX"."AQ$_ORDERS_QUEUETABLE_S"               11.9 KB       4 rows in 0 seconds using external_table
    01-JUL-25 20:25:17.551: W-1 . . imported "SH"."TIMES"                                383.3 KB    1826 rows in 0 seconds using external_table
    01-JUL-25 20:25:17.744: W-1 . . imported "SH"."PRODUCTS"                              27.6 KB      72 rows in 0 seconds using external_table
    01-JUL-25 20:25:17.891: W-1 . . imported "HR"."DEPARTMENTS"                            7.3 KB      27 rows in 0 seconds using external_table
    01-JUL-25 20:25:18.038: W-1 . . imported "HR"."JOBS"                                   7.3 KB      19 rows in 1 seconds using external_table
    01-JUL-25 20:25:18.046: W-1 . . imported "IX"."STREAMS_QUEUE_TABLE"                      0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:18.268: W-1 . . imported "SH"."PROMOTIONS"                            59.6 KB     503 rows in 0 seconds using external_table
    01-JUL-25 20:25:18.457: W-1 . . imported "HR"."EMPLOYEES"                             17.5 KB     107 rows in 0 seconds using external_table
    01-JUL-25 20:25:32.241: W-1 . . imported "PM"."PRINT_MEDIA"                          191.3 KB       4 rows in 13 seconds using external_table
    01-JUL-25 20:25:45.652: W-1 . . imported "PM"."TEXTDOCS_NESTEDTAB"                      88 KB      12 rows in 14 seconds using external_table
    01-JUL-25 20:25:45.718: W-1 . . imported "IX"."ORDERS_QUEUETABLE"                        0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:45.720: W-1 . . imported "IX"."AQ$_ORDERS_QUEUETABLE_T"                  0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:45.722: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_H"                0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:46.139: W-1 . . imported "SH"."COUNTRIES"                             10.9 KB      23 rows in 1 seconds using external_table
    01-JUL-25 20:25:46.365: W-1 . . imported "IX"."AQ$_STREAMS_QUEUE_TABLE_S"             12.3 KB       1 rows in 0 seconds using external_table
    01-JUL-25 20:25:46.577: W-1 . . imported "SH"."CHANNELS"                               7.7 KB       5 rows in 0 seconds using external_table
    01-JUL-25 20:25:46.785: W-1 . . imported "SH"."CAL_MONTH_SALES_MV"                     6.5 KB      48 rows in 0 seconds using external_table
    01-JUL-25 20:25:46.792: W-1 . . imported "SH"."SALES":"SALES_1995"                       0 KB       0 rows in 0 seconds using external_table
    01-JUL-25 20:25:46.868: W-1 Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
    01-JUL-25 20:25:47.014: W-1      Completed 10 OBJECT_GRANT objects in 0 seconds
    01-JUL-25 20:25:47.014: W-1 Processing object type SCHEMA_EXPORT/TABLE/COMMENT
    01-JUL-25 20:25:47.212: W-1      Completed 127 COMMENT objects in 0 seconds
    01-JUL-25 20:25:47.212: W-1 Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
    01-JUL-25 20:25:47.297: W-1      Completed 2 PROCEDURE objects in 0 seconds
    01-JUL-25 20:25:47.297: W-1 Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
    01-JUL-25 20:25:47.423: W-1      Completed 2 ALTER_PROCEDURE objects in 0 seconds
    01-JUL-25 20:25:47.423: W-1 Processing object type SCHEMA_EXPORT/VIEW/VIEW
    01-JUL-25 20:25:47.607: W-1      Completed 8 VIEW objects in 0 seconds
    01-JUL-25 20:25:47.665: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
    01-JUL-25 20:25:48.042: W-1      Completed 22 INDEX objects in 1 seconds
    01-JUL-25 20:25:48.042: W-1 Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
    01-JUL-25 20:25:48.450: W-1      Completed 21 CONSTRAINT objects in 0 seconds
    01-JUL-25 20:25:48.450: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
    01-JUL-25 20:25:48.475: W-1      Completed 54 INDEX_STATISTICS objects in 0 seconds
    01-JUL-25 20:25:48.475: W-1 Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
    01-JUL-25 20:25:48.696: W-1      Completed 20 REF_CONSTRAINT objects in 0 seconds
    01-JUL-25 20:25:48.696: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/BITMAP_INDEX/INDEX
    01-JUL-25 20:25:50.531: W-1      Completed 15 INDEX objects in 2 seconds
    01-JUL-25 20:25:50.531: W-1 Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/BITMAP_INDEX/INDEX_STATISTICS
    01-JUL-25 20:25:50.551: W-1      Completed 15 INDEX_STATISTICS objects in 0 seconds
    01-JUL-25 20:25:50.551: W-1 Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
    01-JUL-25 20:25:50.659: W-1      Completed 2 TRIGGER objects in 0 seconds
    01-JUL-25 20:25:50.659: W-1 Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
    01-JUL-25 20:25:50.681: W-1      Completed 38 TABLE_STATISTICS objects in 0 seconds
    01-JUL-25 20:25:50.705: W-1      Completed 1 [internal] STATISTICS objects in 0 seconds
    01-JUL-25 20:25:50.705: W-1 Processing object type SCHEMA_EXPORT/MATERIALIZED_VIEW
    01-JUL-25 20:25:51.229: W-1      Completed 2 MATERIALIZED_VIEW objects in 1 seconds
    01-JUL-25 20:25:51.229: W-1 Processing object type SCHEMA_EXPORT/DIMENSION
    01-JUL-25 20:25:51.316: W-1      Completed 5 DIMENSION objects in 0 seconds
    01-JUL-25 20:25:51.352: W-1 Processing object type SCHEMA_EXPORT/TABLE/POST_INSTANCE/PROCACT_INSTANCE/AQ
    01-JUL-25 20:25:52.862: W-1      Completed 15 AQ objects in 1 seconds
    01-JUL-25 20:25:53.134: W-1 Processing object type SCHEMA_EXPORT/TABLE/POST_INSTANCE/PROCDEPOBJ/RULE
    01-JUL-25 20:25:53.195: W-1      Completed 6 RULE objects in 0 seconds
    01-JUL-25 20:25:53.198: W-1 Processing object type SCHEMA_EXPORT/TABLE/POST_INSTANCE/PROCDEPOBJ/AQ
    01-JUL-25 20:25:53.437: W-1      Completed 4 AQ objects in 0 seconds
    01-JUL-25 20:25:53.511: W-1 Processing object type SCHEMA_EXPORT/POST_SCHEMA/PROCOBJ/RULE
    01-JUL-25 20:25:53.527: W-1      Completed 6 RULE objects in 0 seconds
    01-JUL-25 20:25:53.566: W-1 Processing object type SCHEMA_EXPORT/POST_SCHEMA/PROCACT_SCHEMA/AQ
    01-JUL-25 20:25:53.620: W-1      Completed 1 AQ objects in 0 seconds
    01-JUL-25 20:25:53.691: W-1      Completed 89 SCHEMA_EXPORT/TABLE/TABLE_DATA objects in 40 seconds
    01-JUL-25 20:25:53.831: Job "ADMIN"."SYS_IMPORT_SCHEMA_01" completed with 1 error(s) at Tue Jul 1 20:25:53 2025 elapsed 0 00:01:01
    ```
    </details>

You may now *proceed to the next lab*.

## Additional information

* Webinar, [Data Pump Best Practices and Real World Scenarios, Metadata](https://www.youtube.com/watch?v=CUHcKHx_YvA&t=1260s)
* Webinar, [Data Pump Best Practices and Real World Scenarios, Generate metadata with SQLFILE](https://www.youtube.com/watch?v=CUHcKHx_YvA&t=4642s)

## Acknowledgments

* **Author** - Rodrigo Jorge
* **Contributors** - William Beauregard, Daniel Overby Hansen, Mike Dietrich, Klaus Gronau, Alex Zaballa
* **Last Updated By/Date** - Rodrigo Jorge, May 2025