---
title: "Válaszfal minta"
description: "Elemek készletekbe vonni az alkalmazások elkülönítése, hogy ha nem sikerül, a többi tovább működik"
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: a2c499d77fafc4bee6b74ee0e0d84e6c23b47851
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="bulkhead-pattern"></a><span data-ttu-id="02af4-103">Válaszfal minta</span><span class="sxs-lookup"><span data-stu-id="02af4-103">Bulkhead pattern</span></span>

<span data-ttu-id="02af4-104">Különítse el a készletekbe kérelem elemei, hogy ha nem sikerül, a többi tovább működik.</span><span class="sxs-lookup"><span data-stu-id="02af4-104">Isolate elements of an application into pools so that if one fails, the others will continue to function.</span></span>

<span data-ttu-id="02af4-105">Ebben a mintában nevű *válaszfal* , mert azt egy szállítási rendelkező üzenetpanelt partíciója hasonlít.</span><span class="sxs-lookup"><span data-stu-id="02af4-105">This pattern is named *Bulkhead* because it resembles the sectioned partitions of a ship's hull.</span></span> <span data-ttu-id="02af4-106">Ha az egy szállítási rendelkező biztonsága sérül, csak a sérült szakasz beírja vízjel, amely megakadályozza a szállítási elhelyezés.</span><span class="sxs-lookup"><span data-stu-id="02af4-106">If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.</span></span> 

## <a name="context-and-problem"></a><span data-ttu-id="02af4-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="02af4-107">Context and problem</span></span>

<span data-ttu-id="02af4-108">A felhő alapú alkalmazás tartalmazhat több szolgáltatás minden egyes szolgáltatással, hogy egy vagy több fogyasztó számára.</span><span class="sxs-lookup"><span data-stu-id="02af4-108">A cloud-based application may include multiple services, with each service having one or more consumers.</span></span> <span data-ttu-id="02af4-109">Túlterhelés vagy hiba történt a szolgáltatás hatással van-e a szolgáltatás összes ügyfelet.</span><span class="sxs-lookup"><span data-stu-id="02af4-109">Excessive load or failure in a service will impact all consumers of the service.</span></span>

<span data-ttu-id="02af4-110">Ezenkívül egy végfelhasználói előfordulhat, hogy kérelmet küldeni több szolgáltatás egyidejűleg, erőforrások használatával az egyes kérelmek.</span><span class="sxs-lookup"><span data-stu-id="02af4-110">Moreover, a consumer may send requests to multiple services simultaneously, using resources for each request.</span></span> <span data-ttu-id="02af4-111">Az ügyfél elküldi a kérelmet egy szolgáltatás, amely helytelenül van konfigurálva, vagy nem válaszol, amikor az ügyfél kérelem által használt erőforrások előfordulhat, hogy nem szabadul fel időben.</span><span class="sxs-lookup"><span data-stu-id="02af4-111">When the consumer sends a request to a service that is misconfigured or not responding, the resources used by the client's request may not be freed in a timely manner.</span></span> <span data-ttu-id="02af4-112">A szolgáltatás tovább kérelmeket, mert ezeket az erőforrásokat is kimerül.</span><span class="sxs-lookup"><span data-stu-id="02af4-112">As requests to the service continue, those resources may be exhausted.</span></span> <span data-ttu-id="02af4-113">Például az ügyfél kapcsolatot is lehet kimerült.</span><span class="sxs-lookup"><span data-stu-id="02af4-113">For example, the client's connection pool may be exhausted.</span></span> <span data-ttu-id="02af4-114">Ezen a ponton kérelmek más szolgáltatások a fogyasztó által érintett.</span><span class="sxs-lookup"><span data-stu-id="02af4-114">At that point, requests by the consumer to other services are impacted.</span></span> <span data-ttu-id="02af4-115">Végül a fogyasztó nem kéréseket küldhetnek más szolgáltatásokkal, nem csak az eredeti nem válaszoló szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="02af4-115">Eventually the consumer can no longer send requests to other services, not just the original unresponsive service.</span></span>

<span data-ttu-id="02af4-116">Az erőforrás-fogyási ugyanazon probléma azokat a szolgáltatásokat, több felhasználóból.</span><span class="sxs-lookup"><span data-stu-id="02af4-116">The same issue of resource exhaustion affects services with multiple consumers.</span></span> <span data-ttu-id="02af4-117">Egyik ügyféltől származó kérelmek nagy számú is lefoglalhat a szolgáltatásban elérhető erőforrások.</span><span class="sxs-lookup"><span data-stu-id="02af4-117">A large number of requests originating from one client may exhaust available resources in the service.</span></span> <span data-ttu-id="02af4-118">Más felhasználói nem tudnak a szolgáltatás felhasználásához effektus a kaszkádolt hibája okozza.</span><span class="sxs-lookup"><span data-stu-id="02af4-118">Other consumers are no longer able to consume the service, causing a cascading failure effect.</span></span>

