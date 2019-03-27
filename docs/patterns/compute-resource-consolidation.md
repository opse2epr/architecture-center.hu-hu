---
title: Számításierőforrás-konszolidálási minta
titleSuffix: Cloud Design Patterns
description: Egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 4cd9b7f02ba3b2a9766a2493353da6b6488ba8a2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299013"
---
# <a name="compute-resource-consolidation-pattern"></a><span data-ttu-id="86f76-104">Számításierőforrás-konszolidálási minta</span><span class="sxs-lookup"><span data-stu-id="86f76-104">Compute Resource Consolidation pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="86f76-105">Egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet.</span><span class="sxs-lookup"><span data-stu-id="86f76-105">Consolidate multiple tasks or operations into a single computational unit.</span></span> <span data-ttu-id="86f76-106">Ez megnövelheti a számítási erőforrások kihasználtságát, valamint csökkentheti a felhőben üzemeltetett alkalmazásokban végzett számítási feldolgozások végrehajtásához kapcsolódó költségeket és munkaterhelést.</span><span class="sxs-lookup"><span data-stu-id="86f76-106">This can increase compute resource utilization, and reduce the costs and management overhead associated with performing compute processing in cloud-hosted applications.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="86f76-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="86f76-107">Context and problem</span></span>

<span data-ttu-id="86f76-108">A felhőalkalmazások gyakran számos különböző műveletet végeznek.</span><span class="sxs-lookup"><span data-stu-id="86f76-108">A cloud application often implements a variety of operations.</span></span> <span data-ttu-id="86f76-109">Néhány megoldásban eleinte érdemes a kockázatok elkülönítésének tervezési alapelvét követni, és felosztani ezeket a műveleteket különálló számítási egységekre, amelyek üzembe helyezése és üzemeltetése egymástól független (például különálló App Service webalkalmazások, különálló virtuális gépek vagy különálló Cloud Service szerepkörök).</span><span class="sxs-lookup"><span data-stu-id="86f76-109">In some solutions it makes sense to follow the design principle of separation of concerns initially, and divide these operations into separate computational units that are hosted and deployed individually (for example, as separate App Service web apps, separate Virtual Machines, or separate Cloud Service roles).</span></span> <span data-ttu-id="86f76-110">Bár ez a stratégia segíthet leegyszerűsíteni a megoldás logikai kialakítását, ha több számítási egységet helyez üzembe ugyanannak az alkalmazásnak a részeként, az megnövelheti a futtatókörnyezet üzemeltetési költségeit, illetve a rendszer kezelését bonyolultabbá teheti.</span><span class="sxs-lookup"><span data-stu-id="86f76-110">However, although this strategy can help simplify the logical design of the solution, deploying a large number of computational units as part of the same application can increase runtime hosting costs and make management of the system more complex.</span></span>

<span data-ttu-id="86f76-111">Például az ábrán egy olyan, felhőben üzemeltetett megoldás egyszerűsített struktúrája látható, amely több mint egy számítási egység használatával lett megvalósítva.</span><span class="sxs-lookup"><span data-stu-id="86f76-111">As an example, the figure shows the simplified structure of a cloud-hosted solution that is implemented using more than one computational unit.</span></span> <span data-ttu-id="86f76-112">Mindegyik számítási egység a saját virtuális környezetét futtatja.</span><span class="sxs-lookup"><span data-stu-id="86f76-112">Each computational unit runs in its own virtual environment.</span></span> <span data-ttu-id="86f76-113">Mindegyik funkció külön feladatként lett megvalósítva (A–E feladat), amely a saját számítási egységét futtatja.</span><span class="sxs-lookup"><span data-stu-id="86f76-113">Each function has been implemented as a separate task (labeled Task A through Task E) running in its own computational unit.</span></span>

![Feladatok futtatása egy felhőkörnyezetben dedikált számítási egységek készletének használatával](./_images/compute-resource-consolidation-diagram.png)

<span data-ttu-id="86f76-115">Mindegyik számítási egység felszámítható erőforrásokat használ, akkor is, ha tétlen vagy kisebb terhelésű.</span><span class="sxs-lookup"><span data-stu-id="86f76-115">Each computational unit consumes chargeable resources, even when it's idle or lightly used.</span></span> <span data-ttu-id="86f76-116">Ezért ez nem mindig a legköltséghatékonyabb megoldás.</span><span class="sxs-lookup"><span data-stu-id="86f76-116">Therefore, this isn't always the most cost-effective solution.</span></span>

<span data-ttu-id="86f76-117">Az Azure-ban ez a probléma a Cloud Service, az App Services és a Virtual Machines szerepköreire vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="86f76-117">In Azure, this concern applies to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="86f76-118">Ezek az elemek a saját virtuális környezetükben futnak.</span><span class="sxs-lookup"><span data-stu-id="86f76-118">These items run in their own virtual environment.</span></span> <span data-ttu-id="86f76-119">Ha olyan különálló szerepkörök, webhelyek vagy virtuális gépek összességét futtatja, amelyek jól körülírt műveletek egy készletének végrehajtására lettek tervezve, azonban egy megoldás részeiként kell kommunikálniuk és együttműködniük, lehetséges, hogy erőforrásait nem hatékonyan használja fel.</span><span class="sxs-lookup"><span data-stu-id="86f76-119">Running a collection of separate roles, websites, or virtual machines that are designed to perform a set of well-defined operations, but that need to communicate and cooperate as part of a single solution, can be an inefficient use of resources.</span></span>

