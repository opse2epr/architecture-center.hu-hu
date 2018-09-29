---
title: Pipes and Filters
description: Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: fd616676f9487bdfe1bf23b3d0fec6c65b97a8f4
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429570"
---
# <a name="pipes-and-filters-pattern"></a><span data-ttu-id="aa6dc-104">Csövek és szűrők mintája</span><span class="sxs-lookup"><span data-stu-id="aa6dc-104">Pipes and Filters pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="aa6dc-105">Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-105">Decompose a task that performs complex processing into a series of separate elements that can be reused.</span></span> <span data-ttu-id="aa6dc-106">Ezzel javítható a teljesítmény, a skálázhatóság és az újrahasznosíthatóság, mivel lehetővé teszi a feldolgozást végső feladatelemek független üzembe helyezését és skálázását.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-106">This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="aa6dc-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="aa6dc-107">Context and problem</span></span>

<span data-ttu-id="aa6dc-108">Az alkalmazásnak több, változó összetettségű feladatot kell végrehajtania a feldolgozandó adatokon.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-108">An application is required to perform a variety of tasks of varying complexity on the information that it processes.</span></span> <span data-ttu-id="aa6dc-109">Egy egyszerű, de rugalmatlan megközelítés az alkalmazások megvalósításához, ha ezt a feldolgozást egy monolitikus modulként hajtatjuk végre.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-109">A straightforward but inflexible approach to implementing an application is to perform this processing as a monolithic module.</span></span> <span data-ttu-id="aa6dc-110">Ez a megközelítés azonban valószínűleg csökkenti a kód átdolgozásának, optimalizálásának vagy ismételt felhasználásának lehetőségét, amennyiben ugyanilyen feldolgozás máshol is szükséges lenne az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-110">However, this approach is likely to reduce the opportunities for refactoring the code, optimizing it, or reusing it if parts of the same processing are required elsewhere within the application.</span></span>

<span data-ttu-id="aa6dc-111">Az ábra az adatfeldolgozással kapcsolatos problémákat szemlélteti a monolitikus megközelítés alkalmazásakor.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-111">The figure illustrates the issues with processing data using the monolithic approach.</span></span> <span data-ttu-id="aa6dc-112">Az alkalmazások két forrásból fogadnak és dolgoznak fel adatokat.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-112">An application receives and processes data from two sources.</span></span> <span data-ttu-id="aa6dc-113">Az egyes forrásokból származó adatok feldolgozása egy-egy különálló modul feladata, amely elvégzi az adatok átalakításához szükséges feladatokat, majd az eredményt az alkalmazás üzleti logikája számára továbbítja.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-113">The data from each source is processed by a separate module that performs a series of tasks to transform this data, before passing the result to the business logic of the application.</span></span>

![1. ábra: Monolitikus modulok használatával implementált megoldás](./_images/pipes-and-filters-modules.png)

<span data-ttu-id="aa6dc-115">A monolitikus modulok által elvégzett feladatok egy része funkcionálisan nagyon hasonló, de az egyes modulok megtervezése külön-külön történt.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-115">Some of the tasks that the monolithic modules perform are functionally very similar, but the modules have been designed separately.</span></span> <span data-ttu-id="aa6dc-116">A feladatokat implementáló kód szorosan össze van kapcsolva a modulokban, és annak kifejlesztése során kevésbé vagy egyáltalán nem vették figyelembe az újrafelhasználhatóságot vagy a skálázhatóságot.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-116">The code that implements the tasks is closely coupled in a module, and has been developed with little or no thought given to reuse or scalability.</span></span>

<span data-ttu-id="aa6dc-117">Az egyes modulok által végrehajtott feldolgozási feladatok, illetve az egyes feladatok üzembe helyezési követelményei azonban megváltozhatnak az üzleti követelmények változásával.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-117">However, the processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated.</span></span> <span data-ttu-id="aa6dc-118">Egyes feladatok nagy számításigényűek lehetnek, így nagy teljesítményű hardveren futtathatók hatékonyan, míg mások nem igényelnek ennyire költséges erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-118">Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources.</span></span> <span data-ttu-id="aa6dc-119">A jövőben azonban további feldolgozásra is szükség lehet, vagy módosulhat a feladatok feldolgozási sorrendje.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-119">Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.</span></span> <span data-ttu-id="aa6dc-120">Olyan megoldásra van szükség, amellyel megoldhatók ezek a problémák, és nő a kód újrafelhasználhatóságának lehetősége.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-120">A solution is required that addresses these issues, and increases the possibilities for code reuse.</span></span>

