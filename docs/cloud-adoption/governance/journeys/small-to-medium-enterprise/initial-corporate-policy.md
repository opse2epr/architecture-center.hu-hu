---
title: 'CAF: Kis és közepes méretű Enterprise - mögött a adatirányítási stratégia kezdeti vállalati szabályzat'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Kis és közepes méretű Enterprise - mögött a adatirányítási stratégia kezdeti vállalati szabályzat
author: BrianBlanchard
ms.openlocfilehash: 4f49d0aae2c6ab5a5c8feba669cbbb904af2773b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899082"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="d01db-103">Small-to-medium enterprise: A cégirányítási stratégia mögött kezdeti vállalati szabályzat</span><span class="sxs-lookup"><span data-stu-id="d01db-103">Small-to-medium enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="d01db-104">A következő vállalati szabályzat határozza meg, hogy egy kezdeti cégirányítási pozíció, amely a kiindulási pontként tartson a folyamatban.</span><span class="sxs-lookup"><span data-stu-id="d01db-104">The following corporate policy defines an initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="d01db-105">Ez a cikk a kockázatok korai szakaszában, a kezdeti házirend-utasítások és a házirend-utasítások végrehajtásához korai folyamatok határozza meg.</span><span class="sxs-lookup"><span data-stu-id="d01db-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="d01db-106">A vállalati szabályzat nem technikai dokumentum, de azt meghajtók számos technikai döntéseket hozhat.</span><span class="sxs-lookup"><span data-stu-id="d01db-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="d01db-107">A cégirányítási MVP ismertetett a [áttekintése](./overview.md) végső soron Ez a szabályzat származik.</span><span class="sxs-lookup"><span data-stu-id="d01db-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="d01db-108">A cégirányítási MVP-k megvalósítása előtt a szervezet kell kidolgozni a vállalati házirend alapján saját célkitűzések és az üzleti kockázatok.</span><span class="sxs-lookup"><span data-stu-id="d01db-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="d01db-109">A felhő Cégirányítási csapata</span><span class="sxs-lookup"><span data-stu-id="d01db-109">Cloud Governance team</span></span>

<span data-ttu-id="d01db-110">Az ezen narratíva a felhő Cégirányítási csapat kell irányítási elismert két rendszerek rendszergazdái áll.</span><span class="sxs-lookup"><span data-stu-id="d01db-110">In this narrative, the Cloud Governance team is comprised of two systems administrators who have recognized the need for governance.</span></span> <span data-ttu-id="d01db-111">A következő néhány hónapban azok öröklik a feladata a vállalat felhőbeli jelenlét, a felhő jegyektől címének megszerzéséhez őket a irányítása karbantartása.</span><span class="sxs-lookup"><span data-stu-id="d01db-111">Over the next several months, they will inherit the job of cleaning up the governance of the company’s cloud presence, earning them the title of Cloud Custodians.</span></span> <span data-ttu-id="d01db-112">A következő fejlesztések Ez a cím valószínűleg változni fognak.</span><span class="sxs-lookup"><span data-stu-id="d01db-112">In subsequent evolutions, this title will likely change.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="d01db-113">Tolerancia mutatók</span><span class="sxs-lookup"><span data-stu-id="d01db-113">Tolerance indicators</span></span>

<span data-ttu-id="d01db-114">Az aktuális kockázati szintű magas és a cégirányítási felhőalapú infrastruktúrán végezhetjük a képességéhez értéke alacsony.</span><span class="sxs-lookup"><span data-stu-id="d01db-114">The current tolerance for risk is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="d01db-115">Mint ilyen a tolerancia mutatók járhat el egy korai figyelmeztető rendszert további befektetések időt és energiát aktiválásához.</span><span class="sxs-lookup"><span data-stu-id="d01db-115">As such, the tolerance indicators act as an early warning system to trigger more investment of time and energy.</span></span> <span data-ttu-id="d01db-116">A következő mutatók jelennek meg, ha, a szabályozási stratégiát kell alakítani.</span><span class="sxs-lookup"><span data-stu-id="d01db-116">If and when the following indicators are observed, you should evolve the governance strategy.</span></span>

- <span data-ttu-id="d01db-117">A Cost Management: A méretezési csoport a központi telepítés meghaladja a 100 eszközök a felhőbe, vagy havi kiadások meghaladja az 1000 USD havonta.</span><span class="sxs-lookup"><span data-stu-id="d01db-117">Cost Management: The scale of deployment exceeds 100 assets to the cloud, or monthly spending exceeds $1,000 USD per month.</span></span>
- <span data-ttu-id="d01db-118">Biztonsági alapkonfiguráció: Védett adatok meghatározott cloud bevezetési terveket felvételét.</span><span class="sxs-lookup"><span data-stu-id="d01db-118">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="d01db-119">Erőforrás konzisztencia: Meghatározott cloud bevezetési terveket alapvető fontosságú alkalmazásokat felvételét.</span><span class="sxs-lookup"><span data-stu-id="d01db-119">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="d01db-120">További lépések</span><span class="sxs-lookup"><span data-stu-id="d01db-120">Next steps</span></span>

<span data-ttu-id="d01db-121">A vállalati házirend előkészíti a felhő Cégirányítási csapat megvalósításához a cégirányítási MVP, amely bevezetésére alapját fogják adni.</span><span class="sxs-lookup"><span data-stu-id="d01db-121">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="d01db-122">A következő lépés, hogy az MVP megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="d01db-122">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d01db-123">Bevált gyakorlat ismertetése</span><span class="sxs-lookup"><span data-stu-id="d01db-123">Best practice explained</span></span>](./best-practice-explained.md)