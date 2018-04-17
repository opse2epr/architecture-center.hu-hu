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
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="44d99-104">Gyorsítótár-feltöltési minta</span><span class="sxs-lookup"><span data-stu-id="44d99-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="44d99-105">Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból.</span><span class="sxs-lookup"><span data-stu-id="44d99-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="44d99-106">Ez javíthatja a teljesítményt, illetve segít a gyorsítótárban és a mögöttes adattárban tárolt adatok konzisztenciájának fenntartásában.</span><span class="sxs-lookup"><span data-stu-id="44d99-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="44d99-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="44d99-107">Context and problem</span></span>

<span data-ttu-id="44d99-108">Az alkalmazások gyorsítótár használatával javítják az adattárakban tárolt információkhoz való ismételt hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="44d99-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="44d99-109">Azt azonban nem célszerű feltételezni, hogy a gyorsítótárazott adatok mindig teljes mértékben konzisztensek az adattárban található adatokkal.</span><span class="sxs-lookup"><span data-stu-id="44d99-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="44d99-110">Az alkalmazásokban olyan stratégiát érdemes megvalósítani, amely segít a lehető legnaprakészebb állapotban tartani a gyorsítótár adatait, de az olyan helyzeteket is tudja észlelni és kezelni, amikor a gyorsítótár adatai elavulttá váltak.</span><span class="sxs-lookup"><span data-stu-id="44d99-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="44d99-111">Megoldás</span><span class="sxs-lookup"><span data-stu-id="44d99-111">Solution</span></span>

<span data-ttu-id="44d99-112">Számos kereskedelmi gyorsítótárazási rendszer kínál visszaolvasási és visszaírási/háttérírási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="44d99-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="44d99-113">Ezekben a rendszerekben az alkalmazás a gyorsítótárra való hivatkozással kéri le az adatokat.</span><span class="sxs-lookup"><span data-stu-id="44d99-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="44d99-114">Ha nincsenek a gyorsítótárban, az alkalmazás lekéri az adatokat az adattárból, majd hozzáadja őket a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="44d99-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="44d99-115">A gyorsítótárban tárolt adatok módosításait a rendszer azonnal visszaírja az adattárba is.</span><span class="sxs-lookup"><span data-stu-id="44d99-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="44d99-116">Az olyan gyorsítótárak esetében, amelyek nem biztosítják ezt a funkciót, a gyorsítótárat használó alkalmazásoknak kell karbantartaniuk az adatokat.</span><span class="sxs-lookup"><span data-stu-id="44d99-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="44d99-117">Az alkalmazások a gyorsítótár-feltöltő stratégiával emulálni tudják a visszaolvasási gyorsítótárazás funkciót.</span><span class="sxs-lookup"><span data-stu-id="44d99-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="44d99-118">Ez a stratégia igény szerint tölti be az adatokat a gyorsítótárba.</span><span class="sxs-lookup"><span data-stu-id="44d99-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="44d99-119">Az ábrán látható, hogyan tárolhatók adatok a gyorsítótárban a gyorsítótár-feltöltési minta használatával.</span><span class="sxs-lookup"><span data-stu-id="44d99-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![Adatok tárolása a gyorsítótárban a gyorsítótár-feltöltési minta használatával](./_images/cache-aside-diagram.png)


<span data-ttu-id="44d99-121">Ha egy alkalmazás adatokat frissít, úgy követheti a visszaírási stratégiát, ha végrehajthatja a módosítást az adattárban, és érvényteleníti a megfelelő elemet a gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="44d99-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="44d99-122">Amikor az elemre legközelebb szükség van, a gyorsítótár-feltöltő stratégia használatának köszönhetően az alkalmazás az adattárból kéri le a frissített adatokat, és újra hozzáadja őket a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="44d99-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="44d99-123">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="44d99-123">Issues and considerations</span></span>

