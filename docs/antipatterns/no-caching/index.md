---
title: Nincs gyorsítótárazás – kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: Ugyanazon adatok ismételt beolvasása csökkentheti a teljesítményt és a méretezhetőséget.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: c2a5cdbb8863f87b8928558c8237e8659032ac32
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011599"
---
# <a name="no-caching-antipattern"></a>Nincs gyorsítótárazás – kizárási minta

Ha egy számos egyidejű kérést kezelő felhőalkalmazásban ismételten beolvassa ugyanazokat az adatokat, csökkenhet a teljesítmény és a méretezhetőség.

## <a name="problem-description"></a>A probléma leírása

Ha az adatok nincsenek gyorsítótárazva, az számos nem kívánt viselkedést okozhat, például:

- Ugyanazon információ ismételt beolvasása olyan erőforrásokból, amelyeknek az elérése költséges (I/O vagy késés szempontjából).
- Ugyanazon objektumok vagy adatstruktúrák ismételt felépítése a különböző kérések eredményeként.
- Túl sok hívás kezdeményezése egy távoli, kvótával rendelkező szolgáltatás irányába, amely egy bizonyos korlátot túllépve szabályozza az ügyfeleket.

Ezek a problémák hosszú válaszidőkhöz, a nagyobb adattárbeli versengéshez és gyenge méretezhetőséghez vezetnek.

A következő példa az Entity Framework használatával csatlakozik egy adatbázishoz. Minden ügyfélkérés az adatbázis hívását eredményezi, még akkor is, ha több kérés is ugyanazt az adatot olvassa be. Az ismételt kérések költsége – az I/O terhelését és az adathozzáférési díjakat illetően – gyorsan megnövekedhet.

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

A teljes kódmintát [itt][sample-app] találja.

A kizárási minta jellemzően az alábbi okokból következhet be:

- A gyorsítótár használatának elkerülése egyszerűbben megvalósítható, és kis terhelés esetén jól működik. A gyorsítótárazás bonyolultabbá teszi a kódot.
- A gyorsítótár használatának előnyei és hátrányai nem teljesen világosak.
- A gyorsítótárazott adatok frissességének és pontosságának fenntartásának költségei aggályokat vetnek fel.
- Az alkalmazást olyan helyszíni rendszerről migrálták, amelyen a hálózati késés nem jelentett problémát, a rendszer pedig drága, nagy teljesítményű hardveren futott, ezért a gyorsítótárazás lehetőségét nem vették figyelembe az eredeti kialakítás során.
- A fejlesztők nem tudnak róla, hogy a gyorsítótárazás lehetséges egy adott helyzetben. Előfordulhat például, hogy a fejlesztők nem gondolnak az ETagek használatára egy webes API megvalósításakor.

## <a name="how-to-fix-the-problem"></a>A probléma megoldása

A legnépszerűbb gyorsítótárazási stratégia az *igény szerinti* vagy *gyorsítótár-feltöltő* módszer.

- Olvasáskor az alkalmazás megpróbálja beolvasni az adatokat a gyorsítótárból. Ha nincsenek a gyorsítótárban, az alkalmazás lekéri az adatokat az adatforrásból, majd hozzáadja őket a gyorsítótárhoz.
- Íráskor az alkalmazás közvetlenül az adatforrásba írja a módosítást, a régi értéket pedig eltávolítja a gyorsítótárból. Amikor legközelebb szükség van rá, a rendszer lekéri, majd hozzáadja a gyorsítótárhoz az adatokat.

Ez a módszer a gyakran módosuló adatokhoz alkalmas. Itt van az előző példa, a [Cache-Aside][cache-aside-pattern] minta használatára frissítve.

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

A `GetAsync` metódus ezúttal nem közvetlenül az adatbázist, hanem a `CacheService` osztályt hívja meg. A `CacheService` osztály először megkísérli lekérni az elemet az Azure Redis Cache-ből. Ha az érték nem található a Redis Cache-ben, a `CacheService` meghív egy lambda függvényt, amelyet a hívó adott át. A lambda függvény felelős az adat adatbázisból történő beolvasásáért. Ez a megvalósítás leválasztja az adattárat az adott gyorsítótárazási megoldásról, valamint a `CacheService`-t elemet az adatbázisról.

## <a name="considerations"></a>Megfontolandó szempontok

- Ha a gyorsítótár nem érhető el, például átmeneti hiba miatt, ne jelezzen hibát az ügyfélnek. Ehelyett olvassa be az adatot az eredeti adatforrásból. Vegye azonban figyelembe, hogy a gyorsítótár helyreállítása közben az eredeti adattárat eláraszthatják a kérések, ami időtúllépéseket és sikertelen kapcsolatokat eredményez. (Eleve ez az egyik fő ok a gyorsítótár használatára.) Az adatforrás túlterhelésének elkerülése érdekében használjon az [Áramköri megszakító mintához][circuit-breaker] hasonló technikát.

