---
title: Képek besorolása a biztosítási kárigényekben, az Azure-ban
description: Bevált forgatókönyv képfeldolgozás, az Azure-alkalmazások készítéséhez.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 361a88234fd9ed918ab7664893f86666b4328b8c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060829"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="12238-103">Képek besorolása a biztosítási kárigényekben, az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="12238-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="12238-104">Ebben a példaforgatókönyvben akkor számára ideális, dolgozhassa fel.</span><span class="sxs-lookup"><span data-stu-id="12238-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="12238-105">Lehetséges alkalmazások közé tartozik a divat webhelyekhez képek besorolása, a szöveget és képeket a biztosítási követeléseket elemzése vagy a játék képernyőképek küldött telemetriai adatok megismerése.</span><span class="sxs-lookup"><span data-stu-id="12238-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="12238-106">Hagyományosan vállalatok lenne kell fejleszthet szaktudását a gépi tanulási modelleket, a modelleket taníthat be, és végül futtassa a rendszerképek saját egyéni folyamaton kívül a képek az adatok beolvasásához.</span><span class="sxs-lookup"><span data-stu-id="12238-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="12238-107">Azure-szolgáltatások például a Computer Vision API és az Azure Functions használatával a vállalatok szükségtelenné teszi a az egyes kiszolgálók felügyelete során költségeit, és azzal a szakértelemmel, amelyeket a Microsoft már körül feldolgozási rendszerképek kihasználva A cognitive services.</span><span class="sxs-lookup"><span data-stu-id="12238-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="12238-108">Ebben a forgatókönyvben kifejezetten címek egy kép feldolgozási forgatókönyvet.</span><span class="sxs-lookup"><span data-stu-id="12238-108">This scenario specifically addresses an image processing scenario.</span></span> <span data-ttu-id="12238-109">Ha eltérő AI igényeit, fontolja meg a teljes körű [Cognitive Services][cognitive-docs].</span><span class="sxs-lookup"><span data-stu-id="12238-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="12238-110">Kapcsolódó alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="12238-110">Related use cases</span></span>

<span data-ttu-id="12238-111">Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="12238-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="12238-112">Divat webhely-rendszerképek besorolása.</span><span class="sxs-lookup"><span data-stu-id="12238-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="12238-113">Játékok pillanatképeiért küldött telemetriai adatok besorolását.</span><span class="sxs-lookup"><span data-stu-id="12238-113">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="12238-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="12238-114">Architecture</span></span>

![Intelligens alkalmazások architektúra - számítógépes látástechnológia][architecture-computer-vision]

<span data-ttu-id="12238-116">Ebben a forgatókönyvben egy webes vagy mobilalkalmazásaiba háttér-összetevőinek ismerteti.</span><span class="sxs-lookup"><span data-stu-id="12238-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="12238-117">Áramlanak keresztül az adatok a forgatókönyv a következő:</span><span class="sxs-lookup"><span data-stu-id="12238-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="12238-118">Az Azure Functions az API-réteget funkcionál.</span><span class="sxs-lookup"><span data-stu-id="12238-118">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="12238-119">Ezen API-k engedélyezése az alkalmazás-rendszerképek feltöltése és a Cosmos DB-adatokat lekérni.</span><span class="sxs-lookup"><span data-stu-id="12238-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="12238-120">Képek feltöltése mikor keresztül egy API-hívás, Blob storage-ban van tárolva.</span><span class="sxs-lookup"><span data-stu-id="12238-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="12238-121">Új fájlok hozzáadásával a Blob storage elindít egy EventGrid értesítést kell küldeni egy Azure-függvényt.</span><span class="sxs-lookup"><span data-stu-id="12238-121">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="12238-122">Az Azure Functions egy hivatkozást küld az újonnan feltöltött fájlt a Computer Vision API elemzéséhez.</span><span class="sxs-lookup"><span data-stu-id="12238-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="12238-123">A Computer Vision API az adatok adott vissza, ha az Azure Functions feljegyzi megőrizni a képek metaadataira mellett az elemzés eredményeit a Cosmos DB-ben.</span><span class="sxs-lookup"><span data-stu-id="12238-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="12238-124">Összetevők</span><span class="sxs-lookup"><span data-stu-id="12238-124">Components</span></span>

