---
title: Számítási erőforrás összevonása
description: Több feladatokat vagy műveleteket egyetlen számítási egységbe összesítése
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
ms.openlocfilehash: 85191fc630549559f8a1395e5a8622a7a6140a2d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="compute-resource-consolidation-pattern"></a><span data-ttu-id="72984-104">Számítási erőforrás-összevonási minta</span><span class="sxs-lookup"><span data-stu-id="72984-104">Compute Resource Consolidation pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="72984-105">Több feladatokat vagy műveleteket egyetlen számítási egységbe egyesíteni.</span><span class="sxs-lookup"><span data-stu-id="72984-105">Consolidate multiple tasks or operations into a single computational unit.</span></span> <span data-ttu-id="72984-106">Ez növeli a számítási erőforrások kihasználtságát, és csökkentik a költségeket és a felügyeleti terhelést hajt végre a felhőben üzemeltetett alkalmazások feldolgozás alatt álló számítási társított.</span><span class="sxs-lookup"><span data-stu-id="72984-106">This can increase compute resource utilization, and reduce the costs and management overhead associated with performing compute processing in cloud-hosted applications.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="72984-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="72984-107">Context and problem</span></span>

<span data-ttu-id="72984-108">A felhőalapú alkalmazások gyakran valósítja meg a különféle műveletek.</span><span class="sxs-lookup"><span data-stu-id="72984-108">A cloud application often implements a variety of operations.</span></span> <span data-ttu-id="72984-109">Néhány érdemes aggályokat elkülönítésének tervezési elvét kezdetben kövesse, és ossza meg ezeket a műveleteket az üzemeltetett és külön-külön (például külön App Service web apps – külön, telepített külön számítási egység Virtuális gépek, vagy külön Felhőszolgáltatási szerepkörök).</span><span class="sxs-lookup"><span data-stu-id="72984-109">In some solutions it makes sense to follow the design principle of separation of concerns initially, and divide these operations into separate computational units that are hosted and deployed individually (for example, as separate App Service web apps, separate Virtual Machines, or separate Cloud Service roles).</span></span> <span data-ttu-id="72984-110">Azonban ezt a stratégiát megkönnyítheti a logikai a megoldás kialakításának, bár telepítése számos számítási egység ugyanahhoz az alkalmazáshoz részeként is üzemeltetési költségek futásidejű növelése és győződjön a rendszer felügyeleti összetettebb.</span><span class="sxs-lookup"><span data-stu-id="72984-110">However, although this strategy can help simplify the logical design of the solution, deploying a large number of computational units as part of the same application can increase runtime hosting costs and make management of the system more complex.</span></span>

<span data-ttu-id="72984-111">Tegyük fel az ábrán látható az egyszerűsített struktúra a felhőben üzemeltetett megoldás, amely több számítási egység segítségével van megvalósítva.</span><span class="sxs-lookup"><span data-stu-id="72984-111">As an example, the figure shows the simplified structure of a cloud-hosted solution that is implemented using more than one computational unit.</span></span> <span data-ttu-id="72984-112">Minden számítási egység a saját virtuális környezetben futtatja.</span><span class="sxs-lookup"><span data-stu-id="72984-112">Each computational unit runs in its own virtual environment.</span></span> <span data-ttu-id="72984-113">Minden egyes függvény van megvalósítva (– a feladat-E a tevékenység neve) külön feladatként futtatja a saját számítási egység.</span><span class="sxs-lookup"><span data-stu-id="72984-113">Each function has been implemented as a separate task (labeled Task A through Task E) running in its own computational unit.</span></span>

![Az éppen futó feladatok egy felhőalapú környezetben használatával dedikált számítási egység](./_images/compute-resource-consolidation-diagram.png)


<span data-ttu-id="72984-115">Minden számítási egység terhelhető erőforrásokat használ fel, akkor is, amikor az üresjárati vagy enyhén használt.</span><span class="sxs-lookup"><span data-stu-id="72984-115">Each computational unit consumes chargeable resources, even when it's idle or lightly used.</span></span> <span data-ttu-id="72984-116">Ezért ezt nem mindig a leginkább költséghatékony megoldás.</span><span class="sxs-lookup"><span data-stu-id="72984-116">Therefore, this isn't always the most cost-effective solution.</span></span>

<span data-ttu-id="72984-117">Ezen probléma Azure, a egy felhőalapú szolgáltatás, az App Service szolgáltatások és virtuális gépek szerepkörök vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="72984-117">In Azure, this concern applies to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="72984-118">Ezeket az elemeket a saját virtuális környezetben futtatja.</span><span class="sxs-lookup"><span data-stu-id="72984-118">These items run in their own virtual environment.</span></span> <span data-ttu-id="72984-119">Külön szerepköröket, a webhelyek vagy a virtuális gépek, amelyek jól meghatározott műveletkészlet végrehajtásához, de a, amelyeknek szükségük van a kommunikációhoz, és egyetlen megoldás részeként együttműködnek gyűjteménye fut, lehet, hogy az erőforrások hatékony használatát.</span><span class="sxs-lookup"><span data-stu-id="72984-119">Running a collection of separate roles, websites, or virtual machines that are designed to perform a set of well-defined operations, but that need to communicate and cooperate as part of a single solution, can be an inefficient use of resources.</span></span>

