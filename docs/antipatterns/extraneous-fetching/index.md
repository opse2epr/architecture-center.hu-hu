---
title: Felesleges beolvasások kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: Ha egyes üzleti műveletekhez a szükségesnél több adatot kér le, az szükségtelen I/O túlműködést eredményez, és csökkenti a válaszképességet.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: dac1b4c1422b447b8a0a9ebe317d5ac246c38da5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010239"
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="275fd-103">Felesleges beolvasások kizárási minta</span><span class="sxs-lookup"><span data-stu-id="275fd-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="275fd-104">Ha egyes üzleti műveletekhez a szükségesnél több adatot kér le, az szükségtelen I/O túlműködést eredményez, és csökkenti a válaszképességet.</span><span class="sxs-lookup"><span data-stu-id="275fd-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="275fd-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="275fd-105">Problem description</span></span>

<span data-ttu-id="275fd-106">Ez a kizárási minta akkor jelentkezik, ha az alkalmazás az I/O-kérések mennyiségének csökkentése érdekében lekéri az összes adatot, amelyre *esetleg* szüksége lehet.</span><span class="sxs-lookup"><span data-stu-id="275fd-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="275fd-107">Ez gyakran a [Forgalmas I/O][chatty-io] kizárási minta ellensúlyozásának eredményeképpen jelentkezik.</span><span class="sxs-lookup"><span data-stu-id="275fd-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="275fd-108">Előfordulhat például. hogy az alkalmazás lekéri az adatbázisban található összes termék adatait.</span><span class="sxs-lookup"><span data-stu-id="275fd-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="275fd-109">A felhasználónak azonban csak az adatok egy részhalmazára van szüksége (a többi nem is feltétlenül érinti a felhasználót), és valószínűleg nem is szeretné az *összes* terméket egyszerre megtekinteni.</span><span class="sxs-lookup"><span data-stu-id="275fd-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="275fd-110">Még ha a felhasználó a teljes katalógusban böngész is, észszerű az eredményeket külön lapokra osztani, például egyszerre csak 20 találat megjelenítésével.</span><span class="sxs-lookup"><span data-stu-id="275fd-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="275fd-111">A probléma másik oka a nem megfelelő programozási és kialakítási gyakorlatokban keresendő.</span><span class="sxs-lookup"><span data-stu-id="275fd-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="275fd-112">A következő kód például az Entity Framework használatával lekéri az összes termék minden adatát.</span><span class="sxs-lookup"><span data-stu-id="275fd-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="275fd-113">Ezután szűri az adatokat, és csak a mezők egy részét adja vissza, a többit pedig elveti.</span><span class="sxs-lookup"><span data-stu-id="275fd-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="275fd-114">A teljes kódmintát [itt][sample-app] találja.</span><span class="sxs-lookup"><span data-stu-id="275fd-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="275fd-115">A következő példában az alkalmazás egy olyan összesítéshez kér le adatokat, amelyet az adatbázis is elvégezhetett volna.</span><span class="sxs-lookup"><span data-stu-id="275fd-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="275fd-116">Az alkalmazás az értékesítések végösszegének kiszámításához lekéri az összes értékesített megrendeléshez tartozó minden bejegyzést, majd ezekből számítja ki a végösszeget.</span><span class="sxs-lookup"><span data-stu-id="275fd-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="275fd-117">A teljes kódmintát [itt][sample-app] találja.</span><span class="sxs-lookup"><span data-stu-id="275fd-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="275fd-118">A következő példa egy kisebb hibát mutat be, amely abból származik, ahogy az Entity Framework a LINQ to Entities műveletet alkalmazza.</span><span class="sxs-lookup"><span data-stu-id="275fd-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span>

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="275fd-119">Az alkalmazás az egy hétnél régebbi `SellStartDate` dátummal rendelkező termékeket keresi.</span><span class="sxs-lookup"><span data-stu-id="275fd-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="275fd-120">A legtöbb esetben a LINQ to Entities a `where` záradékokat az adatbázis által végrehajtandó SQL-utasításokká fordítaná le.</span><span class="sxs-lookup"><span data-stu-id="275fd-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="275fd-121">Ebben az esetben azonban a LINQ to Entities nem tudja az `AddDays` metódust SQL-re leképezni.</span><span class="sxs-lookup"><span data-stu-id="275fd-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="275fd-122">Ehelyett a `Product` tábla minden egyes sorát visszaadja, és az eredmények szűrését a memóriában végzi el.</span><span class="sxs-lookup"><span data-stu-id="275fd-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span>

