---
title: A 5 ésszerűsítés az Rs
titleSuffix: Enterprise Cloud Adoption
description: Ismerteti, amely egy digitális hagyatéki ésszerűsítéséről esetén érhetők el a beállításokat
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 06058967e6ffcd9e3554a46c67144f72fb19078f
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908579"
---
# <a name="enterprise-cloud-adoption-the-5-rs-of-rationalization"></a><span data-ttu-id="6db28-103">Enterprise Cloud Adoption: A 5 ésszerűsítés az Rs</span><span class="sxs-lookup"><span data-stu-id="6db28-103">Enterprise Cloud Adoption: The 5 Rs of rationalization</span></span>

<span data-ttu-id="6db28-104">Felhőbeli ésszerűsítés az a folyamat kiértékelésének adategységeket az áttelepítés vagy a felhőben minden egyes eszköz modernizálhatja a legjobb módja határozza meg.</span><span class="sxs-lookup"><span data-stu-id="6db28-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="6db28-105">Ésszerűsítés a folyamattal kapcsolatos további információk: [digitális hagyatéki mi?](overview.md)</span><span class="sxs-lookup"><span data-stu-id="6db28-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="6db28-106">Az "5 Rs ésszerűsítés az" felsorolt Itt a ésszerűsítés leggyakrabban használt beállításait ismertetik.</span><span class="sxs-lookup"><span data-stu-id="6db28-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="6db28-107">Újratárolás</span><span class="sxs-lookup"><span data-stu-id="6db28-107">Rehost</span></span>

<span data-ttu-id="6db28-108">Más néven "lift- and -shift," egy áthelyezési erőfeszítés helyezi át a jelenlegi állapot az eszközintelligencia a kiválasztott felhőszolgáltató, az általános architektúrát minimális módosítása.</span><span class="sxs-lookup"><span data-stu-id="6db28-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="6db28-109">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="6db28-109">Common drivers could include:</span></span>

* <span data-ttu-id="6db28-110">Reduce CapEx</span><span class="sxs-lookup"><span data-stu-id="6db28-110">Reduce CapEx</span></span>
* <span data-ttu-id="6db28-111">Szabadítson fel lemezterületet a datacenter</span><span class="sxs-lookup"><span data-stu-id="6db28-111">Free up datacenter space</span></span>
* <span data-ttu-id="6db28-112">Gyors felhőbeli megtérülés</span><span class="sxs-lookup"><span data-stu-id="6db28-112">Quick cloud ROI</span></span>

<span data-ttu-id="6db28-113">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6db28-114">Virtuálisgép-méret (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="6db28-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6db28-115">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="6db28-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6db28-116">Az eszközintelligencia-kompatibilitás</span><span class="sxs-lookup"><span data-stu-id="6db28-116">Asset compatibility</span></span>

<span data-ttu-id="6db28-117">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6db28-118">Tolerancia módosítás</span><span class="sxs-lookup"><span data-stu-id="6db28-118">Tolerance for change</span></span>
* <span data-ttu-id="6db28-119">Üzleti prioritásokhoz</span><span class="sxs-lookup"><span data-stu-id="6db28-119">Business priorities</span></span>
* <span data-ttu-id="6db28-120">Kritikus fontosságú üzleti eseményeket</span><span class="sxs-lookup"><span data-stu-id="6db28-120">Critical business events</span></span>
* <span data-ttu-id="6db28-121">Folyamat függőségek</span><span class="sxs-lookup"><span data-stu-id="6db28-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="6db28-122">Újrabontás</span><span class="sxs-lookup"><span data-stu-id="6db28-122">Refactor</span></span>

<span data-ttu-id="6db28-123">Szolgáltatásbeállítások (PaaS) a platformszolgáltatás csökkentheti a működési költségeket számos alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="6db28-123">Platform as a Service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="6db28-124">Egy alkalmazásnak a PaaS-alapú modell némileg bontani körültekintő lehet.</span><span class="sxs-lookup"><span data-stu-id="6db28-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="6db28-125">Az alkalmazás fejlesztési folyamat a kódot, hogy az új üzleti lehetőségek eljuttatható alkalmazások újrabontása újrabontása is hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="6db28-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="6db28-126">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="6db28-126">Common drivers could include:</span></span>

