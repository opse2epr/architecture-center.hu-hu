---
title: Átjáró-útválasztási minta
titleSuffix: Cloud Design Patterns
description: Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 4db98038f582e0315a743a55d46013d2eda187e3
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010477"
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="52a5a-104">Átjáró-útválasztási minta</span><span class="sxs-lookup"><span data-stu-id="52a5a-104">Gateway Routing pattern</span></span>

<span data-ttu-id="52a5a-105">Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.</span><span class="sxs-lookup"><span data-stu-id="52a5a-105">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="52a5a-106">Ez a minta akkor lehet hasznos, ha több szolgáltatást kíván elérhetővé tenni egyetlen végponton, és a kérés alapján szeretné elvégezni az átirányítást a megfelelő szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="52a5a-106">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="52a5a-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="52a5a-107">Context and problem</span></span>

<span data-ttu-id="52a5a-108">Ha az ügyfél több szolgáltatást is használ, az ügyfél számára nehéz feladat külön végpontot létrehozni minden szolgáltatás számára, és felügyelni azokat.</span><span class="sxs-lookup"><span data-stu-id="52a5a-108">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="52a5a-109">Egy e-kereskedelmi alkalmazás például olyan szolgáltatásokat biztosíthat, mint a keresés, az értékelések, a kosár, a fizetés és a rendelési előzmények.</span><span class="sxs-lookup"><span data-stu-id="52a5a-109">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="52a5a-110">Mindegyik szolgáltatáshoz külön API tartozik. Az ügyfél ezeken keresztül éri el a szolgáltatásokat, és ismernie kell minden egyes végpontot ahhoz, hogy csatlakozhasson a szolgáltatásokhoz.</span><span class="sxs-lookup"><span data-stu-id="52a5a-110">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="52a5a-111">Ha egy API megváltozik, az ügyfelet is frissíteni kell.</span><span class="sxs-lookup"><span data-stu-id="52a5a-111">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="52a5a-112">Ha újrabont egy szolgáltatást két vagy három önálló szolgáltatásra, a szolgáltatás és az ügyfél kódját is módosítani kell.</span><span class="sxs-lookup"><span data-stu-id="52a5a-112">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="52a5a-113">Megoldás</span><span class="sxs-lookup"><span data-stu-id="52a5a-113">Solution</span></span>

<span data-ttu-id="52a5a-114">Helyezzen egy átjárót az alkalmazások, szolgáltatások vagy telepítések csoportja elé.</span><span class="sxs-lookup"><span data-stu-id="52a5a-114">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="52a5a-115">Az alkalmazás 7. rétegbeli útválasztását használva irányítsa át a kérést a megfelelő példányokhoz.</span><span class="sxs-lookup"><span data-stu-id="52a5a-115">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="52a5a-116">Ezzel a mintával az ügyfélalkalmazásnak csupán egyetlen végpontot kell ismernie, és azzal kell kommunikálnia.</span><span class="sxs-lookup"><span data-stu-id="52a5a-116">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="52a5a-117">Szolgáltatások egyesítése vagy szétbontása után az ügyfelet nem feltétlenül kell frissíteni.</span><span class="sxs-lookup"><span data-stu-id="52a5a-117">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="52a5a-118">Az továbbra is küldhet kéréseket az átjárónak, csak az útválasztás változik.</span><span class="sxs-lookup"><span data-stu-id="52a5a-118">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="52a5a-119">Az átjáró ezenkívül lehetővé teszi a háttérszolgáltatások elkülönítését az ügyfelektől, így az ügyfél hívásai egyszerűek maradnak, de az átjáró mögötti háttérszolgáltatások módosíthatók lesznek.</span><span class="sxs-lookup"><span data-stu-id="52a5a-119">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="52a5a-120">Az ügyfél hívásai átirányíthatók a szolgáltatáshoz vagy szolgáltatásokhoz, amely(ek)nek kezelnie kell a várt ügyfélviselkedést. Ez lehetővé teszi szolgáltatások hozzáadását, felosztását és átszervezését az átjáró mögött az ügyfél módosítása nélkül.</span><span class="sxs-lookup"><span data-stu-id="52a5a-120">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![Az átjáró-útválasztási minta ábrája](./_images/gateway-routing.png)