<span data-ttu-id="275fd-123">Az `AsEnumerable` meghívása utal arra, hogy itt probléma van.</span><span class="sxs-lookup"><span data-stu-id="275fd-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="275fd-124">Ez a metódus az eredményeket egy `IEnumerable` felületté konvertálja.</span><span class="sxs-lookup"><span data-stu-id="275fd-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="275fd-125">Bár az `IEnumerable` támogatja a szűrést, arra az *ügyféloldalon*, és nem az adatbázisban kerül sor.</span><span class="sxs-lookup"><span data-stu-id="275fd-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="275fd-126">A LINQ to Entities alapértelmezés szerint az `IQueryable` függvényt használja, amely a szűrési feladat felelősségét átadja az adatforrásnak.</span><span class="sxs-lookup"><span data-stu-id="275fd-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="275fd-127">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="275fd-127">How to fix the problem</span></span>

<span data-ttu-id="275fd-128">Kerülje az olyan nagyobb adatmennyiségek lekérését, amelyek elavulhatnak vagy amelyek el lesznek vetve, és csak az épp végrehajtott művelethez szükséges adatokat hívja le.</span><span class="sxs-lookup"><span data-stu-id="275fd-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span>

<span data-ttu-id="275fd-129">Ahelyett, hogy valamely tábla összes oszlopát lekérné, majd szűrést futtatna rajtuk, válassza ki a kívánt oszlopokat az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="275fd-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="275fd-130">Ugyanígy az összesítéseket is az adatbázisban, és ne az alkalmazás memóriájában hajtsa végre.</span><span class="sxs-lookup"><span data-stu-id="275fd-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="275fd-131">Az Entity Framework használatakor bizonyosodjon meg róla, hogy a LINQ-lekérdezéseket az `IQueryable` felület és nem az `IEnumerable` hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="275fd-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="275fd-132">Előfordulhat, hogy módosítania kell a lekérdezést, hogy csak olyan függvényeket használjon, amelyek az adatforrásra vonatkoztathatók.</span><span class="sxs-lookup"><span data-stu-id="275fd-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="275fd-133">A korábbi példa átdolgozható: ha az `AddDays` metódust eltávolítja a lekérdezésből, a szűrést az adatbázisban lehet végrehajtani.</span><span class="sxs-lookup"><span data-stu-id="275fd-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out.
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="275fd-134">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="275fd-134">Considerations</span></span>

- <span data-ttu-id="275fd-135">Egyes esetekben a teljesítmény az adatok vízszintes particionálásával növelhető.</span><span class="sxs-lookup"><span data-stu-id="275fd-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="275fd-136">Ha különböző műveletek az adatok különböző tulajdonságait használják, a vízszintes particionálás csökkentheti a versengést.</span><span class="sxs-lookup"><span data-stu-id="275fd-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="275fd-137">A műveletek nagy része csak az adatok kisebb részhalmazát érinti, így ennek a terhelésnek az elosztásával fokozható a teljesítmény.</span><span class="sxs-lookup"><span data-stu-id="275fd-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="275fd-138">Lásd: [Adatparticionálás][data-partitioning].</span><span class="sxs-lookup"><span data-stu-id="275fd-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="275fd-139">A korlátlan lekérdezéseket támogató műveletnél alkalmazzon tördelést, és egyszerre csak korlátozott számú kérjen le.</span><span class="sxs-lookup"><span data-stu-id="275fd-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="275fd-140">Ha például az ügyfél egy termékkatalógust böngész, egyszerre elég egy oldalnyi találatot megjeleníteni.</span><span class="sxs-lookup"><span data-stu-id="275fd-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="275fd-141">Amikor csak lehetséges, használja az adatbázis beépített szolgáltatásait.</span><span class="sxs-lookup"><span data-stu-id="275fd-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="275fd-142">Az SQL-adatbázisok például általában kínálnak aggregátumfüggvényeket.</span><span class="sxs-lookup"><span data-stu-id="275fd-142">For example, SQL databases typically provide aggregate functions.</span></span>

