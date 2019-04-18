---
title: A mikroszolgáltatások referenciaimplementációja az Azure Kubernetes Service esetében
description: Ez a referenciaimplementáció a mikroszolgáltatási architektúra ajánlott eljárásait mutatja be
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 17e275e5b5f45233f7467192402cb28fce35c57b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068905"
---
# <a name="designing-a-microservices-architecture"></a><span data-ttu-id="a5a2f-103">Mikroszolgáltatási architektúra tervezése</span><span class="sxs-lookup"><span data-stu-id="a5a2f-103">Designing a microservices architecture</span></span>

<span data-ttu-id="a5a2f-104">A mikroszolgáltatás egy népszerű architekturális stílussá vált a rugalmas, hatékonyan méretezhető, függetlenül üzembe helyezhető és gyorsan fejleszthető felhőalkalmazások létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-104">Microservices have become a popular architectural style for building cloud applications that are resilient, highly scalable, independently deployable, and able to evolve quickly.</span></span> <span data-ttu-id="a5a2f-105">Azonban ahhoz, hogy ez több legyen, mint egy divatos kifejezés, a mikroszolgáltatások esetén új megközelítés szükséges az alkalmazások tervezésekor és létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-105">To be more than just a buzzword, however, microservices require a different approach to designing and building applications.</span></span>

<span data-ttu-id="a5a2f-106">Ezekben a cikkekben megismerjük, hogyan hozhat létre és futtathat egy mikroszolgáltatási architektúrát az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-106">In this set of articles, we explore how to build and run a microservices architecture on Azure.</span></span> <span data-ttu-id="a5a2f-107">A témakörök a következők:</span><span class="sxs-lookup"><span data-stu-id="a5a2f-107">Topics include:</span></span>

- [<span data-ttu-id="a5a2f-108">Szolgáltatások közötti kommunikáció</span><span class="sxs-lookup"><span data-stu-id="a5a2f-108">Interservice communication</span></span>](./interservice-communication.md)
- [<span data-ttu-id="a5a2f-109">API-tervezés</span><span class="sxs-lookup"><span data-stu-id="a5a2f-109">API design</span></span>](./api-design.md)
- [<span data-ttu-id="a5a2f-110">API-átjárók</span><span class="sxs-lookup"><span data-stu-id="a5a2f-110">API gateways</span></span>](./gateway.md)
- [<span data-ttu-id="a5a2f-111">Adatkezelési szempontok</span><span class="sxs-lookup"><span data-stu-id="a5a2f-111">Data considerations</span></span>](./data-considerations.md)
- [<span data-ttu-id="a5a2f-112">Tervezési minták</span><span class="sxs-lookup"><span data-stu-id="a5a2f-112">Design patterns</span></span>](./patterns.md)

## <a name="prerequisites"></a><span data-ttu-id="a5a2f-113">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="a5a2f-113">Prerequisites</span></span>

<span data-ttu-id="a5a2f-114">A cikkek elolvasása előtt a következővel kezdhet:</span><span class="sxs-lookup"><span data-stu-id="a5a2f-114">Before reading these articles, you might start with the following:</span></span>

- <span data-ttu-id="a5a2f-115">[A mikroszolgáltatási architektúrák bemutatása](../introduction.md).</span><span class="sxs-lookup"><span data-stu-id="a5a2f-115">[Introduction to microservices architectures](../introduction.md).</span></span> <span data-ttu-id="a5a2f-116">Megismerheti a mikroszolgáltatások előnyeit és kihívásait, valamint azt, hogy mikor érdemes ezt az architektúrastílust használni.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-116">Understand the benefits and challenges of microservices, and when to use this style of architecture.</span></span>
- <span data-ttu-id="a5a2f-117">[Mikroszolgáltatások modellezése tartományelemzéssel](../model/domain-analysis.md).</span><span class="sxs-lookup"><span data-stu-id="a5a2f-117">[Using domain analysis to model microservices](../model/domain-analysis.md).</span></span> <span data-ttu-id="a5a2f-118">A mikroszolgáltatás-modellezés tartományvezérelt megközelítésének megismerése.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-118">Learn a domain-driven approach to modeling microservices.</span></span>

## <a name="reference-implementation"></a><span data-ttu-id="a5a2f-119">Referenciaimplementáció</span><span class="sxs-lookup"><span data-stu-id="a5a2f-119">Reference implementation</span></span>

<span data-ttu-id="a5a2f-120">A mikroszolgáltatási architektúrák ajánlott eljárásainak illusztrálása érdekében létrehoztunk egy referenciaimplementációt, a Drone Delivery alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-120">To illustrate best practices for a microservices architecture, we created a reference implementation that we call the Drone Delivery application.</span></span> <span data-ttu-id="a5a2f-121">Ez az implementáció az Azure Kubernetes Service (AKS) használatával fut a Kubernetes rendszeren.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-121">This implementation runs on Kubernetes using Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="a5a2f-122">A referenciaimplementációt a [GitHubon][drone-ri] találja meg.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-122">You can find the reference implementation on [GitHub][drone-ri].</span></span>

