---
title: Az Azure Databricks mérőszámok küldése az Azure Monitor konfigurálása
description: Engedélyezze a mérőszámok figyelését, és naplózási adatok az Azure Log Analyticsben scala kódtár
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: af6b6433f87964ac60c179ecf498e54129344126
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503309"
---
<!-- markdownlint-disable MD040 -->

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a>Az Azure Databricks mérőszámok küldése az Azure Monitor konfigurálása

Ez a cikk bemutatja, hogyan konfigurálhatja egy Azure Databricks-fürtön való mérőszámok küldése egy [Log Analytics-munkaterület](/azure/azure-monitor/platform/manage-access). Használja a [Azure Databricks-figyelő könyvtár](https://github.com/mspnp/spark-monitoring), amely is elérhető a Githubon. Ismeri a Java, a Scala és a Maven használata akkor javasolt, mint prerequisistes.

## <a name="about-the-azure-databricks-monitoring-library"></a>Tudnivalók az Azure Databricks-tár figyelése

A [GitHub-adattárat](https://github.com/mspnp/spark-monitoring) az Azure Databricks-figyelő könyvtár tartalmaz a következő könyvtárstruktúrát:

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

A **spark-feladatok** könyvtár egy Spark-mintaalkalmazás bemutatja, hogyan lehet megvalósítani egy Spark-alkalmazás metrika számláló.

A **spark-figyelők** könyvtár, amely lehetővé teszi az Azure Databrickshez, amely az Apache Spark események küldése az Azure Log Analytics-munkaterület szolgáltatási szintű funkciókat tartalmaz. Az Azure Databricks egy olyan szolgáltatás,-alapú Apache Spark, amely strukturált API-k kötegelve dolgozzuk adatkészleteket, adatkerettípusokat jelölhet, és az SQL használata az adatok egy készletét tartalmazza. Az Apache Spark 2.0-val, váltak [strukturált Stream](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), egy adatfolyam-feldolgozás API, a Spark a batch API-kat feldolgozó épül.

A **spark-figyelők-loganalytics** könyvtár tartalmaz egy fogadó a Spark figyelői, egy DropWizard fogadóját és egy ügyfél az Azure Log Analytics-munkaterületet. Ez a könyvtár a log4j Naplóírót a az Apache Spark alkalmazásnaplókat is tartalmaz.

A **spark-figyelők-loganalytics** és **spark-figyelők** a könyvtárak tartalmazzák a kódot, amellyel a két JAR-fájlt, amelyre telepítve vannak a Databricks-fürtön. A **spark-figyelők** könyvtár tartalmaz egy **parancsfájlok** egy fürt csomópont inicializálása szkript másolja a fájlt tartalmazó könyvtárba fájlok az átmeneti könyvtár, az Azure Databricks fájlrendszer végrehajtási csomópontok.

A **pom.xml** a teljes projekt fő Maven build fájlt.

## <a name="prerequisites"></a>Előfeltételek

Első lépésként telepítse a következő Azure-erőforrások:

- Azure Databricks-munkaterületet. Lásd: [a rövid útmutató: Spark-feladatok futtatása Azure databricksen az Azure portal használatával](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).
- Egy Log Analytics-munkaterület. Lásd: [Log Analytics-munkaterület létrehozása az Azure Portalon](/azure/azure-monitor/learn/quick-create-workspace).

Ezután telepítse a [Azure Databricks parancssori felület](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli). Az Azure Databricks személyes hozzáférési jogkivonat szükséges a CLI használatával. Útmutatásért lásd: [egy jogkivonatot,](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).

Keresse meg a Log Analytics-munkaterület Azonosítójára és kulcsára. Útmutatásért lásd: [szerezze be a munkaterület Azonosítójára és kulcsára](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key). A Databricks-fürt konfigurálásakor szüksége lesz ezekre az értékekre.

Az ügyfélfigyelési könyvtár hozhat létre, a Java ide Környezethez, a következő források lesz szüksége:

- [Java fejlesztői készlet (JDK) 1.8-as verzió](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Scala nyelvi SDK 2.11-et](https://www.scala-lang.org/download/)
- [Az Apache Maven 3.5.4](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a>Hozhat létre az Azure Databricks-tár figyelése

Az Azure Databricks-figyelő könyvtár létrehozásához hajtsa végre az alábbi lépéseket:

1. Klónozza, ágaztassa vagy töltse le a [GitHub-adattár](https://github.com/mspnp/spark-monitoring).

1. Importálja a Maven project objektum modellfájl _pom.xml_található, a **/src** mappát a projektbe a Finderből. Ezzel importálja a három projekt:

    - Spark-feladatok
    - Spark-figyelők
    - spark-listeners-loganalytics

1. Hajtsa végre a Maven **csomag** build fázis hozhat létre a JAR-fájlok az egyes projektek a Java IDE-ben:

    |Project| JAR-fájlt|
    |-------|---------|
    |Spark-feladatok|Spark-feladatok – 1.0-SNAPSHOT.jar|
    |Spark-figyelők|Spark-figyelők – 1.0-SNAPSHOT.jar|
    |spark-listeners-loganalytics|spark-listeners-loganalytics-1.0-SNAPSHOT.jar|

1. Az Azure Databricks parancssori felület használatával hozzon létre egy könyvtárat nevű **dbfs: / databricks és figyelési-előkészítési**:  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. Az Azure Databricks parancssori felület használatával másolhatja **/src/spark-listeners/scripts/listeners.sh** korábban a könyvtárba:

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. Az Azure Databricks parancssori felület használatával másolhatja **/src/spark-listeners/scripts/metrics.properties** a korábban létrehozott könyvtárba:

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. Az Azure Databricks parancssori felület használatával másolhatja **spark-figyelők – 1.0-SNAPSHOT.jar** és **spark-figyelők-loganalytics – 1.0-SNAPSHOT.jar** a korábban létrehozott könyvtárba:

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a>Létrehozhat és konfigurálhat egy Azure Databricks-fürt

Létrehozni és konfigurálni egy Azure Databricks-fürt, kövesse az alábbi lépéseket:

1. Keresse meg az Azure Databricks-munkaterület az Azure Portalon.
1. A kezdőlapon kattintson **új fürt**.
1. Adja meg a fürt nevét a **fürtnév** szövegmezőben.
1. Az a **Databricks futtatókörnyezet-verziója** legördülő menüben válassza **4.3** vagy nagyobb.
1. Alatt **speciális beállítások**, kattintson a **Spark** fülre. Adja meg a következő név-érték párok az a **Spark Config** szövegbeviteli mező:

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. Miközben továbbra is a a **Spark** lapra, adja meg a következő értékeket a **környezeti változók** szövegbeviteli mező:

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Databricks felhasználói felület képernyőképe](./_images/create-cluster1.png)

1. Miközben továbbra is a a **speciális beállítások** területén kattintson a a **Init parancsfájlok** fülre.
1. Az a **cél** legördülő menüben válassza **DBFS**.
1. A **Init parancsfájl elérési útján**, adja meg `dbfs:/databricks/monitoring-staging/listeners.sh`.
1. Kattintson a **Hozzáadás** parancsra.

    ![Databricks felhasználói felület képernyőképe](./_images/create-cluster2.png)

1. Kattintson a **-fürt létrehozása** a fürt létrehozásához.

## <a name="view-metrics"></a>Metrikák megtekintése

Miután végrehajtotta ezeket a lépéseket, a Databricks-fürt bizonyos metrikaadatok kapcsolatos magához a fürthöz az Azure monitornak elemzésének lehetőségeit. A naplóadatok érhető el az Azure Log Analytics-munkaterület alatt az "Active |} Egyéni naplókkal |} A séma SparkMetric_CL". Naplózott mérőszámok típusaival kapcsolatos további információkért lásd: [metrikák Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) Dropwizard dokumentációjában. A metrika típusa, például a mérőműszer, számláló vagy hisztogram, írja be a Type mezőjében.

A kódtár emellett hibaszintű eseményeket az Apache Spark és a Spark strukturált Stream mérőszámainak a feladatok az Azure monitornak streameli. Ne módosítsa az alkalmazás kódjában ezen események és metrikák nincs szükségünk. A Spark strukturált Stream naplóadatok érhető el az "Active |} Egyéni naplókkal |} A séma SparkListenerEvent_CL".

![Képernyőkép a Log Analytics-munkaterület](./_images/workspace.png)

Naplók megtekintése kapcsolatos további információkért lásd: [megtekintése és elemzése az Azure monitorban naplóadatok](/azure/azure-monitor/log-query/portals).

## <a name="next-steps"></a>További lépések

Ez a cikk ismerteti a mérőszámok küldése az Azure monitornak a fürt konfigurálása. A következő cikk bemutatja, hogyan küldhet, alkalmazásmetrikák és a naplók az Azure Databricks-figyelő könyvtár használatához.

> [!div class="nextstepaction"]
> [Protokoly aplikací küldése az Azure monitornak](./application-logs.md)
