# Lab 8 - Audit Vault #

This lab will demonstrate how to use Audit Vault to manage the audit data produced by an Oracle database.

## Disclaimer ##
The following is intended to outline our general product direction. It is intended for information purposes only, and may not be incorporated into any contract. It is not a commitment to deliver any material, code, or functionality, and should not be relied upon in making purchasing decisions. The development, release, and timing of any features or functionality described for Oracle’s products remains at the sole discretion of Oracle.

## Requirements ##

Instructions in this lab expect that you have completed all the previous labs in the workshop.

## Introduction  ##

A large deployment of Oracle (or non-Oracle) databases can produce a great amount of audit data to consolidate and manage over time. In addition to audit data from databases, other systems such operating systems and file systems produce audit trails that can be used to analyze events relevant to security.

Good practice dictates that audit data should be transmitted to a remote centralized location where it is secure from tampering by the individuals whose activities are being audited.

This is also recommended by DBSAT findings such as the following one:


![Alt text](./images/img01.png " ")

The Audit Vault Server solves these problems by collecting and consolidating audit data, providing comprehensive reports and alerts for forensic and compliance purposes and managing its retention over time.

In this lab, an Audit Vault Server has already been provisioned in virtual machine **av**. We will register **secdb** as a Secure Target, deploy the **Audit Vault agent** to **secdb** and configure Audit Trails to collect audit data from **secdb**.

## Step 1 : Registering host and deploying the agent ##

First thing to do is to register secdb as a secure target and deploy the agent.

From the **VNC connection to secdb**, start Firefox and connect to the Audit Vault as Super Administrator:

*	URL : **https://av**
*	Username	: **AVADMIN**
*	Password	: **Reganam_1**

![Alt text](./images/img02.png " ")

Go to **Hosts** > **Register** and enter the following values then click on **Save**.

*	Host name		: **secdb.localdomain**
*	Host IP			: **10.0.0.2**

Make note of the automatically generated **Agent Activation Key** (e.g. copy and paste it to **Gedit**)

![Alt text](./images/img03.png " ")

Now, go to **Hosts (tab)** > **Hosts (menu)** > **Agent** and click on **Download for Agent Release 12.2**.

Save **agent.jar** to ``/home/oracle/HOL/lab08_av``

![Alt text](./images/img04.png " ")

We can now configure and start the agent.

````
$ <copy>cd /home/oracle/HOL/lab08_av</copy>
````

````
$ <copy>java -jar agent.jar -d /u01/oracle/avagent</copy>

[oracle@secdb lab08_av]$ java -jar agent.jar -d /u01/oracle/avagent
Checking for updates...
Agent is updating. This operation may take a few minutes. Please wait...
Agent updated successfully.
Agent installed successfully.
If deploying hostmonitor please refer to product documentation for additional installation steps.
````

````
$ <copy>cd /u01/oracle/avagent/bin</copy>
````

Run the following statement to enter the **Agent Activation Key** previously generated.

````
$ <copy>./agentctl start -k</copy>

[oracle@secdb lab08_av]$ cd /u01/oracle/avagent/bin
[oracle@secdb bin]$ ./agentctl start -k
Enter Activation Key:
Agent started successfully.
````

The Audit Vault agent should now be running.

````
$ <copy>./agentctl status</copy>

[oracle@secdb bin]$ agentctl status
Agent is running.
````

## Step 2 : Configuring Audit Trails ##

Now that the agent has been installed and started on secdb, it is possible to connect to Audit Vault from a local browser on your workstation.

You can then connect to the Audit Vault Server console using its public IP at the following URL:

**https://&lt AV PUBLIC IP &gt**

* User Name : **AVADMIN**
* Password  : **Reganam_1**

![Alt text](./images/img05.png " ")

Allow a security exception as we are just using a self-signed certificate.

![Alt text](./images/img06.png " ")

![Alt text](./images/img07.png " ")

Please also accept to run Adobe Flash.

![Alt text](./images/img08.png " ")

![Alt text](./images/img09.png " ")

We can now register secure targets and create the audit trails.

Go to **Secured Targets** > **Target** > **Register** and create the following two secure targets:

*	New Secured Target Name: **cont**
*	Description: **Container database**
*	Secured Target Type: **Database**
*	Host name: **secdb**
*	Port: **1521**
*	Service Name: **cont**
*	Username: **system**
*	Password: **MyDbPwd#1**

Click on **Save** in the upper right corner.

*	New Secured Target Name: **pdb1**
*	Description: **Pluggable database**
*	Secured Target Type: **Database**
*	Host name: **secdb**
*	Port: **1521**
*	Service Name: **pdb1**
*	Username: **system**
*	Password: **MyDbPwd#1**

