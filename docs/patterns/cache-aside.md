---
title: Cache-Aside
description: Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: d4d7c9dcd612c780e3e494509a57b6b4a0144423
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012460"
---
# <a name="cache-aside-pattern"></a>Gyorsítótár-feltöltési minta

[!INCLUDE [header](../_includes/header.md)]

Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból. Ez javíthatja a teljesítményt, illetve segít a gyorsítótárban és a mögöttes adattárban tárolt adatok konzisztenciájának fenntartásában.

## <a name="context-and-problem"></a>Kontextus és probléma

Az alkalmazások gyorsítótár használatával javítják az adattárakban tárolt információkhoz való ismételt hozzáférést. Azt azonban nem célszerű feltételezni, hogy a gyorsítótárazott adatok mindig teljes mértékben konzisztensek az adattárban található adatokkal. Az alkalmazásokban olyan stratégiát érdemes megvalósítani, amely segít a lehető legnaprakészebb állapotban tartani a gyorsítótár adatait, de az olyan helyzeteket is tudja észlelni és kezelni, amikor a gyorsítótár adatai elavulttá váltak.

## <a name="solution"></a>Megoldás

Számos kereskedelmi gyorsítótárazási rendszer kínál visszaolvasási és visszaírási/háttérírási műveleteket. Ezekben a rendszerekben az alkalmazás a gyorsítótárra való hivatkozással kéri le az adatokat. Ha nincsenek a gyorsítótárban, az alkalmazás lekéri az adatokat az adattárból, majd hozzáadja őket a gyorsítótárhoz. A gyorsítótárban tárolt adatok módosításait a rendszer azonnal visszaírja az adattárba is.

Az olyan gyorsítótárak esetében, amelyek nem biztosítják ezt a funkciót, a gyorsítótárat használó alkalmazásoknak kell karbantartaniuk az adatokat.

Az alkalmazások a gyorsítótár-feltöltő stratégiával emulálni tudják a visszaolvasási gyorsítótárazás funkciót. Ez a stratégia igény szerint tölti be az adatokat a gyorsítótárba. Az ábrán látható, hogyan tárolhatók adatok a gyorsítótárban a gyorsítótár-feltöltési minta használatával.

![Adatok tárolása a gyorsítótárban a gyorsítótár-feltöltési minta használatával](./_images/cache-aside-diagram.png)


Ha egy alkalmazás adatokat frissít, úgy követheti a visszaírási stratégiát, ha végrehajthatja a módosítást az adattárban, és érvényteleníti a megfelelő elemet a gyorsítótárban.

Amikor az elemre legközelebb szükség van, a gyorsítótár-feltöltő stratégia használatának köszönhetően az alkalmazás az adattárból kéri le a frissített adatokat, és újra hozzáadja őket a gyorsítótárhoz.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe: 

**A gyorsítótárazott adatok élettartama**. Számos gyorsítótár használ elévülési szabályzatot, amely érvényteleníti és eltávolítja az adatokat a gyorsítótárból, ha adott ideig nem történik hozzáférés. A hatékony gyorsítótár-felöltés érdekében győződjön meg arról, hogy az elévülési szabályzat megfelel az adatokat használó alkalmazások hozzáférési mintájának. Ne szabja túl rövidre az elévülési időt, mert ez azt okozhatja, hogy az alkalmazások folyamatosan lekérik az adatokat az adattárból, és hozzáadják őket a gyorsítótárhoz. Az elévülési idő ne legyen olyan hosszú sem, hogy a gyorsítótárazott adatok elavuljanak. Vegye figyelembe, hogy a gyorsítótárazás a viszonylag statikus vagy gyakran beolvasott adatok esetében a leghatékonyabb.

**Adatok kizárása**. A legtöbb gyorsítótár mérete korlátozott azokhoz az adattárakhoz képest, ahonnan az adatok származnak, és szükség esetén kizárják az adatokat. A legtöbb gyorsítótár a legrégebbi használat alapján választja ki a kizárandó elemeket, ez azonban testre szabható lehet. Konfigurálja a globális lejárati tulajdonságot, a gyorsítótár egyéb tulajdonságait és az összes gyorsítótárazott elem lejáratát tulajdonságát úgy, hogy a gyorsítótár a lehető legköltséghatékonyabb legyen. Nem mindig jó megoldás, ha globális kizárási szabályzatot alkalmaz a gyorsítótár összes elemére. Ha egy elemnek például rendkívül költséges az adattárból való lekérése, akkor előnyös lehet az elemet a gyorsítótárban tartani még a gyakrabban használt, de kevésbé költséges elemek rovására is.

**A gyorsítótár előkészítése**. Számos megoldás előre feltölti a gyorsítótárat olyan adatokkal, amelyekre az alkalmazásnak valószínűleg szüksége lesz az első feldolgozás részeként. A gyorsítótár-feltöltési minta az adatok egy részének elavulása vagy kizárása esetén is hasznos lehet.

**Konzisztencia**. A gyorsítótár-feltöltési minta megvalósítása nem garantálja az adattár és a gyorsítótár közötti konzisztenciát. A külső folyamatok bármikor módosíthatják az adattár elemeit, és előfordulhat, hogy a gyorsítótár nem tükrözi ezeket a változásokat az elem következő betöltéséig. Olyan rendszer esetében, amely több adattárba replikál adatokat, ez komoly problémát jelenthet gyakori szinkronizálás esetén.

