---
title: Átjáróösszesítési minta
titleSuffix: Cloud Design Patterns
description: Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c28e93efeae7dffe2f17288f3310f299ec545e1a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298581"
---
# <a name="gateway-aggregation-pattern"></a><span data-ttu-id="515cb-104">Átjáróösszesítési minta</span><span class="sxs-lookup"><span data-stu-id="515cb-104">Gateway Aggregation pattern</span></span>

<span data-ttu-id="515cb-105">Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével.</span><span class="sxs-lookup"><span data-stu-id="515cb-105">Use a gateway to aggregate multiple individual requests into a single request.</span></span> <span data-ttu-id="515cb-106">Ez a minta akkor lehet hasznos, ha az ügyfélnek különböző háttérrendszereket kell több alkalommal hívnia egy művelet végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="515cb-106">This pattern is useful when a client must make multiple calls to different backend systems to perform an operation.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="515cb-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="515cb-107">Context and problem</span></span>

<span data-ttu-id="515cb-108">Előfordulhat, hogy az ügyfélnek különböző háttérszolgáltatásokat kell többször hívnia egy feladat végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="515cb-108">To perform a single task, a client may have to make multiple calls to various backend services.</span></span> <span data-ttu-id="515cb-109">Ha egy alkalmazás számos szolgáltatásra támaszkodik egy feladat végrehajtásakor, minden kéréshez erőforrásokat kell felhasználnia.</span><span class="sxs-lookup"><span data-stu-id="515cb-109">An application that relies on many services to perform a task must expend resources on each request.</span></span> <span data-ttu-id="515cb-110">Ha bármilyen új funkciót vagy szolgáltatást adnak hozzá az alkalmazáshoz, további kérések válnak szükségessé, ami még több erőforrást és hálózati hívást igényel.</span><span class="sxs-lookup"><span data-stu-id="515cb-110">When any new feature or service is added to the application, additional requests are needed, further increasing resource requirements and network calls.</span></span> <span data-ttu-id="515cb-111">Az ügyfél és a háttérrendszerek közötti forgalom csökkentheti az alkalmazás teljesítményét és skálázhatóságát.</span><span class="sxs-lookup"><span data-stu-id="515cb-111">This chattiness between a client and a backend can adversely impact the performance and scale of the application.</span></span>  <span data-ttu-id="515cb-112">A mikroszolgáltatás-architektúrák még gyakoribbá tették ezt a problémát, mivel a számos kisebb szolgáltatásra épülő alkalmazásokban több szolgáltatások közötti hívás fordul elő.</span><span class="sxs-lookup"><span data-stu-id="515cb-112">Microservice architectures have made this problem more common, as applications built around many smaller services naturally have a higher amount of cross-service calls.</span></span>

<span data-ttu-id="515cb-113">A következő ábrán az ügyfél minden egyes szolgáltatásnak (1,2,3) kéréseket küld.</span><span class="sxs-lookup"><span data-stu-id="515cb-113">In the following diagram, the client sends requests to each service (1,2,3).</span></span> <span data-ttu-id="515cb-114">A szolgáltatások feldolgozzák a kéréseket, és válaszolnak az alkalmazásnak (4,5,6).</span><span class="sxs-lookup"><span data-stu-id="515cb-114">Each service processes the request and sends the response back to the application (4,5,6).</span></span> <span data-ttu-id="515cb-115">A jellemzően nagy hálózati késésű mobilhálózatokon az egyedi kérések ilyen módon történő használata nem hatékony, és ilyenkor előfordulhat, hogy megszakad a kapcsolat, vagy a kérések csak részlegesen teljesülnek.</span><span class="sxs-lookup"><span data-stu-id="515cb-115">Over a cellular network with typically high latency, using individual requests in this manner is inefficient and could result in broken connectivity or incomplete requests.</span></span> <span data-ttu-id="515cb-116">Bár az egyes kérések feldolgozása párhuzamosan történik, az alkalmazásnak minden kérés adatait el kell küldenie, megvárni a választ, és feldolgoznia az egyes kérések adatait. Mindez önálló kapcsolatokkal történik, ami növeli a hiba esélyét.</span><span class="sxs-lookup"><span data-stu-id="515cb-116">While each request may be done in parallel, the application must send, wait, and process data for each request, all on separate connections, increasing the chance of failure.</span></span>

![Az átjáró Átjáróösszesítés minta probléma diagramja](./_images/gateway-aggregation-problem.png)

## <a name="solution"></a><span data-ttu-id="515cb-118">Megoldás</span><span class="sxs-lookup"><span data-stu-id="515cb-118">Solution</span></span>

<span data-ttu-id="515cb-119">Egy átjáró használatával csökkentheti az ügyfél és a szolgáltatások közötti forgalmat.</span><span class="sxs-lookup"><span data-stu-id="515cb-119">Use a gateway to reduce chattiness between the client and the services.</span></span> <span data-ttu-id="515cb-120">Az átjáró fogadja az ügyfél kéréseit, továbbítja a kéréseket a különböző háttérrendszerekhez, majd összesíti az eredményeket, és visszaküldi őket a kérést küldő ügyfélnek.</span><span class="sxs-lookup"><span data-stu-id="515cb-120">The gateway receives client requests, dispatches requests to the various backend systems, and then aggregates the results and sends them back to the requesting client.</span></span>

