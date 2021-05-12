# Welcome to the Git repository accompanying the book "SQL Data Warehousing mit SAP HANA"!

## Demo SAP SQL DWH
In this Git repository, we provide the fully functional SAP SQL DWH developed as an example in the book. With the help of SAP Web IDE, you can clone the demo application  into your workspace in just a few clicks and build and implement the DWH with all its layers on your SAP HANA.  

A basic set of **example data** is integrated in the Stage layer in form of CSV files, allowing you to immediately visualise data on the level of the Calculation Views. 

You are also welcome to introduce changes, suggestions for improvement and similar in a **fork** back into the Git and thereby ensure further development of our approach. 

In the book, we describe the development in SAP Web IDE in chapter 9. 

## PowerDesigner models
In addition to the DWH application, in the branches master, release and development, we have also provide the data models modelled in SAP PowerDesigner. You can find them in the PowerDesigner branch in the Demo folder. Here you can use the file **Demo.prj** to load the related PowerDesigner project into your PowerDesigner workspace. 

With the file **S4HANA.xlsx**, which you will find in the PowerDesigner branch, we supply you with the metadata of the tables of the S/4HANA system we used in our example case too. This enables you to reproduce the described reverse engineering yourself. 

It goes without saying that you can adapt the models individually and export your own variants and import them into SAP Web IDE. 

In the book, you will find all aspects of modelling in SAP PowerDesigner in chapters 7 and 8. 

## Jenkinsfile
With the Jenkinsfile, we also offer you the option of automatically deploying the demo SAP SQL DWH via this Git repository in a SAP HANA Space you define. In the book, we describe this topic in chapter 10. For practical implementation, you just need to do the following things:

### Set up a HANA database user and an XSA user.
To build an automated deployment pipeline with Jenkins that uses the XSA client to deploy the DWH application defined in Git to an SAP HANA Space and to run automated tests, you first need a HANA database user and an XSA user that have the required permissions. 

Below you will find a script with SQL commands that create the users **TU_CICD** and **XSA_CICD**. 

The script is divided into the parts **before** and **after** the DWH deployment. The commands before the deployment are necessary to create the users and give them the necessary privileges for deployment via XSA and into the SAP HANA database. With the commands after the deployment, you get access to all DWH objects in the different layers and can use the user TU_CICD not only for the automatic deployment and testing with Jenkins, but also for analyses with SAP Analytics Cloud.  

To do this, you only need to execute the script with a widely authorised database user such as **SYSTEM** via the SQL console in the SAP HANA Database Explorer or SAP HANA Studio on the tenant on which you want to deploy the demo DWH at the times mentioned.

```sql
-- before dwh deployment

-- create hana-user
CREATE USER TU_CICD PASSWORD "Password";
ALTER USER TU_CICD DISABLE PASSWORD LIFETIME;

-- create xsa-user out of hana-user with required priviliges
CREATE USER XSA_CICD PASSWORD "Password";
ALTER USER XSA_CICD DISABLE PASSWORD LIFETIME;
SET PARAMETER 'XS_RC_XS_CONTROLLER_USER' = 'XS_CONTROLLER_USER',
    'XS_RC_XS_USER_PUBLIC'='XS_USER_PUBLIC',
    'XS_RC_XS_CONTROLLER_USER'='XS_CONTROLLER_USER'

-- create role
CREATE ROLE HANA_ACCESS;
GRANT MODELING TO DWH_ACCESS;
GRANT MONITORING TO DWH_ACCESS;
GRANT sap.hana.im.api.roles::Execute TO DWH_ACCESS;

-- grant roles to hana-user
GRANT HANA_ACCESS TO TU_CICD;


-- after dwh deployment

-- create roles
CREATE ROLE DWH_ACCESS;
GRANT DW_STAGE::access_role TO DWH_ACCESS;
GRANT DW_CORE::access_role TO DWH_ACCESS;
GRANT DW_VAL::access_role TO DWH_ACCESS;
GRANT DW_DM::access_role TO DWH_ACCESS;
GRANT CV_DM_SALES_ORDER_WGO TO DWH_ACCESS;

CREATE ROLE SAC_ACCESS;
GRANT sap.bc.ina.service.v2.userRole::INA_USER TO SAC_ACCESS;
GRANT sap.hana.im.dp.monitor.roles::Monitoring TO SAC_ACCESS;
GRANT sap.hana.im.dp.monitor.roles::Operations TO SAC_ACCESS;

-- grant roles to hana-user
GRANT DWH_ACCESS TO TU_CICD;
GRANT SAC_ACCESS TO TU_CICD;
```

### Jenkins configuration
If you have not yet installed Jenkins, download the programme from https://www.jenkins.io/download/ and install it locally or on another server that you can use for this purpose. After the installation, save the users **TU_CICD** and **XSA_CICD** with password in the **Credentials** area in Jenkins, which you can access via the menu items *Manage Jenkins / Manage Credentials*. In addition, create a **Multibranch Pipeline** and define https://github.com/ISR-SAP-HANA-SQL-DWH/Demo as the **Project Repository**.  

### Installing the XSA Client
You can download the **XSA client** from the SAP ONE Support Launchpad at https://launchpad.support.sap.com/#/softwarecenter/search/xsa%2520client for various operating systems. You install the client on the server on which you also installed Jenkins. In the Jenkins file, you will later have to specify the directory in which you installed the XSA client. 

### Installing MBT
You need the **MTA Build Tool (MBT)**, which you can download from https://sap.github.io/cloud-mta-build-tool/download/. Install the MTB on the Jenkins server as well and note the installation directory for the Jenkins file.

### Installing nvm
You need the version manager for node.js **nvm**, which you can get at https://github.com/nvm-sh/nvm. You install nvm on the Jenkins server too.

### Adjustments to the Jenkinsfile
In the Jenkinsfile you have to add all parameters marked with guillemets **<<>>**. The block for test automation is already included. So, if you just want to run a deployment without testing, you have to remove this block. 

## Test automation
A small test is already defined in the Jenkinsfile that checks naming conventions of the schema DW_VAL. The corresponding test module **Demo_TA** is located in the separate Git repository https://github.com/ISR-SAP-HANA-SQL-DWH/Demo_TA. To run the test from the repository as we have defined it, you only need to adjust the parameters marked by Guillemets **<<>>** in the block **'Test'** in the Jenkinsfile. 

