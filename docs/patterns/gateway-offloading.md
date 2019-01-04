---
title: Átjárókiszervezési minta
titleSuffix: Cloud Design Patterns
description: A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 50af3d8593279986ed6efee55667187424c18e56
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010211"
---
# <a name="gateway-offloading-pattern"></a><span data-ttu-id="c8d5c-104">Átjárókiszervezési minta</span><span class="sxs-lookup"><span data-stu-id="c8d5c-104">Gateway Offloading pattern</span></span>

<span data-ttu-id="c8d5c-105">A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-105">Offload shared or specialized service functionality to a gateway proxy.</span></span> <span data-ttu-id="c8d5c-106">Ez a minta leegyszerűsítheti az alkalmazásfejlesztést a megosztott szolgáltatásfunkciók, például az SSL-tanúsítványok használatának az átjáróba való áthelyezésével.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-106">This pattern can simplify application development by moving shared service functionality, such as the use of SSL certificates, from other parts of the application into the gateway.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c8d5c-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="c8d5c-107">Context and problem</span></span>

<span data-ttu-id="c8d5c-108">Bizonyos funkciókat több szolgáltatás is használ, és ezek a funkciók beállítást, felügyeletet és karbantartást igényelnek.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-108">Some features are commonly used across multiple services, and these features require configuration, management, and maintenance.</span></span> <span data-ttu-id="c8d5c-109">A minden alkalmazástelepítéskor elosztott megosztott vagy speciális szolgáltatások növelik az adminisztrációs terhelést és a telepítési hibák esélyét.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-109">A shared or specialized service that is distributed with every application deployment increases the administrative overhead and increases the likelihood of deployment error.</span></span> <span data-ttu-id="c8d5c-110">A megosztott funkciók frissítéseit a funkciót használó összes szolgáltatásban telepíteni kell.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-110">Any updates to a shared feature must be deployed across all services that share that feature.</span></span>

<span data-ttu-id="c8d5c-111">A biztonsági problémák megfelelő kezelése (tokenérvényesítés, titkosítás, SSL-tanúsítványkezelés) és egyéb összetett feladatok elvégzése speciális szaktudást kívánhat meg.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-111">Properly handling security issues (token validation, encryption, SSL certificate management) and other complex tasks can require team members to have highly specialized skills.</span></span> <span data-ttu-id="c8d5c-112">Például az egy alkalmazás számára szükséges tanúsítványt konfigurálni és telepíteni kell az alkalmazás összes példányán.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-112">For example, a certificate needed by an application must be configured and deployed on all application instances.</span></span> <span data-ttu-id="c8d5c-113">Minden új telepítés esetében szükség van a tanúsítvány kezelésére, hogy az ne járjon le.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-113">With each new deployment, the certificate must be managed to ensure that it does not expire.</span></span> <span data-ttu-id="c8d5c-114">Ha közeledik egy közös tanúsítvány lejárati ideje, akkor frissíteni, majd tesztelni és ellenőrizni kell azt minden alkalmazástelepítésen.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-114">Any common certificate that is due to expire must be updated, tested, and verified on every application deployment.</span></span>

<span data-ttu-id="c8d5c-115">Más közös szolgáltatásokat, például a hitelesítést, az engedélyezést, a naplózást, a monitorozást vagy a [szabályozást](./throttling.md) nehéz implementálni és felügyelni nagy számú telepítés esetén.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-115">Other common services such as authentication, authorization, logging, monitoring, or [throttling](./throttling.md) can be difficult to implement and manage across a large number of deployments.</span></span> <span data-ttu-id="c8d5c-116">Érdemes lehet konszolidálni az ilyen típusú funkciókat a terhelés és a hibák esélyének csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-116">It may be better to consolidate this type of functionality, in order to reduce overhead and the chance of errors.</span></span>

## <a name="solution"></a><span data-ttu-id="c8d5c-117">Megoldás</span><span class="sxs-lookup"><span data-stu-id="c8d5c-117">Solution</span></span>

<span data-ttu-id="c8d5c-118">Szervezzen ki egyes funkciókat egy API-átjáróba, különösen az általános feladatokat, mint a tanúsítványkezelés, a hitelesítés, az SSL-lezárás, a monitorozás, a protokollfordítás, valamint a szabályozás.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-118">Offload some features into an API gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.</span></span>

<span data-ttu-id="c8d5c-119">A következő ábrán egy API-átjáró látható, amely lezárja a bejövő SSL-kapcsolatokat.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-119">The following diagram shows an API gateway that terminates inbound SSL connections.</span></span> <span data-ttu-id="c8d5c-120">Az eredeti kérelmező nevében kér adatokat bármely, az API-átjárónál felsőbb rétegbeli HTTP-kiszolgálótól.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-120">It requests data on behalf of the original requestor from any HTTP server upstream of the API gateway.</span></span>

 ![Az átjáró-kiürítés minta ábrája](./_images/gateway-offload.png)

<span data-ttu-id="c8d5c-122">A minta használata többek között a következő előnyökkel jár:</span><span class="sxs-lookup"><span data-stu-id="c8d5c-122">Benefits of this pattern include:</span></span>

- <span data-ttu-id="c8d5c-123">Egyszerűsíti a szolgáltatások fejlesztését, mivel nincs szükség a támogató erőforrások (például a webkiszolgáló-tanúsítványok és a biztonságos webhelyek konfigurációja) elosztására és karbantartására.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-123">Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites.</span></span> <span data-ttu-id="c8d5c-124">Az egyszerűbb konfigurálás könnyebb felügyeletet és skálázhatóságot jelent, és így könnyebb a szolgáltatásfrissítés is.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-124">Simpler configuration results in easier management and scalability and makes service upgrades simpler.</span></span>