<span data-ttu-id="44d99-124">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="44d99-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="44d99-125">**A gyorsítótárazott adatok élettartama**.</span><span class="sxs-lookup"><span data-stu-id="44d99-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="44d99-126">Számos gyorsítótár használ elévülési szabályzatot, amely érvényteleníti és eltávolítja az adatokat a gyorsítótárból, ha adott ideig nem történik hozzáférés.</span><span class="sxs-lookup"><span data-stu-id="44d99-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="44d99-127">A hatékony gyorsítótár-felöltés érdekében győződjön meg arról, hogy az elévülési szabályzat megfelel az adatokat használó alkalmazások hozzáférési mintájának.</span><span class="sxs-lookup"><span data-stu-id="44d99-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="44d99-128">Ne szabja túl rövidre az elévülési időt, mert ez azt okozhatja, hogy az alkalmazások folyamatosan lekérik az adatokat az adattárból, és hozzáadják őket a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="44d99-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="44d99-129">Az elévülési idő ne legyen olyan hosszú sem, hogy a gyorsítótárazott adatok elavuljanak.</span><span class="sxs-lookup"><span data-stu-id="44d99-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="44d99-130">Vegye figyelembe, hogy a gyorsítótárazás a viszonylag statikus vagy gyakran beolvasott adatok esetében a leghatékonyabb.</span><span class="sxs-lookup"><span data-stu-id="44d99-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="44d99-131">**Adatok kizárása**.</span><span class="sxs-lookup"><span data-stu-id="44d99-131">**Evicting data**.</span></span> <span data-ttu-id="44d99-132">A legtöbb gyorsítótár mérete korlátozott azokhoz az adattárakhoz képest, ahonnan az adatok származnak, és szükség esetén kizárják az adatokat.</span><span class="sxs-lookup"><span data-stu-id="44d99-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="44d99-133">A legtöbb gyorsítótár a legrégebbi használat alapján választja ki a kizárandó elemeket, ez azonban testre szabható lehet.</span><span class="sxs-lookup"><span data-stu-id="44d99-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="44d99-134">Konfigurálja a globális lejárati tulajdonságot, a gyorsítótár egyéb tulajdonságait és az összes gyorsítótárazott elem lejáratát tulajdonságát úgy, hogy a gyorsítótár a lehető legköltséghatékonyabb legyen.</span><span class="sxs-lookup"><span data-stu-id="44d99-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="44d99-135">Nem mindig jó megoldás, ha globális kizárási szabályzatot alkalmaz a gyorsítótár összes elemére.</span><span class="sxs-lookup"><span data-stu-id="44d99-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="44d99-136">Ha egy elemnek például rendkívül költséges az adattárból való lekérése, akkor előnyös lehet az elemet a gyorsítótárban tartani még a gyakrabban használt, de kevésbé költséges elemek rovására is.</span><span class="sxs-lookup"><span data-stu-id="44d99-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="44d99-137">**A gyorsítótár előkészítése**.</span><span class="sxs-lookup"><span data-stu-id="44d99-137">**Priming the cache**.</span></span> <span data-ttu-id="44d99-138">Számos megoldás előre feltölti a gyorsítótárat olyan adatokkal, amelyekre az alkalmazásnak valószínűleg szüksége lesz az első feldolgozás részeként.</span><span class="sxs-lookup"><span data-stu-id="44d99-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="44d99-139">A gyorsítótár-feltöltési minta az adatok egy részének elavulása vagy kizárása esetén is hasznos lehet.</span><span class="sxs-lookup"><span data-stu-id="44d99-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="44d99-140">**Konzisztencia**.</span><span class="sxs-lookup"><span data-stu-id="44d99-140">**Consistency**.</span></span> <span data-ttu-id="44d99-141">A gyorsítótár-feltöltési minta megvalósítása nem garantálja az adattár és a gyorsítótár közötti konzisztenciát.</span><span class="sxs-lookup"><span data-stu-id="44d99-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="44d99-142">A külső folyamatok bármikor módosíthatják az adattár elemeit, és előfordulhat, hogy a gyorsítótár nem tükrözi ezeket a változásokat az elem következő betöltéséig.</span><span class="sxs-lookup"><span data-stu-id="44d99-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="44d99-143">Olyan rendszer esetében, amely több adattárba replikál adatokat, ez komoly problémát jelenthet gyakori szinkronizálás esetén.</span><span class="sxs-lookup"><span data-stu-id="44d99-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="44d99-144">**Helyi (memóriában történő) gyorsítótárazás**.</span><span class="sxs-lookup"><span data-stu-id="44d99-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="44d99-145">Az alkalmazáspéldányok helyi gyorsítótárral is rendelkezhetnek, amelyet a memória tárol.</span><span class="sxs-lookup"><span data-stu-id="44d99-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="44d99-146">A gyorsítótár-feltöltés hasznos lehet az ilyen környezetben, ha az alkalmazás gyakran fér hozzá ugyanazon adatokhoz.</span><span class="sxs-lookup"><span data-stu-id="44d99-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="44d99-147">A helyi gyorsítótárak azonban magánjellegűek, ezért előfordulhat, hogy a különböző alkalmazáspéldányok mind rendelkeznek ugyanazon gyorsítótárazott adatok példányaival.</span><span class="sxs-lookup"><span data-stu-id="44d99-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="44d99-148">Ezek az adatok gyorsan inkonzisztenssé válhatnak a gyorsítótárak között, ezért szükség lehet a magánjellegű gyorsítótárak adatainak gyakoribb elévülésére és frissítésére.</span><span class="sxs-lookup"><span data-stu-id="44d99-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="44d99-149">Ezekben az esetekben érdemes megfontolni egy megosztott vagy elosztott gyorsítótárazási mechanizmus használatát.</span><span class="sxs-lookup"><span data-stu-id="44d99-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="44d99-150">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="44d99-150">When to use this pattern</span></span>