Click on **Save** in the upper right corner.

![Alt text](./images/img10.png " ")

Let’s collect audit data created inside the database (view UNIFIED\_AUDIT\_TRAIL) and in operating system files (as per parameter audit\_file\_dest = /u01/oracle/admin/CONT/adump).

Go to **Secured Targets** > **Audit Trails** > **Add** and create the following three audit trails:


*Audit data stored in the Container database:*

*	audit trail type: **TABLE**
*	collection host: **secdb.localdomain**
*	secured target: **cont**
*	trail location: **UNIFIED\_AUDIT\_TRAIL**
*	collection plugin: **com.oracle.av.plugin.oracle**

*Audit data stored in the Pluggable database:*

*	audit trail type: **TABLE**
*	collection host: **secdb.localdomain**
*	secured target: **pdb1**
*	trail location: **UNIFIED\_AUDIT\_TRAIL**
*	collection plugin: **com.oracle.av.plugin.oracle**

*Audit data stored outside the database:*

*	audit trail type: **DIRECTORY**
*	collection host: **secdb.localdomain**
*	secured target: **cont**
*	trail location: **/u01/oracle/db/admin/CONT/adump**
*	collection plugin: **com.oracle.av.plugin.oracle**


The collection should start automatically after a few seconds. The Collection Status arrows will change from down (Red Arrow Down) to up (Green Arrow Up).

![Alt text](./images/img11.png " ")


## Step 3 : Using Audit Vault Reporting ##

The Oracle Audit Vault Server provides powerful built-in reports to monitor a wide range of activity, including privileged user activity and changes to database structures. The reports provide visibility into activities and provide detailed information regarding the who, what, when and where of database access.


Let's generate new audit records by executing the following script which will create a number of violations that will be audited.

````
$ <copy>cd /home/oracle/HOL/lab08_av</copy>
````

````
$ <copy>av_createrecords.sh</copy>

[oracle@secdb lab08_av]$ av_createrecords.sh

(...)

SQL> delete from hr.regions;
delete from hr.regions
               *
ERROR at line 1:
ORA-01031: insufficient privileges
````

To view the audit records just collected, logout from AVADMIN and login as AVAUDITOR using the following credentials:

*	Username	: **AVAUDITOR**
*	Password	: **Reganam_1**

![Alt text](./images/img12.png " ")

Scroll down the Audit Vault and Database Firewall Home page and notice the graphs that will begin to populate as we move through the rest of the lab. Also, you will likely see an indication of failed logins at the far right of the Failed Logins graph. Sample output is shown below.

![Alt text](./images/img13.png " ")

Click on the **Reports** tab and **Activity Reports** under **Built-in Reports**.

Select the **Data Access** report within the **Activity Reports** section (expand that section).  View the report by clicking on its name and verify that you see audit records that were collected on today’s date.

Remove the default  filters by clicking on the cross.

![Alt text](./images/img14.png " ")

Add a new filter by clicking on **Actions** > **Filter** and selecting the secure target **pdb1**

![Alt text](./images/img15.png " ")

![Alt text](./images/img16.png " ")

The report should now be filtered on **pdb1** and show the audit records previously generated.

Detailed information for one record can be obtained by clicking on the left side icon.

![Alt text](./images/img17.png " ")



![Alt text](./images/img18.png " ")
![Alt text](./images/img19.png " ")


Try other interesting reports, for example:

**Reports** > **Activity Reports** > **Login Failures**

![Alt text](./images/img20.png " ")

**Reports** > **Activity Reports** > **Database Schema Changes**

![Alt text](./images/img21.png " ")

**Reports** > **Specialized Reports** > **Database Vault Reports** > **Database Vault Activity**

![Alt text](./images/img22.png " ")

![Alt text](./images/img23.png " ")

Audit Vault can also be used to retrieve and display **user entitlements** (ie granted privileges and roles).

First, retrieve user entitlement data for **pdb1**.

![Alt text](./images/img24.png " ")

Click a couple of times on the **Jobs** link to show the status of the job.

![Alt text](./images/img25.png " ")

Wait until the job shows as **completed**. You should now be able to check the report.

**Reports** > **Activity Reports** > **Entitlement Reports** > **User Accounts**

The **User Accounts** Report displays the latest snapshot of Database users with their account profile and values for sources registered with Oracle Audit Vault.

Initially you will not see any reporting data.  Click on **Go** (next to the **Latest** drop down).  This will bring back the latest Entitlement snapshot that you just retrieved from the source database.  

