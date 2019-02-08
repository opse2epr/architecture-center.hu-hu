---
title: 'CAF: A Cost Management fegyelem fokozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A Cost Management fegyelem fokozása
author: BrianBlanchard
ms.openlocfilehash: 34975d195a95b1a85ada96efe8c76a6138385ec1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899463"
---
# <a name="cost-management-discipline-improvement"></a><span data-ttu-id="885f8-103">A Cost Management fegyelem fokozása</span><span class="sxs-lookup"><span data-stu-id="885f8-103">Cost Management discipline improvement</span></span>

<span data-ttu-id="885f8-104">A Cost Management fegyelem próbál meg címet alapvető üzleti kockázatai kapcsolatos számítási feladatok felhőbeli üzemeltetése esetén felmerülő költségek.</span><span class="sxs-lookup"><span data-stu-id="885f8-104">The Cost Management discipline attempts to address core business risks related to expenses incurred when hosting cloud-based workloads.</span></span> <span data-ttu-id="885f8-105">Felhőalapú irányítási öt állapítják belül Cost Management részt vesz a cél az létrehozása és üzemeltetése egy tervezett költség ciklus felhőalapú erőforrások felhasználási és szabályozása.</span><span class="sxs-lookup"><span data-stu-id="885f8-105">Within the five disciplines of Cloud Governance, Cost Management is involved in controlling cost and usage of cloud resources with the goal of creating and maintaining a planned cost cycle.</span></span>

