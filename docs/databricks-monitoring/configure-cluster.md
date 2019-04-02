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

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a><span data-ttu-id="0dfa1-103">Az Azure Databricks mérőszámok küldése az Azure Monitor konfigurálása</span><span class="sxs-lookup"><span data-stu-id="0dfa1-103">Configure Azure Databricks to send metrics to Azure Monitor</span></span>

<span data-ttu-id="0dfa1-104">Ez a cikk bemutatja, hogyan konfigurálhatja egy Azure Databricks-fürtön való mérőszámok küldése egy [Log Analytics-munkaterület](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-104">This article shows how to configure an Azure Databricks cluster to send metrics to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="0dfa1-105">Használja a [Azure Databricks-figyelő könyvtár](https://github.com/mspnp/spark-monitoring), amely is elérhető a Githubon.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span> <span data-ttu-id="0dfa1-106">Ismeri a Java, a Scala és a Maven használata akkor javasolt, mint prerequisistes.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-106">Understanding of Java, Scala, and Maven are recommended as prerequisistes.</span></span>

## <a name="about-the-azure-databricks-monitoring-library"></a><span data-ttu-id="0dfa1-107">Tudnivalók az Azure Databricks-tár figyelése</span><span class="sxs-lookup"><span data-stu-id="0dfa1-107">About the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="0dfa1-108">A [GitHub-adattárat](https://github.com/mspnp/spark-monitoring) az Azure Databricks-figyelő könyvtár tartalmaz a következő könyvtárstruktúrát:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-108">The [GitHub repo](https://github.com/mspnp/spark-monitoring) for the Azure Databricks Monitoring Library has following directory structure:</span></span>

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

<span data-ttu-id="0dfa1-109">A **spark-feladatok** könyvtár egy Spark-mintaalkalmazás bemutatja, hogyan lehet megvalósítani egy Spark-alkalmazás metrika számláló.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-109">The **spark-jobs** directory is a sample Spark application demonstrating how to implement a Spark application metric counter.</span></span>

<span data-ttu-id="0dfa1-110">A **spark-figyelők** könyvtár, amely lehetővé teszi az Azure Databrickshez, amely az Apache Spark események küldése az Azure Log Analytics-munkaterület szolgáltatási szintű funkciókat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-110">The **spark-listeners** directory includes functionality that enables Azure Databricks to send Apache Spark events at the service level to an Azure Log Analytics workspace.</span></span> <span data-ttu-id="0dfa1-111">Az Azure Databricks egy olyan szolgáltatás,-alapú Apache Spark, amely strukturált API-k kötegelve dolgozzuk adatkészleteket, adatkerettípusokat jelölhet, és az SQL használata az adatok egy készletét tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-111">Azure Databricks is a service based on Apache Spark, which includes a set of structured APIs for batch processing data using Datasets, DataFrames, and SQL.</span></span> <span data-ttu-id="0dfa1-112">Az Apache Spark 2.0-val, váltak [strukturált Stream](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), egy adatfolyam-feldolgozás API, a Spark a batch API-kat feldolgozó épül.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-112">With Apache Spark 2.0, support was added for [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), a data stream processing API built on Spark's batch processing APIs.</span></span>

<span data-ttu-id="0dfa1-113">A **spark-figyelők-loganalytics** könyvtár tartalmaz egy fogadó a Spark figyelői, egy DropWizard fogadóját és egy ügyfél az Azure Log Analytics-munkaterületet.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-113">The **spark-listeners-loganalytics** directory includes a sink for Spark listeners, a sink for DropWizard, and a client for an Azure Log Analytics Workspace.</span></span> <span data-ttu-id="0dfa1-114">Ez a könyvtár a log4j Naplóírót a az Apache Spark alkalmazásnaplókat is tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-114">This directory also includes a log4j Appender for your Apache Spark application logs.</span></span>

<span data-ttu-id="0dfa1-115">A **spark-figyelők-loganalytics** és **spark-figyelők** a könyvtárak tartalmazzák a kódot, amellyel a két JAR-fájlt, amelyre telepítve vannak a Databricks-fürtön.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-115">The **spark-listeners-loganalytics** and **spark-listeners** directories contain the code for building the two JAR files that are deployed to the Databricks cluster.</span></span> <span data-ttu-id="0dfa1-116">A **spark-figyelők** könyvtár tartalmaz egy **parancsfájlok** egy fürt csomópont inicializálása szkript másolja a fájlt tartalmazó könyvtárba fájlok az átmeneti könyvtár, az Azure Databricks fájlrendszer végrehajtási csomópontok.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-116">The **spark-listeners** directory includes a **scripts** directory that contains a cluster node initialization script to copy the JAR files from a staging directory in the Azure Databricks file system to execution nodes.</span></span>

<span data-ttu-id="0dfa1-117">A **pom.xml** a teljes projekt fő Maven build fájlt.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-117">The **pom.xml** file is the main Maven build file for the entire project.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="0dfa1-118">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="0dfa1-118">Prerequisites</span></span>

<span data-ttu-id="0dfa1-119">Első lépésként telepítse a következő Azure-erőforrások:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-119">To get started, deploy the following Azure resources:</span></span>

- <span data-ttu-id="0dfa1-120">Azure Databricks-munkaterületet.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-120">An Azure Databricks workspace.</span></span> <span data-ttu-id="0dfa1-121">Lásd: [a rövid útmutató: Spark-feladatok futtatása Azure databricksen az Azure portal használatával](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-121">See [Quickstart: Run a Spark job on Azure Databricks using the Azure portal](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span></span>
- <span data-ttu-id="0dfa1-122">Egy Log Analytics-munkaterület.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-122">A Log Analytics workspace.</span></span> <span data-ttu-id="0dfa1-123">Lásd: [Log Analytics-munkaterület létrehozása az Azure Portalon](/azure/azure-monitor/learn/quick-create-workspace).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-123">See [Create a Log Analytics workspace in the Azure portal](/azure/azure-monitor/learn/quick-create-workspace).</span></span>

<span data-ttu-id="0dfa1-124">Ezután telepítse a [Azure Databricks parancssori felület](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-124">Next, install the [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span></span> <span data-ttu-id="0dfa1-125">Az Azure Databricks személyes hozzáférési jogkivonat szükséges a CLI használatával.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-125">An Azure Databricks personal access token is required to use the CLI.</span></span> <span data-ttu-id="0dfa1-126">Útmutatásért lásd: [egy jogkivonatot,](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-126">For instructions, see [Generate a token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span></span>

<span data-ttu-id="0dfa1-127">Keresse meg a Log Analytics-munkaterület Azonosítójára és kulcsára.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-127">Find your Log Analytics workspace ID and key.</span></span> <span data-ttu-id="0dfa1-128">Útmutatásért lásd: [szerezze be a munkaterület Azonosítójára és kulcsára](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-128">For instructions, see [Obtain workspace ID and key](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span></span> <span data-ttu-id="0dfa1-129">A Databricks-fürt konfigurálásakor szüksége lesz ezekre az értékekre.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-129">You will need these values when you configure the Databricks cluster.</span></span>

<span data-ttu-id="0dfa1-130">Az ügyfélfigyelési könyvtár hozhat létre, a Java ide Környezethez, a következő források lesz szüksége:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-130">To build the monitoring library, you will need a Java IDE with the following resources:</span></span>

- [<span data-ttu-id="0dfa1-131">Java fejlesztői készlet (JDK) 1.8-as verzió</span><span class="sxs-lookup"><span data-stu-id="0dfa1-131">Java Development Kit (JDK) version 1.8</span></span>](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [<span data-ttu-id="0dfa1-132">Scala nyelvi SDK 2.11-et</span><span class="sxs-lookup"><span data-stu-id="0dfa1-132">Scala language SDK 2.11</span></span>](https://www.scala-lang.org/download/)
- [<span data-ttu-id="0dfa1-133">Az Apache Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="0dfa1-133">Apache Maven 3.5.4</span></span>](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a><span data-ttu-id="0dfa1-134">Hozhat létre az Azure Databricks-tár figyelése</span><span class="sxs-lookup"><span data-stu-id="0dfa1-134">Build the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="0dfa1-135">Az Azure Databricks-figyelő könyvtár létrehozásához hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-135">To build the Azure Databricks Monitoring Library, perform the following steps:</span></span>

1. <span data-ttu-id="0dfa1-136">Klónozza, ágaztassa vagy töltse le a [GitHub-adattár](https://github.com/mspnp/spark-monitoring).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-136">Clone, fork, or download the [GitHub repository](https://github.com/mspnp/spark-monitoring).</span></span>

1. <span data-ttu-id="0dfa1-137">Importálja a Maven project objektum modellfájl _pom.xml_található, a **/src** mappát a projektbe a Finderből.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-137">Import the Maven project object model file, _pom.xml_, located in the **/src** folder into your project.</span></span> <span data-ttu-id="0dfa1-138">Ezzel importálja a három projekt:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-138">This will import three projects:</span></span>

    - <span data-ttu-id="0dfa1-139">Spark-feladatok</span><span class="sxs-lookup"><span data-stu-id="0dfa1-139">spark-jobs</span></span>
    - <span data-ttu-id="0dfa1-140">Spark-figyelők</span><span class="sxs-lookup"><span data-stu-id="0dfa1-140">spark-listeners</span></span>
    - <span data-ttu-id="0dfa1-141">spark-listeners-loganalytics</span><span class="sxs-lookup"><span data-stu-id="0dfa1-141">spark-listeners-loganalytics</span></span>

1. <span data-ttu-id="0dfa1-142">Hajtsa végre a Maven **csomag** build fázis hozhat létre a JAR-fájlok az egyes projektek a Java IDE-ben:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-142">Execute the Maven **package** build phase in your Java IDE to build the JAR files for each of these projects:</span></span>

    |<span data-ttu-id="0dfa1-143">Project</span><span class="sxs-lookup"><span data-stu-id="0dfa1-143">Project</span></span>| <span data-ttu-id="0dfa1-144">JAR-fájlt</span><span class="sxs-lookup"><span data-stu-id="0dfa1-144">JAR file</span></span>|
    |-------|---------|
    |<span data-ttu-id="0dfa1-145">Spark-feladatok</span><span class="sxs-lookup"><span data-stu-id="0dfa1-145">spark-jobs</span></span>|<span data-ttu-id="0dfa1-146">Spark-feladatok – 1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="0dfa1-146">spark-jobs-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="0dfa1-147">Spark-figyelők</span><span class="sxs-lookup"><span data-stu-id="0dfa1-147">spark-listeners</span></span>|<span data-ttu-id="0dfa1-148">Spark-figyelők – 1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="0dfa1-148">spark-listeners-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="0dfa1-149">spark-listeners-loganalytics</span><span class="sxs-lookup"><span data-stu-id="0dfa1-149">spark-listeners-loganalytics</span></span>|<span data-ttu-id="0dfa1-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="0dfa1-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span></span>|

1. <span data-ttu-id="0dfa1-151">Az Azure Databricks parancssori felület használatával hozzon létre egy könyvtárat nevű **dbfs: / databricks és figyelési-előkészítési**:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-151">Use the Azure Databricks CLI to create a directory named **dbfs:/databricks/monitoring-staging**:</span></span>  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. <span data-ttu-id="0dfa1-152">Az Azure Databricks parancssori felület használatával másolhatja **/src/spark-listeners/scripts/listeners.sh** korábban a könyvtárba:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-152">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/listeners.sh** to the directory previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. <span data-ttu-id="0dfa1-153">Az Azure Databricks parancssori felület használatával másolhatja **/src/spark-listeners/scripts/metrics.properties** a korábban létrehozott könyvtárba:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-153">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/metrics.properties** to the directory created previously:</span></span>

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. <span data-ttu-id="0dfa1-154">Az Azure Databricks parancssori felület használatával másolhatja **spark-figyelők – 1.0-SNAPSHOT.jar** és **spark-figyelők-loganalytics – 1.0-SNAPSHOT.jar** a korábban létrehozott könyvtárba:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-154">Use the Azure Databricks CLI to copy **spark-listeners-1.0-SNAPSHOT.jar** and **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** to the directory created previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a><span data-ttu-id="0dfa1-155">Létrehozhat és konfigurálhat egy Azure Databricks-fürt</span><span class="sxs-lookup"><span data-stu-id="0dfa1-155">Create and configure an Azure Databricks cluster</span></span>

<span data-ttu-id="0dfa1-156">Létrehozni és konfigurálni egy Azure Databricks-fürt, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-156">To create and configure an Azure Databricks cluster, follow these steps:</span></span>

1. <span data-ttu-id="0dfa1-157">Keresse meg az Azure Databricks-munkaterület az Azure Portalon.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-157">Navigate to your Azure Databricks workspace in the Azure portal.</span></span>
1. <span data-ttu-id="0dfa1-158">A kezdőlapon kattintson **új fürt**.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-158">On the home page, click **New Cluster**.</span></span>
1. <span data-ttu-id="0dfa1-159">Adja meg a fürt nevét a **fürtnév** szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-159">Enter a name for your cluster in the **Cluster Name** text box.</span></span>
1. <span data-ttu-id="0dfa1-160">Az a **Databricks futtatókörnyezet-verziója** legördülő menüben válassza **4.3** vagy nagyobb.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-160">In the **Databricks Runtime Version** dropdown, select **4.3** or greater.</span></span>
1. <span data-ttu-id="0dfa1-161">Alatt **speciális beállítások**, kattintson a **Spark** fülre. Adja meg a következő név-érték párok az a **Spark Config** szövegbeviteli mező:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-161">Under **Advanced Options**, click on the **Spark** tab. Enter the following name-value pairs in the **Spark Config** text box:</span></span>

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. <span data-ttu-id="0dfa1-162">Miközben továbbra is a a **Spark** lapra, adja meg a következő értékeket a **környezeti változók** szövegbeviteli mező:</span><span class="sxs-lookup"><span data-stu-id="0dfa1-162">While still under the **Spark** tab, enter the following values in the **Environment Variables** text box:</span></span>

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Databricks felhasználói felület képernyőképe](./_images/create-cluster1.png)

1. <span data-ttu-id="0dfa1-164">Miközben továbbra is a a **speciális beállítások** területén kattintson a a **Init parancsfájlok** fülre.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-164">While still under the **Advanced Options** section, click on the **Init Scripts** tab.</span></span>
1. <span data-ttu-id="0dfa1-165">Az a **cél** legördülő menüben válassza **DBFS**.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-165">In the **Destination** dropdown, select **DBFS**.</span></span>
1. <span data-ttu-id="0dfa1-166">A **Init parancsfájl elérési útján**, adja meg `dbfs:/databricks/monitoring-staging/listeners.sh`.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-166">Under **Init Script Path**, enter `dbfs:/databricks/monitoring-staging/listeners.sh`.</span></span>
1. <span data-ttu-id="0dfa1-167">Kattintson a **Hozzáadás** parancsra.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-167">Click **Add**.</span></span>

    ![Databricks felhasználói felület képernyőképe](./_images/create-cluster2.png)

1. <span data-ttu-id="0dfa1-169">Kattintson a **-fürt létrehozása** a fürt létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-169">Click **Create Cluster** to create the cluster.</span></span>

## <a name="view-metrics"></a><span data-ttu-id="0dfa1-170">Metrikák megtekintése</span><span class="sxs-lookup"><span data-stu-id="0dfa1-170">View metrics</span></span>

<span data-ttu-id="0dfa1-171">Miután végrehajtotta ezeket a lépéseket, a Databricks-fürt bizonyos metrikaadatok kapcsolatos magához a fürthöz az Azure monitornak elemzésének lehetőségeit.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-171">After you complete these steps, your Databricks cluster streams some metric data about the cluster itself to Azure Monitor.</span></span> <span data-ttu-id="0dfa1-172">A naplóadatok érhető el az Azure Log Analytics-munkaterület alatt az "Active |} Egyéni naplókkal |} A séma SparkMetric_CL".</span><span class="sxs-lookup"><span data-stu-id="0dfa1-172">This log data is available in your Azure Log Analytics workspace under the "Active | Custom Logs | SparkMetric_CL" schema.</span></span> <span data-ttu-id="0dfa1-173">Naplózott mérőszámok típusaival kapcsolatos további információkért lásd: [metrikák Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) Dropwizard dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-173">For more information about the types of metrics that are logged, see [Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) in the Dropwizard documentation.</span></span> <span data-ttu-id="0dfa1-174">A metrika típusa, például a mérőműszer, számláló vagy hisztogram, írja be a Type mezőjében.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-174">The metric type, such as gauge, counter, or histogram, is written to the Type field.</span></span>

<span data-ttu-id="0dfa1-175">A kódtár emellett hibaszintű eseményeket az Apache Spark és a Spark strukturált Stream mérőszámainak a feladatok az Azure monitornak streameli.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-175">In addition, the library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="0dfa1-176">Ne módosítsa az alkalmazás kódjában ezen események és metrikák nincs szükségünk.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-176">You don't need to make any changes to your application code for these events and metrics.</span></span> <span data-ttu-id="0dfa1-177">A Spark strukturált Stream naplóadatok érhető el az "Active |} Egyéni naplókkal |} A séma SparkListenerEvent_CL".</span><span class="sxs-lookup"><span data-stu-id="0dfa1-177">Spark Structured Streaming log data is available under the "Active | Custom Logs | SparkListenerEvent_CL" schema.</span></span>

![Képernyőkép a Log Analytics-munkaterület](./_images/workspace.png)

<span data-ttu-id="0dfa1-179">Naplók megtekintése kapcsolatos további információkért lásd: [megtekintése és elemzése az Azure monitorban naplóadatok](/azure/azure-monitor/log-query/portals).</span><span class="sxs-lookup"><span data-stu-id="0dfa1-179">For more information about viewing logs, see [Viewing and analyzing log data in Azure Monitor](/azure/azure-monitor/log-query/portals).</span></span>

## <a name="next-steps"></a><span data-ttu-id="0dfa1-180">További lépések</span><span class="sxs-lookup"><span data-stu-id="0dfa1-180">Next steps</span></span>

<span data-ttu-id="0dfa1-181">Ez a cikk ismerteti a mérőszámok küldése az Azure monitornak a fürt konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-181">This article described how to configure your cluster to send metrics to Azure Monitor.</span></span> <span data-ttu-id="0dfa1-182">A következő cikk bemutatja, hogyan küldhet, alkalmazásmetrikák és a naplók az Azure Databricks-figyelő könyvtár használatához.</span><span class="sxs-lookup"><span data-stu-id="0dfa1-182">The next article shows how to use the Azure Databricks Monitoring Library to send application metrics and logs.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="0dfa1-183">Protokoly aplikací küldése az Azure monitornak</span><span class="sxs-lookup"><span data-stu-id="0dfa1-183">Send application logs to Azure Monitor</span></span>](./application-logs.md)
