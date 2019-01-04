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
# <a name="extraneous-fetching-antipattern"></a>Felesleges beolvasások kizárási minta

Ha egyes üzleti műveletekhez a szükségesnél több adatot kér le, az szükségtelen I/O túlműködést eredményez, és csökkenti a válaszképességet.

## <a name="problem-description"></a>A probléma leírása

Ez a kizárási minta akkor jelentkezik, ha az alkalmazás az I/O-kérések mennyiségének csökkentése érdekében lekéri az összes adatot, amelyre *esetleg* szüksége lehet. Ez gyakran a [Forgalmas I/O][chatty-io] kizárási minta ellensúlyozásának eredményeképpen jelentkezik. Előfordulhat például. hogy az alkalmazás lekéri az adatbázisban található összes termék adatait. A felhasználónak azonban csak az adatok egy részhalmazára van szüksége (a többi nem is feltétlenül érinti a felhasználót), és valószínűleg nem is szeretné az *összes* terméket egyszerre megtekinteni. Még ha a felhasználó a teljes katalógusban böngész is, észszerű az eredményeket külön lapokra osztani, például egyszerre csak 20 találat megjelenítésével.

A probléma másik oka a nem megfelelő programozási és kialakítási gyakorlatokban keresendő. A következő kód például az Entity Framework használatával lekéri az összes termék minden adatát. Ezután szűri az adatokat, és csak a mezők egy részét adja vissza, a többit pedig elveti. A teljes kódmintát [itt][sample-app] találja.

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

A következő példában az alkalmazás egy olyan összesítéshez kér le adatokat, amelyet az adatbázis is elvégezhetett volna. Az alkalmazás az értékesítések végösszegének kiszámításához lekéri az összes értékesített megrendeléshez tartozó minden bejegyzést, majd ezekből számítja ki a végösszeget. A teljes kódmintát [itt][sample-app] találja.

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

A következő példa egy kisebb hibát mutat be, amely abból származik, ahogy az Entity Framework a LINQ to Entities műveletet alkalmazza.

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

Az alkalmazás az egy hétnél régebbi `SellStartDate` dátummal rendelkező termékeket keresi. A legtöbb esetben a LINQ to Entities a `where` záradékokat az adatbázis által végrehajtandó SQL-utasításokká fordítaná le. Ebben az esetben azonban a LINQ to Entities nem tudja az `AddDays` metódust SQL-re leképezni. Ehelyett a `Product` tábla minden egyes sorát visszaadja, és az eredmények szűrését a memóriában végzi el.

Az `AsEnumerable` meghívása utal arra, hogy itt probléma van. Ez a metódus az eredményeket egy `IEnumerable` felületté konvertálja. Bár az `IEnumerable` támogatja a szűrést, arra az *ügyféloldalon*, és nem az adatbázisban kerül sor. A LINQ to Entities alapértelmezés szerint az `IQueryable` függvényt használja, amely a szűrési feladat felelősségét átadja az adatforrásnak.

## <a name="how-to-fix-the-problem"></a>A probléma megoldása

Kerülje az olyan nagyobb adatmennyiségek lekérését, amelyek elavulhatnak vagy amelyek el lesznek vetve, és csak az épp végrehajtott művelethez szükséges adatokat hívja le.

Ahelyett, hogy valamely tábla összes oszlopát lekérné, majd szűrést futtatna rajtuk, válassza ki a kívánt oszlopokat az adatbázisban.

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

Ugyanígy az összesítéseket is az adatbázisban, és ne az alkalmazás memóriájában hajtsa végre.

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

Az Entity Framework használatakor bizonyosodjon meg róla, hogy a LINQ-lekérdezéseket az `IQueryable` felület és nem az `IEnumerable` hajtja végre. Előfordulhat, hogy módosítania kell a lekérdezést, hogy csak olyan függvényeket használjon, amelyek az adatforrásra vonatkoztathatók. A korábbi példa átdolgozható: ha az `AddDays` metódust eltávolítja a lekérdezésből, a szűrést az adatbázisban lehet végrehajtani.

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out.
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a>Megfontolandó szempontok

- Egyes esetekben a teljesítmény az adatok vízszintes particionálásával növelhető. Ha különböző műveletek az adatok különböző tulajdonságait használják, a vízszintes particionálás csökkentheti a versengést. A műveletek nagy része csak az adatok kisebb részhalmazát érinti, így ennek a terhelésnek az elosztásával fokozható a teljesítmény. Lásd: [Adatparticionálás][data-partitioning].

