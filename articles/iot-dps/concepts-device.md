---
title: Gerätekonzepte bei der Azure-Gerätebereitstellung | Microsoft-Dokumentation
description: Beschreibt Konzepte der Gerätebereitstellung, die speziell für Geräte mit dem Device Provisioning-Dienst und IoT Hub gelten
services: iot-dps
keywords: ''
author: nberdy
ms.author: nberdy
ms.date: 09/05/2017
ms.topic: article
ms.service: iot-dps
documentationcenter: ''
manager: timlt
ms.devlang: na
ms.custom: mvc
ms.openlocfilehash: 5482801461e2afea33d65d559723116f37a35d1f
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/28/2018
---
# <a name="iot-hub-device-provisioning-service-device-concepts"></a>Konzepte für Geräte mit dem IoT Hub Device Provisioning-Dienst

Der IoT Hub Device Provisioning-Dienst ist ein Hilfsdienst für IoT Hub, mit dem Sie Geräte ohne manuelles Eingreifen auf einem angegebenen IoT Hub konfigurieren können. Mit dem Device Provisioning-Dienst können Sie Millionen von Geräten auf sichere und skalierbare Weise bereitstellen.

Dieser Artikel bietet eine Übersicht über die *Gerätekonzepte* bei der Gerätebereitstellung. Dieser Artikel ist besonders für Personen relevant, die am [Fertigungsschritt](about-iot-dps.md#manufacturing-step) der Vorbereitung von Geräten für die Bereitstellung beteiligt sind.

## <a name="attestation-mechanism"></a>Nachweismechanismus

Der Nachweismechanismus ist die Methode für das Überprüfen der Identität eines Geräts. Der Nachweismechanismus ist auch für die Registrierungsliste wichtig, über die der Bereitstellungsdienst erfährt, welche Methode zum Nachweis für ein bestimmtes Gerät verwendet werden soll.

> [!NOTE]
> IoT Hub verwendet das „Authentifizierungsschema“ für ein ähnliches Konzept in diesem Dienst.

Der Device Provisioning-Dienst unterstützt zwei Arten von Nachweisen:
* **X.509-Zertifikate** – basierend auf dem Standardauthentifizierungsablauf für X.509-Zertifikate.
* **Trusted Platform Module (TPM)** – basierend auf einer Nonce-Herausforderung mit dem TPM-Standard für Schlüssel zum Bereitstellen eines signierten SAS-Tokens (Shared Access Signature). Dies erfordert keine physische TPM-Instanz auf dem Gerät. Der Dienst erwartet für den Nachweis jedoch den Endorsement Key gemäß [TPM-Spezifikation](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/).

## <a name="hardware-security-module"></a>Hardwaresicherheitsmodul

Das Hardwaresicherheitsmodul (HSM) dient der sicheren, hardwarebasierten Speicherung von Gerätegeheimnissen und gilt als die sicherste Form der Geheimnisspeicherung. Sowohl X.509-Zertifikate als auch SAS-Token können im HSM gespeichert werden. HSMs können mit beiden Nachweismechanismen verwendet werden, die der Bereitstellungsdienst unterstützt.

> [!TIP]
> Es wird dringend empfohlen, ein HSM mit Geräten zu verwenden, um Geheimnisse sicher auf Ihren Geräten zu speichern.

Gerätegeheimnisse können auch in Software (Arbeitsspeicher) gespeichert werden. Dies gilt jedoch im Vergleich zu einem HSM als die weniger sichere Speicherform.

## <a name="registration-id"></a>Registrierungs-ID

Die Registrierungs-ID wird zur eindeutigen Identifizierung eines Geräts im Device Provisioning-Dienst verwendet. Die Geräte-ID muss im [ID-Bereich](#id-scope) des Bereitstellungsdiensts eindeutig sein. Jedes Gerät benötigt eine Registrierungs-ID. Die Registrierungs-ID ist alphanumerisch und besteht aus Kleinbuchstaben und ggf. Bindestrichen.

* Bei TPM wird die Registrierungs-ID durch das TPM selbst bereitgestellt.
* Im Fall von X.509-basierten Nachweisen wird die Registrierungs-ID als Antragstellername des Zertifikats bereitgestellt.

## <a name="device-id"></a>Geräte-ID

Die Geräte-ID ist die ID, die in IoT Hub angezeigt wird. Die gewünschte Geräte-ID kann im Registrierungseintrag festgelegt werden, eine Festlegung ist aber nicht zwingend. Wenn in der Registrierungsliste keine gewünschte Geräte-ID angegeben wurde, wird bei der Registrierung des Geräts die Registrierungs-ID als die Geräte-ID verwendet. Erfahren Sie mehr zu [Geräte-IDs in IoT Hub](../iot-hub/iot-hub-devguide-identity-registry.md).

## <a name="id-scope"></a>ID-Bereich

Der ID-Bereich wird einem Device Provisioning-Dienst zugewiesen, wenn er vom Benutzer erstellt wird. Er dient der Identifizierung des jeweiligen Bereitstellungsdiensts, über den sich das Gerät registriert. Der ID-Bereich wird vom Dienst generiert. Er kann nicht geändert werden und stellt Eindeutigkeit sicher.

> [!NOTE]
> Eindeutigkeit ist wichtig für lang andauernde Bereitstellungsvorgänge und Zusammenführungs- sowie Kaufszenarien.
