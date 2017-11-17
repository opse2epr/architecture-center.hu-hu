---
title: "Adatcsatornák és a szűrők"
description: "Egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat lebontva."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: b41f3e46ad5982a3a4ec6635918481cb440c5e02
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="pipes-and-filters-pattern"></a><span data-ttu-id="53afc-104">Adatcsatornák és a szűrők minta</span><span class="sxs-lookup"><span data-stu-id="53afc-104">Pipes and Filters pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="53afc-105">Felbontani egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat.</span><span class="sxs-lookup"><span data-stu-id="53afc-105">Decompose a task that performs complex processing into a series of separate elements that can be reused.</span></span> <span data-ttu-id="53afc-106">Ez javíthatja a teljesítmény, méretezhetőség és újrahasznosításának telepítve, és egymástól függetlenül méretezi feldolgozását végző feladat elemek tételével.</span><span class="sxs-lookup"><span data-stu-id="53afc-106">This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="53afc-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="53afc-107">Context and problem</span></span>

<span data-ttu-id="53afc-108">Egy alkalmazás, amely feldolgozza az adatokat a különböző, eltérően összetett különböző feladatok elvégzéséhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="53afc-108">An application is required to perform a variety of tasks of varying complexity on the information that it processes.</span></span> <span data-ttu-id="53afc-109">Egy egyszerű, de rugalmatlan megközelítése egy alkalmazást a feldolgozási egységes modulként végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="53afc-109">A straightforward but inflexible approach to implementing an application is to perform this processing as a monolithic module.</span></span> <span data-ttu-id="53afc-110">Azonban ez a megközelítés valószínű, hogy csökkentse a lehetőségek a kód újrabontása, optimalizálása, vagy ha azonos feldolgozása részei szükség az alkalmazáson belül máshol újbóli felhasználása.</span><span class="sxs-lookup"><span data-stu-id="53afc-110">However, this approach is likely to reduce the opportunities for refactoring the code, optimizing it, or reusing it if parts of the same processing are required elsewhere within the application.</span></span>

<span data-ttu-id="53afc-111">Az ábra azt mutatja be, milyen probléma lépett fel a egységes megközelítéssel adatok feldolgozása.</span><span class="sxs-lookup"><span data-stu-id="53afc-111">The figure illustrates the issues with processing data using the monolithic approach.</span></span> <span data-ttu-id="53afc-112">Az alkalmazás fogad, és két forrásból származó adatokat dolgozza fel.</span><span class="sxs-lookup"><span data-stu-id="53afc-112">An application receives and processes data from two sources.</span></span> <span data-ttu-id="53afc-113">Minden egyes forrásból származó adatokat dolgozza fel ezeket az adatokat, az eredmény az üzleti logika, az alkalmazás való továbbítása előtt átalakítására feladatok sorozatát végző külön modul.</span><span class="sxs-lookup"><span data-stu-id="53afc-113">The data from each source is processed by a separate module that performs a series of tasks to transform this data, before passing the result to the business logic of the application.</span></span>

![1. ábra – egységes modulok készletével megvalósított megoldás](./_images/pipes-and-filters-modules.png)

<span data-ttu-id="53afc-115">Az egységes modulok végre feladatokat funkcionálisan nagyon hasonló, de a modulok elkülönítetten készített.</span><span class="sxs-lookup"><span data-stu-id="53afc-115">Some of the tasks that the monolithic modules perform are functionally very similar, but the modules have been designed separately.</span></span> <span data-ttu-id="53afc-116">A kódot, amely a feladatok szorosan alapján kialakulhat egy modulban, és az újbóli vagy a méretezhetőség kevéssé vagy egyáltalán ne gondolat identitáskezelési.</span><span class="sxs-lookup"><span data-stu-id="53afc-116">The code that implements the tasks is closely coupled in a module, and has been developed with little or no thought given to reuse or scalability.</span></span>

<span data-ttu-id="53afc-117">A modulokhoz, vagy a telepítési követelményeket az egyes tevékenységek által végrehajtott feladatokat sikerült azonban módosítható, mert a üzleti követelményeinek megfelelően frissülnek.</span><span class="sxs-lookup"><span data-stu-id="53afc-117">However, the processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated.</span></span> <span data-ttu-id="53afc-118">Egyes feladatok intenzív számítási előfordulhat, hogy és hatékony hardver, futó, míg más előfordulhat, hogy nincs ilyen költséges erőforrások hasznosak.</span><span class="sxs-lookup"><span data-stu-id="53afc-118">Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources.</span></span> <span data-ttu-id="53afc-119">Is a jövőben további feldolgozás lehet szükség, vagy módosíthatja a, amelyben a feladatokat végzi el a feldolgozási sorrendben.</span><span class="sxs-lookup"><span data-stu-id="53afc-119">Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.</span></span> <span data-ttu-id="53afc-120">A megoldás szükség, hogy ezeket a problémákat, és növeli a kód újbóli lehetőségeit.</span><span class="sxs-lookup"><span data-stu-id="53afc-120">A solution is required that addresses these issues, and increases the possibilities for code reuse.</span></span>