* <span data-ttu-id="12238-125">[Computer Vision API] [ computer-vision-docs] a Cognitive Services-csomag része, és minden egyes képe kapcsolatos információk olvashatók be.</span><span class="sxs-lookup"><span data-stu-id="12238-125">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="12238-126">[Az Azure Functions] [ functions-docs] a háttérrendszeri API kínál a webes alkalmazás, valamint a feltöltött képek eseményfeldolgozás.</span><span class="sxs-lookup"><span data-stu-id="12238-126">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="12238-127">[Event Grid] [ eventgrid-docs] elindít egy eseményt, amikor a blob storage-bA feltöltött új lemezképet.</span><span class="sxs-lookup"><span data-stu-id="12238-127">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="12238-128">A lemezkép ezután feldolgozása az Azure functions használatával.</span><span class="sxs-lookup"><span data-stu-id="12238-128">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="12238-129">[A BLOB Storage-] [ storage-docs] tárolja az összes rendszerkép fájlt, amely a webes alkalmazás is bármely statikus fájlként a webes alkalmazás használ fel lesz töltve.</span><span class="sxs-lookup"><span data-stu-id="12238-129">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="12238-130">[A cosmos DB] [ cosmos-docs] minden egyes feltöltött, beleértve a Computer Vision API, a feldolgozás eredményét lemezkép metaadatait tárolja.</span><span class="sxs-lookup"><span data-stu-id="12238-130">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="12238-131">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="12238-131">Alternatives</span></span>

