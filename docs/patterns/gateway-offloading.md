---
title: Átjáró Offloading minta
description: Megosztott vagy speciális szolgáltatás funkcióit egy átjáró proxyként kiürítése.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540009"
---
# <a name="gateway-offloading-pattern"></a><span data-ttu-id="31cc8-103">Átjáró Offloading minta</span><span class="sxs-lookup"><span data-stu-id="31cc8-103">Gateway Offloading pattern</span></span>

<span data-ttu-id="31cc8-104">Megosztott vagy speciális szolgáltatás funkcióit egy átjáró proxyként kiürítése.</span><span class="sxs-lookup"><span data-stu-id="31cc8-104">Offload shared or specialized service functionality to a gateway proxy.</span></span> <span data-ttu-id="31cc8-105">Ebben a mintában alkalmazásfejlesztés egyszerűsítheti megosztott szolgáltatás funkciókat, például az SSL-tanúsítványok tér át az alkalmazást az átjáró többi részével.</span><span class="sxs-lookup"><span data-stu-id="31cc8-105">This pattern can simplify application development by moving shared service functionality, such as the use of SSL certificates, from other parts of the application into the gateway.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="31cc8-106">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="31cc8-106">Context and problem</span></span>

<span data-ttu-id="31cc8-107">Néhány gyakran használják több szolgáltatásban, és ezek a szolgáltatások konfigurációs, felügyeleti és karbantartási szükséges.</span><span class="sxs-lookup"><span data-stu-id="31cc8-107">Some features are commonly used across multiple services, and these features require configuration, management, and maintenance.</span></span> <span data-ttu-id="31cc8-108">Egy megosztott vagy speciális szolgáltatás, minden alkalmazás központi telepítése terjesztett növeli az adminisztrációs terhelést, és növeli az elágazás a központi telepítési hiba.</span><span class="sxs-lookup"><span data-stu-id="31cc8-108">A shared or specialized service that is distributed with every application deployment increases the administrative overhead and increases the likelihood of deployment error.</span></span> <span data-ttu-id="31cc8-109">A megosztott szolgáltatás frissítéseit is, hogy az összes szolgáltatásban kell telepíteni.</span><span class="sxs-lookup"><span data-stu-id="31cc8-109">Any updates to a shared feature must be deployed across all services that share that feature.</span></span>

<span data-ttu-id="31cc8-110">Megfelelően kezeli a biztonsági problémák (jogkivonat érvényesítésére, titkosítás, SSL-tanúsítványok kezelését) és egyéb összetett feladatokat megkövetelheti, hogy a csoport tagjai magas speciális ismeretek rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="31cc8-110">Properly handling security issues (token validation, encryption, SSL certificate management) and other complex tasks can require team members to have highly specialized skills.</span></span> <span data-ttu-id="31cc8-111">Például egy alkalmazás által igényelt tanúsítvány kell kell konfigurált és telepített összes alkalmazás-példányokon.</span><span class="sxs-lookup"><span data-stu-id="31cc8-111">For example, a certificate needed by an application must be configured and deployed on all application instances.</span></span> <span data-ttu-id="31cc8-112">Az egyes új központi telepítéséhez annak érdekében, hogy azt nem jár le a tanúsítványt kell kezelni.</span><span class="sxs-lookup"><span data-stu-id="31cc8-112">With each new deployment, the certificate must be managed to ensure that it does not expire.</span></span> <span data-ttu-id="31cc8-113">Bármely általános tanúsítvány lejárati dátuma frissíteni kell, tesztelése és ellenőrzése minden alkalmazás üzembe helyezés.</span><span class="sxs-lookup"><span data-stu-id="31cc8-113">Any common certificate that is due to expire must be updated, tested, and verified on every application deployment.</span></span>

<span data-ttu-id="31cc8-114">Más közös szolgáltatásokon, például a hitelesítési, engedélyezési, naplózási, figyelés, vagy [sávszélesség-szabályozás](./throttling.md) megvalósítása és kezelése a központi telepítések számos különböző nehéz lehet.</span><span class="sxs-lookup"><span data-stu-id="31cc8-114">Other common services such as authentication, authorization, logging, monitoring, or [throttling](./throttling.md) can be difficult to implement and manage across a large number of deployments.</span></span> <span data-ttu-id="31cc8-115">Jobb megoldás lehet vonják össze a funkciót, az ilyen típusú munkaterhet és a hibák esélyét csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="31cc8-115">It may be better to consolidate this type of functionality, in order to reduce overhead and the chance of errors.</span></span>

