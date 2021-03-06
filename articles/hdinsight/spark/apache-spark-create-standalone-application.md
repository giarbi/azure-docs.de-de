---
title: 'Tutorial: Erstellen einer Scala Maven-Anwendung für Spark in HDInsight mithilfe von IntelliJ | Microsoft-Dokumentation'
description: Erstellen Sie eine Spark-Anwendung, die in Scala mit Apache Maven als Buildsystem mit einem vorhandenen von IntelliJ IDEA bereitgestellten Maven-Archetyp für Scala geschrieben ist.
services: hdinsight
documentationcenter: ''
author: mumian
manager: cgronlun
editor: cgronlun
tags: azure-portal
ms.assetid: b2467a40-a340-4b80-bb00-f2c3339db57b
ms.service: hdinsight
ms.custom: hdinsightactive,mvc
ms.devlang: na
ms.topic: tutorial
ms.date: 05/07/2018
ms.author: jgao
ms.openlocfilehash: c72f513c7134c556afa5fa5d0b94c17b1142be54
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2018
---
# <a name="tutorial-create-a-scala-maven-application-for-spark-in-hdinsight-using-intellij"></a>Tutorial: Erstellen einer Scala Maven-Anwendung für Spark in HDInsight mithilfe von IntelliJ

In diesem Tutorial erfahren Sie, wie Sie eine in Scala geschriebene Spark-Anwendung erstellen, die Maven mit IntelliJ IDEA nutzt. Dabei wird Apache Maven als Buildsystem verwendet. Ausgangspunkt ist ein vorhandener, von IntelliJ IDEA bereitgestellter Maven-Archetyp für Scala.  Das Erstellen einer Scala-Anwendung in IntelliJ IDEA umfasst die folgenden Schritte:

* Verwenden von Maven als Buildsystem
* Aktualisieren der Projektobjektmodell-Datei (POM-Datei) zum Auflösen von Spark-Modulabhängigkeiten
* Schreiben der Anwendung in Scala
* Generieren einer JAR-Datei, die an HDInsight Spark-Cluster übermittelt werden kann
* Ausführen der Anwendung in einem Spark-Cluster mithilfe von Livy

> [!NOTE]
> HDInsight stellt auch ein IntelliJ IDEA-Plug-In bereit, um die Erstellung und Übermittlung von Anwendungen an einen HDInsight Spark-Cluster unter Linux zu vereinfachen. Weitere Informationen finden Sie unter [Verwenden des HDInsight-Tools-Plug-Ins für IntelliJ IDEA zum Erstellen von Spark Scala-Anwendungen für HDInsight Spark-Cluster unter Linux (Vorschau)](apache-spark-intellij-tool-plugin.md).
> 

In diesem Tutorial lernen Sie Folgendes:
> [!div class="checklist"]
> * Verwenden von IntelliJ zum Entwickeln einer Scala Maven-Anwendung

Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/) erstellen, bevor Sie beginnen.


## <a name="prerequisites"></a>Voraussetzungen

