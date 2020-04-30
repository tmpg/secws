# Lab 5 - Discover and Mask Sensitive Data   #

## Objectives

*	View sensitive data in your Autonomous Transaction Processing (ATP) database
*	Discover sensitive data by using Data Discovery
*	Mask sensitive data by using Data Masking
*	Validate the masked data in your ATP database

## Disclaimer ##

The following is intended to outline our general product direction. It is intended for information purposes only, and may not be incorporated into any contract. It is not a commitment to deliver any material, code, or functionality, and should not be relied upon in making purchasing decisions. The development, release, and timing of any features or functionality described for Oracle’s products remains at the sole discretion of Oracle.

## Requirements ##

To complete this lab, you need to have the following:
*	Login credentials and a tenancy name for the Oracle Cloud Infrastructure Console
*	A compartment enabled with permission to create and use resources
*	The ATP-S instance provisioned in Lab 1

## STEP 1: Discover sensitive data by using Data Discovery

The Data Discovery wizard generates a sensitive data model that contains sensitive columns in your target database. When working in the wizard, you select sensitive types that you want to discover in your target database.
From the OCI navigation menu, select Data Safe.

<!--![alt text](./images/img01.png " ")-->
<img src="./images/img01.png" alt="picture" width="250" height="400" />

Click Service Console.

![alt text](./images/img02.png " ")

Sign in to Oracle Data Safe using your Oracle Cloud Infrastructure credentials.
The **Home** tab is displayed when you first sign in to Oracle Data Safe.
Access the Data Discovery wizard by clicking the **Data Discovery** tab.

<!--![alt text](./images/img03.png " ")-->
<img src="./images/img03.png" alt="picture" width="250" height="400" />

On the Select Target for Sensitive Data Discovery page, select your target database, and then click Continue.

![alt text](./images/img04.png " ")

On the Select **Sensitive Data Model** page, leave **Create** selected, enter **SDM_HCM** for the name, enable **Show and save sample data**, select the default Resource Group, and then click **Continue**.

![alt text](./images/img05.png " ")

On the **Select Schemas for Sensitive Data Discovery** page, scroll down and select the **HCM** schema, and then click **Continue**.

![alt text](./images/img06.png " ")

On the **Select Sensitive Types for Sensitive Data Discovery** page, expand all of the categories by moving the slider to the right, and then scroll down the page and review the sensitive types. Notice that you can select individual sensitive types, sensitive categories, and all sensitive types.

At the top of the page, select the **Select All** check box, and then click **Continue** to start the data discovery job.

![alt text](./images/img07.png " ")

When the job is completed, ensure that the **Detail** column states Data discovery job finished successfully, and then click **Continue**.

![alt text](./images/img08.png " ")

On the **Sensitive Data Discovery Result** page, examine the sensitive data model created by the Data Discovery wizard. To view all of the sensitive columns, move the **Expand All** slider to the right.
Oracle Data Safe automatically saves your sensitive data model to the Oracle Data Safe Library.

![alt text](./images/img09.png " ")

From the drop-down list, select **Schema View** to sort the sensitive columns by table.

![alt text](./images/img10.png " ")

Scroll down the page to view the sensitive columns.
You can view sample data (if it's available for a sensitive column), column counts, and estimated data counts.
In particular, take a look at the sensitive columns that Data Discovery found in the **EMPLOYEES** table. Columns that do not have a check mark are called referential relationships. They are included because they have a relationship to another sensitive column and that relationship is defined in the database's data dictionary.
Also view the sample data provided to get an idea of what the sensitive data looks like.

![alt text](./images/img11.png " ")

Scroll to the bottom of the page, and then click **Report** to view the Data Discovery report.
The chart compares sensitive categories. You can view totals of sensitive values, sensitive types, sensitive tables, and sensitive columns.
The table displays individual sensitive column names, sample data for the sensitive columns, column counts based on sensitive categories, and estimated data counts.

![alt text](./images/img12.png " ")

Click the chart's Expand button.

![alt text](./images/img13.png " ")

Position your mouse over **Identification Info** to view statistics.

![alt text](./images/img14.png " ")

With your mouse still over **Identification Info**, click the **Expand** button to drill down.

![alt text](./images/img15.png " ")

Notice that the **Identification** Info category is divided into two smaller categories (**Personal IDs** and **Public IDs**). To drill-up, position your mouse over an expanded sensitive category, and then click the **Collapse** button.

![alt text](./images/img16.png " ")

Click the **Close** button (**X**) to close the expanded chart.

![alt text](./images/img17.png " ")

## STEP 2: Mask sensitive data by using Data Masking

The Data Masking wizard generates a masking policy for your target database based on a sensitive data model. In the wizard, you select the sensitive columns that you want to mask and the masking formats to use.
Click **Exit**.

Now select **Data Masking** from the **Home** menu. Select your database and click **Continue**.

![alt text](./images/img18.png " ")

Choose to create a new masking policy:

*	Making policy : **Create**
*	Masking policy name : **MASK_HCM**
*	Sensitive Data Modile : **Pick from Library**
*	Resource Group : **Default Resource Group**

![alt text](./images/img19.png " ")

On the **Select Sensitive Data Model** page, select **SDM_HCM** and make sure “Update the SDM with the target”  is selected. Click **Continue**.

![alt text](./images/img20.png " ")

On the **Sensitive Data Model: SDM_HCM** page, click on View Sensitive Columns

![alt text](./images/img21.png " ")

Make sure all sensitive columns are selected and click **Save and Continue**.

![alt text](./images/img22.png " ")

Move the **Expand All** slider to the right to view all of the sensitive columns. Scroll down the page to view the default masking format selected for each sensitive column.

For instance, **FIRST_NAME** will be masked by selecting a **Random Name**

Click the arrow to the right of the masking format to view other available masking formats.

![alt text](./images/img25.png " ")

Next to the arrow, click the Edit Format button (pencil icon).

![alt text](./images/img26.png " ")

In the **Edit Format** dialog box, view the description, examples, and default configuration for the masking format.
This is where you can modify a masking format.

Click **Cancel**.

At the bottom of the page, click **Confirm Policy**, and then wait a moment while Data Masking creates the masking policy.

![alt text](./images/img27.png " ")

On the **Schedule** the Masking Job page, leave **Right Now** selected, and then click **Review**.

![alt text](./images/img28.png " ")

On the **Review and Submit** page, review the information, and then click **Submit** to start the data masking job.

![alt text](./images/img29.png " ")

Wait for the data masking job to finish. It takes a couple of minutes. You can follow the status of the job on the **Masking Jobs** page. When the job is completed successfully, click **Report**.

![alt text](./images/img30.png " ")

Examine the **Data Masking** report.

The report shows you how many values, sensitive types, tables, and columns were masked.
For each sensitive column, the report shows you a sample masked value (if available), the masking format used, and the number of rows masked.

The table also shows you column counts for the sensitive categories and types.

![alt text](./images/img31.png " ")

Click Generate Report.

![alt text](./images/img32.png " ")

In the **Generate Report** dialog box, leave **PDF** selected, enter **HCM** Masking  for the description, ensure the default resource group is selected, and then click **Generate Report**.

![alt text](./images/img33.png " ")

Wait for the report to generate. When it's generated, click Download Report.
Review the report, and then close it.

![alt text](./images/img34.png " ")

## STEP 3: View masked data in SQL Developer

Now you can use SQL Developer again yo view the anonymized data in the HCM schema.

![alt text](./images/img35.png " ")

End of Lab 5.


## Acknowledgements ##

- **Authors** - Adrian Galindo & François Pons, PTS EMEA - April 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.