## <a name="solution"></a><span data-ttu-id="31cc8-116">Megoldás</span><span class="sxs-lookup"><span data-stu-id="31cc8-116">Solution</span></span>

<span data-ttu-id="31cc8-117">Néhány funkció kiszervezése be egy API-átjárót, különösen több területet vonatkozik, például a tanúsítványkezelés, hitelesítést, SSL lezárást, figyelés, protokollfordításhoz, és sávszélesség-szabályozás.</span><span class="sxs-lookup"><span data-stu-id="31cc8-117">Offload some features into an API gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.</span></span> 

<span data-ttu-id="31cc8-118">Az alábbi ábrán egy API-átjáró, amely megszakítja bejövő SSL-kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="31cc8-118">The following diagram shows an API gateway that terminates inbound SSL connections.</span></span> <span data-ttu-id="31cc8-119">Az eredeti kérelmezőnek bármely HTTP-kiszolgáló előtt az API-átjáró nevében adatokat kér.</span><span class="sxs-lookup"><span data-stu-id="31cc8-119">It requests data on behalf of the original requestor from any HTTP server upstream of the API gateway.</span></span>

 ![](./_images/gateway-offload.png)
 
<span data-ttu-id="31cc8-120">Ez a minta a következő előnyöket nyújtja:</span><span class="sxs-lookup"><span data-stu-id="31cc8-120">Benefits of this pattern include:</span></span>

- <span data-ttu-id="31cc8-121">Leegyszerűsítheti a szolgáltatások fejlesztéséhez kell terjeszteni, és a támogató erőforrások, például a webkiszolgáló-tanúsítványok és a biztonságos webhely konfigurációja karbantartása eltávolításával.</span><span class="sxs-lookup"><span data-stu-id="31cc8-121">Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites.</span></span> <span data-ttu-id="31cc8-122">Egyszerűbb konfiguráció egyszerűbb kezelés és a méretezhetőség eredményez, és így egyszerűbb frissítése.</span><span class="sxs-lookup"><span data-stu-id="31cc8-122">Simpler configuration results in easier management and scalability and makes service upgrades simpler.</span></span>

- <span data-ttu-id="31cc8-123">Dedikált csoportok megvalósításához speciális szakértelem, például a biztonsági igénylő szolgáltatások engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="31cc8-123">Allow dedicated teams to implement features that require specialized expertise, such as security.</span></span> <span data-ttu-id="31cc8-124">Ez lehetővé teszi, hogy az alapvető csapat az alkalmazás funkciói, hagyja a speciális, de több területet problémák a megfelelő szakértőknek összpontosíthat.</span><span class="sxs-lookup"><span data-stu-id="31cc8-124">This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.</span></span>

- <span data-ttu-id="31cc8-125">Adja meg egyes konzisztencia kérés- és naplózás és figyelés.</span><span class="sxs-lookup"><span data-stu-id="31cc8-125">Provide some consistency for request and response logging and monitoring.</span></span> <span data-ttu-id="31cc8-126">Akkor is, ha a szolgáltatás nem megfelelően tagolva, az átjáró beállítható úgy, hogy a figyelés és naplózás minimális szintjét.</span><span class="sxs-lookup"><span data-stu-id="31cc8-126">Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="31cc8-127">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="31cc8-127">Issues and considerations</span></span>

