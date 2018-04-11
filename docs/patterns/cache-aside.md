---
title: Cache-Aside
description: Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 1536a33884c9c9faa1e3702c951067249e691bf8
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="cache-aside-pattern"></a>Gyorsítótár-Tartalékoljon minta

[!INCLUDE [header](../_includes/header.md)]

Az igény szerinti adatok betöltése a gyorsítótárba egy adattárból. Ez javíthatja a teljesítményt, és biztosítja az egységességet a gyorsítótárat, és az adatok az alapul szolgáló adattár-ban tárolt adatok között is segíti.

## <a name="context-and-problem"></a>A környezetben, és probléma

Alkalmazások használják a gyorsítótárat ismételt érhető el a tárolóban tárolt információ javítása érdekében. Azt azonban nem kivitelezhető várható, hogy mindig lesz teljes mértékben egységes az adatokat az adattár a gyorsítótárazott adatokat. Alkalmazások kell végrehajtása, amelyek segítségével győződjön meg arról, hogy a gyorsítótárban lévő adatok legfrissebb, lehetséges, de is észlelni és kezelésére, amikor merülhetnek fel, ha a gyorsítótárban lévő adatok elavult vált.

## <a name="solution"></a>Megoldás

Számos kereskedelmi gyorsítótárazási rendszer read-through és írási-keresztül/késleltetve visszaírt műveletek biztosítanak. Ezek a rendszerek, az alkalmazás lekéri az adatokat a gyorsítótár Vezérlőpultjának. Ha az adatok nem a gyorsítótárban, az adatok tárolójából beolvasni, és a gyorsítótárba. A gyorsítótárban tárolt adatok módosításai automatikusan kerüljenek vissza az adattárban.

Az gyorsítótár, amely nem ad meg ezt a funkciót, a feladata az adatok a gyorsítótárat használó alkalmazások.

Az alkalmazás képes emulálják a read-through gyorsítótárazásához azáltal, hogy a gyorsítótár-tartalékoljon stratégia megvalósításához. Ezt a stratégiát adatokat tölt az igény szerinti a gyorsítótárba. Az ábra azt mutatja be, a gyorsítótár-Tartalékoljon minta használatával az adatok tárolásához a gyorsítótárban.

![A gyorsítótár-Tartalékoljon minta használatával az adatok tárolásához a gyorsítótárban](./_images/cache-aside-diagram.png)


Ha egy alkalmazás frissíti az adatokat, az azáltal, hogy a módosítás az adattárolóhoz, és a megfelelő elem a gyorsítótárban érvénytelenítése kövesse az író stratégia.

Ha az elem melletti szükség, a gyorsítótár-tartalékoljon stratégia használata miatt a frissített adatokat lehet beolvasni az adattár és vissza a gyorsítótárba.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat: 

**A gyorsítótárazott adatokat élettartama**. Sok gyorsítótárak egy elévülési szabályzattal, amely érvénytelenné válik az adatokat, és eltávolítja a gyorsítótárból, ha nem érhető el egy megadott ideig valósítja meg. A gyorsítótár-tartalékoljon hatékony legyen meg kell egyeznie a minta az access, az adatokat használó alkalmazások esetén a lejárati házirendet. A túl rövid lejárati időt nem végre, mert okozhatnak, folyamatosan adatainak lekérése a tárolót, és adja hozzá a gyorsítótár-alkalmazások. Ehhez hasonlóan ne ellenőrizze a lejárati időszak túl sokáig, hogy a gyorsítótárazott adatok várhatóan elavult. Ne feledje, hogy gyorsítótárazás leghatékonyabb viszonylag statikus vagy gyakran beolvasott adatok számára.

**Adatok kizárásának**. A legtöbb gyorsítótárak rendelkezik egy korlátozott méretét, az adatok áruház, ahonnan az adatok származnak, és szükség esetén ezek lesz kizárása adatok képest. A legtöbb gyorsítótárak elfogadják a legkevésbé legutóbb használt házirendet kizárása elemek kijelölése, de ez lehet testre szabható. A globális lejárati tulajdonság és egyéb tulajdonságai, a gyorsítótár és a lejárati tulajdonság minden gyorsítótárazott elem annak biztosításához, hogy a gyorsítótár költséghatékony konfigurálása. Nem mindig megfelelő globális kiürítés házirend alkalmazása a gyorsítótár elemével. Például ha egy gyorsítótárazott elem beolvasni az adattár nagyon költséges, akkor érdemes lehet megtartja ezt az elemet a gyorsítótár rovására több gyakran használt, de kevésbé költséges elemet.

**A gyorsítótár előkészítési**. Sok megoldás az adatok, amelyek az alkalmazás valószínű, hogy az indítási feldolgozási részeként kell gyorsítótár előre. A gyorsítótár-Tartalékoljon mintát továbbra is lehet hasznos, ha az adatok egy részét lejár, vagy ki van zárva.

**Konzisztencia**. A gyorsítótár-Tartalékoljon mintát megvalósító nem biztosítja a tárolót, és a gyorsítótár közötti konzisztenciát. Egy elem szerepel az adattárban módosíthatja bármikor külső folyamat, és ez a változás esetleg nem tükröződnek a gyorsítótárban mindaddig, amíg megint az elem be van töltve. A rendszerben, amely adattárolók között replikálja az adatokat a probléma is súlyossá válhat, ha szinkronizálási gyakran előfordul.

