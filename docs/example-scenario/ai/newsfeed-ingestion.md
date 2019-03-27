---
title: Az Azure-ban hírcsatornák átirányítandó és elemzési hírei
description: Létrehoz egy folyamatot feldolgozására, és a szöveg, képek, hangulatát és más RSS-hírcsatornák csak Azure-szolgáltatások, többek között az Azure Cosmos DB és az Azure Cognitive Services használata az adatok elemzéséhez.
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489236"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a><span data-ttu-id="8a627-103">Az Azure-ban hírcsatornák átirányítandó és elemzési hírei</span><span class="sxs-lookup"><span data-stu-id="8a627-103">Mass ingestion and analysis of news feeds on Azure</span></span>

<span data-ttu-id="8a627-104">Ebben a példaforgatókönyvben átirányítandó és közel valós idejű elemzése a nyilvános RSS-hírcsatornák használata dokumentumok folyamatot ismerteti.</span><span class="sxs-lookup"><span data-stu-id="8a627-104">This example scenario describes a pipeline for mass ingestion and near real-time analysis of documents using public RSS news feeds.</span></span>  <span data-ttu-id="8a627-105">Az Azure Cognitive Services használatával például szövegfordítás, arcfelismerés és hangulatfelismerés hasznos információkat.</span><span class="sxs-lookup"><span data-stu-id="8a627-105">It uses Azure Cognitive Services to offer useful insights including text translation, facial recognition, and sentiment detection.</span></span>

<span data-ttu-id="8a627-106">Ez a forgatókönyv példáit tartalmazza [angol][english], [orosz][russian], és [német] [ german] hírek hírcsatornák, de könnyen kiterjesztheti ezt más RSS-hírcsatornák.</span><span class="sxs-lookup"><span data-stu-id="8a627-106">This scenario contains examples for [English][english], [Russian][russian], and [German][german] news feeds, but you can easily extend it to other RSS feeds.</span></span> <span data-ttu-id="8a627-107">Az üzembe helyezés megkönnyítéséhez az adatok gyűjtése, feldolgozása és elemzése alapján teljes egészében az Azure-szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="8a627-107">For ease of deployment, the data collection, processing, and analysis are based entirely on Azure services.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="8a627-108">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="8a627-108">Relevant use cases</span></span>

<span data-ttu-id="8a627-109">Bár ebben a forgatókönyvben az RSS-hírcsatornák feldolgozási alapul, fontos bármely dokumentum, a webhely vagy a cikk, ahol kell:</span><span class="sxs-lookup"><span data-stu-id="8a627-109">While this scenario is based on processing of RSS feeds, it's relevant to any document, website, or article where you would need to:</span></span>

* <span data-ttu-id="8a627-110">A választott nyelven szöveg lefordítása.</span><span class="sxs-lookup"><span data-stu-id="8a627-110">Translate any text to the language of choice.</span></span>
* <span data-ttu-id="8a627-111">Keresse meg a kulcskifejezéseket, az entitások és a felhasználói vélemények a digitális tartalom.</span><span class="sxs-lookup"><span data-stu-id="8a627-111">Find key phrases, entities, and user sentiment in digital content.</span></span>
* <span data-ttu-id="8a627-112">Képek digitális cikk társított objektumokat, szöveg és tereptárgyak felismerése képes észlelni.</span><span class="sxs-lookup"><span data-stu-id="8a627-112">Detect objects, text, and landmarks in images associated with a digital article.</span></span>
* <span data-ttu-id="8a627-113">Képes észlelni a nemek és a korszűrő személyek digitális tartalmak társított képet.</span><span class="sxs-lookup"><span data-stu-id="8a627-113">Detect people by their gender and age in any image associated with digital content.</span></span>

## <a name="architecture"></a><span data-ttu-id="8a627-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="8a627-114">Architecture</span></span>

![][architecture]

<span data-ttu-id="8a627-115">Az adatok a következőképpen folyamatok a megoldáson keresztül:</span><span class="sxs-lookup"><span data-stu-id="8a627-115">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="8a627-116">Egy RSS-hírcsatorna a generátort, amely lekéri az adatokat egy dokumentum vagy cikk funkcionál.</span><span class="sxs-lookup"><span data-stu-id="8a627-116">An RSS news feed acts as the generator that obtains data from a document or article.</span></span> <span data-ttu-id="8a627-117">Például egy cikken, az adatok általában tartalmaz egy címet, a hír eredeti törzse összegzését, és időnként lemezképeket.</span><span class="sxs-lookup"><span data-stu-id="8a627-117">For example, with an article, data typically includes a title, a summary of the original body of the news item, and sometimes images.</span></span>

