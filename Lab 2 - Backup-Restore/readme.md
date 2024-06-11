# ORACLE DATABASE BACKUP & RESTORE USING RMAN
## 1. Verify the Database have already installed
### Check Database status
```
SQL> col db_unique_name format a15
col open_mode format a12
select dbid, name, db_unique_name, database_role, open_mode, log_mode from v$database;

SQL> set linesize 200
SELECT instance_name, host_name, version, status, startup_time, archiver, database_status from v$instance;

SQL> set pagesize 999;
set linesize 999;
select * from hr.employees;

SQL> set pagesize 999;
set linesize 999;
select * from sys.customers;

SQL> set pagesize 999;
set linesize 999;
SELECT * FROM all_users
ORDER BY created;
select * from books_admin.persons;
```

## BACKUP FULL
### Create file script backup.sh
```
#!/bin/bash

export ORACLE_SID=metrocomdb
BASE_PATH=/usr/sbin:$PATH; export BASE_PATH
PATH=$ORACLE_HOME/bin:$PATH:$ORACLE_HOME/OPatch:$BASE_PATH; export PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH

tag=`date '+%Y%m%d_%H%M'`

rman target / msglog /backup/metrocomdb/rman_logs/metrocomdb_full_backup_$tag.log append <<EOF
SQL 'ALTER SYSTEM ARCHIVE LOG CURRENT';
SQL "ALTER SESSION SET NLS_DATE_FORMAT=''dd.mm.yyyy hh24:mi:ss''";
RUN
{
CROSSCHECK BACKUP;
REPORT OBSOLETE;
DELETE NOPROMPT EXPIRED BACKUP;
DELETE NOPROMPT OBSOLETE;
ALLOCATE CHANNEL ch_bck1 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck2 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck3 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck4 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck5 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck6 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck7 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
ALLOCATE CHANNEL ch_bck8 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_FULL';
SET COMMAND ID TO 'DBOnlineBackupFull';
BACKUP DATABASE TAG = 'metrocomdb_FULL';
RELEASE CHANNEL ch_bck1;
RELEASE CHANNEL ch_bck2;
RELEASE CHANNEL ch_bck3;
RELEASE CHANNEL ch_bck4;
RELEASE CHANNEL ch_bck5;
RELEASE CHANNEL ch_bck6;
RELEASE CHANNEL ch_bck7;
RELEASE CHANNEL ch_bck8;
BACKUP SPFILE FORMAT '/backup/metrocomdb/%d_%T_%s_%p_SPFILE' TAG = 'metrocomdb_SPFILE';
BACKUP CURRENT CONTROLFILE FORMAT '/backup/metrocomdb/%d_%T_%s_%p_CTLFILE' TAG = 'metrocomdb_CTLFILE';
}
EOF

exit
```

## ARCHIVE BACKUP
### Create file script backup_arch.sh
```
#!/bin/bash

export ORACLE_SID=metrocomdb
BASE_PATH=/usr/sbin:$PATH; export BASE_PATH
PATH=$ORACLE_HOME/bin:$PATH:$ORACLE_HOME/OPatch:$BASE_PATH; export PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH

tag=`date '+%Y%m%d_%H%M'`

rman target / msglog /backup/metrocomdb/rman_logs/metrocomdb_archivelog_backup_$tag.log append <<EOF
SQL 'ALTER SYSTEM ARCHIVE LOG CURRENT';
SQL "ALTER SESSION SET NLS_DATE_FORMAT=''dd.mm.yyyy hh24:mi:ss''";
RUN
{
ALLOCATE CHANNEL ch_arc1 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc2 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc3 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc4 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc5 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc6 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc7 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
ALLOCATE CHANNEL ch_arc8 DEVICE TYPE DISK MAXPIECESIZE 1G
FORMAT '/backup/metrocomdb/%d_%T_%s_%p_ARCHIVE';
SET COMMAND ID TO 'Archive_BKP_4_Hr';
CROSSCHECK BACKUP;
CROSSCHECK ARCHIVELOG ALL;
DELETE NOPROMPT EXPIRED BACKUP;
delete noprompt expired archivelog all;
DELETE NOPROMPT OBSOLETE DEVICE TYPE DISK;
BACKUP ARCHIVELOG ALL DELETE INPUT TAG = 'metrocomdb_ARCHIVE';
RELEASE CHANNEL ch_arc1;
RELEASE CHANNEL ch_arc2;
RELEASE CHANNEL ch_arc3;
RELEASE CHANNEL ch_arc4;
RELEASE CHANNEL ch_arc5;
RELEASE CHANNEL ch_arc6;
RELEASE CHANNEL ch_arc7;
RELEASE CHANNEL ch_arc8;
}
EOF
exit
```

## RESTORE
```
---- restore database di standby
rman target /
RMAN> startup nomount force;
RMAN> restore spfile from '/backup/METROCOM_20240611_30_1_SPFILE';
RMAN> restore spfile to pfile '/backup/metrocomdb-pfile.ora' from '/backup/METROCOM_20240611_30_1_SPFILE';

---- cek pfile audit and dir control file and archive dest
mkdir -p /u01/app/oracle/admin/metrocomdb/adump
mkdir -p /u01/app/oracle/oradata/METROCOMDB/
mkdir -p /backup/archivelog/metrocomdb

sqlplus / as sysdba
SQL> shutdown immediate;
SQL> startup nomount;
SQL> show parameter control_files
SQL> show parameter dump
SQL> show parameter create
SQL> show parameter recovery

rman target /
RMAN> RESTORE CONTROLFILE FROM '/backup/METROCOM_20240611_31_1_CTLFILE';
RMAN> alter database mount;
RMAN> report schema;
rman> catalog start with '/backup/data/' noprompt; atau 
rman> catalog start with '/backup/data/';
rman> catalog start with '/backup/arch/';
rman> restore database preview summary;
rman> restore database;
RMAN> report schema;

---- JIKA OFFLINE 
RMAN> recover database noredo;

---- JIKA ONLINE (ARCHIVE LOG AKTIF)
RMAN> catalog backuppiece '/backup/arch/METROCOM_20240611_33_1_ARCHIVE';
RMAN> catalog backuppiece '/backup/arch/METROCOM_20240611_34_1_ARCHIVE';
Run {
     Set archivelog destination to '/backup/arch/';
    SET UNTIL TIME "to_date('12 JUN 2024 12:30:00','DD MON YYYY hh24:mi:ss')";
	recover database;
     }

SQL> set pagesize 999;
set linesize 999;
select * from v$logfile;

SQL> alter database open resetlogs;

SQL> ARCHIVE LOG LIST
```

## Additional
```
# copy listener, tnsnames dan sqlnet 
shutdown
startup
---- reset pass sys
cd $ORACLE_HOME/dbs
orapwd file=orapwmetrocomdb entries=10
Enter password for SYS: (MASUKKAN PASSWORD)
Enter password for SYS: randomize123-

ALTER USER SYS IDENTIFIED BY "randomize123-";
```