- A korlátlan lekérdezéseket támogató műveletnél alkalmazzon tördelést, és egyszerre csak korlátozott számú kérjen le. Ha például az ügyfél egy termékkatalógust böngész, egyszerre elég egy oldalnyi találatot megjeleníteni.

- Amikor csak lehetséges, használja az adatbázis beépített szolgáltatásait. Az SQL-adatbázisok például általában kínálnak aggregátumfüggvényeket.

- Ha olyan adattárat használ, amely nem támogat egy adott függvényt, például az összesítést, a számított eredményeket máshol tárolhatja, és a rekordok hozzáadásakor vagy eltávolításakor frissítheti az értéket, így az alkalmazásnak nem kell azt minden egyes alkalommal újraszámolnia, amikor szükség van rá.

- Ha azt látja, hogy a kérések nagy számú mezőt hívnak le, vizsgálja meg a forráskódot, és állapítsa meg, hogy valóban mindegyik mező szükséges-e. Néha az ilyen kérések rosszul összeállított `SELECT *` lekérdezések eredményei.

- Ugyanígy a nagy mennyiségű entitást lekérő kérések is utalhatnak arra, hogy az alkalmazás nem megfelelően szűri az adatokat. Ellenőrizze, hogy az összes ilyen entitás valóban szükséges-e. Amennyiben lehetséges, alkalmazzon adatbázisoldali szűrést, például az SQL `WHERE` záradékai használatával.

- A feldolgozás kiszervezése az adatbázisba nem minden esetben a legjobb lehetőség. Csak akkor alkalmazza ezt a stratégiát, ha az adatbázis erre van kialakítva vagy optimalizálva. A legtöbb adatbázisrendszer rendkívüli mértékben optimalizálva van adott függvényekre, azonban általános célú alkalmazásmotorként nem alkalmas. További információkért lásd: [Foglalt adatbázis kizárási minta][BusyDatabase].

## <a name="how-to-detect-the-problem"></a>A probléma észlelése

A feladathoz nem kapcsolódó beolvasások tünetei a magas késleltetés és az alacsony átviteli sebesség. Ha az adatok lekérésének forrása egy adattár, megnövekedett mértékű versengéssel is számolni kell. A végfelhasználók várhatóan magas válaszidőket vagy a szolgáltatások időtúllépéséből eredő hibákat fognak jelezni. Ezek a hibák HTTP 500-as (belső kiszolgáló) vagy HTTP 503-as (a szolgáltatás nem érhető el) hibaüzeneteket eredményezhetnek. Vizsgálja át a webkiszolgáló eseménynaplóit, amelyek valószínűleg részletesebb információkat tartalmaznak a hibák okairól és körülményeiről.

A kizárási minta tünetei és a gyűjtött telemetriaadatok nagyon hasonlóak lehetnek a [Monolitikus adatmegőrzés kizárási mintáéihoz][MonolithicPersistence].

A következő lépéseket végezheti el a hiba okának meghatározásához:

1. Azonosítsa a lassabb számítási feladatokat vagy tranzakciókat terheléstesztek, folyamatmonitorozás és a rendszerállapot-adatok rögzítésére szolgáló egyéb módszerek használatával.
2. Figyelje meg a rendszer viselkedési mintáit. Jelentkeznek bizonyos korlátok a tranzakciók másodpercenkénti számában vagy a felhasználók mennyiségében?
3. Vesse össze a lassú számítási feladatok példányait a viselkedési mintákkal.
4. Azonosítsa a használt adattárakat. Mindegyik adatforrás esetében futtasson alacsonyabb szinten is telemetriavizsgálatot a műveletek viselkedésének megfigyelésére.
5. Azonosítsa az ezekre az adatforrásokra hivatkozó lassú lefutású lekérdezéseket.
6. Végezze el a lassú lefutású lekérdezések erőforrás-specifikus elemzését, és ismerje meg az adatok használatának és felhasználásának módját.

Az alábbi tüneteket figyelje:

- Gyakori, nagyméretű I/O kérések, amelyek ugyanarra az erőforrásra vagy adattárra irányulnak.
- Versengés egy megosztott erőforrásban vagy adattárban.
- Valamely művelet gyakran fogad nagy mennyiségű adatot a hálózaton keresztül.
- Az alkalmazások és szolgáltatások jelentős időt töltenek azzal, hogy az I/O befejeztére várnak.