## <a name="solution"></a><span data-ttu-id="aa6dc-121">Megoldás</span><span class="sxs-lookup"><span data-stu-id="aa6dc-121">Solution</span></span>

<span data-ttu-id="aa6dc-122">Ossza fel az egyes streamekhez tartozó feldolgozási folyamatot olyan különálló összetevőkre (vagy szűrőkre), amelyek mindegyike egyetlen feladatot végez.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-122">Break down the processing required for each stream into a set of separate components (or filters), each performing a single task.</span></span> <span data-ttu-id="aa6dc-123">Az egyes összetevők által küldött és fogadott adatok formátumának szabványosításával ezek a szűrők egy folyamattá kombinálhatóak.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-123">By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline.</span></span> <span data-ttu-id="aa6dc-124">Ezzel elkerülhető a kód ismétlődése, illetve megkönnyíti további összetevők eltávolítását, cseréjét vagy integrálását, ha a feldolgozási követelmények változnak.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-124">This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.</span></span> <span data-ttu-id="aa6dc-125">A következő ábra egy csövek és szűrők használatával implementált megoldást mutat be.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-125">The next figure shows a solution implemented using pipes and filters.</span></span>

![2. ábra – Csövek és szűrők használatával implementált megoldás](./_images/pipes-and-filters-solution.png)


<span data-ttu-id="aa6dc-127">Az egy kérelem feldolgozásához szükséges idő a folyamat leglassabb szűrőjének sebességétől függ.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-127">The time it takes to process a single request depends on the speed of the slowest filter in the pipeline.</span></span> <span data-ttu-id="aa6dc-128">Egy vagy több szűrő szűk keresztmetszetté válhat, különösen akkor, ha nagy számú kérelem található egy adott adatforrástól érkező streamben.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-128">One or more filters could be a bottleneck, especially if a large number of requests appear in a stream from a particular data source.</span></span> <span data-ttu-id="aa6dc-129">A folyamatstruktúra legfontosabb előnye, hogy lehetőséget biztosít a lassú szűrők példányainak egyidejű futtatására, lehetővé téve ezzel a rendszer számára a terhelés elosztását és a teljesítmény növelését.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-129">A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span>

<span data-ttu-id="aa6dc-130">A folyamatot alkotó szűrők különböző gépeken futhatnak, így egymástól függetlenül skálázhatóak, és kihasználhatják a számos felhőkörnyezet által nyújtott rugalmasságot.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-130">The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.</span></span> <span data-ttu-id="aa6dc-131">A nagy számítási igényű szűrők nagy teljesítményű hardveren futhatnak, míg más, kevésbé erőforrás-igényes szűrők üzemeltethetők kevésbé költséges, hagyományos hardveren is.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-131">A filter that is computationally intensive can run on high performance hardware, while other less demanding filters can be hosted on less expensive commodity hardware.</span></span> <span data-ttu-id="aa6dc-132">A szűrőknek még csak nem is kell azonos adatközpontban vagy földrajzi helyen lenniük, ami lehetővé teszi, hogy az egyes folyamatelemek olyan környezetben futhassanak, amely közel van a számukra szükséges erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-132">The filters don't even have to be in the same data center or geographical location, which allows each element in a pipeline to run in an environment that is close to the resources it requires.</span></span>  <span data-ttu-id="aa6dc-133">A következő ábrán látható példa az 1. forrásból származó adatokhoz tartozó folyamatra van alkalmazva.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-133">The next figure shows an example applied to the pipeline for the data from Source 1.</span></span>

![A 3. ábrán látható példa az 1. forrásból származó adatokhoz tartozó folyamatra van alkalmazva](./_images/pipes-and-filters-load-balancing.png)