<span data-ttu-id="44d99-151">Használja ezt a mintát, ha:</span><span class="sxs-lookup"><span data-stu-id="44d99-151">Use this pattern when:</span></span>

- <span data-ttu-id="44d99-152">A gyorsítótár nem biztosít natív visszaolvasási és visszaírási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="44d99-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="44d99-153">Az erőforrásigény nem kiszámítható.</span><span class="sxs-lookup"><span data-stu-id="44d99-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="44d99-154">Ez a minta lehetővé teszi az alkalmazások számára az adatok igény szerinti betöltését.</span><span class="sxs-lookup"><span data-stu-id="44d99-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="44d99-155">A minta nem feltételezi előzetesen, hogy az alkalmazásoknak milyen adatokra lesz szükségük.</span><span class="sxs-lookup"><span data-stu-id="44d99-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="44d99-156">Nem érdemes ezt a mintát használni, ha:</span><span class="sxs-lookup"><span data-stu-id="44d99-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="44d99-157">a gyorsítótárazott adatkészlet statikus.</span><span class="sxs-lookup"><span data-stu-id="44d99-157">When the cached data set is static.</span></span> <span data-ttu-id="44d99-158">Ha az adatok elférnek a rendelkezésre álló gyorsítótártérben, töltse fel a gyorsítótárat az adatokkal indításkor, és alkalmazzon egy szabályzatot, amely megakadályozza az adatok elévülését.</span><span class="sxs-lookup"><span data-stu-id="44d99-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="44d99-159">egy webfarmon üzemeltetett webalkalmazásban kíván munkamenet-állapotinformációkat gyorsítótárazni.</span><span class="sxs-lookup"><span data-stu-id="44d99-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="44d99-160">Ebben a környezetben érdemes elkerülnie az ügyfél–kiszolgáló affinitáson alapuló függőségek bevezetését.</span><span class="sxs-lookup"><span data-stu-id="44d99-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="44d99-161">Példa</span><span class="sxs-lookup"><span data-stu-id="44d99-161">Example</span></span>

<span data-ttu-id="44d99-162">A Microsoft Azure-ban az Azure Redis Cache használatával létrehozhat elosztott gyorsítótárakat, amelyek megoszthatók egy alkalmazás több példánya között.</span><span class="sxs-lookup"><span data-stu-id="44d99-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="44d99-163">Egy Azure Redis Cache-példányhoz való csatlakozáshoz hívja meg a `Connect` statikus metódust, és adja meg a kapcsolati karakterláncot.</span><span class="sxs-lookup"><span data-stu-id="44d99-163">To connect to an Azure Redis Cache instance, call the static `Connect` method and pass in the connection string.</span></span> <span data-ttu-id="44d99-164">A metódus egy `ConnectionMultiplexer` értéket ad vissza, amely a kapcsolatot jelöli.</span><span class="sxs-lookup"><span data-stu-id="44d99-164">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="44d99-165">Az alkalmazásban egy `ConnectionMultiplexer` példány megosztására egy lehetséges módszer, ha létrehoz egy statikus tulajdonságot, amely egy csatlakoztatott példányt ad vissza, a következő példához hasonlóan.</span><span class="sxs-lookup"><span data-stu-id="44d99-165">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="44d99-166">Ez a megközelítés egy szálbiztos módszert biztosít egyetlen csatlakoztatott példány inicializálásához.</span><span class="sxs-lookup"><span data-stu-id="44d99-166">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

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

<span data-ttu-id="44d99-167">A következő példakódban szereplő `GetMyEntityAsync` metódus az Azure Redis Cache-en alapuló gyorsítótár-feltöltési minta egy megvalósítását mutatja be.</span><span class="sxs-lookup"><span data-stu-id="44d99-167">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern based on Azure Redis Cache.</span></span> <span data-ttu-id="44d99-168">Ez a módszer egy objektum a read-through megközelítéssel a gyorsítótárból kéri le.</span><span class="sxs-lookup"><span data-stu-id="44d99-168">This method retrieves an object from the cache using the read-through approach.</span></span>

