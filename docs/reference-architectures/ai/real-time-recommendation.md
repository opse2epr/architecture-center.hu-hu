---
title: Valós idejű ajánlás API létrehozása az Azure-ban
description: A gépi tanulási automatizálhatja az Azure Databricks és az Azure adatelemzési virtuális gépek (DSVM) segítségével az Azure-ban a modell betanításához javaslatokat.
author: njray
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 7f10c422c65967701084859e41f9656c818ed818
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298398"
---
# <a name="build-a-real-time-recommendation-api-on-azure"></a><span data-ttu-id="5ccaf-103">Valós idejű ajánlás API létrehozása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="5ccaf-103">Build a real-time recommendation API on Azure</span></span>

<span data-ttu-id="5ccaf-104">Ez a referenciaarchitektúra bemutatja egy Azure Databricks használatával javaslat modell betanítását, és helyezze üzembe, API-k az Azure Cosmos DB, az Azure Machine Learning és az Azure Kubernetes Service (AKS) használatával.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-104">This reference architecture shows how to train a recommendation model using Azure Databricks and deploy it as an API by using Azure Cosmos DB, Azure Machine Learning, and Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="5ccaf-105">Ez az architektúra is általánosítható legtöbb javaslat motor helyzetekre, termékek, filmek és hírek is.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-105">This architecture can be generalized for most recommendation engine scenarios, including recommendations for products, movies, and news.</span></span>

<span data-ttu-id="5ccaf-106">Az architektúra egy referenciaimplementációt érhető el az [GitHub][als-example].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-106">A reference implementation for this architecture is available on [GitHub][als-example].</span></span>

![A gépi tanulási modell betanításához filmjánlásokat architektúrája](./_images/recommenders-architecture.png)

<span data-ttu-id="5ccaf-108">**A forgatókönyv**: Egy media szervezet célja az filmet vagy videó javaslatokat biztosít a felhasználóknak.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-108">**Scenario**: A media organization wants to provide movie or video recommendations to its users.</span></span> <span data-ttu-id="5ccaf-109">Személyre szabott javaslatok biztosításával a szervezet megfelel a több üzleti céljait, beleértve a megnövekedett átkattintásos díjszabás, a webhelyen, és nagyobb felhasználói elégedettséget megnövekedett engagement.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-109">By providing personalized recommendations, the organization meets several business goals, including increased click-through rates, increased engagement on site, and higher user satisfaction.</span></span>

<span data-ttu-id="5ccaf-110">Ez a referenciaarchitektúra és üzembe helyezése egy valós idejű ajánló szolgáltatás API, amely képes biztosítani az adott felhasználó számára a legfontosabb 10 movie javaslatok van.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-110">This reference architecture is for training and deploying a real-time recommender service API that can provide the top 10 movie recommendations for a given user.</span></span>

<span data-ttu-id="5ccaf-111">Ez az ajánlás modellhez az adatok az adatfolyam a következőképpen történik:</span><span class="sxs-lookup"><span data-stu-id="5ccaf-111">The data flow for this recommendation model is as follows:</span></span>

1. <span data-ttu-id="5ccaf-112">Nyomon követheti a felhasználói viselkedés.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-112">Track user behaviors.</span></span> <span data-ttu-id="5ccaf-113">Háttérszolgáltatás naplózásra, például amikor egy felhasználó egy filmet értékeli vagy egy termék vagy news cikk.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-113">For example, a backend service might log when a user rates a movie or clicks a product or news article.</span></span>

2. <span data-ttu-id="5ccaf-114">Az adatok betöltéséhez az Azure Databricksbe az elérhető [adatforrás][data-source].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-114">Load the data into Azure Databricks from an available [data source][data-source].</span></span>

3. <span data-ttu-id="5ccaf-115">Az adatok előkészítése, és azt fel képzési és egy tesztelési a modell betanításához.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-115">Prepare the data and split it into training and testing sets to train the model.</span></span> <span data-ttu-id="5ccaf-116">([Ez az útmutató] [ guide] vágását meghatározó adatok lehetőségeket ismerteti.)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-116">([This guide][guide] describes options for splitting data.)</span></span>

4. <span data-ttu-id="5ccaf-117">Alkalmas a [Spark együttműködési szűrést] [ als] modell az adatokhoz.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-117">Fit the [Spark Collaborative Filtering][als] model to the data.</span></span>

5. <span data-ttu-id="5ccaf-118">Értékelje ki a minősítés és rangsorolása mérőszámok segítségével a modell minőségét.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-118">Evaluate the quality of the model using rating and ranking metrics.</span></span> <span data-ttu-id="5ccaf-119">([Ez az útmutató] [ eval-guide] részletesen a ajánló kiértékelheti a metrikákat.)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-119">([This guide][eval-guide] provides details about the metrics you can evaluate your recommender on.)</span></span>

6. <span data-ttu-id="5ccaf-120">A felső 10 javaslatok felhasználónként és áruházbeli precompute gyorsítótárként Azure Cosmos DB-ben.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-120">Precompute the top 10 recommendations per user and store as a cache in Azure Cosmos DB.</span></span>

7. <span data-ttu-id="5ccaf-121">API-szolgáltatás üzembe az aks-tárolóba, és üzembe helyezése az API-t az Azure Machine Learning API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-121">Deploy an API service to AKS using the Azure Machine Learning APIs to containerize and deploy the API.</span></span>

8. <span data-ttu-id="5ccaf-122">Ha a háttérszolgáltatáshoz kérelem olvas be egy felhasználó, hívja meg a javaslatok az aks-ben a felső 10 javaslatokat kaphat, és megjeleníti őket a felhasználó üzemeltetett API.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-122">When the backend service gets a request from a user, call the recommendations API hosted in AKS to get the top 10 recommendations and display them to the user.</span></span>

## <a name="architecture"></a><span data-ttu-id="5ccaf-123">Architektúra</span><span class="sxs-lookup"><span data-stu-id="5ccaf-123">Architecture</span></span>

<span data-ttu-id="5ccaf-124">Ez az architektúra a következő összetevőkből áll:</span><span class="sxs-lookup"><span data-stu-id="5ccaf-124">This architecture consists of the following components:</span></span>

<span data-ttu-id="5ccaf-125">[Az Azure Databricks][databricks].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-125">[Azure Databricks][databricks].</span></span> <span data-ttu-id="5ccaf-126">Databricks egy fejlesztési környezetet a bemeneti adatok előkészítése, és a Spark-fürtön ajánló modell betanításához.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-126">Databricks is a development environment used to prepare input data and train the recommender model on a Spark cluster.</span></span> <span data-ttu-id="5ccaf-127">Az Azure Databricks egy interaktív munkaterület futtassa, és közösen dolgozhatnak a notebookok bármilyen adatfeldolgozás és a gépi tanulási feladatok is biztosít.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-127">Azure Databricks also provides an interactive workspace to run and collaborate on notebooks for any data processing or machine learning tasks.</span></span>

<span data-ttu-id="5ccaf-128">[Az Azure Kubernetes Service] [ aks] (AKS).</span><span class="sxs-lookup"><span data-stu-id="5ccaf-128">[Azure Kubernetes Service][aks] (AKS).</span></span> <span data-ttu-id="5ccaf-129">Az AKS üzembe helyezése és a egy gépi tanulási modell szolgáltatás API-t a Kubernetes-fürt üzembe helyezése szolgál.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-129">AKS is used to deploy and operationalize a machine learning model service API on a Kubernetes cluster.</span></span> <span data-ttu-id="5ccaf-130">Az AKS üzemelteti a tárolóalapú modell méretezhetőséget, amely megfelel az átviteli sebességet megkövetelő, identitás- és hozzáférés-kezelés és naplózás és szolgáltatásállapot-figyelést.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-130">AKS hosts the containerized model, providing scalability that meets your throughput requirements, identity and access management, and logging and health monitoring.</span></span>

