---
title: Válaszfal minta
titleSuffix: Cloud Design Patterns
description: Készletekbe választja szét egy alkalmazás elemeit, hogy ha az egyik meghibásodna, a többi tovább üzemeljen.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 0a2ae4789d3c1653405a59ef8cb4f6171a8abc81
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112192"
---
# <a name="bulkhead-pattern"></a><span data-ttu-id="54c11-104">Válaszfal minta</span><span class="sxs-lookup"><span data-stu-id="54c11-104">Bulkhead pattern</span></span>

<span data-ttu-id="54c11-105">Készletekbe választja szét egy alkalmazás elemeit, hogy ha az egyik meghibásodna, a többi tovább üzemeljen.</span><span class="sxs-lookup"><span data-stu-id="54c11-105">Isolate elements of an application into pools so that if one fails, the others will continue to function.</span></span>

<span data-ttu-id="54c11-106">Ezt a mintát *Válaszfal* mintának nevezik, mert egy hajótest felosztott részeire emlékeztet.</span><span class="sxs-lookup"><span data-stu-id="54c11-106">This pattern is named *Bulkhead* because it resembles the sectioned partitions of a ship's hull.</span></span> <span data-ttu-id="54c11-107">Ha a hajótest megsérül, csak a sérült szakasz telik meg vízzel, így a hajó nem süllyed el.</span><span class="sxs-lookup"><span data-stu-id="54c11-107">If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="54c11-108">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="54c11-108">Context and problem</span></span>

<span data-ttu-id="54c11-109">Egy felhőalapú alkalmazás több szolgáltatást is tartalmazhat, és minden szolgáltatás egy vagy több fogyasztóval is rendelkezhet.</span><span class="sxs-lookup"><span data-stu-id="54c11-109">A cloud-based application may include multiple services, with each service having one or more consumers.</span></span> <span data-ttu-id="54c11-110">Ha egy szolgáltatás túl van terhelve vagy meghibásodik, az minden fogyasztót érint.</span><span class="sxs-lookup"><span data-stu-id="54c11-110">Excessive load or failure in a service will impact all consumers of the service.</span></span>

<span data-ttu-id="54c11-111">Továbbá egy fogyasztó egyszerre több szolgáltatáshoz is küldhet kéréseket, és minden kéréshez erőforrásokat használ fel.</span><span class="sxs-lookup"><span data-stu-id="54c11-111">Moreover, a consumer may send requests to multiple services simultaneously, using resources for each request.</span></span> <span data-ttu-id="54c11-112">Ha egy fogyasztó kérést küld egy rosszul konfigurált vagy nem válaszoló szolgáltatásnak, előfordulhat, hogy az ügyfél kérése által felhasznált erőforrások hogy nem szabadulnak fel megfelelő időn belül.</span><span class="sxs-lookup"><span data-stu-id="54c11-112">When the consumer sends a request to a service that is misconfigured or not responding, the resources used by the client's request may not be freed in a timely manner.</span></span> <span data-ttu-id="54c11-113">Ahogy újabb kérések érkeznek a szolgáltatáshoz, ezek az erőforrások kimerülhetnek.</span><span class="sxs-lookup"><span data-stu-id="54c11-113">As requests to the service continue, those resources may be exhausted.</span></span> <span data-ttu-id="54c11-114">Kimerülhet például az ügyfél kapcsolatkészlete.</span><span class="sxs-lookup"><span data-stu-id="54c11-114">For example, the client's connection pool may be exhausted.</span></span> <span data-ttu-id="54c11-115">Ez már a fogyasztó más szolgáltatásokhoz küldött kéréseire is kihat.</span><span class="sxs-lookup"><span data-stu-id="54c11-115">At that point, requests by the consumer to other services are impacted.</span></span> <span data-ttu-id="54c11-116">Végül a fogyasztó már más szolgáltatásoknak sem fog tudni kéréseket küldeni, nem csak az eredeti nem válaszoló szolgáltatásnak.</span><span class="sxs-lookup"><span data-stu-id="54c11-116">Eventually the consumer can no longer send requests to other services, not just the original unresponsive service.</span></span>

<span data-ttu-id="54c11-117">Ugyanez az erőforrásfogyás a több fogyasztóval rendelkező szolgáltatásokat is érinti.</span><span class="sxs-lookup"><span data-stu-id="54c11-117">The same issue of resource exhaustion affects services with multiple consumers.</span></span> <span data-ttu-id="54c11-118">Az egy ügyféltől nagy számban érkező kérések kimeríthetik a szolgáltatás elérhető erőforrásait.</span><span class="sxs-lookup"><span data-stu-id="54c11-118">A large number of requests originating from one client may exhaust available resources in the service.</span></span> <span data-ttu-id="54c11-119">Így a többi fogyasztó tudja igénybe venni a szolgáltatást, ez pedig lépcsőzetesen terjedő hibákhoz vezet.</span><span class="sxs-lookup"><span data-stu-id="54c11-119">Other consumers are no longer able to consume the service, causing a cascading failure effect.</span></span>

