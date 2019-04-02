---
title: Az Azure Databricks alkalmazásnaplókat küldése az Azure monitornak
description: Egyéni naplók és mérőszámok küldése az Azure Databricks az Azure monitornak
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 49c631687fb3e3bbd807ffbbb49d9c5f6526bfb4
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503308"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a><span data-ttu-id="e0465-103">Az Azure Databricks alkalmazásnaplókat küldése az Azure monitornak</span><span class="sxs-lookup"><span data-stu-id="e0465-103">Send Azure Databricks application logs to Azure Monitor</span></span>

<span data-ttu-id="e0465-104">Ez a cikk bemutatja, hogyan alkalmazás naplók és mérőszámok küldése az Azure Databrickshez, amely egy [Log Analytics-munkaterület](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="e0465-104">This article shows how to send application logs and metrics from Azure Databricks to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="e0465-105">Használja a [Azure Databricks-figyelő könyvtár](https://github.com/mspnp/spark-monitoring), amely is elérhető a Githubon.</span><span class="sxs-lookup"><span data-stu-id="e0465-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e0465-106">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="e0465-106">Prerequisites</span></span>

<span data-ttu-id="e0465-107">Az Azure Databricks-fürt az ügyfélfigyelési könyvtár használatához konfigurálja a leírtak szerint [konfigurálása az Azure Databricks mérőszámok küldése az Azure monitornak][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="e0465-107">Configure your Azure Databricks cluster to use the monitoring library, as described in [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

> [!NOTE]
> <span data-ttu-id="e0465-108">Az ügyfélfigyelési könyvtár hibaszintű eseményeket az Apache Spark és a Spark strukturált Stream mérőszámainak a feladatok az Azure monitornak streameli.</span><span class="sxs-lookup"><span data-stu-id="e0465-108">The monitoring library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="e0465-109">Ne módosítsa az alkalmazás kódjában ezen események és metrikák nincs szükségünk.</span><span class="sxs-lookup"><span data-stu-id="e0465-109">You don't need to make any changes to your application code for these events and metrics.</span></span>

## <a name="send-application-metrics-using-dropwizard"></a><span data-ttu-id="e0465-110">Alkalmazásmetrikák Dropwizard használatával küldése</span><span class="sxs-lookup"><span data-stu-id="e0465-110">Send application metrics using Dropwizard</span></span>

<span data-ttu-id="e0465-111">A Spark egy konfigurálható metrikák rendszert, a Dropwizard metrikák kódtár alapján használja.</span><span class="sxs-lookup"><span data-stu-id="e0465-111">Spark uses a configurable metrics system based on the Dropwizard Metrics Library.</span></span> <span data-ttu-id="e0465-112">További információkért lásd: [metrikák](https://spark.apache.org/docs/latest/monitoring.html#metrics) a Spark-dokumentáció.</span><span class="sxs-lookup"><span data-stu-id="e0465-112">For more information, see [Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics) in the Spark documentation.</span></span>

<span data-ttu-id="e0465-113">Azure Databricks alkalmazáskód Azure Monitor, alkalmazásmetrikák elküldéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="e0465-113">To send application metrics from Azure Databricks application code to Azure Monitor, follow these steps:</span></span>

1. <span data-ttu-id="e0465-114">Hozhat létre a **spark-figyelők-loganalytics – 1.0-SNAPSHOT.jar** JAR-fájlt a [összeállítása az Azure Databricks-figyelő könyvtár][build-lib].</span><span class="sxs-lookup"><span data-stu-id="e0465-114">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="e0465-115">Hozzon létre Dropwizard [mérőműszer vagy számlálók](https://metrics.dropwizard.io/4.0.0/manual/core.html) az alkalmazás kódjában.</span><span class="sxs-lookup"><span data-stu-id="e0465-115">Create Dropwizard [gauges or counters](https://metrics.dropwizard.io/4.0.0/manual/core.html) in your application code.</span></span> <span data-ttu-id="e0465-116">Használhatja a `UserMetricsSystem` az ügyfélfigyelési könyvtár meghatározott osztály.</span><span class="sxs-lookup"><span data-stu-id="e0465-116">You can use the `UserMetricsSystem` class defined in the monitoring library.</span></span> <span data-ttu-id="e0465-117">A következő példában létrehozunk egy számláló nevű `counter1`.</span><span class="sxs-lookup"><span data-stu-id="e0465-117">The following example creates a counter named `counter1`.</span></span>

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

    <span data-ttu-id="e0465-118">Az ügyfélfigyelési könyvtár tartalmaz egy [mintaalkalmazás] [ sample-app] , amely bemutatja, hogyan használható a `UserMetricsSystem` osztály.</span><span class="sxs-lookup"><span data-stu-id="e0465-118">The monitoring library includes a [sample application][sample-app] that demonstrates how to use the `UserMetricsSystem` class.</span></span>

## <a name="send-application-logs-using-log4j"></a><span data-ttu-id="e0465-119">Protokoly aplikací Log4j használatával küldése</span><span class="sxs-lookup"><span data-stu-id="e0465-119">Send application logs using Log4j</span></span>

<span data-ttu-id="e0465-120">Az Azure Log Analytics használatával az Azure Databricks alkalmazásnaplókat küldhet a [Log4j naplóírói](https://logging.apache.org/log4j/2.x/manual/appenders.html) a könyvtárban, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="e0465-120">To send your Azure Databricks application logs to Azure Log Analytics using the [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) in the library, follow these steps:</span></span>

1. <span data-ttu-id="e0465-121">Hozhat létre a **spark-figyelők-loganalytics – 1.0-SNAPSHOT.jar** JAR-fájlt a [összeállítása az Azure Databricks-figyelő könyvtár][build-lib].</span><span class="sxs-lookup"><span data-stu-id="e0465-121">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="e0465-122">Hozzon létre egy **log4j.properties** [konfigurációs fájl](https://logging.apache.org/log4j/2.x/manual/configuration.html) az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="e0465-122">Create a **log4j.properties** [configuration file](https://logging.apache.org/log4j/2.x/manual/configuration.html) for your application.</span></span> <span data-ttu-id="e0465-123">A következő konfigurációs tulajdonságokat tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="e0465-123">Include the following configuration properties.</span></span> <span data-ttu-id="e0465-124">Helyettesítse be az alkalmazáscsomag neve és a naplózási szint felsoroltak közül:</span><span class="sxs-lookup"><span data-stu-id="e0465-124">Substitute your application package name and log level where indicated:</span></span>

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    <span data-ttu-id="e0465-125">A minta konfigurációs fájlt annak [Itt][log4j.properties].</span><span class="sxs-lookup"><span data-stu-id="e0465-125">You can find a sample configuration file [here][log4j.properties].</span></span>

1. <span data-ttu-id="e0465-126">Az alkalmazás kódjában közé tartozik a **spark-figyelők-loganalytics** projektre, és importálja `com.microsoft.pnp.logging.Log4jconfiguration` az alkalmazás kódjának.</span><span class="sxs-lookup"><span data-stu-id="e0465-126">In your application code, include the **spark-listeners-loganalytics** project, and import `com.microsoft.pnp.logging.Log4jconfiguration` to your application code.</span></span>

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. <span data-ttu-id="e0465-127">Konfigurálás a Log4j használatával a **log4j.properties** a 3. lépésben létrehozott fájlt:</span><span class="sxs-lookup"><span data-stu-id="e0465-127">Configure Log4j using the **log4j.properties** file you created in step 3:</span></span>

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. <span data-ttu-id="e0465-128">Adja hozzá az Apache Spark naplóüzenetek a kódban szükség szerint a megfelelő szinten.</span><span class="sxs-lookup"><span data-stu-id="e0465-128">Add Apache Spark log messages at the appropriate level in your code as required.</span></span> <span data-ttu-id="e0465-129">Például a `logDebug` metódus a hibakeresési napló meesage küldéséhez.</span><span class="sxs-lookup"><span data-stu-id="e0465-129">For example, use the `logDebug` method to send a debug log meesage.</span></span> <span data-ttu-id="e0465-130">További információkért lásd: [naplózás] [ spark-logging] a Spark-dokumentáció.</span><span class="sxs-lookup"><span data-stu-id="e0465-130">For more information, see [Logging][spark-logging] in the Spark documentation.</span></span>

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a><span data-ttu-id="e0465-131">A mintaalkalmazás futtatása</span><span class="sxs-lookup"><span data-stu-id="e0465-131">Run the sample application</span></span>

<span data-ttu-id="e0465-132">Az ügyfélfigyelési könyvtár tartalmaz egy [mintaalkalmazás] [ sample-app] , amely bemutatja, hogyan lehet az Azure Monitor, alkalmazásmetrikák és alkalmazásnaplókat küldjön.</span><span class="sxs-lookup"><span data-stu-id="e0465-132">The monitoring library includes a [sample application][sample-app] that demonstrates how to send both application metrics and application logs to Azure Monitor.</span></span> <span data-ttu-id="e0465-133">A minta futtatása:</span><span class="sxs-lookup"><span data-stu-id="e0465-133">To run the sample:</span></span>

1. <span data-ttu-id="e0465-134">Hozhat létre a **spark-feladatok** leírtak szerint az ügyfélfigyelési könyvtár projekten [összeállítása az Azure Databricks-figyelő könyvtár][build-lib].</span><span class="sxs-lookup"><span data-stu-id="e0465-134">Build the **spark-jobs** project in the monitoring library, as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="e0465-135">Keresse meg a Databricks-munkaterület, és hozzon létre egy új feladatot leírt módon [Itt](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span><span class="sxs-lookup"><span data-stu-id="e0465-135">Navigate to your Databricks workspace and create a new job, as described [here](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span></span>

1. <span data-ttu-id="e0465-136">A feladat részletei lapon válassza ki a **beállítása JAR**.</span><span class="sxs-lookup"><span data-stu-id="e0465-136">In the job detail page, select **Set JAR**.</span></span>

1. <span data-ttu-id="e0465-137">Töltse fel a JAR-fájlt a `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span><span class="sxs-lookup"><span data-stu-id="e0465-137">Upload the JAR file from `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span></span>

1. <span data-ttu-id="e0465-138">A **Main osztály**, adja meg `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span><span class="sxs-lookup"><span data-stu-id="e0465-138">For **Main class**, enter `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span></span>

1. <span data-ttu-id="e0465-139">Válasszon egy fürtöt, amely már konfigurálva van az ügyfélfigyelési könyvtár használatához.</span><span class="sxs-lookup"><span data-stu-id="e0465-139">Select a cluster that is already configured to use the monitoring library.</span></span> <span data-ttu-id="e0465-140">Lásd: [konfigurálása az Azure Databricks mérőszámok küldése az Azure monitornak][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="e0465-140">See [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

<span data-ttu-id="e0465-141">A feladat futtatásakor az alkalmazás naplókat és mérőszámokat megtekintheti az a Log Analytics-munkaterületen.</span><span class="sxs-lookup"><span data-stu-id="e0465-141">When the job runs, you can view the application logs and metrics in your Log Analytics workspace.</span></span>

<span data-ttu-id="e0465-142">Protokoly aplikací SparkLoggingEvent_CL jelennek meg:</span><span class="sxs-lookup"><span data-stu-id="e0465-142">Application logs appear under SparkLoggingEvent_CL:</span></span>

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

<span data-ttu-id="e0465-143">Alkalmazásmetrikák SparkMetric_CL jelennek meg:</span><span class="sxs-lookup"><span data-stu-id="e0465-143">Application metrics appear under SparkMetric_CL:</span></span>

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a><span data-ttu-id="e0465-144">További lépések</span><span class="sxs-lookup"><span data-stu-id="e0465-144">Next steps</span></span>

<span data-ttu-id="e0465-145">Helyezze üzembe a Teljesítményfigyelő irányítópult, amely ezt a kódot könyvtárat kapcsolatos problémák elhárítása az Azure Databricks éles számítási feladatokat társul.</span><span class="sxs-lookup"><span data-stu-id="e0465-145">Deploy the performance monitoring dashboard that accompanies this code library to troubleshoot performance issues in your production Azure Databricks workloads.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="e0465-146">Az irányítópultok használatával jelenítheti meg az Azure Databricks-metrikák</span><span class="sxs-lookup"><span data-stu-id="e0465-146">Use dashboards to visualize Azure Databricks metrics</span></span>](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html