---
title: 'CAF: Kis és közepes méretű vállalat cégirányítási utazás'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Kis és közepes méretű vállalat cégirányítási utazás
author: BrianBlanchard
ms.openlocfilehash: a3e078845038a12977e7be5affbf22708411069f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299148"
---
# <a name="small-to-medium-enterprise-governance-journey"></a><span data-ttu-id="0772b-103">Kis és közepes méretű vállalat cégirányítási utazás</span><span class="sxs-lookup"><span data-stu-id="0772b-103">Small-to-medium enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="0772b-104">Ajánlott eljárások áttekintése</span><span class="sxs-lookup"><span data-stu-id="0772b-104">Best practice overview</span></span>

<span data-ttu-id="0772b-105">Cégirányítási folyamaton keresztül a cégirányítási lejárat különböző szakaszaiban kitalált vállalat tapasztalatainak követi.</span><span class="sxs-lookup"><span data-stu-id="0772b-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="0772b-106">Valós felhasználói utak alapján.</span><span class="sxs-lookup"><span data-stu-id="0772b-106">It is based on real customer journeys.</span></span> <span data-ttu-id="0772b-107">Az ajánlott gyakorlati tanácsok a korlátozásokat, és kitalált vállalat alapulnak.</span><span class="sxs-lookup"><span data-stu-id="0772b-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="0772b-108">Kiindulópontként gyors Ez az Áttekintés egy minimális működőképes termékek (MVP) esetében az ajánlott eljárások alapján cégirányítási határozza meg.</span><span class="sxs-lookup"><span data-stu-id="0772b-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="0772b-109">Egyes cégirányítási fejlesztések, további ajánlott eljárások hozzáadása új üzleti hivatkozásokat is tartalmaz, vagy műszaki kockázatok bontakozik ki.</span><span class="sxs-lookup"><span data-stu-id="0772b-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="0772b-110">Az MVP egy alapkonfiguráció kiindulási pontjaként feltételezések alapján.</span><span class="sxs-lookup"><span data-stu-id="0772b-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="0772b-111">Még a minimálisan szükséges ajánlott eljárásokat egyedi üzleti kockázatot és kockázati tolerancia által üzemeltetett vállalati szabályzatok alapján.</span><span class="sxs-lookup"><span data-stu-id="0772b-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="0772b-112">Ha szeretné látni, ha ilyen Előfeltevések vonatkozik Önre, olvassa el a [hosszabb narratíva](./narrative.md) Ez a cikk következő.</span><span class="sxs-lookup"><span data-stu-id="0772b-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

## <a name="governance-best-practice"></a><span data-ttu-id="0772b-113">Cégirányítási ajánlott eljárás</span><span class="sxs-lookup"><span data-stu-id="0772b-113">Governance best practice</span></span>

<span data-ttu-id="0772b-114">Ez az ajánlott eljárás, amely egy szervezet segítségével gyorsan és következetesen több Azure-előfizetések hozzáadása cégirányítási guardrails alaprendszert funkcionál.</span><span class="sxs-lookup"><span data-stu-id="0772b-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="0772b-115">Erőforrás-szervezet</span><span class="sxs-lookup"><span data-stu-id="0772b-115">Resource organization</span></span>

<span data-ttu-id="0772b-116">Az alábbi ábrán látható a cégirányítási MVP-hierarchia a erőforrások rendszerezéséhez.</span><span class="sxs-lookup"><span data-stu-id="0772b-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![Erőforrás-szervezet diagramja](../../../_images/governance/resource-organization.png)

<span data-ttu-id="0772b-118">Minden alkalmazást kell telepíteni a felügyeleti csoport, az előfizetés és az erőforrás csoporthierarchia megfelelő területén.</span><span class="sxs-lookup"><span data-stu-id="0772b-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="0772b-119">Tervezési központi telepítése során a felhő Cégirányítási csapat hoz létre a szükséges csomópontok biztosítson hatékony eszközöket a felhő bevezetésének csapatok a hierarchiában.</span><span class="sxs-lookup"><span data-stu-id="0772b-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>  

1. <span data-ttu-id="0772b-120">Felügyeleti csoport minden környezetben (például éles, fejlesztési és tesztelési) típusú.</span><span class="sxs-lookup"><span data-stu-id="0772b-120">A management group for each type of environment (such as Production, Development, and Test).</span></span>
2. <span data-ttu-id="0772b-121">Minden egyes "alkalmazás"kategorizálási előfizetés.</span><span class="sxs-lookup"><span data-stu-id="0772b-121">A subscription for each "application categorization".</span></span>
3. <span data-ttu-id="0772b-122">Minden alkalmazáshoz külön erőforrást csoport.</span><span class="sxs-lookup"><span data-stu-id="0772b-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="0772b-123">Ez a csoportosítás a hierarchia minden szintjén egységes elnevezési rendszeréhez kell alkalmazni.</span><span class="sxs-lookup"><span data-stu-id="0772b-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

<span data-ttu-id="0772b-124">Íme egy példa ezt a mintát használja:</span><span class="sxs-lookup"><span data-stu-id="0772b-124">Here is an example of this pattern in use:</span></span>