* Ein Apache Spark-Cluster unter HDInsight. Eine Anleitung finden Sie unter [Erstellen von Apache Spark-Clustern in Azure HDInsight](apache-spark-jupyter-spark-sql.md).
* Oracle Java Development Kit. Das Installationspaket finden Sie [hier](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).
* Eine Java-IDE. In diesem Artikel wird IntelliJ IDEA 18.1.1 verwendet. Das Installationspaket finden Sie [hier](https://www.jetbrains.com/idea/download/).

## <a name="install-scala-plugin-for-intellij-idea"></a>Installieren des Scala-Plug-Ins für IntelliJ IDEA
Installieren Sie das Scala-Plug-In in folgenden Schritten:

1. Öffnen Sie IntelliJ IDEA.
2. Wählen Sie auf dem Begrüßungsbildschirm **Configure (Konfigurieren)** und dann **Plugins (Plug-Ins)** aus.
   
    ![Scala-Plug-In aktivieren](./media/apache-spark-create-standalone-application/enable-scala-plugin.png)
3. Wählen Sie im nächsten Bildschirm links unten **Install JetBrains plugin** (JetBrains-Plug-In installieren) aus. 
4. Suchen Sie im Dialogfeld **Browse JetBrains Plugins (JetBrains-Plug-Ins durchsuchen)** nach **Scala**, und wählen Sie anschließend **Install (Installieren)** aus.
   
    ![Scala-Plug-In installieren](./media/apache-spark-create-standalone-application/install-scala-plugin.png)
5. Nach erfolgreicher Plug-In-Installation müssen Sie die IDE neu starten.

## <a name="create-a-standalone-scala-project"></a>Erstellen eines eigenständigen Scala-Projekts
1. Öffnen Sie IntelliJ IDEA.
2. Wählen Sie im Menü **Datei** zum Erstellen eines neuen Projekts **Neu > Projekt** aus.
3. Wählen Sie im Dialogfeld für das neue Projekt Folgendes aus:
   
    ![Maven-Projekt erstellen](./media/apache-spark-create-standalone-application/create-maven-project.png)
   
   * Wählen Sie den Projekttyp **Maven** aus.
   * Geben Sie ein **Projekt-SDK**an. Klicken Sie auf **Neu**, und navigieren Sie zum Installationsverzeichnis von Java, üblicherweise `C:\Program Files\Java\jdk1.8.0_66`.
   * Aktivieren Sie das Kontrollkästchen **Archetypbasierte Erstellung** .
   * Wählen Sie in der Liste mit den Archetypen die Option **org.scala-tools.archetypes:scala-archetype-simple**aus. Dieser Archetyp erstellt die passende Verzeichnisstruktur und lädt die erforderlichen Standardabhängigkeiten zum Schreiben des Scala-Programms herunter.
4. Klicken Sie auf **Weiter**.
5. Geben Sie passende Werte für **GroupId**, **ArtifactId** und **Version** an. In diesem Tutorial werden die folgenden Werte verwendet:

    - GroupId: com.microsoft.spark.example
    - ArtifactId: SparkSimpleApp
6. Klicken Sie auf **Weiter**.
7. Überprüfen Sie die Einstellungen, und wählen Sie **Weiter** aus.
8. Überprüfen Sie den Projektnamen und einen Speicherort, und wählen Sie anschließend **Fertig stellen** aus.
9. Wählen Sie im linken Bereich **src > test > scala > com > microsoft > spark > example** aus, klicken Sie mit der rechten Maustaste auf **MySpec**, und wählen Sie dann **Delete (Löschen)** aus. Sie benötigen diese Datei nicht für die Anwendung.
  
10. In den folgenden Schritten wird die Datei „pom.xml“ aktualisiert, um die Abhängigkeiten für die Spark Scala-Anwendung zu definieren. Damit diese Abhängigkeiten automatisch heruntergeladen und aufgelöst werden, muss Maven entsprechend konfiguriert sein.
   
    ![Maven für automatische Downloads konfigurieren](./media/apache-spark-create-standalone-application/configure-maven.png)
   
   1. Wählen Sie im Menü **Datei** die Option **Einstellungen**.
   2. Navigieren Sie im Dialogfeld **Einstellungen** zu **Erstellung, Ausführung, Bereitstellung** > **Buildtools** > **Maven** > **Importieren**.
   3. Aktivieren Sie das Kontrollkästchen **Maven-Projekte automatisch importieren**.
   4. Klicken Sie auf **Apply** (Anwenden) und dann auf **OK**.
11. Wählen Sie im linken Bereich **src > main > scala > com.microsoft.spark.example** aus, und doppelklicken Sie dann auf **App**, um „App.scala“ zu öffnen.

12. Ersetzen Sie den vorhandenen Beispielcode durch den folgenden Code, und speichern Sie die Änderungen. Dieser Code liest die Daten aus der Datei „HVAC.csv“ (für alle HDInsight Spark-Cluster verfügbar), ruft die Zeilen ab, die nur eine Ziffer in der sechsten Spalte enthalten, und schreibt die Ausgabe in **/HVACOut** unter dem Standardspeichercontainer für den Cluster.

        package com.microsoft.spark.example
   
        import org.apache.spark.SparkConf
        import org.apache.spark.SparkContext
   
        /**
          * Test IO to wasb
          */
        object WasbIOTest {
          def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("WASBIOTest")
            val sc = new SparkContext(conf)
   
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
   
            //find the rows which have only one digit in the 7th column in the CSV
            val rdd1 = rdd.filter(s => s.split(",")(6).length() == 1)
   
            rdd1.saveAsTextFile("wasb:///HVACout")
          }
        }
13. Doppelklicken Sie im linken Bereich auf **pom.xml**.
   
   1. Fügen Sie in `<project>\<properties>` folgende Segmente hinzu:
      
          <scala.version>2.10.4</scala.version>
          <scala.compat.version>2.10.4</scala.compat.version>
          <scala.binary.version>2.10</scala.binary.version>
   2. Fügen Sie in `<project>\<dependencies>` folgende Segmente hinzu:
      
           <dependency>
             <groupId>org.apache.spark</groupId>
             <artifactId>spark-core_${scala.binary.version}</artifactId>
             <version>1.4.1</version>
           </dependency>
      
      Speichern Sie die Änderungen in „pom.xml“.
