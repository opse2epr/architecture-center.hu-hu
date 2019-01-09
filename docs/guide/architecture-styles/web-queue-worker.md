---
title: Webüzenetsor-feldolgozó architektúrastílus
titleSuffix: Azure Application Architecture Guide
description: Előnyeit, kihívásait és ajánlott eljárások a Webüzenetsor-feldolgozó architektúráinak ismerteti az Azure-ban.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 0b478344c4b64808d30156bd563917d9d8d7ec30
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113280"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="a7a30-103">Webüzenetsor-feldolgozó architektúrastílus</span><span class="sxs-lookup"><span data-stu-id="a7a30-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="a7a30-104">Az ilyen architektúrák alapelemei: egy **webes előtér**, amely kiszolgálja az ügyfélkérelmeket, és egy **feldolgozó**, amely erőforrás-igényes munkákat, hosszan futó munkafolyamatokat vagy kötegelt feladatokat hajt végre.</span><span class="sxs-lookup"><span data-stu-id="a7a30-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="a7a30-105">A webes előtér egy **üzenetsor** segítségével kommunikál a feldolgozóval.</span><span class="sxs-lookup"><span data-stu-id="a7a30-105">The web front end communicates with the worker through a **message queue**.</span></span>

![Webüzenetsor-feldolgozó architektúrastílus logikai diagramja](./images/web-queue-worker-logical.svg)

<span data-ttu-id="a7a30-107">Az ebbe az architektúrába gyakran beillesztett más összetevők:</span><span class="sxs-lookup"><span data-stu-id="a7a30-107">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="a7a30-108">Egy vagy több adatbázis.</span><span class="sxs-lookup"><span data-stu-id="a7a30-108">One or more databases.</span></span>
- <span data-ttu-id="a7a30-109">Egy gyorsítótár az adatbázis adatainak tárolására a gyors olvasás érdekében.</span><span class="sxs-lookup"><span data-stu-id="a7a30-109">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="a7a30-110">CDN a statikus tartalom kiszolgálásához</span><span class="sxs-lookup"><span data-stu-id="a7a30-110">CDN to serve static content</span></span>
- <span data-ttu-id="a7a30-111">Távoli szolgáltatások, például e-mail- vagy SMS-szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="a7a30-111">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="a7a30-112">Ezeket gyakran egy külső fél biztosítja.</span><span class="sxs-lookup"><span data-stu-id="a7a30-112">Often these are provided by third parties.</span></span>
- <span data-ttu-id="a7a30-113">Identitásszolgáltató a hitelesítéshez.</span><span class="sxs-lookup"><span data-stu-id="a7a30-113">Identity provider for authentication.</span></span>

<span data-ttu-id="a7a30-114">A web és a feldolgozó is állapot nélküli.</span><span class="sxs-lookup"><span data-stu-id="a7a30-114">The web and worker are both stateless.</span></span> <span data-ttu-id="a7a30-115">A munkamenet-állapot tárolható egy megosztott gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="a7a30-115">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="a7a30-116">A hosszan futó munkákat a feldolgozó aszinkron módon végzi el.</span><span class="sxs-lookup"><span data-stu-id="a7a30-116">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="a7a30-117">A feldolgozó aktiválható üzenetekkel az üzenetsoron, vagy futhat egy ütemezésben a kötegelt feldolgozáshoz.</span><span class="sxs-lookup"><span data-stu-id="a7a30-117">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="a7a30-118">A feldolgozó egy választható összetevő.</span><span class="sxs-lookup"><span data-stu-id="a7a30-118">The worker is an optional component.</span></span> <span data-ttu-id="a7a30-119">Ha nincs hosszan futó művelet, a feldolgozó elhagyható.</span><span class="sxs-lookup"><span data-stu-id="a7a30-119">If there are no long-running operations, the worker can be omitted.</span></span>

<span data-ttu-id="a7a30-120">Az előtér állhat egy webes API-ból.</span><span class="sxs-lookup"><span data-stu-id="a7a30-120">The front end might consist of a web API.</span></span> <span data-ttu-id="a7a30-121">Ügyféloldalon a webes API-t használhatja egy AJAX-hívásokat indító egyoldalas alkalmazás vagy egy natív ügyfélalkalmazás.</span><span class="sxs-lookup"><span data-stu-id="a7a30-121">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="a7a30-122">Mikor érdemes ezt az architektúrát használni?</span><span class="sxs-lookup"><span data-stu-id="a7a30-122">When to use this architecture</span></span>

<span data-ttu-id="a7a30-123">A webüzenetsor-feldolgozó architektúra implementálása általában felügyelt számítási szolgáltatások használatával történik, vagy az Azure App Service vagy az Azure Cloud Services segítségével.</span><span class="sxs-lookup"><span data-stu-id="a7a30-123">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span>

<span data-ttu-id="a7a30-124">Fontolja meg ennek az architektúrastílusnak a használatát a következőkhöz:</span><span class="sxs-lookup"><span data-stu-id="a7a30-124">Consider this architecture style for:</span></span>