2. <span data-ttu-id="8a627-118">A generátor vagy a betöltési folyamat szúr be egy Azure Cosmos DB a cikket, és minden hozzá kapcsolódó képek [gyűjtemény][collection].</span><span class="sxs-lookup"><span data-stu-id="8a627-118">A generator or ingestion process inserts the article and any associated images into an Azure Cosmos DB [Collection][collection].</span></span>

3. <span data-ttu-id="8a627-119">Értesítés aktiválása egy betöltés függvényt az Azure Functions, amely a cikk szövegébe tárolja a cosmos DB és a cikk képek (ha vannak) az Azure Blob Storage.</span><span class="sxs-lookup"><span data-stu-id="8a627-119">A notification triggers an ingest function in Azure Functions that stores the article text in CosmosDB and the article images (if any) in Azure Blob Storage.</span></span>  <span data-ttu-id="8a627-120">A cikk majd átadja a sorban következő üzenetsornak.</span><span class="sxs-lookup"><span data-stu-id="8a627-120">The article is then passed to the next queue.</span></span>

4. <span data-ttu-id="8a627-121">Fordítás függvény a várólista esemény váltja ki.</span><span class="sxs-lookup"><span data-stu-id="8a627-121">A translate function is triggered by the queue event.</span></span> <span data-ttu-id="8a627-122">Használja a [fordítása Text API] [ translate-text] Azure Cognitive Services, észlelje a nyelvet, fordítás, ha szükséges, valamint a róluk szóló véleményeket, kulcskifejezéseket és entitások gyűjteni a szervezet és a cím.</span><span class="sxs-lookup"><span data-stu-id="8a627-122">It uses the [Translate Text API][translate-text] of Azure Cognitive Services to detect the language, translate if necessary, and collect the sentiment, key phrases, and entities from the body and the title.</span></span> <span data-ttu-id="8a627-123">Ezután átadja a cikk a sorban következő üzenetsornak.</span><span class="sxs-lookup"><span data-stu-id="8a627-123">Then it passes the article to the next queue.</span></span>

5. <span data-ttu-id="8a627-124">A hibakeresés függvény meghívása a sorban álló cikkben.</span><span class="sxs-lookup"><span data-stu-id="8a627-124">A detect function is triggered from the queued article.</span></span> <span data-ttu-id="8a627-125">Használja a [Computer Vision] [ vision] szolgáltatás észleli a objektumok arcrész és írt szavak társított lemezképet, majd átadja a cikk a sorban következő üzenetsornak.</span><span class="sxs-lookup"><span data-stu-id="8a627-125">It uses the [Computer Vision][vision] service to detect objects, landmarks, and written words in the associated image, then passes the article to the next queue.</span></span>

6. <span data-ttu-id="8a627-126">A face függvény meghívása a várólistán lévő cikk indító.</span><span class="sxs-lookup"><span data-stu-id="8a627-126">A face function is triggered is triggered from the queued article.</span></span> <span data-ttu-id="8a627-127">Használja a [Azure Face API] [ face] szolgáltatás észleli a nemek és a korszűrő társított lemezképet az arcokat, majd továbbítja a cikk a sorban következő üzenetsornak.</span><span class="sxs-lookup"><span data-stu-id="8a627-127">It uses the [Azure Face API][face] service to detect faces for gender and age in the associated image, then passes the article to the next queue.</span></span>

7. <span data-ttu-id="8a627-128">Amikor a függvények teljes, a értesítendő függvény aktiválódik.</span><span class="sxs-lookup"><span data-stu-id="8a627-128">When all functions are complete, the notify function is triggered.</span></span> <span data-ttu-id="8a627-129">Ez betölti a cikk a feldolgozott rekordok, és megkeresi azokat a kívánt eredményt.</span><span class="sxs-lookup"><span data-stu-id="8a627-129">It loads the processed records for the article and scans them for any results you want.</span></span> <span data-ttu-id="8a627-130">Ha található, a tartalom meg van jelölve, és a egy értesítést küld a rendszer a kiválasztott.</span><span class="sxs-lookup"><span data-stu-id="8a627-130">If found, the content is flagged and a notification is sent to the system of your choice.</span></span>

