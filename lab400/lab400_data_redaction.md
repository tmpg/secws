# Lab 4: Data Redaction

**Oracle Data Redaction** enables you to move redaction capabilities out of applications and into the database.

## Disclaimer ##

The following is intended to outline our general product direction. It is intended for information purposes only, and may not be incorporated into any contract. It is not a commitment to deliver any material, code, or functionality, and should not be relied upon in making purchasing decisions. The development, release, and timing of any features or functionality described for Oracle’s products remains at the sole discretion of Oracle.

## Introduction  ##

**Oracle Data Redaction** provides an easy way to protect sensitive information that is displayed in applications by replacing it on-the-fly with valid redacted data, while keeping the applications running. Sensitive information is concealed according to flexible policies that provide conditional redaction and that are managed directly within the database.

For maximum transparency, redaction preserves the type and formatting of column data returned to applications, and it does not alter the underlying database blocks on disk or in cache.


You can redact column data by using one of the following methods:

* **Full redaction**. You redact all the contents of the column data. The redacted value returned to the querying user depends on the data type of the column. For example, columns of the NUMBER data type are redacted with a zero (0) and character data types are redacted with a blank space.
* **Partial redaction**. You redact a portion of the column data. For example, you can redact most of a Social Security number with asterisks (*), except for the last four digits.
* **Regular expressions**. You can use regular expressions in both full and partial redaction. This enables you to redact data based on a search pattern for the data. For example, you can use regular expressions to redact specific phone numbers or email addresses in your data.
* **Random redaction**. The redacted data presented to the querying user appears as randomly-generated values each time it is displayed, depending on the data type of the column.

Oracle Database applies the redaction at run time, at the moment users attempt to access the data (that is, at query-execution time). This solution works well in a dynamic production system in which data is constantly changing. During the time that the data is being redacted, all data processing is performed normally, and the back-end referential integrity constraints are preserved.  

Data redaction can help you to comply with industry regulations such as Payment Card Industry Data Security Standard (PCI DSS) and the Sarbanes-Oxley Act.

Redaction can be implemented via Enterprise Manager (which benefits from pre-defined redaction templates), SQL\*Developer and via the PL/SQL API (so SQL\*Plus or other client).


## Requirements ##

* Session open to **secdb** with user **oracle**
* session open to **dbclient** with user **oracle**   

## Creating a Simple Redaction Policy

### Requirements

DBSAT had identified a number of sensitive columns in the HCM schema:

![](./images/sensitive_data.png)

Let us create simple redaction policies for low privileged users on these three columns.

## Step 1: Create a low privileged user

Run the following script from a terminal window to the **secdb** server to create a low privileged user `hcm_clerk`:

````
[oracle@secdb ~]$ <copy>cd /home/oracle/HOL/lab04_redaction</copy>
````

````
[oracle@secdb lab04_redaction]$ <copy>redac10_create_user.sh</copy>

(...)
SQL> create user hcm_clerk identified by MyDbPwd#1 account unlock;
User created.

SQL> grant connect, resource, unlimited tablespace to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.regions to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.countries to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.locations to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.departments to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.jobs to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.employees to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.emp_extended to hcm_clerk;
Grant succeeded.

SQL> grant select on HCM.job_history to hcm_clerk;
Grant succeeded.
(...)
````

## Step 2: Create a redaction policy

Run the following script from a terminal window to the **secdb** server:

````
[oracle@secdb lab04_redaction]$ <copy>redac20_add_pol.sh</copy>

SQL> --
SQL> -- Full redaction of HCM.EMPLOYEES.SALARY except for HCM
SQL> --
SQL> BEGIN
  2  DBMS_REDACT.ADD_POLICY (
  3    object_schema    => 'HCM',
  4    object_name      => 'EMPLOYEES',
  5    policy_name      => 'REDACT_EMPLOYEES',
  6    column_name      => 'SALARY',
  7    function_type => DBMS_REDACT.FULL,
  8    expression       => 'SYS_CONTEXT ( ''USERENV'',''SESSION_USER'' ) != ''HCM''',
  9    enable           => TRUE
 10    );
 11  END;
 12  /