## <a name="solution"></a><span data-ttu-id="72984-120">Megoldás</span><span class="sxs-lookup"><span data-stu-id="72984-120">Solution</span></span>

<span data-ttu-id="72984-121">Segítségével csökkentheti a költségeket, növelje a kihasználtsági, kommunikációs sebesség növelése és csökkentse a felügyeleti rendszer több feladatokat vagy műveleteket egyetlen számítási egységbe vonják össze.</span><span class="sxs-lookup"><span data-stu-id="72984-121">To help reduce costs, increase utilization, improve communication speed, and reduce management it's possible to consolidate multiple tasks or operations into a single computational unit.</span></span>

<span data-ttu-id="72984-122">Feladatok a környezet által nyújtott szolgáltatásokat, és ezek a funkciók kapcsolódó költségeket alapján feltételek szerint csoportosíthatók.</span><span class="sxs-lookup"><span data-stu-id="72984-122">Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features.</span></span> <span data-ttu-id="72984-123">Egy közös megoldás, amelyet meg kíván keresni egy hasonló profilhoz, a méretezhetőséget, a élettartamát és az ügyféloldali bővítmények feldolgozási követelményeivel kapcsolatos feladatok.</span><span class="sxs-lookup"><span data-stu-id="72984-123">A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements.</span></span> <span data-ttu-id="72984-124">Csoportosítás ezen együtt lehetővé teszi a méretezési egységet.</span><span class="sxs-lookup"><span data-stu-id="72984-124">Grouping these together allows them to scale as a unit.</span></span> <span data-ttu-id="72984-125">A sok felhőkörnyezetekben által biztosított rugalmassága lehetővé teszi, hogy egy számítási egység indítása és leállítása a munkaterhelés szerint további példányait.</span><span class="sxs-lookup"><span data-stu-id="72984-125">The elasticity provided by many cloud environments enables additional instances of a computational unit to be started and stopped according to the workload.</span></span> <span data-ttu-id="72984-126">Például az Azure biztosít automatikus skálázás alkalmazható a szerepkörökhöz a egy felhőalapú szolgáltatás, az App Service szolgáltatások és virtuális gépek.</span><span class="sxs-lookup"><span data-stu-id="72984-126">For example, Azure provides autoscaling that you can apply to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="72984-127">További információkért lásd: [automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="72984-127">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span>

<span data-ttu-id="72984-128">Egy számláló példaként hogyan méretezhetőség segítségével határozza meg, mely műveletek nem csoportosított megjelenítendő vegye figyelembe az alábbi két feladatot:</span><span class="sxs-lookup"><span data-stu-id="72984-128">As a counter example to show how scalability can be used to determine which operations shouldn't be grouped together, consider the following two tasks:</span></span>

- <span data-ttu-id="72984-129">1. feladat annak a várólistára küldött alkalomszerű, idő-és nagybetűk megkülönböztetése nélkül üzenet kérdezi le.</span><span class="sxs-lookup"><span data-stu-id="72984-129">Task 1 polls for infrequent, time-insensitive messages sent to a queue.</span></span>
- <span data-ttu-id="72984-130">2. feladat nagy mennyiségű felszakadásáig hálózati forgalmat kezeli.</span><span class="sxs-lookup"><span data-stu-id="72984-130">Task 2 handles high-volume bursts of network traffic.</span></span>

<span data-ttu-id="72984-131">A második feladathoz, amely magába foglaló indítása és leállítása a nagy mennyiségű példánnyal a számítási egység rugalmasság.</span><span class="sxs-lookup"><span data-stu-id="72984-131">The second task requires elasticity that can involve starting and stopping a large number of instances of the computational unit.</span></span> <span data-ttu-id="72984-132">Az azonos méretezés alkalmazása az első lépése egyszerűen eredményeznének további feladatok a ugyanazon a várólistán alkalomszerű üzeneteket figyeli, és a pazarlás erőforrások.</span><span class="sxs-lookup"><span data-stu-id="72984-132">Applying the same scaling to the first task would simply result in more tasks listening for infrequent messages on the same queue, and is a waste of resources.</span></span>

<span data-ttu-id="72984-133">Sok felhőalapú környezetben is lehet egy számítási egységhez CPU mag, memória, lemezterület, és így tovább számára elérhető erőforrások megadására.</span><span class="sxs-lookup"><span data-stu-id="72984-133">In many cloud environments it's possible to specify the resources available to a computational unit in terms of the number of CPU cores, memory, disk space, and so on.</span></span> <span data-ttu-id="72984-134">Általában a megadott, további erőforrások minél nagyobb a költségeket.</span><span class="sxs-lookup"><span data-stu-id="72984-134">Generally, the more resources specified, the greater the cost.</span></span> <span data-ttu-id="72984-135">Kevesebbet költeni, fontos egy drága számítási egység hajt végre, és azt válnak inaktívvá hosszú időn keresztül a munkahelyi maximalizálása érdekében.</span><span class="sxs-lookup"><span data-stu-id="72984-135">To save money, it's important to maximize the work an expensive computational unit performs, and not let it become inactive for an extended period.</span></span>

<span data-ttu-id="72984-136">Ha vannak, amelyek szükségesek a jelentős mértékben a CPU power rövid felszakadásáig, fontolja meg, ezek a szükséges teljesítményt biztosító egyetlen számítási egységbe konszolidálása.</span><span class="sxs-lookup"><span data-stu-id="72984-136">If there are tasks that require a great deal of CPU power in short bursts, consider consolidating these into a single computational unit that provides the necessary power.</span></span> <span data-ttu-id="72984-137">Fontos azonban drága erőforrások foglalt szemben a versengés, amely fordulhat elő, ha azok vannak keresztül hangsúlyozni tartása igénynek elosztása.</span><span class="sxs-lookup"><span data-stu-id="72984-137">However, it's important to balance this need to keep expensive resources busy against the contention that could occur if they are over stressed.</span></span> <span data-ttu-id="72984-138">Hosszan futó, a számítás-igényes feladatok például az azonos számítási egység nem szabad megosztani.</span><span class="sxs-lookup"><span data-stu-id="72984-138">Long-running, compute-intensive tasks shouldn't share the same computational unit, for example.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="72984-139">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="72984-139">Issues and considerations</span></span>

<span data-ttu-id="72984-140">Ebben a mintában végrehajtásakor, vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="72984-140">Consider the following points when implementing this pattern:</span></span>

<span data-ttu-id="72984-141">**Méretezhetőség és a rugalmasság**.</span><span class="sxs-lookup"><span data-stu-id="72984-141">**Scalability and elasticity**.</span></span> <span data-ttu-id="72984-142">Sok felhőalapú megoldás megvalósíthatja méretezhetőség és a rugalmasság a számítási egység szintjén indítása és leállítása egységek példányai.</span><span class="sxs-lookup"><span data-stu-id="72984-142">Many cloud solutions implement scalability and elasticity at the level of the computational unit by starting and stopping instances of units.</span></span> <span data-ttu-id="72984-143">Kerülje a csoportosítási feladatok, amelyek azonos számítási egységben ütköző méretezhetőségi követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="72984-143">Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.</span></span>

<span data-ttu-id="72984-144">**Élettartam**.</span><span class="sxs-lookup"><span data-stu-id="72984-144">**Lifetime**.</span></span> <span data-ttu-id="72984-145">A felhőalapú infrastruktúra rendszeres időközönként újraindul egy számítási egység üzemeltető virtuális környezetben.</span><span class="sxs-lookup"><span data-stu-id="72984-145">The cloud infrastructure periodically recycles the virtual environment that hosts a computational unit.</span></span> <span data-ttu-id="72984-146">Ha sok hosszan futó feladatokat egy számítási egység belül, ezáltal megakadályozhatja, hogy újra lesznek hasznosítva, amíg a feladatok befejezése egység konfigurálása szükség lehet.</span><span class="sxs-lookup"><span data-stu-id="72984-146">When there are many long-running tasks inside a computational unit, it might be necessary to configure the unit to prevent it from being recycled until these tasks have finished.</span></span> <span data-ttu-id="72984-147">Azt is megteheti tervezze meg a feladatok a jelölőnégyzet mutató megközelítés, amely lehetővé teszi, hogy nem szabályszerűen, és azokat a számítási egység újraindításakor program megszakította a pontnál folytatják használatával.</span><span class="sxs-lookup"><span data-stu-id="72984-147">Alternatively, design the tasks by using a check-pointing approach that enables them to stop cleanly, and continue at the point they were interrupted when the computational unit is restarted.</span></span>

<span data-ttu-id="72984-148">**Kiadás ütemben történik**.</span><span class="sxs-lookup"><span data-stu-id="72984-148">**Release cadence**.</span></span> <span data-ttu-id="72984-149">Ha a végrehajtása vagy a feladat konfigurációját változik gyakran, állítsa le a frissített kódot futtató számítási egység, konfigurálja újra, és telepítse újra az egység, és indítsa újra szükség lehet.</span><span class="sxs-lookup"><span data-stu-id="72984-149">If the implementation or configuration of a task changes frequently, it might be necessary to stop the computational unit hosting the updated code, reconfigure and redeploy the unit, and then restart it.</span></span> <span data-ttu-id="72984-150">Ez a folyamat is szükséges, hogy az azonos számítási egység minden más feladat leállt, újratelepíteni, és újraindításra kerülnek.</span><span class="sxs-lookup"><span data-stu-id="72984-150">This process will also require that all other tasks within the same computational unit are stopped, redeployed, and restarted.</span></span>

<span data-ttu-id="72984-151">**Biztonság**.</span><span class="sxs-lookup"><span data-stu-id="72984-151">**Security**.</span></span> <span data-ttu-id="72984-152">Az azonos számítási egység feladatokat előfordulhat, hogy ossza meg ugyanazt a biztonsági környezetet, majd fogja tudni elérni a ugyanazokat az erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="72984-152">Tasks in the same computational unit might share the same security context and be able to access the same resources.</span></span> <span data-ttu-id="72984-153">A feladatok, és abban, hogy közötti megbízhatósági kapcsolat magas fokú kell lennie, hogy egy tevékenység nem sérült, vagy egy másik kedvezőtlen hatással lesz.</span><span class="sxs-lookup"><span data-stu-id="72984-153">There must be a high degree of trust between the tasks, and confidence that one task isn't going to corrupt or adversely affect another.</span></span> <span data-ttu-id="72984-154">Emellett egy számítási egységben futó feladatok számának növelése növeli a támadási felületet a egység.</span><span class="sxs-lookup"><span data-stu-id="72984-154">Additionally, increasing the number of tasks running in a computational unit increases the attack surface of the unit.</span></span> <span data-ttu-id="72984-155">Minden feladat csak olyan biztonságos, mint amelyen a legtöbb biztonsági rések.</span><span class="sxs-lookup"><span data-stu-id="72984-155">Each task is only as secure as the one with the most vulnerabilities.</span></span>

<span data-ttu-id="72984-156">**Hibatűrés**.</span><span class="sxs-lookup"><span data-stu-id="72984-156">**Fault tolerance**.</span></span> <span data-ttu-id="72984-157">Számítási egység egy feladat sikertelen lesz, vagy rendkívül viselkedik-e, ha ez befolyásolhatja a más, ugyanazon egységen belül futó feladatok.</span><span class="sxs-lookup"><span data-stu-id="72984-157">If one task in a computational unit fails or behaves abnormally, it can affect the other tasks running within the same unit.</span></span> <span data-ttu-id="72984-158">Például ha egy feladat nem indítható okozhat a számítási egység sikertelen lesz a teljes ügyfélindítási logikája, és más feladatok azonos egységben tiltsa le a futását.</span><span class="sxs-lookup"><span data-stu-id="72984-158">For example, if one task fails to start correctly it can cause the entire startup logic for the computational unit to fail, and prevent other tasks in the same unit from running.</span></span>

<span data-ttu-id="72984-159">**A versengés**.</span><span class="sxs-lookup"><span data-stu-id="72984-159">**Contention**.</span></span> <span data-ttu-id="72984-160">Kerülje a feladatokat, amelyek "versenyeznek" az számítási egységen belüli erőforrások közötti versengés bemutatása.</span><span class="sxs-lookup"><span data-stu-id="72984-160">Avoid introducing contention between tasks that compete for resources in the same computational unit.</span></span> <span data-ttu-id="72984-161">Ideális esetben az azonos számítási egység tevékenységeket kell mutatnak a különböző Erőforrás kihasználtsága.</span><span class="sxs-lookup"><span data-stu-id="72984-161">Ideally, tasks that share the same computational unit should exhibit different resource utilization characteristics.</span></span> <span data-ttu-id="72984-162">Például két számítási-igényes feladatok valószínűleg nem találhatók a azonos számítási egység, és nem kell a két feladatot, amely nagy mennyiségű memóriát használ.</span><span class="sxs-lookup"><span data-stu-id="72984-162">For example, two compute-intensive tasks should probably not reside in the same computational unit, and neither should two tasks that consume large amounts of memory.</span></span> <span data-ttu-id="72984-163">Azonban jelentős mennyiségű memóriát igénylő számítási intenzív tevékenység keverése nem alkalmazható kombinációját.</span><span class="sxs-lookup"><span data-stu-id="72984-163">However, mixing a compute intensive task with a task that requires a large amount of memory is a workable combination.</span></span>

> [!NOTE]
>  <span data-ttu-id="72984-164">Vegye figyelembe a számítási erőforrásokat csak egy rendszer számára, hogy az operátorok és a fejlesztők a figyelheti a rendszer, és hozzon létre egy adott időn belül az éles környezetben ennyi ideig konszolidálása egy _hőtérkép_ , amely azonosítja, hogyan használja minden feladat eltérő erőforrások.</span><span class="sxs-lookup"><span data-stu-id="72984-164">Consider consolidating compute resources only for a system that's been in production for a period of time so that operators and developers can monitor the system and create a _heat map_ that identifies how each task utilizes differing resources.</span></span> <span data-ttu-id="72984-165">Ez a térkép határozza meg, milyen feladatokat a számítási erőforrások megosztása esetén használható jól használható.</span><span class="sxs-lookup"><span data-stu-id="72984-165">This map can be used to determine which tasks are good candidates for sharing compute resources.</span></span>

<span data-ttu-id="72984-166">**Összetettsége**.</span><span class="sxs-lookup"><span data-stu-id="72984-166">**Complexity**.</span></span> <span data-ttu-id="72984-167">Több feladat egyetlen számítási egységbe egyesítése összetettségét hozzáadja a kódot a egységben, valószínűleg megnehezíti tesztelése, hibakeresését és karbantartása.</span><span class="sxs-lookup"><span data-stu-id="72984-167">Combining multiple tasks into a single computational unit adds complexity to the code in the unit, possibly making it more difficult to test, debug, and maintain.</span></span>

<span data-ttu-id="72984-168">**Logikai architektúra stabil**.</span><span class="sxs-lookup"><span data-stu-id="72984-168">**Stable logical architecture**.</span></span> <span data-ttu-id="72984-169">A tervezést, és minden feladat valósítja meg a kódot, így azt nem kell módosítania, még akkor is, ha a feladat fut a fizikai környezet módosítása.</span><span class="sxs-lookup"><span data-stu-id="72984-169">Design and implement the code in each task so that it shouldn't need to change, even if the physical environment the task runs in does change.</span></span>

<span data-ttu-id="72984-170">**Más stratégiák**.</span><span class="sxs-lookup"><span data-stu-id="72984-170">**Other strategies**.</span></span> <span data-ttu-id="72984-171">Számítási erőforrások konszolidálása csak egy módja több feladat egyidejű futtatásával kapcsolatos költségek csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="72984-171">Consolidating compute resources is only one way to help reduce costs associated with running multiple tasks concurrently.</span></span> <span data-ttu-id="72984-172">Körültekintően megtervezve és figyelés győződjön meg arról, hogy marad egy hatékony módszert igényel.</span><span class="sxs-lookup"><span data-stu-id="72984-172">It requires careful planning and monitoring to ensure that it remains an effective approach.</span></span> <span data-ttu-id="72984-173">Előfordulhat, hogy más stratégiák pontosabb, attól függően, hogy a munkahelyi és hol találhatók a felhasználók, a feladatok futnak.</span><span class="sxs-lookup"><span data-stu-id="72984-173">Other strategies might be more appropriate, depending on the nature of the work and where the users these tasks are running are located.</span></span> <span data-ttu-id="72984-174">A munkaterhelés például működési felbontás ellen (szerint a [particionálás útmutatást számítási](https://msdn.microsoft.com/library/dn589773.aspx)) lehet, hogy jobb megoldás.</span><span class="sxs-lookup"><span data-stu-id="72984-174">For example, functional decomposition of the workload (as described by the [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx)) might be a better option.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="72984-175">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="72984-175">When to use this pattern</span></span>

<span data-ttu-id="72984-176">Ebben a mintában nincsenek költséghatékony, ha futtatja a saját számítási egység feladatokhoz használhatja.</span><span class="sxs-lookup"><span data-stu-id="72984-176">Use this pattern for tasks that are not cost effective if they run in their own computational units.</span></span> <span data-ttu-id="72984-177">Ha egy feladat nagy részét az üresjárati időt tölt, költséges lehet ez a feladat fut egy dedikált egységben.</span><span class="sxs-lookup"><span data-stu-id="72984-177">If a task spends much of its time idle, running this task in a dedicated unit can be expensive.</span></span>

<span data-ttu-id="72984-178">Előfordulhat, hogy ez a minta nem megfelelő-e a feladatokat, kritikus hibatűrő műveleteket, illetve a feladatokat, amelyek rendkívül bizalmas vagy személyes adatok feldolgozása és a saját biztonsági környezet szükséges.</span><span class="sxs-lookup"><span data-stu-id="72984-178">This pattern might not be suitable for tasks that perform critical fault-tolerant operations, or tasks that process highly sensitive or private data and require their own security context.</span></span> <span data-ttu-id="72984-179">Ezeket a feladatokat egy külön számítási egység a saját elkülönített környezetben fusson.</span><span class="sxs-lookup"><span data-stu-id="72984-179">These tasks should run in their own isolated environment, in a separate computational unit.</span></span>

## <a name="example"></a><span data-ttu-id="72984-180">Példa</span><span class="sxs-lookup"><span data-stu-id="72984-180">Example</span></span>

<span data-ttu-id="72984-181">Azure cloud Service szolgáltatásra fejlesztéskor is lehet összevonása akkor történjen meg több feladatok, irányítson át egyetlen szerepkör feldolgozását.</span><span class="sxs-lookup"><span data-stu-id="72984-181">When building a cloud service on Azure, it’s possible to consolidate the processing performed by multiple tasks into a single role.</span></span> <span data-ttu-id="72984-182">Ez általában egy feldolgozói szerepkört, amely a háttér vagy aszinkron feladatokat hajt végre.</span><span class="sxs-lookup"><span data-stu-id="72984-182">Typically this is a worker role that performs background or asynchronous processing tasks.</span></span>

> <span data-ttu-id="72984-183">Néhány esetben lehetőség a háttér vagy aszinkron feldolgozási feladatok a webes szerepkör.</span><span class="sxs-lookup"><span data-stu-id="72984-183">In some cases it's possible to include background or asynchronous processing tasks in the web role.</span></span> <span data-ttu-id="72984-184">Ez a módszer segítségével csökkentheti a költségeket és egyszerűbbé telepítését, bár ez hatással lehet a méretezhetőség és a nyilvánosan elérhető felülete, a webes szerepkör válaszképességét.</span><span class="sxs-lookup"><span data-stu-id="72984-184">This technique helps to reduce costs and simplify deployment, although it can impact the scalability and responsiveness of the public-facing interface provided by the web role.</span></span> <span data-ttu-id="72984-185">A cikk [egyesítésével több Azure feldolgozói szerepkörök be egy Azure webes szerepkör](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) háttér vagy aszinkron feldolgozási feladatainak valósít meg a webes szerepkör részletes leírását tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="72984-185">The article [Combining Multiple Azure Worker Roles into an Azure Web Role](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) contains a detailed description of implementing background or asynchronous processing tasks in a web role.</span></span>

<span data-ttu-id="72984-186">A szerepkör indítása és leállítása a feladatok felelős.</span><span class="sxs-lookup"><span data-stu-id="72984-186">The role is responsible for starting and stopping the tasks.</span></span> <span data-ttu-id="72984-187">Egy szerepkört az Azure fabric controller betöltésekor, a riasztást a `Start` esemény a szerepkörhöz.</span><span class="sxs-lookup"><span data-stu-id="72984-187">When the Azure fabric controller loads a role, it raises the `Start` event for the role.</span></span> <span data-ttu-id="72984-188">Felülírhatja a `OnStart` metódusában a `WebRole` vagy `WorkerRole` osztály kezelni ezt az eseményt, lehet, hogy inicializálni az adatokat és más erőforrások függenek a feladatokat az ezzel a módszerrel.</span><span class="sxs-lookup"><span data-stu-id="72984-188">You can override the `OnStart` method of the `WebRole` or `WorkerRole` class to handle this event, perhaps to initialize the data and other resources the tasks in this method depend on.</span></span>

<span data-ttu-id="72984-189">Ha a `OnStart `metódus befejeződött, a szerepkör elindíthatja válaszol a kérelmekre.</span><span class="sxs-lookup"><span data-stu-id="72984-189">When the `OnStart `method completes, the role can start responding to requests.</span></span> <span data-ttu-id="72984-190">További információt és útmutatást segítségével megtalálhatja a `OnStart` és `Run` módszerek a szerepkörnek a [alkalmazás indítási folyamat](https://msdn.microsoft.com/library/ff803371.aspx#sec16) a minták és gyakorlatok guide című [alkalmazások áthelyezése a felhőbe](https://msdn.microsoft.com/library/ff728592.aspx).</span><span class="sxs-lookup"><span data-stu-id="72984-190">You can find more information and guidance about using the `OnStart` and `Run` methods in a role in the [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) section in the patterns & practices guide [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx).</span></span>

> <span data-ttu-id="72984-191">A kód tartsa a `OnStart` , pontos sikerkritériumokkal biztosíthatja a lehető metódust.</span><span class="sxs-lookup"><span data-stu-id="72984-191">Keep the code in the `OnStart` method as concise as possible.</span></span> <span data-ttu-id="72984-192">Azure nem ugyanazok a metódus végrehajtásához szükséges idő a bármely korlátozását, de a szerepkör nem indítható el, amíg befejeződik ez a módszer küldött hálózati kérésekre válaszol.</span><span class="sxs-lookup"><span data-stu-id="72984-192">Azure doesn't impose any limit on the time taken for this method to complete, but the role won't be able to start responding to network requests sent to it until this method completes.</span></span>

<span data-ttu-id="72984-193">Ha a `OnStart` metódus befejeződött, a szerepkör végrehajtja a `Run` metódust.</span><span class="sxs-lookup"><span data-stu-id="72984-193">When the `OnStart` method has finished, the role executes the `Run` method.</span></span> <span data-ttu-id="72984-194">Ezen a ponton a fabric controller elindíthatja kérelmek küldése a szerepkört.</span><span class="sxs-lookup"><span data-stu-id="72984-194">At this point, the fabric controller can start sending requests to the role.</span></span>

<span data-ttu-id="72984-195">A kódot, amely ténylegesen hoz létre a feladatokat a `Run` metódust.</span><span class="sxs-lookup"><span data-stu-id="72984-195">Place the code that actually creates the tasks in the `Run` method.</span></span> <span data-ttu-id="72984-196">Vegye figyelembe, hogy a `Run` metódus a szerepkörpéldányt élettartamát határozza meg.</span><span class="sxs-lookup"><span data-stu-id="72984-196">Note that the `Run` method defines the lifetime of the role instance.</span></span> <span data-ttu-id="72984-197">Ez a módszer befejezését követően a fabric controller szervez le kell állítani a szerepkörhöz.</span><span class="sxs-lookup"><span data-stu-id="72984-197">When this method completes, the fabric controller will arrange for the role to be shut down.</span></span>

<span data-ttu-id="72984-198">Szerepkör leáll vagy újrahasznosítása esetén a fabric controller megakadályozza, hogy a terheléselosztó és vált ki fogadott több bejövő kérelmek a `Stop` esemény.</span><span class="sxs-lookup"><span data-stu-id="72984-198">When a role shuts down or is recycled, the fabric controller prevents any more incoming requests being received from the load balancer and raises the `Stop` event.</span></span> <span data-ttu-id="72984-199">Ez az esemény felülbírálásával rögzítheti a `OnStop` metódus a szerepkör, és végezze el minden tidying előtt megszakítja a szerepkör szükséges.</span><span class="sxs-lookup"><span data-stu-id="72984-199">You can capture this event by overriding the `OnStop` method of the role and perform any tidying up required before the role terminates.</span></span>

> <span data-ttu-id="72984-200">Semmilyen műveletet végez a `OnStop` metódus öt perc (vagy 30 másodperces használata az Azure emulátorban a helyi számítógép) belül el kell végezni.</span><span class="sxs-lookup"><span data-stu-id="72984-200">Any actions performed in the `OnStop` method must be completed within five minutes (or 30 seconds if you are using the Azure emulator on a local computer).</span></span> <span data-ttu-id="72984-201">Ellenkező esetben az Azure fabric controller feltételezi, hogy a szerepkör leállását észlelte, és arra kényszeríti, hogy állítsa le.</span><span class="sxs-lookup"><span data-stu-id="72984-201">Otherwise the Azure fabric controller assumes that the role has stalled and will force it to stop.</span></span>

<span data-ttu-id="72984-202">A feladatok által elindított a `Run` módszer, amely a feladatok befejezésére vár.</span><span class="sxs-lookup"><span data-stu-id="72984-202">The tasks are started by the `Run` method that waits for the tasks to complete.</span></span> <span data-ttu-id="72984-203">A feladatok valósítja meg az üzleti logika, a felhőalapú szolgáltatás, és képes válaszolni a szerepkört az Azure load balancer keresztül küldött üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="72984-203">The tasks implement the business logic of the cloud service, and can respond to messages posted to the role through the Azure load balancer.</span></span> <span data-ttu-id="72984-204">Az ábrán láthatók az életciklus feladatok és erőforrások olyan szerepkörben, az Azure-felhőszolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="72984-204">The figure shows the lifecycle of tasks and resources in a role in an Azure cloud service.</span></span>

![Az életciklus feladatok és erőforrások olyan szerepkörben, az Azure-felhőszolgáltatás](./_images/compute-resource-consolidation-lifecycle.png)


<span data-ttu-id="72984-206">A _WorkerRole.cs_ fájlt a _ComputeResourceConsolidation.Worker_ projekt szemlélteti, hogyan lehet, hogy implementálja ebben a mintában az Azure-felhőszolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="72984-206">The _WorkerRole.cs_ file in the _ComputeResourceConsolidation.Worker_ project shows an example of how you might implement this pattern in an Azure cloud service.</span></span>

> <span data-ttu-id="72984-207">A _ComputeResourceConsolidation.Worker_ projekt része a _ComputeResourceConsolidation_ letölthető megoldás [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span><span class="sxs-lookup"><span data-stu-id="72984-207">The _ComputeResourceConsolidation.Worker_ project is part of the _ComputeResourceConsolidation_ solution available for download from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>

<span data-ttu-id="72984-208">A `MyWorkerTask1` és a `MyWorkerTask2` módszerek bemutatják, hogyan lehet belül az azonos feldolgozói szerepkör különböző feladatok végrehajtására.</span><span class="sxs-lookup"><span data-stu-id="72984-208">The `MyWorkerTask1` and the `MyWorkerTask2` methods illustrate how to perform different tasks within the same worker role.</span></span> <span data-ttu-id="72984-209">A következő kódot tartalmazza `MyWorkerTask1`.</span><span class="sxs-lookup"><span data-stu-id="72984-209">The following code shows `MyWorkerTask1`.</span></span> <span data-ttu-id="72984-210">Ez egy egyszerű feladat 30 másodpercig alvó állapotba kerül, majd exportálja a nyomkövetési üzenet.</span><span class="sxs-lookup"><span data-stu-id="72984-210">This is a simple task that sleeps for 30 seconds and then outputs a trace message.</span></span> <span data-ttu-id="72984-211">Ez a folyamat ismétlődik, amíg a feladat meg lett szakítva.</span><span class="sxs-lookup"><span data-stu-id="72984-211">It repeats this process until the task is canceled.</span></span> <span data-ttu-id="72984-212">A kód `MyWorkerTask2` hasonló.</span><span class="sxs-lookup"><span data-stu-id="72984-212">The code in `MyWorkerTask2` is similar.</span></span>

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

> <span data-ttu-id="72984-213">A mintakód bemutatja egy közös végrehajtása háttérfolyamatként.</span><span class="sxs-lookup"><span data-stu-id="72984-213">The sample code shows a common implementation of a background process.</span></span> <span data-ttu-id="72984-214">Egy valós alkalmazás követésével Ez ugyanaz a struktúra, azzal a különbséggel, hogy a saját feldolgozó logika helyezze a hurok, amely megvárja a megszakítási kérés törzsében.</span><span class="sxs-lookup"><span data-stu-id="72984-214">In a real world application you can follow this same structure, except that you should place your own processing logic in the body of the loop that waits for the cancellation request.</span></span>

<span data-ttu-id="72984-215">A feldolgozói szerepkör inicializálta a által használt erőforrások után a `Run` metódus elindítja a két feladatot egyszerre, ahogy az itt látható.</span><span class="sxs-lookup"><span data-stu-id="72984-215">After the worker role has initialized the resources it uses, the `Run` method starts the two tasks concurrently, as shown here.</span></span>

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

<span data-ttu-id="72984-216">Ebben a példában a `Run` metódus megvárja-e a feladatot végrehajtani.</span><span class="sxs-lookup"><span data-stu-id="72984-216">In this example, the `Run` method waits for tasks to be completed.</span></span> <span data-ttu-id="72984-217">Ha a feladat meg lett szakítva, a `Run` metódus feltételezi, hogy a szerepkör leállítása folyamatban van, és megvárja-e a befejezése előtt megszakítandó feladataiban (megvárja, legfeljebb öt perc megszakítása előtt).</span><span class="sxs-lookup"><span data-stu-id="72984-217">If a task is canceled, the `Run` method assumes that the role is being shut down and waits for the remaining tasks to be canceled before finishing (it waits for a maximum of five minutes before terminating).</span></span> <span data-ttu-id="72984-218">Ha egy feladat nem várt kivétel miatt a `Run` metódus megszakítja a feladatot.</span><span class="sxs-lookup"><span data-stu-id="72984-218">If a task fails due to an expected exception, the `Run` method cancels the task.</span></span>

> <span data-ttu-id="72984-219">Szélesebb körű figyelési és stratégiák a kivételkezelő sikerült megvalósítása a `Run` módszer például a sikertelen feladatok újraindítása, vagy beleértve a kódot, amely lehetővé teszi a szerepkör leállítása és elindítása az egyes feladatok.</span><span class="sxs-lookup"><span data-stu-id="72984-219">You could implement more comprehensive monitoring and exception handling strategies in the `Run` method such as restarting tasks that have failed, or including code that enables the role to stop and start individual tasks.</span></span>

<span data-ttu-id="72984-220">A `Stop` az alábbi kódban látható metódus lehívásra kerül, amikor a szerepkör példánya leáll a háló vezérlő (a meghívták a `OnStop` metódus).</span><span class="sxs-lookup"><span data-stu-id="72984-220">The `Stop` method shown in the following code is called when the fabric controller shuts down the role instance (it's invoked from the `OnStop` method).</span></span> <span data-ttu-id="72984-221">A kód minden feladat szabályosan leállítja törli azt.</span><span class="sxs-lookup"><span data-stu-id="72984-221">The code stops each task gracefully by canceling it.</span></span> <span data-ttu-id="72984-222">Ha a feladat több mint öt percet is igénybe vehet, feldolgozás alatt álló ezzel a `Stop` metódus várakozási megszűnik, és a szerepkör a rendszer megszakítja.</span><span class="sxs-lookup"><span data-stu-id="72984-222">If any task takes more than five minutes to complete, the cancellation processing in the `Stop` method ceases waiting and the role is terminated.</span></span>

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

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="72984-223">Útmutató és a kapcsolódó minták</span><span class="sxs-lookup"><span data-stu-id="72984-223">Related patterns and guidance</span></span>

<span data-ttu-id="72984-224">A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:</span><span class="sxs-lookup"><span data-stu-id="72984-224">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="72984-225">[Automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="72984-225">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="72984-226">Automatikus skálázás indítása és leállítása a számítási erőforrásokat, attól függően, hogy a várható igény szerinti feldolgozásra üzemeltetési szolgáltatás példányának használható.</span><span class="sxs-lookup"><span data-stu-id="72984-226">Autoscaling can be used to start and stop instances of service hosting computational resources, depending on the anticipated demand for processing.</span></span>

- <span data-ttu-id="72984-227">[Útmutatás particionálás számítási](https://msdn.microsoft.com/library/dn589773.aspx).</span><span class="sxs-lookup"><span data-stu-id="72984-227">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="72984-228">Ismerteti, hogyan lehet a szolgáltatások és összetevők a felhőalapú szolgáltatás, ezáltal segít a méretezhetőséget, teljesítményt, rendelkezésre állási és a szolgáltatás biztonsági megőrzésével futó költségek minimalizálása érdekében a lefoglalni.</span><span class="sxs-lookup"><span data-stu-id="72984-228">Describes how to allocate the services and components in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>

- <span data-ttu-id="72984-229">Ebben a mintában tartalmaz egy letölthető [mintaalkalmazás](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span><span class="sxs-lookup"><span data-stu-id="72984-229">This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>