## <a name="solution"></a><span data-ttu-id="54c11-120">Megoldás</span><span class="sxs-lookup"><span data-stu-id="54c11-120">Solution</span></span>

<span data-ttu-id="54c11-121">Particionálja a szolgáltatáspéldányokat különböző csoportokba a fogyasztók mennyisége és a rendelkezésre állási követelmények alapján.</span><span class="sxs-lookup"><span data-stu-id="54c11-121">Partition service instances into different groups, based on consumer load and availability requirements.</span></span> <span data-ttu-id="54c11-122">Ez a tervezés segít a hibák elkülönítésében, és meghibásodás esetén is biztosítja, hogy a szolgáltatás bizonyos fogyasztók számára elérhető legyen.</span><span class="sxs-lookup"><span data-stu-id="54c11-122">This design helps to isolate failures, and allows you to sustain service functionality for some consumers, even during a failure.</span></span>

<span data-ttu-id="54c11-123">A fogyasztók szintén particionálhatják az erőforrásokat, így az egyik szolgáltatás hívásához használt erőforrások nem befolyásolják a másik szolgáltatás hívásához használtakat.</span><span class="sxs-lookup"><span data-stu-id="54c11-123">A consumer can also partition resources, to ensure that resources used to call one service don't affect the resources used to call another service.</span></span> <span data-ttu-id="54c11-124">Például egy több szolgáltatást is hívó fogyasztóhoz szolgáltatásonként külön kapcsolatkészlet rendelhető hozzá.</span><span class="sxs-lookup"><span data-stu-id="54c11-124">For example, a consumer that calls multiple services may be assigned a connection pool for each service.</span></span> <span data-ttu-id="54c11-125">Ha egy szolgáltatás kezd meghibásodni, ez csak az adott szolgáltatáshoz hozzárendelt kapcsolatkészletet érinti, így a fogyasztó tovább használhatja a többi szolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="54c11-125">If a service begins to fail, it only affects the connection pool assigned for that service, allowing the consumer to continue using the other services.</span></span>

<span data-ttu-id="54c11-126">A minta használata többek között a következő előnyökkel jár:</span><span class="sxs-lookup"><span data-stu-id="54c11-126">The benefits of this pattern include:</span></span>

- <span data-ttu-id="54c11-127">Elkülöníti a fogyasztókat és szolgáltatásokat a lépcsőzetesen terjedő hibáktól.</span><span class="sxs-lookup"><span data-stu-id="54c11-127">Isolates consumers and services from cascading failures.</span></span> <span data-ttu-id="54c11-128">A fogyasztókat vagy szolgáltatásokat érintő hibák elkülöníthetők a saját válaszfalukon belül, így megakadályozható a teljes megoldás meghibásodása.</span><span class="sxs-lookup"><span data-stu-id="54c11-128">An issue affecting a consumer or service can be isolated within its own bulkhead, preventing the entire solution from failing.</span></span>
- <span data-ttu-id="54c11-129">Lehetővé teszi a működésképesség bizonyos szintű fenntartását a szolgáltatás meghibásodása esetén is.</span><span class="sxs-lookup"><span data-stu-id="54c11-129">Allows you to preserve some functionality in the event of a service failure.</span></span> <span data-ttu-id="54c11-130">Az alkalmazás többi szolgáltatása és funkciója továbbra is működhet.</span><span class="sxs-lookup"><span data-stu-id="54c11-130">Other services and features of the application will continue to work.</span></span>
- <span data-ttu-id="54c11-131">Lehetővé teszi olyan szolgáltatások üzembe helyezését, amelyek eltérő minőségű szolgáltatást biztosítanak a felhasználó alkalmazások számára.</span><span class="sxs-lookup"><span data-stu-id="54c11-131">Allows you to deploy services that offer a different quality of service for consuming applications.</span></span> <span data-ttu-id="54c11-132">Beállítható egy magas prioritású fogyasztókészlet a magas prioritású szolgáltatások használatához.</span><span class="sxs-lookup"><span data-stu-id="54c11-132">A high-priority consumer pool can be configured to use high-priority services.</span></span>

