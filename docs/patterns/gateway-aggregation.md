---
title: Átjáró összesítési minta
description: Segítségével egy átjáró több egyes kérelmeket az egy kérelemhez.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541273"
---
# <a name="gateway-aggregation-pattern"></a><span data-ttu-id="71277-103">Átjáró összesítési minta</span><span class="sxs-lookup"><span data-stu-id="71277-103">Gateway Aggregation pattern</span></span>

<span data-ttu-id="71277-104">Segítségével egy átjáró több egyes kérelmeket az egy kérelemhez.</span><span class="sxs-lookup"><span data-stu-id="71277-104">Use a gateway to aggregate multiple individual requests into a single request.</span></span> <span data-ttu-id="71277-105">Ebben a mintában akkor hasznos, ha egy ügyfél kell végrehajtania egy műveletet több különböző háttérrendszerek hívások.</span><span class="sxs-lookup"><span data-stu-id="71277-105">This pattern is useful when a client must make multiple calls to different backend systems to perform an operation.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="71277-106">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="71277-106">Context and problem</span></span>

<span data-ttu-id="71277-107">Egy feladat végrehajtásához egy ügyfél lehet több hívások különböző háttér-szolgáltatásokra.</span><span class="sxs-lookup"><span data-stu-id="71277-107">To perform a single task, a client may have to make multiple calls to various backend services.</span></span> <span data-ttu-id="71277-108">Olyan alkalmazás, amely a feladat végrehajtása számos szolgáltatás alapul erőforrások minden kérelemnél meg kell kiadás.</span><span class="sxs-lookup"><span data-stu-id="71277-108">An application that relies on many services to perform a task must expend resources on each request.</span></span> <span data-ttu-id="71277-109">Bármely új funkció vagy szolgáltatás való hozzáadásakor az alkalmazást, szükség van-e a további kérelmeket, további növekvő erőforrás-követelmények és hálózati hívások.</span><span class="sxs-lookup"><span data-stu-id="71277-109">When any new feature or service is added to the application, additional requests are needed, further increasing resource requirements and network calls.</span></span> <span data-ttu-id="71277-110">Az ügyfél és a háttérkiszolgáló közötti chattiness negatívan befolyásolhatja a teljesítményt és a skála az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="71277-110">This chattiness between a client and a backend can adversely impact the performance and scale of the application.</span></span>  <span data-ttu-id="71277-111">Mikroszolgáltatási architektúra végzett probléma gyakori, számos kisebb szolgáltatás természetes épül alkalmazások a kereszt-hívások nagyobb mennyiségű van.</span><span class="sxs-lookup"><span data-stu-id="71277-111">Microservice architectures have made this problem more common, as applications built around many smaller services naturally have a higher amount of cross-service calls.</span></span> 

<span data-ttu-id="71277-112">Az alábbi ábra az ügyfél kérelmeket küld arra az egyes szolgáltatások (1,2,3).</span><span class="sxs-lookup"><span data-stu-id="71277-112">In the following diagram, the client sends requests to each service (1,2,3).</span></span> <span data-ttu-id="71277-113">Minden szolgáltatás feldolgozza a kérést, és visszaküldi a választ az alkalmazás (4,5,6).</span><span class="sxs-lookup"><span data-stu-id="71277-113">Each service processes the request and sends the response back to the application (4,5,6).</span></span> <span data-ttu-id="71277-114">Mobilhálózati egy és általában nagy késésű az egyes kérelmek használatával az ilyen módon nem hatékony, és megszakadt kapcsolat vagy hiányos kérelmeket eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="71277-114">Over a cellular network with typically high latency, using individual requests in this manner is inefficient and could result in broken connectivity or incomplete requests.</span></span> <span data-ttu-id="71277-115">Minden egyes kérelem végezhető párhuzamos, amíg az alkalmazás kell küldeni, várjon, amíg és feldolgozni az adatokat az egyes kérelmek összes külön kapcsolatok, növekvő valószínűsége. a hiba a.</span><span class="sxs-lookup"><span data-stu-id="71277-115">While each request may be done in parallel, the application must send, wait, and process data for each request, all on separate connections, increasing the chance of failure.</span></span>

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a><span data-ttu-id="71277-116">Megoldás</span><span class="sxs-lookup"><span data-stu-id="71277-116">Solution</span></span>

<span data-ttu-id="71277-117">Átjáró használatával az ügyfél és a szolgáltatások közötti chattiness csökkenthető.</span><span class="sxs-lookup"><span data-stu-id="71277-117">Use a gateway to reduce chattiness between the client and the services.</span></span> <span data-ttu-id="71277-118">Az átjáró ügyfél kéréseket fogad, a különböző rendszerek háttér kérelmek kiszállítja aggregálja az eredményeket és elküldi azokat a kérelmező ügyfélnek.</span><span class="sxs-lookup"><span data-stu-id="71277-118">The gateway receives client requests, dispatches requests to the various backend systems, and then aggregates the results and sends them back to the requesting client.</span></span>