- <span data-ttu-id="c8d5c-125">A speciális szakértelmet igénylő funkciók implementálását (például biztonság) bízza külön erre a célra létrehozott csapatokra.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-125">Allow dedicated teams to implement features that require specialized expertise, such as security.</span></span> <span data-ttu-id="c8d5c-126">Így a központi csapat az alkalmazás működésére koncentrálhat, és a több szolgáltatást érintő, de speciális kérdések megoldását a szakértőkre hagyhatja.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-126">This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.</span></span>

- <span data-ttu-id="c8d5c-127">Egységesítse a kérések és válaszok naplózását és monitorozását.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-127">Provide some consistency for request and response logging and monitoring.</span></span> <span data-ttu-id="c8d5c-128">Beállíthatja, hogy az átjáró akkor is végezzen minimális monitorozási és naplózási tevékenységet, ha a szolgáltatás nem lett megfelelően kialakítva.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-128">Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c8d5c-129">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="c8d5c-129">Issues and considerations</span></span>

- <span data-ttu-id="c8d5c-130">Gondoskodjon arról, hogy az API-átjáró magas rendelkezési állású legyen, és ne legyenek rá hatással a meghibásodások.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-130">Ensure the API gateway is highly available and resilient to failure.</span></span> <span data-ttu-id="c8d5c-131">Kerülje el a kritikus meghibásodási pontok kialakulását: futtassa az API-átjáró több példányát.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-131">Avoid single points of failure by running multiple instances of your API gateway.</span></span>
- <span data-ttu-id="c8d5c-132">Gondoskodjon arról, hogy az átjáró megfelel az alkalmazás és a végpontok kapacitási és skálázási követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-132">Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints.</span></span> <span data-ttu-id="c8d5c-133">Gondoskodjon arról, hogy az átjáró ne váljon szűk keresztmetszetté az alkalmazás számára, és megfelelően skálázható legyen.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-133">Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.</span></span>
- <span data-ttu-id="c8d5c-134">Csak olyan funkciókat szervezzen ki, amelyeket a teljes alkalmazás használ, például a biztonsági és az adatátviteli funkciókat.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-134">Only offload features that are used by the entire application, such as security or data transfer.</span></span>
- <span data-ttu-id="c8d5c-135">Üzleti logikát sosem szabad kiszervezni az API-átjáróba.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-135">Business logic should never be offloaded to the API gateway.</span></span>
- <span data-ttu-id="c8d5c-136">Ha nyomon kell követnie a tranzakciókat, érdemes korrelációs azonosítókat létrehozni a naplózáshoz.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-136">If you need to track transactions, consider generating correlation IDs for logging purposes.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c8d5c-137">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="c8d5c-137">When to use this pattern</span></span>

<span data-ttu-id="c8d5c-138">Használja ezt a mintát, ha:</span><span class="sxs-lookup"><span data-stu-id="c8d5c-138">Use this pattern when:</span></span>

- <span data-ttu-id="c8d5c-139">Az alkalmazástelepítésben vannak közös feladatok, például az SSL-tanúsítványok vagy a titkosítás.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-139">An application deployment has a shared concern such as SSL certificates or encryption.</span></span>
- <span data-ttu-id="c8d5c-140">Közös funkciója van több alkalmazástelepítésnek, amelyek erőforrásigényei eltérőek (például memória-erőforrások, tárterület vagy hálózati kapcsolatok).</span><span class="sxs-lookup"><span data-stu-id="c8d5c-140">A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.</span></span>
- <span data-ttu-id="c8d5c-141">Bizonyos feladatokat egy speciális csapatra szeretne átruházni (például hálózati biztonság, szabályozás, hálózathatárokkal kapcsolatos kérdések).</span><span class="sxs-lookup"><span data-stu-id="c8d5c-141">You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.</span></span>

<span data-ttu-id="c8d5c-142">Nem érdemes ezt a mintát használni, ha összekapcsol egyes szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-142">This pattern may not be suitable if it introduces coupling across services.</span></span>

## <a name="example"></a><span data-ttu-id="c8d5c-143">Példa</span><span class="sxs-lookup"><span data-stu-id="c8d5c-143">Example</span></span>

<span data-ttu-id="c8d5c-144">Ha az Nginxet használja SSL-kiszervezési berendezésként, a következő konfiguráció leállítja az összes beérkező SSL-kapcsolatot, és elosztja azt három felső rétegbeli HTTP-kiszolgáló között.</span><span class="sxs-lookup"><span data-stu-id="c8d5c-144">Using Nginx as the SSL offload appliance, the following configuration terminates an inbound SSL connection and distributes the connection to one of three upstream HTTP servers.</span></span>

```console
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

## <a name="related-guidance"></a><span data-ttu-id="c8d5c-145">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="c8d5c-145">Related guidance</span></span>

- [<span data-ttu-id="c8d5c-146">Háttérrendszerek és előtérrendszerek minta</span><span class="sxs-lookup"><span data-stu-id="c8d5c-146">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="c8d5c-147">Átjáróösszesítés minta</span><span class="sxs-lookup"><span data-stu-id="c8d5c-147">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="c8d5c-148">Átjáró-útválasztási minta</span><span class="sxs-lookup"><span data-stu-id="c8d5c-148">Gateway Routing pattern</span></span>](./gateway-routing.md)
