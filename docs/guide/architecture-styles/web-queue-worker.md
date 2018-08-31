---
title: Webüzenetsor-feldolgozó architektúrastílus
description: Az Azure webüzenetsor-feldolgozó architektúráinak előnyeit, kihívásait és ajánlott eljárásait ismerteti
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 0ebcf49c08c74cec3f1820da2d6f30ba95256e81
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325351"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="736d3-103">Webüzenetsor-feldolgozó architektúrastílus</span><span class="sxs-lookup"><span data-stu-id="736d3-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="736d3-104">Az ilyen architektúrák alapelemei: egy **webes előtér**, amely kiszolgálja az ügyfélkérelmeket, és egy **feldolgozó**, amely erőforrás-igényes munkákat, hosszan futó munkafolyamatokat vagy kötegelt feladatokat hajt végre.</span><span class="sxs-lookup"><span data-stu-id="736d3-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="736d3-105">A webes előtér egy **üzenetsor** segítségével kommunikál a feldolgozóval.</span><span class="sxs-lookup"><span data-stu-id="736d3-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="736d3-106">Az ebbe az architektúrába gyakran beillesztett más összetevők:</span><span class="sxs-lookup"><span data-stu-id="736d3-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="736d3-107">Egy vagy több adatbázis.</span><span class="sxs-lookup"><span data-stu-id="736d3-107">One or more databases.</span></span> 
- <span data-ttu-id="736d3-108">Egy gyorsítótár az adatbázis adatainak tárolására a gyors olvasás érdekében.</span><span class="sxs-lookup"><span data-stu-id="736d3-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="736d3-109">CDN a statikus tartalom kiszolgálásához</span><span class="sxs-lookup"><span data-stu-id="736d3-109">CDN to serve static content</span></span>
- <span data-ttu-id="736d3-110">Távoli szolgáltatások, például e-mail- vagy SMS-szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="736d3-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="736d3-111">Ezeket gyakran egy külső fél biztosítja.</span><span class="sxs-lookup"><span data-stu-id="736d3-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="736d3-112">Identitásszolgáltató a hitelesítéshez.</span><span class="sxs-lookup"><span data-stu-id="736d3-112">Identity provider for authentication.</span></span>

<span data-ttu-id="736d3-113">A web és a feldolgozó is állapot nélküli.</span><span class="sxs-lookup"><span data-stu-id="736d3-113">The web and worker are both stateless.</span></span> <span data-ttu-id="736d3-114">A munkamenet-állapot tárolható egy megosztott gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="736d3-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="736d3-115">A hosszan futó munkákat a feldolgozó aszinkron módon végzi el.</span><span class="sxs-lookup"><span data-stu-id="736d3-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="736d3-116">A feldolgozó aktiválható üzenetekkel az üzenetsoron, vagy futhat egy ütemezésben a kötegelt feldolgozáshoz.</span><span class="sxs-lookup"><span data-stu-id="736d3-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="736d3-117">A feldolgozó egy választható összetevő.</span><span class="sxs-lookup"><span data-stu-id="736d3-117">The worker is an optional component.</span></span> <span data-ttu-id="736d3-118">Ha nincs hosszan futó művelet, a feldolgozó elhagyható.</span><span class="sxs-lookup"><span data-stu-id="736d3-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="736d3-119">Az előtér állhat egy webes API-ból.</span><span class="sxs-lookup"><span data-stu-id="736d3-119">The front end might consist of a web API.</span></span> <span data-ttu-id="736d3-120">Ügyféloldalon a webes API-t használhatja egy AJAX-hívásokat indító egyoldalas alkalmazás vagy egy natív ügyfélalkalmazás.</span><span class="sxs-lookup"><span data-stu-id="736d3-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="736d3-121">Mikor érdemes ezt az architektúrát használni?</span><span class="sxs-lookup"><span data-stu-id="736d3-121">When to use this architecture</span></span>

<span data-ttu-id="736d3-122">A webüzenetsor-feldolgozó architektúra implementálása általában felügyelt számítási szolgáltatások használatával történik, vagy az Azure App Service vagy az Azure Cloud Services segítségével.</span><span class="sxs-lookup"><span data-stu-id="736d3-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="736d3-123">Fontolja meg ennek az architektúrastílusnak a használatát a következőkhöz:</span><span class="sxs-lookup"><span data-stu-id="736d3-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="736d3-124">Alkalmazások viszonylag egyszerű tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="736d3-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="736d3-125">Alkalmazások néhány hosszan futó munkafolyamattal vagy kötegelt műveletekkel.</span><span class="sxs-lookup"><span data-stu-id="736d3-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="736d3-126">Ha felügyelt szolgáltatásokat szeretne használni szolgáltatott infrastruktúra (IaaS) helyett.</span><span class="sxs-lookup"><span data-stu-id="736d3-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="736d3-127">Előnyök</span><span class="sxs-lookup"><span data-stu-id="736d3-127">Benefits</span></span>