<span data-ttu-id="515cb-121">Ez a minta csökkentheti az alkalmazás által a háttérszolgáltatásoknak küldött kérések számát, ami javítja az alkalmazás teljesítményét a nagy késésű hálózatokban.</span><span class="sxs-lookup"><span data-stu-id="515cb-121">This pattern can reduce the number of requests that the application makes to backend services, and improve application performance over high-latency networks.</span></span>

<span data-ttu-id="515cb-122">A következő ábrán az alkalmazás egy kérést küld az átjárónak (1).</span><span class="sxs-lookup"><span data-stu-id="515cb-122">In the following diagram, the application sends a request to the gateway (1).</span></span> <span data-ttu-id="515cb-123">A kérésben egy további kéréseket tartalmazó csomag található.</span><span class="sxs-lookup"><span data-stu-id="515cb-123">The request contains a package of additional requests.</span></span> <span data-ttu-id="515cb-124">Az átjáró kibontja a csomagot, és feldolgozza az egyes kéréseket úgy, hogy a megfelelő szolgáltatáshoz irányítja őket (2).</span><span class="sxs-lookup"><span data-stu-id="515cb-124">The gateway decomposes these and processes each request by sending it to the relevant service (2).</span></span> <span data-ttu-id="515cb-125">Minden egyes szolgáltatás elküldi a választ az átjárónak (3).</span><span class="sxs-lookup"><span data-stu-id="515cb-125">Each service returns a response to the gateway (3).</span></span> <span data-ttu-id="515cb-126">Az átjáró egyesíti az egyes szolgáltatásoktól érkező válaszokat, és azokat egyetlen válaszként továbbítja az alkalmazásnak (4).</span><span class="sxs-lookup"><span data-stu-id="515cb-126">The gateway combines the responses from each service and sends the response to the application (4).</span></span> <span data-ttu-id="515cb-127">Az alkalmazás egyetlen kérést küld, és egyetlen választ kap az átjárótól.</span><span class="sxs-lookup"><span data-stu-id="515cb-127">The application makes a single request and receives only a single response from the gateway.</span></span>