* <span data-ttu-id="6db28-127">Gyorsabb, rövidebb, frissítések</span><span class="sxs-lookup"><span data-stu-id="6db28-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="6db28-128">Kód hordozhatóság</span><span class="sxs-lookup"><span data-stu-id="6db28-128">Code portability</span></span>
* <span data-ttu-id="6db28-129">(Erőforrás, sebességét, költség) nagyobb felhő hatékonyságát.</span><span class="sxs-lookup"><span data-stu-id="6db28-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="6db28-130">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6db28-131">Az eszközintelligencia mérete (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="6db28-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6db28-132">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="6db28-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6db28-133">A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)</span><span class="sxs-lookup"><span data-stu-id="6db28-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="6db28-134">Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)</span><span class="sxs-lookup"><span data-stu-id="6db28-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="6db28-135">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6db28-136">Folyamatos üzletmenet, befektetés</span><span class="sxs-lookup"><span data-stu-id="6db28-136">Continued business investments</span></span>
* <span data-ttu-id="6db28-137">Beállítások/ütemtervek tartalékkapacitás</span><span class="sxs-lookup"><span data-stu-id="6db28-137">Bursting options/timelines</span></span>
* <span data-ttu-id="6db28-138">Üzleti folyamat függőségek</span><span class="sxs-lookup"><span data-stu-id="6db28-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="6db28-139">Újratervezés</span><span class="sxs-lookup"><span data-stu-id="6db28-139">Rearchitect</span></span>

<span data-ttu-id="6db28-140">Egyes elévült alkalmazások nem kompatibilis a felhőszolgáltatók miatt az architekturális döntések, amikor az alkalmazás lett létrehozva.</span><span class="sxs-lookup"><span data-stu-id="6db28-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="6db28-141">Ezekben az esetekben az alkalmazás szükségessé kell rearchitected átalakítása előtt.</span><span class="sxs-lookup"><span data-stu-id="6db28-141">In these cases, the application may need to be rearchitected prior to transformation.</span></span>

<span data-ttu-id="6db28-142">Más esetekben az alkalmazásokat, amelyek a felhő-kompatibilis, de nem felhőbeli natív előnnyel jár, eredményezhet költséghatékonyságot és a működési hatékonyságot átszervezése a natív felhőalkalmazások-megoldás által.</span><span class="sxs-lookup"><span data-stu-id="6db28-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="6db28-143">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="6db28-143">Common drivers could include:</span></span>

* <span data-ttu-id="6db28-144">Alkalmazás méretezhetőség és mozgékonyság</span><span class="sxs-lookup"><span data-stu-id="6db28-144">Application scale and agility</span></span>
* <span data-ttu-id="6db28-145">Új felhőalapú képességek könnyebben elfogadása</span><span class="sxs-lookup"><span data-stu-id="6db28-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="6db28-146">Többféle technológiai csomaggal</span><span class="sxs-lookup"><span data-stu-id="6db28-146">Mix of technology stacks</span></span>

<span data-ttu-id="6db28-147">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6db28-148">Az eszközintelligencia mérete (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="6db28-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6db28-149">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="6db28-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6db28-150">A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)</span><span class="sxs-lookup"><span data-stu-id="6db28-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="6db28-151">Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)</span><span class="sxs-lookup"><span data-stu-id="6db28-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="6db28-152">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6db28-153">Egyre bővülő üzleti befektetéseit</span><span class="sxs-lookup"><span data-stu-id="6db28-153">Growing business investments</span></span>
* <span data-ttu-id="6db28-154">A működési költségek</span><span class="sxs-lookup"><span data-stu-id="6db28-154">Operational costs</span></span>
* <span data-ttu-id="6db28-155">A lehetséges visszajelzési hurkokat és a DevOps, befektetés</span><span class="sxs-lookup"><span data-stu-id="6db28-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="6db28-156">Újraépítés</span><span class="sxs-lookup"><span data-stu-id="6db28-156">Rebuild</span></span>

