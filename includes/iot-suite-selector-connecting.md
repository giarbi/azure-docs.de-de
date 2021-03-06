---
title: Includedatei
description: Includedatei
services: iot-suite
author: dominicbetts
ms.service: iot-suite
ms.topic: include
ms.date: 04/24/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 500e335d0b2eddc56cdfb9828236bc4676d9b6aa
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/20/2018
ms.locfileid: "34371166"
---
> [!div class="op_single_selector"]
> * [C unter Windows](../articles/iot-accelerators/iot-accelerators-connecting-devices.md)
> * [C unter Linux](../articles/iot-accelerators/iot-accelerators-connecting-devices-linux.md)
> * [Node.js (generisch)](../articles/iot-accelerators/iot-accelerators-connecting-devices-node.md)
> * [Node.js auf Raspberry Pi](../articles/iot-accelerators/iot-accelerators-connecting-pi-node.md)
> * [C auf Raspberry Pi](../articles/iot-accelerators/iot-accelerators-connecting-pi-c.md)

In diesem Tutorial implementieren Sie ein **Kühlgerät**, das die folgenden Telemetriedaten an den [Solution Accelerator](../articles/iot-accelerators/iot-accelerators-what-are-solution-accelerators.md) für die Remoteüberwachung sendet:

* Temperatur
* Pressure
* Luftfeuchtigkeit

Der Einfachheit halber generiert der Code Beispieltelemetriewerte für den **Kühler**. Sie können das Beispiel noch erweitern, indem Sie echte Sensoren an Ihr Gerät anschließen und echte Telemetriedaten senden.

Das Beispielgerät führt auch folgende Vorgänge durch:

* Sendet Metadaten an die Lösung, um die jeweiligen Funktionen zu beschreiben
* Antwortet auf Aktionen, die in der Lösung von der Seite **Geräte** ausgelöst wurden
* Antwortet auf Konfigurationsänderungen, die von der Seite **Geräte** in der Lösung gesendet werden

Um dieses Tutorial abzuschließen, benötigen Sie ein aktives Azure-Konto. Wenn Sie über kein Konto verfügen, können Sie in nur wenigen Minuten ein kostenloses Testkonto erstellen. Ausführliche Informationen finden Sie unter [Kostenlose Azure-Testversion](http://azure.microsoft.com/pricing/free-trial/).

## <a name="before-you-start"></a>Vorbereitung

Bevor Sie Code für Ihr Gerät schreiben, stellen Sie Ihren Solution Accelerator für Remoteüberwachung bereit und fügen der Lösung ein neues physisches Gerät hinzu.

### <a name="deploy-your-remote-monitoring-solution-accelerator"></a>Bereitstellen des Solution Accelerators für Remoteüberwachung

Das in diesem Tutorial erstellte **Kühlgerät** sendet Daten an eine Instanz des Solution Accelerators für [Remoteüberwachung](../articles/iot-suite/iot-suite-remote-monitoring-explore.md). Wenn Sie den Solution Accelerator für Remoteüberwachung noch nicht in Ihrem Azure-Konto bereitgestellt haben, lesen Sie den Artikel [Bereitstellen des Solution Accelerators für Remoteüberwachung](../articles/iot-accelerators/iot-accelerators-remote-monitoring-deploy.md).

Wenn der Bereitstellungsvorgang für die Remoteüberwachungslösung abgeschlossen ist, klicken Sie auf **Starten**, um das Lösungsdashboard im Browser zu öffnen.

![Das Lösungsdashboard](media/iot-suite-selector-connecting/dashboard.png)

### <a name="add-your-device-to-the-remote-monitoring-solution"></a>Hinzufügen des Geräts zur Remoteüberwachungslösung

> [!NOTE]
> Wenn Sie der Lösung bereits ein Gerät hinzugefügt haben, können Sie diesen Schritt überspringen. Für den nächsten Schritt ist jedoch die Verbindungszeichenfolge Ihres Geräts erforderlich. Sie können die Verbindungszeichenfolge eines Geräts im [Azure-Portal](https://portal.azure.com) oder über das CLI-Tool [az iot](https://docs.microsoft.com/cli/azure/iot?view=azure-cli-latest) abrufen.

Damit ein Gerät eine Verbindung mit dem Solution Accelerator herstellen kann, muss es sich mit gültigen Anmeldeinformationen bei IoT Hub identifizieren können. Sie haben die Möglichkeit, die Verbindungszeichenfolge mit den Anmeldeinformationen zu speichern, wenn Sie das Gerät der Lösung hinzufügen. Die Geräte-Verbindungszeichenfolge wird zu einem späteren Zeitpunkt in diesem Lernprogramm in Ihre Clientanwendung einbezogen.

Um Ihrer Lösung für die Remoteüberwachung ein Gerät hinzuzufügen, führen Sie in der Lösung auf der Seite **Geräte** die folgenden Schritte durch:

1. Wählen Sie **+ Neues Gerät** und dann **Physisch** als **Gerätetyp** aus:

    ![Hinzufügen eines physischen Geräts](media/iot-suite-selector-connecting/devicesprovision.png)

1. Geben Sie als Geräte-ID **Physical-chiller** ein. Wählen Sie die Optionen **Symmetrischer Schlüssel** und **Schlüssel automatisch generieren**:

    ![Auswählen von Geräteoptionen](media/iot-suite-selector-connecting/devicesoptions.png)

1. Wählen Sie **Anwenden** aus. Notieren Sie sich dann die Werte in den Feldern **Geräte-ID**, **Primärschlüssel** und **Verbindungszeichenfolge – Primärschlüssel**:

    ![Abrufen der Anmeldeinformationen](media/iot-suite-selector-connecting/credentials.png)

Sie haben nun dem Solution Accelerator für Remoteüberwachung ein physisches Gerät hinzugefügt und dessen Verbindungszeichenfolge notiert. In den folgenden Abschnitten implementieren Sie die Clientanwendung, die mithilfe der Verbindungszeichenfolge des Geräts eine Verbindung mit Ihrer Lösung herstellt.

Die Clientanwendung implementiert das integrierte Gerätemodell des **Kühlers**. Das Gerätemodell eines Solution Accelerators gibt folgende Informationen im Zusammenhang mit einem Gerät an:

* Die Eigenschaften, die das Gerät der Lösung meldet. Beispiel: Ein **Kühlgerät** meldet Informationen zur Firmware und zum Standort.
* Die Arten von Telemetriedaten, die das Gerät an die Lösung sendet. Beispiel: Ein **Kühlgerät** sendet Temperatur-, Luftfeuchtigkeits- und Druckwerte.
* Die Methoden, die Sie mit der Lösung zur Ausführung auf dem Gerät planen können. Beispiel: Ein **Kühlgerät** muss die Methoden **Reboot**, **FirmwareUpdate**, **EmergencyValveRelease** und  **IncreasePressure** implementieren.