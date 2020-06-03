# Lab 3 - Provision Audit and Alert Policies #


## Objectives

* Provision audit and alert policies on a target database
* View details for an audit trail in Oracle Data Safe
* Enable a custom audit policy on a target database


## Disclaimer ##

The following is intended to outline our general product direction. It is intended for information purposes only, and may not be incorporated into any contract. It is not a commitment to deliver any material, code, or functionality, and should not be relied upon in making purchasing decisions. The development, release, and timing of any features or functionality described for Oracle’s products remains at the sole discretion of Oracle.

## Requirements ##

To complete this lab, you need to have the following:
* Login credentials and a tenancy name for the Oracle Cloud Infrastructure Console
* A compartment enabled with permission to create and use resources
* The ATP-S instance provisioned in Lab 1
* The access privileges configured in Lab 2

## STEP 1: Provision audit and alert policies on a target database by using the Activity Auditing wizard

In the Oracle Data Safe Console, click the **Home** tab, and then click the **Activity Auditing** tab.

<!--![alt text](./images/img01.png " ")-->
<img src="./images/img01.png" alt="picture" width="300" height="400" />

On the **Select Targets for Auditing** page, select the check box for your target database, and then click **Continue**.

![alt text](./images/img02.png " ")

On the **Retrieve Audit Policies** page, select the check box for your target database, and then click **Retrieve** to retrieve the audit policies for your database. Wait until a green check mark is displayed in the Retrieval Status column, and then click **Continue**.

![alt text](./images/img03.png " ")

Click **continue** to reach the **Review and Provision Audit and Alert Policies** page.

Click on the name of your database to open the Edit Policies dialog box.

![alt text](./images/img05.png " ")

On the **Audit Policies** tab in the Edit Policies dialog box, notice that the following Basic Auditing and Admin Activity Auditing policies are selected by default. Oracle recommends that you provision these policies. They are not provisioned by default.

*	Critical Database Activity
*	Login Events
*	Database Schema Changes (DDL)
*	Admin Activity

![alt text](./images/img06.png " ")

Expand **Oracle Pre-defined Policies** to view the list of Oracle predefined audit policies available on your ATP database. By default, the following policies are provisioned:

![alt text](./images/img07.png " ")

Provision the following additional policies:

![alt text](./images/img08.png " ")

Also the Center for Internet Security (CIS) Configuration policy is not provisioned and enabled by default. Check to enable it, then click **Provision**.

![alt text](./images/img09.png " ")

You can click the **Alert Policies** tab to review the selected alert policies.

![alt text](./images/img10.png " ")

On the **Review and Provision Audit and Alert Policies** page, wait for check marks to appear under all audit policy types, except for All User Activity, and then click **Continue**.

![alt text](./images/img11.png " ")

On the **Start Audit Collection** page, notice the following defaults:

*	The audit trail location is automatically set to UNIFIED\_AUDIT\_TRAIL
*	Audit collection is not yet started.
*	The auto purge feature is enabled.

![alt text](./images/img12.png " ")

Select your database and click **Start** to start collecting audit data.

A message at the top of the page states the **UNIFIED\_AUDIT\_TRAIL** is successfully created.

The Collection State column indicates that collection is **LOADING**, then **STARTING**, and then **COLLECTING**.

![alt text](./images/img13.png " ")

While the audit data is being collected, click **Done**.

## STEP 3: Execute some account management activity on the database

Using SQL Developer as user **ADMIN**, execute the following statements:

````
<copy>
create user bob identified by "OraclePTS#2020";
grant DWROLE to bob;
drop user bob;
</copy>
````

![alt text](./images/img14.png " ")

## STEP 4: View audited activity in Data Safe

Connect to the Data Safe console.

Click on the **Reports** tab.

In the **Auditing Activity** category on the left hand side, select the **All Activity** report.

The user creation records are shown after one or two minutes:

![alt text](./images/img15.png " ")

Continue with Lab 4.

## Acknowledgements ##

- **Authors** - Adrian Galindo & François Pons, PTS EMEA - April 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.