<span data-ttu-id="aa6dc-135">Ha a szűrők be- és kimenete streamként van felépítve, akkor lehetséges az egyes szűrők általi egyidejű feldolgozás is.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-135">If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel.</span></span> <span data-ttu-id="aa6dc-136">A folyamat első szűrője elkezd működni, és létrehozza a kimenetét, amelyet a rendszer még az előtt a soron következő szűrőnek továbbít, hogy az első szűrő végrehajtaná a feladatát.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-136">The first filter in the pipeline can start its work and output its results, which are passed directly on to the next filter in the sequence before the first filter has completed its work.</span></span>

<span data-ttu-id="aa6dc-137">További előnyt jelent a modell által biztosított rugalmasság.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-137">Another benefit is the resiliency that this model can provide.</span></span> <span data-ttu-id="aa6dc-138">Ha egy szűrő meghibásodik vagy ha az azt futtató gép már nem érhető el, akkor a folyamat képes átütemezni a szűrő által végrehajtott feladatot és az összetevő másik példányára átirányítani.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-138">If a filter fails or the machine it's running on is no longer available, the pipeline can reschedule the work that the filter was performing and direct this work to another instance of the component.</span></span> <span data-ttu-id="aa6dc-139">Egyetlen szűrő meghibásodása nem feltétlenül eredményezi a teljes folyamat meghibásodását.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-139">Failure of a single filter doesn't necessarily result in failure of the entire pipeline.</span></span>

<span data-ttu-id="aa6dc-140">A Csövek és szűrők minta és a [Kompenzáló tranzakció minta](compensating-transaction.md) együttes alkalmazása is egy lehetséges alternatív módszer az elosztott tranzakciók implementálására.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-140">Using the Pipes and Filters pattern in conjunction with the [Compensating Transaction pattern](compensating-transaction.md) is an alternative approach to implementing distributed transactions.</span></span> <span data-ttu-id="aa6dc-141">Az elosztott tranzakciók különálló, kompenzálható feladatokra bonthatók le, amelyek egy olyan szűrővel implementálhatók, amely a Kompenzáló tranzakció mintát is implementálja.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-141">A distributed transaction can be broken down into separate, compensable tasks, each of which can be implemented by using a filter that also implements the Compensating Transaction pattern.</span></span> <span data-ttu-id="aa6dc-142">A folyamatban lévő szűrők olyan önállóan üzemeltetett feladatokként is implementálhatók, amelyeket a rendszer az általuk karbantartott adatok közelében futtat.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-142">The filters in a pipeline can be implemented as separate hosted tasks running close to the data that they maintain.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="aa6dc-143">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="aa6dc-143">Issues and considerations</span></span>

<span data-ttu-id="aa6dc-144">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="aa6dc-144">You should consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="aa6dc-145">**Összetettség**.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-145">**Complexity**.</span></span> <span data-ttu-id="aa6dc-146">A minta által biztosított megnövelt rugalmasság összetettséget is eredményezhet, különösen akkor, ha a folyamat szűrői különböző kiszolgálókon vannak elosztva.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-146">The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.</span></span>

- <span data-ttu-id="aa6dc-147">**Megbízhatóság**.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-147">**Reliability**.</span></span> <span data-ttu-id="aa6dc-148">Olyan infrastruktúrát használjon, amely biztosítja, hogy ne legyen adatvesztés a folyamat szűrői közötti adatátvitel során.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-148">Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.</span></span>

