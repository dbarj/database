# Determining Import Success

## Introduction

When you move complex data around or when you're doing full exports/imports, it's common to find errors in the Data Pump log file. In this lab, you will learn about errors and how to determine whether the *SAPHIRE* and *RUBY* imports were successful.

Estimated Time: 10 Minutes

### Objectives

In this lab, you will:

* Examine errors and log files
* Compare schemas

### Prerequisites

This lab assumes:

- You have completed Lab 6: Migration using Data Pump with NFS
- You have completed Lab 7: Migration using Data Pump with DB Link

## Task 1: Errors

During export and import, Data Pump may face errors or situations that can't be resolved.

1. Use the *yellow* terminal 🟨. If Data Pump encounters an error or faces a situation it can't resolve, it will print an error to the console and into the logfile. At the end of the output, Data Pump summarizes and lists the number of errors faced during the job. Examine the last lines of a Data Pump import log file.

    ```
    <copy>
    tail -1 /nfs_mount/schemas_import_nfs.log
    tail -1 /nfs_mount/schemas_import_dblink.log
    </copy>
    ```

    * The last line for *schemas_import_nfs.log* says *completed with 1 error*.
    * You need to examine the entire log file if you want to find the details.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [ADB:oracle@holserv1:~]$ tail -1 /nfs_mount/schemas_import_nfs.log
    02-JUL-25 13:43:11.295: Job "ADMIN"."SYS_IMPORT_SCHEMA_01" completed with 1 error(s) at Wed Jul 2 13:43:11 2025 elapsed 0 00:00:46
    [ADB:oracle@holserv1:~]$ tail -1 /nfs_mount/schemas_import_dblink.log
    02-JUL-25 17:25:27.400: Job "ADMIN"."SYS_IMPORT_SCHEMA_01" successfully completed at Wed Jul 2 17:25:27 2025 elapsed 0 00:01:36
    ```
    </details>

2. In the Data Pump NFS import, we experience an error. What is it about it?

    ```
    <copy>
    grep -A 2 -B 1 'ORA-' /nfs_mount/schemas_import_nfs.log
    </copy>
    ```

    * We are getting all lines containing 'ORA-', plus the 2 lines before and 1 line after it.
    * Notice there was a *insufficient privileges* error when trying to create a DB Link.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    02-JUL-25 13:42:30.736: W-1 Processing object type SCHEMA_EXPORT/DB_LINK
    02-JUL-25 13:42:30.809: ORA-31685: Object type DB_LINK:"SH"."F1" failed due to insufficient privileges. Failing sql is:
    CREATE DATABASE LINK "F1"  CONNECT TO "F1" IDENTIFIED BY VALUES ':1'  USING '//localhost:1521/red'
    02-JUL-25 13:42:30.819: W-1      Completed 1 DB_LINK objects in 0 seconds
    ```
    </details>

3.  These are just examples of the errors you might encounter. In each situation, you must investigate the situation and determine whether or not it influences the integrity of the data you're moving.

## Task 2: Fix the Database Link error on *SAPPHIRE* ADB

In ADB Serverless, the syntax to create a database link is different. We must use DBMS_CLOUD_ADMIN. Let's fix it.

First, this DB Link was connecting from (*BLUE* -> *RED*).

As both databases were migrated, now this needs to be changed to connect from (*SAPPHIRE* -> *RUBY*).

We need to upload the *RED* wallet to ADB directory.

1. Let's first connect on ADB:

    ``` shell
    <copy>
    . adb
    sql admin/Welcome_1234@sapphire_tp
    </copy>

    # Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    [CDB23:oracle@holserv1:~]$ . adb
    [ADB:oracle@holserv1:~]$ sql admin/Welcome_1234@sapphire_tp

    SQLcl: Release 25.1 Production on Wed Jul 02 15:17:06 2025

    Copyright (c) 1982, 2025, Oracle.  All rights reserved.

    Last Successful login time: Wed Jul 02 2025 15:17:06 +00:00

    Connected to:
    Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
    Version 23.8.0.25.05

    SQL>
    ```
    </details>

2. Create a directory to keep the wallet files.

    ``` shell
    <copy>
    create directory ruby_dblink_wallet_dir as 'ruby_dblink_wallet_dir';
    grant execute on dbms_cloud to SH;
    grant execute on dbms_cloud_admin to SH;
    grant read on directory ruby_dblink_wallet_dir to SH;
    </copy>

    -- Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> create directory ruby_dblink_wallet_dir as 'ruby_dblink_wallet_dir';

    Directory RUBY_DBLINK_WALLET_DIR created.

    SQL> grant execute on dbms_cloud to SH;

    Grant succeeded.

    SQL> grant execute on dbms_cloud_admin to SH;

    Grant succeeded.

    SQL> grant read on directory ruby_dblink_wallet_dir to SH;

    Grant succeeded.
    ```
    </details>

