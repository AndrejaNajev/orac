# Oracle duplicate from active database using RMAN on Windows
A step by step guide.

Some important files from the app (e.g. tnsnames.ora, listener.ora) that were used for the duplication are uploaded in the code section.


**Source database:**
* _Database name_: ANAJ
* _Version_: 11.2.0
* _ORACLE_HOME_: C:\app\User\product\11.2.0\dbhome_1\bin;
* _ORACLE_BASE_: C:\app\User;


**Target database:**
* _Database name_: dup
* _Version_: 11.2.0



## STEPS TO DUPLICATE DATABASE
### 1. Create new directories for the target database.

`mkdir -p \C:\app\User\oradata\dup `

`mkdir -p \C:\app\User\fast_recovery_area\dup `

`mkdir -p \C:\app\User\admin\dup\pfile `


### 2.Create a new service _**OracleServicedup**_
`C:\> set ORACLE_SID = ANAJ`

`C:\> app\User\product\11.2.0\dbhome_1\dbs  oradim –new –sid dup`

### 3. Create an _**Oracle Password File**_  for the source and the target database. 
For Active database duplication, it connects directly to the auxiliary instance using the password file with the same SYSDBA password as the target database. In case you are using password file make sure to have same SYSDBA password as the target database. 
Create Oracle Password file (_orapw_) for the duplicate database in _dup session_.

`C:\> app\User\product\11.2.0\dbhome_1\dbs orapwd password=ORCL file=orapwDUP`

Create an Oracle Password file (_orapw$anaj_) for the source database in ANAJ session. 
The password is the same as for the target database.

`C:\> app\User\product\11.2.0\dbhome_1\dbs  orapwd password=ORCL file=orapw$ANAJ entries=5`

### 4. Create initialization Parameter file for the Duplicate database
`db_name='dup'`

`diagnostic_dest='C:\app\User\'`

`DB_FILE_name_CONVERT=('C:\app\User\oradata\ANAJ','C:\app\User\oradata\dup')`

`LOG_FILE_NAME_CONVERT=('C:\app\User\oradata\ANAJ','C:\app\User\oradata\dup')`

`Memory_TARGET=262144000`

`control_files = (C:\app\User\oradata\dup\control01.ctl)`

`db_block_size=8192`

`compatible ='11.2.0'`


### 5. Prepare auxiliary instance with files _**listener.ora**_ and _**tnsnames.ora**_

Location of files is in C:\> app\User\product\11.2.0\dbhome_1\NETWORK\ADMIN

Add the following entries into **listener.ora **

`SID_LIST_LISTENER =`

	`(SID_DESC =`

		`(SID_NAME = dup)`

		`(ORACLE_HOME = C:/app/User/product/11.2.0/dbhome_1)`

	`)`


Add Following TNS Entries To BOTH Auxiliary And Target **Tnsnames.Ora** File:

`dup =`

 `(DESCRIPTION =`

 `(ADDRESS_LIST =`

	`(ADDRESS = (PROTOCOL = TCP)(HOST = localhost.localdomain)(PORT = 1521))`

 `)`

 `(CONNECT_DATA =`

	`(SERVER = DEDICATED)`

	`(SERVICE_NAME = dup)`

 `)`

 `)`

 
`ANAJ =`

 `(DESCRIPTION =`

 `(ADDRESS = (PROTOCOL = TCP)(HOST = localhost.localdomain)(PORT = 1521))`

 `(CONNECT_DATA =`

	`(SERVER = DEDICATED)`

	`(SERVICE_NAME = ANAJ)`

 `)`

 `)`


### 7. In ANAJ session connect to the source database

_Connect to sqlplus_

`C:\> sqlplus /nolog	`

_Connect to SQL as sys user_

`SQL> connect /as sysdba`

_Open database	_

`SQL> startup`

_Check Database log list_

`SQL> archive log list`

_Set Database in „archive mode“ if it's not already set_

`SQL> shutdown immediate`

`SQL> startup mount`

`SQL> alter database archivelog;`

`SQL> alter database open;`


### 8. Test Connectivity To Auxiliary And Target Instances From BOTH Hosts Using **TNS**

`C:\Windows\system32> tnsping ANAJ`

`C:\Windows\system32> tnsping dup`


### 9. Set target database in nomount mode

_Connect to target database_

`C:\> set ORACLE_SID = dup`

`C:\> sqlplus /as sysdba`

`SQL> startup nomount pfile=C:\app\User\product\11.2.0\dbhome_1\database\initdup.ora`


### 10. Reset listener

`C:\Windows\system32> lsnrctl reload`


### 11. On The Auxiliary Host, Start RMAN And Run The DUPLICATE Command.


`C:\Windows\system32> rman target sys/ANAJ auxiliary sys/sys@dup`

_Set files you want to duplicate_

`RMAN run {`

`SET NEWNAME FOR DATAFILE 1 TO 'C:\app\User\oradata\dup\SYSTEM01.DBF';`

`SET NEWNAME FOR DATAFILE 2 TO 'C:\app\User\oradata\dup\SYSAUX01.DBF';`

`SET NEWNAME FOR DATAFILE 3 TO 'C:\app\User\oradata\dup\UNDOTBS01.DBF';`

`SET NEWNAME FOR TEMPFILE 1 TO 'C:\app\User\oradata\dup\TEMP01.DBF';`

`duplicate target database to dup nofilenamecheck;`

`}`

The files should be duplicated in **_C:\app\User\oradata\dup_** directory.