<span data-ttu-id="8a627-131">Minden egyes feldolgozási lépést, a függvény az Azure Cosmos DB kiírja az eredményeket.</span><span class="sxs-lookup"><span data-stu-id="8a627-131">At each processing step, the function writes the results to Azure Cosmos DB.</span></span> <span data-ttu-id="8a627-132">Végső soron az adatok segítségével igény szerint.</span><span class="sxs-lookup"><span data-stu-id="8a627-132">Ultimately, the data can be used as desired.</span></span> <span data-ttu-id="8a627-133">Például használhatja, javíthatja az üzleti folyamatok, keresse meg az új ügyfeleket vagy ügyfél-elégedettségi problémák azonosításához.</span><span class="sxs-lookup"><span data-stu-id="8a627-133">For example, you can use it to enhance business processes, locate new customers, or identify customer satisfaction issues.</span></span>

### <a name="components"></a><span data-ttu-id="8a627-134">Összetevők</span><span class="sxs-lookup"><span data-stu-id="8a627-134">Components</span></span>

<span data-ttu-id="8a627-135">Az alábbi listában szereplő Azure-összetevőket ebben a példában szolgál.</span><span class="sxs-lookup"><span data-stu-id="8a627-135">The following list of Azure components is used in this example.</span></span>

* <span data-ttu-id="8a627-136">[Az Azure Storage] [ storage] nyers kép és a egy cikk társított videofájlok tárolására szolgál.</span><span class="sxs-lookup"><span data-stu-id="8a627-136">[Azure Storage][storage] is used to hold raw image and video files associated with an article.</span></span> <span data-ttu-id="8a627-137">Másodlagos tárfiók létrehozása az Azure App Service és az Azure-függvény kódjának és a naplók tárolására használt.</span><span class="sxs-lookup"><span data-stu-id="8a627-137">A secondary storage account is created with Azure App Service and is used to host the Azure Function code and logs.</span></span>

* <span data-ttu-id="8a627-138">[Az Azure Cosmos DB] [ cosmos-db] visszatartással article szöveg, kép- és nyomkövetési adatokat.</span><span class="sxs-lookup"><span data-stu-id="8a627-138">[Azure Cosmos DB][cosmos-db] holds article text, image, and video tracking information.</span></span> <span data-ttu-id="8a627-139">A Cognitive Services lépések eredményeit a rendszer szintén itt tárolja.</span><span class="sxs-lookup"><span data-stu-id="8a627-139">The results of the Cognitive Services steps are also stored here.</span></span>

* <span data-ttu-id="8a627-140">[Az Azure Functions] [ functions] végrehajtja a függvénykódot az üzenetek üzenetsorba való válaszol, és a bejövő tartalom átalakítása.</span><span class="sxs-lookup"><span data-stu-id="8a627-140">[Azure Functions][functions] executes the function code used to respond to queue messages and transform the incoming content.</span></span> <span data-ttu-id="8a627-141">[Az Azure App Service] [ aas] üzemelteti a függvénykódot és tárolókonfigurációhoz dolgozza fel a rekordokat.</span><span class="sxs-lookup"><span data-stu-id="8a627-141">[Azure App Service][aas] hosts the function code and processes the records serially.</span></span> <span data-ttu-id="8a627-142">Ebben a forgatókönyvben öt függvényt tartalmazza: Betöltési, átalakítási, objektum, arc, észlelése, és értesíti.</span><span class="sxs-lookup"><span data-stu-id="8a627-142">This scenario includes five functions: Ingest, Transform, Detect Object, Face, and Notify.</span></span>

* <span data-ttu-id="8a627-143">[Az Azure Service Bus] [ service-bus] üzemelteti az Azure Service Bus-üzenetsorok, a functions által használt.</span><span class="sxs-lookup"><span data-stu-id="8a627-143">[Azure Service Bus][service-bus] hosts the Azure Service Bus queues used by the functions.</span></span>

* <span data-ttu-id="8a627-144">[Az Azure Cognitive Services] [ acs] biztosít a mesterséges Intelligencia megvalósítása alapján a folyamat a [Computer Vision] [ vision] szolgáltatás [arc API][face], és [szöveg lefordítása] [ translate-text] gépi fordítási szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="8a627-144">[Azure Cognitive Services][acs] provides the AI for the pipeline based on implementations of the [Computer Vision][vision] service, [Face API][face], and [Translate Text][translate-text] machine translation service.</span></span>

