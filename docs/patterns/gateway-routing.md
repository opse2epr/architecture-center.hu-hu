---
title: Átjáró útválasztás minta
description: Útvonal kérelmek több szolgáltatások használatával egy végpontot.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 53239b23cfd98fad1edc38ca37c2274d5a9d7a0f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="81a40-103">Átjáró útválasztás minta</span><span class="sxs-lookup"><span data-stu-id="81a40-103">Gateway Routing pattern</span></span>

<span data-ttu-id="81a40-104">Útvonal kérelmek több szolgáltatások használatával egy végpontot.</span><span class="sxs-lookup"><span data-stu-id="81a40-104">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="81a40-105">Ebben a mintában akkor hasznos, ha egy végpontot, és az útvonalat a megfelelő szolgáltatásra, a kérés alapján több szolgáltatásokat nyújtó kíván.</span><span class="sxs-lookup"><span data-stu-id="81a40-105">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="81a40-106">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="81a40-106">Context and problem</span></span>

<span data-ttu-id="81a40-107">Ha egy ügyfél több szolgáltatásait, egy külön végpont minden egyes szolgáltatás beállítását, valamint hogy az ügyfél-végpontok kezelése kihívást jelenthet.</span><span class="sxs-lookup"><span data-stu-id="81a40-107">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="81a40-108">Például egy e-kereskedelmi alkalmazás előfordulhat, hogy nevünkben szolgáltatásokat például keresés, értékelést, bevásárlókocsiból, vegye ki és rendelési előzményeit.</span><span class="sxs-lookup"><span data-stu-id="81a40-108">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="81a40-109">Minden szolgáltatás van egy másik API, az ügyfél együtt kell működnie, és az ügyfél kell tudnia végpontok a szolgáltatásokhoz való csatlakozás érdekében.</span><span class="sxs-lookup"><span data-stu-id="81a40-109">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="81a40-110">Ha módosítja egy API-t, az ügyfél is frissíteni kell.</span><span class="sxs-lookup"><span data-stu-id="81a40-110">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="81a40-111">Ha két vagy több különálló szolgáltatás refactor, egy szolgáltatás, a kód a szolgáltatás és az ügyfél is módosítania kell.</span><span class="sxs-lookup"><span data-stu-id="81a40-111">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="81a40-112">Megoldás</span><span class="sxs-lookup"><span data-stu-id="81a40-112">Solution</span></span>

<span data-ttu-id="81a40-113">Egy átjáró alkalmazások, szolgáltatások vagy a központi telepítések elé helyezze el.</span><span class="sxs-lookup"><span data-stu-id="81a40-113">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="81a40-114">Alkalmazás réteg 7 útválasztás használatával továbbítja a kérelmet a megfelelő példányt.</span><span class="sxs-lookup"><span data-stu-id="81a40-114">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="81a40-115">Az ebben a mintában az ügyfélalkalmazás csak kell tudni kommunikálni egy végpontot.</span><span class="sxs-lookup"><span data-stu-id="81a40-115">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="81a40-116">A szolgáltatás konszolidált vagy kiválasztott, az ügyfél nem feltétlenül igényel frissítése.</span><span class="sxs-lookup"><span data-stu-id="81a40-116">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="81a40-117">Az átjáró, és csak a útválasztási kérelmet benyújtó továbbra is azt.</span><span class="sxs-lookup"><span data-stu-id="81a40-117">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="81a40-118">Átjáró emellett lehetővé teszi a háttér-szolgáltatást, az ügyfelek így ügyfélhívásokat egyszerű tarthatja engedélyezze a háttér-szolgáltatások, az átjáró mögötti változásainak absztrakt.</span><span class="sxs-lookup"><span data-stu-id="81a40-118">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="81a40-119">Ügyfél hívások átirányítható bármilyen szolgáltatást vagy a szolgáltatások kell kezelni a várható ügyfél viselkedését, így hozzáadásához ossza fel, és az átjáró mögötti szolgáltatások biztosítására átrendezéséhez az ügyfél módosítása nélkül.</span><span class="sxs-lookup"><span data-stu-id="81a40-119">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![](./_images/gateway-routing.png)
 