## <a name="solution"></a><span data-ttu-id="02af4-119">Megoldás</span><span class="sxs-lookup"><span data-stu-id="02af4-119">Solution</span></span>

<span data-ttu-id="02af4-120">A szolgáltatáspéldány partíció különböző csoportok fogyasztói terhelés és a rendelkezésre állási követelmények alapján.</span><span class="sxs-lookup"><span data-stu-id="02af4-120">Partition service instances into different groups, based on consumer load and availability requirements.</span></span> <span data-ttu-id="02af4-121">Ez a kialakítás segít elkülöníteni a hibákat, és lehetővé teszi szolgáltatási szolgáltatások fenntartása az egyes fogyasztók hiba esetén.</span><span class="sxs-lookup"><span data-stu-id="02af4-121">This design helps to isolate failures, and allows you to sustain service functionality for some consumers, even during a failure.</span></span>

<span data-ttu-id="02af4-122">A fogyasztó is képes partícióazonosító erőforrásokat, győződjön meg arról, hogy egy szolgáltatás meghívásához használt erőforrásokat egy másik szolgáltatás hívásához használt erőforrások nem befolyásolják.</span><span class="sxs-lookup"><span data-stu-id="02af4-122">A consumer can also partition resources, to ensure that resources used to call one service don't affect the resources used to call another service.</span></span> <span data-ttu-id="02af4-123">Például, amely behívja több szolgáltatás ügyféllel rendelt minden egyes szolgáltatás kapcsolatkészlet.</span><span class="sxs-lookup"><span data-stu-id="02af4-123">For example, a consumer that calls multiple services may be assigned a connection pool for each service.</span></span> <span data-ttu-id="02af4-124">Ha egy szolgáltatás megkezdi az sikertelen lesz, csak az adott szolgáltatáshoz, lehetővé téve az ügyfélnek továbbra is használja a többi szolgáltatáshoz hozzárendelt kapcsolatkészlet hatással van.</span><span class="sxs-lookup"><span data-stu-id="02af4-124">If a service begins to fail, it only affects the connection pool assigned for that service, allowing the consumer to continue using the other services.</span></span>

<span data-ttu-id="02af4-125">Ez a minta a következő előnyöket nyújtja:</span><span class="sxs-lookup"><span data-stu-id="02af4-125">The benefits of this pattern include:</span></span>

- <span data-ttu-id="02af4-126">Elkülöníti a fogyasztók és szolgáltatások az egymásra épülő hibák.</span><span class="sxs-lookup"><span data-stu-id="02af4-126">Isolates consumers and services from cascading failures.</span></span> <span data-ttu-id="02af4-127">Lehet, hogy egy felhasználó vagy a szolgáltatást érintő probléma elkülönített belül a saját válaszfal, akadályozza meg, hogy a teljes megoldás is megfelelően működjenek.</span><span class="sxs-lookup"><span data-stu-id="02af4-127">An issue affecting a consumer or service can be isolated within its own bulkhead, preventing the entire solution from failing.</span></span>
- <span data-ttu-id="02af4-128">Lehetővé teszi bizonyos funkciók szolgáltatás meghibásodása megőrzése érdekében.</span><span class="sxs-lookup"><span data-stu-id="02af4-128">Allows you to preserve some functionality in the event of a service failure.</span></span> <span data-ttu-id="02af4-129">Szolgáltatások és funkciók az alkalmazás továbbra is működnek majd.</span><span class="sxs-lookup"><span data-stu-id="02af4-129">Other services and features of the application will continue to work.</span></span>
- <span data-ttu-id="02af4-130">Szolgáltatások telepítését, amelyek a felhasználó alkalmazásokban különböző szolgáltatásminőség lehetővé teszi lehetővé.</span><span class="sxs-lookup"><span data-stu-id="02af4-130">Allows you to deploy services that offer a different quality of service for consuming applications.</span></span> <span data-ttu-id="02af4-131">Egy magas prioritású fogyasztói készlet beállítható úgy, hogy fontos szolgáltatások használatára.</span><span class="sxs-lookup"><span data-stu-id="02af4-131">A high-priority consumer pool can be configured to use high-priority services.</span></span> 