* <span data-ttu-id="8a627-145">[Az Azure Application Insights] [ aai] analytics segítségével diagnosztizálhatja a problémákat és az alkalmazás működésének megértése biztosít.</span><span class="sxs-lookup"><span data-stu-id="8a627-145">[Azure Application Insights][aai] provides analytics to help you diagnose issues and to understand functionality of your application.</span></span>

### <a name="alternatives"></a><span data-ttu-id="8a627-146">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="8a627-146">Alternatives</span></span>

* <span data-ttu-id="8a627-147">Ez az adatfolyam egy másik minta használata helyett egy minta alapján az értesítési várólista és az Azure Functions használatával.</span><span class="sxs-lookup"><span data-stu-id="8a627-147">Instead of using a pattern based on queue notification and Azure Functions, use another pattern for this data flow.</span></span> <span data-ttu-id="8a627-148">Ha például [Azure Service Bus-témakörök] [ topics] segítségével párhuzamosan, a cikk különböző részeinek kommunikálniuk kész ebben a példában a soros feldolgozási folyamatokat.</span><span class="sxs-lookup"><span data-stu-id="8a627-148">For example, [Azure Service Bus Topics][topics] can be used to processes the various parts of the article in parallel as opposed to the serial processing done in this example.</span></span> <span data-ttu-id="8a627-149">További információkért olvassa el a [üzenetsorok és témakörök][queues-topics].</span><span class="sxs-lookup"><span data-stu-id="8a627-149">For more information, compare [queues and topics][queues-topics].</span></span>

* <span data-ttu-id="8a627-150">Használat [Azure Logic Apps] [ logic-app] megvalósítása a függvénykódot és a végrehajtásához a rekord szintű zárolás például [Redlock] [ redlock] (párhuzamos szükséges amíg az Azure Cosmos DB támogatja a [részleges dokumentum frissítések][partial]).</span><span class="sxs-lookup"><span data-stu-id="8a627-150">Use [Azure Logic App][logic-app] to implement the function code and implement record-level locking such as [Redlock][redlock] (needed for parallel processing until Azure Cosmos DB supports [partial document updates][partial]).</span></span> <span data-ttu-id="8a627-151">További információ [funkciók és a Logic Apps összehasonlítása][compare].</span><span class="sxs-lookup"><span data-stu-id="8a627-151">For more information, [compare Functions and Logic Apps][compare].</span></span>

* <span data-ttu-id="8a627-152">Ez az architektúra a meglévő Azure-szolgáltatások helyett egyéni AI-összetevők használatával hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="8a627-152">Implement this architecture using customized AI components rather than existing Azure services.</span></span> <span data-ttu-id="8a627-153">Például az a folyamat használatával egy testre szabott modellt, amely észleli az adott személyek figyelésekor az általános személyek száma, a Nemet, a kép kiterjesztése, és ebben a példában az életkor adatokat gyűjteni.</span><span class="sxs-lookup"><span data-stu-id="8a627-153">For example, extend the pipeline using a customized model that detects certain people in an image as opposed to the generic people count, gender, and age data collected in this example.</span></span> <span data-ttu-id="8a627-154">Testre szabott gépi tanulási és AI-modellek használata ebben az architektúrában, hozhat létre a modellek RESTful végpontokon, így azok nem hívható meg az Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="8a627-154">To use customized machine learning or AI models with this architecture, build the models as RESTful endpoints so they can be called from Azure Functions.</span></span>

* <span data-ttu-id="8a627-155">Egy másik bemeneti mechanizmus használata az RSS-hírcsatornák helyett.</span><span class="sxs-lookup"><span data-stu-id="8a627-155">Use a different input mechanism instead of RSS feeds.</span></span> <span data-ttu-id="8a627-156">Több generátorok vagy betöltési folyamatok segítségével Azure Cosmos DB és az Azure Storage-hírcsatorna.</span><span class="sxs-lookup"><span data-stu-id="8a627-156">Use multiple generators or ingestion processes to feed Azure Cosmos DB and Azure Storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="8a627-157">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="8a627-157">Considerations</span></span>