- A nem statikus adatokat gyorsítótárazó alkalmazásokat úgy kell megtervezni, hogy támogassák a későbbi konzisztenciát.

- A webes API-k esetében támogatható az ügyféloldali gyorsítótárazás, ha elhelyez egy gyorsítótár-vezérlő fejlécet a kérés- és válaszüzenetekben, valamint ETageket használ az objektumok verzióinak azonosítására. További információ: [API-megvalósítás][api-implementation].

- Nem szükséges egész entitásokat gyorsítótáraznia. Ha egy entitás legnagyobbrészt statikus, de egy kis része gyakran változik, akkor a statikus elemeket gyorsítótárazza, a dinamikus elemeket pedig kérje le az adatforrásból. Ez a módszer segíthet csökkenteni az adatforráson végzett I/O-műveletek mennyiségét.

- Bizonyos esetekben hasznos lehet gyorsítótárazni a rövid életű ideiglenes adatokat is. Vegyünk például egy olyan eszközt, amely folyamatosan állapotfrissítéseket küld. Érdemes lehet a gyorsítótárba helyezni a beérkező információt, és egyáltalán nem írni állandó tárolóba.

- Az adatok elavulásának megakadályozása érdekében számos gyorsítótárazási megoldás támogatja a lejárati idő konfigurálását. A rendszer így automatikusan eltávolítja az adatokat a gyorsítótárból a megadott idő elteltével. A lejárati időt a forgatókönyvnek megfelelően érdemes beállítani. A nagymértékben statikus adatok hosszabb ideig a gyorsítótárban maradhatnak, mint az ideiglenes, gyorsan elavuló adatok.

- Ha a gyorsítótárazási megoldás nem biztosít beépített lejárati időt, akkor szükséges lehet egy olyan háttérfolyamat megvalósítása, amely időnként kiüríti a gyorsítótárat, hogy ne tudjon korlátlanul nőni.

- A gyorsítótárazás a külső adatforrásból érkező adatok tárolása mellett a komplex számítások eredményeinek mentésére is alkalmas. Előbb azonban meg kell határozni, hogy az alkalmazás valóban a processzorhoz van-e kötve.

- Hasznos lehet előkészíteni a gyorsítótárat az alkalmazás indításakor. Töltse fel a gyorsítótárat azokkal az adatokkal, amelyekre a legnagyobb valószínűséggel lesz szükség.

- Minden esetben olyan kialakítást válasszon, amely észleli a sikeres és sikertelen gyorsítótárbeli kereséseket. Ezekből az információkból gyorsítótárazási szabályokat alakíthat ki, amelyek meghatározzák például azt, hogy mely adatokat kell gyorsítótárazni, illetve meddig kell megtartani őket, mielőtt elavulnak.

- Ha a gyorsítótárazás hiánya szűk keresztmetszetet jelent, akkor a hozzáadása olyan mértékben növelheti a kérések számát, hogy a webes kezelőfelület túlterhelését okozhatja. Előfordulhat, hogy az ügyfelek HTTP 503 (A szolgáltatás nem érhető el) hibákat kapnak. Ez arra utal, hogy a kezelőfelület horizontális felskálázást igényel.

## <a name="how-to-detect-the-problem"></a>A probléma észlelése

A következő lépésekkel határozhatja meg, hogy a gyorsítótárazás hiánya okoz-e teljesítményproblémákat:

1. Tekintse át az alkalmazás kialakítását. Készítsen leltárt az alkalmazás által használt összes adattárolóról. Mindegyik esetében állapítsa meg, hogy az alkalmazás használ-e gyorsítótárat. Ha lehetséges, állapítsa meg az adatok változásának gyakoriságát. Elsőként olyan adatokat célszerű gyorsítótárazni, amely ritkán változnak, valamint a gyakran olvasott statikus referenciaadatokat.

2. Alakítsa ki az alkalmazást, és monitorozza az élő rendszert annak megállapításához, hogy az alkalmazás milyen gyakorisággal kér le adatokat vagy számít ki információkat.

3. Készítse el az alkalmazás profilját tesztkörnyezetben az adathozzáférési műveletekhez vagy gyakran elvégzett számításokhoz kapcsolódó terhelés részletes metrikáinak rögzítéséhez.

4. Végezzen terhelési tesztet tesztkörnyezetben annak megállapításához, hogy a rendszer hogyan reagál átlagos, illetve nagy terhelés alatt. A terhelési tesztnek az éles környezetben megfigyelt, valósághű számítási feladatok mellett megfigyelt adat-hozzáférési mintákat.

5. Vizsgálja meg a teszt alapját képező adattárolók adat-hozzáférési statisztikáit, és ellenőrizze, milyen gyakran ismétlődnek az ugyanazon adatokra vonatkozó kérések.

## <a name="example-diagnosis"></a>Diagnosztikai példa

Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.