PL/SQL procedure successfully completed.

SQL> show errors
No errors.
SQL> --
SQL> -- Partial redaction of HCM.EMP_EXTENDED.TAXPAYERID except for HCM
SQL> --
SQL> BEGIN
  2  DBMS_REDACT.ADD_POLICY (
  3    object_schema       => 'HCM',
  4    object_name         => 'EMP_EXTENDED',
  5    policy_name         => 'REDACT_EMP_EXTENDED',
  6    column_name         => 'TAXPAYERID',
  7    function_type       => DBMS_REDACT.PARTIAL,
  8    function_parameters => 'VVVFVVFVVVV,VVV-VV-VVVV,X,1,5',
  9    expression          => 'SYS_CONTEXT ( ''USERENV'',''SESSION_USER'' ) != ''HCM''',
 10    enable              => TRUE
 11    );
 12  END;
 13  /
PL/SQL procedure successfully completed.

SQL> show errors
No errors.
SQL> --
SQL> -- Partial redaction of HCM.EMP_EXTENDED.PAYMENTACCOUNTNO except for HCM
SQL> --
SQL> BEGIN
  2  DBMS_REDACT.ALTER_POLICY (
  3    object_schema       => 'HCM',
  4    object_name         => 'EMP_EXTENDED',
  5    policy_name         => 'REDACT_EMP_EXTENDED',
  6    column_name         => 'PAYMENTACCOUNTNO',
  7    action              => DBMS_REDACT.ADD_COLUMN,
  8    function_type       => DBMS_REDACT.PARTIAL,
  9    function_parameters => 'VVVVVVVVVVVVVVVV,VVVVVVVVVVVVVVVV,X,1,12'
 10  );
 11  END;
 12  /
PL/SQL procedure successfully completed.
````

These policies mandate to connect as **HCM** in order to see real values for the three redacted columns.

However note that **SYSTEM** has the **EXP\_FULL\_DATABASE** role which includes the **EXEMPT REDACTION POLICY** system privilege. This means that the **SYS** and **SYSTEM** users can always bypass any existing Oracle Data Redaction policies and will always be able to view data from tables (or views) that have Data Redaction policies defined on them.


## Step 3: Verify the redaction policy

Let us run a quick test of our redaction policies.

Run the following script from a terminal window to the **secdb** server:

````
[oracle@secdb lab04_redaction]$ <copy>redac30_test_pol.sh</copy>

(...)
SQL> select SYS_CONTEXT('USERENV','SESSION_USER') from dual;
SYS_CONTEXT('USERENV','SESSION_USER')
-------------------------------------
HCM

SQL> select first_name, last_name, salary, taxpayerid, paymentaccountno
  2  from hcm.employees e1, hcm.emp_extended e2
  3  where e1.employee_id=e2.employee_id and rownum <10;

FIRST_NAME           LAST_NAME                     SALARY TAXPAYERID      PAYMENTACCOUNTNO
-------------------- ------------------------- ---------- --------------- --------------------
Donald               OConnell                        2600 123-45-6198     4321123454326198
Douglas              Grant                           2600 123-45-6199     4321123454326199
Jennifer             Whalen                          4400 123-45-6200     4321123454326200
Michael              Hartstein                      13000 123-45-6201     4321123454326201
Pat                  Fay                             6000 123-45-6202     4321123454326202
Susan                Mavris                          6500 123-45-6203     4321123454326203
Hermann              Baer                           10000 123-45-6204     4321123454326204
Shelley              Higgins                        12008 123-45-6205     4321123454326205
William              Gietz                           8300 123-45-6206     4321123454326206
9 rows selected.

SQL> connect hcm_clerk/MyDbPwd#1@pdb1
Connected.

SQL> show user
USER is "HCM_CLERK"
SQL> select SYS_CONTEXT('USERENV','SESSION_USER') from dual;

SYS_CONTEXT('USERENV','SESSION_USER')
-------------------------------------
HCM_CLERK

