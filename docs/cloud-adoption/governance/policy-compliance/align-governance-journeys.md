---
title: 'CAF: Tervezési útmutatók szabályzattal igazítása'
description: Hogyan igazítása tervezési útmutatók házirenddel?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899034"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a><span data-ttu-id="48b3a-103">Hogyan igazítása tervezési útmutatók házirenddel?</span><span class="sxs-lookup"><span data-stu-id="48b3a-103">How do you align design guides with policy?</span></span>

<span data-ttu-id="48b3a-104">Miután [meghatározott felhőalapú házirendek](define-policy.md) alapján a [azonosított kockázatok](understanding-business-risk.md), kell létrehozni, amely igazodik az informatikai részleg és a fejlesztők számára, hogy tekintse meg ezeket a házirendeket a hasznos útmutatást.</span><span class="sxs-lookup"><span data-stu-id="48b3a-104">After you've [defined cloud policies](define-policy.md) based on your [identified risks](understanding-business-risk.md), you'll need to generate actionable guidance that aligns with these policies for your IT staff and developers to refer to.</span></span> <span data-ttu-id="48b3a-105">Tervezése felhőalapú cégirányítási tervezési útmutató lehetővé teszi, hogy adja meg adott szerkezeti, technológiai, és a folyamat lehetőségek az öt cégirányítási állapítják minden egyes létrehozott szabályzat utasítások alapján.</span><span class="sxs-lookup"><span data-stu-id="48b3a-105">Drafting a cloud governance design guide allows you to specify specific structural, technological, and process choices based on the policy statements you generated for each of the five governance disciplines.</span></span>

<span data-ttu-id="48b3a-106">Egy felhőalapú cégirányítási kialakítási útmutató kell létesíteni az architektúrát és tervezési minták minden, felhőben üzemelő példánya, amely a legjobban megfelel a házirend követelményeinek infrastruktúrájának alapvető összetevői.</span><span class="sxs-lookup"><span data-stu-id="48b3a-106">A cloud governance design guide should establish the architecture choices and design patterns for each of the core infrastructure components of cloud deployments that best meet your policy requirements.</span></span> <span data-ttu-id="48b3a-107">Ezek mellett adja meg a technológia, eszközöket és folyamatokat, amelyek támogatni fogja az alábbi tervezési döntéseket minden egyes magas szintű leírását.</span><span class="sxs-lookup"><span data-stu-id="48b3a-107">Alongside these you should provide a high-level explanation of the technology, tools, and processes that will support each of these design decisions.</span></span>

<span data-ttu-id="48b3a-108">Bár a kockázati elemzés és a házirend utasítások előfordulhat, hogy, hogy bizonyos fokon cloud platform-agnosztikus, lehet a kialakítási útmutató kell biztosítania platformspecifikus megvalósítási részletek, amelyek az informatikai RÉSZLEG és dev.</span><span class="sxs-lookup"><span data-stu-id="48b3a-108">Although your risk analysis and policy statements may, to some degree, be cloud platform agnostic, your design guide should provide platform-specific implementation details that your IT and dev.</span></span> <span data-ttu-id="48b3a-109">Tervezési döntés meghozatalakor az architektúra, eszközök és a választott platformtól funkciók összpontosíthat, és útmutatást nyújtunk.</span><span class="sxs-lookup"><span data-stu-id="48b3a-109">Focus on the architecture, tools, and features of your chosen platform when making design decision and providing guidance.</span></span>

<span data-ttu-id="48b3a-110">Felhőbeli tervezési útmutatók figyelembe kell vennie néhány az infrastruktúra-összetevőkhöz kapcsolódó technikai részletek, amíg nem jogosultak széles körű műszaki dokumentumok vagy specifikációk kell lennie.</span><span class="sxs-lookup"><span data-stu-id="48b3a-110">While cloud design guides should take into account some of the technical details associated with each infrastructure component, they are not meant to be extensive technical documents or specifications.</span></span> <span data-ttu-id="48b3a-111">Ellenőrizze, hogy az összes házirend-utasítások és egyértelműen állapot tervezési döntéseket könnyen formátumú személyzetnek ismertetése és oldja meg az útmutatók.</span><span class="sxs-lookup"><span data-stu-id="48b3a-111">Make sure your guides address all of your policy statements and clearly state design decisions in a format easy for staff to understand and reference.</span></span>

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a><span data-ttu-id="48b3a-112">A gyakorlatban hasznosítható cégirányítási Journey használatával</span><span class="sxs-lookup"><span data-stu-id="48b3a-112">Using the actionable governance journeys</span></span>

<span data-ttu-id="48b3a-113">Ha az Azure platform használatához a felhő bevezetésére vonatkozóan, a CAF biztosít [cégirányítási Journey](../journeys/overview.md) ábrázoló a növekményes megközelítés a CAF cégirányítási modell.</span><span class="sxs-lookup"><span data-stu-id="48b3a-113">If you're planning to use the Azure platform for your cloud adoption, the CAF provides [governance journeys](../journeys/overview.md) illustrating the incremental approach of the CAF governance model.</span></span> <span data-ttu-id="48b3a-114">Ezek az nyújt narratíva utak általános bevezetési forgatókönyveket, például az üzleti kockázatok, követelményeket a tolerancia és adtuk ki egy termék létrehozásához keres cégirányítási minimális működőképes (MVP) házirend-utasítások számos terjed ki.</span><span class="sxs-lookup"><span data-stu-id="48b3a-114">These narrative journeys cover a range of common adoption scenarios, including the business risks, tolerance requirements, and policy statements that went into creating a governance minimum viable product (MVP).</span></span> <span data-ttu-id="48b3a-115">Ezek az utak képviseli egy összefoglaló, való életből vett vásárlói élmény, a felhő bevezetésének folyamatát az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="48b3a-115">These journeys represent a synthesis of real-world customer experience of the cloud adoption process in Azure.</span></span>

<span data-ttu-id="48b3a-116">Míg minden felhőre való áttérés egyedi célok prioritások és kihívásokat, ezek a minták kell adja meg egy megfelelő sablont a házirend átalakítása útmutatást.</span><span class="sxs-lookup"><span data-stu-id="48b3a-116">While every cloud adoption has unique goals, priorities, and challenges, these samples should provide a good template for converting your policy into guidance.</span></span> <span data-ttu-id="48b3a-117">Válassza ki a legközelebbi forgatókönyvet az adott helyzethez kiindulási pontként, és elképzelt adott csoportházirend igényeihez igazítva.</span><span class="sxs-lookup"><span data-stu-id="48b3a-117">Pick the closest scenario to your situation as a starting point, and mold it to fit your specific policy needs.</span></span>

## <a name="next-steps"></a><span data-ttu-id="48b3a-118">További lépések</span><span class="sxs-lookup"><span data-stu-id="48b3a-118">Next steps</span></span>

<span data-ttu-id="48b3a-119">Tervezési útmutatást helyezze el, és szabályzat betartása folyamatok szabályzatoknak való megfelelőség biztosításához kialakítása.</span><span class="sxs-lookup"><span data-stu-id="48b3a-119">With design guidance in place, establish policy adherence processes to ensure policy compliance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="48b3a-120">Szabályzat betartása folyamatok</span><span class="sxs-lookup"><span data-stu-id="48b3a-120">Policy adherence processes</span></span>](processes.md)