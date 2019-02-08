---
title: 'CAF: Nagyvállalati cégirányítási utazás'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati cégirányítási utazás
author: BrianBlanchard
ms.openlocfilehash: 689b600524c3be6c505ade8c5aaa447d772c6e35
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899061"
---
# <a name="caf-large-enterprise-governance-journey"></a><span data-ttu-id="f4e82-103">CAF: Nagyvállalati cégirányítási utazás</span><span class="sxs-lookup"><span data-stu-id="f4e82-103">CAF: Large enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="f4e82-104">Ajánlott eljárások áttekintése</span><span class="sxs-lookup"><span data-stu-id="f4e82-104">Best practice overview</span></span>

<span data-ttu-id="f4e82-105">Cégirányítási folyamaton keresztül a cégirányítási lejárat különböző szakaszaiban kitalált vállalat tapasztalatainak követi.</span><span class="sxs-lookup"><span data-stu-id="f4e82-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="f4e82-106">Valós felhasználói utak alapján.</span><span class="sxs-lookup"><span data-stu-id="f4e82-106">It is based on real customer journeys.</span></span> <span data-ttu-id="f4e82-107">Az ajánlott gyakorlati tanácsok a korlátozásokat, és kitalált vállalat alapulnak.</span><span class="sxs-lookup"><span data-stu-id="f4e82-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="f4e82-108">Kiindulópontként gyors Ez az Áttekintés egy minimális működőképes termékek (MVP) esetében az ajánlott eljárások alapján cégirányítási határozza meg.</span><span class="sxs-lookup"><span data-stu-id="f4e82-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="f4e82-109">Egyes cégirányítási fejlesztések, további ajánlott eljárások hozzáadása új üzleti hivatkozásokat is tartalmaz, vagy műszaki kockázatok bontakozik ki.</span><span class="sxs-lookup"><span data-stu-id="f4e82-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="f4e82-110">Az MVP egy alapkonfiguráció kiindulási pontjaként feltételezések alapján.</span><span class="sxs-lookup"><span data-stu-id="f4e82-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="f4e82-111">Még a minimálisan szükséges ajánlott eljárásokat egyedi üzleti kockázatot és kockázati tolerancia által üzemeltetett vállalati szabályzatok alapján.</span><span class="sxs-lookup"><span data-stu-id="f4e82-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="f4e82-112">Ha szeretné látni, ha ilyen Előfeltevések vonatkozik Önre, olvassa el a [hosszabb narratíva](./narrative.md) Ez a cikk következő.</span><span class="sxs-lookup"><span data-stu-id="f4e82-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

### <a name="governance-best-practice"></a><span data-ttu-id="f4e82-113">Cégirányítási ajánlott eljárás</span><span class="sxs-lookup"><span data-stu-id="f4e82-113">Governance best practice</span></span>

<span data-ttu-id="f4e82-114">Ez az ajánlott eljárás, amely egy szervezet segítségével gyorsan és következetesen több Azure-előfizetések hozzáadása cégirányítási guardrails alaprendszert funkcionál.</span><span class="sxs-lookup"><span data-stu-id="f4e82-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="f4e82-115">Erőforrás-szervezet</span><span class="sxs-lookup"><span data-stu-id="f4e82-115">Resource organization</span></span>

<span data-ttu-id="f4e82-116">Az alábbi ábrán látható a cégirányítási MVP-hierarchia a erőforrások rendszerezéséhez.</span><span class="sxs-lookup"><span data-stu-id="f4e82-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![Erőforrás-szervezet diagramja](../../../_images/governance/resource-organization.png)

<span data-ttu-id="f4e82-118">Minden alkalmazást kell telepíteni a felügyeleti csoport, az előfizetés és az erőforrás csoporthierarchia megfelelő területén.</span><span class="sxs-lookup"><span data-stu-id="f4e82-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="f4e82-119">Tervezési központi telepítése során a felhő Cégirányítási csapat hoz létre a szükséges csomópontok biztosítson hatékony eszközöket a felhő bevezetésének csapatok a hierarchiában.</span><span class="sxs-lookup"><span data-stu-id="f4e82-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>

1. <span data-ttu-id="f4e82-120">Felügyeleti csoport részletes hierarchia, amely tükrözi a geography környezet (éles környezet, nem éles üzemi) írja be mindegyik üzleti egység számára.</span><span class="sxs-lookup"><span data-stu-id="f4e82-120">A management group for each business unit with a detailed hierarchy that reflects geography then environment type (Production, Non-Production).</span></span>
2. <span data-ttu-id="f4e82-121">Egy előfizetés az egyes egyedi kombinációja üzleti egység, a földrajzi hely, a környezet és a "Kategorizálási Application."</span><span class="sxs-lookup"><span data-stu-id="f4e82-121">A subscription for each unique combination of business unit, geography, environment, and "Application Categorization."</span></span>
3. <span data-ttu-id="f4e82-122">Minden alkalmazáshoz külön erőforrást csoport.</span><span class="sxs-lookup"><span data-stu-id="f4e82-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="f4e82-123">Ez a csoportosítás a hierarchia minden szintjén egységes elnevezési rendszeréhez kell alkalmazni.</span><span class="sxs-lookup"><span data-stu-id="f4e82-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

