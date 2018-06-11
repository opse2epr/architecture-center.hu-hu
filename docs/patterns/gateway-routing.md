---
title: Átjáró-útválasztási minta
description: Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ea0bc4d31b745043a7ac3afb277dfc46d87ff109
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252600"
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="7ed00-103">Átjáró-útválasztási minta</span><span class="sxs-lookup"><span data-stu-id="7ed00-103">Gateway Routing pattern</span></span>

<span data-ttu-id="7ed00-104">Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.</span><span class="sxs-lookup"><span data-stu-id="7ed00-104">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="7ed00-105">Ez a minta akkor lehet hasznos, ha több szolgáltatást kíván elérhetővé tenni egyetlen végponton, és a kérés alapján szeretné elvégezni az átirányítást a megfelelő szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="7ed00-105">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="7ed00-106">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="7ed00-106">Context and problem</span></span>

<span data-ttu-id="7ed00-107">Ha az ügyfél több szolgáltatást is használ, az ügyfél számára nehéz feladat külön végpontot létrehozni minden szolgáltatás számára, és felügyelni azokat.</span><span class="sxs-lookup"><span data-stu-id="7ed00-107">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="7ed00-108">Egy e-kereskedelmi alkalmazás például olyan szolgáltatásokat biztosíthat, mint a keresés, az értékelések, a kosár, a fizetés és a rendelési előzmények.</span><span class="sxs-lookup"><span data-stu-id="7ed00-108">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="7ed00-109">Mindegyik szolgáltatáshoz külön API tartozik. Az ügyfél ezeken keresztül éri el a szolgáltatásokat, és ismernie kell minden egyes végpontot ahhoz, hogy csatlakozhasson a szolgáltatásokhoz.</span><span class="sxs-lookup"><span data-stu-id="7ed00-109">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="7ed00-110">Ha egy API megváltozik, az ügyfelet is frissíteni kell.</span><span class="sxs-lookup"><span data-stu-id="7ed00-110">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="7ed00-111">Ha újrabont egy szolgáltatást két vagy három önálló szolgáltatásra, a szolgáltatás és az ügyfél kódját is módosítani kell.</span><span class="sxs-lookup"><span data-stu-id="7ed00-111">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="7ed00-112">Megoldás</span><span class="sxs-lookup"><span data-stu-id="7ed00-112">Solution</span></span>

<span data-ttu-id="7ed00-113">Helyezzen egy átjárót az alkalmazások, szolgáltatások vagy telepítések csoportja elé.</span><span class="sxs-lookup"><span data-stu-id="7ed00-113">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="7ed00-114">Az alkalmazás 7. rétegbeli útválasztását használva irányítsa át a kérést a megfelelő példányokhoz.</span><span class="sxs-lookup"><span data-stu-id="7ed00-114">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="7ed00-115">Ezzel a mintával az ügyfélalkalmazásnak csupán egyetlen végpontot kell ismernie, és azzal kell kommunikálnia.</span><span class="sxs-lookup"><span data-stu-id="7ed00-115">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="7ed00-116">Szolgáltatások egyesítése vagy szétbontása után az ügyfelet nem feltétlenül kell frissíteni.</span><span class="sxs-lookup"><span data-stu-id="7ed00-116">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="7ed00-117">Az továbbra is küldhet kéréseket az átjárónak, csak az útválasztás változik.</span><span class="sxs-lookup"><span data-stu-id="7ed00-117">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="7ed00-118">Az átjáró ezenkívül lehetővé teszi a háttérszolgáltatások elkülönítését az ügyfelektől, így az ügyfél hívásai egyszerűek maradnak, de az átjáró mögötti háttérszolgáltatások módosíthatók lesznek.</span><span class="sxs-lookup"><span data-stu-id="7ed00-118">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="7ed00-119">Az ügyfél hívásai átirányíthatók a szolgáltatáshoz vagy szolgáltatásokhoz, amely(ek)nek kezelnie kell a várt ügyfélviselkedést. Ez lehetővé teszi szolgáltatások hozzáadását, felosztását és átszervezését az átjáró mögött az ügyfél módosítása nélkül.</span><span class="sxs-lookup"><span data-stu-id="7ed00-119">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![](./_images/gateway-routing.png)
 