<span data-ttu-id="44d99-169">A metódus egy egész számból álló azonosítót használ kulcsként az objektumok azonosításához.</span><span class="sxs-lookup"><span data-stu-id="44d99-169">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="44d99-170">A `GetMyEntityAsync` metódus megkísérel lekérdezni egy elemet a gyorsítótárból ezzel a kulccsal.</span><span class="sxs-lookup"><span data-stu-id="44d99-170">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="44d99-171">Ha a metódus talál egyező elemet, akkor visszaadja.</span><span class="sxs-lookup"><span data-stu-id="44d99-171">If a matching item is found, it's returned.</span></span> <span data-ttu-id="44d99-172">Ha nincs egyezés a gyorsítótárban, a `GetMyEntityAsync` metódus lekérdezi az objektumot az adattárból, hozzáadja a gyorsítótárhoz, majd visszaadja.</span><span class="sxs-lookup"><span data-stu-id="44d99-172">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="44d99-173">Az adatokat az adattárból ténylegesen beolvasó kód itt nem látható, mert az az adattártól függően változhat.</span><span class="sxs-lookup"><span data-stu-id="44d99-173">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="44d99-174">Figyelje meg, hogy a gyorsítótárazott elem úgy van konfigurálva, hogy elévüljön, és ne váljon elavulttá, ha máshol frissítik.</span><span class="sxs-lookup"><span data-stu-id="44d99-174">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


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

>  <span data-ttu-id="44d99-175">A példák az Azure Redis Cache API használatával férnek hozzá a tárolóhoz és kérdeznek le adatokat a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="44d99-175">The examples use the Azure Redis Cache API to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="44d99-176">További információ: [A Microsoft Azure Redis Cache használata](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) és [Webalkalmazás létrehozása a Redis Cache használatával](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span><span class="sxs-lookup"><span data-stu-id="44d99-176">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="44d99-177">Az alább látható `UpdateEntityAsync` metódus azt mutatja be, hogyan érvényteleníthető egy objektum a gyorsítótárban, ha egy alkalmazás módosítja az értékét.</span><span class="sxs-lookup"><span data-stu-id="44d99-177">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="44d99-178">A kód frissíti az eredeti adattárat, majd eltávolítja a gyorsítótárazott elemet a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="44d99-178">The code updates the original data store and then removes the cached item from the cache.</span></span>

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
> <span data-ttu-id="44d99-179">A lépések sorrendje rendkívül fontos.</span><span class="sxs-lookup"><span data-stu-id="44d99-179">The order of the steps is important.</span></span> <span data-ttu-id="44d99-180">Frissítse az adattárat, *mielőtt* eltávolítaná az elemet a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="44d99-180">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="44d99-181">Ha először eltávolítja a gyorsítótárazott elemet, akkor az ügyfélnek csak rövid ideje lesz az elem lekérésére az adattár frissítése előtt.</span><span class="sxs-lookup"><span data-stu-id="44d99-181">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="44d99-182">Emiatt a gyorsítótárból nem lesz találat (mivel az elemet eltávolították a gyorsítótárból), így a rendszer az elem egy korábbi változatát kéri le az adattárból és adja hozzá újra a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="44d99-182">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="44d99-183">Ez pedig elavult gyorsítótár-adatokat eredményez.</span><span class="sxs-lookup"><span data-stu-id="44d99-183">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="44d99-184">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="44d99-184">Related guidance</span></span> 

<span data-ttu-id="44d99-185">Az alábbi információk segíthetnek a minta megvalósításakor:</span><span class="sxs-lookup"><span data-stu-id="44d99-185">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="44d99-186">[Gyorsítótárazási útmutató](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span><span class="sxs-lookup"><span data-stu-id="44d99-186">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="44d99-187">További információkat biztosít arról, hogyan gyorsítótárazhat adatokat a felhőalapú megoldásokban, valamint a gyorsítótárak kialakítása előtt megfontolandó szempontokról.</span><span class="sxs-lookup"><span data-stu-id="44d99-187">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="44d99-188">[Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="44d99-188">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="44d99-189">A felhőalapú alkalmazások általában több adattárból származó adatokat használnak.</span><span class="sxs-lookup"><span data-stu-id="44d99-189">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="44d99-190">Az adatkonzisztencia felügyelete és fenntartása ebben a környezetben a rendszer kritikus fontosságú tényezője, különösen az egyidejűségi és rendelkezésre állási problémák miatt.</span><span class="sxs-lookup"><span data-stu-id="44d99-190">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="44d99-191">Ez az ismertető az elosztott adatok esetében felmerülő konzisztenciaproblémákat mutatja be, és összefoglalja, hogyan biztosíthatják az alkalmazások az adatok rendelkezésre állását a végleges konzisztencia megvalósításával.</span><span class="sxs-lookup"><span data-stu-id="44d99-191">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>