## <a name="solution"></a><span data-ttu-id="53afc-121">Megoldás</span><span class="sxs-lookup"><span data-stu-id="53afc-121">Solution</span></span>

<span data-ttu-id="53afc-122">A feldolgozás le szükséges az egyes adatfolyamokkal külön összetevők (vagy a szűrők), be egy feladat végrehajtása.</span><span class="sxs-lookup"><span data-stu-id="53afc-122">Break down the processing required for each stream into a set of separate components (or filters), each performing a single task.</span></span> <span data-ttu-id="53afc-123">Minden egyes összetevő fogad és küld adatok formátuma szabványosításával ezek a szűrők is egyesíthet bele a folyamatba.</span><span class="sxs-lookup"><span data-stu-id="53afc-123">By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline.</span></span> <span data-ttu-id="53afc-124">A kód duplikálását elkerülje használatával, és megkönnyíti, hogy távolítsa el, cseréje vagy integrálható a további összetevők, ha az ügyféloldali bővítmények feldolgozási követelményeivel módosítása.</span><span class="sxs-lookup"><span data-stu-id="53afc-124">This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.</span></span> <span data-ttu-id="53afc-125">Az alábbi ábrán egy pipe-ok és a szűrők használatával megvalósított megoldást mutat.</span><span class="sxs-lookup"><span data-stu-id="53afc-125">The next figure shows a solution implemented using pipes and filters.</span></span>

![2. ábra - pipe-ok és a szűrők használatával megvalósított megoldás](./_images/pipes-and-filters-solution.png)


<span data-ttu-id="53afc-127">Egyetlen kérelem feldolgozásához szükséges idő a leglassabb szűrő a feldolgozási sebességétől függ.</span><span class="sxs-lookup"><span data-stu-id="53afc-127">The time it takes to process a single request depends on the speed of the slowest filter in the pipeline.</span></span> <span data-ttu-id="53afc-128">Egy vagy több szűrőben lehet szűk keresztmetszet, különösen akkor, ha nagyszámú kérelmek egy adott adatforrásból származó adatfolyam jelennek meg.</span><span class="sxs-lookup"><span data-stu-id="53afc-128">One or more filters could be a bottleneck, especially if a large number of requests appear in a stream from a particular data source.</span></span> <span data-ttu-id="53afc-129">A kulcs az adatcsatorna struktúra előnye teret hagynak a futó párhuzamos példányait lassú szűrők engedélyezésével a rendszer egyenletes terhelését, valamint javítja a teljesítményt biztosít.</span><span class="sxs-lookup"><span data-stu-id="53afc-129">A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span>

<span data-ttu-id="53afc-130">A szűrők adatcsatorna alkotó futtathatja eltérő gépeken, engedélyezése, hogy egymástól függetlenül lehet méretezni, és sok felhőkörnyezetekben adja meg, hogy a rugalmasság előnyeinek kihasználása.</span><span class="sxs-lookup"><span data-stu-id="53afc-130">The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.</span></span> <span data-ttu-id="53afc-131">Egy szűrő, amely számításilag intenzív nagy teljesítményű hardverre, futtathatja, miközben más kevesebb, mint a szűrők igényelnek az alábbiakon tárolható olcsóbb a hagyományos hardvereken.</span><span class="sxs-lookup"><span data-stu-id="53afc-131">A filter that is computationally intensive can run on high performance hardware, while other less demanding filters can be hosted on less expensive commodity hardware.</span></span> <span data-ttu-id="53afc-132">A szűrők nem is kell ugyanabban az adatközpontban vagy földrajzi hely, amely lehetővé teszi, hogy az egyes elemei a folyamat futtatásához olyan környezetben, amelynek mérete megközelítőleg az erőforrásokat igényel.</span><span class="sxs-lookup"><span data-stu-id="53afc-132">The filters don't even have to be in the same data center or geographical location, which allows each element in a pipeline to run in an environment that is close to the resources it requires.</span></span>  <span data-ttu-id="53afc-133">Az alábbi ábrán látható forrás 1 az adatokat a folyamatot alkalmazza.</span><span class="sxs-lookup"><span data-stu-id="53afc-133">The next figure shows an example applied to the pipeline for the data from Source 1.</span></span>

![3. ábrán látható egy példa a feldolgozási sor az adatok alkalmazott forrás 1](./_images/pipes-and-filters-load-balancing.png)

