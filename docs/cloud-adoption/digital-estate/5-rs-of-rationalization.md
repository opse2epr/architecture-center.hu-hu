---
title: A 5 ésszerűsítés az Rs
titleSuffix: Enterprise Cloud Adoption
description: Ismerteti, amely egy digitális hagyatéki ésszerűsítéséről esetén érhetők el a beállításokat
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 4e2765198b64c36470adc9fbe35872e4714780e8
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179452"
---
# <a name="enterprise-cloud-adoption-the-5-rs-of-rationalization"></a><span data-ttu-id="10da1-103">Enterprise Cloud Adoption: A 5 ésszerűsítés az Rs</span><span class="sxs-lookup"><span data-stu-id="10da1-103">Enterprise Cloud Adoption: The 5 Rs of rationalization</span></span>

<span data-ttu-id="10da1-104">Felhőbeli ésszerűsítés az a folyamat kiértékelésének adategységeket az áttelepítés vagy a felhőben minden egyes eszköz modernizálhatja a legjobb módja határozza meg.</span><span class="sxs-lookup"><span data-stu-id="10da1-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="10da1-105">Ésszerűsítés a folyamattal kapcsolatos további információk: [digitális hagyatéki mi?](overview.md)</span><span class="sxs-lookup"><span data-stu-id="10da1-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="10da1-106">Az "5 Rs ésszerűsítés az" felsorolt Itt a ésszerűsítés leggyakrabban használt beállításait ismertetik.</span><span class="sxs-lookup"><span data-stu-id="10da1-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="10da1-107">Újratárolás</span><span class="sxs-lookup"><span data-stu-id="10da1-107">Rehost</span></span>

<span data-ttu-id="10da1-108">Más néven "lift- and -shift," egy áthelyezési erőfeszítés helyezi át a jelenlegi állapot az eszközintelligencia a kiválasztott felhőszolgáltató, az általános architektúrát minimális módosítása.</span><span class="sxs-lookup"><span data-stu-id="10da1-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="10da1-109">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="10da1-109">Common drivers could include:</span></span>

* <span data-ttu-id="10da1-110">Csökkentheti a tőkeráfordítás mértékét</span><span class="sxs-lookup"><span data-stu-id="10da1-110">Reduce CapEx</span></span>
* <span data-ttu-id="10da1-111">Szabadítson fel lemezterületet a datacenter</span><span class="sxs-lookup"><span data-stu-id="10da1-111">Free up datacenter space</span></span>
* <span data-ttu-id="10da1-112">Gyors felhőbeli megtérülés</span><span class="sxs-lookup"><span data-stu-id="10da1-112">Quick cloud ROI</span></span>

<span data-ttu-id="10da1-113">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="10da1-114">Virtuálisgép-méret (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="10da1-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="10da1-115">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="10da1-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="10da1-116">Az eszközintelligencia-kompatibilitás</span><span class="sxs-lookup"><span data-stu-id="10da1-116">Asset compatibility</span></span>

<span data-ttu-id="10da1-117">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="10da1-118">Tolerancia módosítás</span><span class="sxs-lookup"><span data-stu-id="10da1-118">Tolerance for change</span></span>
* <span data-ttu-id="10da1-119">Üzleti prioritásokhoz</span><span class="sxs-lookup"><span data-stu-id="10da1-119">Business priorities</span></span>
* <span data-ttu-id="10da1-120">Kritikus fontosságú üzleti eseményeket</span><span class="sxs-lookup"><span data-stu-id="10da1-120">Critical business events</span></span>
* <span data-ttu-id="10da1-121">Folyamat függőségek</span><span class="sxs-lookup"><span data-stu-id="10da1-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="10da1-122">Újrabontás</span><span class="sxs-lookup"><span data-stu-id="10da1-122">Refactor</span></span>

<span data-ttu-id="10da1-123">Szolgáltatásbeállítások (PaaS) a platformszolgáltatás csökkentheti a működési költségeket számos alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="10da1-123">Platform as a Service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="10da1-124">Egy alkalmazásnak a PaaS-alapú modell némileg bontani körültekintő lehet.</span><span class="sxs-lookup"><span data-stu-id="10da1-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="10da1-125">Az alkalmazás fejlesztési folyamat a kódot, hogy az új üzleti lehetőségek eljuttatható alkalmazások újrabontása újrabontása is hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="10da1-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="10da1-126">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="10da1-126">Common drivers could include:</span></span>

* <span data-ttu-id="10da1-127">Gyorsabb, rövidebb, frissítések</span><span class="sxs-lookup"><span data-stu-id="10da1-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="10da1-128">Kód hordozhatóság</span><span class="sxs-lookup"><span data-stu-id="10da1-128">Code portability</span></span>
* <span data-ttu-id="10da1-129">(Erőforrás, sebességét, költség) nagyobb felhő hatékonyságát.</span><span class="sxs-lookup"><span data-stu-id="10da1-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="10da1-130">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="10da1-131">Az eszközintelligencia mérete (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="10da1-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="10da1-132">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="10da1-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="10da1-133">A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)</span><span class="sxs-lookup"><span data-stu-id="10da1-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="10da1-134">Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)</span><span class="sxs-lookup"><span data-stu-id="10da1-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="10da1-135">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="10da1-136">Folyamatos üzletmenet, befektetés</span><span class="sxs-lookup"><span data-stu-id="10da1-136">Continued business investments</span></span>
* <span data-ttu-id="10da1-137">Beállítások/ütemtervek tartalékkapacitás</span><span class="sxs-lookup"><span data-stu-id="10da1-137">Bursting options/timelines</span></span>
* <span data-ttu-id="10da1-138">Üzleti folyamat függőségek</span><span class="sxs-lookup"><span data-stu-id="10da1-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="10da1-139">Újratervezés</span><span class="sxs-lookup"><span data-stu-id="10da1-139">Rearchitect</span></span>