<span data-ttu-id="5ccaf-131">[Az Azure Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-131">[Azure Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="5ccaf-132">A cosmos DB egy globálisan elosztott adatbázis-szolgáltatás tárolja a felső 10 ajánlott filmek minden felhasználóhoz.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-132">Cosmos DB is a globally distributed database service used to store the top 10 recommended movies for each user.</span></span> <span data-ttu-id="5ccaf-133">Azure Cosmos DB a ebben a forgatókönyvben is használható, mert biztosít alacsony késéssel (10 ezredmásodperc 99. percentilis), olvassa el a legfontosabb ajánlott elemeket az adott felhasználó számára.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-133">Azure Cosmos DB is well-suited for this scenario, because it provides low latency (10 ms at 99th percentile) to read the top recommended items for a given user.</span></span>

<span data-ttu-id="5ccaf-134">[Az Azure Machine Learning szolgáltatás][mls].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-134">[Azure Machine Learning Service][mls].</span></span> <span data-ttu-id="5ccaf-135">Ez a szolgáltatás segítségével nyomon követheti és kezelheti a machine learning-modellek, és majd becsomagolhatja és ezek a modellek üzembe helyezése méretezhető AKS-környezetben.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-135">This service is used to track and manage machine learning models, and then package and deploy these models to a scalable AKS environment.</span></span>

<span data-ttu-id="5ccaf-136">[A Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-136">[Microsoft Recommenders][github].</span></span> <span data-ttu-id="5ccaf-137">A nyílt forráskódú tárház segédprogram-kódot tartalmaz, és különböző minták segítik a felhasználók létrehozásához, kiértékelése, modellezést és a egy ajánló rendszer használatának megkezdéséhez.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-137">This open-source repository contains utility code and samples to help users get started in building, evaluating, and operationalizing a recommender system.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="5ccaf-138">A teljesítménnyel kapcsolatos megfontolások</span><span class="sxs-lookup"><span data-stu-id="5ccaf-138">Performance considerations</span></span>

<span data-ttu-id="5ccaf-139">Teljesítményének valós idejű javaslatokat számára elsődleges jelentőségű azért ajánlások általában esik a kritikus útvonalat a kérelem egy felhasználó által a webhelyen.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-139">Performance is a primary consideration for real-time recommendations, because recommendations usually fall in the critical path of the request a user makes on your site.</span></span>

<span data-ttu-id="5ccaf-140">AKS és Azure Cosmos DB kombinációja lehetővé teszi, hogy ez az architektúra a javaslatokat is ad egy közepes méretű számítási feladatok minimális fenntartási az jó kiindulási pont biztosítása.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-140">The combination of AKS and Azure Cosmos DB enables this architecture to provide a good starting point to provide recommendations for a medium-sized workload with minimal overhead.</span></span> <span data-ttu-id="5ccaf-141">Ez az architektúra egy terhelési teszt 200 egyidejű felhasználóval, ajánlásokkal medián késése, 60 MS-os, és 180 kérelmek másodpercenkénti adatátviteli kapacitással hajt végre.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-141">Under a load test with 200 concurrent users, this architecture provides recommendations at a median latency of about 60 ms and performs at a throughput of 180 requests per second.</span></span> <span data-ttu-id="5ccaf-142">A terhelési teszt volt futtatva, szemben az alapértelmezett telepítési konfigurációt (egy 3 12 vcpu-k x D3 v2 AKS-fürtöt, 42 GB memória, és 11 000 [kérelemegység (RU) másodpercenként] [ ru] kiépítése az Azure Cosmos DB).</span><span class="sxs-lookup"><span data-stu-id="5ccaf-142">The load test was run against the default deployment configuration (a 3x D3 v2 AKS cluster with 12 vCPUs, 42 GB of memory, and 11,000 [Request Units (RUs) per second][ru] provisioned for Azure Cosmos DB).</span></span>

![Teljesítmény-grafikon](./_images/recommenders-performance.png)

![Átviteli sebesség-grafikon](./_images/recommenders-throughput.png)

<span data-ttu-id="5ccaf-145">Az Azure Cosmos DB a kulcsrakész globális disztribúciót és az összes adatbázis követelményeknek, az alkalmazás hasznosságát használata ajánlott.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-145">Azure Cosmos DB is recommended for its turnkey global distribution and usefulness in meeting any database requirements your app has.</span></span> <span data-ttu-id="5ccaf-146">A kis mértékben [gyorsabb késés][latency], érdemes lehet [Azure Redis Cache] [ redis] helyett az Azure Cosmos DB keresések kiszolgálása érdekében.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-146">For slightly [faster latency][latency], consider using [Azure Redis Cache][redis] instead of Azure Cosmos DB to serve lookups.</span></span> <span data-ttu-id="5ccaf-147">Redis Cache javíthatja a teljesítményt a háttér-tárolja az adatok magas használó rendszerek.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-147">Redis Cache can improve performance of systems that rely highly on data in back-end stores.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="5ccaf-148">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="5ccaf-148">Scalability considerations</span></span>

<span data-ttu-id="5ccaf-149">Ha nem tervezi a Spark használata, vagy ha kisebb számítási feladatokhoz, ahol nem kell terjesztési, érdemes [adatelemző virtuális gép] [ dsvm] (DSVM) az Azure Databricks helyett.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-149">If you don't plan to use Spark, or you have a smaller workload where you don't need distribution, consider using [Data Science Virtual Machine][dsvm] (DSVM) instead of Azure Databricks.</span></span> <span data-ttu-id="5ccaf-150">DSVM használata egy Azure virtuális gépen, a deep learning-keretrendszerek és gép tanulási és adatelemző eszközei.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-150">DSVM is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="5ccaf-151">Mint az Azure Databricks, egy adatelemző virtuális GÉPET hoz létre minden olyan modellt is kell üzembe helyezte azt az aks-en keresztül az Azure Machine Learning szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-151">As with Azure Databricks, any model you create in a DSVM can be operationalized as a service on AKS via Azure Machine Learning.</span></span>

<span data-ttu-id="5ccaf-152">Során képzés, nagyobb rögzített méretű Spark-fürt kiépítése az Azure Databricks, vagy konfigurálja [az automatikus skálázás][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-152">During training, provision a larger fixed-size Spark cluster in Azure Databricks or configure [autoscaling][autoscaling].</span></span> <span data-ttu-id="5ccaf-153">Ha az automatikus skálázás engedélyezve van, Databricks figyeli a fürt terhelésének és felskálázással, és ahonnan szükség esetén.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-153">When autoscaling is enabled, Databricks monitors the load on your cluster and scales up and downs when required.</span></span> <span data-ttu-id="5ccaf-154">Kiépítés vagy horizontális felskálázás nagyobb fürt rendelkezik egy nagy mennyiségű adat méretét, és ha szeretné az adat-előkészítési vagy feladatok modellezési szükséges idő csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-154">Provision or scale out a larger cluster if you have a large data size and you want to reduce the amount of time it takes for data preparation or modeling tasks.</span></span>

<span data-ttu-id="5ccaf-155">Az AKS-fürtöt, a teljesítmény, átviteli sebesség igényeinek megfelelően méretezhető.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-155">Scale the AKS cluster to meet your performance and throughput requirements.</span></span> <span data-ttu-id="5ccaf-156">Ügyeljen arra, hogy a vertikális felskálázás száma [podok] [ scale] használja ki teljesen a fürtöt, és skálázhatja a [csomópontok] [ nodes] az igény, a fürt a szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-156">Take care to scale up the number of [pods][scale] to fully utilize the cluster, and to scale the [nodes][nodes] of the cluster to meet the demand of your service.</span></span> <span data-ttu-id="5ccaf-157">A teljesítmény-és átviteli sebességet megkövetelő a ajánló szolgáltatás fürt méretezése további információkért lásd: [méretezése az Azure Container Service-fürtök][blog].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-157">For more information on how to scale your cluster to meet the performance and throughput requirements of your recommender service, see [Scaling Azure Container Service Clusters][blog].</span></span>

<span data-ttu-id="5ccaf-158">Azure Cosmos DB teljesítményének kezelésére, megbecsülheti a szükséges másodpercenként olvasott számát, és kiépítése az száma [kérelemegység / másodperc] [ ru] (teljesítmény) szükséges.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-158">To manage Azure Cosmos DB performance, estimate the number of reads required per second, and provision the number of [RUs per second][ru] (throughput) needed.</span></span> <span data-ttu-id="5ccaf-159">Ajánlott eljárások használata [particionálási és horizontális skálázást][partition-data].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-159">Use best practices for [partitioning and horizontal scaling][partition-data].</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="5ccaf-160">Költségekkel kapcsolatos szempontok</span><span class="sxs-lookup"><span data-stu-id="5ccaf-160">Cost considerations</span></span>

<span data-ttu-id="5ccaf-161">Költség ebben a forgatókönyvben a fő illesztőprogramok a következők:</span><span class="sxs-lookup"><span data-stu-id="5ccaf-161">The main drivers of cost in this scenario are:</span></span>

- <span data-ttu-id="5ccaf-162">Az Azure Databricks-fürt szükséges képzés méretének.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-162">The Azure Databricks cluster size required for training.</span></span>
- <span data-ttu-id="5ccaf-163">Az AKS-fürt méretét a teljesítményre vonatkozó követelmények teljesítéséhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-163">The AKS cluster size required to meet your performance requirements.</span></span>
- <span data-ttu-id="5ccaf-164">Az Azure Cosmos DB Kérelemegységek kiépítette a hálózatiteljesítmény-igények kielégítéséhez.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-164">Azure Cosmos DB RUs provisioned to meet your performance requirements.</span></span>

<span data-ttu-id="5ccaf-165">Az Azure Databricks költségek kezelése az átképezési ritkábban, és kikapcsolja a Spark-fürtöt, amikor nincs használatban.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-165">Manage the Azure Databricks costs by retraining less frequently and turning off the Spark cluster when not in use.</span></span> <span data-ttu-id="5ccaf-166">Az AKS és Azure Cosmos DB költségek kötődnek, az átviteli sebesség és a hely által igényelt teljesítmény, és a webhely forgalmának mennyiségétől függően felfelé és lefelé skálázhatja.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-166">The AKS and Azure Cosmos DB costs are tied to the throughput and performance required by your site and will scale up and down depending on the volume of traffic to your site.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="5ccaf-167">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="5ccaf-167">Deploy the solution</span></span>

<span data-ttu-id="5ccaf-168">Ez az architektúra üzembe helyezéséhez kövesse a **Azure Databricks** utasításait a [telepítő dokumentum][setup].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-168">To deploy this architecture, follow the **Azure Databricks** instructions in the [setup document][setup].</span></span> <span data-ttu-id="5ccaf-169">Röviden utasításokat kell tennie:</span><span class="sxs-lookup"><span data-stu-id="5ccaf-169">Briefly, the instructions require you to:</span></span>

1. <span data-ttu-id="5ccaf-170">Hozzon létre egy [Azure Databricks-munkaterület][workspace].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-170">Create an [Azure Databricks workspace][workspace].</span></span>

2. <span data-ttu-id="5ccaf-171">Hozzon létre egy új fürtöt az Azure Databricksben a következő beállításokkal:</span><span class="sxs-lookup"><span data-stu-id="5ccaf-171">Create a new cluster with the following configuration in Azure Databricks:</span></span>

    - <span data-ttu-id="5ccaf-172">Fürt üzemmód: Standard</span><span class="sxs-lookup"><span data-stu-id="5ccaf-172">Cluster mode: Standard</span></span>
    - <span data-ttu-id="5ccaf-173">Databricks futtatókörnyezet-verziója: 4.3-as (tartalmazza az Apache Spark 2.3.1, Scala 2.11-et)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-173">Databricks Runtime Version: 4.3 (includes Apache Spark 2.3.1, Scala 2.11)</span></span>
    - <span data-ttu-id="5ccaf-174">Python-verzió: 3</span><span class="sxs-lookup"><span data-stu-id="5ccaf-174">Python Version: 3</span></span>
    - <span data-ttu-id="5ccaf-175">Illesztőprogram típusa: Standard\_DS3\_v2</span><span class="sxs-lookup"><span data-stu-id="5ccaf-175">Driver Type: Standard\_DS3\_v2</span></span>
    - <span data-ttu-id="5ccaf-176">Feldolgozó típusa: Standard szintű\_DS3\_v2 (minimális és maximális száma szükség szerint)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-176">Worker Type: Standard\_DS3\_v2 (min and max as required)</span></span>
    - <span data-ttu-id="5ccaf-177">Automatikus lezárás: (a megadása kötelező)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-177">Auto Termination: (as required)</span></span>
    - <span data-ttu-id="5ccaf-178">Spark-Config: (a megadása kötelező)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-178">Spark Config: (as required)</span></span>
    - <span data-ttu-id="5ccaf-179">Környezeti változók: (kötelezőként)</span><span class="sxs-lookup"><span data-stu-id="5ccaf-179">Environment Variables: (as required)</span></span>

3. <span data-ttu-id="5ccaf-180">Hozzon létre egy személyes hozzáférési tokent belül a [Azure Databricks-munkaterület][workspace].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-180">Create a personal access token within the [Azure Databricks workspace][workspace].</span></span> <span data-ttu-id="5ccaf-181">Tekintse meg az Azure Databricks hitelesítési [dokumentáció] [ adbauthentication] részleteiről.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-181">See the Azure Databricks authentication [documentation][adbauthentication] for details.</span></span>

3. <span data-ttu-id="5ccaf-182">Klónozás a [Microsoft Recommenders] [ github] környezetbe adattár, ahol futtathat parancsfájlokat (pl. a helyi számítógép).</span><span class="sxs-lookup"><span data-stu-id="5ccaf-182">Clone the [Microsoft Recommenders][github] repository into an environment where you can execute scripts (e.g. your local computer).</span></span>

4. <span data-ttu-id="5ccaf-183">Kövesse a **rövid telepítési** telepítési utasításokat a [a megfelelő kódtárak telepítése] [ setup] az Azure databricks szolgáltatásban.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-183">Follow the **Quick install** setup instructions to [install the relevant libraries][setup] on Azure Databricks.</span></span>

5. <span data-ttu-id="5ccaf-184">Kövesse a **rövid telepítési** telepítési utasításokat a [előkészítése az Azure Databricks az operacionalizáláshoz][setupo16n].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-184">Follow the **Quick install** setup instructions to [prepare Azure Databricks for operationalization][setupo16n].</span></span>

6. <span data-ttu-id="5ccaf-185">Importálás a [ALS Movie Operacionalizálás notebook] [ als-example] egyszerűen a munkaterületre.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-185">Import the [ALS Movie Operationalization notebook][als-example] into your workspace.</span></span> <span data-ttu-id="5ccaf-186">Miután bejelentkezett az Azure Databricks-munkaterület, tegye a következőket:</span><span class="sxs-lookup"><span data-stu-id="5ccaf-186">After logging into your Azure Databricks Workspace, do the following:</span></span>

    <span data-ttu-id="5ccaf-187">a.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-187">a.</span></span> <span data-ttu-id="5ccaf-188">Kattintson a **kezdőlap** a munkaterület bal oldalán.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-188">Click **Home** on the left side of the workspace.</span></span>

    <span data-ttu-id="5ccaf-189">b.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-189">b.</span></span> <span data-ttu-id="5ccaf-190">Kattintson a jobb gombbal az üres helyet a kezdőkönyvtárban.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-190">Right-click on white space in your home directory.</span></span> <span data-ttu-id="5ccaf-191">Válassza ki **importálás**.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-191">Select **Import**.</span></span>
    
    <span data-ttu-id="5ccaf-192">c.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-192">c.</span></span> <span data-ttu-id="5ccaf-193">Válassza ki **URL-cím**, és illessze be a következőt a szövegmezőhöz: `https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb`</span><span class="sxs-lookup"><span data-stu-id="5ccaf-193">Select **URL**, and paste the following into the text field: `https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb`</span></span>
    
    <span data-ttu-id="5ccaf-194">d.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-194">d.</span></span> <span data-ttu-id="5ccaf-195">Kattintson az **Importálás** gombra.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-195">Click **Import**.</span></span>

7. <span data-ttu-id="5ccaf-196">Nyissa meg a notebook az Azure databricksben, és csatolja a konfigurált fürtöt.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-196">Open the notebook within Azure Databricks and attach the configured cluster.</span></span>

8. <span data-ttu-id="5ccaf-197">A jegyzetfüzet futtatásához használja a javaslatok API létrehozásához szükséges Azure-erőforrások létrehozása, amely az adott felhasználó számára a top-10 movie javaslatot is tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-197">Run the notebook to create the Azure resources required to create a recommendation API that provides the top-10 movie recommendations for a given user.</span></span>

## <a name="related-architectures"></a><span data-ttu-id="5ccaf-198">Kapcsolódó referenciaarchitektúrák</span><span class="sxs-lookup"><span data-stu-id="5ccaf-198">Related architectures</span></span>

<span data-ttu-id="5ccaf-199">Is készítettük el egy referencia-architektúra, amely a Spark és az Azure Databricks végrehajtásához ütemezett [kötegelt pontozási folyamatokat][batch-scoring].</span><span class="sxs-lookup"><span data-stu-id="5ccaf-199">We have also built a reference architecture that uses Spark and Azure Databricks to execute scheduled [batch-scoring processes][batch-scoring].</span></span> <span data-ttu-id="5ccaf-200">A referenciaarchitektúra egy új javaslatok rendszeresen létrehozásához ajánlott módszer megismeréséhez tekintse meg.</span><span class="sxs-lookup"><span data-stu-id="5ccaf-200">See that reference architecture to understand a recommended approach for generating new recommendations routinely.</span></span>

<!-- links -->
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[adbauthentication]: https://docs.azuredatabricks.net/api/latest/authentication.html#generate-a-token
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[als-example]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb
[autoscaling]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[availability]: /azure/architecture/checklist/availability
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-databricks
[blob]: /azure/storage/blobs/storage-blobs-introduction
[blog]: https://blogs.technet.microsoft.com/machinelearning/2018/03/20/scaling-azure-container-service-cluster/
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[data-source]: https://docs.azuredatabricks.net/spark/latest/data-sources/index.html
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[eval-guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/03_evaluate/evaluation.ipynb
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/01_prepare_data/data_split.ipynb
[latency]: https://github.com/jessebenson/azure-performance
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[nodes]: /azure/aks/scale-cluster
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[partition-data]: /azure/cosmos-db/partition-data
[redis]: /azure/redis-cache/cache-overview
[regions]: https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[ru]: /azure/cosmos-db/request-units
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#repository-installation
[setupo16n]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#prepare-azure-databricks-for-operationalization
[scale]: /azure/aks/tutorial-kubernetes-scale
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[workspace]: https://docs.azuredatabricks.net/getting-started/index.html