<span data-ttu-id="53afc-135">Ha a bemeneti és kimeneti egy szűrő felépítése adatfolyamként, akkor lehet minden szűrőhöz párhuzamos feldolgozása végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="53afc-135">If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel.</span></span> <span data-ttu-id="53afc-136">Az adatcsatorna első szűrő indítsa el a tevékenységeket és a kimeneti az eredményeket, amely átadott közvetlenül be a következő szűrő sorrendben az első szűrő teendőit befejezése előtt.</span><span class="sxs-lookup"><span data-stu-id="53afc-136">The first filter in the pipeline can start its work and output its results, which are passed directly on to the next filter in the sequence before the first filter has completed its work.</span></span>

<span data-ttu-id="53afc-137">További előny, a rugalmasságot biztosít. ezt a modellt.</span><span class="sxs-lookup"><span data-stu-id="53afc-137">Another benefit is the resiliency that this model can provide.</span></span> <span data-ttu-id="53afc-138">Szűrő meghibásodik, vagy a futtató gép már nem érhető el, ha a feldolgozási sor ütemezze újra a szűrő teljesített munkáját, és közvetlen a munka az összetevő egy másik példánya.</span><span class="sxs-lookup"><span data-stu-id="53afc-138">If a filter fails or the machine it's running on is no longer available, the pipeline can reschedule the work that the filter was performing and direct this work to another instance of the component.</span></span> <span data-ttu-id="53afc-139">Sikertelen volt-e egyetlen szűrőt, nem feltétlenül ezért nem a teljes folyamat.</span><span class="sxs-lookup"><span data-stu-id="53afc-139">Failure of a single filter doesn't necessarily result in failure of the entire pipeline.</span></span>

<span data-ttu-id="53afc-140">A pipe-ok és a szűrők minta használatával együtt a [Compensating tranzakció mintát](compensating-transaction.md) van egy másik módjáról az elosztott tranzakciók végrehajtására.</span><span class="sxs-lookup"><span data-stu-id="53afc-140">Using the Pipes and Filters pattern in conjunction with the [Compensating Transaction pattern](compensating-transaction.md) is an alternative approach to implementing distributed transactions.</span></span> <span data-ttu-id="53afc-141">Elosztott tranzakció bonthatók be különálló, compensable feladatokat, amelyek a tranzakció Compensating mintát megvalósító szűrő használatával valósítható.</span><span class="sxs-lookup"><span data-stu-id="53afc-141">A distributed transaction can be broken down into separate, compensable tasks, each of which can be implemented by using a filter that also implements the Compensating Transaction pattern.</span></span> <span data-ttu-id="53afc-142">A szűrők egy folyamaton belül, az adatokat, amelyek megőriznek hamarosan futó külön futtatott feladatok valósítható meg.</span><span class="sxs-lookup"><span data-stu-id="53afc-142">The filters in a pipeline can be implemented as separate hosted tasks running close to the data that they maintain.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="53afc-143">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="53afc-143">Issues and considerations</span></span>

<span data-ttu-id="53afc-144">Ez a kialakítás megvalósítása meghatározásakor a következő szempontokat kell figyelembe vennie:</span><span class="sxs-lookup"><span data-stu-id="53afc-144">You should consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="53afc-145">**Összetettsége**.</span><span class="sxs-lookup"><span data-stu-id="53afc-145">**Complexity**.</span></span> <span data-ttu-id="53afc-146">A nagyobb rugalmasság, amely ebben a mintában is vezethet összetettségét, különösen akkor, ha egy folyamaton belül szűrők elosztott különböző kiszolgálókon.</span><span class="sxs-lookup"><span data-stu-id="53afc-146">The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.</span></span>

- <span data-ttu-id="53afc-147">**Megbízhatóság**.</span><span class="sxs-lookup"><span data-stu-id="53afc-147">**Reliability**.</span></span> <span data-ttu-id="53afc-148">Használjon olyan infrastruktúra, amely biztosítja, hogy egy folyamat szűrő áramló adatokat nem lesz elveszett.</span><span class="sxs-lookup"><span data-stu-id="53afc-148">Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.</span></span>