- <span data-ttu-id="275fd-143">Ha olyan adattárat használ, amely nem támogat egy adott függvényt, például az összesítést, a számított eredményeket máshol tárolhatja, és a rekordok hozzáadásakor vagy eltávolításakor frissítheti az értéket, így az alkalmazásnak nem kell azt minden egyes alkalommal újraszámolnia, amikor szükség van rá.</span><span class="sxs-lookup"><span data-stu-id="275fd-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="275fd-144">Ha azt látja, hogy a kérések nagy számú mezőt hívnak le, vizsgálja meg a forráskódot, és állapítsa meg, hogy valóban mindegyik mező szükséges-e.</span><span class="sxs-lookup"><span data-stu-id="275fd-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="275fd-145">Néha az ilyen kérések rosszul összeállított `SELECT *` lekérdezések eredményei.</span><span class="sxs-lookup"><span data-stu-id="275fd-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span>

- <span data-ttu-id="275fd-146">Ugyanígy a nagy mennyiségű entitást lekérő kérések is utalhatnak arra, hogy az alkalmazás nem megfelelően szűri az adatokat.</span><span class="sxs-lookup"><span data-stu-id="275fd-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="275fd-147">Ellenőrizze, hogy az összes ilyen entitás valóban szükséges-e.</span><span class="sxs-lookup"><span data-stu-id="275fd-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="275fd-148">Amennyiben lehetséges, alkalmazzon adatbázisoldali szűrést, például az SQL `WHERE` záradékai használatával.</span><span class="sxs-lookup"><span data-stu-id="275fd-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span>

- <span data-ttu-id="275fd-149">A feldolgozás kiszervezése az adatbázisba nem minden esetben a legjobb lehetőség.</span><span class="sxs-lookup"><span data-stu-id="275fd-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="275fd-150">Csak akkor alkalmazza ezt a stratégiát, ha az adatbázis erre van kialakítva vagy optimalizálva.</span><span class="sxs-lookup"><span data-stu-id="275fd-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="275fd-151">A legtöbb adatbázisrendszer rendkívüli mértékben optimalizálva van adott függvényekre, azonban általános célú alkalmazásmotorként nem alkalmas.</span><span class="sxs-lookup"><span data-stu-id="275fd-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="275fd-152">További információkért lásd: [Foglalt adatbázis kizárási minta][BusyDatabase].</span><span class="sxs-lookup"><span data-stu-id="275fd-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="275fd-153">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="275fd-153">How to detect the problem</span></span>

<span data-ttu-id="275fd-154">A feladathoz nem kapcsolódó beolvasások tünetei a magas késleltetés és az alacsony átviteli sebesség.</span><span class="sxs-lookup"><span data-stu-id="275fd-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="275fd-155">Ha az adatok lekérésének forrása egy adattár, megnövekedett mértékű versengéssel is számolni kell.</span><span class="sxs-lookup"><span data-stu-id="275fd-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="275fd-156">A végfelhasználók várhatóan magas válaszidőket vagy a szolgáltatások időtúllépéséből eredő hibákat fognak jelezni. Ezek a hibák HTTP 500-as (belső kiszolgáló) vagy HTTP 503-as (a szolgáltatás nem érhető el) hibaüzeneteket eredményezhetnek.</span><span class="sxs-lookup"><span data-stu-id="275fd-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="275fd-157">Vizsgálja át a webkiszolgáló eseménynaplóit, amelyek valószínűleg részletesebb információkat tartalmaznak a hibák okairól és körülményeiről.</span><span class="sxs-lookup"><span data-stu-id="275fd-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="275fd-158">A kizárási minta tünetei és a gyűjtött telemetriaadatok nagyon hasonlóak lehetnek a [Monolitikus adatmegőrzés kizárási mintáéihoz][MonolithicPersistence].</span><span class="sxs-lookup"><span data-stu-id="275fd-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span>