## <a name="solution"></a><span data-ttu-id="86f76-120">Megoldás</span><span class="sxs-lookup"><span data-stu-id="86f76-120">Solution</span></span>

<span data-ttu-id="86f76-121">A költségek csökkentése, a kihasználtság optimalizálása, a kommunikációs sebesség javítása és a kezelési szükségletek csökkentése érdekében egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet.</span><span class="sxs-lookup"><span data-stu-id="86f76-121">To help reduce costs, increase utilization, improve communication speed, and reduce management it's possible to consolidate multiple tasks or operations into a single computational unit.</span></span>

<span data-ttu-id="86f76-122">A feladatokat a környezet által biztosított funkciók és a funkciókhoz kapcsolódó költségek szerint megállapított kritériumok alapján csoportosíthatja.</span><span class="sxs-lookup"><span data-stu-id="86f76-122">Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features.</span></span> <span data-ttu-id="86f76-123">Általánosan használt megközelítés olyan feladatok keresése, amelyek hasonló profillal rendelkeznek a skálázhatóságuk, élettartamuk és feldolgozási követelményeik szempontjából.</span><span class="sxs-lookup"><span data-stu-id="86f76-123">A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements.</span></span> <span data-ttu-id="86f76-124">A csoportosításukkal egy egységként skálázhatja őket.</span><span class="sxs-lookup"><span data-stu-id="86f76-124">Grouping these together allows them to scale as a unit.</span></span> <span data-ttu-id="86f76-125">A több felhőkörnyezet által nyújtott rugalmasság lehetővé teszi, hogy a számítási feladattól függően egy számítási egység további példányait indítsa el vagy állítsa le.</span><span class="sxs-lookup"><span data-stu-id="86f76-125">The elasticity provided by many cloud environments enables additional instances of a computational unit to be started and stopped according to the workload.</span></span> <span data-ttu-id="86f76-126">Például az Azure használatával automatikusan skálázhatja a Cloud Service, az App Services vagy a Virtual Machines szerepköreit.</span><span class="sxs-lookup"><span data-stu-id="86f76-126">For example, Azure provides autoscaling that you can apply to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="86f76-127">További információért lásd az [automatikus skálázás útmutatóját](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="86f76-127">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span>

<span data-ttu-id="86f76-128">Az alábbi két feladat segítségével ellenpéldaként bemutatjuk, hogyan állapítható meg a skálázhatósággal, hogy mely műveleteket nem érdemes csoportosítanunk:</span><span class="sxs-lookup"><span data-stu-id="86f76-128">As a counter example to show how scalability can be used to determine which operations shouldn't be grouped together, consider the following two tasks:</span></span>

- <span data-ttu-id="86f76-129">Az 1. feladat az üzenetsorba küldött ritka, nem időérzékeny üzeneteket kérdezi le.</span><span class="sxs-lookup"><span data-stu-id="86f76-129">Task 1 polls for infrequent, time-insensitive messages sent to a queue.</span></span>
- <span data-ttu-id="86f76-130">A 2. feladat a hálózati forgalom csúcsterheléseit kezeli.</span><span class="sxs-lookup"><span data-stu-id="86f76-130">Task 2 handles high-volume bursts of network traffic.</span></span>

<span data-ttu-id="86f76-131">A második feladathoz rugalmasság szükséges, amelyhez hozzátartozik a számítási egység számos példányának elindítása és leállítása is.</span><span class="sxs-lookup"><span data-stu-id="86f76-131">The second task requires elasticity that can involve starting and stopping a large number of instances of the computational unit.</span></span> <span data-ttu-id="86f76-132">Amennyiben ugyanezt a skálázást alkalmazná az első feladatra, az egyszerűen azt eredményezné, hogy több feladat figyelne ritka üzenetekre ugyanabban az üzenetsorban, így feleslegesen használna erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="86f76-132">Applying the same scaling to the first task would simply result in more tasks listening for infrequent messages on the same queue, and is a waste of resources.</span></span>

<span data-ttu-id="86f76-133">Számos felhőkörnyezetben a számítási egységek számára elérhető erőforrások meghatározhatók a processzormagok számával, a memória és a lemezterület méretével, illetve egyéb értékekkel.</span><span class="sxs-lookup"><span data-stu-id="86f76-133">In many cloud environments it's possible to specify the resources available to a computational unit in terms of the number of CPU cores, memory, disk space, and so on.</span></span> <span data-ttu-id="86f76-134">Általában minél több erőforrás van meghatározva, annál nagyobb a költség is.</span><span class="sxs-lookup"><span data-stu-id="86f76-134">Generally, the more resources specified, the greater the cost.</span></span> <span data-ttu-id="86f76-135">Pénz megtakarításához fontos maximalizálni a költséges számítási egységek által végrehajtott munkát, illetve biztosítani, hogy ne legyen hosszabb időre inaktív.</span><span class="sxs-lookup"><span data-stu-id="86f76-135">To save money, it's important to maximize the work an expensive computational unit performs, and not let it become inactive for an extended period.</span></span>

<span data-ttu-id="86f76-136">Ha előfordulnak olyan feladatok, amelyeknek jelentős processzorteljesítményre van szükségük rövidebb időintervallumokban, fontolja meg ezek konszolidálását egy számítási egységbe, amely biztosítja a szükséges teljesítményt.</span><span class="sxs-lookup"><span data-stu-id="86f76-136">If there are tasks that require a great deal of CPU power in short bursts, consider consolidating these into a single computational unit that provides the necessary power.</span></span> <span data-ttu-id="86f76-137">Fontos azonban megtalálni az egyensúlyt a költséges erőforrások folyamatos működtetése és a túlterhelés által előidézett versengés között.</span><span class="sxs-lookup"><span data-stu-id="86f76-137">However, it's important to balance this need to keep expensive resources busy against the contention that could occur if they are over stressed.</span></span> <span data-ttu-id="86f76-138">A hosszan futó, nagy számítási igényű feladatok például ne osztozzanak ugyanazon a számítási egységen.</span><span class="sxs-lookup"><span data-stu-id="86f76-138">Long-running, compute-intensive tasks shouldn't share the same computational unit, for example.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="86f76-139">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="86f76-139">Issues and considerations</span></span>

<span data-ttu-id="86f76-140">Vegye figyelembe az alábbi pontokat a minta megvalósításakor:</span><span class="sxs-lookup"><span data-stu-id="86f76-140">Consider the following points when implementing this pattern:</span></span>

<span data-ttu-id="86f76-141">**Méretezhetőség és rugalmasság**.</span><span class="sxs-lookup"><span data-stu-id="86f76-141">**Scalability and elasticity**.</span></span> <span data-ttu-id="86f76-142">Számos felhőalapú megoldás a számítási egység szintjén a skálázhatóságot és a rugalmasságot az egységek példányainak elindításával és leállításával valósítja meg.</span><span class="sxs-lookup"><span data-stu-id="86f76-142">Many cloud solutions implement scalability and elasticity at the level of the computational unit by starting and stopping instances of units.</span></span> <span data-ttu-id="86f76-143">Ne csoportosítsa azokat a feladatokat egy számítási egységbe, amelyek skálázási követelményei ütköznek.</span><span class="sxs-lookup"><span data-stu-id="86f76-143">Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.</span></span>

<span data-ttu-id="86f76-144">**Élettartam**.</span><span class="sxs-lookup"><span data-stu-id="86f76-144">**Lifetime**.</span></span> <span data-ttu-id="86f76-145">A felhőalapú infrastruktúra rendszeres időközönként újraindítja a számítási egységeket üzemeltető virtuális környezeteket.</span><span class="sxs-lookup"><span data-stu-id="86f76-145">The cloud infrastructure periodically recycles the virtual environment that hosts a computational unit.</span></span> <span data-ttu-id="86f76-146">Ha számos hosszú lefutású feladat található egy számítási egységben, szükséges lehet úgy konfigurálni az egységet, hogy megakadályozza az újraindítását, míg ezek a feladatok be nem fejeződnek.</span><span class="sxs-lookup"><span data-stu-id="86f76-146">When there are many long-running tasks inside a computational unit, it might be necessary to configure the unit to prevent it from being recycled until these tasks have finished.</span></span> <span data-ttu-id="86f76-147">Másik megoldásként a feladatokat ellenőrzőpontos megközelítéssel is megtervezheti, amely lehetővé teszi, hogy megfelelően leálljanak, majd arról a pontról folytatódjanak, ahol az adott folyamat megszakadt a számítási egység újraindításakor.</span><span class="sxs-lookup"><span data-stu-id="86f76-147">Alternatively, design the tasks by using a check-pointing approach that enables them to stop cleanly, and continue at the point they were interrupted when the computational unit is restarted.</span></span>

<span data-ttu-id="86f76-148">**Kiadási ütem**.</span><span class="sxs-lookup"><span data-stu-id="86f76-148">**Release cadence**.</span></span> <span data-ttu-id="86f76-149">Ha egy feladat megvalósítása vagy konfigurációja gyakran változik, szükséges lehet a frissített kódot üzemeltető számítási egység leállítása, az egység újrakonfigurálása és újbóli üzembe helyezése, majd újraindítása.</span><span class="sxs-lookup"><span data-stu-id="86f76-149">If the implementation or configuration of a task changes frequently, it might be necessary to stop the computational unit hosting the updated code, reconfigure and redeploy the unit, and then restart it.</span></span> <span data-ttu-id="86f76-150">Ehhez a folyamathoz a számítási egységben futtatott összes egyéb feladatot is le kell állítani, újból üzembe kell helyezni, majd újra kell indítani.</span><span class="sxs-lookup"><span data-stu-id="86f76-150">This process will also require that all other tasks within the same computational unit are stopped, redeployed, and restarted.</span></span>

<span data-ttu-id="86f76-151">**Biztonság**.</span><span class="sxs-lookup"><span data-stu-id="86f76-151">**Security**.</span></span> <span data-ttu-id="86f76-152">Az azonos számítási egységekben futtatott feladatok közös biztonsági környezettel rendelkezhetnek, illetve ugyanazokhoz az erőforrásokhoz férhetnek hozzá.</span><span class="sxs-lookup"><span data-stu-id="86f76-152">Tasks in the same computational unit might share the same security context and be able to access the same resources.</span></span> <span data-ttu-id="86f76-153">A feladatoknak magas szintű megbízhatósági kapcsolatban kell lenniük, valamint meg kell bizonyosodniuk arról, hogy az egyik feladat nem fogja károsítani vagy negatívan befolyásolni a másikat.</span><span class="sxs-lookup"><span data-stu-id="86f76-153">There must be a high degree of trust between the tasks, and confidence that one task isn't going to corrupt or adversely affect another.</span></span> <span data-ttu-id="86f76-154">Emellett egy adott számítási egységben futtatott feladatok számának emelése megnöveli az egység támadható felületét.</span><span class="sxs-lookup"><span data-stu-id="86f76-154">Additionally, increasing the number of tasks running in a computational unit increases the attack surface of the unit.</span></span> <span data-ttu-id="86f76-155">Ha egy feladat nem biztonságos, az a többi feladat biztonságát is rontja.</span><span class="sxs-lookup"><span data-stu-id="86f76-155">Each task is only as secure as the one with the most vulnerabilities.</span></span>

<span data-ttu-id="86f76-156">**Hibatűrés**.</span><span class="sxs-lookup"><span data-stu-id="86f76-156">**Fault tolerance**.</span></span> <span data-ttu-id="86f76-157">Ha egy feladat a számítási egységben meghibásodik vagy rendellenesen viselkedik, az befolyásolhatja az azonos egységen futtatott egyéb feladatokat is.</span><span class="sxs-lookup"><span data-stu-id="86f76-157">If one task in a computational unit fails or behaves abnormally, it can affect the other tasks running within the same unit.</span></span> <span data-ttu-id="86f76-158">Ha például egy feladat képtelen megfelelően elindulni, az a számítási egység teljes indítási logikájának meghibásodásához vezethet, megakadályozva az egységen a többi feladat futtatását is.</span><span class="sxs-lookup"><span data-stu-id="86f76-158">For example, if one task fails to start correctly it can cause the entire startup logic for the computational unit to fail, and prevent other tasks in the same unit from running.</span></span>

<span data-ttu-id="86f76-159">**Versengés**.</span><span class="sxs-lookup"><span data-stu-id="86f76-159">**Contention**.</span></span> <span data-ttu-id="86f76-160">Ne idézzen elő versengést olyan feladatok létrehozásával, amelyek egy adott számítási egységben osztoznak az erőforrásokon.</span><span class="sxs-lookup"><span data-stu-id="86f76-160">Avoid introducing contention between tasks that compete for resources in the same computational unit.</span></span> <span data-ttu-id="86f76-161">Ideális esetben az egy számítási egységben futtatott feladatok különböző erőforrás-felhasználási jellemzőkkel rendelkeznek.</span><span class="sxs-lookup"><span data-stu-id="86f76-161">Ideally, tasks that share the same computational unit should exhibit different resource utilization characteristics.</span></span> <span data-ttu-id="86f76-162">Ne futtasson például két nagy számítási igényű feladatot ugyanabban a számítási egységben, valamint két sok memóriát felhasználó feladatot se.</span><span class="sxs-lookup"><span data-stu-id="86f76-162">For example, two compute-intensive tasks should probably not reside in the same computational unit, and neither should two tasks that consume large amounts of memory.</span></span> <span data-ttu-id="86f76-163">Azonban egy nagy számítási igényű feladat és egy sok memóriát igénylő feladat közös üzemeltetése működőképes kombináció.</span><span class="sxs-lookup"><span data-stu-id="86f76-163">However, mixing a compute intensive task with a task that requires a large amount of memory is a workable combination.</span></span>

> [!NOTE]
> <span data-ttu-id="86f76-164">Próbálja csak olyan rendszerek számítási erőforrásait konszolidálni, amelyek már egy ideje éles környezetben üzemelnek, hogy a kezelők és fejlesztők a rendszer monitorozásával létrehozhassanak egy _intenzitástérképet_, amely azonosítja, hogy az egyes feladatok hogyan használják a különböző erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="86f76-164">Consider consolidating compute resources only for a system that's been in production for a period of time so that operators and developers can monitor the system and create a _heat map_ that identifies how each task utilizes differing resources.</span></span> <span data-ttu-id="86f76-165">A térkép segítségével megállapíthatja, hogy mely feladatok között érdemes megosztani a számítási egységeket.</span><span class="sxs-lookup"><span data-stu-id="86f76-165">This map can be used to determine which tasks are good candidates for sharing compute resources.</span></span>

<span data-ttu-id="86f76-166">**Összetettség**.</span><span class="sxs-lookup"><span data-stu-id="86f76-166">**Complexity**.</span></span> <span data-ttu-id="86f76-167">Amennyiben több feladatot kombinál egy számítási egységben, az megnöveli a kód összetettségét, ami esetleg megnehezítheti a tesztelést, a hibakeresést és a karbantartást.</span><span class="sxs-lookup"><span data-stu-id="86f76-167">Combining multiple tasks into a single computational unit adds complexity to the code in the unit, possibly making it more difficult to test, debug, and maintain.</span></span>

<span data-ttu-id="86f76-168">**Stabil logikai architektúra**.</span><span class="sxs-lookup"><span data-stu-id="86f76-168">**Stable logical architecture**.</span></span> <span data-ttu-id="86f76-169">A kódot úgy tervezze és valósítsa meg az egyes feladatokban, hogy akkor se kelljen módosítani, ha a feladatot futtató fizikai környezet megváltozik.</span><span class="sxs-lookup"><span data-stu-id="86f76-169">Design and implement the code in each task so that it shouldn't need to change, even if the physical environment the task runs in does change.</span></span>

<span data-ttu-id="86f76-170">**Egyéb stratégiák**.</span><span class="sxs-lookup"><span data-stu-id="86f76-170">**Other strategies**.</span></span> <span data-ttu-id="86f76-171">A számítási erőforrások konszolidálása csak egy módja a több feladat egyidejű futtatásával járó költségek csökkentésének.</span><span class="sxs-lookup"><span data-stu-id="86f76-171">Consolidating compute resources is only one way to help reduce costs associated with running multiple tasks concurrently.</span></span> <span data-ttu-id="86f76-172">Csak gondos tervezéssel és monitorozással biztosítható, hogy hatékony megközelítés maradjon.</span><span class="sxs-lookup"><span data-stu-id="86f76-172">It requires careful planning and monitoring to ensure that it remains an effective approach.</span></span> <span data-ttu-id="86f76-173">A munka természetétől és a feladatokat futtató felhasználók tartózkodási helyétől függően egyéb stratégiák megfelelőbbek lehetnek.</span><span class="sxs-lookup"><span data-stu-id="86f76-173">Other strategies might be more appropriate, depending on the nature of the work and where the users these tasks are running are located.</span></span> <span data-ttu-id="86f76-174">A számítási feladatok funkcionális felbontása (a [Compute-particionálási útmutatónak](https://msdn.microsoft.com/library/dn589773.aspx) megfelelően) például jobb megoldás lehet.</span><span class="sxs-lookup"><span data-stu-id="86f76-174">For example, functional decomposition of the workload (as described by the [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx)) might be a better option.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="86f76-175">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="86f76-175">When to use this pattern</span></span>

<span data-ttu-id="86f76-176">Ezt a mintát olyan feladatok esetében alkalmazza, amelyek nem költséghatékonyak, ha a saját számítási egységeikben futnak.</span><span class="sxs-lookup"><span data-stu-id="86f76-176">Use this pattern for tasks that are not cost effective if they run in their own computational units.</span></span> <span data-ttu-id="86f76-177">Ha egy feladat az idő nagy részében tétlen, a feladat futtatása egy dedikált egységen költséges lehet.</span><span class="sxs-lookup"><span data-stu-id="86f76-177">If a task spends much of its time idle, running this task in a dedicated unit can be expensive.</span></span>

<span data-ttu-id="86f76-178">Ez a minta lehet, hogy nem megfelelő olyan feladatokhoz, amelyek kritikus hibatűrő műveleteket hajtanak végre, illetve rendkívül bizalmas vagy személyes adatokat dolgoznak fel, és saját biztonsági környezetre van szükségük.</span><span class="sxs-lookup"><span data-stu-id="86f76-178">This pattern might not be suitable for tasks that perform critical fault-tolerant operations, or tasks that process highly sensitive or private data and require their own security context.</span></span> <span data-ttu-id="86f76-179">Ezeket a feladatokat a saját elkülönített környezetükben érdemes futtatni, különálló számítási egységben.</span><span class="sxs-lookup"><span data-stu-id="86f76-179">These tasks should run in their own isolated environment, in a separate computational unit.</span></span>

## <a name="example"></a><span data-ttu-id="86f76-180">Példa</span><span class="sxs-lookup"><span data-stu-id="86f76-180">Example</span></span>

<span data-ttu-id="86f76-181">Amikor felhőszolgáltatást fejleszt az Azure-ban, a több feladat által végzett feldolgozást egyetlen szerepkörbe konszolidálhatja.</span><span class="sxs-lookup"><span data-stu-id="86f76-181">When building a cloud service on Azure, it’s possible to consolidate the processing performed by multiple tasks into a single role.</span></span> <span data-ttu-id="86f76-182">Általában ez egy feldolgozói szerepkör, amely háttérben történő vagy aszinkron feldolgozási feladatokat végez.</span><span class="sxs-lookup"><span data-stu-id="86f76-182">Typically this is a worker role that performs background or asynchronous processing tasks.</span></span>

> <span data-ttu-id="86f76-183">Néhány esetben a háttérben történő vagy aszinkron feldolgozási feladatokat belefoglalhatja a webes szerepkörbe.</span><span class="sxs-lookup"><span data-stu-id="86f76-183">In some cases it's possible to include background or asynchronous processing tasks in the web role.</span></span> <span data-ttu-id="86f76-184">Ezzel a módszerrel csökkentheti a költségeket és egyszerűsítheti az üzembe helyezést, azonban ez befolyásolhatja a webes szerepkör által biztosított nyilvános felület skálázhatóságát és válaszképességét.</span><span class="sxs-lookup"><span data-stu-id="86f76-184">This technique helps to reduce costs and simplify deployment, although it can impact the scalability and responsiveness of the public-facing interface provided by the web role.</span></span>

<span data-ttu-id="86f76-185">Ez a szerepkör felelős a feladatok indításáért és leállításáért.</span><span class="sxs-lookup"><span data-stu-id="86f76-185">The role is responsible for starting and stopping the tasks.</span></span> <span data-ttu-id="86f76-186">Amikor az Azure Fabric Controller betölt egy szerepkört, kiváltja a szerepkör `Start` eseményét.</span><span class="sxs-lookup"><span data-stu-id="86f76-186">When the Azure fabric controller loads a role, it raises the `Start` event for the role.</span></span> <span data-ttu-id="86f76-187">Felülírhatja a `WebRole` vagy `WorkerRole` osztály `OnStart` metódusát, hogy kezelje az eseményt, például azoknak az adatoknak és egyéb erőforrásoknak az inicializálásával, amelyektől a metódus feladatai függnek.</span><span class="sxs-lookup"><span data-stu-id="86f76-187">You can override the `OnStart` method of the `WebRole` or `WorkerRole` class to handle this event, perhaps to initialize the data and other resources the tasks in this method depend on.</span></span>

<span data-ttu-id="86f76-188">Ha a `OnStart` metódus befejeződik, a szerepkör elkezdhet válaszolni a kérésekre.</span><span class="sxs-lookup"><span data-stu-id="86f76-188">When the `OnStart` method completes, the role can start responding to requests.</span></span> <span data-ttu-id="86f76-189">Az `OnStart` és a `Run` metódus szerepkörben történő használatáról további információkat és útmutatást az [alkalmazások felhőbe való áthelyezését tárgyaló](https://msdn.microsoft.com/library/ff728592.aspx) minta- és gyakorlati útmutató [alkalmazásindítási folyamatokról szóló](https://msdn.microsoft.com/library/ff803371.aspx#sec16) szakaszában talál.</span><span class="sxs-lookup"><span data-stu-id="86f76-189">You can find more information and guidance about using the `OnStart` and `Run` methods in a role in the [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) section in the patterns & practices guide [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx).</span></span>

> <span data-ttu-id="86f76-190">Az `OnStart` metódus kódja legyen a lehető legtömörebb.</span><span class="sxs-lookup"><span data-stu-id="86f76-190">Keep the code in the `OnStart` method as concise as possible.</span></span> <span data-ttu-id="86f76-191">Az Azure nem határoz meg küszöbértéket a metódus befejezésére engedélyezett időtartamnak, azonban a szerepkör nem kezdi megválaszolni az elküldött hálózati kéréseket, amíg a metódus be nem fejeződik.</span><span class="sxs-lookup"><span data-stu-id="86f76-191">Azure doesn't impose any limit on the time taken for this method to complete, but the role won't be able to start responding to network requests sent to it until this method completes.</span></span>

<span data-ttu-id="86f76-192">Ha az `OnStart` metódus befejeződött, a szerepkör végrehajtja a `Run` metódust.</span><span class="sxs-lookup"><span data-stu-id="86f76-192">When the `OnStart` method has finished, the role executes the `Run` method.</span></span> <span data-ttu-id="86f76-193">Ekkor a hálóvezérlő megkezdi a kérések küldését a szerepkörnek.</span><span class="sxs-lookup"><span data-stu-id="86f76-193">At this point, the fabric controller can start sending requests to the role.</span></span>

<span data-ttu-id="86f76-194">Helyezze a feladatokat ténylegesen létrehozó kódot a `Run` metódusba.</span><span class="sxs-lookup"><span data-stu-id="86f76-194">Place the code that actually creates the tasks in the `Run` method.</span></span> <span data-ttu-id="86f76-195">Vegye figyelembe, hogy a `Run` metódus határozza meg a szerepkörpéldány élettartamát.</span><span class="sxs-lookup"><span data-stu-id="86f76-195">Note that the `Run` method defines the lifetime of the role instance.</span></span> <span data-ttu-id="86f76-196">A metódus befejezésekor a hálóvezérlő kezdeményezi a szerepkör leállítását.</span><span class="sxs-lookup"><span data-stu-id="86f76-196">When this method completes, the fabric controller will arrange for the role to be shut down.</span></span>

<span data-ttu-id="86f76-197">Amikor egy szerepkör leáll vagy újraindul, a hálóvezérlő megakadályozza, hogy további bejövő kérések érkezzenek a terheléselosztóból, és kiváltja a `Stop` eseményt.</span><span class="sxs-lookup"><span data-stu-id="86f76-197">When a role shuts down or is recycled, the fabric controller prevents any more incoming requests being received from the load balancer and raises the `Stop` event.</span></span> <span data-ttu-id="86f76-198">Ezt az eseményt a szerepkör `OnStop` metódusának felülbírálásával rögzítheti, majd hajtsa végre a szükséges tisztítási műveleteket a szerepkör leállítása előtt.</span><span class="sxs-lookup"><span data-stu-id="86f76-198">You can capture this event by overriding the `OnStop` method of the role and perform any tidying up required before the role terminates.</span></span>

> <span data-ttu-id="86f76-199">Az `OnStop` metódusban végrehajtott összes műveletet öt percen belül be kell fejezni (vagy 30 másodpercen belül, ha Azure emulátort használt helyi számítógépen).</span><span class="sxs-lookup"><span data-stu-id="86f76-199">Any actions performed in the `OnStop` method must be completed within five minutes (or 30 seconds if you are using the Azure emulator on a local computer).</span></span> <span data-ttu-id="86f76-200">Ellenkező esetben az Azure Fabric Controller azt feltételezi, hogy a szerepkör elakadt, és leállásra kényszeríti.</span><span class="sxs-lookup"><span data-stu-id="86f76-200">Otherwise the Azure fabric controller assumes that the role has stalled and will force it to stop.</span></span>

<span data-ttu-id="86f76-201">A feladatokat a `Run` metódus indítja el, amely megvárja a feladatok befejezését.</span><span class="sxs-lookup"><span data-stu-id="86f76-201">The tasks are started by the `Run` method that waits for the tasks to complete.</span></span> <span data-ttu-id="86f76-202">A feladatok megvalósítják a felhőszolgáltatás üzleti logikáját, és a szerepkörnek küldött üzeneteket az Azure Load Balanceren keresztül válaszolják meg.</span><span class="sxs-lookup"><span data-stu-id="86f76-202">The tasks implement the business logic of the cloud service, and can respond to messages posted to the role through the Azure load balancer.</span></span> <span data-ttu-id="86f76-203">Az ábrán egy Azure-felhőszolgáltatás szerepköréhez tartozó feladatok és erőforrások életciklusa látható.</span><span class="sxs-lookup"><span data-stu-id="86f76-203">The figure shows the lifecycle of tasks and resources in a role in an Azure cloud service.</span></span>

![Egy Azure-felhőszolgáltatás szerepköréhez tartozó feladatok és erőforrások életciklusa](./_images/compute-resource-consolidation-lifecycle.png)

<span data-ttu-id="86f76-205">A _WorkerRole.cs_ fájl a _ComputeResourceConsolidation.Worker_ projektben arra mutat be egy példát, hogyan valósíthatja meg ezt a mintát egy Azure felhőszolgáltatásban.</span><span class="sxs-lookup"><span data-stu-id="86f76-205">The _WorkerRole.cs_ file in the _ComputeResourceConsolidation.Worker_ project shows an example of how you might implement this pattern in an Azure cloud service.</span></span>

> <span data-ttu-id="86f76-206">A _ComputeResourceConsolidation.Worker_ projekt a _ComputeResourceConsolidation_ megoldás része, amely letölthető a [GitHubról](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span><span class="sxs-lookup"><span data-stu-id="86f76-206">The _ComputeResourceConsolidation.Worker_ project is part of the _ComputeResourceConsolidation_ solution available for download from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>

<span data-ttu-id="86f76-207">A `MyWorkerTask1` és a `MyWorkerTask2` metódus bemutatja, hogyan hajthat végre különböző feladatokat ugyanabban a feldolgozói szerepkörben.</span><span class="sxs-lookup"><span data-stu-id="86f76-207">The `MyWorkerTask1` and the `MyWorkerTask2` methods illustrate how to perform different tasks within the same worker role.</span></span> <span data-ttu-id="86f76-208">Az alábbi kódban a `MyWorkerTask1` látható.</span><span class="sxs-lookup"><span data-stu-id="86f76-208">The following code shows `MyWorkerTask1`.</span></span> <span data-ttu-id="86f76-209">Ez egy egyszerű feladat, amely 30 másodpercig alvó állapotba lép, majd elküld egy nyomkövetési üzenetet.</span><span class="sxs-lookup"><span data-stu-id="86f76-209">This is a simple task that sleeps for 30 seconds and then outputs a trace message.</span></span> <span data-ttu-id="86f76-210">Ez a folyamat a feladat megszakításáig ismétlődik.</span><span class="sxs-lookup"><span data-stu-id="86f76-210">It repeats this process until the task is canceled.</span></span> <span data-ttu-id="86f76-211">A `MyWorkerTask2` kódja hasonló.</span><span class="sxs-lookup"><span data-stu-id="86f76-211">The code in `MyWorkerTask2` is similar.</span></span>

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> <span data-ttu-id="86f76-212">A mintakód a háttérfolyamatok egy gyakori megvalósítását mutatja be.</span><span class="sxs-lookup"><span data-stu-id="86f76-212">The sample code shows a common implementation of a background process.</span></span> <span data-ttu-id="86f76-213">Egy valós alkalmazásban követheti ugyanezt a struktúrát, azonban helyezze a saját feldolgozási logikáját a megszakítási kérelemre váró hurok törzsébe.</span><span class="sxs-lookup"><span data-stu-id="86f76-213">In a real world application you can follow this same structure, except that you should place your own processing logic in the body of the loop that waits for the cancellation request.</span></span>

<span data-ttu-id="86f76-214">Miután a feldolgozói szerepkör inicializálta a felhasználni kívánt erőforrásokat, a `Run` metódus egyszerre elindítja a két feladatot az itt látható módon.</span><span class="sxs-lookup"><span data-stu-id="86f76-214">After the worker role has initialized the resources it uses, the `Run` method starts the two tasks concurrently, as shown here.</span></span>

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

<span data-ttu-id="86f76-215">Ebben a példában a `Run` metódus a feladatok befejezésére vár.</span><span class="sxs-lookup"><span data-stu-id="86f76-215">In this example, the `Run` method waits for tasks to be completed.</span></span> <span data-ttu-id="86f76-216">Ha a feladat meg lett szakítva, a `Run` metódus azt feltételezi, hogy a szerepkör leáll, és megvárja a hátralévő feladatok megszakítását a befejezés előtt (legfeljebb öt percet vár a leállításra).</span><span class="sxs-lookup"><span data-stu-id="86f76-216">If a task is canceled, the `Run` method assumes that the role is being shut down and waits for the remaining tasks to be canceled before finishing (it waits for a maximum of five minutes before terminating).</span></span> <span data-ttu-id="86f76-217">Ha a feladat egy várt kivétel miatt hiúsul meg, a `Run` metódus megszakítja a feladatot.</span><span class="sxs-lookup"><span data-stu-id="86f76-217">If a task fails due to an expected exception, the `Run` method cancels the task.</span></span>

> <span data-ttu-id="86f76-218">Olyan átfogóbb monitorozási és kivételkezelési stratégiákat is megvalósíthat a `Run` metódusban, mint például a meghiúsult feladatok újraindítása, illetve olyan kód beillesztése, amely engedélyezi a szerepkör számára egyes feladatok leállítását és elindítását.</span><span class="sxs-lookup"><span data-stu-id="86f76-218">You could implement more comprehensive monitoring and exception handling strategies in the `Run` method such as restarting tasks that have failed, or including code that enables the role to stop and start individual tasks.</span></span>

<span data-ttu-id="86f76-219">A következő kódban látható `Stop` metódust a rendszer akkor hívja meg, amikor a hálóvezérlő leállítja a szerepkörpéldányt (az `OnStop` metódusból indítva).</span><span class="sxs-lookup"><span data-stu-id="86f76-219">The `Stop` method shown in the following code is called when the fabric controller shuts down the role instance (it's invoked from the `OnStop` method).</span></span> <span data-ttu-id="86f76-220">A kód minden feladatot szabályosan állít le azok megszakításával.</span><span class="sxs-lookup"><span data-stu-id="86f76-220">The code stops each task gracefully by canceling it.</span></span> <span data-ttu-id="86f76-221">Ha a feladat befejezése több mint öt percet vesz igénybe, a `Stop` metódus megszakítási feldolgozása nem vár tovább, és a szerepkört leállítja.</span><span class="sxs-lookup"><span data-stu-id="86f76-221">If any task takes more than five minutes to complete, the cancellation processing in the `Stop` method ceases waiting and the role is terminated.</span></span>

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="86f76-222">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="86f76-222">Related patterns and guidance</span></span>

<span data-ttu-id="86f76-223">Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:</span><span class="sxs-lookup"><span data-stu-id="86f76-223">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="86f76-224">[Útmutató az automatikus skálázáshoz](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="86f76-224">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="86f76-225">Az automatikus skálázás használatával szolgáltatásokat üzemeltető számítási erőforrásokat indíthat el vagy állíthat le a várható feldolgozási igényektől függően.</span><span class="sxs-lookup"><span data-stu-id="86f76-225">Autoscaling can be used to start and stop instances of service hosting computational resources, depending on the anticipated demand for processing.</span></span>

- <span data-ttu-id="86f76-226">[Compute-particionálási útmutató](https://msdn.microsoft.com/library/dn589773.aspx)</span><span class="sxs-lookup"><span data-stu-id="86f76-226">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="86f76-227">Bemutatja, hogyan foglalhat le szolgáltatásokat és összetevőket egy felhőszolgáltatásban olyan módon, hogy az segítsen minimalizálni a futtatási költségeket a szolgáltatás skálázhatóságának, teljesítményének, rendelkezésre állásának és biztonságának megőrzése mellett.</span><span class="sxs-lookup"><span data-stu-id="86f76-227">Describes how to allocate the services and components in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>

- <span data-ttu-id="86f76-228">Ez a minta egy letölthető [mintaalkalmazást](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="86f76-228">This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>
