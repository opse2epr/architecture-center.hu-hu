---
title: Monolitikus adatmegőrzési kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: Egy alkalmazás összes adatának egyetlen adattárolóba való helyezése hátrányosan befolyásolhatja a teljesítményt.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: e3d6fbb789e422af22ce54b0defd6bb73099f15a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346331"
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="2706d-103">Monolitikus adatmegőrzési kizárási minta</span><span class="sxs-lookup"><span data-stu-id="2706d-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="2706d-104">Egy alkalmazás összes adatának egyetlen adattárolóba való helyezése hátrányosan befolyásolhatja a teljesítményt, mert erőforrás-versengéshez vezet, vagy mert az adattároló nem alkalmas az adatok egy részének tárolására.</span><span class="sxs-lookup"><span data-stu-id="2706d-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="2706d-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="2706d-105">Problem description</span></span>

<span data-ttu-id="2706d-106">Az alkalmazások korábban egyetlen adattárolót használtak, függetlenül attól, hogy az alkalmazásnak adott esetben különböző adattípusokat kellett tárolnia.</span><span class="sxs-lookup"><span data-stu-id="2706d-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="2706d-107">Ezzel általában az alkalmazás tervezését kívánták leegyszerűsíteni, vagy pedig ez a megoldás felelt meg a fejlesztői csapat ismereteinek.</span><span class="sxs-lookup"><span data-stu-id="2706d-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span>

<span data-ttu-id="2706d-108">A modern felhőalapú rendszerek további funkcionális és nem funkcionális követelményekkel rendelkeznek, és sok különféle adattípust kell tárolniuk, például dokumentumokat, képeket, gyorsítótárazott adatokat, üzenetsorokat, alkalmazásnaplókat és telemetriai adatokat.</span><span class="sxs-lookup"><span data-stu-id="2706d-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="2706d-109">A hagyományos megoldás követése, vagyis az összes fenti információ egyetlen adattárolóba való helyezése negatívan befolyásolja a teljesítményt. Ennek két fő oka van:</span><span class="sxs-lookup"><span data-stu-id="2706d-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="2706d-110">A nagy mennyiségű, egymáshoz nem kapcsolódó adatok egyetlen adattárolóban történő tárolása és lekérése versengést idéz elő, amely lassú válaszidőkhöz és csatlakozási hibákhoz vezet.</span><span class="sxs-lookup"><span data-stu-id="2706d-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="2706d-111">Egy adattároló sem biztosan megfelelő egy adott adattípus tárolásához, illetve előfordulhat, hogy nem az alkalmazás által végzett műveletekhez optimalizálták.</span><span class="sxs-lookup"><span data-stu-id="2706d-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span>

<span data-ttu-id="2706d-112">A következő példában egy ASP.NET webes API-vezérlő látható, amely új rekordot ad hozzá egy adatbázishoz, az eredményt pedig feljegyzi egy naplóban.</span><span class="sxs-lookup"><span data-stu-id="2706d-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="2706d-113">A rendszer ugyanabban az adatbázisban tárolja a naplót, mint az üzleti adatokat.</span><span class="sxs-lookup"><span data-stu-id="2706d-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="2706d-114">A teljes kódmintát [itt][sample-app] találja.</span><span class="sxs-lookup"><span data-stu-id="2706d-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="2706d-115">A naplórekordok létrehozásának sebessége valószínűleg kihat az üzleti műveletek teljesítményére.</span><span class="sxs-lookup"><span data-stu-id="2706d-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="2706d-116">Ha egy másik összetevő, például egy alkalmazásfolyamat-figyelő rendszeresen olvassa és feldolgozza a naplóadatokat, az szintén kihathat az üzleti tevékenységekre.</span><span class="sxs-lookup"><span data-stu-id="2706d-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="2706d-117">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="2706d-117">How to fix the problem</span></span>

<span data-ttu-id="2706d-118">Válassza szét az adatokat a felhasználásuk módja szerint.</span><span class="sxs-lookup"><span data-stu-id="2706d-118">Separate data according to its use.</span></span> <span data-ttu-id="2706d-119">Minden egyes adatkészlethez külön adattárolót válasszon ki, amely a legalkalmasabb az adott adat felhasználási módjához.</span><span class="sxs-lookup"><span data-stu-id="2706d-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="2706d-120">Az előző példában az alkalmazás naplóját el kell különíteni az üzleti adatokat tartalmazó adatbázistól:</span><span class="sxs-lookup"><span data-stu-id="2706d-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span>

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="2706d-121">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="2706d-121">Considerations</span></span>