![Nagyvállalati erőforrás szervezeti diagram](../../../_images/governance/large-enterprise-resource-organization.png)

<span data-ttu-id="f4e82-125">Ezek a minták a növekedésre feleslegesen megfelel a hierarchia nélkül adja meg.</span><span class="sxs-lookup"><span data-stu-id="f4e82-125">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="f4e82-126">Cégirányítási fejlesztések</span><span class="sxs-lookup"><span data-stu-id="f4e82-126">Governance evolutions</span></span>

<span data-ttu-id="f4e82-127">Az MVP üzembe helyezését követően további rétegeit cégirányítási gyorsan beépíthető a környezetben.</span><span class="sxs-lookup"><span data-stu-id="f4e82-127">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="f4e82-128">Íme néhány módszer az MVP speciális üzleti igényeinek változását követve:</span><span class="sxs-lookup"><span data-stu-id="f4e82-128">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="f4e82-129">A védett adatok alapvető biztonsági</span><span class="sxs-lookup"><span data-stu-id="f4e82-129">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="f4e82-130">Az üzletmenet szempontjából kritikus alkalmazások erőforrás-konfigurációk</span><span class="sxs-lookup"><span data-stu-id="f4e82-130">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="f4e82-131">A Cost Management vezérlők</span><span class="sxs-lookup"><span data-stu-id="f4e82-131">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="f4e82-132">Vezérlők többfelhős fejlődést szem előtt tartva</span><span class="sxs-lookup"><span data-stu-id="f4e82-132">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="f4e82-133">Mit jelent ez az ajánlott eljárás?</span><span class="sxs-lookup"><span data-stu-id="f4e82-133">What does this best practice do?</span></span>

<span data-ttu-id="f4e82-134">Az MVP a módszereket, és az eszközök a [üzembe helyezési gyorsítás](../../deployment-acceleration/overview.md) fegyelem számára a vállalati szabályzat alkalmazását létrehozása történik.</span><span class="sxs-lookup"><span data-stu-id="f4e82-134">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="f4e82-135">Különösen az MVP Azure tervezetek, Azure Policy és használ az Azure felügyeleti csoportok néhány alapvető vállalati házirendek alkalmazása a fiktív cég a narratíva meghatározottak szerint.</span><span class="sxs-lookup"><span data-stu-id="f4e82-135">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="f4e82-136">Ezek a vállalati házirendek érvénybe lépnek az Azure Resource Manager-sablonokkal és az Azure házirendek használatával az identitások és a biztonság nagyon kis méretű alap.</span><span class="sxs-lookup"><span data-stu-id="f4e82-136">Those corporate policies are applied using Azure Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="f4e82-138">Az ajánlott eljárás szerint növekvő</span><span class="sxs-lookup"><span data-stu-id="f4e82-138">Evolving the best practice</span></span>

<span data-ttu-id="f4e82-139">Az idő múlásával a cégirányítási MVP való összpontosításnak köszönhetően a felügyeleti eljárásaink használható.</span><span class="sxs-lookup"><span data-stu-id="f4e82-139">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="f4e82-140">Bevezetési aknázhatja nyújtotta fejlett lehetőségeket, mint az üzleti kockázat nő.</span><span class="sxs-lookup"><span data-stu-id="f4e82-140">As adoption advances, business risk grows.</span></span> <span data-ttu-id="f4e82-141">Ezek a kockázatok csökkentése érdekében a CAF cégirányítási modellen belül különböző oktatnak fejlődik.</span><span class="sxs-lookup"><span data-stu-id="f4e82-141">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="f4e82-142">Az oktatóanyag-sorozatban a további cikkek a vállalati házirend, kitalált vállalat érintő alakulása tárgyalják.</span><span class="sxs-lookup"><span data-stu-id="f4e82-142">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="f4e82-143">Ezek a fejlesztések között három oktatnak fordulhat elő:</span><span class="sxs-lookup"><span data-stu-id="f4e82-143">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="f4e82-144">Identitásra építve, áttelepítési függőségek, a a narratíva változása</span><span class="sxs-lookup"><span data-stu-id="f4e82-144">Identity Baseline, as migration dependencies evolve in the narrative</span></span>
- <span data-ttu-id="f4e82-145">A Cost Management bevezetési skálázását követve rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="f4e82-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="f4e82-146">Biztonsági alapterv, mint a védett adatok van üzembe helyezve.</span><span class="sxs-lookup"><span data-stu-id="f4e82-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="f4e82-147">Erőforrás konzisztencia, mint az IT-üzemeltetési kezdődik, üzleti szempontból alapvető fontosságú számítási feladatok támogatása.</span><span class="sxs-lookup"><span data-stu-id="f4e82-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-evolution-large.png)

## <a name="next-steps"></a><span data-ttu-id="f4e82-149">További lépések</span><span class="sxs-lookup"><span data-stu-id="f4e82-149">Next steps</span></span>

<span data-ttu-id="f4e82-150">Most, hogy ismeri a cégirányítási MVP, és kövesse a cégirányítási fejlesztések, ötlete van, olvassa el a támogató narratíva további környezet.</span><span class="sxs-lookup"><span data-stu-id="f4e82-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="f4e82-151">Olvassa el a támogató narratíva</span><span class="sxs-lookup"><span data-stu-id="f4e82-151">Read the supporting narrative</span></span>](./narrative.md)
