# Lab 6: Database Vault

Initial description

## Disclaimer ##
The following is intended to outline our general product direction. It is intended for information purposes only, and may not be incorporated into any contract. It is not a commitment to deliver any material, code, or functionality, and should not be relied upon in making purchasing decisions. The development, release, and timing of any features or functionality described for Oracle’s products remains at the sole discretion of Oracle.

## Requirements ##

Instructions in this lab expect that you have completed all the previous labs in the workshop.

## Introduction  ##

Many regulations require that privileged users such as DBAs be not able to read sensitive data in the database.
Oracle Database Vault provides this kind of control, which prevent unauthorized privileged users from accessing sensitive data. It can also prevent unauthorized database changes.
DBSAT has detected that Database Vault was not yet is use in our database.

![Alt text](./images/img01.png " ")

In this chapter, we will configure Database Vault for both the Container Database **CONT** and **PDB1**.

## Step 1 : Configure Database Vault ##

### Step 1a : Enable Database Vault in the Container Database ###

We first create (as **SYS**) the common users which will become the **Database Vault Owner** and the **Database Vault Account Manager** and then call **configure_dv** to configure Database Vault.

Run the following script from a terminal window to the secdb server.

````
$ <copy>cd /home/oracle/HOL/lab06_dbv/a_setup</copy>
````

````
$ <copy>dbvsetup10_config.sh</copy>

(...)
SQL> --
SQL> -- create users
SQL> --
SQL> grant create session, set container to c##dbvo identified by "_c2h5oh_" container=all;
Grant succeeded.
SQL> grant create session, set container to c##dbvam identified by "_c2h5oh_" container=all;
Grant succeeded.
SQL> --
SQL> -- Configure Database Vault as SYS at CDB level
SQL> --
SQL> begin
  2    dvsys.configure_dv(
  3      dvowner_uname     => 'C##DBVO',
  4      dvacctmgr_uname   => 'C##DBVAM');
  5  end;
  6  /
PL/SQL procedure successfully completed.
(...)
````

We now need to **enable** Database Vault in the CDB.

````
$ <copy>dbvsetup11_enable.sh</copy>

(...)
SQL> begin
  2    dbms_macadm.enable_dv;
  3  end;
  4  /
PL/SQL procedure successfully completed.
(...)
````

Finally we need to **restart** the instance.

````
$ <copy>dbvsetup12_restart.sh</copy>

(...)
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.
(...)

SQL> -- Control status
SQL> select parameter, value from v$option where parameter = 'Oracle Database Vault';

PARAMETER
---------
VALUE
-----
Oracle Database Vault
TRUE
````

### Step 1b : Enable Database Vault in the Pluggable Database ###

We need to run similar scripts for the pluggable database **PDB1**.

Create common users to become the **Database Vault Owner** and the **Database Account Manager**.

````
$ <copy>dbvsetup20_configPDB1.sh</copy>

(...)
SQL> alter session set container=pdb1;
Session altered.

SQL> --
SQL> -- grant privileges
SQL> --
SQL> grant create session, set container to c##dbvo;
Grant succeeded.

SQL> grant create session, set container to c##dbvam;
Grant succeeded.

SQL> --
SQL> -- Configure Database Vault as SYS
SQL> --
SQL> begin
  2    dvsys.configure_dv(
  3      dvowner_uname     => 'C##DBVO',
  4      dvacctmgr_uname   => 'C##DBVAM');
  5  end;
  6  /
PL/SQL procedure successfully completed.
(...)
````

**Enable** Database Vault in PDB1.

````
$ <copy>dbvsetup21_enablePDB1.sh</copy>

(...)
SQL> begin
  2    dbms_macadm.enable_dv;
  3  end;
  4  /
PL/SQL procedure successfully completed.
(...)
````

**Restart** PDB1.

````
$ <copy>dbvsetup22_restartPDB1.sh</copy>

(...)
SQL> alter session set container=pdb1;
Session altered.

SQL> alter pluggable database pdb1 close immediate;
Pluggable database altered.

SQL> alter pluggable database pdb1 open;
Pluggable database altered.

SQL> -- Control status
SQL> select parameter, value from v$option where parameter = 'Oracle Database Vault';

PARAMETER
---------
VALUE
-----
Oracle Database Vault
TRUE
(...)
````


## Step 2 : Operations Control  ##

**Operations Control** is a Database Vault **19c** new feature which makes very easy to restrict common users (e.g. **SYS** or **SYSTEM**) from accessing pluggable database local data (e.g. **PDB1** local data).