- <span data-ttu-id="2706d-122">Válassza szét az adatokat a felhasználási és hozzáférési módjuk szerint.</span><span class="sxs-lookup"><span data-stu-id="2706d-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="2706d-123">Ne tároljon például naplózási információkat és üzleti adatokat ugyanabban az adattárban.</span><span class="sxs-lookup"><span data-stu-id="2706d-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="2706d-124">Ezekre az adatokra ugyanis különböző követelmények vonatkoznak, és a hozzáférési mintáik is eltérnek.</span><span class="sxs-lookup"><span data-stu-id="2706d-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="2706d-125">A naplórekordok természetüknél fogva szekvenciálisak, míg az üzleti adatok esetében valószínűleg többször van szükség véletlen sorrendű hozzáférésre, és gyakran relációsak.</span><span class="sxs-lookup"><span data-stu-id="2706d-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="2706d-126">Tartsa szem előtt az egyes adattípusok adathozzáférési mintáit.</span><span class="sxs-lookup"><span data-stu-id="2706d-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="2706d-127">A formázott jelentéseket és dokumentumokat például dokumentum-adatbázisban, például a [Cosmos DB][CosmosDB]-ben ajánlott tárolni, az ideiglenes adatok gyorsítótárazásához azonban az [Azure Redis Cache][Azure-cache] használata javasolt.</span><span class="sxs-lookup"><span data-stu-id="2706d-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="2706d-128">Ha ezt az útmutatót követve is eléri az adatbázis korlátait, valószínűleg vertikálisan fel kell skáláznia az adatbázist.</span><span class="sxs-lookup"><span data-stu-id="2706d-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="2706d-129">Érdemes megfontolnia a horizontális skálázást és a terhelés több adatbázis-kiszolgálón történő particionálását is.</span><span class="sxs-lookup"><span data-stu-id="2706d-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="2706d-130">A particionálás azonban szükségessé teheti az alkalmazás újratervezését.</span><span class="sxs-lookup"><span data-stu-id="2706d-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="2706d-131">További információkért lásd az [adatparticionálást][DataPartitioningGuidance] ismertető részt.</span><span class="sxs-lookup"><span data-stu-id="2706d-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="2706d-132">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="2706d-132">How to detect the problem</span></span>

<span data-ttu-id="2706d-133">A rendszer valószínűleg nagymértékben lelassul, végül hibával leáll, mert elfogynak az erőforrások, például az adatbázis-kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="2706d-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="2706d-134">A következő lépéseket végezheti el a hiba okának meghatározásához.</span><span class="sxs-lookup"><span data-stu-id="2706d-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="2706d-135">Alakítsa ki úgy a rendszert, hogy feljegyezze a fő teljesítménystatisztikákat.</span><span class="sxs-lookup"><span data-stu-id="2706d-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="2706d-136">Rögzítse az egyes műveletek időzítési információit, valamint azokat a pontokat, ahol az alkalmazás adatokat olvas be ír.</span><span class="sxs-lookup"><span data-stu-id="2706d-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
2. <span data-ttu-id="2706d-137">Ha lehetséges, monitorozza a rendszer futását néhány napig éles környezetben, így valós információkat szerezhet a rendszer használatának módjáról.</span><span class="sxs-lookup"><span data-stu-id="2706d-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="2706d-138">Ha ez nem lehetséges, futtasson szkriptelt terhelési teszteket, amelyek a valóságnak megfelelő számú felhasználó által végzett jellemző művelettípusokat tartalmazzák.</span><span class="sxs-lookup"><span data-stu-id="2706d-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
3. <span data-ttu-id="2706d-139">Használja a telemetriaadatokat a gyenge teljesítmény időszakainak meghatározásához.</span><span class="sxs-lookup"><span data-stu-id="2706d-139">Use the telemetry data to identify periods of poor performance.</span></span>
4. <span data-ttu-id="2706d-140">Állapítsa meg, melyik adattárolók voltak használatban az érintett időszakokban.</span><span class="sxs-lookup"><span data-stu-id="2706d-140">Identify which data stores were accessed during those periods.</span></span>
5. <span data-ttu-id="2706d-141">Azonosítsa az adattároló-erőforrásokat, amelyek esetleg versengésre kényszerülnek.</span><span class="sxs-lookup"><span data-stu-id="2706d-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="2706d-142">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="2706d-142">Example diagnosis</span></span>

<span data-ttu-id="2706d-143">Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.</span><span class="sxs-lookup"><span data-stu-id="2706d-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="2706d-144">A rendszer kialakítása és monitorozása</span><span class="sxs-lookup"><span data-stu-id="2706d-144">Instrument and monitor the system</span></span>

