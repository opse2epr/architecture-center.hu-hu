---
title: Webüzenetsor-feldolgozó architektúrastílus
titleSuffix: Azure Application Architecture Guide
description: Előnyeit, kihívásait és ajánlott eljárások a Webüzenetsor-feldolgozó architektúráinak ismerteti az Azure-ban.
author: MikeWasson
ms.date: 04/10/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 974b8b8595d6d9333552c41dfe1f3f2af848d264
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740391"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="2448f-103">Webüzenetsor-feldolgozó architektúrastílus</span><span class="sxs-lookup"><span data-stu-id="2448f-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="2448f-104">Az ilyen architektúrák alapelemei: egy **webes előtér**, amely kiszolgálja az ügyfélkérelmeket, és egy **feldolgozó**, amely erőforrás-igényes munkákat, hosszan futó munkafolyamatokat vagy kötegelt feladatokat hajt végre.</span><span class="sxs-lookup"><span data-stu-id="2448f-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="2448f-105">A webes előtér egy **üzenetsor** segítségével kommunikál a feldolgozóval.</span><span class="sxs-lookup"><span data-stu-id="2448f-105">The web front end communicates with the worker through a **message queue**.</span></span>

![Webüzenetsor-feldolgozó architektúrastílus logikai diagramja](./images/web-queue-worker-logical.svg)

<span data-ttu-id="2448f-107">Az ebbe az architektúrába gyakran beillesztett más összetevők:</span><span class="sxs-lookup"><span data-stu-id="2448f-107">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="2448f-108">Egy vagy több adatbázis.</span><span class="sxs-lookup"><span data-stu-id="2448f-108">One or more databases.</span></span>
- <span data-ttu-id="2448f-109">Egy gyorsítótár az adatbázis adatainak tárolására a gyors olvasás érdekében.</span><span class="sxs-lookup"><span data-stu-id="2448f-109">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="2448f-110">CDN a statikus tartalom kiszolgálásához</span><span class="sxs-lookup"><span data-stu-id="2448f-110">CDN to serve static content</span></span>
- <span data-ttu-id="2448f-111">Távoli szolgáltatások, például e-mail- vagy SMS-szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="2448f-111">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="2448f-112">Ezeket gyakran egy külső fél biztosítja.</span><span class="sxs-lookup"><span data-stu-id="2448f-112">Often these are provided by third parties.</span></span>
- <span data-ttu-id="2448f-113">Identitásszolgáltató a hitelesítéshez.</span><span class="sxs-lookup"><span data-stu-id="2448f-113">Identity provider for authentication.</span></span>

<span data-ttu-id="2448f-114">A web és a feldolgozó is állapot nélküli.</span><span class="sxs-lookup"><span data-stu-id="2448f-114">The web and worker are both stateless.</span></span> <span data-ttu-id="2448f-115">A munkamenet-állapot tárolható egy megosztott gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="2448f-115">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="2448f-116">A hosszan futó munkákat a feldolgozó aszinkron módon végzi el.</span><span class="sxs-lookup"><span data-stu-id="2448f-116">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="2448f-117">A feldolgozó aktiválható üzenetekkel az üzenetsoron, vagy futhat egy ütemezésben a kötegelt feldolgozáshoz.</span><span class="sxs-lookup"><span data-stu-id="2448f-117">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="2448f-118">A feldolgozó egy választható összetevő.</span><span class="sxs-lookup"><span data-stu-id="2448f-118">The worker is an optional component.</span></span> <span data-ttu-id="2448f-119">Ha nincs hosszan futó művelet, a feldolgozó elhagyható.</span><span class="sxs-lookup"><span data-stu-id="2448f-119">If there are no long-running operations, the worker can be omitted.</span></span>

<span data-ttu-id="2448f-120">Az előtér állhat egy webes API-ból.</span><span class="sxs-lookup"><span data-stu-id="2448f-120">The front end might consist of a web API.</span></span> <span data-ttu-id="2448f-121">Ügyféloldalon a webes API-t használhatja egy AJAX-hívásokat indító egyoldalas alkalmazás vagy egy natív ügyfélalkalmazás.</span><span class="sxs-lookup"><span data-stu-id="2448f-121">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="2448f-122">Mikor érdemes ezt az architektúrát használni?</span><span class="sxs-lookup"><span data-stu-id="2448f-122">When to use this architecture</span></span>

<span data-ttu-id="2448f-123">A webüzenetsor-feldolgozó architektúra implementálása általában felügyelt számítási szolgáltatások használatával történik, vagy az Azure App Service vagy az Azure Cloud Services segítségével.</span><span class="sxs-lookup"><span data-stu-id="2448f-123">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span>

<span data-ttu-id="2448f-124">Fontolja meg ennek az architektúrastílusnak a használatát a következőkhöz:</span><span class="sxs-lookup"><span data-stu-id="2448f-124">Consider this architecture style for:</span></span>