<span data-ttu-id="52a5a-122">Ez a minta segíthet a telepítés során is, mivel lehetővé teszi a frissítések bevezetésének kezelését.</span><span class="sxs-lookup"><span data-stu-id="52a5a-122">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="52a5a-123">A szolgáltatás újabb verziójának telepítésekor az az aktuális verzióval egyidejűleg telepíthető.</span><span class="sxs-lookup"><span data-stu-id="52a5a-123">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="52a5a-124">Útválasztás segítségével szabályozhatja, hogy a szolgáltatás melyik verzióját bemutatják az ügyfelektől, így rugalmasan kiadási stratégiát alkalmazhat fokozatos, egyidejű vagy frissítések kibocsátások befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="52a5a-124">Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="52a5a-125">Az új szolgáltatás telepítése után felfedezett hibák gyorsan visszavonhatók az átjáró konfigurációjának módosításával, ami nem befolyásolja az ügyfeleket.</span><span class="sxs-lookup"><span data-stu-id="52a5a-125">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="52a5a-126">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="52a5a-126">Issues and considerations</span></span>

- <span data-ttu-id="52a5a-127">Az átjárószolgáltatás kritikus meghibásodási pont lehet a rendszeren belül.</span><span class="sxs-lookup"><span data-stu-id="52a5a-127">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="52a5a-128">Győződjön meg arról, hogy az átjáró konfigurációja megfelelően ki tudja szolgálni a rendelkezésre állási követelményeket.</span><span class="sxs-lookup"><span data-stu-id="52a5a-128">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="52a5a-129">Az implementálás során tartsa szem előtt a rugalmassági és hibatűrési képességeket.</span><span class="sxs-lookup"><span data-stu-id="52a5a-129">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="52a5a-130">Az átjárószolgáltatás szűk keresztmetszetté válhat.</span><span class="sxs-lookup"><span data-stu-id="52a5a-130">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="52a5a-131">Győződjön meg arról, hogy az átjáró megfelelő teljesítménnyel bír a várható terhelés kezelésére, és skálázható a várható későbbi növekedésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="52a5a-131">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="52a5a-132">Végezzen terhelésteszteket az átjárón annak ellenőrzéséhez, hogy az nem indít-e el hibasorozatokat a szolgáltatásokban.</span><span class="sxs-lookup"><span data-stu-id="52a5a-132">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="52a5a-133">Az átjáró útválasztása 7. szintű.</span><span class="sxs-lookup"><span data-stu-id="52a5a-133">Gateway routing is level 7.</span></span> <span data-ttu-id="52a5a-134">Az útválasztás történhet IP-cím, port, fejléc vagy URL-cím alapján.</span><span class="sxs-lookup"><span data-stu-id="52a5a-134">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="52a5a-135">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="52a5a-135">When to use this pattern</span></span>

<span data-ttu-id="52a5a-136">Használja ezt a mintát, ha:</span><span class="sxs-lookup"><span data-stu-id="52a5a-136">Use this pattern when:</span></span>

- <span data-ttu-id="52a5a-137">Az ügyfélnek több szolgáltatást kell használnia, amelyek egy átjáró mögött érhetőek el.</span><span class="sxs-lookup"><span data-stu-id="52a5a-137">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="52a5a-138">Egyetlen végpont használatával kívánja egyszerűsíteni az ügyfélalkalmazásokat.</span><span class="sxs-lookup"><span data-stu-id="52a5a-138">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="52a5a-139">A kérést kívülről címezhető végpontokról belső virtuális végpontokra kell átirányítani (például egy virtuális gép portjainak elérhetővé tétele virtuális IP-címek fürtözéséhez).</span><span class="sxs-lookup"><span data-stu-id="52a5a-139">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="52a5a-140">Nem érdemes ezt a mintát használni olyan egyszerű alkalmazásokhoz, amelyek csupán egy vagy két szolgáltatást használnak.</span><span class="sxs-lookup"><span data-stu-id="52a5a-140">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="52a5a-141">Példa</span><span class="sxs-lookup"><span data-stu-id="52a5a-141">Example</span></span>

<span data-ttu-id="52a5a-142">Az Nginxet útválasztóként használva létrehozhatja ezt az egyszerű konfigurációs fájt, amely a különböző virtuális könyvtárakban található alkalmazások kéréseit a háttérrendszer különböző gépeire irányíthatja át.</span><span class="sxs-lookup"><span data-stu-id="52a5a-142">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```console
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a><span data-ttu-id="52a5a-143">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="52a5a-143">Related guidance</span></span>

- [<span data-ttu-id="52a5a-144">Háttérrendszerek és előtérrendszerek minta</span><span class="sxs-lookup"><span data-stu-id="52a5a-144">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="52a5a-145">Átjáróösszesítés minta</span><span class="sxs-lookup"><span data-stu-id="52a5a-145">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="52a5a-146">Átjárókiürítés minta</span><span class="sxs-lookup"><span data-stu-id="52a5a-146">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