![Alt text](./images/img26.png " ")

It is possible to take multiple snapshots of user entitlement data.

This report lets you look at the data for any of these snapshots.  You can also compare different snapshots to see how the data has changed over time, as it is generally be more useful to report on what has **changed** over a specific period.

![Alt text](./images/img27.png " ")

## Step 4 : Compliance Reports ##

The reports shown here are intended to help you meet your compliance reporting requirements as quickly as possible, across **GDPR**, **PCI**, **Sarbanes-Oxley**, **HIPAA** (healthcare-related) and other areas.  In order to generate compliance reports for a secured target, you must add it to a compliance report group.

Let us start by adding our secure target **pdb1** as member of a compliance group (e.g. **Payment Card Industry**).

Click on the **Compliance Reports** menu and open the **Payment Card Industry (PCI) Reports** category.  Click the **Go** button, as shown in the figure below.

![Alt text](./images/img28.png " ")

Select **pdb1**, click on **Add Members** and **save** the change.

![Alt text](./images/img29.png " ")

We can now review some reports.  Click on the **Logins Failures** link under the **Payment Card Industry (PCI)** Reports section.

![Alt text](./images/img30.png " ")

Select the **Database Schema Changes** report in the **PCI** section, but this time we will schedule the report so we can create a PDF version of a report, which can then be sent to people who require it.

![Alt text](./images/img31.png " ")

You can schedule a report to run immediately or on a regular basis.
Click on Schedule to run this job immediately.

![Alt text](./images/img32.png " ")

Once the report has been generated (click on the **Jobs** link), you can save it or open it.

![Alt text](./images/img33.png " ")

![Alt text](./images/img34.png " ")

![Alt text](./images/img35.png " ")


## Step 5 : Capturing All Activity ##

One advantage of Unified Auditing is that all database activities are captured in the same audit trail, including **SQL*Loader Direct Mode** or **Data Pump**.

Let us verify that **UNIFIED\_AUDIT\_TRAIL** does capture **DATA PUMP** activity.

In a default **Database Vault** environment, **SYSTEM** loses the **BECOME USER** privilege, which is required to run Data Pump jobs.

We will first give back this privilege to SYSTEM. Please note that this change would not be sufficient to run Data Pump as SYSTEM on realm-protected data.

Run the following scripts from a terminal window to the secdb server.

````
$ <copy>cd /home/oracle/HOL/lab08_av/dp</copy>
````

````
$ <copy>dp00_grant_become_user.sh</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Apr 28 12:55:08 2020
Version 19.6.0.0.0
Connected.

SQL> alter session set container=PDB1;
Session altered.

SQL> grant become user to system;
Grant succeeded.
````

Create a DIRECTORY object.

````
$ <copy>sqlplus /nolog</copy>
````


````
$ <copy>@dp10_data_dir.sql</copy>

SQL*Plus: Release 19.0.0.0.0 - Production on Tue Apr 28 12:55:08 2020
Version 19.6.0.0.0
Connected.
SQL> alter session set container=pdb1;
Session altered.

SQL> create or replace directory data_dir as '/home/oracle/HOL/lab08_av/dp';
Directory created.

SQL> grant read, write on directory data_dir to system;
Grant succeeded.
````

We can now run a Data Pump import of the SCOTT schema as SYSTEM:


````
$ <copy>dp20_import.sh</copy>

[oracle@secdb dp]$ dp20_import.sh