- <span data-ttu-id="736d3-128">Viszonylag egyszerű és könnyen érthető architektúra.</span><span class="sxs-lookup"><span data-stu-id="736d3-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="736d3-129">Könnyen telepíthető és felügyelhető.</span><span class="sxs-lookup"><span data-stu-id="736d3-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="736d3-130">A kockázatok egyértelmű elkülönítése.</span><span class="sxs-lookup"><span data-stu-id="736d3-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="736d3-131">Az előteret a rendszer aszinkron üzenetküldés segítségével választja le a feldolgozóról.</span><span class="sxs-lookup"><span data-stu-id="736d3-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="736d3-132">Az előtér és a feldolgozó egymástól függetlenül skálázható.</span><span class="sxs-lookup"><span data-stu-id="736d3-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="736d3-133">Problémák</span><span class="sxs-lookup"><span data-stu-id="736d3-133">Challenges</span></span>

- <span data-ttu-id="736d3-134">Gondos tervezés nélkül az előtér és a feldolgozó hatalmas, monolitikus, végül nehezen karbantartható és frissíthető összetevővé válhat.</span><span class="sxs-lookup"><span data-stu-id="736d3-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="736d3-135">Lehetnek rejtett függőségek, ha az előtér és a feldolgozó közös sémákat vagy kódmodulokat használ.</span><span class="sxs-lookup"><span data-stu-id="736d3-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="736d3-136">Ajánlott eljárások</span><span class="sxs-lookup"><span data-stu-id="736d3-136">Best practices</span></span>

- <span data-ttu-id="736d3-137">Jól megtervezett API közzététele az ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="736d3-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="736d3-138">Tekintse meg az [API-tervezés – Ajánlott eljárások][api-design] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="736d3-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="736d3-139">Automatikus skálázás a terhelés változásának kezelésére.</span><span class="sxs-lookup"><span data-stu-id="736d3-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="736d3-140">Lásd az [automatikus skálázással kapcsolatos ajánlott eljárásokat][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="736d3-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="736d3-141">Gyorsítótárazza a félig statikus adatokat.</span><span class="sxs-lookup"><span data-stu-id="736d3-141">Cache semi-static data.</span></span> <span data-ttu-id="736d3-142">Lásd a [gyorsítótárazás ajánlott eljárásait][caching].</span><span class="sxs-lookup"><span data-stu-id="736d3-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="736d3-143">CDN használata a statikus tartalom üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="736d3-143">Use a CDN to host static content.</span></span> <span data-ttu-id="736d3-144">Tekintse meg az [Ajánlott tanácsok az Azure CDN-nel kapcsolatban][cdn] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="736d3-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="736d3-145">Többnyelvű adatmegőrzés használata, ha szükséges.</span><span class="sxs-lookup"><span data-stu-id="736d3-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="736d3-146">Tekintse meg a [Használja a feladathoz legmegfelelőbb adattárat][polyglot] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="736d3-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="736d3-147">Adatok particionálása a méretezhetőség növeléséhez, a versengés kiküszöböléséhez és a teljesítmény optimalizálásához.</span><span class="sxs-lookup"><span data-stu-id="736d3-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="736d3-148">Tekintse meg az [Ajánlott adatparticionálási eljárások][data-partition] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="736d3-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="736d3-149">A webüzenetsor-feldolgozó az Azure App Service-en</span><span class="sxs-lookup"><span data-stu-id="736d3-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="736d3-150">Ez a szakasz ismertet egy ajánlott, az Azure App Service-t használó webüzenetsor-feldolgozót.</span><span class="sxs-lookup"><span data-stu-id="736d3-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="736d3-151">Az előtér egy ismert Azure App Service webalkalmazásként, a feldolgozó pedig WebJob-feladatként van implementálva.</span><span class="sxs-lookup"><span data-stu-id="736d3-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="736d3-152">A webalkalmazás és a WebJob is társítva van egy App Service-csomaghoz, amely biztosítja a virtuálisgép-példányokat.</span><span class="sxs-lookup"><span data-stu-id="736d3-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="736d3-153">Az üzenetsorhoz használhat Azure Service Bus- vagy Azure Storage-üzenetsort is.</span><span class="sxs-lookup"><span data-stu-id="736d3-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="736d3-154">(Az ábrán egy Azure Storage-üzenetsor látható.)</span><span class="sxs-lookup"><span data-stu-id="736d3-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="736d3-155">Az Azure Redis Cache tárolja a munkamenet állapotát, és más, alacsony késésű elérést igénylő adatokat.</span><span class="sxs-lookup"><span data-stu-id="736d3-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="736d3-156">Az Azure CDN használatával gyorsítótárazható statikus tartalom, például képek, CSS vagy HTML.</span><span class="sxs-lookup"><span data-stu-id="736d3-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="736d3-157">A tároláshoz válassza ki az alkalmazás igényeinek legmegfelelőbb tárolási technológiákat.</span><span class="sxs-lookup"><span data-stu-id="736d3-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="736d3-158">Használhat több tárolási technológiát (többnyelvű adatmegőrzés).</span><span class="sxs-lookup"><span data-stu-id="736d3-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="736d3-159">Az elképzelés ábrázolására az ábra bemutatja az Azure SQL Database-t és az Azure Cosmos DB-t.</span><span class="sxs-lookup"><span data-stu-id="736d3-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="736d3-160">További részletek: [App Service-webalkalmazás referenciaarchitektúrája][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="736d3-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="736d3-161">Néhány fontos megjegyzés</span><span class="sxs-lookup"><span data-stu-id="736d3-161">Additional considerations</span></span>