<span data-ttu-id="2706d-145">A következő diagram a korábban ismertetett mintaalkalmazás terhelési tesztjének eredményeit mutatja be.</span><span class="sxs-lookup"><span data-stu-id="2706d-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="2706d-146">Ez a teszt lépéses terhelést használt legfeljebb 1000 egyidejű felhasználóval.</span><span class="sxs-lookup"><span data-stu-id="2706d-146">The test used a step load of up to 1000 concurrent users.</span></span>

![Az SQL-alapú vezérlő terhelésiteszt-teljesítményének eredményei][MonolithicScenarioLoadTest]

<span data-ttu-id="2706d-148">Ahogy a terhelés 700 felhasználóra emelkedik, úgy nő az átviteli teljesítmény is.</span><span class="sxs-lookup"><span data-stu-id="2706d-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="2706d-149">Az átviteli teljesítmény növekedése itt megáll, a rendszer pedig a jelek szerint maximális kapacitással működik.</span><span class="sxs-lookup"><span data-stu-id="2706d-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="2706d-150">Az átlagos válaszidő fokozatosan nő a felhasználói terheléssel együtt, ami jelzi, hogy a rendszer nem tud lépést tartani az igényekkel.</span><span class="sxs-lookup"><span data-stu-id="2706d-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="2706d-151">A gyenge teljesítmény időszakainak meghatározása</span><span class="sxs-lookup"><span data-stu-id="2706d-151">Identify periods of poor performance</span></span>