Ops Control is useful for situations where a database administrator must log in to the CDB root as a highly privileged user, but still not be able to access PDB customer data.

Database operations control does not block PDB database administrators. To block these users, enable Oracle Database Vault in the PDB and then use other Database Vault features such as **realm control** to block these users, as we'll see in **Step 3**.

Run the following script to enable Ops Control in the CDB.

````
$ <copy>cd /home/oracle/HOL/lab06_dbv/b_ops_control</copy>
````

````
$ <copy>opsctl_10_enable.sh</copy>

(...)
SQL*Plus: Release 19.0.0.0.0 - Production on Tue May 12 14:33:47 2020
Version 19.6.0.0.0

SQL> begin
  2          dbms_macadm.enable_app_protection;
  3  end;
  4  /
PL/SQL procedure successfully completed.
(...)
````

Verify the status (**DV\_APP\_PROTECTION** is **ENABLED**).

````
$ <copy>opsctl_20_status.sh</copy>

(...)
SQL> select * from dba_dv_status;

NAME                 STATUS
-------------------- --------------------
DV_APP_PROTECTION    ENABLED
DV_CONFIGURE_STATUS  TRUE
DV_ENABLE_STATUS     TRUE

(...)
````

Let us now verify that **SYS** or **SYSTEM** cannot access local data in **PDB1**.

````
$ <copy>opsctl_30_test_access.sh</copy>

(...)
SQL> connect system/"MyDbPwd#1"@secdb/pdb1
Connected.
SQL> select * from hr.regions;
select * from hr.regions
                 *
ERROR at line 1:
ORA-01031: insufficient privileges

SQL> select * from scott.dept;
select * from scott.dept
                    *
ERROR at line 1:
ORA-01031: insufficient privileges

SQL> -- Testing access from SYS
SQL> connect sys/"MyDbPwd#1"@secdb/pdb1 as sysdba
Connected.
SQL> col country_name for a20
SQL> select * from hcm.countries where rownum <6;
select * from hcm.countries where rownum <6
                  *
ERROR at line 1:
ORA-01031: insufficient privileges
(...)
````

For the rest of the workshop, we will however **disable Operations Control**. Please run the following script.

````
$ <copy>opsctl_90_disable.sh</copy>

(...)
SQL> begin
  2          dbms_macadm.disable_app_protection;
  3  end;
  4  /

PL/SQL procedure successfully completed.
(...)
````


## Step 3 : Create a Database Vault Realm over HR schema in PDB1  ##

A **realm** is a grouping of database schemas, database objects, and database roles that must be secured for a given application. Only users who have been granted **realm authorization** as either a realm **owner** or a realm **participant** can use their **system privileges** to access secured objects in the realm.

In the following demo, we will execute the following scenario:

*	Create  a realm over the **HR** schema in **PDB1** – done by the Database Vault Owner
*	Create a **HR_ROLE** role to grant application privileges to users. We’ll need to also protect this role by putting in inside the realm to prevent privileged users (**SYS** or **SYSTEM**) to modify it or to grant it to themselves
*	We’ll also create two application users **appuser1** and **appuser2** and grant the required role only to **appuser1**

### Step 3a : Create a realm HR_REALM over the HR schema ###

Create realm **HR_REALM** over the **HR** schema in **PDB1**. Run the following script from a terminal window to the secdb server

````
$ <copy>cd /home/oracle/HOL/lab06_dbv/b_realm</copy>
````

````
$ <copy>dbv10_dbvo_realm.sh</copy>

(...)
SQL> /*
SQL>  * Create a mandatory realm to protect schema HR
SQL>  */
SQL> BEGIN
  2    DBMS_MACADM.CREATE_REALM(
  3      realm_name    => 'HR Realm',
  4      description   => 'This mandatory realm protects schema HR',
  5      enabled       => 'Y',
  6      audit_options => '1',
  7      realm_type    =>'1' );
  8    DBMS_MACADM.ADD_OBJECT_TO_REALM(
  9      realm_name    => 'HR Realm',
 10      object_owner  => 'HR',
 11      object_name   => '%',
 12      object_type   => '%' );
 13    DBMS_MACADM.ADD_AUTH_TO_REALM(
 14      realm_name    => 'HR Realm',
 15      grantee       => 'HR',
 16      rule_set_name => '',
 17      auth_options  => DBMS_MACUTL.G_REALM_AUTH_OWNER );
 18  END;
 19  /
PL/SQL procedure successfully completed.
(...)
````