<span data-ttu-id="8a627-158">Ebben a példaforgatókönyvben az egyszerűség kedvéért használja a mindössze néhány elérhető API-k és az Azure Cognitive Services szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="8a627-158">For simplicity, this example scenario uses only a few of the available APIs and services from Azure Cognitive Services.</span></span> <span data-ttu-id="8a627-159">Például képek képes elemezni a [Text Analytics API][text-analytics].</span><span class="sxs-lookup"><span data-stu-id="8a627-159">For example, text in images can be analyzed using the [Text Analytics API][text-analytics].</span></span> <span data-ttu-id="8a627-160">Ebben a forgatókönyvben a Célnyelv adatforrásmérete angol, de módosíthatja a bemenetet bármely [támogatott nyelven] [ language] tetszőleges.</span><span class="sxs-lookup"><span data-stu-id="8a627-160">The target language in this scenario is assumed to be English, but you can change the input to any [supported language][language] of your choice.</span></span>

### <a name="scalability"></a><span data-ttu-id="8a627-161">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="8a627-161">Scalability</span></span>

<span data-ttu-id="8a627-162">Az Azure Functions méretezése függ a [szolgáltatási csomag] [ plan] használja.</span><span class="sxs-lookup"><span data-stu-id="8a627-162">Azure Functions scaling depends on the [hosting plan][plan] you use.</span></span> <span data-ttu-id="8a627-163">Ez a megoldás feltételezi, hogy egy [Használatalapú csomag][plan-c], mely számítási power lefoglalása automatikusan történik az funkciók, szükség esetén.</span><span class="sxs-lookup"><span data-stu-id="8a627-163">This solution assumes a [Consumption plan][plan-c], in which compute power is automatically allocated to the functions when required.</span></span> <span data-ttu-id="8a627-164">A fizetés csak amikor a függvények futnak.</span><span class="sxs-lookup"><span data-stu-id="8a627-164">You pay only when your functions are running.</span></span> <span data-ttu-id="8a627-165">Egy másik lehetőség egy [Azure App Service] [ plan-aas] csomagra, amely lehetővé teszi a különböző mennyiségű erőforrás lefoglalásához a rétegek közötti skálázását.</span><span class="sxs-lookup"><span data-stu-id="8a627-165">Another option is to use an [Azure App Service][plan-aas] plan, which allows you to scale between tiers to allocate a different amount of resources.</span></span>

<span data-ttu-id="8a627-166">Az Azure Cosmos DB, a fontos, hogy a munkaterhelés nagyjából egyenletes elosztása bizonyos megfelelően nagy számú [kulcsok particionálása][keys].</span><span class="sxs-lookup"><span data-stu-id="8a627-166">With Azure Cosmos DB, the key is to distribute your workload roughly evenly among a sufficiently large number of [partition keys][keys].</span></span> <span data-ttu-id="8a627-167">Egy tárolóban tárolt adatok teljes mennyisége vagy a teljes mennyisége nem korlátozott [átviteli] [ throughput] egy tároló által támogatott.</span><span class="sxs-lookup"><span data-stu-id="8a627-167">There's no limit to the total amount of data that a container can store or to the total amount of [throughput][throughput] that a container can support.</span></span>

### <a name="management-and-logging"></a><span data-ttu-id="8a627-168">Kezelés és naplózás</span><span class="sxs-lookup"><span data-stu-id="8a627-168">Management and logging</span></span>

<span data-ttu-id="8a627-169">Ez a megoldás használ [Application Insights] [ aai] teljesítmény és a naplózási információk gyűjtéséhez.</span><span class="sxs-lookup"><span data-stu-id="8a627-169">This solution uses [Application Insights][aai] to collect performance and logging information.</span></span> <span data-ttu-id="8a627-170">Az Application Insights egy példányát a központi telepítést, mint az üzembe helyezéshez szükséges egyéb szolgáltatásokat ugyanabban az erőforráscsoportban jön létre.</span><span class="sxs-lookup"><span data-stu-id="8a627-170">An instance of Application Insights is created with the deployment in the same resource group as the other services needed for this deployment.</span></span>

<span data-ttu-id="8a627-171">A megoldás által létrehozott naplók megtekintése:</span><span class="sxs-lookup"><span data-stu-id="8a627-171">To view the logs generated by the solution:</span></span>

