# Oracle Database Recovery and Management Guide

This guide provides step-by-step instructions for Oracle Database operations including connecting as SYSDBA, creating directories, importing dump files, managing users, and recovering databases.

## Connecting to Oracle Database

Connect to Oracle Database as SYSDBA:

```sql
sqlplus / as sysdba
```

This command connects using OS authentication with administrative privileges.

## Creating Directory Object

Before importing a dump file, create a directory object that points to the location of your dump file:

```sql
CREATE DIRECTORY dump AS '/home/oracle/dbdump';
```

This creates a logical directory object named `dump` that references the physical path `/home/oracle/dbdump`.

### Verify Directory Creation

```sql
SELECT directory_name, directory_path FROM dba_directories WHERE directory_name = 'DUMP';
```

### Grant Permissions (if needed)

```sql
GRANT READ, WRITE ON DIRECTORY dump TO demo;
```

## Importing Database Dump

Use Data Pump Import (impdp) to restore the database from a dump file:

```bash
impdp "\"/ as sysdba\"" directory=dump dumpfile=demo.dmp schemas=demo remap_schema=test:demo remap_tablespace=tbs_test:tbs_demo parallel=4 exclude=PACKAGE,RLS_POLICY,TRIGGER:\"IN\(\'TRI_GROUP_PRIV\',\'TRD_GROUP_PRIV\',\'TRU_GROUP_PRIV\'\)\",statistics,table:\"in\(\'ORCA_DOCUMENT_BATCH\',\'ORCA_DOCUMENT_BATCH_ITEM\',\'ORCA_EXCEPTION_LOG\',\'ORCA_DOCUMENT_BATCH_GEN_ITEM\',\'ORCA_CACHE_CLUSTER_EVENT_LOG\'\)\"
```

## User Management

### Change User Password

After import, set a new password for the database user:

```sql
ALTER USER demo IDENTIFIED BY "PASSWORD";
```

Replace `PASSWORD` with your desired password. Use quotes for special characters or mixed-case passwords.

### Password Best Practices

```sql
-- Strong password example
ALTER USER demo IDENTIFIED BY "MyStr0ng#Pass2024";

-- Unlock user if locked
ALTER USER demo ACCOUNT UNLOCK;

-- Check user status
SELECT username, account_status, lock_date FROM dba_users WHERE username = 'DEMO';
```

### Drop User and Schema

To completely remove a user and all associated objects:

```sql
DROP USER demo CASCADE;
```

The `CASCADE` option removes the user along with all objects owned by the user (tables, views, procedures, etc.).
