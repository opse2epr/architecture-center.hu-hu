---
title: Tervezési minták a mikroszolgáltatások
description: Tervezési minták, amely hatékony mikroszolgáltatási architektúra megvalósítása.
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 8cdbf95c753770910fa4a94384c9809db9fda746
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298834"
---
# <a name="design-patterns-for-microservices"></a><span data-ttu-id="956c6-103">Tervezési minták a mikroszolgáltatások</span><span class="sxs-lookup"><span data-stu-id="956c6-103">Design patterns for microservices</span></span>

<span data-ttu-id="956c6-104">A mikroszolgáltatások célja decomposing kis autonóm servicesbe, egymástól függetlenül telepíthető az alkalmazás által a alkalmazás kiadások, a sebesség növelése érdekében.</span><span class="sxs-lookup"><span data-stu-id="956c6-104">The goal of microservices is to increase the velocity of application releases, by decomposing the application into small autonomous services that can be deployed independently.</span></span> <span data-ttu-id="956c6-105">A mikroszolgáltatási architektúra is elérhetővé teszi a áttekinthet néhány problémát.</span><span class="sxs-lookup"><span data-stu-id="956c6-105">A microservices architecture also brings some challenges.</span></span> <span data-ttu-id="956c6-106">Az itt látható tervezési minták segítségével, ezek a kihívások megoldásához.</span><span class="sxs-lookup"><span data-stu-id="956c6-106">The design patterns shown here can help mitigate these challenges.</span></span>

![A Mikroszolgáltatások tervezési minták](../images/microservices-patterns.png)

<span data-ttu-id="956c6-108">[**Nagykövet** ](../../patterns/ambassador.md) kiszervezheti a gyakori ügyfélkapcsolati feladatok, például a figyelés, naplózás, Útválasztás és biztonság (pl. TLS) nyelven használható független módon.</span><span class="sxs-lookup"><span data-stu-id="956c6-108">[**Ambassador**](../../patterns/ambassador.md) can be used to offload common client connectivity tasks such as monitoring, logging, routing, and security (such as TLS) in a language agnostic way.</span></span> <span data-ttu-id="956c6-109">Nagykövet-szolgáltatások gyakran települnek oldalkocsiként (lásd alább).</span><span class="sxs-lookup"><span data-stu-id="956c6-109">Ambassador services are often deployed as a sidecar (see below).</span></span>

<span data-ttu-id="956c6-110">[**Sérülésgátló réteg** ](../../patterns/anti-corruption-layer.md) valósít meg egy a adapterréteget új és örökölt alkalmazások között, annak érdekében, hogy egy új alkalmazás kialakítása nem korlátozódik a régi rendszerektől való függőségek.</span><span class="sxs-lookup"><span data-stu-id="956c6-110">[**Anti-corruption layer**](../../patterns/anti-corruption-layer.md) implements a façade between new and legacy applications, to ensure that the design of a new application is not limited by dependencies on legacy systems.</span></span>

<span data-ttu-id="956c6-111">[**Háttérrendszerek és Előtérrendszerek** ](../../patterns/backends-for-frontends.md) hoz létre a különböző típusú ügyfelek, például az asztali és mobil külön háttérrendszerekhez.</span><span class="sxs-lookup"><span data-stu-id="956c6-111">[**Backends for Frontends**](../../patterns/backends-for-frontends.md) creates separate backend services for different types of clients, such as desktop and mobile.</span></span> <span data-ttu-id="956c6-112">Ezzel a módszerrel az egyetlen háttérszolgáltatás ütköző igényeit kielégítő különböző ügyfél kezeléséhez nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="956c6-112">That way, a single backend service doesn’t need to handle the conflicting requirements of various client types.</span></span> <span data-ttu-id="956c6-113">Ez a minta segíthet használni, hogy mindegyik mikroszolgáltatás egyszerű, elválasztó ügyfelekre vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="956c6-113">This pattern can help keep each microservice simple, by separating client-specific concerns.</span></span>