1. <span data-ttu-id="8a627-172">Lépjen a [az Azure portal] [ portal] , és keresse meg az üzembe helyezéshez létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="8a627-172">Go to [Azure portal][portal] and navigate to the resource group created for the deployment.</span></span>

2. <span data-ttu-id="8a627-173">Kattintson a **Application Insights** példány.</span><span class="sxs-lookup"><span data-stu-id="8a627-173">Click the **Application Insights** instance.</span></span>

3. <span data-ttu-id="8a627-174">Az a **Application Insights** területén lépjen **vizsgálat\\keresési** , és keresse meg az adatokat.</span><span class="sxs-lookup"><span data-stu-id="8a627-174">From the **Application Insights** section, navigate to **Investigate\\Search** and search the data.</span></span>

### <a name="security"></a><span data-ttu-id="8a627-175">Biztonság</span><span class="sxs-lookup"><span data-stu-id="8a627-175">Security</span></span>

<span data-ttu-id="8a627-176">Az Azure Cosmos DB használ a biztonságos kapcsolódás és a közös hozzáférésű jogosultságkód – a C\# SDK-t a Microsoft által biztosított.</span><span class="sxs-lookup"><span data-stu-id="8a627-176">Azure Cosmos DB uses a secured connection and shared access signature through the C\# SDK provided by Microsoft.</span></span> <span data-ttu-id="8a627-177">Nincsenek nincs más kifelé irányuló surface területeket.</span><span class="sxs-lookup"><span data-stu-id="8a627-177">There are no other externally facing surface areas.</span></span> <span data-ttu-id="8a627-178">További tudnivalók a biztonságról [ajánlott eljárások] [ db-practices] az Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="8a627-178">Learn more about security [best practices][db-practices] for Azure Cosmos DB.</span></span>

## <a name="pricing"></a><span data-ttu-id="8a627-179">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="8a627-179">Pricing</span></span>

<span data-ttu-id="8a627-180">A becsült napi költség, hogy a központi telepítés rendszer körülbelül \$20 áthelyezése a rendszeren keresztül adatot nem tartalmazó régiója.</span><span class="sxs-lookup"><span data-stu-id="8a627-180">The estimated daily cost to keep this deployment available is approximately \$20 U.S. with no data moving through the system.</span></span>

<span data-ttu-id="8a627-181">Az Azure Cosmos DB hatékony, de a lehető legnagyobb tekintetében [költség] [ db-cost] ebben a telepítésben.</span><span class="sxs-lookup"><span data-stu-id="8a627-181">Azure Cosmos DB is powerful but incurs the greatest [cost][db-cost] in this deployment.</span></span> <span data-ttu-id="8a627-182">Egy másik tárolási megoldás az Azure Functions kódjaiba, a megadott újrabontás használhatja.</span><span class="sxs-lookup"><span data-stu-id="8a627-182">You can use another storage solution by refactoring the Azure Functions code provided.</span></span>

<span data-ttu-id="8a627-183">Díjszabás attól függően változik, az Azure Functions a [terv] [ function-plan] fusson.</span><span class="sxs-lookup"><span data-stu-id="8a627-183">Pricing for Azure Functions varies depending on the [plan][function-plan] it runs in.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="8a627-184">A forgatókönyv megvalósításához</span><span class="sxs-lookup"><span data-stu-id="8a627-184">Deploy the scenario</span></span>

> [!NOTE]
> <span data-ttu-id="8a627-185">Meglévő Azure-fiókkal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="8a627-185">You must have an existing Azure account.</span></span> <span data-ttu-id="8a627-186">Ha nem rendelkezik Azure-előfizetéssel, első lépésként létrehozhat egy [ingyenes][free] fiókot.</span><span class="sxs-lookup"><span data-stu-id="8a627-186">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="8a627-187">Ebben a forgatókönyvben a kód érhető el a [GitHub] [ github] tárház.</span><span class="sxs-lookup"><span data-stu-id="8a627-187">All the code for this scenario is available in the [GitHub][github] repository.</span></span> <span data-ttu-id="8a627-188">Ez a tárház tartalmazza az eseménylétrehozó alkalmazást, amely a folyamat ebben a bemutatóban hírcsatornák összeállításához forráskódját.</span><span class="sxs-lookup"><span data-stu-id="8a627-188">This repository contains the source code used to build the generator application that feeds the pipeline for this demo.</span></span>

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