SQL> select first_name, last_name, salary, taxpayerid, paymentaccountno
  2  from hcm.employees e1, hcm.emp_extended e2
  3  where e1.employee_id=e2.employee_id and rownum <10;

FIRST_NAME           LAST_NAME                     SALARY TAXPAYERID      PAYMENTACCOUNTNO
-------------------- ------------------------- ---------- --------------- --------------------
Donald               OConnell                           0 XXX-XX-6198     XXXXXXXXXXXX6198
Douglas              Grant                              0 XXX-XX-6199     XXXXXXXXXXXX6199
Jennifer             Whalen                             0 XXX-XX-6200     XXXXXXXXXXXX6200
Michael              Hartstein                          0 XXX-XX-6201     XXXXXXXXXXXX6201
Pat                  Fay                                0 XXX-XX-6202     XXXXXXXXXXXX6202
Susan                Mavris                             0 XXX-XX-6203     XXXXXXXXXXXX6203
Hermann              Baer                               0 XXX-XX-6204     XXXXXXXXXXXX6204
Shelley              Higgins                            0 XXX-XX-6205     XXXXXXXXXXXX6205
William              Gietz                              0 XXX-XX-6206     XXXXXXXXXXXX6206
9 rows selected.
````

Let us also test our redaction policies from the client machine **dbclient**.

We will use an interesting **Window SQL** query which retrieves the highest salary in each department. Take the time to read the query and understand the syntax!

Use a terminal window to **dbclient** (as oracle) and run the following:

````
[oracle@dbclient ~]$ <copy>cd /home/oracle/HOL/lab04_redaction</copy>
````

````
[oracle@dbclient lab04_redaction]$ <copy>get_highest_sal.sh HCM</copy>

(...)
SQL> connect &&db_user/MyDbPwd#1@secdb/pdb1
Connected.
SQL> show user
USER is "HCM"
SQL> select x.first_name, x.last_name, x.salary
  2  from
  3  (
  4     select e.first_name, e.last_name, e.salary, e.department_id,
  5      row_number() over (partition by department_id order by salary desc) rn
  6          from HCM.employees e
  7  ) x
  8  where x.rn=1;

FIRST_NAME           LAST_NAME                     SALARY
-------------------- ------------------------- ----------
Jennifer             Whalen                          4400
Michael              Hartstein                      13000
Den                  Raphaely                       11000
Susan                Mavris                          6500
Adam                 Fripp                           8200
Alexander            Hunold                          9000
Hermann              Baer                           10000
John                 Russell                        14000
Steven               King                           24000
Nancy                Greenberg                      12008
Shelley              Higgins                        12008
Kimberely            Grant                           7000

12 rows selected.
(...)
````

As we connected as **HCM**, the data was not redacted.
Now let us run the same query as **HCM_CLERK** and the data will be redacted:

````
[oracle@dbclient lab04_redaction]$ <copy>get_highest_sal.sh HCM_CLERK</copy>

(...)
SQL> connect &&db_user/MyDbPwd#1@secdb/pdb1
Connected.
SQL> show user
USER is "HCM_CLERK"
SQL> select x.first_name, x.last_name, x.salary
  2  from
  3  (
  4     select e.first_name, e.last_name, e.salary, e.department_id,
  5      row_number() over (partition by department_id order by salary desc) rn
  6          from HCM.employees e
  7  ) x
  8  where x.rn=1;

FIRST_NAME           LAST_NAME                     SALARY
-------------------- ------------------------- ----------
Jennifer             Whalen                             0
Michael              Hartstein                          0
Den                  Raphaely                           0
Susan                Mavris                             0
Adam                 Fripp                              0
Alexander            Hunold                             0
Hermann              Baer                               0
John                 Russell                            0
Steven               King                               0
Nancy                Greenberg                          0
Shelley              Higgins                            0
Kimberely            Grant                              0

12 rows selected.
(...)
````
You can also verify that the same data is not redacted for **SYSTEM**.

This completes the **Data Redaction** lab. You can continue with **Lab 5: Privilege Analysis**

## Acknowledgements

- **Authors** - Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - May 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.