<span data-ttu-id="02af4-132">Az alábbi ábrán látható kapcsolatkészletek hívó egyes szolgáltatások köré strukturált válaszfalak.</span><span class="sxs-lookup"><span data-stu-id="02af4-132">The following diagram shows bulkheads structured around connection pools that call individual services.</span></span> <span data-ttu-id="02af4-133">Ha a szolgáltatás A nem sikerül, vagy valamilyen egyéb miatt nem probléma, a kapcsolatot a kapcsolatkészletből, elkülönített, ezért a szolgáltatáshoz rendelt szálkészlet csak munkaterhelés egy befolyásolja.</span><span class="sxs-lookup"><span data-stu-id="02af4-133">If Service A fails or causes some other issue, the connection pool is isolated, so only workloads using the thread pool assigned to Service A are affected.</span></span> <span data-ttu-id="02af4-134">Szolgáltatás B és C használó munkaterhelések nem érinti, és folytathatja megszakítás nélkül.</span><span class="sxs-lookup"><span data-stu-id="02af4-134">Workloads that use Service B and C are not affected and can continue working without interruption.</span></span>

![](./_images/bulkhead-1.png) 

<span data-ttu-id="02af4-135">A következő ábrán egy egyetlen szolgáltatást több ügyfél.</span><span class="sxs-lookup"><span data-stu-id="02af4-135">The next diagram shows multiple clients calling a single service.</span></span> <span data-ttu-id="02af4-136">Minden egyes ügyfélnek külön szolgáltatáspéldány van hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="02af4-136">Each client is assigned a separate service instance.</span></span> <span data-ttu-id="02af4-137">Ügyfél 1 túl sok kérelmet és a példány túlterhelik.</span><span class="sxs-lookup"><span data-stu-id="02af4-137">Client 1 has made too many requests and overwhelmed its instance.</span></span> <span data-ttu-id="02af4-138">Minden szolgáltatáspéldány el különítve a többi, mert az ügyfelek továbbra is hívások.</span><span class="sxs-lookup"><span data-stu-id="02af4-138">Because each service instance is isolated from the others, the other clients can continue making calls.</span></span>

![](./_images/bulkhead-2.png)
     
## <a name="issues-and-considerations"></a><span data-ttu-id="02af4-139">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="02af4-139">Issues and considerations</span></span>

- <span data-ttu-id="02af4-140">Definiáljon partíciókat a üzleti és műszaki követelményeit, az alkalmazás körül.</span><span class="sxs-lookup"><span data-stu-id="02af4-140">Define partitions around the business and technical requirements of the application.</span></span>
- <span data-ttu-id="02af4-141">Particionálás és fogyasztók válaszfalak be, vegye figyelembe a technológiai, valamint a terhelés költségének, teljesítményének és kezelhetőség szempontjából által kínált elkülönítési szintjét.</span><span class="sxs-lookup"><span data-stu-id="02af4-141">When partitioning services or consumers into bulkheads, consider the level of isolation offered by the technology as well as the overhead in terms of cost, performance and manageability.</span></span>
- <span data-ttu-id="02af4-142">Vegye figyelembe a válaszfalak kombinálásával ismételt próbálkozás után, az áramköri megszakító, és a szabályozás minták kifinomultabb tartalék kezelési biztosításához.</span><span class="sxs-lookup"><span data-stu-id="02af4-142">Consider combining bulkheads with retry, circuit breaker, and throttling patterns to provide more sophisticated fault handling.</span></span>
- <span data-ttu-id="02af4-143">Válaszfalak a fogyasztók particionálásakor érdemes folyamatok, a szál készletek és a szemaforok.</span><span class="sxs-lookup"><span data-stu-id="02af4-143">When partitioning consumers into bulkheads, consider using processes, thread pools, and semaphores.</span></span> <span data-ttu-id="02af4-144">Például a projektek [Netflix Hystrix] [ hystrix] és [Polly] [ polly] keretrendszere, amely fogyasztói válaszfalak létrehozása kínálnak.</span><span class="sxs-lookup"><span data-stu-id="02af4-144">Projects like [Netflix Hystrix][hystrix] and [Polly][polly] offer a framework for creating consumer bulkheads.</span></span>
- <span data-ttu-id="02af4-145">Szolgáltatások a válaszfalak particionálásakor fontolja meg olyan önálló virtuális gépek, a tárolók és a folyamatok üzembe őket.</span><span class="sxs-lookup"><span data-stu-id="02af4-145">When partitioning services into bulkheads, consider deploying them into separate virtual machines, containers, or processes.</span></span> <span data-ttu-id="02af4-146">Tárolók rövid egyensúlyt erőforrás-elkülönítést, viszonylag alacsony terhelés mellett.</span><span class="sxs-lookup"><span data-stu-id="02af4-146">Containers offer a good balance of resource isolation with fairly low overhead.</span></span>
- <span data-ttu-id="02af4-147">Aszinkron üzenetek használatával kommunikáló szolgáltatások el lehet különíteni a várólisták más-más részhalmazához.</span><span class="sxs-lookup"><span data-stu-id="02af4-147">Services that communicate using asynchronous messages can be isolated through different sets of queues.</span></span> <span data-ttu-id="02af4-148">Minden várólista rendelkezhet egy dedikált példánya a várólista, vagy egy különálló csoportot olyan algoritmussal created és feldolgozási mennyi példányok üzenetek feldolgozása.</span><span class="sxs-lookup"><span data-stu-id="02af4-148">Each queue can have a dedicated set of instances processing messages on the queue, or a single group of instances using an algorithm to dequeue and dispatch processing.</span></span>
- <span data-ttu-id="02af4-149">Meghatározza a válaszfalak részletességgel szintjét.</span><span class="sxs-lookup"><span data-stu-id="02af4-149">Determine the level of granularity for the bulkheads.</span></span> <span data-ttu-id="02af4-150">Például ha azt szeretné, partíciók bérlők szét, akkor sikerült helyezze mindegyik bérlő külön partíción, put több bérlő több partícióra.</span><span class="sxs-lookup"><span data-stu-id="02af4-150">For example, if you want to distribute tenants across partitions, you could place each tenant into a separate partition, a put several tenants into one partition.</span></span>
- <span data-ttu-id="02af4-151">Mindegyik partíció teljesítmény- és SLA figyelése.</span><span class="sxs-lookup"><span data-stu-id="02af4-151">Monitor each partition’s performance and SLA.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="02af4-152">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="02af4-152">When to use this pattern</span></span>

