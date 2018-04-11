---
title: Webalkalmazás-várólista-munkavégző architektúra stílus
description: Előnyeit, kihívást és ajánlott eljárások a webalkalmazás-várólista-munkavégző architektúrák ismerteti az Azure-on
author: MikeWasson
ms.openlocfilehash: 545472e71ffcd43717ad24af0dc9218a221ca910
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="f9f59-103">Webalkalmazás-várólista-munkavégző architektúra stílus</span><span class="sxs-lookup"><span data-stu-id="f9f59-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="f9f59-104">Ebbe az architektúrába alapvető összetevői vannak egy **előtér-webkiszolgáló** ügyfélkérelmek, szolgál, és egy **munkavégző** , erőforrás-igényes feladatok, hosszan futó munkafolyamatok vagy kötegelt feladatot hajt végre.</span><span class="sxs-lookup"><span data-stu-id="f9f59-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="f9f59-105">A dolgozó keresztül kommunikál a előtér egy **üzenet-várólista**.</span><span class="sxs-lookup"><span data-stu-id="f9f59-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="f9f59-106">Más összetevők, amelyek gyakran vannak építve az architektúra a következők:</span><span class="sxs-lookup"><span data-stu-id="f9f59-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="f9f59-107">Egy vagy több adatbázist.</span><span class="sxs-lookup"><span data-stu-id="f9f59-107">One or more databases.</span></span> 
- <span data-ttu-id="f9f59-108">A gyorsítótár az adatbázisból tárolásához a gyors olvasása.</span><span class="sxs-lookup"><span data-stu-id="f9f59-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="f9f59-109">Statikus tartalom szolgáltatásához CDN</span><span class="sxs-lookup"><span data-stu-id="f9f59-109">CDN to serve static content</span></span>
- <span data-ttu-id="f9f59-110">Távoli szolgáltatások, például az e-mailben vagy SMS-szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="f9f59-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="f9f59-111">Gyakran ezek által biztosított harmadik felek számára.</span><span class="sxs-lookup"><span data-stu-id="f9f59-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="f9f59-112">Identitásszolgáltató-hitelesítéshez.</span><span class="sxs-lookup"><span data-stu-id="f9f59-112">Identity provider for authentication.</span></span>

<span data-ttu-id="f9f59-113">A webes és feldolgozói olyan állapot nélküli is.</span><span class="sxs-lookup"><span data-stu-id="f9f59-113">The web and worker are both stateless.</span></span> <span data-ttu-id="f9f59-114">A munkamenet-állapot egy elosztott gyorsítótár is tárolható.</span><span class="sxs-lookup"><span data-stu-id="f9f59-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="f9f59-115">Hosszan futó munka a dolgozó aszinkron módon végezhető el.</span><span class="sxs-lookup"><span data-stu-id="f9f59-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="f9f59-116">A munkavégző üzenetek várakozási által kiváltott, vagy kötegelt feldolgozásra ütemezhető.</span><span class="sxs-lookup"><span data-stu-id="f9f59-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="f9f59-117">A dolgozó választható összetevőként.</span><span class="sxs-lookup"><span data-stu-id="f9f59-117">The worker is an optional component.</span></span> <span data-ttu-id="f9f59-118">Ha hosszú ideig futó művelet, a munkavégző elhagyható.</span><span class="sxs-lookup"><span data-stu-id="f9f59-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="f9f59-119">Az előtérbeli webes API-k állhat.</span><span class="sxs-lookup"><span data-stu-id="f9f59-119">The front end might consist of a web API.</span></span> <span data-ttu-id="f9f59-120">Ügyféloldali a webes API képes használni egy egyoldalas alkalmazás, amely lehetővé teszi az AJAX-hívások, illetve egy webalkalmazást.</span><span class="sxs-lookup"><span data-stu-id="f9f59-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="f9f59-121">Mikor érdemes használni, ez az architektúra</span><span class="sxs-lookup"><span data-stu-id="f9f59-121">When to use this architecture</span></span>