**(A memóriában) helyi gyorsítótárazás**. Lehet, hogy helyi egy alkalmazáspéldányt, és a memóriában tárolt gyorsítótár. Gyorsítótár-tartalékoljon ebben a környezetben hasznos lehet, ha egy alkalmazás ismételten ugyanazokhoz az adatokhoz fér hozzá. Azonban a helyi gyorsítótárba személyes, és így különböző alkalmazáspéldányok sikerült mindegyik rendelkezik-e másolatával azonos gyorsítótárazott adatokat. Ezek az adatok gyorsan válhat,-gyorsítótárak közötti inkonzisztens titkos gyorsítótárban tárolt adatok lejárnak, és frissítse az gyakrabban szükség lehet így. A következő használati helyzetekben vizsgálja meg a megosztott vagy elosztott gyorsítótárazást.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- A gyorsítótár nem biztosít natív read-through és író művelet.
- Erőforrás igény szerint előre nem látható. Ez a minta segítségével az alkalmazások az igény szerinti adatok betöltése. Így nincs mely adatokra vonatkozó kérelmet kell előzetesen feltételezéseket.

Ez a minta nem feltétlenül alkalmas:

- Ha a gyorsítótárazott adatkészlet statikus. Ha az adatokat a rendelkezésre álló gyorsítótár-terület elfér, a gyorsítótárban levő adatokkal indítási rendszerlemez, és alkalmazzon, amely megakadályozza az adatok lejárjanak házirendet.
- A munkamenet állapotinformációja webfarm egy webalkalmazást a gyorsítótárazáshoz. Ebben a környezetben ne ügyfél-kiszolgáló affinitás alapján függőségek bemutatása.

## <a name="example"></a>Példa

A Microsoft Azure-ban Azure Redis Cache segítségével hozzon létre egy elosztott gyorsítótár, amely egy alkalmazás több példánya megoszthatók. 

Csatlakozás az Azure Redis Cache példányt, hívja meg a statikus `Connect` metódus és pass a kapcsolati karakterláncban. A metódus visszaadja a `ConnectionMultiplexer` , amely kapcsolatot jelöli. Az alkalmazásban egy `ConnectionMultiplexer` példány megosztására egy lehetséges módszer, ha létrehoz egy statikus tulajdonságot, amely egy csatlakoztatott példányt ad vissza, a következő példához hasonlóan. Ez a megközelítés biztosítja az szálbiztos csak egyetlen csatlakoztatott példány inicializálása.

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

A `GetMyEntityAsync` metódus a következő kódrészlet példa a megvalósítást a gyorsítótár-Tartalékoljon minta Azure Redis Cache alapján jeleníti meg. Ez a módszer egy objektum a Ha megközelítéssel a gyorsítótárból kéri le.

Az objektum azonosít egy egész azonosítójával kulcsként. A `GetMyEntityAsync` metódus megkísérli lekérni egy elemet az ezt a kulcsot a gyorsítótárból. Ha egy egyező elem található, az adott vissza. Ha nem szerepel a gyorsítótárban a `GetMyEntityAsync` módszer lekéri az objektum egy adattárból, hozzáadja a gyorsítótárhoz, és majd visszaadja. A kódot, amely ténylegesen az adatokat olvas az adattár nem itt jelenik meg, mert az adattár függ. Vegye figyelembe, hogy a gyorsítótárazott elem van-e konfigurálva, nehogy azt a elévültek, ha frissítette máshol lejár.


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  A példák a tárolóhoz, és információt lekérni a gyorsítótár az Azure Redis Cache API segítségével. További információkért lásd: [használatával a Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) és [Redis Cache-webalkalmazás létrehozása](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)

A `UpdateEntityAsync` alább látható módszer bemutatja, hogyan kell érvénytelenné válnak a gyorsítótár objektumára, az alkalmazás által az érték megváltozásakor. A kód frissíti az eredeti adattárolóban, és majd eltávolítja a gyorsítótárazott elem a gyorsítótárból.

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> Fontos, a lépések sorrendjét. Az adattár frissítése *előtt* elem eltávolítása a gyorsítótárból. Ha először távolítsa el a gyorsítótárazott elem, akkor egy kis ablakban idő, amikor egy ügyfél előfordulhat, hogy beolvasni az elem az adattár frissítése előtt. Vezethet a gyorsítótár-tévesztései (mivel az elem el lett távolítva a gyorsítótárból), a korábbi verziót az adattárból beolvasását elem okozza, és vissza a gyorsítótár fel. A gyorsítótár elavult adatokat lesz.


## <a name="related-guidance"></a>Kapcsolódó útmutatók 

A következő információkat lehet megfelelő, ebben a mintában végrehajtása során:

- [Útmutatás gyorsítótárazás](https://docs.microsoft.com/azure/architecture/best-practices/caching). További tájékoztatást nyújt hogyan egy felhőalapú megoldáson, és a problémák is figyelembe kell vennie a gyorsítótár bevezetésekor adatokat képes gyorsítótárazni.

- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). A felhőalapú alkalmazásokhoz általában használja megjelenített adattárolókhoz. Kezelése, és biztosítja az adatok konzisztenciáját ebben a környezetben az egyik fontos szempontja, hogy a rendszer, különösen a feldolgozási mód és a rendelkezésre állási kapcsolatban felmerülő hibákat. A kezdés konzisztencia kapcsolatos problémákat ismerteti elosztott adatokon keresztül, és összefoglalja, hogyan alkalmazás rendelkezésre állásának adatok végleges konzisztencia is létrehozható.