<span data-ttu-id="275fd-159">A következő lépéseket végezheti el a hiba okának meghatározásához:</span><span class="sxs-lookup"><span data-stu-id="275fd-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="275fd-160">Azonosítsa a lassabb számítási feladatokat vagy tranzakciókat terheléstesztek, folyamatmonitorozás és a rendszerállapot-adatok rögzítésére szolgáló egyéb módszerek használatával.</span><span class="sxs-lookup"><span data-stu-id="275fd-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="275fd-161">Figyelje meg a rendszer viselkedési mintáit.</span><span class="sxs-lookup"><span data-stu-id="275fd-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="275fd-162">Jelentkeznek bizonyos korlátok a tranzakciók másodpercenkénti számában vagy a felhasználók mennyiségében?</span><span class="sxs-lookup"><span data-stu-id="275fd-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="275fd-163">Vesse össze a lassú számítási feladatok példányait a viselkedési mintákkal.</span><span class="sxs-lookup"><span data-stu-id="275fd-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="275fd-164">Azonosítsa a használt adattárakat.</span><span class="sxs-lookup"><span data-stu-id="275fd-164">Identify the data stores being used.</span></span> <span data-ttu-id="275fd-165">Mindegyik adatforrás esetében futtasson alacsonyabb szinten is telemetriavizsgálatot a műveletek viselkedésének megfigyelésére.</span><span class="sxs-lookup"><span data-stu-id="275fd-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
5. <span data-ttu-id="275fd-166">Azonosítsa az ezekre az adatforrásokra hivatkozó lassú lefutású lekérdezéseket.</span><span class="sxs-lookup"><span data-stu-id="275fd-166">Identify any slow-running queries that reference these data sources.</span></span>
6. <span data-ttu-id="275fd-167">Végezze el a lassú lefutású lekérdezések erőforrás-specifikus elemzését, és ismerje meg az adatok használatának és felhasználásának módját.</span><span class="sxs-lookup"><span data-stu-id="275fd-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="275fd-168">Az alábbi tüneteket figyelje:</span><span class="sxs-lookup"><span data-stu-id="275fd-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="275fd-169">Gyakori, nagyméretű I/O kérések, amelyek ugyanarra az erőforrásra vagy adattárra irányulnak.</span><span class="sxs-lookup"><span data-stu-id="275fd-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="275fd-170">Versengés egy megosztott erőforrásban vagy adattárban.</span><span class="sxs-lookup"><span data-stu-id="275fd-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="275fd-171">Valamely művelet gyakran fogad nagy mennyiségű adatot a hálózaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="275fd-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="275fd-172">Az alkalmazások és szolgáltatások jelentős időt töltenek azzal, hogy az I/O befejeztére várnak.</span><span class="sxs-lookup"><span data-stu-id="275fd-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="275fd-173">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="275fd-173">Example diagnosis</span></span>

<span data-ttu-id="275fd-174">Az alábbi szakaszokban ezeket a lépéseket az előzőleg ismertetett példákon hajtjuk végre.</span><span class="sxs-lookup"><span data-stu-id="275fd-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="275fd-175">A lassú számítási feladatok azonosítása</span><span class="sxs-lookup"><span data-stu-id="275fd-175">Identify slow workloads</span></span>

<span data-ttu-id="275fd-176">A diagram egy, a korábban bemutatott `GetAllFieldsAsync` metódust futtató, akár 400 egyidejű felhasználót szimuláló terhelésteszt teljesítményeredményeit mutatja.</span><span class="sxs-lookup"><span data-stu-id="275fd-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="275fd-177">A terhelés növekedtével a teljesítmény lassan csökken.</span><span class="sxs-lookup"><span data-stu-id="275fd-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="275fd-178">Az átlagos válaszidő a számítási feladatok növekedtével szintén növekszik.</span><span class="sxs-lookup"><span data-stu-id="275fd-178">Average response time goes up as the workload increases.</span></span>

![A GetAllFieldsAsync metódus terheléstesztjének eredményei][Load-Test-Results-Client-Side1]