- <span data-ttu-id="aa6dc-149">**Idempotencia**.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-149">**Idempotency**.</span></span> <span data-ttu-id="aa6dc-150">Ha egy üzenet fogadását követően a folyamatban lévő szűrő meghibásodik, és a rendszer a feladatot a szűrő egy másik példányára ütemezi át, akkor lehetséges, hogy a feladat egy része már végre lett hajtva.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-150">If a filter in a pipeline fails after receiving a message and the work is rescheduled to another instance of the filter, part of the work might have already been completed.</span></span> <span data-ttu-id="aa6dc-151">Ha ez a feladat frissíti a globális állapot egyes aspektusait (például az adatbázisban tárolt információkat), akkor meg lehet ismételni ugyanezt a frissítést.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-151">If this work updates some aspect of the global state (such as information stored in a database), the same update could be repeated.</span></span> <span data-ttu-id="aa6dc-152">Hasonló probléma fordulhat elő akkor, ha egy szűrő azután hibásodik meg, miután közzétette az eredményeit a folyamat következő szűrőjének, de még nem jelezte, hogy a feladatot sikeresen elvégezte.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-152">A similar issue might occur if a filter fails after posting its results to the next filter in the pipeline, but before indicating that it's completed its work successfully.</span></span> <span data-ttu-id="aa6dc-153">Ezekben az esetekben lehetséges, hogy ugyanazt a feladatot a szűrő egy másik példánya is elvégzi, így ugyanazok az eredmények kétszer lesznek közzétéve.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-153">In these cases, the same work could be repeated by another instance of the filter, causing the same results to be posted twice.</span></span> <span data-ttu-id="aa6dc-154">Ez azt eredményezheti, hogy a folyamat soron következő szűrői ugyanazokat az adatokat kétszer fogják feldolgozni.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-154">This could result in subsequent filters in the pipeline processing the same data twice.</span></span> <span data-ttu-id="aa6dc-155">Ezért a folyamatok szűrőinek idempotensnek kell lenniük.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-155">Therefore filters in a pipeline should be designed to be idempotent.</span></span> <span data-ttu-id="aa6dc-156">További információ: [Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) (Jonathan Oliver blogjában).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-156">For more information see [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

- <span data-ttu-id="aa6dc-157">**Ismétlődő üzenetek**.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-157">**Repeated messages**.</span></span> <span data-ttu-id="aa6dc-158">Ha egy folyamat szűrője azután hibásodik meg, hogy közzétett egy üzenetet a folyamat következő szakaszában, akkor a rendszer futtathatja a szűrő másik példányát, és az ugyanennek az üzenetnek a másolatát fogja közzétenni a folyamatban.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-158">If a filter in a pipeline fails after posting a message to the next stage of the pipeline, another instance of the filter might be run, and it'll post a copy of the same message to the pipeline.</span></span> <span data-ttu-id="aa6dc-159">Ez azt eredményezheti, hogy ugyanazon üzenet két példánya lesz elküldve a következő szűrő számára.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-159">This could cause two instances of the same message to be passed to the next filter.</span></span> <span data-ttu-id="aa6dc-160">Ennek elkerülése érdekében a folyamatnak észlelnie kell és el kell távolítania az ismétlődő üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-160">To avoid this, the pipeline should detect and eliminate duplicate messages.</span></span>

    >  <span data-ttu-id="aa6dc-161">Ha az adott folyamatot üzenetsorok (például Microsoft Azure Service Bus-üzenetsorok) használatával implementálja, az üzenetsor-kezelési infrastruktúra az ismétlődő üzenetek automatikus felismerését és eltávolítását biztosíthatja.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-161">If you're implementing the pipeline by using message queues (such as Microsoft Azure Service Bus queues), the message queuing infrastructure might provide automatic duplicate message detection and removal.</span></span>

- <span data-ttu-id="aa6dc-162">**Kontextus és állapot**.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-162">**Context and state**.</span></span> <span data-ttu-id="aa6dc-163">A folyamatokban a szűrők gyakorlatilag elkülönítve futnak, és nem tudják feltételezni azok aktiválásuk módját.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-163">In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked.</span></span> <span data-ttu-id="aa6dc-164">Ez azt jelenti, hogy mindegyik szűrő számára a feladatának elvégzéséhez megfelelő kontextust kell biztosítani.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-164">This means that each filter should be provided with sufficient context to perform its work.</span></span> <span data-ttu-id="aa6dc-165">A kontextus nagy mennyiségű állapotinformációt is tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-165">This context could include a large amount of state information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="aa6dc-166">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="aa6dc-166">When to use this pattern</span></span>

<span data-ttu-id="aa6dc-167">Használja ezt a mintát, ha:</span><span class="sxs-lookup"><span data-stu-id="aa6dc-167">Use this pattern when:</span></span>
- <span data-ttu-id="aa6dc-168">Az alkalmazások számára szükséges feldolgozási folyamat egyszerűen lebontható független lépések sorozatára.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-168">The processing required by an application can easily be broken down into a set of independent steps.</span></span>