<span data-ttu-id="956c6-114">[**Válaszfal** ](../../patterns/bulkhead.md) különíti el a kritikus fontosságú erőforrások, például kapcsolatkészletben, a memória és a Processzor, a számítási feladat vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="956c6-114">[**Bulkhead**](../../patterns/bulkhead.md) isolates critical resources, such as connection pool, memory, and CPU, for each workload or service.</span></span> <span data-ttu-id="956c6-115">Válaszfalak használatával egy egyetlen számítási feladat (vagy szolgáltatás) nem lehet felhasználni mások starving erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="956c6-115">By using bulkheads, a single workload (or service) can’t consume all of the resources, starving others.</span></span> <span data-ttu-id="956c6-116">Ezt a mintát növeli a rugalmasságot, a rendszer megakadályozza, hogy egy szolgáltatás által okozott hiba egész hibasorozatot indíthat.</span><span class="sxs-lookup"><span data-stu-id="956c6-116">This pattern increases the resiliency of the system by preventing cascading failures caused by one service.</span></span>

<span data-ttu-id="956c6-117">[**Átjáró-összesítési** ](../../patterns/gateway-aggregation.md) összesíti az egyes mikroszolgáltatásokat több kérelmeket egyetlen kérelemmé, csökkentve a fogyasztók és a szolgáltatások közötti forgalmat.</span><span class="sxs-lookup"><span data-stu-id="956c6-117">[**Gateway Aggregation**](../../patterns/gateway-aggregation.md) aggregates requests to multiple individual microservices into a single request, reducing chattiness between consumers and services.</span></span>

<span data-ttu-id="956c6-118">[**Átjáró Átjárókiürítés** ](../../patterns/gateway-offloading.md) lehetővé teszi, hogy mindegyik mikroszolgáltatás kiszervezheti a megosztott szolgáltatás funkciókat, például SSL-tanúsítványokat, API-átjáró használatát.</span><span class="sxs-lookup"><span data-stu-id="956c6-118">[**Gateway Offloading**](../../patterns/gateway-offloading.md) enables each microservice to offload shared service functionality, such as the use of SSL certificates, to an API gateway.</span></span>

<span data-ttu-id="956c6-119">[**Átjáró útválasztási** ](../../patterns/gateway-routing.md) irányítja a kérelmeket, egyetlen végpont használatával több mikroszolgáltatásból, hogy a felhasználók számos különböző végpontok kezelése nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="956c6-119">[**Gateway Routing**](../../patterns/gateway-routing.md) routes requests to multiple microservices using a single endpoint, so that consumers don't need to manage many separate endpoints.</span></span>

<span data-ttu-id="956c6-120">[**Az oldalkocsi** ](../../patterns/sidecar.md) helyez üzembe egy alkalmazás egy külön tároló vagy folyamat elkülönítést és beágyazást biztosíthat biztosít segítő összetevőit.</span><span class="sxs-lookup"><span data-stu-id="956c6-120">[**Sidecar**](../../patterns/sidecar.md) deploys helper components of an application as a separate container or process to provide isolation and encapsulation.</span></span>

<span data-ttu-id="956c6-121">[**Leépítő** ](../../patterns/strangler.md) támogatja a növekményes újrabontás egy alkalmazás oly módon, hogy egyes funkciódarabokat új szolgáltatásokkal.</span><span class="sxs-lookup"><span data-stu-id="956c6-121">[**Strangler**](../../patterns/strangler.md) supports incremental refactoring of an application, by gradually replacing specific pieces of functionality with new services.</span></span>

<span data-ttu-id="956c6-122">A felhőkialakítási minták az Azure Architecture Centert a kész katalógust, lásd: [tervezési minták Felhőkhöz](../../patterns/index.md).</span><span class="sxs-lookup"><span data-stu-id="956c6-122">For the complete catalog of cloud design patterns on the Azure Architecture Center, see [Cloud Design Patterns](../../patterns/index.md).</span></span>