- <span data-ttu-id="53afc-149">**Idempotencia**.</span><span class="sxs-lookup"><span data-stu-id="53afc-149">**Idempotency**.</span></span> <span data-ttu-id="53afc-150">Ha egy szűrőt a folyamat meghiúsul, miután üzenet és a munka átütemezése a szűrő másik példányára, része a munka lehet, hogy rendelkezik már befejeződött.</span><span class="sxs-lookup"><span data-stu-id="53afc-150">If a filter in a pipeline fails after receiving a message and the work is rescheduled to another instance of the filter, part of the work might have already been completed.</span></span> <span data-ttu-id="53afc-151">Ha ez a munkahelyi néhány szempontja, hogy a globális állapota (például egy adatbázisában tárolt információit) frissíti, a sikerült ismétlődő ugyanazt a frissítést.</span><span class="sxs-lookup"><span data-stu-id="53afc-151">If this work updates some aspect of the global state (such as information stored in a database), the same update could be repeated.</span></span> <span data-ttu-id="53afc-152">Probléma akkor fordulhat elő, szűrő meghibásodásakor könyvelési az eredményeket a következő szűrő, az adatcsatorna, még mielőtt elvégezte teendőit sikeresen jelző után.</span><span class="sxs-lookup"><span data-stu-id="53afc-152">A similar issue might occur if a filter fails after posting its results to the next filter in the pipeline, but before indicating that it's completed its work successfully.</span></span> <span data-ttu-id="53afc-153">Ebben az esetben ugyanaz a munkahelyi sikerült meg kell ismételni a szűrőt, hogy a program kétszer ugyanazt az eredményt, amely egy másik példánya.</span><span class="sxs-lookup"><span data-stu-id="53afc-153">In these cases, the same work could be repeated by another instance of the filter, causing the same results to be posted twice.</span></span> <span data-ttu-id="53afc-154">Ez a feldolgozási sorban lévő ugyanazokat az adatokat kétszer a későbbi szűrőket vezethet.</span><span class="sxs-lookup"><span data-stu-id="53afc-154">This could result in subsequent filters in the pipeline processing the same data twice.</span></span> <span data-ttu-id="53afc-155">Ezért a folyamat a szűrőket úgy kell megtervezni, kell lennie az idempotent.</span><span class="sxs-lookup"><span data-stu-id="53afc-155">Therefore filters in a pipeline should be designed to be idempotent.</span></span> <span data-ttu-id="53afc-156">További információ: [idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathan Oliver blogjában.</span><span class="sxs-lookup"><span data-stu-id="53afc-156">For more information see [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

- <span data-ttu-id="53afc-157">**Ismétlődő üzenetek**.</span><span class="sxs-lookup"><span data-stu-id="53afc-157">**Repeated messages**.</span></span> <span data-ttu-id="53afc-158">Ha egy folyamaton belül szűrő meghiúsul, miután egy üzenetet, amely a feldolgozási folyamat következő szakasza könyvelés, a szűrő egy másik példánya lehet, hogy futtatható, és azt fogja elküldeni ugyanazt az üzenetet, a folyamat egy példányát.</span><span class="sxs-lookup"><span data-stu-id="53afc-158">If a filter in a pipeline fails after posting a message to the next stage of the pipeline, another instance of the filter might be run, and it'll post a copy of the same message to the pipeline.</span></span> <span data-ttu-id="53afc-159">Ez ugyanaz az üzenet a következő szűrő átadandó két példánya okozhat.</span><span class="sxs-lookup"><span data-stu-id="53afc-159">This could cause two instances of the same message to be passed to the next filter.</span></span> <span data-ttu-id="53afc-160">Ennek elkerülése érdekében a feldolgozási sorban lévő kell észlelése és kiküszöbölheti a duplikált üzenetek.</span><span class="sxs-lookup"><span data-stu-id="53afc-160">To avoid this, the pipeline should detect and eliminate duplicate messages.</span></span>

    >  <span data-ttu-id="53afc-161">Üzenetsorok (például a Microsoft Azure Service Bus-üzenetsorok) használatával megvalósításához a folyamatot, ha az üzenet üzenetsor-kezelési infrastruktúra nyújthatnak automatikus duplikált üzenetek észlelésének és eltávolítása.</span><span class="sxs-lookup"><span data-stu-id="53afc-161">If you're implementing the pipeline by using message queues (such as Microsoft Azure Service Bus queues), the message queuing infrastructure might provide automatic duplicate message detection and removal.</span></span>

- <span data-ttu-id="53afc-162">**És az állapot**.</span><span class="sxs-lookup"><span data-stu-id="53afc-162">**Context and state**.</span></span> <span data-ttu-id="53afc-163">Egy sorban a minden szűrő lényegében elkülönítési fut, és ne ellenőrizze a bármely feltételezéseket hogyan lett meghívva.</span><span class="sxs-lookup"><span data-stu-id="53afc-163">In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked.</span></span> <span data-ttu-id="53afc-164">Ez azt jelenti, hogy minden szűrő a munkájuk elvégzéséhez elegendő kontextusú kell megadni.</span><span class="sxs-lookup"><span data-stu-id="53afc-164">This means that each filter should be provided with sufficient context to perform its work.</span></span> <span data-ttu-id="53afc-165">Ebben a környezetben állapotadatokat nagy mennyiségű tartalmazhatnak.</span><span class="sxs-lookup"><span data-stu-id="53afc-165">This context could include a large amount of state information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="53afc-166">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="53afc-166">When to use this pattern</span></span>