<span data-ttu-id="885f8-106">Ez a cikk ismerteti a lehetséges feladatokat a vállalat fejlesztését, és a Cost Management fegyelem részletes hajtsa végre.</span><span class="sxs-lookup"><span data-stu-id="885f8-106">This article outlines potential tasks your company perform to develop and mature your Cost Management discipline.</span></span> <span data-ttu-id="885f8-107">Ezeket a feladatokat is kell bontani tervezése, létrehozása, bevezetése és a egy felhőalapú megoldás megvalósítása fázisai működik, amely vannak majd iterálni fejlesztését lehetővé teszi a egy [növekményes megközelítés a felhő cégirányítási](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span><span class="sxs-lookup"><span data-stu-id="885f8-107">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![Bevezetési négy fázisa](../../_images/adoption-phases.png)

<span data-ttu-id="885f8-109">*1. ábra A növekményes megközelítés a felhő cégirányítási bevezetési fázisait.*</span><span class="sxs-lookup"><span data-stu-id="885f8-109">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="885f8-110">Nincs egyetlen dokumentum összes vállalatok igényeit is figyelembe.</span><span class="sxs-lookup"><span data-stu-id="885f8-110">No single document can account for the requirements of all businesses.</span></span> <span data-ttu-id="885f8-111">Ezért ez a cikk ismerteti javasolt minimális és a lehetséges példa tevékenységek a cégirányítási érlelését egyes fázisaiban.</span><span class="sxs-lookup"><span data-stu-id="885f8-111">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="885f8-112">Ezek a tevékenységek kezdeti célja segíteni az egy [házirend MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) , valamint olyan keretrendszer, a növekményes házirend fejlődést szem előtt tartva.</span><span class="sxs-lookup"><span data-stu-id="885f8-112">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="885f8-113">A felhő Cégirányítási csapat mennyi Döntse el, hogy ezeket a tevékenységeket a Cost Management irányítási lehetőségek javítása érdekében be kell.</span><span class="sxs-lookup"><span data-stu-id="885f8-113">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Cost Management governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="885f8-114">Sem a minimális vagy potenciális tevékenységek ebben a cikkben leírt adott vállalati szabályzatok vagy harmadik féltől származó megfelelőségi követelmények vannak igazítva.</span><span class="sxs-lookup"><span data-stu-id="885f8-114">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="885f8-115">Ez az útmutató célja a beszélgetések, amely mindkét felhőalapú cégirányítási modell követelményekhez igazítását vezetnek megkönnyítése érdekében.</span><span class="sxs-lookup"><span data-stu-id="885f8-115">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="885f8-116">Tervezés és a készültségi</span><span class="sxs-lookup"><span data-stu-id="885f8-116">Planning and readiness</span></span>

<span data-ttu-id="885f8-117">Ebben a fázisban cégirányítási azt áthidalja a üzleti eredmények és gyakorlatban hasznosítható stratégiák közötti szakadékot.</span><span class="sxs-lookup"><span data-stu-id="885f8-117">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="885f8-118">A folyamat során a a vezetőségi határozza meg az adott mérőszámok, követhesse a digitális hagyatéki vannak leképezve, és megkezdi a teljes áttelepítés sikeresebbé megtervezése.</span><span class="sxs-lookup"><span data-stu-id="885f8-118">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="885f8-119">**Minimális javasolt tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-119">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="885f8-120">Kiértékelheti a [Cost Management eszközlánc](toolchain.md) beállítások.</span><span class="sxs-lookup"><span data-stu-id="885f8-120">Evaluate your [Cost Management toolchain](toolchain.md) options.</span></span>
* <span data-ttu-id="885f8-121">Fejlesztése a draft, architektúra-útmutató a dokumentum, és eljuttatja a kulcsfontosságú érintettekkel.</span><span class="sxs-lookup"><span data-stu-id="885f8-121">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="885f8-122">Ismertetni, és a csapatok architektúra irányelvek fejlesztésének által érintett felhasználók között.</span><span class="sxs-lookup"><span data-stu-id="885f8-122">Educate and involve the people and teams affected by the development of Architecture Guidelines.</span></span>

<span data-ttu-id="885f8-123">**Lehetséges-tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-123">**Potential activities:**</span></span>

* <span data-ttu-id="885f8-124">Győződjön meg arról, amelyek az üzleti indoklás támogatása a stratégia költségvetési döntéseket hozhat.</span><span class="sxs-lookup"><span data-stu-id="885f8-124">Ensure budgetary decisions that support the business justification for your cloud strategy.</span></span>
* <span data-ttu-id="885f8-125">Ellenőrizze a tanulási metrikák, amellyel a sikeres lefoglalt finanszírozási jelentést.</span><span class="sxs-lookup"><span data-stu-id="885f8-125">Validate learning metrics that you use to report on the successful allocation of funding.</span></span>
* <span data-ttu-id="885f8-126">Ismerje meg a kívánt felhő számlázási modell, amely befolyásolja, hogyan felhőköltségek elszámolni.</span><span class="sxs-lookup"><span data-stu-id="885f8-126">Understand the desired cloud accounting model that affects how cloud costs should be accounted for.</span></span>
* <span data-ttu-id="885f8-127">Megismerkedhet a digitális hagyatéki csomaggal, és ellenőrizze a pontos költségszámítási elvárásainak.</span><span class="sxs-lookup"><span data-stu-id="885f8-127">Become familiar with the digital estate plan and validate accurate costing expectations.</span></span>
* <span data-ttu-id="885f8-128">Állapítsa meg, hogy jobban "utólagos fizetést", illetve, hogy egy precommitment megvásárlása nagyvállalati szerződés vásárlási lehetőségek kiértékelése.</span><span class="sxs-lookup"><span data-stu-id="885f8-128">Evaluate buying options to determine if it's better to "pay as you go" or to make a precommitment by purchasing an Enterprise Agreement.</span></span>
* <span data-ttu-id="885f8-129">Üzleti célok összekapcsolása, tervezett költségvetések igazítása és költségvetési tervek szükség szerint módosíthatja.</span><span class="sxs-lookup"><span data-stu-id="885f8-129">Align business goals with planned budgets and adjust budgetary plans as necessary.</span></span>
* <span data-ttu-id="885f8-130">Fejlesztés a célok, és költségvetés-jelentéskészítési mechanizmus, amellyel értesíteni technikai és üzleti résztvevőkkel minden költség ciklus végén.</span><span class="sxs-lookup"><span data-stu-id="885f8-130">Develop a goals and budget reporting mechanism to notify technical and business stakeholders at the end of each cost cycle.</span></span>

## <a name="build-and-pre-deployment"></a><span data-ttu-id="885f8-131">A build és a központi telepítés előtti</span><span class="sxs-lookup"><span data-stu-id="885f8-131">Build and pre-deployment</span></span>

<span data-ttu-id="885f8-132">Technikai és felhasználóknak előfeltételnek környezet sikeres áttelepítéséhez szükségesek.</span><span class="sxs-lookup"><span data-stu-id="885f8-132">A number of technical and nontechnical prerequisites are required to successfully migrate an environment.</span></span> <span data-ttu-id="885f8-133">Ez a folyamat a döntéseket hozhat, készültségi és alapvető infrastruktúra, amely az áttelepítés onnantól összpontosít.</span><span class="sxs-lookup"><span data-stu-id="885f8-133">This process focuses on the decisions, readiness, and core infrastructure that proceeds a migration.</span></span>

<span data-ttu-id="885f8-134">**Minimális javasolt tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-134">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="885f8-135">Alkalmazzon a [Cost Management eszközlánc](toolchain.md) szerint jelennek meg a központi telepítés előtti fázisban.</span><span class="sxs-lookup"><span data-stu-id="885f8-135">Implement your [Cost Management toolchain](toolchain.md) by rolling out in a pre-deployment phase.</span></span>
* <span data-ttu-id="885f8-136">Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.</span><span class="sxs-lookup"><span data-stu-id="885f8-136">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="885f8-137">Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.</span><span class="sxs-lookup"><span data-stu-id="885f8-137">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>
* <span data-ttu-id="885f8-138">Határozza meg, ha a vásárlási követelmény igazítása a költségvetés és a célokat.</span><span class="sxs-lookup"><span data-stu-id="885f8-138">Determine if your purchase requirements align with your budgets and goals.</span></span>

<span data-ttu-id="885f8-139">**Lehetséges-tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-139">**Potential activities:**</span></span>

* <span data-ttu-id="885f8-140">Igazítás a költségvetési terveket biztosít a [előfizetés stratégia](../../decision-guides/subscriptions/overview.md) , amely meghatározza, hogy a core tulajdonjog modellt.</span><span class="sxs-lookup"><span data-stu-id="885f8-140">Align your budgetary plans with the [Subscription Strategy](../../decision-guides/subscriptions/overview.md) that defines your core ownership model.</span></span>
* <span data-ttu-id="885f8-141">Használja ki a [erőforrás konzisztencia stratégia](../../decision-guides/resource-consistency/overview.md) architektúra kényszerítése és irányelveket költségek időbeli alakulása.</span><span class="sxs-lookup"><span data-stu-id="885f8-141">Leverage the [Resource Consistency Strategy](../../decision-guides/resource-consistency/overview.md) to enforce architecture and cost guidelines over time.</span></span>
* <span data-ttu-id="885f8-142">Határozza meg, hogy vannak-e bármilyen költség rendellenességeket, amelyek befolyásolják a bevezetési és áttelepítési terveket.</span><span class="sxs-lookup"><span data-stu-id="885f8-142">Determine if there are any cost anomalies that affect your adoption and migration plans.</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="885f8-143">Elfogadja és áttelepítése</span><span class="sxs-lookup"><span data-stu-id="885f8-143">Adopt and migrate</span></span>

<span data-ttu-id="885f8-144">Áttelepítés az egy növekményes, amely a adatátviteli, a tesztelés és a bevezetési az alkalmazások és számítási feladatok egy meglévő digitális hagyatéki összpontosít.</span><span class="sxs-lookup"><span data-stu-id="885f8-144">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="885f8-145">**Minimális javasolt tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-145">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="885f8-146">Áttelepíteni a [Cost Management eszközlánc](toolchain.md) éles üzem előtti telepítéséből.</span><span class="sxs-lookup"><span data-stu-id="885f8-146">Migrate your [Cost Management toolchain](toolchain.md) from pre-deployment to production.</span></span>
* <span data-ttu-id="885f8-147">Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.</span><span class="sxs-lookup"><span data-stu-id="885f8-147">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="885f8-148">Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.</span><span class="sxs-lookup"><span data-stu-id="885f8-148">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>

<span data-ttu-id="885f8-149">**Lehetséges-tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-149">**Potential activities:**</span></span>

* <span data-ttu-id="885f8-150">A felhő számlázási modell megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="885f8-150">Implement your Cloud Accounting Model.</span></span>
* <span data-ttu-id="885f8-151">Győződjön meg arról, hogy a költségvetés tükrözik a tényleges kiadásokat során minden kiadással, és szükség szerint módosíthatja.</span><span class="sxs-lookup"><span data-stu-id="885f8-151">Ensure that your budgets reflect your actual spending during each release and adjust as necessary.</span></span>
* <span data-ttu-id="885f8-152">Monitorozhatja a változásokat a költségvetési tervek, és ellenőrizze az érintettekkel való, ha további bejelentkezési kompromisszumot van szükség.</span><span class="sxs-lookup"><span data-stu-id="885f8-152">Monitor changes in budgetary plans and validate with stakeholders if additional sign-offs are needed.</span></span>
* <span data-ttu-id="885f8-153">Frissítse a módosításokat az architektúra-útmutató a dokumentumot, tükrözik a tényleges költségek.</span><span class="sxs-lookup"><span data-stu-id="885f8-153">Update changes to the Architecture Guidelines document to reflect actual costs.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="885f8-154">Üzemeltetése és a megvalósítás utáni</span><span class="sxs-lookup"><span data-stu-id="885f8-154">Operate and post-implementation</span></span>

<span data-ttu-id="885f8-155">Az átalakítás befejeződése után cégirányítási műveleteket kell élő és egy alkalmazás vagy munkaterhelés természetes élettartama.</span><span class="sxs-lookup"><span data-stu-id="885f8-155">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="885f8-156">Ebben a fázisban cégirányítási azt tárgyalja a tevékenységek által gyakran után a megoldás bevezetése, és az átalakítási ciklus kerülési kezdődik.</span><span class="sxs-lookup"><span data-stu-id="885f8-156">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="885f8-157">**Minimális javasolt tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-157">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="885f8-158">Testre szabhatja a [Cost Management eszközlánc](toolchain.md) a szervezet cost management igények változása alapján.</span><span class="sxs-lookup"><span data-stu-id="885f8-158">Customize your [Cost Management toolchain](toolchain.md) based on changes in your organization’s cost management needs.</span></span>
* <span data-ttu-id="885f8-159">Vegye figyelembe, hogy a tényleges kiadásokat bármely értesítések és jelentések automatizálása.</span><span class="sxs-lookup"><span data-stu-id="885f8-159">Consider automating any notifications and reports to reflect actual spending.</span></span>
* <span data-ttu-id="885f8-160">Az útmutató későbbi bevezetési folyamatok architektúra irányelvek pontosításához.</span><span class="sxs-lookup"><span data-stu-id="885f8-160">Refine Architecture Guidelines to guide future adoption processes.</span></span>
* <span data-ttu-id="885f8-161">Az architektúra irányelvek betartása folyamatos biztosítása érdekében rendszeresen érintett csapatok oktatása.</span><span class="sxs-lookup"><span data-stu-id="885f8-161">Educate affected teams on a periodic basis to ensure ongoing adherence to the Architecture Guidelines.</span></span>

<span data-ttu-id="885f8-162">**Lehetséges-tevékenységek:**</span><span class="sxs-lookup"><span data-stu-id="885f8-162">**Potential activities:**</span></span>

* <span data-ttu-id="885f8-163">Hajtsa végre a negyedéves felhőalapú üzleti felülvizsgálat való kommunikációhoz, az üzleti és az kapcsolódó költségek értéket.</span><span class="sxs-lookup"><span data-stu-id="885f8-163">Execute a quarterly cloud business review to communicate value delivered to the business and associated costs.</span></span>
* <span data-ttu-id="885f8-164">Módosítsa a tényleges kiadásokat módosításait negyedéves terveket.</span><span class="sxs-lookup"><span data-stu-id="885f8-164">Adjust plans quarterly to reflect changes to actual spending.</span></span>
* <span data-ttu-id="885f8-165">Határozza meg, az üzleti egység előfizetések P & Ls pénzügyi igazítását.</span><span class="sxs-lookup"><span data-stu-id="885f8-165">Determine financial alignment to P&Ls for business unit subscriptions.</span></span>
* <span data-ttu-id="885f8-166">Elemezheti az érdekelt érték és jelentéskészítési módszerrel havi költséget.</span><span class="sxs-lookup"><span data-stu-id="885f8-166">Analyze stakeholder value and cost reporting methods on a monthly basis.</span></span>
* <span data-ttu-id="885f8-167">Használaton eszközök javítása, és határozza meg, érdemes Folytatás zajlik.</span><span class="sxs-lookup"><span data-stu-id="885f8-167">Remediate underused assets and determine if they're worth continuing.</span></span>
* <span data-ttu-id="885f8-168">Ők olyan eltéréseket és rendellenességeket a terv és a tényleges kiadásokat közötti észleli.</span><span class="sxs-lookup"><span data-stu-id="885f8-168">Detect misalignments and anomalies between the plan and actual spending.</span></span>
* <span data-ttu-id="885f8-169">Segítséget nyújt a felhő bevezetésének csapatok és a Felhőstratégia csapat megértése és megoldása a rendellenességeket.</span><span class="sxs-lookup"><span data-stu-id="885f8-169">Assist the cloud adoption teams and the Cloud Strategy team with understanding and resolving these anomalies.</span></span>

## <a name="next-steps"></a><span data-ttu-id="885f8-170">További lépések</span><span class="sxs-lookup"><span data-stu-id="885f8-170">Next steps</span></span>

<span data-ttu-id="885f8-171">Most, hogy megismerkedett a felhőalapú identitáskezelést fogalmát, vizsgálja meg a [Cost Management eszközlánc](toolchain.md) azonosításához az Azure-eszközök és funkciók, amelyek az Azure Cost Management cégirányítási tantárgy fejlesztése során lesz szüksége Platform.</span><span class="sxs-lookup"><span data-stu-id="885f8-171">Now that you understand the concept of cloud identity governance, examine the [Cost Management toolchain](toolchain.md) to identify Azure tools and features that you'll need when developing the Cost Management governance discipline on the Azure platform.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="885f8-172">A Cost Management eszközláncban az Azure-hoz</span><span class="sxs-lookup"><span data-stu-id="885f8-172">Cost Management toolchain for Azure</span></span>](toolchain.md)