3. Next, let's upload the local wallet files to this directory.

    ``` shell
    <copy>
    @~/scripts/adb-07-upload_file.sql /home/oracle/adb_tls_wallet/cwallet.sso RUBY_DBLINK_WALLET_DIR cwallet.sso
    </copy>

    -- Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> @~/scripts/adb-07-upload_file.sql /home/oracle/adb_tls_wallet/cwallet.sso RUBY_DBLINK_WALLET_DIR cwallet.sso
    Starting upload_file.js script...
    Local file path: /home/oracle/adb_tls_wallet/cwallet.sso
    Oracle directory: RUBY_DBLINK_WALLET_DIR
    Target file name: cwallet.sso
    Creating BLOB and opening binary stream...
    Reading local file into stream...
    File read and copied into BLOB stream.
    Saving BLOB to Oracle Directory...
    File successfully written to Oracle directory.
    File uploaded successfully.
    ```
    </details>

4. Check if file was uploaded.

    ``` shell
    <copy>
    select * from dbms_cloud.list_files('ruby_dblink_wallet_dir');
    </copy>

    -- Be sure to hit RETURN
    ```

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> select * from dbms_cloud.list_files('ruby_dblink_wallet_dir');

    OBJECT_NAME       BYTES CHECKSUM    CREATED                                LAST_MODIFIED
    ______________ ________ ___________ ______________________________________ ______________________________________
    cwallet.sso        3899             02-JUL-25 03.28.11.618341000 PM GMT    02-JUL-25 03.28.11.713437000 PM GMT
    ```
    </details>

5. Now connect as SH user. Create the DB link credentials.

    ``` shell
    <copy>
    conn sh/oracle@sapphire_tp
    begin
      dbms_cloud.create_credential(
        credential_name => 'F1_RUBY_CRED',
        username => 'F1',
        password => 'oracle');
    end;
    /
    </copy>

    -- Be sure to hit RETURN
    ```

    * Please note that *username* and *password* are case sensitive.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> conn sh/oracle@sapphire_tp

    Connected.
    SQL> begin
      2    dbms_cloud.create_credential(
      3      credential_name => 'F1_RUBY_CRED',
      4      username => 'F1',
      5      password => 'oracle');
      6  end;
      7* /

    PL/SQL procedure successfully completed.
    ```
    </details>

6. Create the DB link and test it.

    ``` shell
    <copy>
    begin
      dbms_cloud_admin.create_database_link(
        db_link_name => 'F1',
        hostname => 'adb-free',
        port => '1522',
        service_name => 'ruby_tp.adb.oraclecloud.com',
        ssl_server_cert_dn => 'CN=93ced68f921a',
        credential_name => 'F1_RUBY_CRED',
        directory_name => 'RUBY_DBLINK_WALLET_DIR');
    end;
    /
    select count(*) from F1_RESULTS@F1;
    </copy>

    -- Be sure to hit RETURN
    ```

    * To get CN (Common Name), you can run: *openssl x509 -in ~/adb_tls_wallet/adb_container.cert -noout -subject*

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> begin
      2    dbms_cloud_admin.create_database_link(
      3      db_link_name => 'F1',
      4      hostname => 'adb-free',
      5      port => '1522',
      6      service_name => 'ruby_tp.adb.oraclecloud.com',
      7      ssl_server_cert_dn => 'CN=93ced68f921a',
      8      credential_name => 'F1_RUBY_CRED',
      9      directory_name => 'RUBY_DBLINK_WALLET_DIR');
     10  end;
     11* /

    PL/SQL procedure successfully completed.

    SQL> select count(*) from F1_RESULTS@F1;

       COUNT(*)
    ___________
          26439
    ```
    </details>

## Task 2: Comparing source and target data on *RUBY*

After moving data you can perform simple checks to validate the outcome. You will try such on the *F1* schema in the *RUBY* PDB that you imported in lab 7.

1. Still in the *yellow* terminal 🟨. Connect to the *RUBY* PDB. This is our target database.

    ```
    <copy>
    . adb
    sql admin/Welcome_1234@ruby_tp
    </copy>

    -- Be sure to hit RETURN
    ```

    * The *RUBY* PDB contains an *F1* schema moved from the *RED* database.

2. Count the number of objects grouped by types in the target database and compare it to the source database.

    ```
    <copy>
    select object_type, count(*) from dba_objects where owner='F1' group by object_type
    minus
    select object_type, count(*) from dba_objects@source_dblink where owner='F1' group by object_type;
    </copy>

    -- Be sure to hit RETURN
    ```

    * *No rows selected* means the count of different object types matches.
    * It does not mean that there are no differences between the source and target.
    * This is just a simple count.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> select object_type, count(*) from dba_objects where owner='F1' group by object_type
         minus
         select object_type, count(*) from dba_objects@source_dblink where owner='F1' group by object_type;

    no rows selected
    ```
    </details>