![Az átjáró Átjáróösszesítés minta megoldás diagramja](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="515cb-129">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="515cb-129">Issues and considerations</span></span>

- <span data-ttu-id="515cb-130">Érdemes ügyelni rá, hogy az átjáró ne kapcsoljon össze háttérszolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="515cb-130">The gateway should not introduce service coupling across the backend services.</span></span>
- <span data-ttu-id="515cb-131">Az átjárót a háttérszolgáltatások közelében kell elhelyezni, hogy a lehető legkisebb késés jelentkezzen.</span><span class="sxs-lookup"><span data-stu-id="515cb-131">The gateway should be located near the backend services to reduce latency as much as possible.</span></span>
- <span data-ttu-id="515cb-132">Az átjárószolgáltatás kritikus meghibásodási pont lehet a rendszeren belül.</span><span class="sxs-lookup"><span data-stu-id="515cb-132">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="515cb-133">Győződjön meg arról, hogy az átjáró megfelelően ki tudja szolgálni az alkalmazás rendelkezésre állási követelményeit.</span><span class="sxs-lookup"><span data-stu-id="515cb-133">Ensure the gateway is properly designed to meet your application's availability requirements.</span></span>
- <span data-ttu-id="515cb-134">Az átjáró szűk keresztmetszetté válhat.</span><span class="sxs-lookup"><span data-stu-id="515cb-134">The gateway may introduce a bottleneck.</span></span> <span data-ttu-id="515cb-135">Ellenőrizze, hogy az átjáró teljesítménye megfelelő-e a várható terhelés kezeléséhez, és skálázható-e az esetleges későbbi növekedésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="515cb-135">Ensure the gateway has adequate performance to handle load and can be scaled to meet your anticipated growth.</span></span>
- <span data-ttu-id="515cb-136">Végezzen terhelésteszteket az átjárón annak ellenőrzéséhez, hogy az nem indít-e el hibasorozatokat a szolgáltatásokban.</span><span class="sxs-lookup"><span data-stu-id="515cb-136">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="515cb-137">Implementáljon egy rugalmas rendszert [válaszfalak][bulkhead], [áramkör-megszakítás][circuit-breaker], [újrapróbálkozások][retry], időtúllépés és hasonló technikák használatával.</span><span class="sxs-lookup"><span data-stu-id="515cb-137">Implement a resilient design, using techniques such as [bulkheads][bulkhead], [circuit breaking][circuit-breaker], [retry][retry], and timeouts.</span></span>
- <span data-ttu-id="515cb-138">Ha egy vagy több szolgáltatás hívása túl sok időt vesz igénybe, akkor bizonyos esetekben elfogadható lehet, ha időtúllépés miatt megszakad a kapcsolat, és a válasz részleges adatokat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="515cb-138">If one or more service calls takes too long, it may be acceptable to timeout and return a partial set of data.</span></span> <span data-ttu-id="515cb-139">Gondolja át, hogyan fogja kezelni az alkalmazása ezt a forgatókönyvet.</span><span class="sxs-lookup"><span data-stu-id="515cb-139">Consider how your application will handle this scenario.</span></span>
- <span data-ttu-id="515cb-140">Alkalmazzon aszinkron I/O-t, hogy a háttérrendszer késése ne legyen hatással az alkalmazás teljesítményére.</span><span class="sxs-lookup"><span data-stu-id="515cb-140">Use asynchronous I/O to ensure that a delay at the backend doesn't cause performance issues in the application.</span></span>
- <span data-ttu-id="515cb-141">Implementáljon elosztott nyomkövetést korrelációs azonosítók használatával az egyes hívások nyomon követéséhez.</span><span class="sxs-lookup"><span data-stu-id="515cb-141">Implement distributed tracing using correlation IDs to track each individual call.</span></span>
- <span data-ttu-id="515cb-142">Monitorozza a kérésmetrikákat és a válaszok méretét.</span><span class="sxs-lookup"><span data-stu-id="515cb-142">Monitor request metrics and response sizes.</span></span>
- <span data-ttu-id="515cb-143">Fontolja meg gyorsítótárazott adatok visszaadását a feladatátvételi stratégia részeként a hibák kezelésére.</span><span class="sxs-lookup"><span data-stu-id="515cb-143">Consider returning cached data as a failover strategy to handle failures.</span></span>
- <span data-ttu-id="515cb-144">Megteheti, hogy nem az átjáróba építi be az összesítést, hanem egy összesítési szolgáltatást helyez el az átjáró mögött.</span><span class="sxs-lookup"><span data-stu-id="515cb-144">Instead of building aggregation into the gateway, consider placing an aggregation service behind the gateway.</span></span> <span data-ttu-id="515cb-145">A kérések összesítésének valószínűleg eltérő erőforrásigénye van, mint az átjáró többi szolgáltatásának, ami befolyásolhatja az átjáró útválasztási és kiszervezési funkcióit.</span><span class="sxs-lookup"><span data-stu-id="515cb-145">Request aggregation will likely have different resource requirements than other services in the gateway and may impact the gateway's routing and offloading functionality.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="515cb-146">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="515cb-146">When to use this pattern</span></span>

<span data-ttu-id="515cb-147">Használja ezt a mintát, ha:</span><span class="sxs-lookup"><span data-stu-id="515cb-147">Use this pattern when:</span></span>

- <span data-ttu-id="515cb-148">Az ügyfélnek több háttérszolgáltatással kell kommunikálnia az adott művelet végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="515cb-148">A client needs to communicate with multiple backend services to perform an operation.</span></span>
- <span data-ttu-id="515cb-149">Előfordulhat, hogy az ügyfél csak nagy késésű hálózatokhoz, például mobilhálózatokhoz fér hozzá.</span><span class="sxs-lookup"><span data-stu-id="515cb-149">The client may use networks with significant latency, such as cellular networks.</span></span>

<span data-ttu-id="515cb-150">Nem érdemes ezt a mintát használni, ha:</span><span class="sxs-lookup"><span data-stu-id="515cb-150">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="515cb-151">Csökkenteni szeretné az ügyfél és egy szolgáltatás közötti hívások számát több műveletben.</span><span class="sxs-lookup"><span data-stu-id="515cb-151">You want to reduce the number of calls between a client and a single service across multiple operations.</span></span> <span data-ttu-id="515cb-152">Ebben az esetben fontolja meg egy kötegelt művelet hozzáadását a szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="515cb-152">In that scenario, it may be better to add a batch operation to the service.</span></span>
- <span data-ttu-id="515cb-153">Az ügyfél vagy az alkalmazás a háttérszolgáltatások közelében található, és a késés elhanyagolható.</span><span class="sxs-lookup"><span data-stu-id="515cb-153">The client or application is located near the backend services and latency is not a significant factor.</span></span>

## <a name="example"></a><span data-ttu-id="515cb-154">Példa</span><span class="sxs-lookup"><span data-stu-id="515cb-154">Example</span></span>

<span data-ttu-id="515cb-155">A következő példa azt mutatja be, hogyan hozhat létre egyszerű átjáró-összesítési NGINX szolgáltatást a Lua használatával.</span><span class="sxs-lookup"><span data-stu-id="515cb-155">The following example illustrates how to create a simple a gateway aggregation NGINX service using Lua.</span></span>

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

## <a name="related-guidance"></a><span data-ttu-id="515cb-156">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="515cb-156">Related guidance</span></span>

- [<span data-ttu-id="515cb-157">Háttérrendszerek és előtérrendszerek minta</span><span class="sxs-lookup"><span data-stu-id="515cb-157">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="515cb-158">Átjárókiürítés minta</span><span class="sxs-lookup"><span data-stu-id="515cb-158">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
- [<span data-ttu-id="515cb-159">Átjáró-útválasztási minta</span><span class="sxs-lookup"><span data-stu-id="515cb-159">Gateway Routing pattern</span></span>](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md