- <span data-ttu-id="aa6dc-169">Az alkalmazások által végrehajtott feldolgozási lépések különböző skálázhatósági követelményekkel rendelkeznek.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-169">The processing steps performed by an application have different scalability requirements.</span></span>

    >  <span data-ttu-id="aa6dc-170">Lehetőség van csoportosítani olyan szűrőket, amelyeknek azonos skálázásúaknak kell lenniük egy adott folyamatban.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-170">It's possible to group filters that should scale together in the same process.</span></span> <span data-ttu-id="aa6dc-171">További információkért lásd a [számításierőforrás-konszolidálási mintát](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-171">For more information, see the [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

- <span data-ttu-id="aa6dc-172">Rugalmas kialakítás szükséges ahhoz, hogy lehetővé váljon az alkalmazás által végrehajtott feldolgozási lépések sorrendjének módosítása, illetve lépések hozzáadása és eltávolítása.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-172">Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.</span></span>

- <span data-ttu-id="aa6dc-173">A rendszer számára előnyös lehet, ha az egyes lépésekhez tartozó feldolgozási folyamatok különböző kiszolgálókra vannak elosztva.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-173">The system can benefit from distributing the processing for steps across different servers.</span></span>

- <span data-ttu-id="aa6dc-174">Olyan megbízható megoldásra van szükség, amely minimálisra csökkenti egy adott lépés meghibásodásának hatását az adatok feldolgozása során.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-174">A reliable solution is required that minimizes the effects of failure in a step while data is being processed.</span></span>

<span data-ttu-id="aa6dc-175">Nem érdemes ezt a mintát használni, ha:</span><span class="sxs-lookup"><span data-stu-id="aa6dc-175">This pattern might not be useful when:</span></span>
- <span data-ttu-id="aa6dc-176">Az alkalmazások által végrehajtott feldolgozási lépések nem függetlenek egymástól, illetve azokat egy adott tranzakció részeként, együtt kell végrehajtani.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-176">The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.</span></span>

- <span data-ttu-id="aa6dc-177">Az egyes lépések végrehajtásához szükséges kontextus- vagy állapotinformációk mennyisége csökkenti ennek a módszernek a hatékonyságát.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-177">The amount of context or state information required by a step makes this approach inefficient.</span></span> <span data-ttu-id="aa6dc-178">Az állapotinformációkat továbbra is lehet inkább adatbázisban megőrizni, de ne használja ezt a stratégiát, ha az adatbázis további terhelése túlzott mértékű versenyt okozna.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-178">It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.</span></span>

## <a name="example"></a><span data-ttu-id="aa6dc-179">Példa</span><span class="sxs-lookup"><span data-stu-id="aa6dc-179">Example</span></span>

<span data-ttu-id="aa6dc-180">A folyamat implementálásához szükséges infrastruktúra biztosításához üzenetsorok sorozatát is használhatja.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-180">You can use a sequence of message queues to provide the infrastructure required to implement a pipeline.</span></span> <span data-ttu-id="aa6dc-181">A kezdő üzenetsor feldolgozatlan üzeneteket kap.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-181">An initial message queue receives unprocessed messages.</span></span> <span data-ttu-id="aa6dc-182">A szűrési feladatként megvalósított összetevő figyeli az adott üzenetsor üzeneteit, elvégzi a feladatát, majd közzéteszi az átalakított üzenetet a sorban következő üzenetsornak.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-182">A component implemented as a filter task listens for a message on this queue, performs its work, and then posts the transformed message to the next queue in the sequence.</span></span> <span data-ttu-id="aa6dc-183">Egy másik szűrési feladat figyelheti ennek az üzenetsornak az üzeneteit, feldolgozhatja azokat, majd elküldheti az eredményeket egy másik üzenetsornak, és így tovább, amíg a teljes körűen átalakított adatok meg nem jelennek az üzenetsor végső üzenetében.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-183">Another filter task can listen for messages on this queue, process them, post the results to another queue, and so on until the fully transformed data appears in the final message in the queue.</span></span> <span data-ttu-id="aa6dc-184">A következő ábrán egy üzenetsorok használatával megvalósított folyamat látható.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-184">The next figure illustrates implementing a pipeline using message queues.</span></span>

![4. ábra – Üzenetsorok használatával megvalósított folyamat](./_images/pipes-and-filters-message-queues.png)


<span data-ttu-id="aa6dc-186">Ha a megoldást az Azure-ban hozza létre, a Service Bus-üzenetsorok megbízható és skálázható üzenetsor-kezelési mechanizmust biztosítanak.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-186">If you're building a solution on Azure you can use Service Bus queues to provide a reliable and scalable queuing mechanism.</span></span> <span data-ttu-id="aa6dc-187">Az alábbi, C# nyelven írt `ServiceBusPipeFilter` osztály példáján egy olyan szűrő implementálását mutatjuk be, amely a bemeneti üzeneteket egy üzenetsorból kapja, feldolgozza azokat, majd az eredményeket egy másik üzenetsorban teszi közzé.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-187">The `ServiceBusPipeFilter` class shown below in C# demonstrates how you can implement a filter that receives input messages from a queue, processes these messages, and posts the results to another queue.</span></span>

>  <span data-ttu-id="aa6dc-188">A `ServiceBusPipeFilter` osztály a PipesAndFilters.Shared projektben van meghatározva, amely a [GitHubról](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters) érhető el.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-188">The `ServiceBusPipeFilter` class is defined in the PipesAndFilters.Shared project available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>

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

<span data-ttu-id="aa6dc-189">A `ServiceBusPipeFilter` osztály `Start` metódusa be- és kimeneti üzenetsorpárhoz csatlakozik, a `Close` metódus pedig lecsatlakozik a bemeneti üzenetsorról.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-189">The `Start` method in the `ServiceBusPipeFilter` class connects to a pair of input and output queues, and the `Close` method disconnects from the input queue.</span></span> <span data-ttu-id="aa6dc-190">Az `OnPipeFilterMessageAsync` metódus végzi el az üzenetek tényleges feldolgozását, a metódus `asyncFilterTask` paramétere pedig meghatározza a végrehajtandó feldolgozást.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-190">The `OnPipeFilterMessageAsync` method performs the actual processing of messages, the `asyncFilterTask` parameter to this method specifies the processing to be performed.</span></span> <span data-ttu-id="aa6dc-191">Az `OnPipeFilterMessageAsync` metódus a bemeneti üzenetsor bejövő üzeneteire vár, azok beérkezési sorrendjében futtatja az `asyncFilterTask` paraméter által megadott kódot az üzeneteken, végül a kimeneti üzenetsoron teszi közzé az eredményeket.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-191">The `OnPipeFilterMessageAsync` method waits for incoming messages on the input queue, runs the code specified by the `asyncFilterTask` parameter over each message as it arrives, and posts the results to the output queue.</span></span> <span data-ttu-id="aa6dc-192">Magukat az üzenetsorokat a konstruktor adja meg.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-192">The queues themselves are specified by the constructor.</span></span>

<span data-ttu-id="aa6dc-193">A mintául szolgáló megoldás szűrőket implementál feldolgozói szerepkörökben.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-193">The sample solution implements filters in a set of worker roles.</span></span> <span data-ttu-id="aa6dc-194">Az egyes feldolgozói szerepkörök egymástól függetlenül skálázhatók az általuk végzett üzleti feldolgozás összetettségétől vagy a feldolgozáshoz szükséges erőforrásoktól függően.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-194">Each worker role can be scaled independently, depending on the complexity of the business processing that it performs or the resources required for processing.</span></span> <span data-ttu-id="aa6dc-195">Ezenkívül az egyes feldolgozói szerepkörök több példánya párhuzamosan is futtatható a teljesítmény növelése érdekében.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-195">Additionally, multiple instances of each worker role can be run in parallel to improve throughput.</span></span>

<span data-ttu-id="aa6dc-196">Az alábbi kód egy `PipeFilterARoleEntry` nevű Azure-beli feldolgozói szerepkörre mutat példát, amelynek meghatározása a mintamegoldás PipeFilterA projektjében történt.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-196">The following code shows an Azure worker role named `PipeFilterARoleEntry`, defined in the PipeFilterA project in the sample solution.</span></span>

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

<span data-ttu-id="aa6dc-197">Ez a szerepkör tartalmaz egy `ServiceBusPipeFilter` objektumot.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-197">This role contains a `ServiceBusPipeFilter` object.</span></span> <span data-ttu-id="aa6dc-198">A szerepkör `OnStart` metódusa a bemeneti üzenetek fogadásához és a kimeneti üzenetek közzétételéhez csatlakozik az üzenetsorokhoz (az üzenetsorok nevei a `Constants` osztályban vannak meghatározva).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-198">The `OnStart` method in the role connects to the queues for receiving input messages and posting output messages (the names of the queues are defined in the `Constants` class).</span></span> <span data-ttu-id="aa6dc-199">A `Run` metódus meghívja az `OnPipeFilterMessagesAsync` metódust a fogadott üzenetek bizonyos mértékű feldolgozására (ebben a példában a feldolgozást rövid idejű várakozással szimuláljuk).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-199">The `Run` method invokes the `OnPipeFilterMessagesAsync` method to perform some processing on each message that's received (in this example, the processing is simulated by waiting for a short period of time).</span></span> <span data-ttu-id="aa6dc-200">Amikor a feldolgozás befejeződött, a rendszer egy új üzenetet állít össze, amely tartalmazza az eredményeket (ebben az esetben a bemeneti üzenet egy hozzáadott egyéni tulajdonsággal rendelkezik) és a kimeneti üzenetsorban lesz közzétéve.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-200">When processing is complete, a new message is constructed containing the results (in this case, the input message has a custom property added), and this message is posted to the output queue.</span></span>

<span data-ttu-id="aa6dc-201">A mintakód egy másik, `PipeFilterBRoleEntry` nevű feldolgozói szerepkört is tartalmaz a PipeFilterB projektben.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-201">The sample code contains another worker role named `PipeFilterBRoleEntry` in the PipeFilterB project.</span></span> <span data-ttu-id="aa6dc-202">Ez a szerepkör hasonlít a `PipeFilterARoleEntry` szerepkörhöz azzal a különbséggel, hogy eltérő feldolgozási műveletet hajt végre a `Run` metódusban.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-202">This role is similar to `PipeFilterARoleEntry` except that it performs different processing in the `Run` method.</span></span> <span data-ttu-id="aa6dc-203">A példában szereplő megoldásban ennek a két szerepkörnek a kombinálásával hozunk létre egy folyamatot, ahol a `PipeFilterARoleEntry` szerepkör kimeneti üzenetsora lesz a `PipeFilterBRoleEntry` szerepkör bemeneti üzenetsora.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-203">In the example solution, these two roles are combined to construct a pipeline, the output queue for the `PipeFilterARoleEntry` role is the input queue for the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="aa6dc-204">A mintamegoldás két további szerepkört is biztosít: `InitialSenderRoleEntry` (az InitialSender projektben) és `FinalReceiverRoleEntry` (a FinalReceiver projektben).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-204">The sample solution also provides two additional roles named `InitialSenderRoleEntry` (in the InitialSender project) and `FinalReceiverRoleEntry` (in the FinalReceiver project).</span></span> <span data-ttu-id="aa6dc-205">A folyamat kezdő üzenetét az `InitialSenderRoleEntry` szerepkör biztosítja.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-205">The `InitialSenderRoleEntry` role provides the initial message in the pipeline.</span></span> <span data-ttu-id="aa6dc-206">Az `OnStart` metódus egyetlen üzenetsorhoz kapcsolódik, és ezen az üzenetsoron a `Run` metódus közzétesz egy metódust.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-206">The `OnStart` method connects to a single queue and the `Run` method posts a method to this queue.</span></span> <span data-ttu-id="aa6dc-207">Ezt az üzenetsort használja a `PipeFilterARoleEntry` szerepkör bemeneti üzenetsorként, így az itt közzétett üzenetek fogadását és feldolgozását a `PipeFilterARoleEntry` szerepkör végzi.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-207">This queue is the input queue used by the `PipeFilterARoleEntry` role, so posting a message to it causes the message to be received and processed by the `PipeFilterARoleEntry` role.</span></span> <span data-ttu-id="aa6dc-208">A feldolgozott üzenet ezt követően a `PipeFilterBRoleEntry` szerepkörön halad át.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-208">The processed message then passes through the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="aa6dc-209">A `FinalReceiveRoleEntry` szerepkör bemeneti üzenetsora a `PipeFilterBRoleEntry` szerepkör kimeneti üzenetsora.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-209">The input queue for the `FinalReceiveRoleEntry` role is the output queue for the `PipeFilterBRoleEntry` role.</span></span> <span data-ttu-id="aa6dc-210">Az üzenetet a `FinalReceiveRoleEntry` szerepkör `Run` metódusa fogadja (lásd alább), és hajtja végre rajta a végső feldolgozási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-210">The `Run` method in the `FinalReceiveRoleEntry` role, shown below, receives the message and performs some final processing.</span></span> <span data-ttu-id="aa6dc-211">Ezt követően a metódus a folyamat szűrői által hozzáadott egyéni tulajdonságokat a nyomkövetési kimenetbe írja.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-211">Then it writes the values of the custom properties added by the filters in the pipeline to the trace output.</span></span>

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

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="aa6dc-212">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="aa6dc-212">Related patterns and guidance</span></span>

<span data-ttu-id="aa6dc-213">Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:</span><span class="sxs-lookup"><span data-stu-id="aa6dc-213">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="aa6dc-214">A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters) talál egy, a minta bemutatására szolgáló példát.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-214">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>
- <span data-ttu-id="aa6dc-215">[Versengő felhasználókat ismertető minta](competing-consumers.md).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-215">[Competing Consumers pattern](competing-consumers.md).</span></span> <span data-ttu-id="aa6dc-216">A folyamatok adott szűrők több példányát is tartalmazhatják.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-216">A pipeline can contain multiple instances of one or more filters.</span></span> <span data-ttu-id="aa6dc-217">Ezen megközelítés lehetőséget biztosít a lassú szűrők példányainak egyidejű futtatására, lehetővé téve ezzel a rendszer számára a terhelés elosztását és a teljesítmény növelését.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-217">This approach is useful for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span> <span data-ttu-id="aa6dc-218">A szűrőpéldányok versenyezni fognak egymással a bemeneti adatokért, egy adott szűrő két példánya nem dolgozhatja fel ugyanazokat az adatokat.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-218">Each instance of a filter will compete for input with the other instances, two instances of a filter shouldn't be able to process the same data.</span></span> <span data-ttu-id="aa6dc-219">Megmagyarázza ezt a megközelítést.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-219">Provides an explanation of this approach.</span></span>
- <span data-ttu-id="aa6dc-220">[Számításierőforrás-konszolidálási minta](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-220">[Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="aa6dc-221">Lehetőség van csoportosítani olyan szűrőket, amelyeknek azonos skálázásúaknak kell lenniük egy adott folyamatban.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-221">It might be possible to group filters that should scale together into the same process.</span></span> <span data-ttu-id="aa6dc-222">További információkat biztosít ezen stratégia előnyeiről és hátrányairól.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-222">Provides more information about the benefits and tradeoffs of this strategy.</span></span>
- <span data-ttu-id="aa6dc-223">[Kompenzáló tranzakció mintája](compensating-transaction.md).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-223">[Compensating Transaction pattern](compensating-transaction.md).</span></span> <span data-ttu-id="aa6dc-224">A szűrők implementálhatóak visszavonható műveletként is, vagy olyan kompenzáló műveletként, amely hiba esetén visszaállít egy korábbi verzió szerinti állapotot.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-224">A filter can be implemented as an operation that can be reversed, or that has a compensating operation that restores the state to a previous version in the event of a failure.</span></span> <span data-ttu-id="aa6dc-225">Ismerteti, hogy ez a módszer hogyan implementálható a végső konzisztencia fenntartásához vagy eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="aa6dc-225">Explains how this can be implemented to maintain or achieve eventual consistency.</span></span>
- <span data-ttu-id="aa6dc-226">[Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) (Jonathan Oliver blogjában).</span><span class="sxs-lookup"><span data-stu-id="aa6dc-226">[Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>
