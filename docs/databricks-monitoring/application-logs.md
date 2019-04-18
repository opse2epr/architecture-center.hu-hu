---
title: Azure Databricks-alkalmazásnaplók küldése az Azure Monitornak
description: Egyéni naplók és mérőszámok küldése az Azure Databricks az Azure monitornak
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: ea67122d7871663e8aaf42b7af0043492f63b6b1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639185"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a>Azure Databricks-alkalmazásnaplók küldése az Azure Monitornak

Ez a cikk bemutatja, hogyan alkalmazás naplók és mérőszámok küldése az Azure Databrickshez, amely egy [Log Analytics-munkaterület](/azure/azure-monitor/platform/manage-access). Használja a [Azure Databricks-figyelő könyvtár](https://github.com/mspnp/spark-monitoring), amely is elérhető a Githubon.

## <a name="prerequisites"></a>Előfeltételek

Az Azure Databricks-fürt az ügyfélfigyelési könyvtár használatához konfigurálja a leírtak szerint [konfigurálása az Azure Databricks mérőszámok küldése az Azure monitornak][config-cluster].

> [!NOTE]
> Az ügyfélfigyelési könyvtár hibaszintű eseményeket az Apache Spark és a Spark strukturált Stream mérőszámainak a feladatok az Azure monitornak streameli. Ne módosítsa az alkalmazás kódjában ezen események és metrikák nincs szükségünk.

## <a name="send-application-metrics-using-dropwizard"></a>Alkalmazásmetrikák Dropwizard használatával küldése

A Spark egy konfigurálható metrikák rendszert, a Dropwizard metrikák kódtár alapján használja. További információkért lásd: [metrikák](https://spark.apache.org/docs/latest/monitoring.html#metrics) a Spark-dokumentáció.

Azure Databricks alkalmazáskód Azure Monitor, alkalmazásmetrikák elküldéséhez kövesse az alábbi lépéseket:

1. Hozhat létre a **spark-figyelők-loganalytics – 1.0-SNAPSHOT.jar** JAR-fájlt a [összeállítása az Azure Databricks-figyelő könyvtár][build-lib].

1. Hozzon létre Dropwizard [mérőműszer vagy számlálók](https://metrics.dropwizard.io/4.0.0/manual/core.html) az alkalmazás kódjában. Használhatja a `UserMetricsSystem` az ügyfélfigyelési könyvtár meghatározott osztály. A következő példában létrehozunk egy számláló nevű `counter1`.

    ```Scala
    import org.apache.spark.metrics.UserMetricsSystems
    import org.apache.spark.sql.SparkSession

    object StreamingQueryListenerSampleJob  {

      private final val METRICS_NAMESPACE = "samplejob"
      private final val COUNTER_NAME = "counter1"

      def main(args: Array[String]): Unit = {

        val spark = SparkSession
          .builder
          .getOrCreate

        val driverMetricsSystem = UserMetricsSystems
            .getMetricSystem(METRICS_NAMESPACE, builder => {
              builder.registerCounter(COUNTER_NAME)
            })

        driverMetricsSystem.counter(COUNTER_NAME).inc(5)
      }
    }
    ```

    Az ügyfélfigyelési könyvtár tartalmaz egy [mintaalkalmazás] [ sample-app] , amely bemutatja, hogyan használható a `UserMetricsSystem` osztály.

## <a name="send-application-logs-using-log4j"></a>Protokoly aplikací Log4j használatával küldése

Az Azure Log Analytics használatával az Azure Databricks alkalmazásnaplókat küldhet a [Log4j naplóírói](https://logging.apache.org/log4j/2.x/manual/appenders.html) a könyvtárban, kövesse az alábbi lépéseket:

1. Hozhat létre a **spark-figyelők-loganalytics – 1.0-SNAPSHOT.jar** JAR-fájlt a [összeállítása az Azure Databricks-figyelő könyvtár][build-lib].

1. Hozzon létre egy **log4j.properties** [konfigurációs fájl](https://logging.apache.org/log4j/2.x/manual/configuration.html) az alkalmazáshoz. A következő konfigurációs tulajdonságokat tartalmazza. Helyettesítse be az alkalmazáscsomag neve és a naplózási szint felsoroltak közül:

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    A minta konfigurációs fájlt annak [Itt][log4j.properties].

1. Az alkalmazás kódjában közé tartozik a **spark-figyelők-loganalytics** projektre, és importálja `com.microsoft.pnp.logging.Log4jconfiguration` az alkalmazás kódjának.

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. Konfigurálás a Log4j használatával a **log4j.properties** a 3. lépésben létrehozott fájlt:

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. Adja hozzá az Apache Spark naplóüzenetek a kódban szükség szerint a megfelelő szinten. Például a `logDebug` metódus hibakeresési napló üzenet küldése. További információkért lásd: [naplózás] [ spark-logging] a Spark-dokumentáció.

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a>A mintaalkalmazás futtatása

Az ügyfélfigyelési könyvtár tartalmaz egy [mintaalkalmazás] [ sample-app] , amely bemutatja, hogyan lehet az Azure Monitor, alkalmazásmetrikák és alkalmazásnaplókat küldjön. A minta futtatása:

1. Hozhat létre a **spark-feladatok** leírtak szerint az ügyfélfigyelési könyvtár projekten [összeállítása az Azure Databricks-figyelő könyvtár][build-lib].

1. Keresse meg a Databricks-munkaterület, és hozzon létre egy új feladatot leírt módon [Itt](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).

1. A feladat részletei lapon válassza ki a **beállítása JAR**.

1. Töltse fel a JAR-fájlt a `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.

1. A **Main osztály**, adja meg `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.

1. Válasszon egy fürtöt, amely már konfigurálva van az ügyfélfigyelési könyvtár használatához. Lásd: [konfigurálása az Azure Databricks mérőszámok küldése az Azure monitornak][config-cluster].

A feladat futtatásakor az alkalmazás naplókat és mérőszámokat megtekintheti az a Log Analytics-munkaterületen.

Protokoly aplikací SparkLoggingEvent_CL jelennek meg:

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

Alkalmazásmetrikák SparkMetric_CL jelennek meg:

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a>További lépések

Helyezze üzembe a Teljesítményfigyelő irányítópult, amely ezt a kódot könyvtárat kapcsolatos problémák elhárítása az Azure Databricks éles számítási feladatokat társul.

> [!div class="nextstepaction"]
> [Irányítópultok használata az Azure Databricks-metrikák megjelenítésére](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html