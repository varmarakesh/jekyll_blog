---
layout: post
title:  "Oracle Database Install"
date:   2015-09-01 15:34:11
categories: oracle
comments: True
published: False
---
Three important directories referred by environment variables.

ORACLE_BASE - Top most directory for Oracle software installations.
ORACLE_HOME - Installation location of software for a particular software product
TNS_ADMIN - Network configuration files directory.


Implementing a database

Step-1 - Set the environment variables.

ORACLE_HOME
ORACLE_SID - Defines the default name of the database you are attempting to create.
LD_LIBRARY_PATH - Search for the libraries, usually $ORACLE_HOME/lib

Step-2 - Configure the initialization file.

init.ora - Used to configure all options such as control files, memory parameters for the database.

It is located in $ORACLE_HOME/dbs directory. The file name should reflect this format init<ORACLE_SID>.ora

The contents of a typical init file are:

db_name=DTC
db_block_size=8192
memory_target=800M


Step-3 - Create the required directories.

{% highlight bash %}

mkdir -p /orahome/admin/DTC/adump
mkdir -p /orahome/admin/DTC/arch
mkdir -p /u01/oradata/dtc

{% endhighlight %}

Step-4 - Create the database

export ORACLE_SID=DTC
{% highlight sql %}

connect / as sysdba
sho user
startup nomount pfile = /orahome/oracle/11204/dbs/initDTC.ora;
create database DTC
LOGFILE
                GROUP 11 ('/u01/oradata/dtc/dtc_redo11_01.log') SIZE 100m reuse,
                GROUP 12 ('/u01/oradata/dtc/dtc_redo12_01.log') SIZE 100m reuse,
                GROUP 13 ('/u01/oradata/dtc/dtc_redo13_01.log') SIZE 100m reuse,
                GROUP 14 ('/u01/oradata/dtc/dtc_redo14_01.log') SIZE 100m reuse,
                GROUP 15 ('/u01/oradata/dtc/dtc_redo15_01.log') SIZE 100m reuse,
                GROUP 16 ('/u01/oradata/dtc/dtc_redo16_01.log') SIZE 100m reuse
maxdatafiles 4096
maxinstances 32
maxlogfiles 256
maxlogmembers 4
maxloghistory 4096
character set WE8ISO8859P15
national character set AL16UTF16
DATAFILE '/u01/oradata/dtc/dtc_system.dbf' SIZE 2097216K reuse extent management local
SYSAUX DATAFILE '/u01/oradata/dtc/dtc_sysaux.dbf' SIZE 8388672K reuse
DEFAULT TEMPORARY TABLESPACE temp TEMPFILE '/u01/oradata/dtc/dtc_temp.dbf' SIZE 8388672K reuse autoextend off
extent management local uniform size 1M
undo tablespace RBS1 datafile '/u01/oradata/dtc/rbs11.dbf' size 8388672K reuse autoextend off
user SYS identified by "manager" user SYSTEM identified by "manager"
noarchivelog;

create undo tablespace RBS2 datafile '/u01/oradata/dtc/rbs21.dbf' size 8388672K reuse autoextend off;
create undo tablespace RBS3 datafile '/u01/oradata/dtc/rbs31.dbf' size 8388672K reuse autoextend off;

create tablespace XDB
datafile '/u01/oradata/dtc/xdb.dbf' size 512M reuse autoextend off extent management local segment space management auto;

create tablespace DBDATA
datafile '/u01/oradata/dtc/dbdata01.dbf' size 16777344K reuse autoextend off extent management local segment space management
manual;

alter tablespace DBDATA add datafile '/u01/oradata/dtc/dbdata02.dbf' size 16777344K reuse autoextend off;

create tablespace DBINDX
datafile '/u01/oradata/dtc/dbindx01.dbf' size 16777344K reuse autoextend off extent management local segment space management
manual;

alter tablespace DBINDX add datafile '/u01/oradata/dtc/dbindx02.dbf' size 16777344K reuse autoextend off;


alter user sys temporary tablespace TEMP;
alter user system temporary tablespace TEMP;
alter system checkpoint;
alter system checkpoint global;
disconnect;
exit;

{%endhighlight%}


Step -5 Create a data dictionary

After the database is succcessfully created, the data dictionary is created by running 2 scripts.

{% highlight sql %}
cd $ORACLE_HOME/rdbms/admin
@catalog.sql
@catproc,sql

{%endhighlight%}