**Helyi (memóriában történő) gyorsítótárazás**. Az alkalmazáspéldányok helyi gyorsítótárral is rendelkezhetnek, amelyet a memória tárol. A gyorsítótár-feltöltés hasznos lehet az ilyen környezetben, ha az alkalmazás gyakran fér hozzá ugyanazon adatokhoz. A helyi gyorsítótárak azonban magánjellegűek, ezért előfordulhat, hogy a különböző alkalmazáspéldányok mind rendelkeznek ugyanazon gyorsítótárazott adatok példányaival. Ezek az adatok gyorsan inkonzisztenssé válhatnak a gyorsítótárak között, ezért szükség lehet a magánjellegű gyorsítótárak adatainak gyakoribb elévülésére és frissítésére. Ezekben az esetekben érdemes megfontolni egy megosztott vagy elosztott gyorsítótárazási mechanizmus használatát.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- A gyorsítótár nem biztosít natív visszaolvasási és visszaírási műveleteket.
- Az erőforrásigény nem kiszámítható. Ez a minta lehetővé teszi az alkalmazások számára az adatok igény szerinti betöltését. A minta nem feltételezi előzetesen, hogy az alkalmazásoknak milyen adatokra lesz szükségük.

Nem érdemes ezt a mintát használni, ha:

- a gyorsítótárazott adatkészlet statikus. Ha az adatok elférnek a rendelkezésre álló gyorsítótártérben, töltse fel a gyorsítótárat az adatokkal indításkor, és alkalmazzon egy szabályzatot, amely megakadályozza az adatok elévülését.
- egy webfarmon üzemeltetett webalkalmazásban kíván munkamenet-állapotinformációkat gyorsítótárazni. Ebben a környezetben érdemes elkerülnie az ügyfél–kiszolgáló affinitáson alapuló függőségek bevezetését.

## <a name="example"></a>Példa

A Microsoft Azure-ban az Azure Redis Cache használatával létrehozhat elosztott gyorsítótárakat, amelyek megoszthatók egy alkalmazás több példánya között. 

Egy Azure Redis Cache-példányhoz való csatlakozáshoz hívja meg a `Connect` statikus metódust, és adja meg a kapcsolati karakterláncot. A metódus egy `ConnectionMultiplexer` értéket ad vissza, amely a kapcsolatot jelöli. Az alkalmazásban egy `ConnectionMultiplexer` példány megosztására egy lehetséges módszer, ha létrehoz egy statikus tulajdonságot, amely egy csatlakoztatott példányt ad vissza, a következő példához hasonlóan. Ez a megközelítés egy szálbiztos módszert biztosít egyetlen csatlakoztatott példány inicializálásához.

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

A következő példakódban szereplő `GetMyEntityAsync` metódus az Azure Redis Cache-en alapuló gyorsítótár-feltöltési minta egy megvalósítását mutatja be. Ez a módszer egy objektum a read-through megközelítéssel a gyorsítótárból kéri le.

A metódus egy egész számból álló azonosítót használ kulcsként az objektumok azonosításához. A `GetMyEntityAsync` metódus megkísérel lekérdezni egy elemet a gyorsítótárból ezzel a kulccsal. Ha a metódus talál egyező elemet, akkor visszaadja. Ha nincs egyezés a gyorsítótárban, a `GetMyEntityAsync` metódus lekérdezi az objektumot az adattárból, hozzáadja a gyorsítótárhoz, majd visszaadja. Az adatokat az adattárból ténylegesen beolvasó kód itt nem látható, mert az az adattártól függően változhat. Figyelje meg, hogy a gyorsítótárazott elem úgy van konfigurálva, hogy elévüljön, és ne váljon elavulttá, ha máshol frissítik.


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

>  A példák az Azure Redis Cache API használatával férnek hozzá a tárolóhoz és kérdeznek le adatokat a gyorsítótárból. További információ: [A Microsoft Azure Redis Cache használata](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) és [Webalkalmazás létrehozása a Redis Cache használatával](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)

Az alább látható `UpdateEntityAsync` metódus azt mutatja be, hogyan érvényteleníthető egy objektum a gyorsítótárban, ha egy alkalmazás módosítja az értékét. A kód frissíti az eredeti adattárat, majd eltávolítja a gyorsítótárazott elemet a gyorsítótárból.

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
> A lépések sorrendje rendkívül fontos. Frissítse az adattárat, *mielőtt* eltávolítaná az elemet a gyorsítótárból. Ha először eltávolítja a gyorsítótárazott elemet, akkor az ügyfélnek csak rövid ideje lesz az elem lekérésére az adattár frissítése előtt. Emiatt a gyorsítótárból nem lesz találat (mivel az elemet eltávolították a gyorsítótárból), így a rendszer az elem egy korábbi változatát kéri le az adattárból és adja hozzá újra a gyorsítótárhoz. Ez pedig elavult gyorsítótár-adatokat eredményez.


## <a name="related-guidance"></a>Kapcsolódó útmutatók 

Az alábbi információk segíthetnek a minta megvalósításakor:

- [Gyorsítótárazási útmutató](https://docs.microsoft.com/azure/architecture/best-practices/caching). További információkat biztosít arról, hogyan gyorsítótárazhat adatokat a felhőalapú megoldásokban, valamint a gyorsítótárak kialakítása előtt megfontolandó szempontokról.

- [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx). A felhőalapú alkalmazások általában több adattárból származó adatokat használnak. Az adatkonzisztencia felügyelete és fenntartása ebben a környezetben a rendszer kritikus fontosságú tényezője, különösen az egyidejűségi és rendelkezésre állási problémák miatt. Ez az ismertető az elosztott adatok esetében felmerülő konzisztenciaproblémákat mutatja be, és összefoglalja, hogyan biztosíthatják az alkalmazások az adatok rendelkezésre állását a végleges konzisztencia megvalósításával.