<span data-ttu-id="10da1-140">Egyes elévült alkalmazások nem kompatibilis a felhőszolgáltatók miatt az architekturális döntések, amikor az alkalmazás lett létrehozva.</span><span class="sxs-lookup"><span data-stu-id="10da1-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="10da1-141">Ezekben az esetekben az alkalmazás szükségessé kell rearchitected átalakítása előtt.</span><span class="sxs-lookup"><span data-stu-id="10da1-141">In these cases, the application may need to be rearchitected prior to transformation.</span></span>

<span data-ttu-id="10da1-142">Más esetekben az alkalmazásokat, amelyek a felhő-kompatibilis, de nem felhőbeli natív előnnyel jár, eredményezhet költséghatékonyságot és a működési hatékonyságot átszervezése a natív felhőalkalmazások-megoldás által.</span><span class="sxs-lookup"><span data-stu-id="10da1-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="10da1-143">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="10da1-143">Common drivers could include:</span></span>

* <span data-ttu-id="10da1-144">Alkalmazás méretezhetőség és mozgékonyság</span><span class="sxs-lookup"><span data-stu-id="10da1-144">Application scale and agility</span></span>
* <span data-ttu-id="10da1-145">Új felhőalapú képességek könnyebben elfogadása</span><span class="sxs-lookup"><span data-stu-id="10da1-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="10da1-146">Többféle technológiai csomaggal</span><span class="sxs-lookup"><span data-stu-id="10da1-146">Mix of technology stacks</span></span>

<span data-ttu-id="10da1-147">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="10da1-148">Az eszközintelligencia mérete (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="10da1-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="10da1-149">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="10da1-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="10da1-150">A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)</span><span class="sxs-lookup"><span data-stu-id="10da1-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="10da1-151">Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)</span><span class="sxs-lookup"><span data-stu-id="10da1-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="10da1-152">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="10da1-153">Egyre bővülő üzleti befektetéseit</span><span class="sxs-lookup"><span data-stu-id="10da1-153">Growing business investments</span></span>
* <span data-ttu-id="10da1-154">A működési költségek</span><span class="sxs-lookup"><span data-stu-id="10da1-154">Operational costs</span></span>
* <span data-ttu-id="10da1-155">A lehetséges visszajelzési hurkokat és a DevOps, befektetés</span><span class="sxs-lookup"><span data-stu-id="10da1-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="10da1-156">Újraépítés</span><span class="sxs-lookup"><span data-stu-id="10da1-156">Rebuild</span></span>

<span data-ttu-id="10da1-157">Bizonyos esetekben a különbözeti, amely kell meg lehet oldani a átviszi az alkalmazás lehet túl nagy, befektetés további igazoló.</span><span class="sxs-lookup"><span data-stu-id="10da1-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="10da1-158">Ez különösen igaz a alkalmazásokat, amelyek segítségével az üzleti, de a rendszer most már nem támogatott vagy a helytelen igazítású, hogy az üzleti folyamatok végrehajtása még ma az igényeinek.</span><span class="sxs-lookup"><span data-stu-id="10da1-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="10da1-159">Ebben az esetben egy új kódbázis egy felhőbeli natív megközelítés igazodva jön létre.</span><span class="sxs-lookup"><span data-stu-id="10da1-159">In this case, a new code base is created to align with a cloud native approach.</span></span>

<span data-ttu-id="10da1-160">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="10da1-160">Common drivers could include:</span></span>

* <span data-ttu-id="10da1-161">Gyorsabb innováció</span><span class="sxs-lookup"><span data-stu-id="10da1-161">Accelerate innovation</span></span>
* <span data-ttu-id="10da1-162">Gyorsabb alkalmazáskészítés</span><span class="sxs-lookup"><span data-stu-id="10da1-162">Build apps faster</span></span>
* <span data-ttu-id="10da1-163">Működési költségek csökkentése</span><span class="sxs-lookup"><span data-stu-id="10da1-163">Reduce operational cost</span></span>

<span data-ttu-id="10da1-164">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="10da1-165">Az eszközintelligencia mérete (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="10da1-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="10da1-166">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="10da1-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="10da1-167">A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)</span><span class="sxs-lookup"><span data-stu-id="10da1-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="10da1-168">Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)</span><span class="sxs-lookup"><span data-stu-id="10da1-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="10da1-169">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="10da1-170">Végfelhasználói elégedettség elutasítása</span><span class="sxs-lookup"><span data-stu-id="10da1-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="10da1-171">Üzleti folyamatok korlátozott funkciók szerint</span><span class="sxs-lookup"><span data-stu-id="10da1-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="10da1-172">A lehetséges díj, a felhasználói élményt és a bevétel nyereség</span><span class="sxs-lookup"><span data-stu-id="10da1-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="10da1-173">Csere</span><span class="sxs-lookup"><span data-stu-id="10da1-173">Replace</span></span>

<span data-ttu-id="10da1-174">Megoldások általában a legjobb technológiával és rendelkezésre álló módszer használatával időben vannak megvalósítva.</span><span class="sxs-lookup"><span data-stu-id="10da1-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="10da1-175">Bizonyos esetekben az üzemeltetett alkalmazás szükséges funkciót összes szoftvert, mint a szoftverszolgáltatások (SaaS) alkalmazások is megfelelnek.</span><span class="sxs-lookup"><span data-stu-id="10da1-175">In some cases, Software as a Service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="10da1-176">Ezekben az esetekben egy számítási feladat sikerült kell nyár kerül piacra jövőbeli kell cserélni, hatékonyan eltávolítaná az átalakítási beavatkozást.</span><span class="sxs-lookup"><span data-stu-id="10da1-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="10da1-177">Közös illesztőprogramot lehetnek:</span><span class="sxs-lookup"><span data-stu-id="10da1-177">Common drivers could include:</span></span>

* <span data-ttu-id="10da1-178">Iparág-– gyakorlati tanácsok körül szabványosítása</span><span class="sxs-lookup"><span data-stu-id="10da1-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="10da1-179">Gyorsabb üzleti folyamat megközelítések driven elfogadása</span><span class="sxs-lookup"><span data-stu-id="10da1-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="10da1-180">Foglalja le újból a fejlesztési beruházások kiterjeszthetők az alkalmazásokat, amelyek versenyképes differenciálás vagy előnyeit létrehozása</span><span class="sxs-lookup"><span data-stu-id="10da1-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="10da1-181">A mennyiségi elemzés tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="10da1-182">Általános működési költségek csökkentése</span><span class="sxs-lookup"><span data-stu-id="10da1-182">General operating cost reductions</span></span>
* <span data-ttu-id="10da1-183">Virtuálisgép-méret (CPU, memória, tároló)</span><span class="sxs-lookup"><span data-stu-id="10da1-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="10da1-184">Függőségek (hálózati forgalom)</span><span class="sxs-lookup"><span data-stu-id="10da1-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="10da1-185">Eszközök megszűnik</span><span class="sxs-lookup"><span data-stu-id="10da1-185">Assets to be retired</span></span>

<span data-ttu-id="10da1-186">Minőségi elemzési tényezők:</span><span class="sxs-lookup"><span data-stu-id="10da1-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="10da1-187">A Cost haszon elemzése a jelenlegi architektúra vs Szolgáltatottszoftver-megoldás</span><span class="sxs-lookup"><span data-stu-id="10da1-187">Cost benefit analysis of current architecture vs SaaS solution</span></span>
* <span data-ttu-id="10da1-188">Üzleti folyamatok térképek</span><span class="sxs-lookup"><span data-stu-id="10da1-188">Business process maps</span></span>
* <span data-ttu-id="10da1-189">Adatsémák</span><span class="sxs-lookup"><span data-stu-id="10da1-189">Data schemas</span></span>
* <span data-ttu-id="10da1-190">Egyéni vagy automatizált folyamatokkal</span><span class="sxs-lookup"><span data-stu-id="10da1-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="10da1-191">További lépések</span><span class="sxs-lookup"><span data-stu-id="10da1-191">Next steps</span></span>

<span data-ttu-id="10da1-192">Együttesen ezek ésszerűsítés 5 Rs is alkalmazható egy digitális hagyatéki ésszerűsítés minden alkalmazás jövőbeli állapota kapcsolatos döntéseket.</span><span class="sxs-lookup"><span data-stu-id="10da1-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="10da1-193">Mi az a digial hagyatéki?</span><span class="sxs-lookup"><span data-stu-id="10da1-193">What is a digial estate?</span></span>](overview.md)