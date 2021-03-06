---
title: Erstellen eines Azure Active Directory B2C-Mandanten | Microsoft-Dokumentation
description: Thema zum Erstellen eines Azure Active Directory B2C-Mandanten
services: active-directory-b2c
documentationcenter: ''
author: davidmu1
manager: mtillman
editor: ''
ms.service: active-directory-b2c
ms.workload: identity
ms.topic: article
ms.date: 06/07/2017
ms.author: davidmu
ms.openlocfilehash: 56e0ae7454e86911c894da88b5aa8ccc03a08af3
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/23/2018
---
# <a name="create-an-azure-active-directory-b2c-tenant-in-the-azure-portal"></a>Erstellen eines Azure Active Directory B2C-Mandanten im Azure-Portal

In dieser Schnellstartanleitung erfahren Sie, wie Sie in wenigen Minuten einen Microsoft Azure Active Directory B2C-Mandanten (Azure AD) erstellen. Dann verfügen Sie über einen B2C-Mandanten (auch als Verzeichnis bezeichnet) zum Registrieren von B2C-Anwendungen.

## <a name="prerequisites"></a>Voraussetzungen

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="log-in-to-azure"></a>Anmelden an Azure

Melden Sie sich beim [Azure-Portal](https://portal.azure.com/)an.

## <a name="create-an-azure-ad-b2c-tenant"></a>Erstellen eines Azure AD B2C-Mandanten

In Ihren vorhandenen Mandanten können B2C-Features nicht aktiviert werden. Sie müssen einen Azure AD B2C-Mandanten erstellen.

[!INCLUDE [active-directory-b2c-create-tenant](../../includes/active-directory-b2c-create-tenant.md)]

Glückwunsch, Sie haben einen Azure Active Directory B2C-Mandanten erstellt. Sie sind ein globaler Administrator des Mandanten. Sie können bei Bedarf weitere globale Administratoren hinzufügen. Um zu Ihrem neuen Mandanten zu wechseln, klicken Sie auf den *Link zur Verwaltung Ihres neuen Mandanten*.

![Link zur Verwaltung Ihres neuen Mandanten](./media/active-directory-b2c-get-started/manage-new-b2c-tenant-link.png)

> [!IMPORTANT]
> Wenn Sie einen B2C-Mandanten für eine Produktions-App verwenden möchten, lesen Sie den Artikel zum [Vergleich zwischen Produktionsmandanten und B2C-Vorschaumandanten](active-directory-b2c-reference-tenant-type.md). Beim Löschen eines vorhandenen B2C-Mandanten und erneuten Erstellen mit dem gleichen Domänennamen treten bekannte Probleme auf. Sie müssen einen B2C-Mandanten mit einem anderen Domänennamen erstellen.
>
>

## <a name="link-your-tenant-to-your-subscription"></a>Verknüpfen Ihres Mandanten mit Ihrem Abonnement

Sie müssen Ihren Azure AD B2C-Mandanten mit Ihrem Azure-Abonnement verknüpfen, um die gesamte B2C-Funktionalität zu aktivieren und die Nutzungsgebühren zahlen zu können. Weitere Informationen finden Sie in [diesem Artikel](active-directory-b2c-how-to-enable-billing.md). Wenn Sie Ihren Azure AD B2C-Mandanten nicht mit Ihrem Azure-Abonnement verknüpfen, wird ein Teil der Funktionalität blockiert und eine Warnmeldung in den B2C-Einstellungen angezeigt („Mit diesem B2C-Mandanten ist kein Abonnement verknüpft, oder das Abonnement benötigt Ihre Aufmerksamkeit.“). Es ist wichtig, dass Sie diesen Schritt ausführen, bevor Sie Ihre Apps in die Produktion übertragen.

## <a name="easy-access-to-settings"></a>Einfacher Zugriff auf Einstellungen

[!INCLUDE [active-directory-b2c-find-service-settings](../../includes/active-directory-b2c-find-service-settings.md)]

Sie können auf das Blatt auch zugreifen, indem Sie am oberen Rand des Portals `Azure AD B2C` in **Ressourcen suchen** eingeben. Wählen Sie in der Ergebnisliste **Azure AD B2C**, um auf das B2C-Einstellungenblatt zuzugreifen.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Registrieren Ihrer B2C-Anwendung in einem B2C-Mandanten](active-directory-b2c-app-registration.md)