<span data-ttu-id="02af4-153">A minta használata:</span><span class="sxs-lookup"><span data-stu-id="02af4-153">Use this pattern to:</span></span>

- <span data-ttu-id="02af4-154">Különítse el a használt háttér szolgáltatások felhasználásához, különösen akkor, ha az alkalmazás biztosíthat bizonyos fokú funkciót, akkor is, ha egy szolgáltatás nem válaszol.</span><span class="sxs-lookup"><span data-stu-id="02af4-154">Isolate resources used to consume a set of backend services, especially if the application can provide some level of functionality even when one of the services is not responding.</span></span>
- <span data-ttu-id="02af4-155">Különítse el a fogyasztói kritikus fogyasztók.</span><span class="sxs-lookup"><span data-stu-id="02af4-155">Isolate critical consumers from standard consumers.</span></span>
- <span data-ttu-id="02af4-156">Az alkalmazás a kaszkádolt meghibásodásával szembeni védelemhez.</span><span class="sxs-lookup"><span data-stu-id="02af4-156">Protect the application from cascading failures.</span></span>

<span data-ttu-id="02af4-157">Ez a minta nem alkalmasak lehetnek esetén:</span><span class="sxs-lookup"><span data-stu-id="02af4-157">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="02af4-158">Erőforrások kevésbé hatékony használatát nem lehet a projekt elfogadható.</span><span class="sxs-lookup"><span data-stu-id="02af4-158">Less efficient use of resources may not be acceptable in the project.</span></span>
- <span data-ttu-id="02af4-159">Nincs szükség a hozzáadott összetettsége</span><span class="sxs-lookup"><span data-stu-id="02af4-159">The added complexity is not necessary</span></span>

## <a name="example"></a><span data-ttu-id="02af4-160">Példa</span><span class="sxs-lookup"><span data-stu-id="02af4-160">Example</span></span>

<span data-ttu-id="02af4-161">A következő Kubernetes konfigurációs fájl létrehoz egy elszigetelt tárolót, hogy egyetlen szolgáltatás, futtassa a saját CPU és memória-erőforrásokat és a határértékeket.</span><span class="sxs-lookup"><span data-stu-id="02af4-161">The following Kubernetes configuration file creates an isolated container to run a single service, with its own CPU and memory resources and limits.</span></span>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: drone-management
spec:
  containers:
  - name: drone-management-container
    image: drone-service
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
```

## <a name="related-guidance"></a><span data-ttu-id="02af4-162">Kapcsolódó útmutató</span><span class="sxs-lookup"><span data-stu-id="02af4-162">Related guidance</span></span>

- [<span data-ttu-id="02af4-163">Áramköri megszakító minta</span><span class="sxs-lookup"><span data-stu-id="02af4-163">Circuit Breaker pattern</span></span>](./circuit-breaker.md)
- [<span data-ttu-id="02af4-164">Rugalmas alkalmazások tervezése az Azure számára</span><span class="sxs-lookup"><span data-stu-id="02af4-164">Designing resilient applications for Azure</span></span>](../resiliency/index.md)
- [<span data-ttu-id="02af4-165">Ismételje meg a minta</span><span class="sxs-lookup"><span data-stu-id="02af4-165">Retry pattern</span></span>](./retry.md)
- [<span data-ttu-id="02af4-166">Sávszélesség-szabályozási minta</span><span class="sxs-lookup"><span data-stu-id="02af4-166">Throttling pattern</span></span>](./throttling.md)


<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly