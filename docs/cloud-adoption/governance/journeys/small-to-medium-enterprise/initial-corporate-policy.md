---
title: 'CAF: Kis és közepes méretű enterprise - kezdeti vállalati szabályzat a adatirányítási stratégia mögött'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Kis és közepes méretű enterprise - kezdeti vállalati szabályzat a cégirányítási strateg mögött
author: BrianBlanchard
ms.openlocfilehash: 35c9398684416b5f4df1d5ea69ac3634b2f73ff3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298441"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="12fa8-103">Small-to-medium enterprise: A cégirányítási stratégia mögött kezdeti vállalati szabályzat</span><span class="sxs-lookup"><span data-stu-id="12fa8-103">Small-to-medium enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="12fa8-104">A következő vállalati szabályzat határozza meg, hogy egy kezdeti cégirányítási pozíció, amely a kiindulási pontként tartson a folyamatban.</span><span class="sxs-lookup"><span data-stu-id="12fa8-104">The following corporate policy defines an initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="12fa8-105">Ez a cikk a kockázatok korai szakaszában, a kezdeti házirend-utasítások és a házirend-utasítások végrehajtásához korai folyamatok határozza meg.</span><span class="sxs-lookup"><span data-stu-id="12fa8-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="12fa8-106">A vállalati szabályzat nem technikai dokumentum, de azt meghajtók számos technikai döntéseket hozhat.</span><span class="sxs-lookup"><span data-stu-id="12fa8-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="12fa8-107">A cégirányítási MVP ismertetett a [áttekintése](./overview.md) végső soron Ez a szabályzat származik.</span><span class="sxs-lookup"><span data-stu-id="12fa8-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="12fa8-108">A cégirányítási MVP-k megvalósítása előtt a szervezet kell kidolgozni a vállalati házirend alapján saját célkitűzések és az üzleti kockázatok.</span><span class="sxs-lookup"><span data-stu-id="12fa8-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="12fa8-109">A felhő Cégirányítási csapata</span><span class="sxs-lookup"><span data-stu-id="12fa8-109">Cloud Governance team</span></span>

<span data-ttu-id="12fa8-110">Az ezen narratíva a felhő Cégirányítási csapat kell irányítási elismert két rendszerek rendszergazdái áll.</span><span class="sxs-lookup"><span data-stu-id="12fa8-110">In this narrative, the Cloud Governance team is comprised of two systems administrators who have recognized the need for governance.</span></span> <span data-ttu-id="12fa8-111">A következő néhány hónapban azok öröklik a feladata a vállalat felhőbeli jelenlét, a felhő jegyektől címének megszerzéséhez őket a irányítása karbantartása.</span><span class="sxs-lookup"><span data-stu-id="12fa8-111">Over the next several months, they will inherit the job of cleaning up the governance of the company’s cloud presence, earning them the title of Cloud Custodians.</span></span> <span data-ttu-id="12fa8-112">A következő fejlesztések Ez a cím valószínűleg változni fognak.</span><span class="sxs-lookup"><span data-stu-id="12fa8-112">In subsequent evolutions, this title will likely change.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="12fa8-113">Tolerancia mutatók</span><span class="sxs-lookup"><span data-stu-id="12fa8-113">Tolerance indicators</span></span>

<span data-ttu-id="12fa8-114">Az aktuális kockázati szintű magas és a cégirányítási felhőalapú infrastruktúrán végezhetjük a képességéhez értéke alacsony.</span><span class="sxs-lookup"><span data-stu-id="12fa8-114">The current tolerance for risk is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="12fa8-115">Mint ilyen a tolerancia mutatók járhat el egy korai figyelmeztető rendszert további befektetések időt és energiát aktiválásához.</span><span class="sxs-lookup"><span data-stu-id="12fa8-115">As such, the tolerance indicators act as an early warning system to trigger more investment of time and energy.</span></span> <span data-ttu-id="12fa8-116">A következő mutatók jelennek meg, ha, a szabályozási stratégiát kell alakítani.</span><span class="sxs-lookup"><span data-stu-id="12fa8-116">If and when the following indicators are observed, you should evolve the governance strategy.</span></span>

- <span data-ttu-id="12fa8-117">A Cost Management: A méretezési csoport a központi telepítés meghaladja a 100 eszközök a felhőbe, vagy havi kiadások meghaladja az 1000 USD havonta.</span><span class="sxs-lookup"><span data-stu-id="12fa8-117">Cost Management: The scale of deployment exceeds 100 assets to the cloud, or monthly spending exceeds $1,000 USD per month.</span></span>
- <span data-ttu-id="12fa8-118">Biztonsági alapkonfiguráció: Védett adatok meghatározott cloud bevezetési terveket felvételét.</span><span class="sxs-lookup"><span data-stu-id="12fa8-118">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="12fa8-119">Erőforrás konzisztencia: Meghatározott cloud bevezetési terveket alapvető fontosságú alkalmazásokat felvételét.</span><span class="sxs-lookup"><span data-stu-id="12fa8-119">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="12fa8-120">További lépések</span><span class="sxs-lookup"><span data-stu-id="12fa8-120">Next steps</span></span>

<span data-ttu-id="12fa8-121">A vállalati házirend előkészíti a felhő Cégirányítási csapat megvalósításához a cégirányítási MVP, amely bevezetésére alapját fogják adni.</span><span class="sxs-lookup"><span data-stu-id="12fa8-121">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="12fa8-122">A következő lépés, hogy az MVP megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="12fa8-122">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="12fa8-123">Bevált gyakorlat ismertetése</span><span class="sxs-lookup"><span data-stu-id="12fa8-123">Best practice explained</span></span>](./best-practice-explained.md)