<span data-ttu-id="53afc-167">Ez mintát, mikor használja:</span><span class="sxs-lookup"><span data-stu-id="53afc-167">Use this pattern when:</span></span>
- <span data-ttu-id="53afc-168">Az alkalmazás által igényelt feldolgozását is könnyen oszlanak független ismertetett lépések.</span><span class="sxs-lookup"><span data-stu-id="53afc-168">The processing required by an application can easily be broken down into a set of independent steps.</span></span>

- <span data-ttu-id="53afc-169">A feldolgozási lépéseket végzi el az alkalmazás különböző méretezhetőség követelményekkel rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="53afc-169">The processing steps performed by an application have different scalability requirements.</span></span>

    >  <span data-ttu-id="53afc-170">Lehetőség biztonságicsoport-szűrőkkel kell méretezni együtt ugyanabban a folyamatban.</span><span class="sxs-lookup"><span data-stu-id="53afc-170">It's possible to group filters that should scale together in the same process.</span></span> <span data-ttu-id="53afc-171">További információkért lásd: a [számítási erőforrás-összevonási mintát](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="53afc-171">For more information, see the [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

- <span data-ttu-id="53afc-172">Rugalmasság szükséges rendelje újra a feldolgozási lépéseket végzi el egy alkalmazást vagy a funkció hozzáadása és eltávolítása a lépéseket a engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="53afc-172">Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.</span></span>

- <span data-ttu-id="53afc-173">A rendszer vonatkozhat a lépéseket a feldolgozási terjesztése a különféle kiszolgálók között.</span><span class="sxs-lookup"><span data-stu-id="53afc-173">The system can benefit from distributing the processing for steps across different servers.</span></span>

- <span data-ttu-id="53afc-174">A megbízható megoldás szükség, amely minimálisra csökkenti egy lépésben sikertelensége által okozott hatások, amíg az adatok feldolgozása folyamatban van.</span><span class="sxs-lookup"><span data-stu-id="53afc-174">A reliable solution is required that minimizes the effects of failure in a step while data is being processed.</span></span>

<span data-ttu-id="53afc-175">Ebben a mintában előfordulhat, hogy nem lehet hasznos:</span><span class="sxs-lookup"><span data-stu-id="53afc-175">This pattern might not be useful when:</span></span>
- <span data-ttu-id="53afc-176">A feldolgozási lépéseket végzi el az alkalmazás nem független, és/vagy együtt ugyanabban a tranzakcióban részeként végrehajtását.</span><span class="sxs-lookup"><span data-stu-id="53afc-176">The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.</span></span>

- <span data-ttu-id="53afc-177">A lépés szükséges környezet vagy az állapot adatmennyiség nem elég hatékony teszi ezt a módszert használja.</span><span class="sxs-lookup"><span data-stu-id="53afc-177">The amount of context or state information required by a step makes this approach inefficient.</span></span> <span data-ttu-id="53afc-178">Ehelyett megőrizni az adatait az adatbázis esetleg, de nem használja ezt a stratégiát, ha az adatbázis további terhelése túl sok versengés hatására.</span><span class="sxs-lookup"><span data-stu-id="53afc-178">It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.</span></span>

## <a name="example"></a><span data-ttu-id="53afc-179">Példa</span><span class="sxs-lookup"><span data-stu-id="53afc-179">Example</span></span>

<span data-ttu-id="53afc-180">Üzenet-várólistákból sorozatát segítségével biztosítja a folyamat végrehajtásához szükséges infrastruktúrát.</span><span class="sxs-lookup"><span data-stu-id="53afc-180">You can use a sequence of message queues to provide the infrastructure required to implement a pipeline.</span></span> <span data-ttu-id="53afc-181">Az első üzenet-várólista feldolgozatlan üzeneteket fogad.</span><span class="sxs-lookup"><span data-stu-id="53afc-181">An initial message queue receives unprocessed messages.</span></span> <span data-ttu-id="53afc-182">Egy szűrő tevékenységként megvalósított összetevő figyeli az ennél a várakozási üzenet, a tevékenységeket hajt végre, és majd küldi az átalakított üzenet a következő várólistára sorrendben.</span><span class="sxs-lookup"><span data-stu-id="53afc-182">A component implemented as a filter task listens for a message on this queue, performs its work, and then posts the transformed message to the next queue in the sequence.</span></span> <span data-ttu-id="53afc-183">Egy másik szűrő feladat az ennél a várakozási üzeneteket, dolgozza fel őket, és így tovább utáni a eredményeket egy másik várólistához, amíg a teljes átalakított adatok megjelennek-e a várólista utolsó üzenetébe.</span><span class="sxs-lookup"><span data-stu-id="53afc-183">Another filter task can listen for messages on this queue, process them, post the results to another queue, and so on until the fully transformed data appears in the final message in the queue.</span></span> <span data-ttu-id="53afc-184">Az alábbi ábrán láthatja a végrehajtási folyamat üzenetsorok használatával.</span><span class="sxs-lookup"><span data-stu-id="53afc-184">The next figure illustrates implementing a pipeline using message queues.</span></span>

