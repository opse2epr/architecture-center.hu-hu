---
title: 'CAF: Nagyvállalati - kezdeti vállalati szabályzat a adatirányítási stratégia mögött'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati - kezdeti vállalati szabályzat a adatirányítási stratégia mögött.
author: BrianBlanchard
ms.openlocfilehash: 51b709b3ae6d704a229a08611a3cbc0da974c6c2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299379"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="3e81c-103">Nagyvállalati: A cégirányítási stratégia mögött kezdeti vállalati szabályzat</span><span class="sxs-lookup"><span data-stu-id="3e81c-103">Large enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="3e81c-104">A következő vállalati szabályzat határozza meg a kezdeti cégirányítási pozíció, amely a kiindulási pontként tartson a folyamatban.</span><span class="sxs-lookup"><span data-stu-id="3e81c-104">The following corporate policy defines the initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="3e81c-105">Ez a cikk a kockázatok korai szakaszában, a kezdeti házirend-utasítások és a házirend-utasítások végrehajtásához korai folyamatok határozza meg.</span><span class="sxs-lookup"><span data-stu-id="3e81c-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="3e81c-106">A vállalati szabályzat nem technikai dokumentum, de azt meghajtók számos technikai döntéseket hozhat.</span><span class="sxs-lookup"><span data-stu-id="3e81c-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="3e81c-107">A cégirányítási MVP ismertetett a [áttekintése](./overview.md) végső soron Ez a szabályzat származik.</span><span class="sxs-lookup"><span data-stu-id="3e81c-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="3e81c-108">A cégirányítási MVP-k megvalósítása előtt a szervezet kell kidolgozni a vállalati házirend alapján saját célkitűzések és az üzleti kockázatok.</span><span class="sxs-lookup"><span data-stu-id="3e81c-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="3e81c-109">A felhő Cégirányítási csapata</span><span class="sxs-lookup"><span data-stu-id="3e81c-109">Cloud Governance team</span></span>

<span data-ttu-id="3e81c-110">A CIO nemrég tartani az informatikai szabályozással csapatával az előzményeit a személyazonosításra alkalmas adatok és a kritikus fontosságú házirendek megértéséhez, valamint ellenőrizze annak hatását, ezek a házirendek módosítása az értekezletről.</span><span class="sxs-lookup"><span data-stu-id="3e81c-110">The CIO recently held a meeting with the IT Governance team to understand the history of the PII and mission-critical policies and review the effect of changing those policies.</span></span> <span data-ttu-id="3e81c-111">Ő is olvashat a teljes lehetséges a felhőben az informatikai és a vállalat.</span><span class="sxs-lookup"><span data-stu-id="3e81c-111">She also discussed the overall potential of the cloud for IT and the company.</span></span>

<span data-ttu-id="3e81c-112">A konferencia után az informatikai szabályozással csoport két tagjára kutatási és támogatja a felhő tervezése erőfeszítések engedélyt kért.</span><span class="sxs-lookup"><span data-stu-id="3e81c-112">After the meeting, two members of the IT Governance team requested permission to research and support the cloud planning efforts.</span></span> <span data-ttu-id="3e81c-113">FELISMERVE kell irányítási és informatikai árnyék-infrastruktúra korlátozása lehetőséghez, Informatikai igazgató Cégirányítási támogatja ezt az elképzelést.</span><span class="sxs-lookup"><span data-stu-id="3e81c-113">Recognizing the need for governance and an opportunity to limit shadow IT, the Director of IT Governance supported this idea.</span></span> <span data-ttu-id="3e81c-114">Az adott a felhő Cégirányítási csapat született.</span><span class="sxs-lookup"><span data-stu-id="3e81c-114">With that, the Cloud Governance team was born.</span></span> <span data-ttu-id="3e81c-115">A következő néhány hónapban, a karbantartás során feltárása a felhőben a cégirányítási szempontjából sok a hiba, öröklik.</span><span class="sxs-lookup"><span data-stu-id="3e81c-115">Over the next several months, they will inherit the cleanup of many mistakes made during exploration in the cloud from a governance perspective.</span></span> <span data-ttu-id="3e81c-116">Ez kap őket a Felhőbeli jegyektől a kézjegy.</span><span class="sxs-lookup"><span data-stu-id="3e81c-116">This will earn them the moniker of Cloud Custodians.</span></span> <span data-ttu-id="3e81c-117">A későbbi fejlesztések folyamaton jelennek meg hogyan azok a szerepkörök az idő előrehaladtával változik.</span><span class="sxs-lookup"><span data-stu-id="3e81c-117">In later evolutions, this journey will show how their roles change over time.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="3e81c-118">Tolerancia mutatók</span><span class="sxs-lookup"><span data-stu-id="3e81c-118">Tolerance indicators</span></span>

<span data-ttu-id="3e81c-119">Az aktuális kockázattűrése magas és a cégirányítási felhőalapú infrastruktúrán végezhetjük a képességéhez értéke alacsony.</span><span class="sxs-lookup"><span data-stu-id="3e81c-119">The current risk tolerance is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="3e81c-120">Ennek megfelelően a tolerancia mutatók járhat el egy korai figyelmeztető rendszert időt és energiát, befektetés aktiválásához.</span><span class="sxs-lookup"><span data-stu-id="3e81c-120">As such, the tolerance indicators act as an early warning system to trigger the investment of time and energy.</span></span> <span data-ttu-id="3e81c-121">Ha a következő mutatók jelennek meg, célszerű előre megtervezni a adatirányítási stratégia fejlődnek lenne.</span><span class="sxs-lookup"><span data-stu-id="3e81c-121">If the following indicators are observed, it would be wise to evolve the governance strategy.</span></span>

- <span data-ttu-id="3e81c-122">A Cost Management: Méretezés a központi telepítés meghaladja a felhőbe 1000 eszközök, vagy havi kiadások meghaladja a 10 000 USD havonta.</span><span class="sxs-lookup"><span data-stu-id="3e81c-122">Cost Management: Scale of deployment exceeds 1,000 assets to the cloud, or monthly spending exceeds $10,000 USD per month.</span></span>
- <span data-ttu-id="3e81c-123">Identitásra építve: Belefoglalási örökölt vagy harmadik felek többtényezős hitelesítés (MFA) követelményekkel rendelkező alkalmazások.</span><span class="sxs-lookup"><span data-stu-id="3e81c-123">Identity Baseline: Inclusion of applications with legacy or third-party multifactor authentication (MFA) requirements.</span></span>
- <span data-ttu-id="3e81c-124">Biztonsági alapkonfiguráció: Védett adatok meghatározott cloud bevezetési terveket felvételét.</span><span class="sxs-lookup"><span data-stu-id="3e81c-124">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="3e81c-125">Erőforrás konzisztencia: Meghatározott cloud bevezetési terveket alapvető fontosságú alkalmazásokat felvételét.</span><span class="sxs-lookup"><span data-stu-id="3e81c-125">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="3e81c-126">További lépések</span><span class="sxs-lookup"><span data-stu-id="3e81c-126">Next steps</span></span>

<span data-ttu-id="3e81c-127">A vállalati házirend előkészíti a felhő Cégirányítási csapat megvalósításához a cégirányítási MVP, amely bevezetésére alapját fogják adni.</span><span class="sxs-lookup"><span data-stu-id="3e81c-127">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="3e81c-128">A következő lépés, hogy az MVP megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="3e81c-128">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="3e81c-129">Bevált gyakorlat ismertetése</span><span class="sxs-lookup"><span data-stu-id="3e81c-129">Best practice explained</span></span>](./best-practice-explained.md)