<span data-ttu-id="54c11-133">Az alábbi ábra a különálló szolgáltatásokat hívó kapcsolatkészletek köré strukturált válaszfalakat ábrázol.</span><span class="sxs-lookup"><span data-stu-id="54c11-133">The following diagram shows bulkheads structured around connection pools that call individual services.</span></span> <span data-ttu-id="54c11-134">Ha az A szolgáltatás meghibásodik vagy egyéb hibát okoz, a kapcsolatkészlet elkülönítése miatt a hiba csak azokat a számítási feladatokat érinti, amelyek az A szolgáltatáshoz hozzárendelt szálkészletet használják.</span><span class="sxs-lookup"><span data-stu-id="54c11-134">If Service A fails or causes some other issue, the connection pool is isolated, so only workloads using the thread pool assigned to Service A are affected.</span></span> <span data-ttu-id="54c11-135">A B és C szolgáltatásokat használó számítási feladatokat ez nem befolyásolja, és megszakítás nélkül működhetnek tovább.</span><span class="sxs-lookup"><span data-stu-id="54c11-135">Workloads that use Service B and C are not affected and can continue working without interruption.</span></span>

![A válaszfal minta első ábrája](./_images/bulkhead-1.png)

<span data-ttu-id="54c11-137">A következő diagram több ügyfelet ábrázol, amelyek ugyanazt a szolgáltatást hívják.</span><span class="sxs-lookup"><span data-stu-id="54c11-137">The next diagram shows multiple clients calling a single service.</span></span> <span data-ttu-id="54c11-138">Minden ügyfélhez különálló szolgáltatáspéldány van hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="54c11-138">Each client is assigned a separate service instance.</span></span> <span data-ttu-id="54c11-139">Az 1. ügyfél túl sok kérést küldött, és túlterhelte a példányát.</span><span class="sxs-lookup"><span data-stu-id="54c11-139">Client 1 has made too many requests and overwhelmed its instance.</span></span> <span data-ttu-id="54c11-140">Mivel minden szolgáltatáspéldány külön van választva a többitől, a többi ügyfél továbbra is küldhet hívásokat.</span><span class="sxs-lookup"><span data-stu-id="54c11-140">Because each service instance is isolated from the others, the other clients can continue making calls.</span></span>