(...)
Starting "SYSTEM"."SYS_IMPORT_SCHEMA_01":  system/********@secdb/pdb1 parfile=dp20_import.ini
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "SCOTT"."EMP"                               8.773 KB      14 rows
. . imported "SCOTT"."DEPT"                              6.023 KB       4 rows
. . imported "SCOTT"."SALGRADE"                          5.953 KB       5 rows
. . imported "SCOTT"."BONUS"                                 0 KB       0 rows
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/POST_SCHEMA/PROCACT_SCHEMA
Job "SYSTEM"."SYS_IMPORT_SCHEMA_01" successfully completed at Tue (...)
````

Select the **Data Schema Changes** report in the **Built-in Reports** section and filter information on **PDB1**.

You should see the audit trace of Data Pump workers (**DW00**) processes.

![Alt text](./images/img36.png " ")

## Step 6 : Data Privacy Reports ##

DBSAT Discoverer CSV output can be uploaded in order to run Data Privacy reports from Audit Vault (see Auditor's guide, chapter 6 - Data Privacy reports).

We will start by loading DBSAT Discoverer’s report into Audit Vault.

From the desktop connection to **secdb** (using **VNC**):

*	Login to Audit Vault as **AVADMIN/Reganam_1**
*	Go to **Secure Targets**
*	Click on **pdb1** to show the **Modify Secure Target** page
*	Scroll  down to **Sensitive Objects**

![Alt text](./images/img37.png " ")

* Browse to /home/oracle/HOL/lab01\_dbsat/dbsat/install/pdb1sensitivedata_discover.csv
*	Upload the csv file
*	click on **Save**

We can now add **PDB1** to the **Data Privacy compliance** report
group and view **Data Privacy** reports.

*	Login to Audit Vault Server as **AVAUDITOR** / **Reganam_1**
*	Go to **Reports** > **Built-in Reports** > **Compliance Reports**
*	Click on the **Go** button for **Data Privacy Reports**

![Alt text](./images/img38.png " ")

Select Secured Target **pdb1** and click **Add Members** and **Save**.

![Alt text](./images/img39.png " ")

We can finally review the built-in Data Privacy reports.

![Alt text](./images/img40.png " ")

For instance:
**Sensitive Data**

![Alt text](./images/img41.png " ")

## Step 7 : Audit Vault Alerting ##

After monitoring and collecting database activity your organization may want to be alerted when certain activities occur.  It is possible to identify high-risk activities that the security team needs to be made aware of.  These may include account management and database structural changes.  The alerts will let the security team know when there has been activity in these areas

During this lab you will:

1. Modify the email template for Audit Vault Alerts
2. Add a new Audit Vault Alert Status
3. Create an Audit Vault Alert with the Web Interface
4. Test that the alert is functioning
5. View the near real-time nature of alert functionality

We'll start by modifying the email template for Audit Vault Alerts.

Log into the Audit Vault console as **AVAUDITOR**.

Select the **Notifications** / **Email Templates** page.  From here will be able to manage the existing template definitions and create new ones.  

![Alt text](./images/img42.png " ")

Edit the **Alert Notification** Template, which is the default template used for sending emails.  You could create a new template, but for the purpose of this lab we will just edit the existing one.  Add the **#AlertStatus#** field into the email subject, as shown in the screen below.  Click the **Save** button once completed.

![Alt text](./images/img43.png " ")


Let's add a new Audit Vault **Alert Status**

Navigate to the **Policy** tab, then to the **Alert** page.  Click on the **Manage Alert Status Values** button.

![Alt text](./images/img44.png " ")

You will see that there are two default Alert status values.  

These values are used to maintain a status for each alert that is created in Audit Vault.  You can then manage alerts according to your business requirements.   

We will be adding a new status to record that we are reviewing a given alert.  

Click on the **Create** button. Now add a new Alert Status:

* Status Value: **Reviewing**
* Description: **Alert being reviewed**

![Alt text](./images/img45.png " ")


We will now create a new alert in Audit Vault. This alert will let us know when a new table has been created. Click on **Policy** > **Alert Definition**.  Click on the **Create** button.

* Name: **CREATE_TABLE**
* Secured Target Type: **Oracle Database**
* Alert Severity: **Critical**
* Threshold (times): **1**
* Duration (min): **0**
* Group By (field): **&ltDefault&gt**
* Description: **Alert when a table is created**
* Condition: **:EVENT_NAME='CREATE TABLE'**
* Notification Action: Select the **Alert Notification Template**
* Mailing List: **&ltLeave Blank&gt**

![Alt text](./images/img46.png " ")

Let us test that the alert is functioning! Run the following script to create once again a new table.

````
$ <copy>cd /home/oracle/HOL/lab08_av</copy>
````


````
$ <copy>av_createrecords.sh</copy>

[oracle@secdb lab08_av]$ av_createrecords.sh

(…)
SQL> create table job2 as select * from jobs;
Table created.
(…)
````

In the Audit Vault Server, the **Home** page should now show the alert (you may have to wait for a few seconds for the alert to be reported).

![Alt text](./images/img47.png " ")

You can also view the alert report:

**Reports** > **Built-in Reports** > **Activity Reports** > **Critical Alerts**

![Alt text](./images/img48.png " ")

From the home page, you can now edit the **alert status**, for example to close the alert.

![Alt text](./images/img49.png " ")


## Step 8 : Archiving and Purging the Audit Trail ##

Audit information should follow a full lifecycle:

1. Audit data is captured in the database and stored in the AUDSYS schema;
2. Audit Vault Agent collects this information  and automatically sets a timestamp on audit data that has been collected;
3. A database job can then run regularly and purge audit information that has been already collected;
4. Audit information stays in Audit Vault during a customizable retention time;
5. At the expiration of this retention time, audit data can be extracted from Audit Vault and archived;
6. At the expiration of the archival time, the archive may be destroyed.

Oracle AVDF is integrated with the **DBMS\_AUDIT\_MGMT** package in an Oracle Database. This integration automates the purging of audit records from **UNIFIED\_AUDIT\_TRAIL**, **AUD$**, and **FGA\_LOG$** views and from the operating system **.aud** and **.xml** files after they have been successfully inserted into the Audit Vault Server repository.

The Audit Vault Agent automatically sets a timestamp on audit data that has been collected. We can use the system procedure **DBMS\_AUDIT\_MGMT.CREATE\_PURGE\_JOB** to periodically purge audit data that has been already collected.

1) Let us count the existing rows in audit tables.

````
$ <copy>cd /home/oracle/HOL/lab08_av/purgejob</copy>
````


````
$ <copy>cleanup00_count.sh</copy>

SQL> select count(*) from sys.aud$;
  COUNT(*)
----------
         0

SQL> select count(*) from unified_audit_trail;

  COUNT(*)
----------
       312

SQL> alter session set container=pdb1;
Session altered.

SQL> select count(*) from sys.aud$;
  COUNT(*)
----------
         0

SQL> select count(*) from unified_audit_trail;
  COUNT(*)
----------
       556
````

2) Initialize the purging job by setting it to run every hour by default

````
$ <copy>cleanup10_init.sh</copy>

SQL> BEGIN
  2   DBMS_AUDIT_MGMT.INIT_CLEANUP(
  3    AUDIT_TRAIL_TYPE            => DBMS_AUDIT_MGMT.AUDIT_TRAIL_ALL,
  4    DEFAULT_CLEANUP_INTERVAL    => 1 );
  5  END;
  6  /
PL/SQL procedure successfully completed.

SQL> alter session set container=pdb1;
Session altered.

SQL> BEGIN
  2   DBMS_AUDIT_MGMT.INIT_CLEANUP(
  3    AUDIT_TRAIL_TYPE            => DBMS_AUDIT_MGMT.AUDIT_TRAIL_ALL,
  4    DEFAULT_CLEANUP_INTERVAL    => 1 );
  5  END;
  6  /
PL/SQL procedure successfully completed.
````

3) Check the status of the purging job

````
$ <copy>cleanup20_show.sh</copy>

(…)
CDB is initialized for cleanup

(…)
PDB1 is initialized for cleanup
````

4) Schedule the purging job

````
$ <copy>cleanup30_schedule.sh</copy>

SQL> -- creates a purge job to run every hour to purge the audit records
SQL> -- in the container database
SQL> --
SQL> BEGIN
  2    DBMS_AUDIT_MGMT.CREATE_PURGE_JOB (
  3     AUDIT_TRAIL_TYPE            => DBMS_AUDIT_MGMT.AUDIT_TRAIL_ALL,
  4     AUDIT_TRAIL_PURGE_INTERVAL  => 1,
  5     AUDIT_TRAIL_PURGE_NAME      => 'AUDIT_CLEANUP_CDB',
  6     USE_LAST_ARCH_TIMESTAMP     => TRUE );
  7  END;
  8  /
PL/SQL procedure successfully completed.

SQL> alter session set container=pdb1;
Session altered.

SQL> -- creates a purge job to run every hour to purge the audit records
SQL> -- in the pluggable database
SQL> --
SQL> BEGIN
  2    DBMS_AUDIT_MGMT.CREATE_PURGE_JOB (
  3     AUDIT_TRAIL_TYPE            => DBMS_AUDIT_MGMT.AUDIT_TRAIL_ALL,
  4     AUDIT_TRAIL_PURGE_INTERVAL  => 1,
  5     AUDIT_TRAIL_PURGE_NAME      => 'AUDIT_CLEANUP_PDB1',
  6     USE_LAST_ARCH_TIMESTAMP     => TRUE );
  7  END;
  8  /
PL/SQL procedure successfully completed.
````

5) Verify that the number of rows in audit tables drops

````
$ <copy>cleanup00_count.sh</copy>

SQL> select count(*) from sys.aud$;
  COUNT(*)
----------
         0

SQL> select count(*) from unified_audit_trail;
  COUNT(*)
----------
         5

SQL> alter session set container=pdb1;
Session altered.

SQL> select count(*) from sys.aud$;
  COUNT(*)
----------
         0

SQL> select count(*) from unified_audit_trail;
  COUNT(*)
----------
        15
````

This completes the **Audit Vault** lab. You can continue with **Lab 9: Data Masking.**

## Acknowledgements

- **Authors** - Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - May 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.
