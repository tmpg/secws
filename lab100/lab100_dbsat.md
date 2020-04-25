# Lab 1: Database Security Assessment Tool

The Oracle Database Security Assessment Tool (DBSAT) identifies sensitive data, analyzes database configurations, users, their entitlements and security policies to uncover security risks and improve the security posture of Oracle Databases within your organization. You can use DBSAT to implement and enforce security best practices in your organization and accelerate compliance with regulations such as the EU GDPR. 

DBSAT reports on existing sensitive data, the state of user accounts, role and privilege grants, and policies that control the use of various security features in the database. 

DBSAT generates two types of reports: 
- Database Security Risk Assessment report 
- Database Sensitive Data Assessment report 

You can use report findings to: 
- Fix immediate short-term risks 
- Implement a comprehensive security strategy 


## Requirements ##

Lab 0 completed. 

## Step 1: Installing DBSAT

We will now install and run DBSAT on PDB1 in Container Database CONT.

### Create a DBSAT user with appropiate privileges

Run `dbsat10_user.sh` to create a DBSAT local user with appropriate privileges.

````
[oracle@secdb ~ ]$ cd <copy>/home/oracle/HOL/lab01_dbsat/</copy>
[oracle@secdb lab01_dbsat]$ <copy>./dbsat10_user.sh</copy>

SQL*Plus: Release 18.0.0.0.0 - Production on Wed Mar 6 09:55:46 2019
Version 18.5.0.0.0
Connected.

SQL> alter session set container=pdb1;
Session altered.

SQL> --drop user dbsat cascade;
SQL> grant create session to dbsat identified by "MyDbPwd#1";
Grant succeeded.

SQL> grant select on sys.registry$history to dbsat;
Grant succeeded.

SQL> grant select_catalog_role to dbsat;
Grant succeeded.

SQL> -- if Database Vault has been enabled
SQL> grant dv_secanalyst to dbsat;
Grant succeeded.

SQL> -- 12c/18c only
SQL> grant audit_viewer to dbsat;
Grant succeeded.

SQL> grant capture_admin to dbsat;
Grant succeeded.

SQL> -- 11g and 12c/18c
SQL> grant select on sys.dba_users_with_defpwd to dbsat;
Grant succeeded.

SQL> -- 12c/18c only
SQL> grant select on audsys.aud$unified to dbsat;
Grant succeeded.

SQL> exit
Disconnected from Oracle Database 18c Enterprise Edition Release 18.0.0.0.0 - Production
Version 18.5.0.0.0

````

## Credits

**Authors** 

- Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - April 2020.