- <span data-ttu-id="2448f-125">Alkalmazások viszonylag egyszerű tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="2448f-125">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="2448f-126">Alkalmazások néhány hosszan futó munkafolyamattal vagy kötegelt műveletekkel.</span><span class="sxs-lookup"><span data-stu-id="2448f-126">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="2448f-127">Ha felügyelt szolgáltatásokat szeretne használni szolgáltatott infrastruktúra (IaaS) helyett.</span><span class="sxs-lookup"><span data-stu-id="2448f-127">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="2448f-128">Előnyök</span><span class="sxs-lookup"><span data-stu-id="2448f-128">Benefits</span></span>

- <span data-ttu-id="2448f-129">Viszonylag egyszerű és könnyen érthető architektúra.</span><span class="sxs-lookup"><span data-stu-id="2448f-129">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="2448f-130">Könnyen telepíthető és felügyelhető.</span><span class="sxs-lookup"><span data-stu-id="2448f-130">Easy to deploy and manage.</span></span>
- <span data-ttu-id="2448f-131">A kockázatok egyértelmű elkülönítése.</span><span class="sxs-lookup"><span data-stu-id="2448f-131">Clear separation of concerns.</span></span>
- <span data-ttu-id="2448f-132">Az előteret a rendszer aszinkron üzenetküldés segítségével választja le a feldolgozóról.</span><span class="sxs-lookup"><span data-stu-id="2448f-132">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="2448f-133">Az előtér és a feldolgozó egymástól függetlenül skálázható.</span><span class="sxs-lookup"><span data-stu-id="2448f-133">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="2448f-134">Problémák</span><span class="sxs-lookup"><span data-stu-id="2448f-134">Challenges</span></span>

- <span data-ttu-id="2448f-135">Gondos tervezés nélkül az előtér és a feldolgozó hatalmas, monolitikus, végül nehezen karbantartható és frissíthető összetevővé válhat.</span><span class="sxs-lookup"><span data-stu-id="2448f-135">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="2448f-136">Lehetnek rejtett függőségek, ha az előtér és a feldolgozó közös sémákat vagy kódmodulokat használ.</span><span class="sxs-lookup"><span data-stu-id="2448f-136">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span>

## <a name="best-practices"></a><span data-ttu-id="2448f-137">Ajánlott eljárások</span><span class="sxs-lookup"><span data-stu-id="2448f-137">Best practices</span></span>

