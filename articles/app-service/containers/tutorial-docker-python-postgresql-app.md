---
title: Erstellen einer Docker Python- und PostgreSQL-Web-App in Azure | Microsoft-Dokumentation
description: Erfahren Sie etwas über das Ausführen einer Docker Python-App in Azure mit Verbindung mit einer PostgreSQL-Datenbank.
services: app-service\web
documentationcenter: python
author: berndverst
manager: cfowler
ms.service: app-service-web
ms.workload: web
ms.devlang: python
ms.topic: tutorial
ms.date: 01/28/2018
ms.author: beverst;cephalin
ms.custom: mvc
ms.openlocfilehash: 2728c354a84c4b13b0ad8509d038837733251975
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/23/2018
---
# <a name="build-a-docker-python-and-postgresql-web-app-in-azure"></a>Erstellen einer Docker Python- und PostgreSQL-Web-App in Azure

Web-App für Container bietet einen hochgradig skalierbaren Webhostingdienst mit Self-Patching. In diesem Tutorial wird gezeigt, wie eine einfache Docker Python-Web-App in Azure erstellt wird. Sie verbinden diese App zudem mit einer PostgreSQL-Datenbank. Wenn Sie diese Schritte ausgeführt haben, verfügen Sie über eine Python Flask-Anwendung, die in einem Docker-Container in [App Service unter Linux](app-service-linux-intro.md) ausgeführt wird.

![Docker Python Flask-App in App Service unter Linux](./media/tutorial-docker-python-postgresql-app/docker-flask-in-azure.png)

In diesem Tutorial lernen Sie Folgendes:

> [!div class="checklist"]
> * Erstellen einer PostgreSQL-Datenbank in Azure
> * Herstellen einer Verbindung mit PostgreSQL für eine Python-App
> * Bereitstellen der Anwendung in Azure
> * Aktualisieren des Datenmodells und erneutes Bereitstellen der App
> * Verwalten der App im Azure-Portal

Sie können die Schritte in diesem Artikel auch unter macOS ausführen. Die Anweisungen für Linux und Windows sind in den meisten Fällen identisch, doch die Unterschiede werden in diesem Tutorial nicht beschrieben.
 