### Step 3b : Grant CREATE ROLE to the Application Manager ###

It is important to not create the role as **SYS**, but as the **Application Manager**. Otherwise the DBA will be able to later modify the role or grant it to himself. Run the following script from a terminal window to the secdb server:

````
$ <copy>cd /home/oracle/HOL/lab06_dbv/c_role</copy>
````

````
$ <copy>dbv20_sys_grant.sh</copy>

(...)
SQL> alter session set container=PDB1;
Session altered.

SQL> /*
SQL>  * Role will be created by HR and protected in the realm
SQL>  * => cannot be modified by DBAs
SQL>  */
SQL> grant create role to HR;
Grant succeeded.
(...)
````

### Step 3c : Create demo users ###

Run the following script.

````
$ <copy>dbv30_dbvam_create_users.sh</copy>

(...)
SQL> --
SQL> -- Need to be DBVAM to create users and grant CONNECT
SQL> --
SQL> create user appuser1 identified by MyDbPwd#1 account unlock;
User created.

SQL> create user appuser2 identified by MyDbPwd#1 account unlock;
User created.

SQL> grant create session to appuser1;
Grant succeeded.

SQL> grant create session to appuser2;
Grant succeeded.
(...)
````

### Step 3d :  Create an application role ###

Connect as the **Application Manager (HR)** to create the application role **APPROLE**. Run the following script from a terminal window to the secdb server. Note that the role is only granted to **APPUSER1** and not to **APPUSER2**.

````
$ <copy>dbv40_hr_roleprivs.sh</copy>

(...)
SQL> create role APPROLE;
Role created.

SQL> grant select, insert, update, delete on HR.regions to APPROLE;
Grant succeeded.

SQL> grant select, insert, update, delete on HR.countries to APPROLE;
Grant succeeded.

SQL> grant select, insert, update, delete on HR.locations to APPROLE;
Grant succeeded.

(...)

SQL> grant select, insert, update, delete on HR.job_history to APPROLE;
Grant succeeded.

SQL> grant execute on hr.ADD_JOB_HISTORY to APPROLE;
Grant succeeded.

SQL> grant execute on hr.SECURE_DML to APPROLE;
Grant succeeded.

SQL> grant APPROLE to appuser1;
Grant succeeded.
(...)
````

### Step 3e :  Protect the APPROLE role ###

Protect role APPROLE by placing it in the realm to prevent privileged users such as DBAs from granting the role to themselves.

Please note that because we created a **mandatory** realm, we also need to make it realm **participant** in order to allow users granted this role to use their privileges.

````
$ <copy>dbv50_dbvo_approle.sh</copy>

(...)
SQL> /*
SQL>  * Protects role APPROLE
SQL>  * - putting role APPROLE in the realm prevents DBA from granting the role to themselves
SQL>  * - putting role APPROLE as realm participant allows users granted this role to use their privileges
SQL>  */
SQL> BEGIN
  2    DBMS_MACADM.ADD_OBJECT_TO_REALM(
  3      realm_name      => 'HR Realm',
  4      object_owner    => '%',
  5      object_name     => 'APPROLE',
  6      object_type     => 'ROLE' );
  7    DBMS_MACADM.ADD_AUTH_TO_REALM(
  8      realm_name      => 'HR Realm',
  9      grantee         => 'APPROLE',
 10      rule_set_name   => '',
 11      auth_options    => DBMS_MACUTL.G_REALM_AUTH_PARTICIPANT );
 12  END;
 13  /
PL/SQL procedure successfully completed.
(...)
````

### Step 3f : Verification ###

We can now test from the dbclient client that only **APPUSER1** (and not **APPUSER2**) is able to run the application. Run the following script from a terminal window to the **dbclient** client.

First from APPUSER1 :

````
$ <copy>cd /home/oracle/HOL/lab06_dbv</copy>
````

````
$ <copy>run_applic_appuser1.sh</copy>

(...)
SQL> select * from hr.regions;

 REGION_ID REGION_NAME
---------- -------------------------
         1 Europe
         2 Americas
         3 Asia
         4 Middle East and Africa
(...)
````


Then from APPUSER2 :

````
$ <copy>run_applic_appuser2.sh</copy>

(...)
SQL> select * from hr.regions;
select * from hr.regions
                 *
ERROR at line 1:
ORA-00942: table or view does not exist
(...)
````

This completes the **Database Vault** lab. You can continue with **Lab 7: Database Audit**

## Acknowledgements

- **Authors** - Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - May 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.