3. If you want to compare the amount of rows, you can do that too

    ```
    <copy>
    select
       (select count(*) from f1.f1_laptimes@source_dblink) as source,
       (select /*+parallel*/ count(*) from f1.f1_laptimes) as target
    from dual;
    </copy>

    -- Be sure to hit RETURN
    ```

    * You can use parallel query to speed up the process, however, selects over a database link can't use parallel query.
    * For big tables it might be faster to run the query on each database, spool to a file and compare the two files.
    * Counting rows works most efficiently if the table has an index on a column with a NOT NULL constraint, like a primary key index.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SOURCE     TARGET
    ---------- ----------
    571047     571047
    ```
    </details>

4. The examples used in this task are not a complete guide. It should give you an idea of how you can use the data dictionary information and queries to compare your source and target environments. Comparing objects becomes more complicated when you have system-generated names for indexes and partitions and when you use Advanced Queueing. The latter because it creates a varying number of objects recursively depending on how you use the queues.

## Task 3: DBMS_COMPARISON

The `DBMS_COMPARISON` package allows you to compare the rows of the same table in two different databases.

1. Still in the *yellow* terminal 🟨 and connected to the *RUBY* PDB from the previous task. Create a new comparison.

    ```
    <copy>
    SET SERVEROUT ON
    
    DECLARE
      V_COMPARISON_NAME VARCHAR2(128);
      V_RESULT          BOOLEAN;
      V_COMPARISON_OUT  DBMS_COMPARISON.COMPARISON_TYPE;
    BEGIN
      FOR T IN (
        SELECT OWNER, TABLE_NAME
          FROM ALL_TABLES
         WHERE OWNER IN ('F1','HR','PM','IX','SH','BI')
      ) LOOP
        BEGIN
          V_COMPARISON_NAME := 'CMP_' || T.TABLE_NAME;
    
          -- Drop if exists
          BEGIN
            DBMS_COMPARISON.DROP_COMPARISON(V_COMPARISON_NAME);
          EXCEPTION
            WHEN OTHERS THEN
              NULL;
          END;
    
          -- Create comparison
          DBMS_COMPARISON.CREATE_COMPARISON(
            COMPARISON_NAME => V_COMPARISON_NAME,
            SCHEMA_NAME => T.OWNER,
            OBJECT_NAME => T.TABLE_NAME,
            DBLINK_NAME => 'SOURCE_DBLINK',
            REMOTE_SCHEMA_NAME => T.OWNER,
            REMOTE_OBJECT_NAME => T.TABLE_NAME,
            SCAN_MODE => DBMS_COMPARISON.CMP_SCAN_MODE_FULL
          );
    
          -- Run comparison
          V_RESULT := DBMS_COMPARISON.COMPARE(COMPARISON_NAME => V_COMPARISON_NAME, SCAN_INFO => V_COMPARISON_OUT);
          DBMS_OUTPUT.PUT_LINE(T.OWNER || '.' || T.TABLE_NAME || ': ' || CASE
            WHEN V_RESULT THEN
              'MATCH'
            ELSE 'DIFFER'
          END);
    
        EXCEPTION
          WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('Error comparing ' || T.TABLE_NAME || ': ' || SQLERRM);
        END;
      END LOOP;
    END;
    /
    </copy>

    -- Be sure to hit RETURN
    ```

    * The `SCAN_MODE` is set to full because it is a small table.
    * If you have bigger tables, you can select just a sample.
    * You can also call *@/home/oracle/scripts/adb-08-dbms_compare.sql*

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> begin
            dbms_comparison.create_comparison (
                comparison_name => 'AFTER_MIGRATION',
                schema_name => 'F1',
                object_name => 'F1_CONSTRUCTORS',
                dblink_name => 'SOURCE_DBLINK',
                scan_mode => DBMS_COMPARISON.CMP_SCAN_MODE_FULL);
        end;
        /  2    3    4    5    6    7    8    9

    PL/SQL procedure successfully completed.

    SQL>
    ```
    </details>