![A Drone Delivery alkalmazás architektúrája](../images/drone-delivery.png)

## <a name="scenario"></a><span data-ttu-id="a5a2f-124">Forgatókönyv</span><span class="sxs-lookup"><span data-stu-id="a5a2f-124">Scenario</span></span>

<span data-ttu-id="a5a2f-125">A Fabrikam vállalat egy drónos szállítási szolgáltatást indít.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-125">Fabrikam, Inc. is starting a drone delivery service.</span></span> <span data-ttu-id="a5a2f-126">A cég egy drónokból álló flottát kezel.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-126">The company manages a fleet of drone aircraft.</span></span> <span data-ttu-id="a5a2f-127">A vállalkozások regisztrálnak a szolgáltatásban, és a felhasználók kérhetik egy termék drónos kézbesítését.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-127">Businesses register with the service, and users can request a drone to pick up goods for delivery.</span></span> <span data-ttu-id="a5a2f-128">Amikor az ügyfél ütemezi a felvételt, egy háttérrendszer hozzárendel egy drónt, és értesíti a felhasználót a várható szállítási időről.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-128">When a customer schedules a pickup, a backend system assigns a drone and notifies the user with an estimated delivery time.</span></span> <span data-ttu-id="a5a2f-129">Amíg a kézbesítés folyamatban van, az ügyfél nyomon követheti a drón helyét, miközben a becsült érkezési időpont folyamatosan frissül.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-129">While the delivery is in progress, the customer can track the location of the drone, with a continuously updated ETA.</span></span>

<span data-ttu-id="a5a2f-130">Ez a forgatókönyv egy viszonylag összetett tartományt foglal magába.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-130">This scenario involves a fairly complicated domain.</span></span> <span data-ttu-id="a5a2f-131">A vállalat számára problémát jelenthet a drónok ütemezése, a csomagok nyomon követése, a felhasználói fiókok felügyelete, valamint az előzményadatok tárolása és elemzése.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-131">Some of the business concerns include scheduling drones, tracking packages, managing user accounts, and storing and analyzing historical data.</span></span> <span data-ttu-id="a5a2f-132">Ezenkívül a Fabrikam gyorsan piacra szeretne lépni, majd gyors iterációkat kíván végezni, új funkciók és képességek hozzáadásával.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-132">Moreover, Fabrikam wants to get to market quickly and then iterate quickly, adding new functionality and capabilities.</span></span> <span data-ttu-id="a5a2f-133">Az alkalmazásnak felhőszinten kell üzemelnie, magas szolgáltatási szintű célkitűzéssel (service level objective, SLO).</span><span class="sxs-lookup"><span data-stu-id="a5a2f-133">The application needs to operate at cloud scale, with a high service level objective (SLO).</span></span> <span data-ttu-id="a5a2f-134">A Fabrikam emellett arra számít, hogy a rendszer különböző részeinek nagyon eltérő adattárolási és lekérdezési igényei lesznek.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-134">Fabrikam also expects that different parts of the system will have very different requirements for data storage and querying.</span></span> <span data-ttu-id="a5a2f-135">A megfontolandó szempontok alapján a Fabrikam azt a döntést hozta, hogy egy mikroszolgáltatási architektúrát választ a Drone Delivery alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-135">All of these considerations lead Fabrikam to choose a microservices architecture for the Drone Delivery application.</span></span>

> [!NOTE]
> <span data-ttu-id="a5a2f-136">Amennyiben segítségre van szüksége a mikroszolgáltatási architektúra és egyéb architekturális stílusok közötti választáshoz, tekintse meg az [Azure-alkalmazásarchitektúrákhoz készült útmutatót](../../guide/index.md).</span><span class="sxs-lookup"><span data-stu-id="a5a2f-136">For help in choosing between a microservices architecture and other architectural styles, see the [Azure Application Architecture Guide](../../guide/index.md).</span></span>

<span data-ttu-id="a5a2f-137">A referenciaimplementáció Kubernetest és [Azure Kubernetes Service-t](/azure/aks/) (AKS) használ.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-137">Our reference implementation uses Kubernetes with [Azure Kubernetes Service](/azure/aks/) (AKS).</span></span> <span data-ttu-id="a5a2f-138">A magas szintű architekturális döntések és kihívások azonban minden tárolóvezénylőre érvényesek, beleértve az [Azure Service Fabricet](/azure/service-fabric/) is.</span><span class="sxs-lookup"><span data-stu-id="a5a2f-138">However, many of the high-level architectural decisions and challenges will apply to any container orchestrator, including [Azure Service Fabric](/azure/service-fabric/).</span></span>

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation/tree/v0.1.0-orig

## <a name="next-steps"></a><span data-ttu-id="a5a2f-139">További lépések</span><span class="sxs-lookup"><span data-stu-id="a5a2f-139">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="a5a2f-140">Számítási lehetőség kiválasztása</span><span class="sxs-lookup"><span data-stu-id="a5a2f-140">Choose a compute option</span></span>](./compute-options.md)