<span data-ttu-id="6db28-157">Bizonyos esetekben a különbözeti, amely kell meg lehet oldani a átviszi az alkalmazás lehet túl nagy, befektetés további igazoló.</span><span class="sxs-lookup"><span data-stu-id="6db28-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="6db28-158">Ez különösen igaz a alkalmazásokat, amelyek segítségével az üzleti, de a rendszer most már nem támogatott vagy a helytelen igazítású, hogy az üzleti folyamatok végrehajtása még ma az igényeinek.</span><span class="sxs-lookup"><span data-stu-id="6db28-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="6db28-159">Ebben az esetben egy új kódbázis jön létre, amelyek összhangban vannak a [felhőbeli natív](https://azure.microsoft.com/overview/cloudnative/) megközelítést.</span><span class="sxs-lookup"><span data-stu-id="6db28-159">In this case, a new code base is created to align with a [cloud native](https://azure.microsoft.com/overview/cloudnative/) approach.</span></span>

<span data-ttu-id="6db28-160">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="6db28-160">Common drivers could include:</span></span>

* <span data-ttu-id="6db28-161">Gyorsabb innováció</span><span class="sxs-lookup"><span data-stu-id="6db28-161">Accelerate innovation</span></span>
* <span data-ttu-id="6db28-162">Gyorsabb alkalmazáskészítés</span><span class="sxs-lookup"><span data-stu-id="6db28-162">Build apps faster</span></span>
* <span data-ttu-id="6db28-163">Működési költségek csökkentése</span><span class="sxs-lookup"><span data-stu-id="6db28-163">Reduce operational cost</span></span>

<span data-ttu-id="6db28-164">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6db28-165">Az eszközintelligencia mérete (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="6db28-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6db28-166">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="6db28-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6db28-167">A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)</span><span class="sxs-lookup"><span data-stu-id="6db28-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="6db28-168">Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)</span><span class="sxs-lookup"><span data-stu-id="6db28-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="6db28-169">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="6db28-170">Végfelhasználói elégedettség elutasítása</span><span class="sxs-lookup"><span data-stu-id="6db28-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="6db28-171">Üzleti folyamatok korlátozott funkciók szerint</span><span class="sxs-lookup"><span data-stu-id="6db28-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="6db28-172">A lehetséges díj, a felhasználói élményt és a bevétel nyereség</span><span class="sxs-lookup"><span data-stu-id="6db28-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="6db28-173">Csere</span><span class="sxs-lookup"><span data-stu-id="6db28-173">Replace</span></span>

<span data-ttu-id="6db28-174">Megoldások általában a legjobb technológiával és rendelkezésre álló módszer használatával időben vannak megvalósítva.</span><span class="sxs-lookup"><span data-stu-id="6db28-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="6db28-175">Bizonyos esetekben az üzemeltetett alkalmazás szükséges funkciót összes szoftvert, mint a szoftverszolgáltatások (SaaS) alkalmazások is megfelelnek.</span><span class="sxs-lookup"><span data-stu-id="6db28-175">In some cases, Software as a Service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="6db28-176">Ezekben az esetekben egy számítási feladat sikerült kell nyár kerül piacra jövőbeli kell cserélni, hatékonyan eltávolítaná az átalakítási beavatkozást.</span><span class="sxs-lookup"><span data-stu-id="6db28-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="6db28-177">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="6db28-177">Common drivers could include:</span></span>

* <span data-ttu-id="6db28-178">Iparág-– gyakorlati tanácsok körül szabványosítása</span><span class="sxs-lookup"><span data-stu-id="6db28-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="6db28-179">Gyorsabb üzleti folyamat megközelítések driven elfogadása</span><span class="sxs-lookup"><span data-stu-id="6db28-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="6db28-180">Foglalja le újból a fejlesztési beruházások kiterjeszthetők az alkalmazásokat, amelyek versenyképes differenciálás vagy előnyeit létrehozása</span><span class="sxs-lookup"><span data-stu-id="6db28-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="6db28-181">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="6db28-182">Általános működési költségek csökkentése</span><span class="sxs-lookup"><span data-stu-id="6db28-182">General operating cost reductions</span></span>
* <span data-ttu-id="6db28-183">Virtuálisgép-méret (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="6db28-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="6db28-184">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="6db28-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="6db28-185">Eszközök megszűnik</span><span class="sxs-lookup"><span data-stu-id="6db28-185">Assets to be retired</span></span>

<span data-ttu-id="6db28-186">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="6db28-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="6db28-187">A Cost haszon elemzése a jelenlegi architektúra vs Szolgáltatottszoftver-megoldás</span><span class="sxs-lookup"><span data-stu-id="6db28-187">Cost benefit analysis of current architecture vs SaaS solution</span></span>
* <span data-ttu-id="6db28-188">Üzleti folyamatok térképek</span><span class="sxs-lookup"><span data-stu-id="6db28-188">Business process maps</span></span>
* <span data-ttu-id="6db28-189">Adatsémák</span><span class="sxs-lookup"><span data-stu-id="6db28-189">Data schemas</span></span>
* <span data-ttu-id="6db28-190">Egyéni vagy automatizált folyamatokkal</span><span class="sxs-lookup"><span data-stu-id="6db28-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="6db28-191">További lépések</span><span class="sxs-lookup"><span data-stu-id="6db28-191">Next steps</span></span>

<span data-ttu-id="6db28-192">Együttesen ezek ésszerűsítés 5 Rs is alkalmazható egy digitális hagyatéki ésszerűsítés minden alkalmazás jövőbeli állapota kapcsolatos döntéseket.</span><span class="sxs-lookup"><span data-stu-id="6db28-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="6db28-193">Mi az a digitális hagyatéki?</span><span class="sxs-lookup"><span data-stu-id="6db28-193">What is a digital estate?</span></span>](overview.md)