2. Execute the comparison, list the number of differences and the query to find the rows.

    ```
    <copy>
    set line 150
    set serveroutput on

    col constructorref format a14
    col name format a30
    col nationality format a20
    col url format a50

    declare
        l_scan_info     dbms_comparison.comparison_type;
        l_outcome       boolean;
        l_count         number;
        l_diff          number;
        l_localrowid    user_comparison_row_dif.local_rowid%type;
        l_remoterowid   user_comparison_row_dif.remote_rowid%type;
        l_scanid        user_comparison_row_dif.scan_id%type;
    begin
        l_outcome := dbms_comparison.compare (
            comparison_name => 'AFTER_MIGRATION',
            scan_info       => l_scan_info,
            perform_row_dif => true);

        if l_outcome then
            dbms_output.put_line('No differences found!');
            return;
        end if;

        dbms_output.put_line('Differences found!');

        select current_dif_count, count_rows into l_diff, l_count from user_comparison_scan_summary where scan_id = l_scan_info.scan_id;

        dbms_output.put_line('Total rows: ' || l_count);
        dbms_output.put_line('Diff rows:  ' || l_diff);

        select   local_rowid, remote_rowid, max(scan_id)
        into     l_localrowid, l_remoterowid, l_scanid
        from     user_comparison_row_dif
        where    comparison_name='AFTER_MIGRATION'
        group by local_rowid, remote_rowid;

        dbms_output.put_line('Local row:  select * from f1.f1_constructors where rowid='''|| l_localrowid || ''';');
        dbms_output.put_line('Remote row: select * from f1.f1_constructors@source_dblink where rowid='''|| l_remoterowid || ''';');

    end;
    /
    </copy>

    -- Be sure to hit RETURN
    ```

    * For simplicity, the bits and pieces have been glued together in a piece of PL/SQL.
    * The code conducts a comparison and reports the differences.
    * It also lists a query to find the offending rows in the remote/source and local/target databases.
    * In this case, the comparison correctly finds the one row you changed previously.

    <details>
    <summary>*click to see the output*</summary>
    ``` text
    Differences found!
    Total rows: 212
    Diff rows:  1
    Local row:  select * from f1.f1_constructors where rowid='AAAG3lAAAAAAAPuAAW';
    Remote row: select * from f1.f1_constructors@source_dblink where rowid='AAAF6aAAEAAAAFkAAX';

    PL/SQL procedure successfully completed.
    ```
    </details>

3. Execute the two queries. Don't copy and paste from the instructions. Use the queries in your output.

    ```
    select * from f1.f1_constructors where rowid='AAAG3lAAAAAAAPuAAW';
    select * from f1.f1_constructors@source_dblink where rowid='AAAF6aAAEAAAAFkAAX';
    ```

    * Spot the difference.
    * The name from the local - or target - database has *##42##* appended.
    * If you get `ORA-01410: invalid ROWID` or `ORA-01410: The ROWID is invalid.` you are using the wrong queries. Don't copy/paste from the instructions. Use the queries generated by your output.


    <details>
    <summary>*click to see the output*</summary>
    ``` text
    SQL> select * from f1.f1_constructors where rowid='AAAG3lAAAAAAAPuAAW';

    CONSTRUCTORID CONSTRUCTORREF NAME                           NATIONALITY          URL
    ------------- -------------- ------------------------------ -------------------- ------------------------------------------------------------
    210           haas           Haas F1 Team##42##             American             http://en.wikipedia.org/wiki/Haas_F1_Team

    SQL> select * from f1.f1_constructors@source_dblink where rowid='AAAF6aAAEAAAAFkAAX';

    CONSTRUCTORID CONSTRUCTORREF NAME                           NATIONALITY          URL
    ------------- -------------- ------------------------------ -------------------- ------------------------------------------------------------
    210           haas           Haas F1 Team                   American             http://en.wikipedia.org/wiki/Haas_F1_Team
    ```
    </details>


5. Exit SQL*Plus.

    ```
    <copy>
    exit
    </copy>
    ```

You may now *proceed to the next lab*.

## Additional information

* Webinar, [Data Pump Best Practices and Real World Scenarios, Verification and Checks when you use Data Pump](https://www.youtube.com/watch?v=960ToLE-ZE8&t=4857s)
* [Data Pump Log Analyzer](https://github.com/macsdata/data-pump-log-analyzer)

## Acknowledgments

* **Author** - Rodrigo Jorge
* **Contributors** - William Beauregard, Daniel Overby Hansen, Mike Dietrich, Klaus Gronau, Alex Zaballa
* **Last Updated By/Date** - Rodrigo Jorge, May 2025