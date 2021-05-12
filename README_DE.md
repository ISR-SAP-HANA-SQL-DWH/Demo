# Herzlich Willkommen im Git Repository zum Buch »SQL Data Warehousing mit SAP HANA«!

## Demo SAP SQL DWH
In diesem Git haben wir das im Buch beispielhaft entwickelte SAP SQL DWH voll funktionstüchtig für Sie abgelegt. Mithilfe von SAP Web IDE sind Sie so in der Lage in wenigen Klicks die Beispiel-Anwendung als Klon in Ihren Workspace herunterzuladen und das DWH mit all seinen Schichten auf Ihrer SAP HANA zu bauen und umzusetzen. 
Ein Grundstock an Beispieldaten ist dazu im Stage-Layer in Form von CSV-Dateien integriert, so dass Sie auf Ebene der Calculation Views unmittelbar Daten visualisieren können. 

Die Anwendung steht Ihnen zur freien Verfügung. Sie können den Beschreibungen im Buch entsprechend auch eigene Datenquellen anbinden und das SAP SQL DWH nach Ihren Vorstellungen verändern. 

Im Buch haben wir die Entwicklung in SAP Web IDE in Kapitel 9 beschrieben. 

## PowerDesigner-Modelle
Neben der DWH-Anwendung, in den branches master, release und development, haben wir auch die in SAP PowerDesigner modellierten Datenmodelle für Sie abgelegt. Diese finden Sie im branch PowerDesigner im Ordner Demo. Hier können Sie sich mit der Datei Demo.prj das zusammenhängende PowerDesigner-Projekt in Ihren PowerDesigner-Workspace laden. 

Mit der Datei S4HANA.xlsx, die Sie im branch PowerDesigner finden, stellen wir Ihnen auch die Metadaten der Tabellen des von uns in unserem Beispielfall verwendeten S/4HANA-Systems zur Verfügung. Hierdurch sind Sie in der Lage das beschriebene Reverse Engineering auch selbst nachzuvollziehen. 

Sie können die Modelle selbstverständlich individuell anpassen und eigene Varianten exportieren und in SAP Web IDE importieren. 

Im Buch finden Sie alles Aspekte zur Modellierung in SAP PowerDesigner in den Kapiteln 7 und 8. 

## Jenkinsfile
Mit der Jenkinsfile bieten wir Ihnen zudem die Möglichkeit, das Demo SAP SQL DWH über dieses Git-Repository automatisch in einem von Ihnen definierten SAP HANA Space zu deployen. Im Buch haben wir diese Thematik in Kapitel 10 beschriebe. Zur praktischen Umsetzung müssen Sie nur folgende Dinge tun:

### Aufbau eines HANA-Datenbankusers
Um mit Jenkins eine automatisierte Deployment Pipeline aufzubauen, die den XSA-Client nutzt um die im Git definierte DWH-Applikation in einem SAP HANA Space umzusetzen, brauchen Sie zunächst einen Datenbank- und einen XSA-User, der über die nötigen Berechtigungen verfügt. 

Nachfolgend finden Sie ein Skript mit SQL-Befehlen, die den User **TU_CICD** und **XSA_CICD** erstellen. Dieser wird gleichzeitig zum XSA-User und erhält die nötigen XSA-Berechtigungen. Das Skript teilt sich in die Teile vor und nach dem Deployment. Mit den Befehlen nach dem Deplyoment erhalten Sie Zugriff auf alle DWH-Objekte in den verschiedenen Schichten und können den User TU_CICD nicht nur für das automatische Deployment mit Jenkins, sondern auch für Analysen mit SAP Analytics Cloud nutzen. 

Sie müssen das Skript dazu nur mit einem umfassend berechtigten Datenbankuser, wie **SYSTEM** über die SQL Konsole im SAP HANA Database Explorer oder SAP HANA Studio zu den genanten Zeitpunkten auf dem Tenant, auf dem Sie das Demo-DWH deployen wollen, das Skript ausführen.