10. Erstellen Sie die JAR-Datei. IntelliJ IDEA ermöglicht die Erstellung von JAR als Projektartefakt. Führen Sie die folgenden Schritte aus:
    
    1. Wählen Sie im Menü **File** (Datei) die Option **Project Structure** (Projektstruktur) aus.
    2. Wählen Sie im Dialogfeld **Project Structure (Projektstruktur)** die Option **Artifacts (Artefakte)** und anschließend das Pluszeichen aus. Wählen Sie im Popupdialogfeld **JAR** und anschließend **From modules with dependencies (Aus Modulen mit Abhängigkeiten)** aus.
       
        ![Erstellen einer JAR-Datei](./media/apache-spark-create-standalone-application/create-jar-1.png)
    3. Wählen Sie im Dialogfeld **Create JAR from Modules (JAR aus Modulen erstellen)** die Auslassungspunkte (![...](./media/apache-spark-create-standalone-application/ellipsis.png)) für die **Main Class (Hauptklasse)** aus.
    4. Wählen Sie im Dialogfeld **Select Main Class (Hauptklasse auswählen)** die standardmäßig angezeigte Klasse und anschließend **OK** aus.
       
        ![Erstellen einer JAR-Datei](./media/apache-spark-create-standalone-application/create-jar-2.png)
    5. Stellen Sie im Dialogfeld **Create JAR from Modules (JAR aus Modulen erstellen)** sicher, dass die Option zum **Extrahieren in die JAR-Zieldatei** aktiviert ist, und wählen Sie anschließend **OK** aus.  Mit dieser Einstellung wird eine einzelne JAR-Datei mit allen Abhängigkeiten erstellt.
       
        ![Erstellen einer JAR-Datei](./media/apache-spark-create-standalone-application/create-jar-3.png)
    6. Die Ausgabelayout-Registerkarte führt alle JAR-Dateien des Maven-Projekts auf. Sie können die Dateien auswählen und löschen, zu denen die Scala-Anwendung keine direkte Abhängigkeit hat. Bei der hier erstellten Anwendung können Sie alle bis auf die letzte (**SparkSimpleApp-Kompilierungsausgabe**) entfernen. Wählen Sie die zu löschenden JAR-Dateien und anschließend das Symbol **Löschen** aus.
       
        ![Erstellen einer JAR-Datei](./media/apache-spark-create-standalone-application/delete-output-jars.png)
       
        Vergewissern Sie sich, dass das Kontrollkästchen **Include in project build (In Projektbuild einbeziehen)** aktiviert ist, damit die JAR-Datei bei jeder Projekterstellung oder -aktualisierung erstellt wird. Wählen Sie **Übernehmen** und dann **OK** aus.
    7. Wählen Sie im Menü **Build (Erstellen)** die Option **Build Artifacts (Artefakte erstellen)** aus, um die JAR-Datei zu erstellen. Die JAR-Ausgabe wird unter **\out\artifacts** erstellt.
       
        ![Erstellen einer JAR-Datei](./media/apache-spark-create-standalone-application/output.png)

## <a name="run-the-application-on-the-spark-cluster"></a>Ausführen der Anwendung im Spark-Cluster
Sie können die folgenden Ansätze nutzen, um die Anwendung im Cluster auszuführen:

* **Kopieren Sie die JAR-Anwendungsdatei in das dem Cluster zugeordnete Azure-Speicher-BLOB**. Hierzu können Sie das Befehlszeilenprogramm [**AzCopy**](../../storage/common/storage-use-azcopy.md) verwenden. Daneben gibt es aber auch noch zahlreiche andere Clients, die Sie zum Hochladen von Daten verwenden können. Weitere Informationen finden Sie unter [Hochladen von Daten für Hadoop-Aufträge in HDInsight](../hdinsight-upload-data.md).
* **Verwenden Sie Livy, um einen Anwendungsauftrag remote an den Spark-Cluster zu übermitteln**. Spark-Cluster in HDInsight enthalten Livy, um REST-Endpunkte für die Remoteübermittlung von Spark-Aufträgen verfügbar zu machen. Weitere Informationen finden Sie unter [Remoteübermittlung von Spark-Aufträgen unter Verwendung von Livy mit Spark-Clustern in HDInsight](apache-spark-livy-rest-interface.md).

## <a name="next-step"></a>Nächster Schritt

In diesem Artikel haben Sie gelernt, wie eine Spark Scala-Anwendung erstellt wird. Im nächsten Artikel erfahren Sie, wie Sie diese Anwendung in einem HDInsight Spark-Cluster mit Livy ausführen.

> [!div class="nextstepaction"]
>[Remoteausführung von Aufträgen in einem Spark-Cluster mithilfe von Livy](./apache-spark-livy-rest-interface.md)