- <span data-ttu-id="736d3-162">Nem minden tranzakciónak kell áthaladnia az üzenetsoron és a feldolgozón a tároló felé.</span><span class="sxs-lookup"><span data-stu-id="736d3-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="736d3-163">A webes előtér közvetlenül hajthat végre egyszerű olvasási/írási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="736d3-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="736d3-164">A feldolgozók erőforrás-igényes feladatokhoz vagy hosszan futó munkafolyamatokhoz lettek kialakítva.</span><span class="sxs-lookup"><span data-stu-id="736d3-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="736d3-165">Bizonyos esetekben előfordulhat, hogy nincs is szükség feldolgozóra.</span><span class="sxs-lookup"><span data-stu-id="736d3-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="736d3-166">Az App Service beépített automatikus skálázási szolgáltatásával horizontálisan felskálázhatja a virtuálisgép-példányok számát.</span><span class="sxs-lookup"><span data-stu-id="736d3-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="736d3-167">Ha az alkalmazás terhelése előre megjósolható mintákat követ, használjon ütemezésalapú automatikus skálázást.</span><span class="sxs-lookup"><span data-stu-id="736d3-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="736d3-168">Ha a terhelés nem jósolható meg előre, használjon mérőszám-alapú automatikus méretezési szabályokat.</span><span class="sxs-lookup"><span data-stu-id="736d3-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="736d3-169">Érdemes lehet a webalkalmazást és a WebJob-feladatot külön App Service-csomagba helyezni.</span><span class="sxs-lookup"><span data-stu-id="736d3-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="736d3-170">Ily módon üzemeltethetők külön virtuálisgép-példányokon, és skálázhatók egymástól függetlenül.</span><span class="sxs-lookup"><span data-stu-id="736d3-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="736d3-171">Az éles üzemhez és a teszteléshez használjon külön App Service-csomagot.</span><span class="sxs-lookup"><span data-stu-id="736d3-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="736d3-172">Ha ugyanis ugyanazt a csomagot használja az éles üzemhez és a teszteléshez, akkor a tesztek élesben működő virtuális gépeken futnak.</span><span class="sxs-lookup"><span data-stu-id="736d3-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="736d3-173">Az üzembe helyezések kezeléséhez használjon üzembehelyezési pontokat.</span><span class="sxs-lookup"><span data-stu-id="736d3-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="736d3-174">Ez lehetővé teszi, hogy telepítsen egy frissített verziót egy átmeneti helyen, majd átváltson az új verzióra.</span><span class="sxs-lookup"><span data-stu-id="736d3-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="736d3-175">Így arra is lehetősége van, hogy visszaváltson a korábbi verzióra, ha probléma adódna a frissítéssel.</span><span class="sxs-lookup"><span data-stu-id="736d3-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md