<span data-ttu-id="7ed00-120">Ez a minta segíthet a telepítés során is, mivel lehetővé teszi a frissítések bevezetésének kezelését.</span><span class="sxs-lookup"><span data-stu-id="7ed00-120">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="7ed00-121">A szolgáltatás újabb verziójának telepítésekor az az aktuális verzióval egyidejűleg telepíthető.</span><span class="sxs-lookup"><span data-stu-id="7ed00-121">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="7ed00-122">Útválasztás lehetővé teszi szabályozhatja, hogy melyik verzió a szolgáltatás nem jelenik meg az ügyfelek, felkínálva a rugalmasságot különböző kiadás stratégiák használata növekményes, hogy párhuzamosan, vagy a frissítések végrehajtása befejeződik.</span><span class="sxs-lookup"><span data-stu-id="7ed00-122">Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="7ed00-123">Az új szolgáltatás telepítése után felfedezett hibák gyorsan visszavonhatók az átjáró konfigurációjának módosításával, ami nem befolyásolja az ügyfeleket.</span><span class="sxs-lookup"><span data-stu-id="7ed00-123">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="7ed00-124">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="7ed00-124">Issues and considerations</span></span>

- <span data-ttu-id="7ed00-125">Az átjárószolgáltatás kritikus meghibásodási pont lehet a rendszeren belül.</span><span class="sxs-lookup"><span data-stu-id="7ed00-125">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="7ed00-126">Győződjön meg arról, hogy az átjáró konfigurációja megfelelően ki tudja szolgálni a rendelkezésre állási követelményeket.</span><span class="sxs-lookup"><span data-stu-id="7ed00-126">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="7ed00-127">Az implementálás során tartsa szem előtt a rugalmassági és hibatűrési képességeket.</span><span class="sxs-lookup"><span data-stu-id="7ed00-127">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="7ed00-128">Az átjárószolgáltatás szűk keresztmetszetté válhat.</span><span class="sxs-lookup"><span data-stu-id="7ed00-128">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="7ed00-129">Győződjön meg arról, hogy az átjáró megfelelő teljesítménnyel bír a várható terhelés kezelésére, és skálázható a várható későbbi növekedésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="7ed00-129">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="7ed00-130">Végezzen terhelésteszteket az átjárón annak ellenőrzéséhez, hogy az nem indít-e el hibasorozatokat a szolgáltatásokban.</span><span class="sxs-lookup"><span data-stu-id="7ed00-130">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="7ed00-131">Az átjáró útválasztása 7. szintű.</span><span class="sxs-lookup"><span data-stu-id="7ed00-131">Gateway routing is level 7.</span></span> <span data-ttu-id="7ed00-132">Az útválasztás történhet IP-cím, port, fejléc vagy URL-cím alapján.</span><span class="sxs-lookup"><span data-stu-id="7ed00-132">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="7ed00-133">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="7ed00-133">When to use this pattern</span></span>

<span data-ttu-id="7ed00-134">Használja ezt a mintát, ha:</span><span class="sxs-lookup"><span data-stu-id="7ed00-134">Use this pattern when:</span></span>

- <span data-ttu-id="7ed00-135">Az ügyfélnek több szolgáltatást kell használnia, amelyek egy átjáró mögött érhetőek el.</span><span class="sxs-lookup"><span data-stu-id="7ed00-135">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="7ed00-136">Egyetlen végpont használatával kívánja egyszerűsíteni az ügyfélalkalmazásokat.</span><span class="sxs-lookup"><span data-stu-id="7ed00-136">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="7ed00-137">A kérést kívülről címezhető végpontokról belső virtuális végpontokra kell átirányítani (például egy virtuális gép portjainak elérhetővé tétele virtuális IP-címek fürtözéséhez).</span><span class="sxs-lookup"><span data-stu-id="7ed00-137">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="7ed00-138">Nem érdemes ezt a mintát használni olyan egyszerű alkalmazásokhoz, amelyek csupán egy vagy két szolgáltatást használnak.</span><span class="sxs-lookup"><span data-stu-id="7ed00-138">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="7ed00-139">Példa</span><span class="sxs-lookup"><span data-stu-id="7ed00-139">Example</span></span>

<span data-ttu-id="7ed00-140">Az Nginxet útválasztóként használva létrehozhatja ezt az egyszerű konfigurációs fájt, amely a különböző virtuális könyvtárakban található alkalmazások kéréseit a háttérrendszer különböző gépeire irányíthatja át.</span><span class="sxs-lookup"><span data-stu-id="7ed00-140">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```
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

## <a name="related-guidance"></a><span data-ttu-id="7ed00-141">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="7ed00-141">Related guidance</span></span>

- [<span data-ttu-id="7ed00-142">Háttérrendszerek és előtérrendszerek minta</span><span class="sxs-lookup"><span data-stu-id="7ed00-142">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="7ed00-143">Átjáróösszesítés minta</span><span class="sxs-lookup"><span data-stu-id="7ed00-143">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="7ed00-144">Átjárókiürítés minta</span><span class="sxs-lookup"><span data-stu-id="7ed00-144">Gateway Offloading pattern</span></span>](./gateway-offloading.md)