![A piaci vállalati erőforrás szervezet példa](../../../_images/governance/mid-market-resource-organization.png)

<span data-ttu-id="0772b-126">Ezek a minták a növekedésre feleslegesen megfelel a hierarchia nélkül adja meg.</span><span class="sxs-lookup"><span data-stu-id="0772b-126">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="0772b-127">Cégirányítási fejlesztések</span><span class="sxs-lookup"><span data-stu-id="0772b-127">Governance evolutions</span></span>

<span data-ttu-id="0772b-128">Az MVP üzembe helyezését követően további rétegeit cégirányítási gyorsan beépíthető a környezetben.</span><span class="sxs-lookup"><span data-stu-id="0772b-128">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="0772b-129">Íme néhány módszer az MVP speciális üzleti igényeinek változását követve:</span><span class="sxs-lookup"><span data-stu-id="0772b-129">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="0772b-130">A védett adatok alapvető biztonsági</span><span class="sxs-lookup"><span data-stu-id="0772b-130">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="0772b-131">Az üzletmenet szempontjából kritikus alkalmazások erőforrás-konfigurációk</span><span class="sxs-lookup"><span data-stu-id="0772b-131">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="0772b-132">A Cost Management vezérlők</span><span class="sxs-lookup"><span data-stu-id="0772b-132">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="0772b-133">Vezérlők többfelhős fejlődést szem előtt tartva</span><span class="sxs-lookup"><span data-stu-id="0772b-133">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="0772b-134">Mit jelent ez az ajánlott eljárás?</span><span class="sxs-lookup"><span data-stu-id="0772b-134">What does this best practice do?</span></span>

<span data-ttu-id="0772b-135">Az MVP a módszereket, és az eszközök a [üzembe helyezési gyorsítás](../../deployment-acceleration/overview.md) fegyelem számára a vállalati szabályzat alkalmazását létrehozása történik.</span><span class="sxs-lookup"><span data-stu-id="0772b-135">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="0772b-136">Különösen az MVP Azure tervezetek, Azure Policy és használ az Azure felügyeleti csoportok néhány alapvető vállalati házirendek alkalmazása a fiktív cég a narratíva meghatározottak szerint.</span><span class="sxs-lookup"><span data-stu-id="0772b-136">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="0772b-137">Ezek a vállalati házirendek érvénybe lépnek Resource Manager-sablonokkal és az Azure szabályzatok használata az identitások és a biztonság nagyon kis méretű alap.</span><span class="sxs-lookup"><span data-stu-id="0772b-137">Those corporate policies are applied using Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="0772b-139">Az ajánlott eljárás szerint növekvő</span><span class="sxs-lookup"><span data-stu-id="0772b-139">Evolving the best practice</span></span>

<span data-ttu-id="0772b-140">Az idő múlásával a cégirányítási MVP való összpontosításnak köszönhetően a felügyeleti eljárásaink használható.</span><span class="sxs-lookup"><span data-stu-id="0772b-140">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="0772b-141">Bevezetési aknázhatja nyújtotta fejlett lehetőségeket, mint az üzleti kockázat nő.</span><span class="sxs-lookup"><span data-stu-id="0772b-141">As adoption advances, business risk grows.</span></span> <span data-ttu-id="0772b-142">Ezek a kockázatok csökkentése érdekében a CAF cégirányítási modellen belül különböző oktatnak fejlődik.</span><span class="sxs-lookup"><span data-stu-id="0772b-142">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="0772b-143">Az oktatóanyag-sorozatban a további cikkek a vállalati házirend, kitalált vállalat érintő alakulása tárgyalják.</span><span class="sxs-lookup"><span data-stu-id="0772b-143">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="0772b-144">Ezek a fejlesztések között három oktatnak fordulhat elő:</span><span class="sxs-lookup"><span data-stu-id="0772b-144">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="0772b-145">A Cost Management bevezetési skálázását követve rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="0772b-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="0772b-146">Biztonsági alapterv, mint a védett adatok van üzembe helyezve.</span><span class="sxs-lookup"><span data-stu-id="0772b-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="0772b-147">Erőforrás konzisztencia, mint az IT-üzemeltetési kezdődik, üzleti szempontból alapvető fontosságú számítási feladatok támogatása.</span><span class="sxs-lookup"><span data-stu-id="0772b-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-evolution.png)

## <a name="next-steps"></a><span data-ttu-id="0772b-149">További lépések</span><span class="sxs-lookup"><span data-stu-id="0772b-149">Next steps</span></span>

<span data-ttu-id="0772b-150">Most, hogy ismeri a cégirányítási MVP, és kövesse a cégirányítási fejlesztések, ötlete van, olvassa el a támogató narratíva további környezet.</span><span class="sxs-lookup"><span data-stu-id="0772b-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="0772b-151">Olvassa el a támogató narratíva</span><span class="sxs-lookup"><span data-stu-id="0772b-151">Read the supporting narrative</span></span>](./narrative.md)