<span data-ttu-id="81a40-120">Ebben a mintában az üzembe helyezéssel is hozzájárulhat a azáltal, hogy hogyan frissítések már megkezdődött a felhasználók kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="81a40-120">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="81a40-121">Ha a szolgáltatás egy új verziója van telepítve, azt a meglévő verziót párhuzamosan is telepíthető.</span><span class="sxs-lookup"><span data-stu-id="81a40-121">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="81a40-122">Útválasztás segítségével szabályozhatja, hogy melyik verzió a szolgáltatás nem jelenik meg az ügyfelek, felkínálva a rugalmasságot különböző kiadás stratégiák használata növekményes, hogy párhuzamosan, vagy végezze el a frissítések végrehajtása.</span><span class="sxs-lookup"><span data-stu-id="81a40-122">Routing let you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="81a40-123">Probléma merül fel az új szolgáltatás telepítése után felderített gyorsan beállításai állíthatók vissza azáltal, hogy olyan konfigurációs módosítást az átjárón, az ügyfelek befolyásolása nélkül.</span><span class="sxs-lookup"><span data-stu-id="81a40-123">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="81a40-124">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="81a40-124">Issues and considerations</span></span>

- <span data-ttu-id="81a40-125">Az átjárószolgáltatás vezethet be a hibaérzékeny pontok kialakulását.</span><span class="sxs-lookup"><span data-stu-id="81a40-125">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="81a40-126">Győződjön meg arról, a rendelkezésre állási követelményeinek megfelelően tervezték.</span><span class="sxs-lookup"><span data-stu-id="81a40-126">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="81a40-127">Érdemes lehet rugalmasság és a hibatűrés tolerancia képességek végrehajtása során.</span><span class="sxs-lookup"><span data-stu-id="81a40-127">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="81a40-128">Az átjárószolgáltatás vezethet be a szűk keresztmetszetek.</span><span class="sxs-lookup"><span data-stu-id="81a40-128">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="81a40-129">Győződjön meg arról, az átjáró rendelkezik a megfelelő teljesítmény terhelés kezelésére, és könnyedén méretezhető az növekedési igényeinek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="81a40-129">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="81a40-130">Hajtsa végre a terhelés tesztelése az átjárón nem vezetnek be kaszkádolt probléma van a szolgáltatások biztosításához.</span><span class="sxs-lookup"><span data-stu-id="81a40-130">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="81a40-131">7. szint átjáró útválasztási.</span><span class="sxs-lookup"><span data-stu-id="81a40-131">Gateway routing is level 7.</span></span> <span data-ttu-id="81a40-132">Ez alapulhat a IP-, port, fejléc vagy URL-CÍMÉT.</span><span class="sxs-lookup"><span data-stu-id="81a40-132">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="81a40-133">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="81a40-133">When to use this pattern</span></span>

<span data-ttu-id="81a40-134">Ez mintát, mikor használja:</span><span class="sxs-lookup"><span data-stu-id="81a40-134">Use this pattern when:</span></span>

- <span data-ttu-id="81a40-135">Egy ügyfél hozzáfér egy átjáró mögötti több szolgáltatásait kell.</span><span class="sxs-lookup"><span data-stu-id="81a40-135">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="81a40-136">Egyszerűbbé teheti az ügyfélalkalmazások egyetlen végpont használatával kívánja.</span><span class="sxs-lookup"><span data-stu-id="81a40-136">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="81a40-137">Belső virtuális végpontok, például a fürt virtuális IP-címek egy virtuális gép portját az ilyen külsőleg megcímezhető végpontok érkező kéréseket továbbítani kell.</span><span class="sxs-lookup"><span data-stu-id="81a40-137">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="81a40-138">Ez a minta nem lehet megfelelő, ha csak egy vagy két szolgáltatást használó egyszerű alkalmazással.</span><span class="sxs-lookup"><span data-stu-id="81a40-138">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="81a40-139">Példa</span><span class="sxs-lookup"><span data-stu-id="81a40-139">Example</span></span>

<span data-ttu-id="81a40-140">Az útválasztó Nginx használ, a következő: egy egyszerű példa konfigurációs fájlt a kiszolgáló, amely az alkalmazások különböző virtuális könyvtárak különböző gépek hátsó végén levő kérelmek</span><span class="sxs-lookup"><span data-stu-id="81a40-140">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

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

## <a name="related-guidance"></a><span data-ttu-id="81a40-141">Kapcsolódó útmutató</span><span class="sxs-lookup"><span data-stu-id="81a40-141">Related guidance</span></span>

- [<span data-ttu-id="81a40-142">Háttérkiszolgálókon Frontends minta</span><span class="sxs-lookup"><span data-stu-id="81a40-142">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="81a40-143">Átjáró összesítési minta</span><span class="sxs-lookup"><span data-stu-id="81a40-143">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="81a40-144">Átjáró Offloading minta</span><span class="sxs-lookup"><span data-stu-id="81a40-144">Gateway Offloading pattern</span></span>](./gateway-offloading.md)