<span data-ttu-id="f9f59-122">A webalkalmazás-várólista-munkavégző architektúra általában felügyelt számítási szolgáltatás, vagy az Azure App Service szolgáltatásban, vagy az Azure Cloud Services segítségével van megvalósítva.</span><span class="sxs-lookup"><span data-stu-id="f9f59-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="f9f59-123">Vegye figyelembe a architektúra stílus:</span><span class="sxs-lookup"><span data-stu-id="f9f59-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="f9f59-124">Viszonylag egyszerű tartománnyal rendelkező alkalmazások.</span><span class="sxs-lookup"><span data-stu-id="f9f59-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="f9f59-125">Az alkalmazásoknak néhány hosszan futó munkafolyamatok vagy kötegelt műveleteket.</span><span class="sxs-lookup"><span data-stu-id="f9f59-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="f9f59-126">Ha használni kívánt felügyelt szolgáltatásokat, nem pedig infrastruktúra (IaaS) szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="f9f59-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="f9f59-127">Előnyök</span><span class="sxs-lookup"><span data-stu-id="f9f59-127">Benefits</span></span>

- <span data-ttu-id="f9f59-128">Viszonylag egyszerű architektúra, amely könnyen értelmezhető.</span><span class="sxs-lookup"><span data-stu-id="f9f59-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="f9f59-129">Könnyű telepítése és kezelése.</span><span class="sxs-lookup"><span data-stu-id="f9f59-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="f9f59-130">Egyértelmű elkülönítése vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="f9f59-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="f9f59-131">Az előtér a rendszer leválasztja a dolgozó aszinkron üzenetküldéshez használ.</span><span class="sxs-lookup"><span data-stu-id="f9f59-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="f9f59-132">Az előtér- és a feldolgozói is méretezhető egymástól függetlenül.</span><span class="sxs-lookup"><span data-stu-id="f9f59-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="f9f59-133">Kihívásai</span><span class="sxs-lookup"><span data-stu-id="f9f59-133">Challenges</span></span>

- <span data-ttu-id="f9f59-134">Gondos tervezés nélkül az előtér- és a feldolgozói válhat nagy, egységes összetevők, amelyek a nehezen karbantartása és frissítése.</span><span class="sxs-lookup"><span data-stu-id="f9f59-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="f9f59-135">Nem lehet rejtett függőségei, ha az előtér- és munkavégző adatok sémák vagy kódmodulok megosztása.</span><span class="sxs-lookup"><span data-stu-id="f9f59-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="f9f59-136">Ajánlott eljárások</span><span class="sxs-lookup"><span data-stu-id="f9f59-136">Best practices</span></span>

- <span data-ttu-id="f9f59-137">Az ügyfél egy tetszetős API tenni.</span><span class="sxs-lookup"><span data-stu-id="f9f59-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="f9f59-138">Lásd: [API ajánlott tervezési eljárásokat][api-design].</span><span class="sxs-lookup"><span data-stu-id="f9f59-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="f9f59-139">Automatikus skálázás módosítások a terhelés kezelésére.</span><span class="sxs-lookup"><span data-stu-id="f9f59-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="f9f59-140">Lásd: [automatikus skálázás gyakorlati tanácsok][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="f9f59-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="f9f59-141">Gyorsítótár félig statikus adatok.</span><span class="sxs-lookup"><span data-stu-id="f9f59-141">Cache semi-static data.</span></span> <span data-ttu-id="f9f59-142">Lásd: [gyakorlati tanácsok gyorsítótárazás][caching].</span><span class="sxs-lookup"><span data-stu-id="f9f59-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="f9f59-143">A CDN és a gazdagép statikus tartalmat használja.</span><span class="sxs-lookup"><span data-stu-id="f9f59-143">Use a CDN to host static content.</span></span> <span data-ttu-id="f9f59-144">Lásd: [CDN gyakorlati tanácsok][cdn].</span><span class="sxs-lookup"><span data-stu-id="f9f59-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="f9f59-145">Adott esetben polyglot adatmegőrzési használja.</span><span class="sxs-lookup"><span data-stu-id="f9f59-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="f9f59-146">Lásd: [használjon a legjobb adatok tárolásához a feladat][polyglot].</span><span class="sxs-lookup"><span data-stu-id="f9f59-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="f9f59-147">Partíció adatot, ezzel növelve a méretezhetőséget, versengés csökkentése és a teljesítmény optimalizálása.</span><span class="sxs-lookup"><span data-stu-id="f9f59-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="f9f59-148">Lásd: [gyakorlati tanácsok particionálás adatok][data-partition].</span><span class="sxs-lookup"><span data-stu-id="f9f59-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="f9f59-149">Az Azure App Service Web-várólista-Worker</span><span class="sxs-lookup"><span data-stu-id="f9f59-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="f9f59-150">Ez a szakasz ismerteti a ajánlott webes-várólista-munkavégző architektúra, amely használja az Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="f9f59-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="f9f59-151">Az előtér egy Azure App Service webalkalmazásba, lett megvalósítva, és a dolgozó, webjobs-feladat lett megvalósítva.</span><span class="sxs-lookup"><span data-stu-id="f9f59-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="f9f59-152">A web app és a webjobs-feladat az App Service-csomag, amely a Virtuálisgép-példányok társított.</span><span class="sxs-lookup"><span data-stu-id="f9f59-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="f9f59-153">Az üzenet-várólista vagy az Azure Service Bus, vagy az Azure Storage várólisták használható.</span><span class="sxs-lookup"><span data-stu-id="f9f59-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="f9f59-154">(Az ábrán látható az Azure Storage üzenetsorába.)</span><span class="sxs-lookup"><span data-stu-id="f9f59-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="f9f59-155">Azure Redis Cache munkamenet-állapot és az egyéb kis késésű hozzáférést igénylő adatokat tárolja.</span><span class="sxs-lookup"><span data-stu-id="f9f59-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="f9f59-156">Az Azure CDN használatával például képek, CSS vagy HTML statikus tartalom gyorsítótárazása.</span><span class="sxs-lookup"><span data-stu-id="f9f59-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="f9f59-157">A tároláshoz válassza ki a tárolási technológiákat, az alkalmazás igényeinek leginkább illő.</span><span class="sxs-lookup"><span data-stu-id="f9f59-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="f9f59-158">Használhat több tárolási technológiákat (polyglot adatmegőrzési).</span><span class="sxs-lookup"><span data-stu-id="f9f59-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="f9f59-159">Az ábrán látható, az Azure SQL Database és Azure Cosmos DB ezt az elképzelést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="f9f59-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="f9f59-160">További részletekért lásd: [App Service web alkalmazás referencia-architektúrában][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="f9f59-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="f9f59-161">Néhány fontos megjegyzés</span><span class="sxs-lookup"><span data-stu-id="f9f59-161">Additional considerations</span></span>