- <span data-ttu-id="31cc8-128">Győződjön meg arról az API-átjáró magas rendelkezésre állású és rugalmas hiba esetén.</span><span class="sxs-lookup"><span data-stu-id="31cc8-128">Ensure the API gateway is highly available and resilient to failure.</span></span> <span data-ttu-id="31cc8-129">Kerülje a hibaérzékeny pontokat által az API-átjáró több példányát futtatni.</span><span class="sxs-lookup"><span data-stu-id="31cc8-129">Avoid single points of failure by running multiple instances of your API gateway.</span></span> 
- <span data-ttu-id="31cc8-130">Győződjön meg arról, az átjáró a kapacitás készült, és a méretezésről az alkalmazás- és végpontok követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="31cc8-130">Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints.</span></span> <span data-ttu-id="31cc8-131">Győződjön meg arról, hogy az átjáró nem keresztmetszetet jelenthet az alkalmazáshoz, és megfelelően méretezhető.</span><span class="sxs-lookup"><span data-stu-id="31cc8-131">Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.</span></span>
- <span data-ttu-id="31cc8-132">A teljes alkalmazás, például biztonsági vagy átviteli által használt funkciók csak kiürítése.</span><span class="sxs-lookup"><span data-stu-id="31cc8-132">Only offload features that are used by the entire application, such as security or data transfer.</span></span>
- <span data-ttu-id="31cc8-133">Az API-átjárónak soha nem kell kiszervezett üzleti logikát.</span><span class="sxs-lookup"><span data-stu-id="31cc8-133">Business logic should never be offloaded to the API gateway.</span></span> 
- <span data-ttu-id="31cc8-134">Ha tranzakciók nyomon van szüksége, fontolja meg, generálásához. korrelációs azonosító naplózási célokra.</span><span class="sxs-lookup"><span data-stu-id="31cc8-134">If you need to track transactions, consider generating correlation IDs for logging purposes.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="31cc8-135">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="31cc8-135">When to use this pattern</span></span>

<span data-ttu-id="31cc8-136">Ez mintát, mikor használja:</span><span class="sxs-lookup"><span data-stu-id="31cc8-136">Use this pattern when:</span></span>

- <span data-ttu-id="31cc8-137">Az alkalmazás központi telepítése egy megosztott szempont, például az SSL-tanúsítványok és titkosítás rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="31cc8-137">An application deployment has a shared concern such as SSL certificates or encryption.</span></span>
- <span data-ttu-id="31cc8-138">Ez a szolgáltatás által közösen alkalmazások központi telepítéseit, amelyek különböző erőforrás-követelményei, például a memória-erőforrásokat, a tárolási kapacitás vagy a hálózati kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="31cc8-138">A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.</span></span>
- <span data-ttu-id="31cc8-139">Helyezze át az ezzel kapcsolatos problémák, például a hálózati biztonság, a sávszélesség-szabályozás és a más hálózati határ észrevételeivel konkrétabb team kívánja.</span><span class="sxs-lookup"><span data-stu-id="31cc8-139">You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.</span></span>

<span data-ttu-id="31cc8-140">Előfordulhat, hogy ez a minta nem megfelelő, ha a kapcsoló szolgáltatásban okozna.</span><span class="sxs-lookup"><span data-stu-id="31cc8-140">This pattern may not be suitable if it introduces coupling across services.</span></span>

## <a name="example"></a><span data-ttu-id="31cc8-141">Példa</span><span class="sxs-lookup"><span data-stu-id="31cc8-141">Example</span></span>

<span data-ttu-id="31cc8-142">A Nginx-t az SSL kiszervezési akkor, a következő konfigurációs bejövő SSL-kapcsolat leállítása, majd továbbítja a három fölérendelt HTTP-kiszolgálóhoz csatlakozni.</span><span class="sxs-lookup"><span data-stu-id="31cc8-142">Using Nginx as the SSL offload appliance, the following configuration terminates an inbound SSL connection and distributes the connection to one of three upstream HTTP servers.</span></span>

```
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a><span data-ttu-id="31cc8-143">Kapcsolódó útmutató</span><span class="sxs-lookup"><span data-stu-id="31cc8-143">Related guidance</span></span>

- [<span data-ttu-id="31cc8-144">Háttérkiszolgálókon Frontends minta</span><span class="sxs-lookup"><span data-stu-id="31cc8-144">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="31cc8-145">Átjáró összesítési minta</span><span class="sxs-lookup"><span data-stu-id="31cc8-145">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="31cc8-146">Átjáró útválasztás minta</span><span class="sxs-lookup"><span data-stu-id="31cc8-146">Gateway Routing pattern</span></span>](./gateway-routing.md)