## <a name="example-diagnosis"></a>Diagnosztikai példa

Az alábbi szakaszokban ezeket a lépéseket az előzőleg ismertetett példákon hajtjuk végre.

### <a name="identify-slow-workloads"></a>A lassú számítási feladatok azonosítása

A diagram egy, a korábban bemutatott `GetAllFieldsAsync` metódust futtató, akár 400 egyidejű felhasználót szimuláló terhelésteszt teljesítményeredményeit mutatja. A terhelés növekedtével a teljesítmény lassan csökken. Az átlagos válaszidő a számítási feladatok növekedtével szintén növekszik.

![A GetAllFieldsAsync metódus terheléstesztjének eredményei][Load-Test-Results-Client-Side1]

Az `AggregateOnClientAsync` művelet terheléstesztje hasonló eredményeket mutat. A kérések mennyisége viszonylag stabil. A számítási feladatok növekedtével az átlagos válaszidő is nő, de lassabb tempóban, mint az előző ábrán.

![Az AggregateOnClientAsync metódus terheléstesztjének eredményei][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a>A számítási feladatok összevetése a viselkedési mintákkal

A magas kihasználtságú időszakok és a teljesítménylassulás rendszeres egybeesése problémás területeket jelezhet. Vizsgálja meg alaposan a gyaníthatóan lassan futó funkciók teljesítményprofilját, és állapítsa meg, hogy egyezik-e a korábban végrehajtott terhelésteszt eredményeivel.

Végezze el ugyanezeknek a funkcióknak a terheléstesztjét lépésközönként növelt felhasználóterhelésekkel, és keresse meg azt a pontot, ahol a teljesítmény jelentős mértékben csökken vagy teljesen meg is szűnik. Ha ez a pont a várható valós használati esetek határain belül esik, vizsgálja meg a funkciók megvalósítását.

Egy lassú művelet nem feltétlenül jelent problémát, ha nem a rendszer terhelt állapotában fut le, ha nem időfüggő, és ha nem befolyásolja negatívan más fontos műveletek teljesítményét. Például a havi működési statisztikák elkészítése hosszú lefutású művelet, azonban valószínűleg végrehajtható kötegelt műveletként, és futtatható alacsony prioritású feladatként. Másfelől a termékkatalógus ügyfelek általi böngészése üzleti szempontból kritikus művelet. Összpontosítson az ilyen kritikus műveletek eredményezte telemetriaadatokra, mivel ezekből látható, hogy a teljesítmény hogyan alakul a magas kihasználtságú időszakokban.

### <a name="identify-data-sources-in-slow-workloads"></a>A lassú számítási feladatok adatforrásainak azonosítása

Ha azt gyanítja, hogy egy szolgáltatás az adatokat lehívásának módja miatt teljesít gyengén, vizsgálja meg, hogy az alkalmazás hogyan kapcsolódik az általa használt adattárakhoz. Az éles rendszer monitorozásával vizsgálja meg, hogy az alkalmazás melyik forrásokhoz fér hozzá azokban az időszakokban, amikor romlik a teljesítmény.

Mindegyik adatforrás esetében állítsa be úgy a rendszert, hogy a következőket rögzítse:

- Az egyes adattárak hozzáférésének gyakoriságát.
- Az adattárba be- és onnan kilépő adatok mennyiségét.
- A műveletek időzítését, különös tekintettel a kérések késleltetésére.
- Az egyes adatforrások tipikus terhelés melletti hozzáférése esetén jelentkező hibák természetét és arányát.

Vesse össze ezeket az információkat az alkalmazás által az ügyfélnek visszaadott adatok mennyiségével. Vesse össze az adattár által visszaadott adatok mennyiségi arányát az ügyfélnek visszaadott adatok mennyiségével. Amennyiben nagy eltérés tapasztalható, vizsgálja meg a alkalmazást, és állapítsa meg, hogy hív-e le olyan adatokat, amelyekre nincs szüksége.

Az adatok az élő rendszer megfigyelésével, az egyes felhasználói kérések életciklusának nyomon követésével rögzíthetőek, vagy lemodellezheti szintetikus számítási feladatok egy sorozatát, amelyeket lefuttathat egy tesztrendszeren.

A következő diagramok a [New Relic APM][new-relic] használatával a `GetAllFieldsAsync` metódus terheléstesztje során rögzített telemetriaadatokat mutatják. Figyelje meg az eltérést az adatbázisról fogadott adatok és a kapcsolódó HTTP-válaszok mennyiségei között.

![A GetAllFieldsAsync metódus telemetriája][TelemetryAllFields]

Az adatbázis minden kérésnél 80 503 bájt adatot adott vissza, az ügyfélnek küldött válasz azonban csak 19 855 bájt adatot tartalmazott, ami az adatbázis válaszának hozzávetőleg 25%-a. Az ügyfélnek visszaadott adatok mérete a formátumtól függően változhat. Ebben a terheléstesztben az ügyfél JSON-adatokat kért. Egy másik, az XML formátum használatával végzett teszt során (nem látható) a válasz mérete 35 655 bájt volt, vagyis az adatbázis válaszának 44%-a.

Az `AggregateOnClientAsync` metódus terheléstesztje még szélsőségesebb eredményeket mutat. Ebben az esetben mindegyik teszt egy lekérdezést hajtott végre, amely több mint 280 kB adatot kért le az adatbázisból, a JSON-válasz mérete azonban mindössze 14 bájt volt. A jelentős eltérés oka, hogy a metódus egy összesített eredményt számít ki nagy mennyiségű adatból.

![Az AggregateOnClientAsync metódus telemetriája][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a>A lassú lekérdezések azonosítása és elemzése

Keresse meg azokat az adatbázis-lekérdezéseket, amelyek a legtöbb erőforrást emésztik fel, és a végrehajtásuk a legtöbb időt veszi igénybe. Megadhat olyan rendszerállapotokat, amelyek lehetővé teszik számos adatbázis-művelet kezdési és befejezési időpontjának kimutatását. Sok adattár emellett a lekérdezések végrehajtásának és optimalizálásának módjáról is részletes adatokat biztosít. Például az Azure SQL Database felügyeleti portál Lekérdezés teljesítménye paneljén megtekintheti a kiválasztott lekérdezés futásidejű teljesítményadatait. A `GetAllFieldsAsync` művelet által létrehozott lekérdezés a következő:

![A Microsoft Azure SQL Database felügyeleti portál Lekérdezés részletei panelje][QueryDetails]

## <a name="implement-the-solution-and-verify-the-result"></a>A megoldás megvalósítása és az eredmény ellenőrzése

Miután módosítottuk a `GetRequiredFieldsAsync` metódust, hogy egy SELECT utasítást használjon az adatbázisoldalon, a terhelésteszt a következő eredményeket mutatta.

![A GetRequiredFieldsAsync metódus terhelésiteszt-eredményei][Load-Test-Results-Database-Side1]

Ez a terhelésteszt ugyanazon az üzemelő példányon és a 400 egyidejű felhasználót szimuláló terhelésen alapul. A diagram sokkal alacsonyabb késleltetést mutat. A válaszidő a terhelés hatására hozzávetőlegesen 1,3 másodpercre emelkedik (az előző esetben 4 másodperc volt). Az átviteli sebesség is magasabb, korábban másodpercenként 100 kérés érkezett be, ezúttal 350. Az adatbázisból lekért adatmennyiség most már megközelíti a HTTP-válaszüzenetek méretét.

![A GetRequiredFieldsAsync metódus telemetriája][TelemetryRequiredFields]

Az `AggregateOnDatabaseAsync` metódus terhelési tesztje a következő eredményeket hozza:

![Az AggregateOnDatabaseAsync metódus terhelésiteszt-eredményei][Load-Test-Results-Database-Side2]

Az átlagos válaszidő most minimális. Ez egy teljes nagyságrendnyi teljesítménynövekedést jelent, amely az adatbázis ki- és bemenő adatforgalmában beállt csökkentésének köszönhető.

Itt van az `AggregateOnDatabaseAsync` metódushoz tartozó megfelelő telemetria. Az adatbázisból lekért adatok mennyisége nagymértékben lecsökkent, tranzakciónkénti 280 kilobájtról 53 bájtra. Ennek eredménye, hogy a maximálisan fenntartható percenkénti kérések száma 2000-ről 25 000-re növekedett.

![Az AggregateOnDatabaseAsync metódus telemetriája][TelemetryAggregateInDatabaseAsync]

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

- [Foglalt adatbázissal kapcsolatos kizárási minták][BusyDatabase]
- [Forgalmas I/O kizárási minta][chatty-io]
- [Ajánlott adatparticionálási eljárások][data-partitioning]

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