![4. ábra - üzenetsorok használatával folyamat végrehajtása](./_images/pipes-and-filters-message-queues.png)


<span data-ttu-id="53afc-186">Ha van olyan megoldást épülő Azure Service Bus-üzenetsorok használatával egy olyan mechanizmust, megbízható és méretezhető Üzenetsor-kezelés.</span><span class="sxs-lookup"><span data-stu-id="53afc-186">If you're building a solution on Azure you can use Service Bus queues to provide a reliable and scalable queuing mechanism.</span></span> <span data-ttu-id="53afc-187">A `ServiceBusPipeFilter` osztály a C# alább látható bemutatja, hogyan implementálható, amely megkapja a bemeneti üzenetet az üzenetsorból, ezek az üzenetek folyamatokat, és egy másik várólistához eredmények visszaküldés szűrőt.</span><span class="sxs-lookup"><span data-stu-id="53afc-187">The `ServiceBusPipeFilter` class shown below in C# demonstrates how you can implement a filter that receives input messages from a queue, processes these messages, and posts the results to another queue.</span></span>

>  <span data-ttu-id="53afc-188">A `ServiceBusPipeFilter` osztály definiálva van a PipesAndFilters.Shared projekt eléréséhez [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="53afc-188">The `ServiceBusPipeFilter` class is defined in the PipesAndFilters.Shared project available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

<span data-ttu-id="53afc-189">A `Start` metódust a `ServiceBusPipeFilter` osztály csatlakozik a két bemeneti és kimeneti várólisták, és a `Close` metódus bontja a kapcsolatot a bemeneti várólistát.</span><span class="sxs-lookup"><span data-stu-id="53afc-189">The `Start` method in the `ServiceBusPipeFilter` class connects to a pair of input and output queues, and the `Close` method disconnects from the input queue.</span></span> <span data-ttu-id="53afc-190">A `OnPipeFilterMessageAsync` metódus hajtja végre a tényleges feldolgozását az üzeneteket, a `asyncFilterTask` ezt a módszert paraméter határozza meg a feldolgozás végezhető el.</span><span class="sxs-lookup"><span data-stu-id="53afc-190">The `OnPipeFilterMessageAsync` method performs the actual processing of messages, the `asyncFilterTask` parameter to this method specifies the processing to be performed.</span></span> <span data-ttu-id="53afc-191">A `OnPipeFilterMessageAsync` a bemeneti várólistára, a bejövő üzenetek metódust vár a megadott kód futtatása a `asyncFilterTask` paraméter keresztül minden üzenetet, mert megérkeznek, és az eredményeket a kimeneti várólista küldi.</span><span class="sxs-lookup"><span data-stu-id="53afc-191">The `OnPipeFilterMessageAsync` method waits for incoming messages on the input queue, runs the code specified by the `asyncFilterTask` parameter over each message as it arrives, and posts the results to the output queue.</span></span> <span data-ttu-id="53afc-192">A várólisták magukat a gyártó által meg van adva.</span><span class="sxs-lookup"><span data-stu-id="53afc-192">The queues themselves are specified by the constructor.</span></span>

<span data-ttu-id="53afc-193">A minta megoldás szűrők feldolgozói szerepkörök meg valósítja meg.</span><span class="sxs-lookup"><span data-stu-id="53afc-193">The sample solution implements filters in a set of worker roles.</span></span> <span data-ttu-id="53afc-194">Egymástól függetlenül, minden egyes feldolgozói szerepkör is méretezhető, amely elvégzi az üzleti feldolgozása vagy a feldolgozáshoz szükséges erőforrások összetettségétől függően.</span><span class="sxs-lookup"><span data-stu-id="53afc-194">Each worker role can be scaled independently, depending on the complexity of the business processing that it performs or the resources required for processing.</span></span> <span data-ttu-id="53afc-195">Emellett minden egyes feldolgozói szerepkör több példánya is futtatható párhuzamos átviteli sebesség növelése érdekében.</span><span class="sxs-lookup"><span data-stu-id="53afc-195">Additionally, multiple instances of each worker role can be run in parallel to improve throughput.</span></span>

<span data-ttu-id="53afc-196">A következő kód bemutatja egy Azure feldolgozói szerepkörnek nevű `PipeFilterARoleEntry`az PipeFilterA projekt, a megoldást a definiált.</span><span class="sxs-lookup"><span data-stu-id="53afc-196">The following code shows an Azure worker role named `PipeFilterARoleEntry`, defined in the PipeFilterA project in the sample solution.</span></span>

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

<span data-ttu-id="53afc-197">Ez a szerepkör tartalmazza a `ServiceBusPipeFilter` objektum.</span><span class="sxs-lookup"><span data-stu-id="53afc-197">This role contains a `ServiceBusPipeFilter` object.</span></span> <span data-ttu-id="53afc-198">A `OnStart` metódus a szerepkör a fogadással bemeneti és kimeneti üzenetek közzététele a várólisták csatlakozik (határozzák meg a várólista nevét a `Constants` osztály).</span><span class="sxs-lookup"><span data-stu-id="53afc-198">The `OnStart` method in the role connects to the queues for receiving input messages and posting output messages (the names of the queues are defined in the `Constants` class).</span></span> <span data-ttu-id="53afc-199">A `Run` metódus meghívja a `OnPipeFilterMessagesAsync` egyes feldolgozási végre minden üzenetet kapott metódus (ebben a példában a feldolgozási szimulálják várakozással rövid idő alatt).</span><span class="sxs-lookup"><span data-stu-id="53afc-199">The `Run` method invokes the `OnPipeFilterMessagesAsync` method to perform some processing on each message that's received (in this example, the processing is simulated by waiting for a short period of time).</span></span> <span data-ttu-id="53afc-200">Ha feldolgozása befejeződött, egy új üzenet összeállított az eredményeket tartalmazó (a bemeneti üzenet van ebben az esetben a hozzáadott egyéni tulajdonság is), és ezt az üzenetet a rendszer visszaküldi a kimeneti várólistában.</span><span class="sxs-lookup"><span data-stu-id="53afc-200">When processing is complete, a new message is constructed containing the results (in this case, the input message has a custom property added), and this message is posted to the output queue.</span></span>

<span data-ttu-id="53afc-201">A mintakód tartalmaz egy másik feldolgozói szerepkör nevű `PipeFilterBRoleEntry` a PipeFilterB projektben.</span><span class="sxs-lookup"><span data-stu-id="53afc-201">The sample code contains another worker role named `PipeFilterBRoleEntry` in the PipeFilterB project.</span></span> <span data-ttu-id="53afc-202">Ez a szerepkör hasonlít `PipeFilterARoleEntry` azzal a különbséggel, hogy a különböző feldolgozó segítségével végzi a `Run` metódust.</span><span class="sxs-lookup"><span data-stu-id="53afc-202">This role is similar to `PipeFilterARoleEntry` except that it performs different processing in the `Run` method.</span></span> <span data-ttu-id="53afc-203">A példa a megoldásban ez a két szerepkör kombinált egy folyamatot, a kimeneti várólista összeállítani a `PipeFilterARoleEntry` szerepköre a bemeneti várólista a `PipeFilterBRoleEntry` szerepkör.</span><span class="sxs-lookup"><span data-stu-id="53afc-203">In the example solution, these two roles are combined to construct a pipeline, the output queue for the `PipeFilterARoleEntry` role is the input queue for the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="53afc-204">A minta megoldás is biztosít két további szerepkörök nevű `InitialSenderRoleEntry` (a projektben InitialSender) és `FinalReceiverRoleEntry` (a projektben FinalReceiver).</span><span class="sxs-lookup"><span data-stu-id="53afc-204">The sample solution also provides two additional roles named `InitialSenderRoleEntry` (in the InitialSender project) and `FinalReceiverRoleEntry` (in the FinalReceiver project).</span></span> <span data-ttu-id="53afc-205">A `InitialSenderRoleEntry` szerepkör szolgáltatás a kezdeti üzenetet jelenít meg a folyamat.</span><span class="sxs-lookup"><span data-stu-id="53afc-205">The `InitialSenderRoleEntry` role provides the initial message in the pipeline.</span></span> <span data-ttu-id="53afc-206">A `OnStart` módszer egy adott sorba csatlakozik, és a `Run` metódus által a várólistának metódus.</span><span class="sxs-lookup"><span data-stu-id="53afc-206">The `OnStart` method connects to a single queue and the `Run` method posts a method to this queue.</span></span> <span data-ttu-id="53afc-207">A várólista által használt bemeneti várólista a `PipeFilterARoleEntry` szerepkör, ezért fogad és dolgoz fel üzenetet egy üzenet azt eredményezi a `PipeFilterARoleEntry` szerepkör.</span><span class="sxs-lookup"><span data-stu-id="53afc-207">This queue is the input queue used by the `PipeFilterARoleEntry` role, so posting a message to it causes the message to be received and processed by the `PipeFilterARoleEntry` role.</span></span> <span data-ttu-id="53afc-208">A feldolgozott üzenet majd haladnak keresztül az `PipeFilterBRoleEntry` szerepkör.</span><span class="sxs-lookup"><span data-stu-id="53afc-208">The processed message then passes through the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="53afc-209">A bemeneti várólista a `FinalReceiveRoleEntry` szerepköre a kimeneti várólista a `PipeFilterBRoleEntry` szerepkör.</span><span class="sxs-lookup"><span data-stu-id="53afc-209">The input queue for the `FinalReceiveRoleEntry` role is the output queue for the `PipeFilterBRoleEntry` role.</span></span> <span data-ttu-id="53afc-210">A `Run` metódust a `FinalReceiveRoleEntry` szerepkör az alábbi megkapja az üzenetet, és néhány végső feldolgozásra.</span><span class="sxs-lookup"><span data-stu-id="53afc-210">The `Run` method in the `FinalReceiveRoleEntry` role, shown below, receives the message and performs some final processing.</span></span> <span data-ttu-id="53afc-211">Ezután ír a követés eredményét a szűrők a folyamat által hozzáadott egyéni tulajdonságok értékeit.</span><span class="sxs-lookup"><span data-stu-id="53afc-211">Then it writes the values of the custom properties added by the filters in the pipeline to the trace output.</span></span>

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

##<a name="related-patterns-and-guidance"></a><span data-ttu-id="53afc-212">Útmutató és a kapcsolódó minták</span><span class="sxs-lookup"><span data-stu-id="53afc-212">Related patterns and guidance</span></span>

<span data-ttu-id="53afc-213">A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:</span><span class="sxs-lookup"><span data-stu-id="53afc-213">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="53afc-214">Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="53afc-214">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>
- <span data-ttu-id="53afc-215">[Versengő fogyasztók mintát](competing-consumers.md).</span><span class="sxs-lookup"><span data-stu-id="53afc-215">[Competing Consumers pattern](competing-consumers.md).</span></span> <span data-ttu-id="53afc-216">Egy folyamat egy vagy több szűrő több példányát is tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="53afc-216">A pipeline can contain multiple instances of one or more filters.</span></span> <span data-ttu-id="53afc-217">Ez a megközelítés futó párhuzamos példányait lassú szűrők engedélyezésével a rendszer egyenletes terhelését, valamint javítja a teljesítményt.</span><span class="sxs-lookup"><span data-stu-id="53afc-217">This approach is useful for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span> <span data-ttu-id="53afc-218">Szűrő minden példánya a rendszer "versenyeznek" az bemeneti a más osztályt, szűrő két példánya nem fogja tudni feldolgozni ugyanazokat az adatokat.</span><span class="sxs-lookup"><span data-stu-id="53afc-218">Each instance of a filter will compete for input with the other instances, two instances of a filter shouldn't be able to process the same data.</span></span> <span data-ttu-id="53afc-219">Ez a megközelítés magyarázattal szolgál.</span><span class="sxs-lookup"><span data-stu-id="53afc-219">Provides an explanation of this approach.</span></span>
- <span data-ttu-id="53afc-220">[Számítási erőforrás-összevonási mintát](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="53afc-220">[Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="53afc-221">Biztonságicsoport-szűrőkkel, amelyek ugyanabba a folyamatba együtt kell méretezni is lehet.</span><span class="sxs-lookup"><span data-stu-id="53afc-221">It might be possible to group filters that should scale together into the same process.</span></span> <span data-ttu-id="53afc-222">További információt az előnyöket és a stratégia kompromisszumot nyújt.</span><span class="sxs-lookup"><span data-stu-id="53afc-222">Provides more information about the benefits and tradeoffs of this strategy.</span></span>
- <span data-ttu-id="53afc-223">[Kompenzációs tranzakció mintát](compensating-transaction.md).</span><span class="sxs-lookup"><span data-stu-id="53afc-223">[Compensating Transaction pattern](compensating-transaction.md).</span></span> <span data-ttu-id="53afc-224">Egy szűrő, amely visszafordítható, illetve egy kompenzációs művelet, amely visszaállítja az állapotát egy korábbi verzióját meghibásodása lett műveletként valósítható meg.</span><span class="sxs-lookup"><span data-stu-id="53afc-224">A filter can be implemented as an operation that can be reversed, or that has a compensating operation that restores the state to a previous version in the event of a failure.</span></span> <span data-ttu-id="53afc-225">Ismerteti, hogyan ez valósítható karbantartása, vagy a végleges konzisztencia elérése.</span><span class="sxs-lookup"><span data-stu-id="53afc-225">Explains how this can be implemented to maintain or achieve eventual consistency.</span></span>
- <span data-ttu-id="53afc-226">[Idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathan Oliver blogjában.</span><span class="sxs-lookup"><span data-stu-id="53afc-226">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>