- <span data-ttu-id="f9f59-162">Nem minden tranzakcióban halad át a várólista és a feldolgozói tárhelyre.</span><span class="sxs-lookup"><span data-stu-id="f9f59-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="f9f59-163">Az előtér-webkiszolgáló közvetlenül egyszerű olvasási/írási műveleteket hajthat végre.</span><span class="sxs-lookup"><span data-stu-id="f9f59-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="f9f59-164">Erőforrás-igényes feladatok vagy hosszan futó munkafolyamatok munkavállalók készültek.</span><span class="sxs-lookup"><span data-stu-id="f9f59-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="f9f59-165">Bizonyos esetekben előfordulhat, nem kell egy munkavégző egyáltalán.</span><span class="sxs-lookup"><span data-stu-id="f9f59-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="f9f59-166">A beépített automatikus skálázási funkciót, az App Service segítségével terjessze ki a Virtuálisgép-példányok száma.</span><span class="sxs-lookup"><span data-stu-id="f9f59-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="f9f59-167">Ha az alkalmazás terhelésének előre jelezhető mintákat követi, használja az ütemezés alapján automatikus skálázási.</span><span class="sxs-lookup"><span data-stu-id="f9f59-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="f9f59-168">Ha a terhelést előre nem látható, a metrikák-alapú automatikus skálázás szabályok használata.</span><span class="sxs-lookup"><span data-stu-id="f9f59-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="f9f59-169">Érdemes a webalkalmazást és a webjobs-feladat üzembe külön App Service-csomagokról.</span><span class="sxs-lookup"><span data-stu-id="f9f59-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="f9f59-170">Ily módon, ezeket külön Virtuálisgép-példányok üzemelnek, és egymástól függetlenül is méretezhető.</span><span class="sxs-lookup"><span data-stu-id="f9f59-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="f9f59-171">Használjon külön App Service-csomagokról éles és tesztelését.</span><span class="sxs-lookup"><span data-stu-id="f9f59-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="f9f59-172">Ellenkező esetben termelésre használatakor a csomagot, akkor a tesztet futtatja a virtuális gépek éles.</span><span class="sxs-lookup"><span data-stu-id="f9f59-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="f9f59-173">Üzembe helyezési segítségével központi telepítések felügyeletéhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="f9f59-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="f9f59-174">Ez lehetővé teszi egy átmeneti helyet frissített verzióját telepítse, majd az új verzióra keresztül felcserélése.</span><span class="sxs-lookup"><span data-stu-id="f9f59-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="f9f59-175">Ezenkívül lehetővé teszi vissza az előző verzió a felcserélése, ha a frissítés probléma merült fel.</span><span class="sxs-lookup"><span data-stu-id="f9f59-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md