<span data-ttu-id="275fd-180">Az `AggregateOnClientAsync` művelet terheléstesztje hasonló eredményeket mutat.</span><span class="sxs-lookup"><span data-stu-id="275fd-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="275fd-181">A kérések mennyisége viszonylag stabil.</span><span class="sxs-lookup"><span data-stu-id="275fd-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="275fd-182">A számítási feladatok növekedtével az átlagos válaszidő is nő, de lassabb tempóban, mint az előző ábrán.</span><span class="sxs-lookup"><span data-stu-id="275fd-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![Az AggregateOnClientAsync metódus terheléstesztjének eredményei][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="275fd-184">A számítási feladatok összevetése a viselkedési mintákkal</span><span class="sxs-lookup"><span data-stu-id="275fd-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="275fd-185">A magas kihasználtságú időszakok és a teljesítménylassulás rendszeres egybeesése problémás területeket jelezhet.</span><span class="sxs-lookup"><span data-stu-id="275fd-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="275fd-186">Vizsgálja meg alaposan a gyaníthatóan lassan futó funkciók teljesítményprofilját, és állapítsa meg, hogy egyezik-e a korábban végrehajtott terhelésteszt eredményeivel.</span><span class="sxs-lookup"><span data-stu-id="275fd-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="275fd-187">Végezze el ugyanezeknek a funkcióknak a terheléstesztjét lépésközönként növelt felhasználóterhelésekkel, és keresse meg azt a pontot, ahol a teljesítmény jelentős mértékben csökken vagy teljesen meg is szűnik.</span><span class="sxs-lookup"><span data-stu-id="275fd-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="275fd-188">Ha ez a pont a várható valós használati esetek határain belül esik, vizsgálja meg a funkciók megvalósítását.</span><span class="sxs-lookup"><span data-stu-id="275fd-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="275fd-189">Egy lassú művelet nem feltétlenül jelent problémát, ha nem a rendszer terhelt állapotában fut le, ha nem időfüggő, és ha nem befolyásolja negatívan más fontos műveletek teljesítményét.</span><span class="sxs-lookup"><span data-stu-id="275fd-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="275fd-190">Például a havi működési statisztikák elkészítése hosszú lefutású művelet, azonban valószínűleg végrehajtható kötegelt műveletként, és futtatható alacsony prioritású feladatként.</span><span class="sxs-lookup"><span data-stu-id="275fd-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="275fd-191">Másfelől a termékkatalógus ügyfelek általi böngészése üzleti szempontból kritikus művelet.</span><span class="sxs-lookup"><span data-stu-id="275fd-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="275fd-192">Összpontosítson az ilyen kritikus műveletek eredményezte telemetriaadatokra, mivel ezekből látható, hogy a teljesítmény hogyan alakul a magas kihasználtságú időszakokban.</span><span class="sxs-lookup"><span data-stu-id="275fd-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="275fd-193">A lassú számítási feladatok adatforrásainak azonosítása</span><span class="sxs-lookup"><span data-stu-id="275fd-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="275fd-194">Ha azt gyanítja, hogy egy szolgáltatás az adatokat lehívásának módja miatt teljesít gyengén, vizsgálja meg, hogy az alkalmazás hogyan kapcsolódik az általa használt adattárakhoz.</span><span class="sxs-lookup"><span data-stu-id="275fd-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="275fd-195">Az éles rendszer monitorozásával vizsgálja meg, hogy az alkalmazás melyik forrásokhoz fér hozzá azokban az időszakokban, amikor romlik a teljesítmény.</span><span class="sxs-lookup"><span data-stu-id="275fd-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span>