- <span data-ttu-id="a7a30-125">Alkalmazások viszonylag egyszerű tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="a7a30-125">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="a7a30-126">Alkalmazások néhány hosszan futó munkafolyamattal vagy kötegelt műveletekkel.</span><span class="sxs-lookup"><span data-stu-id="a7a30-126">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="a7a30-127">Ha felügyelt szolgáltatásokat szeretne használni szolgáltatott infrastruktúra (IaaS) helyett.</span><span class="sxs-lookup"><span data-stu-id="a7a30-127">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="a7a30-128">Előnyök</span><span class="sxs-lookup"><span data-stu-id="a7a30-128">Benefits</span></span>

- <span data-ttu-id="a7a30-129">Viszonylag egyszerű és könnyen érthető architektúra.</span><span class="sxs-lookup"><span data-stu-id="a7a30-129">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="a7a30-130">Könnyen telepíthető és felügyelhető.</span><span class="sxs-lookup"><span data-stu-id="a7a30-130">Easy to deploy and manage.</span></span>
- <span data-ttu-id="a7a30-131">A kockázatok egyértelmű elkülönítése.</span><span class="sxs-lookup"><span data-stu-id="a7a30-131">Clear separation of concerns.</span></span>
- <span data-ttu-id="a7a30-132">Az előteret a rendszer aszinkron üzenetküldés segítségével választja le a feldolgozóról.</span><span class="sxs-lookup"><span data-stu-id="a7a30-132">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="a7a30-133">Az előtér és a feldolgozó egymástól függetlenül skálázható.</span><span class="sxs-lookup"><span data-stu-id="a7a30-133">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="a7a30-134">Problémák</span><span class="sxs-lookup"><span data-stu-id="a7a30-134">Challenges</span></span>

- <span data-ttu-id="a7a30-135">Gondos tervezés nélkül az előtér és a feldolgozó hatalmas, monolitikus, végül nehezen karbantartható és frissíthető összetevővé válhat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-135">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="a7a30-136">Lehetnek rejtett függőségek, ha az előtér és a feldolgozó közös sémákat vagy kódmodulokat használ.</span><span class="sxs-lookup"><span data-stu-id="a7a30-136">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span>

## <a name="best-practices"></a><span data-ttu-id="a7a30-137">Ajánlott eljárások</span><span class="sxs-lookup"><span data-stu-id="a7a30-137">Best practices</span></span>