- <span data-ttu-id="2448f-138">Jól megtervezett API közzététele az ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="2448f-138">Expose a well-designed API to the client.</span></span> <span data-ttu-id="2448f-139">Tekintse meg az [API-tervezés – Ajánlott eljárások][api-design] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="2448f-139">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="2448f-140">Automatikus skálázás a terhelés változásának kezelésére.</span><span class="sxs-lookup"><span data-stu-id="2448f-140">Autoscale to handle changes in load.</span></span> <span data-ttu-id="2448f-141">Lásd az [automatikus skálázással kapcsolatos ajánlott eljárásokat][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="2448f-141">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="2448f-142">Gyorsítótárazza a félig statikus adatokat.</span><span class="sxs-lookup"><span data-stu-id="2448f-142">Cache semi-static data.</span></span> <span data-ttu-id="2448f-143">Lásd a [gyorsítótárazás ajánlott eljárásait][caching].</span><span class="sxs-lookup"><span data-stu-id="2448f-143">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="2448f-144">CDN használata a statikus tartalom üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="2448f-144">Use a CDN to host static content.</span></span> <span data-ttu-id="2448f-145">Tekintse meg az [Ajánlott tanácsok az Azure CDN-nel kapcsolatban][cdn] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="2448f-145">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="2448f-146">Többnyelvű adatmegőrzés használata, ha szükséges.</span><span class="sxs-lookup"><span data-stu-id="2448f-146">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="2448f-147">Tekintse meg a [Használja a feladathoz legmegfelelőbb adattárat][polyglot] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="2448f-147">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="2448f-148">Adatok particionálása a méretezhetőség növeléséhez, a versengés kiküszöböléséhez és a teljesítmény optimalizálásához.</span><span class="sxs-lookup"><span data-stu-id="2448f-148">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="2448f-149">Tekintse meg az [Ajánlott adatparticionálási eljárások][data-partition] szakaszt.</span><span class="sxs-lookup"><span data-stu-id="2448f-149">See [Data partitioning best practices][data-partition].</span></span>

## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="2448f-150">A webüzenetsor-feldolgozó az Azure App Service-en</span><span class="sxs-lookup"><span data-stu-id="2448f-150">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="2448f-151">Ez a szakasz ismertet egy ajánlott, az Azure App Service-t használó webüzenetsor-feldolgozót.</span><span class="sxs-lookup"><span data-stu-id="2448f-151">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span>

![Webüzenetsor-feldolgozó architektúrastílus fizikai diagramja](./images/web-queue-worker-physical.png)

- <span data-ttu-id="2448f-153">Az előtér egy Azure App Service webalkalmazás van megvalósítva, és a feldolgozó megvalósítása az Azure Functions-alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="2448f-153">The front end is implemented as an Azure App Service web app, and the worker is implemented as an Azure Functions app.</span></span> <span data-ttu-id="2448f-154">A web app és a függvényalkalmazáshoz tartozó App Service-csomag, amely a Virtuálisgép-példányok biztosít.</span><span class="sxs-lookup"><span data-stu-id="2448f-154">The web app and the function app are both associated with an App Service plan that provides the VM instances.</span></span>

- <span data-ttu-id="2448f-155">Az üzenetsorhoz használhat Azure Service Bus- vagy Azure Storage-üzenetsort is.</span><span class="sxs-lookup"><span data-stu-id="2448f-155">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="2448f-156">(Az ábrán egy Azure Storage-üzenetsor látható.)</span><span class="sxs-lookup"><span data-stu-id="2448f-156">(The diagram shows an Azure Storage queue.)</span></span>

- <span data-ttu-id="2448f-157">Az Azure Redis Cache tárolja a munkamenet állapotát, és más, alacsony késésű elérést igénylő adatokat.</span><span class="sxs-lookup"><span data-stu-id="2448f-157">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

- <span data-ttu-id="2448f-158">Az Azure CDN használatával gyorsítótárazható statikus tartalom, például képek, CSS vagy HTML.</span><span class="sxs-lookup"><span data-stu-id="2448f-158">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

- <span data-ttu-id="2448f-159">A tároláshoz válassza ki az alkalmazás igényeinek legmegfelelőbb tárolási technológiákat.</span><span class="sxs-lookup"><span data-stu-id="2448f-159">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="2448f-160">Használhat több tárolási technológiát (többnyelvű adatmegőrzés).</span><span class="sxs-lookup"><span data-stu-id="2448f-160">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="2448f-161">Az elképzelés ábrázolására az ábra bemutatja az Azure SQL Database-t és az Azure Cosmos DB-t.</span><span class="sxs-lookup"><span data-stu-id="2448f-161">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>

<span data-ttu-id="2448f-162">További részletek: [App Service-webalkalmazás referenciaarchitektúrája][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="2448f-162">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="2448f-163">Néhány fontos megjegyzés</span><span class="sxs-lookup"><span data-stu-id="2448f-163">Additional considerations</span></span>

- <span data-ttu-id="2448f-164">Nem minden tranzakciónak kell áthaladnia az üzenetsoron és a feldolgozón a tároló felé.</span><span class="sxs-lookup"><span data-stu-id="2448f-164">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="2448f-165">A webes előtér közvetlenül hajthat végre egyszerű olvasási/írási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="2448f-165">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="2448f-166">A feldolgozók erőforrás-igényes feladatokhoz vagy hosszan futó munkafolyamatokhoz lettek kialakítva.</span><span class="sxs-lookup"><span data-stu-id="2448f-166">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="2448f-167">Bizonyos esetekben előfordulhat, hogy nincs is szükség feldolgozóra.</span><span class="sxs-lookup"><span data-stu-id="2448f-167">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="2448f-168">Az App Service beépített automatikus skálázási szolgáltatásával horizontálisan felskálázhatja a virtuálisgép-példányok számát.</span><span class="sxs-lookup"><span data-stu-id="2448f-168">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="2448f-169">Ha az alkalmazás terhelése előre megjósolható mintákat követ, használjon ütemezésalapú automatikus skálázást.</span><span class="sxs-lookup"><span data-stu-id="2448f-169">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="2448f-170">Ha a terhelés nem jósolható meg előre, használjon mérőszám-alapú automatikus méretezési szabályokat.</span><span class="sxs-lookup"><span data-stu-id="2448f-170">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>

- <span data-ttu-id="2448f-171">Vegye figyelembe, hogy a webalkalmazás és a függvényalkalmazás üzembe külön App Service-csomagok.</span><span class="sxs-lookup"><span data-stu-id="2448f-171">Consider putting the web app and the function app into separate App Service plans.</span></span> <span data-ttu-id="2448f-172">Ezzel a módszerrel, egymástól függetlenül skálázhatók.</span><span class="sxs-lookup"><span data-stu-id="2448f-172">That way, they can be scaled independently.</span></span>

- <span data-ttu-id="2448f-173">Az éles üzemhez és a teszteléshez használjon külön App Service-csomagot.</span><span class="sxs-lookup"><span data-stu-id="2448f-173">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="2448f-174">Ha ugyanis ugyanazt a csomagot használja az éles üzemhez és a teszteléshez, akkor a tesztek élesben működő virtuális gépeken futnak.</span><span class="sxs-lookup"><span data-stu-id="2448f-174">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="2448f-175">Az üzembe helyezések kezeléséhez használjon üzembehelyezési pontokat.</span><span class="sxs-lookup"><span data-stu-id="2448f-175">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="2448f-176">Ez lehetővé teszi, hogy telepítsen egy frissített verziót egy átmeneti helyen, majd átváltson az új verzióra.</span><span class="sxs-lookup"><span data-stu-id="2448f-176">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="2448f-177">Így arra is lehetősége van, hogy visszaváltson a korábbi verzióra, ha probléma adódna a frissítéssel.</span><span class="sxs-lookup"><span data-stu-id="2448f-177">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md