<span data-ttu-id="2706d-152">Ha az üzemi környezetet monitorozza, felfigyelhet bizonyos mintákra.</span><span class="sxs-lookup"><span data-stu-id="2706d-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="2706d-153">Előfordulhat például, hogy a válaszidők jelentősen nőnek minden nap ugyanabban az időben.</span><span class="sxs-lookup"><span data-stu-id="2706d-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="2706d-154">Ezt okozhatja rendszeres számítási feladat vagy ütemezett kötegelt folyamat, vagy hogy bizonyos időszakokban egyszerűen több felhasználó csatlakozik a rendszerhez.</span><span class="sxs-lookup"><span data-stu-id="2706d-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="2706d-155">Ilyen esetben tekintse át a telemetriai adatokat.</span><span class="sxs-lookup"><span data-stu-id="2706d-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="2706d-156">Keressen összefüggéseket a nagyobb válaszidők, illetve a nagyobb mértékű adatbázis-tevékenység vagy a közös erőforrásokra irányuló I/O között.</span><span class="sxs-lookup"><span data-stu-id="2706d-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="2706d-157">Ha talál összefüggéseket, lehetséges, hogy az adatbázis a szűk keresztmetszet.</span><span class="sxs-lookup"><span data-stu-id="2706d-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="2706d-158">Az érintett időszakokban elért adattárak azonosítása</span><span class="sxs-lookup"><span data-stu-id="2706d-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="2706d-159">A következő diagramon az adatbázis átviteli egységeinek (DTU) terhelési teszt közbeni felhasználása látható.</span><span class="sxs-lookup"><span data-stu-id="2706d-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="2706d-160">(A DTU az elérhető kapacitás mértékegysége, amelyet a processzorfelhasználás, a lefoglalt memória és az I/O-sebesség kombinációja alkot.) A DTU-k felhasználása gyorsan elérte a 100%-ot.</span><span class="sxs-lookup"><span data-stu-id="2706d-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="2706d-161">Ez nagyjából az a pont, ahol az átviteli teljesítmény az előző diagramon elérte a maximumot.</span><span class="sxs-lookup"><span data-stu-id="2706d-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="2706d-162">Az adatbázis kihasználtsága a teszt befejezéséig nagyon magas maradt.</span><span class="sxs-lookup"><span data-stu-id="2706d-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="2706d-163">A vége felé enyhe csökkenés látható, amelyet a szabályozás, az adatbázis-kapcsolatokért folytatott versengés vagy egyéb tényezők okozhattak.</span><span class="sxs-lookup"><span data-stu-id="2706d-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![A klasszikus Azure portál adatbázis-figyelője az adatbázis erőforrás-felhasználásával][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="2706d-165">Az adattárolók telemetriai vizsgálata</span><span class="sxs-lookup"><span data-stu-id="2706d-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="2706d-166">Alakítsa ki az adattárolókat úgy, hogy részletesen rögzítsék a tevékenységeket.</span><span class="sxs-lookup"><span data-stu-id="2706d-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="2706d-167">A mintaalkalmazásban az adathozzáférési statisztikák nagy számú beszúrási műveleteket mutattak a `PurchaseOrderHeader` és a `MonoLog` táblában is.</span><span class="sxs-lookup"><span data-stu-id="2706d-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span>

![A mintaalkalmazáshoz tartozó adathozzáférési statisztika][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="2706d-169">Az erőforrás-versengés azonosítása</span><span class="sxs-lookup"><span data-stu-id="2706d-169">Identify resource contention</span></span>

<span data-ttu-id="2706d-170">Megkeresheti a forráskódban azokat a pontokat, ahol az alkalmazás olyan erőforrásokhoz fér hozzá, amelyekért versengés folyik.</span><span class="sxs-lookup"><span data-stu-id="2706d-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="2706d-171">Keressen a következőkhöz hasonló helyzeteket:</span><span class="sxs-lookup"><span data-stu-id="2706d-171">Look for situations such as:</span></span>

- <span data-ttu-id="2706d-172">Logikailag különálló adat, amelyet a rendszer ugyanazon tárolóba ír.</span><span class="sxs-lookup"><span data-stu-id="2706d-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="2706d-173">A naplók, jelentések, üzenetsorok és egyéb hasonló adatokat nem ajánlott ugyanabban az adatbázisban tartani, mint az üzleti információkat.</span><span class="sxs-lookup"><span data-stu-id="2706d-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="2706d-174">Eltérés az adattároló típusa és az adat típusa között, például ha nagyméretű blobok vagy XML-dokumentumok találhatók egy relációs adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="2706d-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="2706d-175">Lényegesen eltérő felhasználási mintájú adatok vannak egyazon tárolóban, például magas írású/alacsony olvasású és alacsony írású/magas olvasású adatok.</span><span class="sxs-lookup"><span data-stu-id="2706d-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="2706d-176">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="2706d-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="2706d-177">Az alkalmazás módosítva lett, hogy külön adattárolóba írja a naplókat.</span><span class="sxs-lookup"><span data-stu-id="2706d-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="2706d-178">Itt láthatók a terhelési teszt eredményei:</span><span class="sxs-lookup"><span data-stu-id="2706d-178">Here are the load test results:</span></span>

![Terhelésiteszt-teljesítmény eredmények a Polyglot vezérlő használatával][PolyglotScenarioLoadTest]

<span data-ttu-id="2706d-180">Az átviteli teljesítmény mintája hasonló az előző diagramon látotthoz, de a maximális teljesítmény körülbelül 500 kérés/másodperccel magasabb terhelésnél következik be.</span><span class="sxs-lookup"><span data-stu-id="2706d-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="2706d-181">Az átlagos válaszidő valamivel alacsonyabb.</span><span class="sxs-lookup"><span data-stu-id="2706d-181">The average response time is marginally lower.</span></span> <span data-ttu-id="2706d-182">Azonban ezekből a statisztikákból nem derül ki minden.</span><span class="sxs-lookup"><span data-stu-id="2706d-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="2706d-183">Az üzleti adatbázis telemetriája azt mutatja, hogy a DTU-felhasználás nem 100, hanem 75%-nál éri el a maximumot.</span><span class="sxs-lookup"><span data-stu-id="2706d-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![A klasszikus Azure portál adatbázis-figyelője az adatbázis erőforrás-felhasználásával a Polyglot-forgatókönyvben][PolyglotDatabaseUtilization]

<span data-ttu-id="2706d-185">Hasonló módon a naplóadatbázis maximális DTU-kihasználtsága csak 70% körüli értéket ér el.</span><span class="sxs-lookup"><span data-stu-id="2706d-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="2706d-186">Az adatbázisok már nem korlátozzák a rendszer teljesítményét.</span><span class="sxs-lookup"><span data-stu-id="2706d-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![A klasszikus Azure portál adatbázis-figyelője a naplóadatbázis erőforrás-felhasználásával a Polyglot-forgatókönyvben][LogDatabaseUtilization]

## <a name="related-resources"></a><span data-ttu-id="2706d-188">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="2706d-188">Related resources</span></span>

- <span data-ttu-id="2706d-189">[A megfelelő adattároló kiválasztása][data-store-overview]</span><span class="sxs-lookup"><span data-stu-id="2706d-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="2706d-190">[Az adattár kiválasztásának kritériumai][data-store-comparison]</span><span class="sxs-lookup"><span data-stu-id="2706d-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="2706d-191">[Adathozzáférés nagymértékben skálázható megoldások esetén: az SQL, a NoSQL és a többplatformos adattárolás használata][Data-Access-Guide]</span><span class="sxs-lookup"><span data-stu-id="2706d-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="2706d-192">[Adatparticionálás][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="2706d-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: https://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