<span data-ttu-id="71277-119">Ebben a mintában csökkenti az alkalmazás által az háttér-services-kérelmek számát, és az alkalmazások teljesítményének javítása a nagy késleltetésű hálózaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="71277-119">This pattern can reduce the number of requests that the application makes to backend services, and improve application performance over high-latency networks.</span></span>

<span data-ttu-id="71277-120">A következő ábrán az az alkalmazás egy kérést küld az átjáró (1).</span><span class="sxs-lookup"><span data-stu-id="71277-120">In the following diagram, the application sends a request to the gateway (1).</span></span> <span data-ttu-id="71277-121">A kérelem tartalmazza a további kérelmeket tartalmazó csomagot.</span><span class="sxs-lookup"><span data-stu-id="71277-121">The request contains a package of additional requests.</span></span> <span data-ttu-id="71277-122">Az átjáró bontja a ezeket, és feldolgozza a minden kérelmet elküldheti a megfelelő szolgáltatást (2).</span><span class="sxs-lookup"><span data-stu-id="71277-122">The gateway decomposes these and processes each request by sending it to the relevant service (2).</span></span> <span data-ttu-id="71277-123">Minden szolgáltatás visszaküld egy választ az átjáró (3).</span><span class="sxs-lookup"><span data-stu-id="71277-123">Each service returns a response to the gateway (3).</span></span> <span data-ttu-id="71277-124">Az átjáró egyesíti az egyes szolgáltatások válaszát, és visszaküldi a választ az alkalmazáshoz (4).</span><span class="sxs-lookup"><span data-stu-id="71277-124">The gateway combines the responses from each service and sends the response to the application (4).</span></span> <span data-ttu-id="71277-125">Az alkalmazás egy egyetlen kérést, és csak egyetlen választ kap az átjárót.</span><span class="sxs-lookup"><span data-stu-id="71277-125">The application makes a single request and receives only a single response from the gateway.</span></span>

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="71277-126">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="71277-126">Issues and considerations</span></span>

- <span data-ttu-id="71277-127">Az átjáró nem kell vezetnie a szolgáltatás kapcsolódási háttér szolgáltatásban.</span><span class="sxs-lookup"><span data-stu-id="71277-127">The gateway should not introduce service coupling across the backend services.</span></span>
- <span data-ttu-id="71277-128">Az átjáró közelében a háttér-szolgáltatások késés lehető legnagyobb mértékben csökkenteni kell elhelyezni.</span><span class="sxs-lookup"><span data-stu-id="71277-128">The gateway should be located near the backend services to reduce latency as much as possible.</span></span>
- <span data-ttu-id="71277-129">Az átjárószolgáltatás vezethet be a hibaérzékeny pontok kialakulását.</span><span class="sxs-lookup"><span data-stu-id="71277-129">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="71277-130">Gondoskodjon arról, hogy az átjáró megfelelően célja, hogy az alkalmazás rendelkezésre állási követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="71277-130">Ensure the gateway is properly designed to meet your application's availability requirements.</span></span>
- <span data-ttu-id="71277-131">Az átjáró a szűk keresztmetszetek vezethetnek.</span><span class="sxs-lookup"><span data-stu-id="71277-131">The gateway may introduce a bottleneck.</span></span> <span data-ttu-id="71277-132">Győződjön meg arról, az átjáró rendelkezik a megfelelő teljesítmény terhelés kezelésére, és megfelelnek a várható növekedési is méretezhető.</span><span class="sxs-lookup"><span data-stu-id="71277-132">Ensure the gateway has adequate performance to handle load and can be scaled to meet your anticipated growth.</span></span>
- <span data-ttu-id="71277-133">Hajtsa végre a terhelés tesztelése az átjárón nem vezetnek be kaszkádolt probléma van a szolgáltatások biztosításához.</span><span class="sxs-lookup"><span data-stu-id="71277-133">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="71277-134">Egy rugalmas, például a módszereket kialakításának megvalósítása [válaszfalak][bulkhead], [áramkör legfrissebb][circuit-breaker], [újra] [ retry], és időtúllépéseket okoz.</span><span class="sxs-lookup"><span data-stu-id="71277-134">Implement a resilient design, using techniques such as [bulkheads][bulkhead], [circuit breaking][circuit-breaker], [retry][retry], and timeouts.</span></span>
- <span data-ttu-id="71277-135">Ha egy vagy több hívások túl sokáig tart, előfordulhat, hogy elfogadható időtúllépési és adatok részhalmaza vissza.</span><span class="sxs-lookup"><span data-stu-id="71277-135">If one or more service calls takes too long, it may be acceptable to timeout and return a partial set of data.</span></span> <span data-ttu-id="71277-136">Vegye figyelembe, hogy az alkalmazás ebben a forgatókönyvben fogja kezelni.</span><span class="sxs-lookup"><span data-stu-id="71277-136">Consider how your application will handle this scenario.</span></span>
- <span data-ttu-id="71277-137">Aszinkron I/O segítségével győződjön meg arról, hogy a háttérkiszolgálón a késés nem teljesítményproblémákat okozhatnak az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="71277-137">Use asynchronous I/O to ensure that a delay at the backend doesn't cause performance issues in the application.</span></span>
- <span data-ttu-id="71277-138">Korrelációs azonosító segítségével nyomon követheti a minden egyes hívás elosztott nyomkövetés megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="71277-138">Implement distributed tracing using correlation IDs to track each individual call.</span></span>
- <span data-ttu-id="71277-139">A figyelő kérelem metrikák és a válasz méretét.</span><span class="sxs-lookup"><span data-stu-id="71277-139">Monitor request metrics and response sizes.</span></span>
- <span data-ttu-id="71277-140">Fontolja meg a gyorsítótárazott adatokat ad vissza, a hibák kezelésének feladatátvételi stratégiát.</span><span class="sxs-lookup"><span data-stu-id="71277-140">Consider returning cached data as a failover strategy to handle failures.</span></span>
- <span data-ttu-id="71277-141">Helyett a átjáróra kerülő összesítés létrehozása, vegye figyelembe, ha kialakít egy összesítési szolgáltatás az átjáró mögötti.</span><span class="sxs-lookup"><span data-stu-id="71277-141">Instead of building aggregation into the gateway, consider placing an aggregation service behind the gateway.</span></span> <span data-ttu-id="71277-142">A kérelem összesítési valószínűleg lesz a különböző erőforrás követelményei, mint más szolgáltatások az átjáró, és hatással lehet az átjáró az Útválasztás és kiszervezésével funkciót.</span><span class="sxs-lookup"><span data-stu-id="71277-142">Request aggregation will likely have different resource requirements than other services in the gateway and may impact the gateway's routing and offloading functionality.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="71277-143">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="71277-143">When to use this pattern</span></span>