### <a name="instrument-the-application-and-monitor-the-live-system"></a>Az alkalmazás kialakítása és az élő rendszer monitorozása

Alakítsa ki és monitorozza az alkalmazást, hogy információkhoz jusson a felhasználók által az alkalmazás működése közben küldött kérésekről.

A következő képen a [New Relic][NewRelic] által egy terhelési teszt során rögzített monitorozási adatok láthatók. Ebben az esetben az egyetlen végrehajtott HTTP GET művelet a következő: `Person/GetAsync`. Éles környezetben azonban, ha ismeri az egyes kérések relatív gyakoriságát, információhoz juthat arról, hogy mely erőforrásokat érdemes gyorsítótárazni.

![A New Relic által ábrázolt kiszolgálói kérések a CachingDemo alkalmazáshoz][NewRelic-server-requests]

Ha mélyebb elemzésre van szükség, használhat profilkészítőt az alacsony szintű teljesítményadatok tesztkörnyezetben (nem üzemi környezetben) történő rögzítéséhez. Különösen figyeljen az I/O-kérések mennyiségére, valamint a memória és a processzor kihasználtságára vonatkozó metrikákra. Ezek a metrikák egy adattárolóra vagy szolgáltatásra irányuló, nagy mennyiségű kérést mutathatnak, vagy ismételt folyamatokat, amelyek ugyanazon számítást végzik el.

### <a name="load-test-the-application"></a>Az alkalmazás terheléstesztje

A következő diagram a mintaalkalmazás terhelési tesztjének eredményeit ábrázolja. A terhelési teszt legfeljebb 800 felhasználó által egyidejűleg végzett, jellemző művelettípusok lépéses terhelését szimulálja.

![A teljesítményterhelési teszt eredményei a gyorsítótárazás nélküli forgatókönyv esetében][Performance-Load-Test-Results-Uncached]

A másodpercenként végzett sikeres tesztek száma eléri a maximumot, emiatt pedig a további kérések lelassulnak. Az átlagos tesztelési idő a terheléssel arányosan, egyenletesen növekszik. A válaszidőszintek nem változnak, ahogy a felhasználói terhelés eléri a maximumot.

### <a name="examine-data-access-statistics"></a>Az adathozzáférési statisztikák vizsgálata

Az adathozzáférési statisztikák és az adattároló által szolgáltatott egyéb információk hasznosak lehetnek például annak megállapításában, hogy melyik lekérdezések ismétlődnek a leggyakrabban. A Microsoft SQL Serverben például a `sys.dm_exec_query_stats` felügyeleti nézet statisztikai információkkal szolgál a legutóbb végrehajtott lekérdezésekről. Az egyes lekérdezések szövege elérhető a `sys.dm_exec-query_plan` nézetben. Egy erre alkalmas eszköz, például az SQL Server Management Studio használatával futtathatja a következő SQL-lekérdezést annak megállapításához, milyen gyakran végez a rendszer lekérdezéseket.

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

Az eredmények között látható `UseCount` oszlop jelzi, hogy az egyes lekérdezések milyen gyakran futnak. A következő képen látható, hogy a harmadik lekérdezés több mint 250 000 alkalommal futott, vagyis lényegesen többször, mint bármely más lekérdezés.

![A SQL Server Management Server dinamikus felügyeleti nézeteinek lekérdezési eredményei][Dynamic-Management-Views]

Itt látható az SQL-lekérdezés, amely ennyi adatbázis-kérés benyújtását eredményezi:

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

Ez az Entity Framework által a korábban bemutatott `GetByIdAsync` metódusban létrehozott lekérdezés.

### <a name="implement-the-solution-and-verify-the-result"></a>A megoldás megvalósítása és az eredmény ellenőrzése

A gyorsítótár megvalósítása után ismételje meg a terhelési teszteket, majd hasonlítsa össze az eredményeket a korábbi, gyorsítótár nélküli terhelési tesztekével. Itt láthatók a gyorsítótár mintaalkalmazáshoz való hozzáadása után végzett teszt eredményei.

![A teljesítményterhelési teszt eredményei a gyorsítótáras forgatókönyv esetében][Performance-Load-Test-Results-Cached]

A sikeres tesztek mennyisége így is tetőzik, de magasabb felhasználói terhelésnél. A kérések mennyisége e terhelés esetében a korábbinál sokkal magasabb. Az átlagos tesztelési idő továbbra is nő a terheléssel, viszont a maximális válaszidő 0,05 ezredmásodperc. A korábban mért 1 ezredmásodperchez képest ez &mdash;20-szoros&times; javulást jelent.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

- [API-implementáció – Ajánlott eljárások][api-implementation]
- [Gyorsítótár-feltöltő mód][cache-aside-pattern]
- [Gyorsítótárazás – Ajánlott eljárások][caching-guidance]
- [Áramköri megszakítási minta][circuit-breaker]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: https://newrelic.com/partner/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg
