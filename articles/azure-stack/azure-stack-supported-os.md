---
title: Unterstützte Gastbetriebssysteme für Azure Stack | Microsoft-Dokumentation
description: Diese Gastbetriebssysteme können mit Azure Stack verwendet werden.
services: azure-stack
documentationcenter: ''
author: Brenduns
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/11/2018
ms.author: Brenduns
ms.reviewer: JeffGoldner
ms.openlocfilehash: d90ac726ae689cf299ee41d4e7ff4c17769e7455
ms.sourcegitcommit: fc64acba9d9b9784e3662327414e5fe7bd3e972e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/12/2018
ms.locfileid: "34074829"
---
# <a name="guest-operating-systems-supported-on-azure-stack"></a>Für Azure Stack unterstützte Gastbetriebssysteme

*Gilt für: integrierte Azure Stack-Systeme und Azure Stack Development Kit*

## <a name="windows"></a>Windows

Azure Stack unterstützt die Windows-Gastbetriebssysteme aus der folgenden Tabelle:

| Betriebssystem | Beschreibung | Im Marketplace verfügbar |
| --- | --- | --- | --- | --- | --- |
| Windows Server, Version 1709 | 64 Bit | Core mit Containern |
| Windows Server 2016 | 64 Bit |  Datacenter, Datacenter Core, Datacenter mit Containern |
| Windows Server 2012 R2 | 64 Bit |  Datacenter |
| Windows Server 2012 | 64 Bit |  Datacenter |
| Windows Server 2008 R2 SP1 | 64 Bit |  Datacenter |
| Windows Server 2008 SP2 | 64 Bit |  Verwendung eigener Images (Bring Your Own Image, BYOI) |
| Windows 10 *(siehe Hinweis 1)* | 64 Bit, Pro und Enterprise | Verwendung eigener Images (Bring Your Own Image, BYOI) |

***Hinweis 1:*** *Wenn Sie Windows 10-Clientbetriebssysteme für Azure Stack bereitstellen möchten, müssen Sie über eine [benutzerspezifische Windows-Lizenzierung](https://www.microsoft.com/Licensing/product-licensing/windows10.aspx) verfügen oder den Kauf über einen [QMTH](https://www.microsoft.com/CloudandHosting/licensing_sca.aspx) (Qualified Multitenant Hoster) abwickeln.*

Marketplace-Images stehen für eine Lizenzierung mit nutzungsbasierter Abrechnung oder für eine BYOL-Lizenzierung (EA/SPLA) zur Verfügung. Es kann jedoch nicht beides für die gleiche Azure Stack-Instanz verwendet werden. Während der Bereitstellung fügt Azure Stack eine geeignete Version des Gast-Agents in das Image ein.

 Datacenter-Editionen stehen im Marketplace zum Download zur Verfügung. Kunden können ihre eigenen Serverimages mit anderen Editionen verwenden. Windows-Clientimages stehen im Marketplace nicht zur Verfügung.

## <a name="linux"></a>Linux

Die im Marketplace verfügbaren Linux-Distributionen enthalten den erforderlichen Windows Azure-Linux-Agent (WALA). Wenn Sie in Azure Stack Ihr eigenes Image verwenden möchten, beachten Sie die Richtlinien unter [Hinzufügen von Linux-Images zu Azure Stack](azure-stack-linux.md).

> [!NOTE]
> Benutzerdefinierte Images müssen mit der neuesten öffentlichen WALA-Version erstellt werden. Versionen vor 2.2.18 funktionieren in Azure Stack möglicherweise nicht ordnungsgemäß.
>
> [cloud-init](https://cloud-init.io/) wird derzeit für Azure Stack nicht unterstützt.

| Distribution | Beschreibung | Herausgeber | Marketplace |
| --- | --- | --- | --- | --- | --- |
| 6.9 (CentOS-basiert) | 64 Bit | Rogue Wave | Ja |
| 7.4 (CentOS-basiert) | 64 Bit | Rogue Wave | Ja |
| ClearLinux | 64 Bit | ClearLinux.org | Ja |
| Container Linux |  64 Bit | CoreOS | Stable |
| Debian 8 „Jessie“ | 64 Bit | credativ |  Ja |
| Debian 9 „Stretch“ | 64 Bit | credativ | Ja |
| Red Hat Enterprise Linux 7.x | 64 Bit | Red Hat |Verwendung eigener Images (Bring Your Own Image, BYOI) |
| SLES 11SP4 | 64 Bit | SUSE | Ja |
| SLES 12SP3 | 64 Bit | SUSE | Ja |
| Ubuntu 14.04-LTS | 64 Bit | Canonical | Ja |
| Ubuntu 16.04-LTS | 64 Bit | Canonical | Ja |
| Ubuntu 18.04-LTS | 64 Bit | Canonical | Ja |

Andere Linux-Distributionen können in Zukunft unterstützt werden.

Informationen zur Unterstützung von Red Hat Enterprise Linux finden Sie unter [Red Hat and Azure Stack: Frequently Asked Questions](https://access.redhat.com/articles/3413531) (Red Hat und Azure Stack: Häufig gestellte Fragen).