<span data-ttu-id="71277-144">Ez mintát, mikor használja:</span><span class="sxs-lookup"><span data-stu-id="71277-144">Use this pattern when:</span></span>

- <span data-ttu-id="71277-145">Egy ügyfélnek kell végrehajtania egy műveletet több háttérbeli szolgáltatásokkal kommunikálni.</span><span class="sxs-lookup"><span data-stu-id="71277-145">A client needs to communicate with multiple backend services to perform an operation.</span></span>
- <span data-ttu-id="71277-146">Az ügyfél használhatja hálózatok jelentős késés, például cellás hálózatokon.</span><span class="sxs-lookup"><span data-stu-id="71277-146">The client may use networks with significant latency, such as cellular networks.</span></span>

<span data-ttu-id="71277-147">Ez a minta nem alkalmasak lehetnek esetén:</span><span class="sxs-lookup"><span data-stu-id="71277-147">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="71277-148">Több műveletek között az ügyfél és az egyetlen szolgáltatás közötti hívások számát csökkenteni szeretné.</span><span class="sxs-lookup"><span data-stu-id="71277-148">You want to reduce the number of calls between a client and a single service across multiple operations.</span></span> <span data-ttu-id="71277-149">A forgatókönyv, a jobb megoldás lehet egy kötegelt művelet hozzáadása a szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="71277-149">In that scenario, it may be better to add a batch operation to the service.</span></span>
- <span data-ttu-id="71277-150">Az ügyfél vagy alkalmazás található a háttér-szolgáltatások és a késés nem lényeges szempontja.</span><span class="sxs-lookup"><span data-stu-id="71277-150">The client or application is located near the backend services and latency is not a significant factor.</span></span>

## <a name="example"></a><span data-ttu-id="71277-151">Példa</span><span class="sxs-lookup"><span data-stu-id="71277-151">Example</span></span>

<span data-ttu-id="71277-152">A következő példa bemutatja, hogyan hozhat létre egy egyszerű átjáró összesítési NGINX szolgáltatás Lua.</span><span class="sxs-lookup"><span data-stu-id="71277-152">The following example illustrates how to create a simple a gateway aggregation NGINX service using Lua.</span></span>

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a><span data-ttu-id="71277-153">Kapcsolódó útmutató</span><span class="sxs-lookup"><span data-stu-id="71277-153">Related guidance</span></span>

- [<span data-ttu-id="71277-154">Háttérkiszolgálókon Frontends minta</span><span class="sxs-lookup"><span data-stu-id="71277-154">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="71277-155">Átjáró Offloading minta</span><span class="sxs-lookup"><span data-stu-id="71277-155">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
- [<span data-ttu-id="71277-156">Átjáró útválasztás minta</span><span class="sxs-lookup"><span data-stu-id="71277-156">Gateway Routing pattern</span></span>](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md