* <span data-ttu-id="12238-132">[Custom Vision Service][custom-vision-docs].</span><span class="sxs-lookup"><span data-stu-id="12238-132">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="12238-133">A Computer Vision API-készletet ad vissza, [besorolás-alapú kategóriák][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="12238-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="12238-134">Ha kell, amely nem a Computer Vision API által visszaadott adatok feldolgozásához, fontolja meg a Custom Vision Service, amely hozhat létre olyan egyéni rendszerkép osztályozó eszközökkel.</span><span class="sxs-lookup"><span data-stu-id="12238-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="12238-135">[Az Azure Search][azure-search-docs].</span><span class="sxs-lookup"><span data-stu-id="12238-135">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="12238-136">Ha a használati eset szerint az adott feltételnek-rendszerképek keresése a metaadatok lekérdezése, fontolja meg az Azure Search használatával.</span><span class="sxs-lookup"><span data-stu-id="12238-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="12238-137">Jelenleg előzetes verzióban elérhető [Cognitive search] [ cognitive-search] zökkenőmentesen integrálható az ebben a munkafolyamatban.</span><span class="sxs-lookup"><span data-stu-id="12238-137">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="12238-138">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="12238-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="12238-139">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="12238-139">Scalability</span></span>

<span data-ttu-id="12238-140">Az esetek többségében ez a forgatókönyv összetevőinek összes felügyelt szolgáltatások, amelyek automatikusan skálázzák.</span><span class="sxs-lookup"><span data-stu-id="12238-140">For the most part all of the components of this scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="12238-141">Néhány fontosabb kivételek: az Azure Functions egy legfeljebb 200 példányra határral rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="12238-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="12238-142">Ha ezen felüli van szüksége, fontolja meg több régióban vagy alkalmazás tervek.</span><span class="sxs-lookup"><span data-stu-id="12238-142">If you need to scale beyond, consider multiple regions or app plans.</span></span>

<span data-ttu-id="12238-143">A cosmos DB nem automatikus skálázás kiosztott kérelemegységek (RU) tekintetében.</span><span class="sxs-lookup"><span data-stu-id="12238-143">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="12238-144">A követelmények becsléséhez tekintse át [kérelemegységek] [ request-units] dokumentációban.</span><span class="sxs-lookup"><span data-stu-id="12238-144">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="12238-145">A teljes mértékben kihasználhatja a Cosmos DB méretezése meg kell is vessen egy pillantást [kulcsok particionálása][partition-key].</span><span class="sxs-lookup"><span data-stu-id="12238-145">To fully take advantage of the scaling in Cosmos DB you should also take a look at [partition keys][partition-key].</span></span>

<span data-ttu-id="12238-146">NoSQL-adatbázisok gyakran kereskedelmi rendelkezésre állását, méretezhetőségét és a partíció (abban az értelemben, a CAP-tétel) konzisztencia.</span><span class="sxs-lookup"><span data-stu-id="12238-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability and partition.</span></span>  <span data-ttu-id="12238-147">Azonban esetén kulcs-érték típusú adatmodelleket ebben a forgatókönyvben használt, tranzakció-konzisztencia ritkán szükség van, a legtöbb műveletekre atomi definíció szerint.</span><span class="sxs-lookup"><span data-stu-id="12238-147">However, in the case of key-value data models which is used in this scenario, transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="12238-148">További útmutatást [a megfelelő adattároló kiválasztása](../../guide/technology-choices/data-store-overview.md) az architektúra-központ érhető el.</span><span class="sxs-lookup"><span data-stu-id="12238-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the architecture center.</span></span>

<span data-ttu-id="12238-149">Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="12238-149">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="12238-150">Biztonság</span><span class="sxs-lookup"><span data-stu-id="12238-150">Security</span></span>

<span data-ttu-id="12238-151">[Felügyelt szolgáltatásidentitások] [ msi] (MSI) belső fiókjába más erőforrásokhoz való hozzáférést biztosítanak, és az Azure Functions majd hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="12238-151">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="12238-152">Csak a szükséges erőforrásokhoz való hozzáférés engedélyezése ezen identitások győződjön meg arról, hogy semmi sem extra ki van téve a függvények (és esetleg az ügyfelek számára).</span><span class="sxs-lookup"><span data-stu-id="12238-152">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="12238-153">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="12238-153">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="12238-154">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="12238-154">Resiliency</span></span>

<span data-ttu-id="12238-155">Ebben a forgatókönyvben a összetevőket felügyeli, így egy regionális szinten az összes rugalmas automatikusan.</span><span class="sxs-lookup"><span data-stu-id="12238-155">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="12238-156">Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="12238-156">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="12238-157">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="12238-157">Pricing</span></span>

<span data-ttu-id="12238-158">Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált.</span><span class="sxs-lookup"><span data-stu-id="12238-158">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="12238-159">Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.</span><span class="sxs-lookup"><span data-stu-id="12238-159">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="12238-160">Három példa költség-profilok forgalom mennyisége alapján biztosítunk (feltételezzük összes rendszerképekkel 100kb méretű):</span><span class="sxs-lookup"><span data-stu-id="12238-160">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100kb in size):</span></span>

* <span data-ttu-id="12238-161">[Kis][pricing]: Ez ad eredményül, amelyek feldolgozása &lt; 5000 képek egy hónapban.</span><span class="sxs-lookup"><span data-stu-id="12238-161">[Small][pricing]: this correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="12238-162">[Közepes][medium-pricing]: Ez havi 500 000 lemezképek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="12238-162">[Medium][medium-pricing]: this correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="12238-163">[Nagy][large-pricing]: Ez havi 50 milliót lemezképek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="12238-163">[Large][large-pricing]: this correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="12238-164">Kapcsolódó erőforrások</span><span class="sxs-lookup"><span data-stu-id="12238-164">Related Resources</span></span>

<span data-ttu-id="12238-165">A képzési ebben a forgatókönyvben, lásd: [egy kiszolgáló nélküli webalkalmazás létrehozása az Azure-ban][serverless].</span><span class="sxs-lookup"><span data-stu-id="12238-165">For a guided learning path of this scenario, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="12238-166">Mielőtt ez éles környezetben, tekintse át az Azure Functions [ajánlott eljárások][functions-best-practices].</span><span class="sxs-lookup"><span data-stu-id="12238-166">Before putting this in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data