<span data-ttu-id="275fd-196">Mindegyik adatforrás esetében állítsa be úgy a rendszert, hogy a következőket rögzítse:</span><span class="sxs-lookup"><span data-stu-id="275fd-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="275fd-197">Az egyes adattárak hozzáférésének gyakoriságát.</span><span class="sxs-lookup"><span data-stu-id="275fd-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="275fd-198">Az adattárba be- és onnan kilépő adatok mennyiségét.</span><span class="sxs-lookup"><span data-stu-id="275fd-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="275fd-199">A műveletek időzítését, különös tekintettel a kérések késleltetésére.</span><span class="sxs-lookup"><span data-stu-id="275fd-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="275fd-200">Az egyes adatforrások tipikus terhelés melletti hozzáférése esetén jelentkező hibák természetét és arányát.</span><span class="sxs-lookup"><span data-stu-id="275fd-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="275fd-201">Vesse össze ezeket az információkat az alkalmazás által az ügyfélnek visszaadott adatok mennyiségével.</span><span class="sxs-lookup"><span data-stu-id="275fd-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="275fd-202">Vesse össze az adattár által visszaadott adatok mennyiségi arányát az ügyfélnek visszaadott adatok mennyiségével.</span><span class="sxs-lookup"><span data-stu-id="275fd-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="275fd-203">Amennyiben nagy eltérés tapasztalható, vizsgálja meg a alkalmazást, és állapítsa meg, hogy hív-e le olyan adatokat, amelyekre nincs szüksége.</span><span class="sxs-lookup"><span data-stu-id="275fd-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="275fd-204">Az adatok az élő rendszer megfigyelésével, az egyes felhasználói kérések életciklusának nyomon követésével rögzíthetőek, vagy lemodellezheti szintetikus számítási feladatok egy sorozatát, amelyeket lefuttathat egy tesztrendszeren.</span><span class="sxs-lookup"><span data-stu-id="275fd-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="275fd-205">A következő diagramok a [New Relic APM][new-relic] használatával a `GetAllFieldsAsync` metódus terheléstesztje során rögzített telemetriaadatokat mutatják.</span><span class="sxs-lookup"><span data-stu-id="275fd-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="275fd-206">Figyelje meg az eltérést az adatbázisról fogadott adatok és a kapcsolódó HTTP-válaszok mennyiségei között.</span><span class="sxs-lookup"><span data-stu-id="275fd-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![A GetAllFieldsAsync metódus telemetriája][TelemetryAllFields]

<span data-ttu-id="275fd-208">Az adatbázis minden kérésnél 80 503 bájt adatot adott vissza, az ügyfélnek küldött válasz azonban csak 19 855 bájt adatot tartalmazott, ami az adatbázis válaszának hozzávetőleg 25%-a.</span><span class="sxs-lookup"><span data-stu-id="275fd-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="275fd-209">Az ügyfélnek visszaadott adatok mérete a formátumtól függően változhat.</span><span class="sxs-lookup"><span data-stu-id="275fd-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="275fd-210">Ebben a terheléstesztben az ügyfél JSON-adatokat kért.</span><span class="sxs-lookup"><span data-stu-id="275fd-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="275fd-211">Egy másik, az XML formátum használatával végzett teszt során (nem látható) a válasz mérete 35 655 bájt volt, vagyis az adatbázis válaszának 44%-a.</span><span class="sxs-lookup"><span data-stu-id="275fd-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="275fd-212">Az `AggregateOnClientAsync` metódus terheléstesztje még szélsőségesebb eredményeket mutat.</span><span class="sxs-lookup"><span data-stu-id="275fd-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="275fd-213">Ebben az esetben mindegyik teszt egy lekérdezést hajtott végre, amely több mint 280 kB adatot kért le az adatbázisból, a JSON-válasz mérete azonban mindössze 14 bájt volt.</span><span class="sxs-lookup"><span data-stu-id="275fd-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="275fd-214">A jelentős eltérés oka, hogy a metódus egy összesített eredményt számít ki nagy mennyiségű adatból.</span><span class="sxs-lookup"><span data-stu-id="275fd-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![Az AggregateOnClientAsync metódus telemetriája][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="275fd-216">A lassú lekérdezések azonosítása és elemzése</span><span class="sxs-lookup"><span data-stu-id="275fd-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="275fd-217">Keresse meg azokat az adatbázis-lekérdezéseket, amelyek a legtöbb erőforrást emésztik fel, és a végrehajtásuk a legtöbb időt veszi igénybe.</span><span class="sxs-lookup"><span data-stu-id="275fd-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="275fd-218">Megadhat olyan rendszerállapotokat, amelyek lehetővé teszik számos adatbázis-művelet kezdési és befejezési időpontjának kimutatását.</span><span class="sxs-lookup"><span data-stu-id="275fd-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="275fd-219">Sok adattár emellett a lekérdezések végrehajtásának és optimalizálásának módjáról is részletes adatokat biztosít.</span><span class="sxs-lookup"><span data-stu-id="275fd-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="275fd-220">Például az Azure SQL Database felügyeleti portál Lekérdezés teljesítménye paneljén megtekintheti a kiválasztott lekérdezés futásidejű teljesítményadatait.</span><span class="sxs-lookup"><span data-stu-id="275fd-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="275fd-221">A `GetAllFieldsAsync` művelet által létrehozott lekérdezés a következő:</span><span class="sxs-lookup"><span data-stu-id="275fd-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![A Microsoft Azure SQL Database felügyeleti portál Lekérdezés részletei panelje][QueryDetails]

## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="275fd-223">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="275fd-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="275fd-224">Miután módosítottuk a `GetRequiredFieldsAsync` metódust, hogy egy SELECT utasítást használjon az adatbázisoldalon, a terhelésteszt a következő eredményeket mutatta.</span><span class="sxs-lookup"><span data-stu-id="275fd-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![A GetRequiredFieldsAsync metódus terhelésiteszt-eredményei][Load-Test-Results-Database-Side1]

<span data-ttu-id="275fd-226">Ez a terhelésteszt ugyanazon az üzemelő példányon és a 400 egyidejű felhasználót szimuláló terhelésen alapul.</span><span class="sxs-lookup"><span data-stu-id="275fd-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="275fd-227">A diagram sokkal alacsonyabb késleltetést mutat.</span><span class="sxs-lookup"><span data-stu-id="275fd-227">The graph shows much lower latency.</span></span> <span data-ttu-id="275fd-228">A válaszidő a terhelés hatására hozzávetőlegesen 1,3 másodpercre emelkedik (az előző esetben 4 másodperc volt).</span><span class="sxs-lookup"><span data-stu-id="275fd-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="275fd-229">Az átviteli sebesség is magasabb, korábban másodpercenként 100 kérés érkezett be, ezúttal 350.</span><span class="sxs-lookup"><span data-stu-id="275fd-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="275fd-230">Az adatbázisból lekért adatmennyiség most már megközelíti a HTTP-válaszüzenetek méretét.</span><span class="sxs-lookup"><span data-stu-id="275fd-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![A GetRequiredFieldsAsync metódus telemetriája][TelemetryRequiredFields]

<span data-ttu-id="275fd-232">Az `AggregateOnDatabaseAsync` metódus terhelési tesztje a következő eredményeket hozza:</span><span class="sxs-lookup"><span data-stu-id="275fd-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![Az AggregateOnDatabaseAsync metódus terhelésiteszt-eredményei][Load-Test-Results-Database-Side2]

<span data-ttu-id="275fd-234">Az átlagos válaszidő most minimális.</span><span class="sxs-lookup"><span data-stu-id="275fd-234">The average response time is now minimal.</span></span> <span data-ttu-id="275fd-235">Ez egy teljes nagyságrendnyi teljesítménynövekedést jelent, amely az adatbázis ki- és bemenő adatforgalmában beállt csökkentésének köszönhető.</span><span class="sxs-lookup"><span data-stu-id="275fd-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span>

<span data-ttu-id="275fd-236">Itt van az `AggregateOnDatabaseAsync` metódushoz tartozó megfelelő telemetria.</span><span class="sxs-lookup"><span data-stu-id="275fd-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="275fd-237">Az adatbázisból lekért adatok mennyisége nagymértékben lecsökkent, tranzakciónkénti 280 kilobájtról 53 bájtra.</span><span class="sxs-lookup"><span data-stu-id="275fd-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="275fd-238">Ennek eredménye, hogy a maximálisan fenntartható percenkénti kérések száma 2000-ről 25 000-re növekedett.</span><span class="sxs-lookup"><span data-stu-id="275fd-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![Az AggregateOnDatabaseAsync metódus telemetriája][TelemetryAggregateInDatabaseAsync]

## <a name="related-resources"></a><span data-ttu-id="275fd-240">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="275fd-240">Related resources</span></span>

- <span data-ttu-id="275fd-241">[Foglalt adatbázissal kapcsolatos kizárási minták][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="275fd-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="275fd-242">[Forgalmas I/O kizárási minta][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="275fd-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="275fd-243">[Ajánlott adatparticionálási eljárások][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="275fd-243">[Data partitioning best practices][data-partitioning]</span></span>

[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