[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Voraussetzungen

Für dieses Tutorial benötigen Sie Folgendes:

1. [Installation von Git](https://git-scm.com/)
1. [Installieren Sie Python.](https://www.python.org/downloads/)
1. [Laden Sie PostgreSQL herunter, und führen Sie es aus.](https://www.postgresql.org/download/)
1. [Installieren Sie Docker Community Edition.](https://www.docker.com/community-edition)

## <a name="test-local-postgresql-installation-and-create-a-database"></a>Testen der lokalen PostgreSQL-Installation und Erstellen einer Datenbank

Öffnen Sie das Terminalfenster, und führen Sie `psql` aus, um eine Verbindung mit Ihrem lokalen PostgreSQL-Server herzustellen.

```bash
sudo -u postgres psql
```

Wenn die Verbindung erfolgreich ist, wird die PostgreSQL-Datenbank ausgeführt. Ist dies nicht der Fall, stellen Sie sicher, dass die lokale PostgreSQL-Datenbank gestartet wurde, indem Sie die Schritte unter [Downloads: PostgreSQL Core Distribution](https://www.postgresql.org/download/) ausführen.

Erstellen Sie eine Datenbank mit dem Namen *eventregistration*, und richten Sie einen separaten Datenbankbenutzer namens *manager* mit Kennwort *supersecretpass* ein.

```bash
CREATE DATABASE eventregistration;
CREATE USER manager WITH PASSWORD 'supersecretpass';
GRANT ALL PRIVILEGES ON DATABASE eventregistration TO manager;
```
Geben Sie `\q` ein, um den PostgreSQL-Client zu beenden. 

<a name="step2"></a>

## <a name="create-local-python-flask-application"></a>Erstellen der lokalen Python Flask-Anwendung

In diesem Schritt richten Sie das lokale Python Flask-Projekt ein.

### <a name="clone-the-sample-application"></a>Klonen der Beispielanwendung

Öffnen Sie das Terminalfenster, und wechseln Sie mit `CD` in ein Arbeitsverzeichnis.

Führen Sie die folgenden Befehle aus, um das Beispielrepository zu klonen und zum Release *0.1-initialapp* zu wechseln.

```bash
git clone https://github.com/Azure-Samples/docker-flask-postgres.git
cd docker-flask-postgres
git checkout tags/0.1-initialapp
```

Dieses Beispielrepository enthält eine [Flask](http://flask.pocoo.org/)-Anwendung. 

### <a name="run-the-application"></a>Ausführen der Anwendung

> [!NOTE] 
> Wir werden diesen Prozess in einem späteren Schritt vereinfachen, indem wir einen Docker-Container für die Produktionsdatenbank erstellen.

Installieren Sie die erforderlichen Pakete, und starten Sie die Anwendung.

```bash
pip install virtualenv
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
cd app
FLASK_APP=app.py DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
FLASK_APP=app.py DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask run
```

Wenn die App vollständig geladen wurde, sollte eine ähnliche Meldung wie diese angezeigt werden:

```bash
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 791cd7d80402, empty message
 * Serving Flask app "app"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Navigieren Sie in einem Browser zu `http://localhost:5000`. Klicken Sie auf **Registrieren**, und erstellen Sie einen Testbenutzer.

![Lokal ausgeführte Python Flask-Anwendung](./media/tutorial-docker-python-postgresql-app/local-app.png)

Die Flask-Beispielanwendung speichert Benutzerdaten in der Datenbank. Wenn Sie einen Benutzer erfolgreich registriert haben, schreibt Ihre App Daten in die lokale PostgreSQL-Datenbank.

Sie können den Flask-Server jederzeit beenden, indem Sie im Terminal STRG+C eingeben. 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-production-postgresql-database"></a>Erstellen einer PostgreSQL-Produktionsdatenbank

In diesem Schritt erstellen Sie eine PostgreSQL-Datenbank in Azure. Wenn Ihre App in Azure bereitgestellt ist, nutzt sie diese Clouddatenbank.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-a-resource-group"></a>Erstellen einer Ressourcengruppe

[!INCLUDE [Create resource group](../../../includes/app-service-web-create-resource-group-linux-no-h.md)] 

### <a name="create-an-azure-database-for-postgresql-server"></a>Erstellen einer Azure-Datenbank für PostgreSQL-Server

Erstellen Sie mit dem Befehl [`az postgres server create`](/cli/azure/postgres/server?view=azure-cli-latest#az_postgres_server_create) einen PostgreSQL-Server.

Ersetzen Sie im folgenden Befehl den Platzhalter *\<postgresql_name>* durch einen eindeutigen Servernamen und den Platzhalter *\<admin_username>* durch einen Benutzernamen. Der Servername dient als Teil Ihrer PostgreSQL-Endpunkts (`https://<postgresql_name>.postgres.database.azure.com`). Daher muss der Name auf allen Servern in Azure eindeutig sein. Der Benutzername ist erforderlich, um das anfängliche Konto des Administratorbenutzers für die Datenbank zu erstellen. Sie werden aufgefordert, ein Kennwort für diesen Benutzer auszuwählen.

```azurecli-interactive
az postgres server create --resource-group myResourceGroup --name <postgresql_name> --admin-user <admin_username>  --storage-size 51200
```

Nach dem Erstellen der Azure-Datenbank für den PostgreSQL-Server zeigt die Azure-Befehlszeilenschnittstelle Informationen wie im folgenden Beispiel an:

```json
{
  "administratorLogin": "<my_admin_username>",
  "fullyQualifiedDomainName": "<postgresql_name>.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforPostgreSQL/servers/<postgresql_name>",
  "location": "westus",
  "name": "<postgresql_name>",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "capacity": 100,
    "family": null,
    "name": "PGSQLS3M100",
    "size": null,
    "tier": "Basic"
  },
  "sslEnforcement": null,
  "storageMb": 2048,
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": null
}
```

### <a name="create-a-firewall-rule-for-the-azure-database-for-postgresql-server"></a>Erstellen einer Firewallregel für die Azure-Datenbank für den PostgreSQL-Server

Führen Sie den folgenden Azure CLI-Befehl aus, um den Zugriff auf die Datenbank von allen IP-Adressen zu ermöglichen. Wenn sowohl Start- als auch End-IP auf 0.0.0.0 festgelegt ist, wird die Firewall nur für andere Azure-Ressourcen geöffnet. 

```azurecli-interactive
az postgres server firewall-rule create --resource-group myResourceGroup --server-name <postgresql_name> --start-ip-address=0.0.0.0 --end-ip-address=0.0.0.0 --name AllowAzureIPs
```

Die Azure CLI bestätigt das Erstellen der Firewallregel mit mithilfe einer Ausgabe ähnlich wie im folgenden Beispiel:

```json
{
  "endIpAddress": "0.0.0.0",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforPostgreSQL/servers/<postgresql_name>/firewallRules/AllowAzureIPs",
  "name": "AllowAzureIPs",
  "resourceGroup": "myResourceGroup",
  "startIpAddress": "0.0.0.0",
  "type": "Microsoft.DBforPostgreSQL/servers/firewallRules"
}
```

> [!TIP] 
> Sie können Ihre Firewallregel auch noch restriktiver gestalten und [nur die ausgehenden IP-Adressen verwenden, die Ihre App verwendet](../app-service-ip-addresses.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#find-outbound-ips).
>

## <a name="connect-your-python-flask-application-to-the-database"></a>Verbinden der Python Flask-Anwendung mit der Datenbank

In diesem Schritt verbinden Sie Ihre Python Flask-Beispielanwendung mit der Azure-Datenbank für den PostgreSQL-Server, den Sie erstellt haben.

### <a name="create-an-empty-database-and-set-up-a-new-database-application-user"></a>Erstellen einer leeren Datenbank und Einrichten eines neuen Datenbankanwendungsbenutzers

Erstellen Sie einen neuen Datenbankbenutzer mit Zugriff auf nur eine einzelne Datenbank. Sie verwenden diese Anmeldeinformationen, um zu vermeiden, dass der Anwendung Vollzugriff auf den Server gewährt wird.

Stellen Sie eine Verbindung mit der Datenbank her (Sie werden zur Eingabe Ihres Administratorkennworts aufgefordert).

```bash
psql -h <postgresql_name>.postgres.database.azure.com -U <my_admin_username>@<postgresql_name> postgres
```

Erstellen Sie die Datenbank und den Benutzer über die PostgreSQL CLI.

```bash
CREATE DATABASE eventregistration;
CREATE USER manager WITH PASSWORD 'supersecretpass';
GRANT ALL PRIVILEGES ON DATABASE eventregistration TO manager;
```

Geben Sie `\q` ein, um den PostgreSQL-Client zu beenden.

### <a name="test-the-application-locally-against-the-azure-postgresql-database"></a>Lokales Testen der Anwendung mit der Azure-PostgreSQL-Datenbank

Nach dem Wechseln zurück zum Ordner *app* des geklonten GitHub-Repositorys können Sie die Python Flask-Anwendung ausführen, indem Sie die Umgebungsvariablen der Datenbank aktualisieren.

```bash
FLASK_APP=app.py DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
FLASK_APP=app.py DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBNAME="eventregistration" DBPASS="supersecretpass" flask run
```

Wenn die App vollständig geladen wurde, sollte eine ähnliche Meldung wie diese angezeigt werden:

```bash
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 791cd7d80402, empty message
 * Serving Flask app "app"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

Navigieren Sie in einem Browser zu http://localhost:5000. Klicken Sie auf **Registrieren**, und erstellen Sie eine Testregistrierung. Sie schreiben nun Daten in die Datenbank in Azure.

![Lokal ausgeführte Python Flask-Anwendung](./media/tutorial-docker-python-postgresql-app/local-app.png)

### <a name="running-the-application-from-a-docker-container"></a>Ausführen der Anwendung aus einem Docker-Container

Erstellen Sie das Docker-Containerimage.

```bash
cd ..
docker build -t flask-postgresql-sample .
```

Docker zeigt eine Bestätigung der Erstellung des Containers an.

```bash
Successfully built 7548f983a36b
```

Fügen Sie am Repositorystamm eine Umgebungsvariablendatei namens _db.env_ und dieser Datei anschließend folgende Umgebungsvariablen für die Datenbank hinzu. Die App stellt eine Verbindung mit der Azure Database for PostgreSQL-Produktionsdatenbank.

```text
DBHOST=<postgresql_name>.postgres.database.azure.com
DBUSER=manager@<postgresql_name>
DBNAME=eventregistration
DBPASS=supersecretpass
```

Führen Sie die App im Docker-Container aus. Der folgende Befehl gibt die Umgebungsvariablendatei an und ordnet den Flask-Standardport 5.000 dem lokalen Port 5.000 zu.

```bash
docker run -it --env-file db.env -p 5000:5000 flask-postgresql-sample
```

Die Ausgabe ähnelt der weiter oben aufgeführten. Allerdings muss die anfängliche Datenbankmigration nicht mehr ausgeführt werden und wird daher übersprungen.

```bash
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
 * Serving Flask app "app"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Die Datenbank enthält bereits die Registrierung, die Sie zuvor erstellt haben.

![Lokal ausgeführte Python Flask-Anwendung basierend auf einem Docker-Container](./media/tutorial-docker-python-postgresql-app/local-docker.png)

## <a name="upload-the-docker-container-to-a-container-registry"></a>Hochladen des Docker-Containers in eine Containerregistrierung

In diesem Schritt laden Sie den Docker-Container in eine Containerregistrierung hoch. Sie verwenden hier Azure Container Registry, können aber auch andere verbreitete Lösungen wie z.B. Docker Hub verwenden.

### <a name="create-an-azure-container-registry"></a>Erstellen einer Azure-Containerregistrierung

Ersetzen Sie im folgenden Befehl *\<registry_name>* durch einen eindeutigen Azure-Containerregistrierungsnamen Ihrer Wahl, um eine Containerregistrierung zu erstellen.

```azurecli-interactive
az acr create --name <registry_name> --resource-group myResourceGroup --location "West US" --sku Basic
```

Output

```json
{
  "adminUserEnabled": false,
  "creationDate": "2017-05-04T08:50:55.635688+00:00",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/<registry_name>",
  "location": "westus",
  "loginServer": "<registry_name>.azurecr.io",
  "name": "<registry_name>",
  "provisioningState": "Succeeded",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "storageAccount": {
    "name": "<registry_name>01234"
  },
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

### <a name="retrieve-the-registry-credentials-for-pushing-and-pulling-docker-images"></a>Abrufen der Anmeldeinformationen für die Registrierung für Push- und Pullvorgänge bei Docker-Images

Aktivieren Sie zum Anzeigen der Registrierungsanmeldeinformationen zunächst den Administratormodus.

```azurecli-interactive
az acr update --name <registry_name> --admin-enabled true
az acr credential show -n <registry_name>
```

Sie finden zwei Kennwörter vor. Notieren Sie sich den Benutzernamen und das erste Kennwort.

```json
{
  "passwords": [
    {
      "name": "password",
      "value": "<registry_password>"
    },
    {
      "name": "password2",
      "value": "<registry_password2>"
    }
  ],
  "username": "<registry_name>"
}
```

### <a name="upload-your-docker-container-to-azure-container-registry"></a>Hochladen des Docker-Containers in Azure Container Registry

Melden Sie sich bei Ihrer Registrierung an. Geben Sie das abgerufene Kennwort an, wenn Sie dazu aufgefordert werden.

```bash
docker login <registry_name>.azurecr.io -u <registry_name>
```

Übertragen Sie Ihr Docker-Image mithilfe von Push in die Registrierung.

```bash
docker tag flask-postgresql-sample <registry_name>.azurecr.io/flask-postgresql-sample
docker push <registry_name>.azurecr.io/flask-postgresql-sample
```

## <a name="deploy-the-docker-python-flask-application-to-azure"></a>Bereitstellen der Docker Python Flask-Anwendung in Azure

In diesem Schritt stellen Sie die auf einem Docker Container basierende Python Flask-Anwendung in Azure App Service bereit.

### <a name="create-an-app-service-plan"></a>Wie erstelle ich einen Plan?

[!INCLUDE [Create app service plan](../../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

### <a name="create-a-web-app"></a>Erstellen einer Web-App

Erstellen Sie mit dem Befehl [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az_webapp_create) eine Web-App im App Service-Plan *myAppServicePlan*.

Die Web-App bietet Ihnen eine Hostingumgebung zum Bereitstellen Ihres Codes sowie eine URL, unter der Sie sich die bereitgestellte Anwendung ansehen können. Erstellen Sie damit die Web-App.

Ersetzen Sie im folgenden Befehl den Platzhalter *\<app_name>* durch einen eindeutigen App-Namen. Dieser Name ist Teil der Standard-URL der Web-App. Daher muss er für alle Apps in Azure App Service eindeutig sein.

```azurecli
az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan --deployment-container-image-name "<registry_name>.azurecr.io/flask-postgresql-sample"
```

Nach dem Erstellen der Web-App zeigt die Azure-Befehlszeilenschnittstelle Informationen wie im folgenden Beispiel an:

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  ...
  < Output has been truncated for readability >
}
```

### <a name="configure-the-database-environment-variables"></a>Konfigurieren der Umgebungsvariablen für die Datenbank

Weiter oben in diesem Tutorial haben Sie die Umgebungsvariablen für die Verbindung mit der PostgreSQL-Datenbank definiert.

Legen Sie in App Service mit dem Befehl [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az_webapp_config_appsettings_set) Umgebungsvariablen als _App-Einstellungen_ fest.

Im folgenden Beispiel werden die Details der Datenbankverbindung als App-Einstellungen angeben. Darüber hinaus wird die Variable *PORT* zum Zuordnen von PORT 5000 aus dem Docker-Container zum Empfangen von HTTP-Datenverkehr über PORT 80 verwendet.

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group myResourceGroup --settings DBHOST="<postgresql_name>.postgres.database.azure.com" DBUSER="manager@<postgresql_name>" DBPASS="supersecretpass" DBNAME="eventregistration" PORT=5000
```

### <a name="configure-docker-container-deployment"></a>Konfigurieren der Docker-Containerbereitstellung

AppService kann einen Docker-Container automatisch herunterladen und ausführen.

```azurecli
az webapp config container set --resource-group myResourceGroup --name <app_name> --docker-registry-server-user "<registry_name>" --docker-registry-server-password "<registry_password>" --docker-registry-server-url "https://<registry_name>.azurecr.io"
```

Nachdem Sie den Docker-Container aktualisiert oder die Einstellungen geändert haben, starten Sie die App neu. Durch den Neustart wird sichergestellt, dass alle Einstellungen übernommen werden und der neueste Container per Pull aus der Registrierung abgerufen wird.

```azurecli-interactive
az webapp restart --resource-group myResourceGroup --name <app_name>
```

### <a name="browse-to-the-azure-web-app"></a>Navigieren Sie zur Azure-Web-App. 

Navigieren Sie in Ihrem Webbrowser zu der bereitgestellten Web-App. 

```bash 
http://<app_name>.azurewebsites.net 
```
> [!NOTE]
> Das Laden der Web-App dauer länger, da der Container heruntergeladen und nach der Änderungen der Containerkonfiguration gestartet werden muss.

Es werden die zuvor registrierten Gäste angezeigt, die im vorherigen Schritt in der Azure-Produktionsdatenbank gespeichert wurden.

![Lokal ausgeführte Python Flask-Anwendung basierend auf einem Docker-Container](./media/tutorial-docker-python-postgresql-app/docker-app-deployed.png)

**Glückwunsch!** Sie führen damit eine auf einem Docker-Container basierende Python Flask-App in Azure App Service aus.

## <a name="update-data-model-and-redeploy"></a>Aktualisieren und erneutes Bereitstellen des Datenmodells

In diesem Schritt fügen Sie die Anzahl der Teilnehmer für jede Ereignisregistrierung hinzu, indem wir das Gastmodell aktualisieren.

Checken Sie das Release *0.2-migration* mit dem folgenden Git-Befehl aus:

```bash
git checkout tags/0.2-migration
```

Dieses Release enthält bereits die erforderlichen Änderungen an Sichten, Controllern und Modell. Es enthält auch eine Datenbankmigration, die über *alembic* (`flask db migrate`) generiert wurde. Sie können alle vorgenommenen Änderungen über den folgenden Git-Befehl anzeigen:

```bash
git diff 0.1-initialapp 0.2-migration
```

### <a name="test-your-changes-locally"></a>Lokales Testen der Änderungen

Führen Sie die folgenden Befehle aus, um Ihre Änderungen lokal zu testen, indem Sie den Flask-Server ausführen.

```bash
source venv/bin/activate
cd app
FLASK_APP=app.py DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask db upgrade
FLASK_APP=app.py DBHOST="localhost" DBUSER="manager" DBNAME="eventregistration" DBPASS="supersecretpass" flask run
```

Navigieren Sie in Ihrem Browser zu http://localhost:5000, um die Änderungen anzuzeigen. Erstellen Sie eine Testregistrierung.

![Lokal ausgeführte Python Flask-Anwendung basierend auf einem Docker-Container](./media/tutorial-docker-python-postgresql-app/local-app-v2.png)

### <a name="publish-changes-to-azure"></a>Veröffentlichen von Änderungen in Azure

Erstellen Sie das neue Docker-Image, übermitteln Sie es an die Containerregistrierung, und starten Sie die App neu.

```bash
cd ..
docker build -t flask-postgresql-sample .
docker tag flask-postgresql-sample <registry_name>.azurecr.io/flask-postgresql-sample
docker push <registry_name>.azurecr.io/flask-postgresql-sample
az appservice web restart --resource-group myResourceGroup --name <app_name>
```

Wechseln Sie zu Ihrer Azure-Web-App, und testen Sie die neuen Funktionen erneut. Erstellen Sie eine weitere Ereignisregistrierung.

```bash 
http://<app_name>.azurewebsites.net 
```

![Docker Python Flask-App in Azure App Service](./media/tutorial-docker-python-postgresql-app/docker-flask-in-azure.png)

## <a name="manage-your-azure-web-app"></a>Verwalten Ihrer Azure-Web-App

Wechseln Sie zum [Azure-Portal](https://portal.azure.com), um die erstellte Web-App anzuzeigen.

Klicken Sie im linken Menü auf **App Services** und anschließend auf den Namen Ihrer Azure-Web-App.

![Portalnavigation zur Azure-Web-App](./media/tutorial-docker-python-postgresql-app/app-resource.png)

Standardmäßig wird im Portal die Seite **Übersicht** Ihrer Web-App angezeigt. Diese Seite bietet einen Überblick über den Status Ihrer App. Hier können Sie auch einfache Verwaltungsaufgaben wie Durchsuchen, Beenden, Neustarten und Löschen durchführen. Die Registerkarten links auf der Seite zeigen die verschiedenen Konfigurationsseiten, die Sie öffnen können.

![App Service-Seite im Azure-Portal](./media/tutorial-docker-python-postgresql-app/app-mgmt.png)

## <a name="next-steps"></a>Nächste Schritte

Fahren Sie mit dem nächsten Tutorial fort, um zu erfahren, wie Sie Ihrer Web-App einen benutzerdefinierten DNS-Namen zuordnen.

> [!div class="nextstepaction"]
> [Zuordnen eines vorhandenen benutzerdefinierten DNS-Namens zu Azure-Web-Apps](../app-service-web-tutorial-custom-domain.md)