```sql
-- before deployment

-- create hana-user
CREATE USER TU_CICD PASSWORD "Password";
ALTER USER TU_CICD DISABLE PASSWORD LIFETIME;

-- create xsa-user out of hana-user with required priviliges
CREATE USER XSA_CICD PASSWORD "Password";
ALTER USER XSA_CICD DISABLE PASSWORD LIFETIME;
SET PARAMETER 'XS_RC_XS_CONTROLLER_USER' = 'XS_CONTROLLER_USER',
    'XS_RC_XS_USER_PUBLIC'='XS_USER_PUBLIC',
    'XS_RC_XS_CONTROLLER_USER'='XS_CONTROLLER_USER'

-- create roles
CREATE ROLE HANA_ACCESS;
GRANT MODELING TO DWH_ACCESS;
GRANT MONITORING TO DWH_ACCESS;
GRANT sap.hana.im.api.roles::Execute TO DWH_ACCESS;

CREATE ROLE SAC_ACCESS;
GRANT sap.bc.ina.service.v2.userRole::INA_USER TO SAC_ACCESS;
GRANT sap.hana.im.dp.monitor.roles::Monitoring TO SAC_ACCESS;
GRANT sap.hana.im.dp.monitor.roles::Operations TO SAC_ACCESS;

-- grant roles to hana-user
GRANT HANA_ACCESS TO TU_CICD;
GRANT SAC_ACCESS TO TU_CICD;


-- after dwh-deployment

-- create role
CREATE ROLE DWH_ACCESS;
GRANT DW_STAGE::access_role TO DWH_ACCESS;
GRANT DW_CORE::access_role TO DWH_ACCESS;
GRANT DW_VAL::access_role TO DWH_ACCESS;
GRANT DW_DM::access_role TO DWH_ACCESS;
GRANT CV_DM_SALES_ORDER_WGO TO DWH_ACCESS;

-- grant roles to hana-user
GRANT DWH_ACCESS TO TU_CICD;
```

### Jenkins Konfiguration
Sie müssen in Jenkins die User TU_CICD und XSA_CICD erstellen. Sie müssen eine Multibranch Pipeline erstellen. Für 

### Installation XSA-Client
Sie können den **XSA-Client** im SAP ONE Support Launchpad unter https://launchpad.support.sap.com/#/softwarecenter/search/xsa%2520client für verschiedene Betriebssysteme herunterladen. Den Client installieren Sie auf dem Server, auf dem Sie auch Jenkins installiert haben. In der Jenkinsfile müssen Sie später Angaben zu dem Verzeichnis machen, in dem der XSA-Client installiert ist. 

### Installation MBT
Sie benötigen das **MTA Build Tool (MBT)**, das Sie unter https://sap.github.io/cloud-mta-build-tool/download/ herunterladen können. Auch das MTB installieren Sie auf dem Jenkins Server und merken sich das Installationsverzeichnis für die Jenkinsfile.

### Installation nvm
Des Weiteren benötigen Sie noch den Versionsmanager für node.js **nvm**, den Sie unter https://github.com/nvm-sh/nvm bekommen. nvm installieren Sie ebenfalls auf dem Jenkins Server.

### Anpassungen Jenkinsfile
In der Jenkinsfile müssen Sie alle durch Guillemets **<<>>** gekennzeichneten Parameter noch einfügen. Der Block zur Testautomation ist bereits enthalten. Wenn Sie also zunächst nur ein Deployment, ohne Test durchführen möchten, müssen Sie diesen Block entfernen. 

## Testautomation
In der Jenkinsfile ist bereits ein kleiner Test definiert, der Namenskonventionen des Schemas DW_VAL überprüft. Das entsprechende Testmodul **Demo_TA** liegt in dem separaten Git Repository https://github.com/ISR-SAP-HANA-SQL-DWH/Demo_TA. Um den Test aus dem Repository, so wie wir ihn definiert haben auszuführen, müssen Sie lediglich in der Jenkinsfile im Block **'Test'** die durch Guillemets **<<>>** gekennzeichneten Parameter anpassen. 