- <span data-ttu-id="a7a30-138">Jól megtervezett API közzététele az ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="a7a30-138">Expose a well-designed API to the client.</span></span> <span data-ttu-id="a7a30-139">Tekintse meg az [API-tervezés – Ajánlott eljárások][api-design] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="a7a30-139">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="a7a30-140">Automatikus skálázás a terhelés változásának kezelésére.</span><span class="sxs-lookup"><span data-stu-id="a7a30-140">Autoscale to handle changes in load.</span></span> <span data-ttu-id="a7a30-141">Lásd az [automatikus skálázással kapcsolatos ajánlott eljárásokat][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="a7a30-141">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="a7a30-142">Gyorsítótárazza a félig statikus adatokat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-142">Cache semi-static data.</span></span> <span data-ttu-id="a7a30-143">Lásd a [gyorsítótárazás ajánlott eljárásait][caching].</span><span class="sxs-lookup"><span data-stu-id="a7a30-143">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="a7a30-144">CDN használata a statikus tartalom üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="a7a30-144">Use a CDN to host static content.</span></span> <span data-ttu-id="a7a30-145">Tekintse meg az [Ajánlott tanácsok az Azure CDN-nel kapcsolatban][cdn] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="a7a30-145">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="a7a30-146">Többnyelvű adatmegőrzés használata, ha szükséges.</span><span class="sxs-lookup"><span data-stu-id="a7a30-146">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="a7a30-147">Tekintse meg a [Használja a feladathoz legmegfelelőbb adattárat][polyglot] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="a7a30-147">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="a7a30-148">Adatok particionálása a méretezhetőség növeléséhez, a versengés kiküszöböléséhez és a teljesítmény optimalizálásához.</span><span class="sxs-lookup"><span data-stu-id="a7a30-148">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="a7a30-149">Tekintse meg az [Ajánlott adatparticionálási eljárások][data-partition] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="a7a30-149">See [Data partitioning best practices][data-partition].</span></span>

## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="a7a30-150">A webüzenetsor-feldolgozó az Azure App Service-en</span><span class="sxs-lookup"><span data-stu-id="a7a30-150">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="a7a30-151">Ez a szakasz ismertet egy ajánlott, az Azure App Service-t használó webüzenetsor-feldolgozót.</span><span class="sxs-lookup"><span data-stu-id="a7a30-151">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span>

![Webüzenetsor-feldolgozó architektúrastílus fizikai diagramja](./images/web-queue-worker-physical.png)

<span data-ttu-id="a7a30-153">Az előtér egy ismert Azure App Service webalkalmazásként, a feldolgozó pedig WebJob-feladatként van implementálva.</span><span class="sxs-lookup"><span data-stu-id="a7a30-153">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="a7a30-154">A webalkalmazás és a WebJob is társítva van egy App Service-csomaghoz, amely biztosítja a virtuálisgép-példányokat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-154">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span>

<span data-ttu-id="a7a30-155">Az üzenetsorhoz használhat Azure Service Bus- vagy Azure Storage-üzenetsort is.</span><span class="sxs-lookup"><span data-stu-id="a7a30-155">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="a7a30-156">(Az ábrán egy Azure Storage-üzenetsor látható.)</span><span class="sxs-lookup"><span data-stu-id="a7a30-156">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="a7a30-157">Az Azure Redis Cache tárolja a munkamenet állapotát, és más, alacsony késésű elérést igénylő adatokat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-157">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="a7a30-158">Az Azure CDN használatával gyorsítótárazható statikus tartalom, például képek, CSS vagy HTML.</span><span class="sxs-lookup"><span data-stu-id="a7a30-158">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="a7a30-159">A tároláshoz válassza ki az alkalmazás igényeinek legmegfelelőbb tárolási technológiákat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-159">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="a7a30-160">Használhat több tárolási technológiát (többnyelvű adatmegőrzés).</span><span class="sxs-lookup"><span data-stu-id="a7a30-160">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="a7a30-161">Az elképzelés ábrázolására az ábra bemutatja az Azure SQL Database-t és az Azure Cosmos DB-t.</span><span class="sxs-lookup"><span data-stu-id="a7a30-161">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>

<span data-ttu-id="a7a30-162">További részletek: [App Service-webalkalmazás referenciaarchitektúrája][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="a7a30-162">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="a7a30-163">Néhány fontos megjegyzés</span><span class="sxs-lookup"><span data-stu-id="a7a30-163">Additional considerations</span></span>

- <span data-ttu-id="a7a30-164">Nem minden tranzakciónak kell áthaladnia az üzenetsoron és a feldolgozón a tároló felé.</span><span class="sxs-lookup"><span data-stu-id="a7a30-164">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="a7a30-165">A webes előtér közvetlenül hajthat végre egyszerű olvasási/írási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="a7a30-165">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="a7a30-166">A feldolgozók erőforrás-igényes feladatokhoz vagy hosszan futó munkafolyamatokhoz lettek kialakítva.</span><span class="sxs-lookup"><span data-stu-id="a7a30-166">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="a7a30-167">Bizonyos esetekben előfordulhat, hogy nincs is szükség feldolgozóra.</span><span class="sxs-lookup"><span data-stu-id="a7a30-167">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="a7a30-168">Az App Service beépített automatikus skálázási szolgáltatásával horizontálisan felskálázhatja a virtuálisgép-példányok számát.</span><span class="sxs-lookup"><span data-stu-id="a7a30-168">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="a7a30-169">Ha az alkalmazás terhelése előre megjósolható mintákat követ, használjon ütemezésalapú automatikus skálázást.</span><span class="sxs-lookup"><span data-stu-id="a7a30-169">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="a7a30-170">Ha a terhelés nem jósolható meg előre, használjon mérőszám-alapú automatikus méretezési szabályokat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-170">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>

- <span data-ttu-id="a7a30-171">Érdemes lehet a webalkalmazást és a WebJob-feladatot külön App Service-csomagba helyezni.</span><span class="sxs-lookup"><span data-stu-id="a7a30-171">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="a7a30-172">Ily módon üzemeltethetők külön virtuálisgép-példányokon, és skálázhatók egymástól függetlenül.</span><span class="sxs-lookup"><span data-stu-id="a7a30-172">That way, they are hosted on separate VM instances and can be scaled independently.</span></span>

- <span data-ttu-id="a7a30-173">Az éles üzemhez és a teszteléshez használjon külön App Service-csomagot.</span><span class="sxs-lookup"><span data-stu-id="a7a30-173">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="a7a30-174">Ha ugyanis ugyanazt a csomagot használja az éles üzemhez és a teszteléshez, akkor a tesztek élesben működő virtuális gépeken futnak.</span><span class="sxs-lookup"><span data-stu-id="a7a30-174">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="a7a30-175">Az üzembe helyezések kezeléséhez használjon üzembehelyezési pontokat.</span><span class="sxs-lookup"><span data-stu-id="a7a30-175">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="a7a30-176">Ez lehetővé teszi, hogy telepítsen egy frissített verziót egy átmeneti helyen, majd átváltson az új verzióra.</span><span class="sxs-lookup"><span data-stu-id="a7a30-176">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="a7a30-177">Így arra is lehetősége van, hogy visszaváltson a korábbi verzióra, ha probléma adódna a frissítéssel.</span><span class="sxs-lookup"><span data-stu-id="a7a30-177">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md