![A válaszfal minta első ábrája](./_images/bulkhead-2.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="54c11-142">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="54c11-142">Issues and considerations</span></span>

- <span data-ttu-id="54c11-143">Határozza meg a partíciókat az alkalmazás üzleti és műszaki követelményei alapján.</span><span class="sxs-lookup"><span data-stu-id="54c11-143">Define partitions around the business and technical requirements of the application.</span></span>
- <span data-ttu-id="54c11-144">Amikor válaszfalakkal particionálja a szolgáltatásokat vagy fogyasztókat, vegye figyelembe a technológia által nyújtott elkülönítési szinteket és a terhelést (költségek, teljesítmény és kezelhetőség) is.</span><span class="sxs-lookup"><span data-stu-id="54c11-144">When partitioning services or consumers into bulkheads, consider the level of isolation offered by the technology as well as the overhead in terms of cost, performance and manageability.</span></span>
- <span data-ttu-id="54c11-145">Vegye fontolóra a válaszfalak újrapróbálkozási, áramkör-megszakító és szabályozási mintákkal történő kombinálását a kifinomultabb hibakezelés biztosítása érdekében.</span><span class="sxs-lookup"><span data-stu-id="54c11-145">Consider combining bulkheads with retry, circuit breaker, and throttling patterns to provide more sophisticated fault handling.</span></span>
- <span data-ttu-id="54c11-146">A fogyasztók válaszfalakkal történő particionálása esetén érdemes megfontolni a folyamatok, szálkészletek és szemaforok használatát.</span><span class="sxs-lookup"><span data-stu-id="54c11-146">When partitioning consumers into bulkheads, consider using processes, thread pools, and semaphores.</span></span> <span data-ttu-id="54c11-147">A [Netflix Hystrix][hystrix] és [Polly][polly], illetve hasonló projektek biztosítják a keretrendszert a fogyasztói válaszfalak létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="54c11-147">Projects like [Netflix Hystrix][hystrix] and [Polly][polly] offer a framework for creating consumer bulkheads.</span></span>
- <span data-ttu-id="54c11-148">A szolgáltatások válaszfalakkal történő particionálása esetén érdemes megfontolni a különálló virtuális gépeken, tárolókban vagy folyamatokban történő üzembe helyezést.</span><span class="sxs-lookup"><span data-stu-id="54c11-148">When partitioning services into bulkheads, consider deploying them into separate virtual machines, containers, or processes.</span></span> <span data-ttu-id="54c11-149">A tárolók viszonylag csekély többletterhelés mellett biztosítják az erőforrások elkülönítését.</span><span class="sxs-lookup"><span data-stu-id="54c11-149">Containers offer a good balance of resource isolation with fairly low overhead.</span></span>
- <span data-ttu-id="54c11-150">Az aszinkron üzenetekkel kommunikáló szolgáltatások különböző üzenetsor-készletekkel különíthetők el.</span><span class="sxs-lookup"><span data-stu-id="54c11-150">Services that communicate using asynchronous messages can be isolated through different sets of queues.</span></span> <span data-ttu-id="54c11-151">Minden üzenetsor rendelkezhet egy dedikált példánykészlettel, amely feldolgozza az üzenetsorban található üzeneteket, vagy egyetlen példánycsoporttal, amely egy algoritmussal eltávolítja az üzeneteket a sorból, és kiosztja őket feldolgozásra.</span><span class="sxs-lookup"><span data-stu-id="54c11-151">Each queue can have a dedicated set of instances processing messages on the queue, or a single group of instances using an algorithm to dequeue and dispatch processing.</span></span>
- <span data-ttu-id="54c11-152">Határozza meg a választófalak részletességi szintjét.</span><span class="sxs-lookup"><span data-stu-id="54c11-152">Determine the level of granularity for the bulkheads.</span></span> <span data-ttu-id="54c11-153">Például több partícióra kiterjedő bérlők terjeszteni szeretné, ha, sikerült helyezze el az egyes bérlők egy külön partícióban, vagy akár többet elhelyezi egy partíciót.</span><span class="sxs-lookup"><span data-stu-id="54c11-153">For example, if you want to distribute tenants across partitions, you could place each tenant into a separate partition, or put several tenants into one partition.</span></span>
- <span data-ttu-id="54c11-154">Monitorozza az egyes partíciók teljesítményét és SLA-ját.</span><span class="sxs-lookup"><span data-stu-id="54c11-154">Monitor each partition’s performance and SLA.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="54c11-155">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="54c11-155">When to use this pattern</span></span>

<span data-ttu-id="54c11-156">Ez a minta a következőkre használható:</span><span class="sxs-lookup"><span data-stu-id="54c11-156">Use this pattern to:</span></span>

- <span data-ttu-id="54c11-157">háttérszolgáltatások készletét fogyasztó erőforrások elkülönítése, különösen, ha az alkalmazás akkor is működőképes marad valamilyen szinten, ha az egyik szolgáltatás nem válaszol;</span><span class="sxs-lookup"><span data-stu-id="54c11-157">Isolate resources used to consume a set of backend services, especially if the application can provide some level of functionality even when one of the services is not responding.</span></span>
- <span data-ttu-id="54c11-158">kritikus fontosságú fogyasztók különválasztása a standard fogyasztóktól;</span><span class="sxs-lookup"><span data-stu-id="54c11-158">Isolate critical consumers from standard consumers.</span></span>
- <span data-ttu-id="54c11-159">az alkalmazás védelme a lépcsőzetesen terjedő hibákkal szemben.</span><span class="sxs-lookup"><span data-stu-id="54c11-159">Protect the application from cascading failures.</span></span>

<span data-ttu-id="54c11-160">Nem érdemes ezt a mintát használni, ha:</span><span class="sxs-lookup"><span data-stu-id="54c11-160">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="54c11-161">az erőforrások kevésbé hatékony felhasználása nem elfogadható a projektben;</span><span class="sxs-lookup"><span data-stu-id="54c11-161">Less efficient use of resources may not be acceptable in the project.</span></span>
- <span data-ttu-id="54c11-162">a hozzáadott összetettség nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="54c11-162">The added complexity is not necessary</span></span>

## <a name="example"></a><span data-ttu-id="54c11-163">Példa</span><span class="sxs-lookup"><span data-stu-id="54c11-163">Example</span></span>

<span data-ttu-id="54c11-164">Az alábbi Kubernetes konfigurációs fájl egy egyetlen szolgáltatást futtató elkülönített tárolót hoz létre, amely saját processzor- és memória-erőforrásokkal, valamint korlátokkal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="54c11-164">The following Kubernetes configuration file creates an isolated container to run a single service, with its own CPU and memory resources and limits.</span></span>

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

## <a name="related-guidance"></a><span data-ttu-id="54c11-165">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="54c11-165">Related guidance</span></span>

- [<span data-ttu-id="54c11-166">Rugalmas alkalmazások tervezése az Azure számára</span><span class="sxs-lookup"><span data-stu-id="54c11-166">Designing resilient applications for Azure</span></span>](../resiliency/index.md)
- [<span data-ttu-id="54c11-167">Áramkör-megszakító minta</span><span class="sxs-lookup"><span data-stu-id="54c11-167">Circuit Breaker pattern</span></span>](./circuit-breaker.md)
- [<span data-ttu-id="54c11-168">Újrapróbálkozási minta</span><span class="sxs-lookup"><span data-stu-id="54c11-168">Retry pattern</span></span>](./retry.md)
- [<span data-ttu-id="54c11-169">Szabályozási minta</span><span class="sxs-lookup"><span data-stu-id="54c11-169">Throttling pattern</span></span>](./throttling.md)

<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly
