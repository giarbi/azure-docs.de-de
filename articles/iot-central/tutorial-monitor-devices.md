---
title: Überwachen Ihrer Geräte in Azure IoT Central | Microsoft-Dokumentation
description: Als Bediener können Sie mit der Azure IoT Central-Anwendung Ihre Geräte überwachen.
services: iot-central
author: tanmaybhagwat
ms.author: tanmayb
ms.date: 04/16/2018
ms.topic: tutorial
ms.prod: microsoft-iot-central
manager: timlt
ms.openlocfilehash: a07c9e3c28fadaead8bfaaebe4d1ee06ac66a99e
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/16/2018
ms.locfileid: "34201386"
---
# <a name="use-azure-iot-central-to-monitor-your-devices"></a>Überwachen Ihrer Geräte mithilfe von Azure IoT Central

Dieses Tutorial richtet sich an Bediener und erläutert, wie Sie mithilfe Ihrer Microsoft Azure IoT Central-Anwendung Ihre Geräte überwachen und Einstellungen ändern.

In diesem Tutorial lernen Sie Folgendes:

> [!div class="checklist"]
> * Empfangen einer Benachrichtigung
> * Untersuchen eines Problems
> * Beheben eines Problems

## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie beginnen, muss der Ersteller die drei Tutorials für Ersteller absolvieren, um die Azure IoT Central-Anwendung zu erstellen:

* [Definieren eines neuen Gerätetyps](tutorial-define-device-type.md)
* [Konfigurieren von Regeln und Aktionen für Ihr Gerät in Azure IoT Central](tutorial-configure-rules.md)
* [Anpassen der Azure IoT Central-Ansicht für Bediener](tutorial-customize-operator.md)

## <a name="receive-a-notification"></a>Empfangen einer Benachrichtigung

Azure IoT Central sendet Benachrichtigungen für Geräte in Form von E-Mails. Der Ersteller hat eine Regel hinzugefügt, die dafür sorgt, dass eine Benachrichtigung gesendet wird, wenn die Temperatur in einer verbundenen Klimaanlage einen Schwellenwert übersteigt. Überprüfen Sie die E-Mails, die an das Konto gesendet wurden, das der Ersteller für den Empfang von Benachrichtigungen ausgewählt hat.

Öffnen Sie die E-Mail, die Sie am Ende des Tutorials [Konfigurieren von Regeln und Aktionen für Ihr Gerät in Azure IoT Central](tutorial-configure-rules.md) erhalten haben. Klicken Sie in der E-Mail auf **Click here to open your device** (Klicken Sie hier, um Ihr Gerät öffnen):

![Regeln im Anwendungs-Generator](media/tutorial-monitor-devices/email.png)

Die Seite **Gerät** für das simulierte Gerät **Connected Air Conditioner-1**, das Sie in den vorherigen Tutorials erstellt haben, wird in Ihrem Browser geöffnet:

![Gerät, das die Benachrichtigungs-E-Mail ausgelöst hat](media/tutorial-monitor-devices/sourcedevice.png)

## <a name="investigate-an-issue"></a>Untersuchen eines Problems

Als Bediener können Sie auf den Seiten **Measurements** (Messungen), **Einstellungen**, **Eigenschaften**, **Regeln** und **Dashboard** Informationen zum Gerät anzeigen. Der Ersteller hat das **Dashboard** so angepasst, dass dort wichtige Informationen zu einer verbundenen Klimaanlage angezeigt werden.

* Klicken Sie auf die Ansicht **Dashboard**, um Informationen zum Gerät anzuzeigen:

    ![Gerätedashboard](media/tutorial-monitor-devices/initial_screen.png)

    Das Diagramm auf dem Dashboard zeigt einen Plot der Gerätetemperatur. Auf der Kachel **Set target temperature** (Festgelegte Zieltemperatur) wird zudem die aktuelle Zieltemperatur für das Gerät angezeigt. Sie entscheiden, dass die Zieltemperatur zu hoch ist.

## <a name="remediate-an-issue"></a>Beheben eines Problems

Verwenden Sie zum Ändern der Zieltemperatur des Geräts die Seite **Einstellungen**:

1. Klicken Sie auf **Einstellungen**. Legen Sie den Wert für **Set Temperature** (Sollwerttemperatur) auf „100“ fest. Klicken Sie auf **Aktualisieren**, um die neue Zieltemperatur an das Gerät zu senden. Wenn das Gerät die Einstellungsänderung bestätigt, ändert sich der Status des Einstellungswerts in **Synchronisiert**:

    ![Aktualisieren der Einstellungen](media/tutorial-monitor-devices/change_settings.png)

1. Klicken Sie auf **Dashboard**, und überprüfen Sie den neuen Einstellungswert:

    ![Aktualisiertes Gerätedashboard](media/tutorial-monitor-devices/new_settings.png)

## <a name="next-steps"></a>Nächste Schritte

In diesem Tutorial haben Sie Folgendes gelernt:

> [!div class="nextstepaction"]
> * Empfangen einer Benachrichtigung
> * Untersuchen eines Problems
> * Beheben eines Problems

Nachdem Sie nun Ihr Gerät überwacht haben, können Sie als Nächstes [ein Gerät hinzufügen](tutorial-add-device.md).
