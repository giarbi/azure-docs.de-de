---
title: Konfigurieren der Sicherheit von Azure SQL-Datenbank für die Notfallwiederherstellung | Microsoft Docs
description: Machen Sie sich mit Überlegungen zur Sicherheit vertraut, die für das Konfigurieren und Verwalten der Sicherheit nach einer Wiederherstellung der Datenbank oder einem Failover auf einen sekundären Server gelten.
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.custom: business continuity
ms.topic: article
ms.date: 04/01/2018
ms.author: sashan
ms.openlocfilehash: 6b4b599fee9a8ff5b4674cf26ca19b22230d6364
ms.sourcegitcommit: 3a4ebcb58192f5bf7969482393090cb356294399
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
# <a name="configure-and-manage-azure-sql-database-security-for-geo-restore-or-failover"></a>Konfigurieren und Verwalten der Sicherheit von Azure SQL-Datenbank für die Geowiederherstellung oder den Failover 

In diesem Thema werden die Authentifizierungsanforderungen zum Konfigurieren und Steuern der [aktiven Georeplikation](sql-database-geo-replication-overview.md) sowie die erforderlichen Schritte zum Einrichten des Benutzerzugriffs auf die sekundäre Datenbank beschrieben. Außerdem wird beschrieben, wie nach der [Geowiederherstellung](sql-database-recovery-using-backups.md#geo-restore) der Zugriff auf die wiederhergestellte Datenbank aktiviert wird. Weitere Informationen zu Wiederherstellungsoptionen finden Sie unter [Übersicht über die Geschäftskontinuität](sql-database-business-continuity.md).

> [!NOTE]
> Die [aktive Georeplikation](sql-database-geo-replication-overview.md) ist jetzt für alle Datenbanken in allen Diensttarifen verfügbar.
>  

## <a name="disaster-recovery-with-contained-users"></a>Notfallwiederherstellung mit eigenständigen Benutzern
Im Gegensatz zu herkömmlichen Benutzern, die Anmeldungen in der „master“-Datenbank zugeordnet werden müssen, wird ein eigenständiger Benutzer vollständig von der Datenbank selbst verwaltet. Dies hat zwei Vorteile. Beim Notfallwiederherstellungs-Szenario können sich die Benutzer weiter ohne zusätzliche Konfiguration mit der neuen primären Datenbank bzw. mit der Geowiederherstellung wiederhergestellten Datenbank verbinden, da die Datenbank die Benutzer verwaltet. Es gibt bei dieser Konfiguration auch vom Standpunkt der Anmeldung potenzielle Skalierbarkeits- und Leistungsvorteile. Weitere Informationen finden Sie unter [Eigenständige Datenbankbenutzer – machen Sie Ihre Datenbank portabel](https://msdn.microsoft.com/library/ff929188.aspx). 

Der Hauptaspekt des Kompromisses ist, dass die Verwaltung des Notfallwiederherstellungs-Prozesses im Verhältnis eine größere Herausforderung darstellt. Wenn mehrere Ihrer Datenbanken dieselben Anmeldedaten verwenden, kann sich die Verwaltung der Anmeldeinformationen anhand von eigenständigen Benutzern in mehreren Datenbanken nachteilig auswirken. Die Kennwortrotationsrichtlinie erfordert beispielsweise, dass Änderungen konsistent in mehreren Datenbanken durchgeführt werden, anstatt das Kennwort für die Anmeldung einmal in der Masterdatenbank zu ändern. Wenn Sie mehrere Datenbanken mit dem gleichen Benutzernamen und Kennwort verwenden, sollten Sie darum keine eigenständigen Benutzer verwenden. 

## <a name="how-to-configure-logins-and-users"></a>Konfigurieren von Anmeldungen und Benutzern
Wenn Sie mit Anmeldungen und Benutzern (nicht mit eigenständigen Benutzern) arbeiten, müssen Sie zusätzliche Schritte durchführen, um sicherzustellen, dass die gleichen Anmeldenamen in der Masterdatenbank vorhanden sind. In den folgenden Abschnitten werden die entsprechenden Schritte und zusätzliche Aspekte behandelt.

### <a name="set-up-user-access-to-a-secondary-or-recovered-database"></a>Einrichten des Benutzerzugriffs auf eine sekundäre bzw. wiederhergestellte Datenbank
Damit die sekundäre Datenbank als schreibgeschützte sekundäre Datenbank verwendet werden kann, und um den ordnungsgemäßen Zugriff auf die neue primäre Datenbank bzw. mit der Geowiederherstellung wiederhergestellte Datenbank sicherzustellen, muss für die Masterdatenbank des Zielservers vor der Wiederherstellung die geeignete Sicherheitskonfiguration durchgeführt werden.

Die spezifischen Berechtigungen für die einzelnen Schritte werden weiter unten in diesem Thema beschrieben.

Die Vorbereitung des Benutzerzugriffs auf eine mit Georeplikation erzeugte sekundäre Datenbank sollte im Rahmen der Konfiguration der Georeplikation erfolgen. Die Vorbereitung des Benutzerzugriffs auf mit der Geowiederherstellung wiederhergestellte Datenbanken sollte zu einem beliebigen Zeitpunkt erfolgen, wenn der ursprüngliche Server online ist (z.B. als Teil des DR-Drills).

> [!NOTE]
> Wenn Sie das Failover zu einem Server oder die Geowiederherstellung auf einem Server ausführen, für den die Anmeldenamen nicht ordnungsgemäß konfiguriert sind, beschränkt sich der Zugriff auf das Serveradministratorkonto.
> 
> 

Das Einrichten von Anmeldenamen auf dem Zielserver umfasst die unten beschriebenen drei Schritte:

#### <a name="1-determine-logins-with-access-to-the-primary-database"></a>1. Bestimmen der Anmeldungen mit Zugriff auf die primäre Datenbank:
Der erste Schritt des Prozesses ist das Bestimmen, welche Anmeldungen auf dem Zielserver dupliziert werden müssen. Dies geschieht mit zwei SELECT-Anweisungen, die auf die logische „master“-Datenbank auf dem Quellserver und auf die primäre Datenbank selbst angewendet werden.

Nur der Serveradministrator oder ein Mitglied der Serverrolle **LoginManager** kann mit der folgenden SELECT-Anweisung die Anmeldungen auf dem Quellserver bestimmen. 

    SELECT [name], [sid] 
    FROM [sys].[sql_logins] 
    WHERE [type_desc] = 'SQL_Login'

Nur ein Mitglied der Datenbankrolle „db_owner“, der Benutzer „dbo“ oder der Serveradministrator kann alle Datenbankbenutzerprinzipale in der primären Datenbank bestimmen.

    SELECT [name], [sid]
    FROM [sys].[database_principals]
    WHERE [type_desc] = 'SQL_USER'

#### <a name="2-find-the-sid-for-the-logins-identified-in-step-1"></a>2. Suchen der SID für die in Schritt 1 identifizierten Anmeldungen:
Durch Vergleichen der Ausgabe der Abfragen aus dem vorherigen Abschnitt und Zuordnen der SIDs können Sie die Serveranmeldung dem Datenbankbenutzer zuordnen. Anmeldungen, die über einen Datenbankbenutzer mit einer entsprechenden SID verfügen, haben als Datenbankbenutzerprinzipal Benutzerzugriff auf diese Datenbank. 

Die folgende Abfrage kann verwendet werden, um alle Benutzerprinzipale mit ihre SIDs in einer Datenbank anzuzeigen. Nur ein Mitglied der Datenbankrolle „db_owner“ oder Serveradministrator kann diese Abfrage ausführen.

    SELECT [name], [sid]
    FROM [sys].[database_principals]
    WHERE [type_desc] = 'SQL_USER'

> [!NOTE]
> Die Benutzer **INFORMATION_SCHEMA** und **sys** haben die SID *NULL*, und die SID von **guest** lautet **0x00**. Die SID von **dbo** kann mit *0x01060000000001648000000000048454* beginnen, wenn der Datenbankersteller der Serveradministrator und kein Mitglied von **DbManager** war.
> 
> 

#### <a name="3-create-the-logins-on-the-target-server"></a>3. Erstellen der Anmeldenamen auf dem Zielserver:
Der letzte Schritt besteht darin, auf dem oder den Zielservern die Anmeldungen mit den entsprechenden SIDs zu erstellen. Die grundlegende Syntax ist wie folgt.

    CREATE LOGIN [<login name>]
    WITH PASSWORD = <login password>,
    SID = <desired login SID>

> [!NOTE]
> Wenn Sie Benutzerzugriff auf die sekundäre Datenbank, aber nicht auf die primäre gewähren möchten, ändern Sie dazu die Benutzeranmeldung auf dem primären Server mithilfe der folgenden Syntax.
> 
> ALTER LOGIN <login name> DISABLE
> 
> DISABLE ändert nicht das Kennwort, sodass Sie sie bei Bedarf stets aktivieren können.
> 
> 

## <a name="next-steps"></a>Nächste Schritte
* Weitere Informationen zum Verwalten von Datenbankzugriff und Anmeldungen finden Sie unter [Sicherheit von SQL-Datenbank: Verwalten von Datenbankzugriff und Anmeldesicherheit](sql-database-manage-logins.md).
* Weitere Informationen zu eigenständigen Datenbankbenutzern finden Sie unter [Eigenständige Datenbankbenutzer – machen Sie Ihre Datenbank portabel](https://msdn.microsoft.com/library/ff929188.aspx).
* Informationen zur Verwendung und Konfiguration der aktiven Georeplikation finden Sie unter [aktive Georeplikation](sql-database-geo-replication-overview.md)
* Informationen zur Verwendung der Geowiederherstellung finden Sie unter [Geowiederherstellung](sql-database-recovery-using-backups.md#geo-restore)

