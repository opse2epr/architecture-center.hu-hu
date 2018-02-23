---
title: Cache-Aside
description: "Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból"
keywords: "Kialakítási mintája"
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
# <a name="cache-aside-pattern"></a><span data-ttu-id="5fa2e-104">Gyorsítótár-Tartalékoljon minta</span><span class="sxs-lookup"><span data-stu-id="5fa2e-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="5fa2e-105">Az igény szerinti adatok betöltése a gyorsítótárba egy adattárból.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="5fa2e-106">Ez javíthatja a teljesítményt, és biztosítja az egységességet a gyorsítótárat, és az adatok az alapul szolgáló adattár-ban tárolt adatok között is segíti.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5fa2e-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="5fa2e-107">Context and problem</span></span>

<span data-ttu-id="5fa2e-108">Alkalmazások használják a gyorsítótárat ismételt érhető el a tárolóban tárolt információ javítása érdekében.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="5fa2e-109">Azt azonban nem kivitelezhető várható, hogy mindig lesz teljes mértékben egységes az adatokat az adattár a gyorsítótárazott adatokat.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="5fa2e-110">Alkalmazások kell végrehajtása, amelyek segítségével győződjön meg arról, hogy a gyorsítótárban lévő adatok legfrissebb, lehetséges, de is észlelni és kezelésére, amikor merülhetnek fel, ha a gyorsítótárban lévő adatok elavult vált.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="5fa2e-111">Megoldás</span><span class="sxs-lookup"><span data-stu-id="5fa2e-111">Solution</span></span>

<span data-ttu-id="5fa2e-112">Számos kereskedelmi gyorsítótárazási rendszer read-through és írási-keresztül/késleltetve visszaírt műveletek biztosítanak.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="5fa2e-113">Ezek a rendszerek, az alkalmazás lekéri az adatokat a gyorsítótár Vezérlőpultjának.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="5fa2e-114">Ha az adatok nem a gyorsítótárban, az adatok tárolójából beolvasni, és a gyorsítótárba.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="5fa2e-115">A gyorsítótárban tárolt adatok módosításai automatikusan kerüljenek vissza az adattárban.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="5fa2e-116">Az gyorsítótár, amely nem ad meg ezt a funkciót, a feladata az adatok a gyorsítótárat használó alkalmazások.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="5fa2e-117">Az alkalmazás képes emulálják a read-through gyorsítótárazásához azáltal, hogy a gyorsítótár-tartalékoljon stratégia megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="5fa2e-118">Ezt a stratégiát adatokat tölt az igény szerinti a gyorsítótárba.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="5fa2e-119">Az ábra azt mutatja be, a gyorsítótár-Tartalékoljon minta használatával az adatok tárolásához a gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![A gyorsítótár-Tartalékoljon minta használatával az adatok tárolásához a gyorsítótárban](./_images/cache-aside-diagram.png)


<span data-ttu-id="5fa2e-121">Ha egy alkalmazás frissíti az adatokat, az azáltal, hogy a módosítás az adattárolóhoz, és a megfelelő elem a gyorsítótárban érvénytelenítése kövesse az író stratégia.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="5fa2e-122">Ha az elem melletti szükség, a gyorsítótár-tartalékoljon stratégia használata miatt a frissített adatokat lehet beolvasni az adattár és vissza a gyorsítótárba.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="5fa2e-123">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="5fa2e-123">Issues and considerations</span></span>

<span data-ttu-id="5fa2e-124">Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="5fa2e-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="5fa2e-125">**A gyorsítótárazott adatokat élettartama**.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="5fa2e-126">Sok gyorsítótárak egy elévülési szabályzattal, amely érvénytelenné válik az adatokat, és eltávolítja a gyorsítótárból, ha nem érhető el egy megadott ideig valósítja meg.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="5fa2e-127">A gyorsítótár-tartalékoljon hatékony legyen meg kell egyeznie a minta az access, az adatokat használó alkalmazások esetén a lejárati házirendet.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="5fa2e-128">A túl rövid lejárati időt nem végre, mert okozhatnak, folyamatosan adatainak lekérése a tárolót, és adja hozzá a gyorsítótár-alkalmazások.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="5fa2e-129">Ehhez hasonlóan ne ellenőrizze a lejárati időszak túl sokáig, hogy a gyorsítótárazott adatok várhatóan elavult.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="5fa2e-130">Ne feledje, hogy gyorsítótárazás leghatékonyabb viszonylag statikus vagy gyakran beolvasott adatok számára.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="5fa2e-131">**Adatok kizárásának**.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-131">**Evicting data**.</span></span> <span data-ttu-id="5fa2e-132">A legtöbb gyorsítótárak rendelkezik egy korlátozott méretét, az adatok áruház, ahonnan az adatok származnak, és szükség esetén ezek lesz kizárása adatok képest.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="5fa2e-133">A legtöbb gyorsítótárak elfogadják a legkevésbé legutóbb használt házirendet kizárása elemek kijelölése, de ez lehet testre szabható.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="5fa2e-134">A globális lejárati tulajdonság és egyéb tulajdonságai, a gyorsítótár és a lejárati tulajdonság minden gyorsítótárazott elem annak biztosításához, hogy a gyorsítótár költséghatékony konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="5fa2e-135">Nem mindig megfelelő globális kiürítés házirend alkalmazása a gyorsítótár elemével.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="5fa2e-136">Például ha egy gyorsítótárazott elem beolvasni az adattár nagyon költséges, akkor érdemes lehet megtartja ezt az elemet a gyorsítótár rovására több gyakran használt, de kevésbé költséges elemet.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="5fa2e-137">**A gyorsítótár előkészítési**.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-137">**Priming the cache**.</span></span> <span data-ttu-id="5fa2e-138">Sok megoldás az adatok, amelyek az alkalmazás valószínű, hogy az indítási feldolgozási részeként kell gyorsítótár előre.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="5fa2e-139">A gyorsítótár-Tartalékoljon mintát továbbra is lehet hasznos, ha az adatok egy részét lejár, vagy ki van zárva.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="5fa2e-140">**Konzisztencia**.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-140">**Consistency**.</span></span> <span data-ttu-id="5fa2e-141">A gyorsítótár-Tartalékoljon mintát megvalósító nem biztosítja a tárolót, és a gyorsítótár közötti konzisztenciát.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="5fa2e-142">Egy elem szerepel az adattárban módosíthatja bármikor külső folyamat, és ez a változás esetleg nem tükröződnek a gyorsítótárban mindaddig, amíg megint az elem be van töltve.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="5fa2e-143">A rendszerben, amely adattárolók között replikálja az adatokat a probléma is súlyossá válhat, ha szinkronizálási gyakran előfordul.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="5fa2e-144">**(A memóriában) helyi gyorsítótárazás**.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="5fa2e-145">Lehet, hogy helyi egy alkalmazáspéldányt, és a memóriában tárolt gyorsítótár.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="5fa2e-146">Gyorsítótár-tartalékoljon ebben a környezetben hasznos lehet, ha egy alkalmazás ismételten ugyanazokhoz az adatokhoz fér hozzá.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="5fa2e-147">Azonban a helyi gyorsítótárba személyes, és így különböző alkalmazáspéldányok sikerült mindegyik rendelkezik-e másolatával azonos gyorsítótárazott adatokat.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="5fa2e-148">Ezek az adatok gyorsan válhat,-gyorsítótárak közötti inkonzisztens titkos gyorsítótárban tárolt adatok lejárnak, és frissítse az gyakrabban szükség lehet így.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="5fa2e-149">A következő használati helyzetekben vizsgálja meg a megosztott vagy elosztott gyorsítótárazást.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="5fa2e-150">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="5fa2e-150">When to use this pattern</span></span>

<span data-ttu-id="5fa2e-151">Ez mintát, mikor használja:</span><span class="sxs-lookup"><span data-stu-id="5fa2e-151">Use this pattern when:</span></span>

- <span data-ttu-id="5fa2e-152">A gyorsítótár nem biztosít natív read-through és író művelet.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="5fa2e-153">Erőforrás igény szerint előre nem látható.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="5fa2e-154">Ez a minta segítségével az alkalmazások az igény szerinti adatok betöltése.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="5fa2e-155">Így nincs mely adatokra vonatkozó kérelmet kell előzetesen feltételezéseket.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="5fa2e-156">Ez a minta nem feltétlenül alkalmas:</span><span class="sxs-lookup"><span data-stu-id="5fa2e-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="5fa2e-157">Ha a gyorsítótárazott adatkészlet statikus.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-157">When the cached data set is static.</span></span> <span data-ttu-id="5fa2e-158">Ha az adatokat a rendelkezésre álló gyorsítótár-terület elfér, a gyorsítótárban levő adatokkal indítási rendszerlemez, és alkalmazzon, amely megakadályozza az adatok lejárjanak házirendet.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="5fa2e-159">A munkamenet állapotinformációja webfarm egy webalkalmazást a gyorsítótárazáshoz.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="5fa2e-160">Ebben a környezetben ne ügyfél-kiszolgáló affinitás alapján függőségek bemutatása.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="5fa2e-161">Példa</span><span class="sxs-lookup"><span data-stu-id="5fa2e-161">Example</span></span>

<span data-ttu-id="5fa2e-162">A Microsoft Azure-ban Azure Redis Cache segítségével hozzon létre egy elosztott gyorsítótár, amely egy alkalmazás több példánya megoszthatók.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="5fa2e-163">Csatlakozás az Azure Redis Cache példányt, hívja meg a statikus `Connect` metódus és pass a kapcsolati karakterláncban.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-163">To connect to an Azure Redis Cache instance, call the static `Connect` method and pass in the connection string.</span></span> <span data-ttu-id="5fa2e-164">A metódus visszaadja a `ConnectionMultiplexer` , amely kapcsolatot jelöli.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-164">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="5fa2e-165">Az alkalmazásban egy `ConnectionMultiplexer` példány megosztására egy lehetséges módszer, ha létrehoz egy statikus tulajdonságot, amely egy csatlakoztatott példányt ad vissza, a következő példához hasonlóan.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-165">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="5fa2e-166">Ez a megközelítés biztosítja az szálbiztos csak egyetlen csatlakoztatott példány inicializálása.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-166">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

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

<span data-ttu-id="5fa2e-167">A `GetMyEntityAsync` metódus a következő kódrészlet példa a megvalósítást a gyorsítótár-Tartalékoljon minta Azure Redis Cache alapján jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-167">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern based on Azure Redis Cache.</span></span> <span data-ttu-id="5fa2e-168">Ez a módszer egy objektum a Ha megközelítéssel a gyorsítótárból kéri le.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-168">This method retrieves an object from the cache using the read-though approach.</span></span>

<span data-ttu-id="5fa2e-169">Az objektum azonosít egy egész azonosítójával kulcsként.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-169">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="5fa2e-170">A `GetMyEntityAsync` metódus megkísérli lekérni egy elemet az ezt a kulcsot a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-170">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="5fa2e-171">Ha egy egyező elem található, az adott vissza.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-171">If a matching item is found, it's returned.</span></span> <span data-ttu-id="5fa2e-172">Ha nem szerepel a gyorsítótárban a `GetMyEntityAsync` módszer lekéri az objektum egy adattárból, hozzáadja a gyorsítótárhoz, és majd visszaadja.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-172">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="5fa2e-173">A kódot, amely ténylegesen az adatokat olvas az adattár nem itt jelenik meg, mert az adattár függ.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-173">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="5fa2e-174">Vegye figyelembe, hogy a gyorsítótárazott elem van-e konfigurálva, nehogy azt a elévültek, ha frissítette máshol lejár.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-174">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


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

>  <span data-ttu-id="5fa2e-175">A példák a tárolóhoz, és információt lekérni a gyorsítótár az Azure Redis Cache API segítségével.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-175">The examples use the Azure Redis Cache API to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="5fa2e-176">További információkért lásd: [használatával a Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) és [Redis Cache-webalkalmazás létrehozása](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span><span class="sxs-lookup"><span data-stu-id="5fa2e-176">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="5fa2e-177">A `UpdateEntityAsync` alább látható módszer bemutatja, hogyan kell érvénytelenné válnak a gyorsítótár objektumára, az alkalmazás által az érték megváltozásakor.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-177">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="5fa2e-178">A kód frissíti az eredeti adattárolóban, és majd eltávolítja a gyorsítótárazott elem a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-178">The code updates the original data store and then removes the cached item from the cache.</span></span>

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
> <span data-ttu-id="5fa2e-179">Fontos, a lépések sorrendjét.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-179">The order of the steps is important.</span></span> <span data-ttu-id="5fa2e-180">Az adattár frissítése *előtt* elem eltávolítása a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-180">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="5fa2e-181">Ha először távolítsa el a gyorsítótárazott elem, akkor egy kis ablakban idő, amikor egy ügyfél előfordulhat, hogy beolvasni az elem az adattár frissítése előtt.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-181">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="5fa2e-182">Vezethet a gyorsítótár-tévesztései (mivel az elem el lett távolítva a gyorsítótárból), a korábbi verziót az adattárból beolvasását elem okozza, és vissza a gyorsítótár fel.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-182">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="5fa2e-183">A gyorsítótár elavult adatokat lesz.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-183">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="5fa2e-184">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="5fa2e-184">Related guidance</span></span> 

<span data-ttu-id="5fa2e-185">A következő információkat lehet megfelelő, ebben a mintában végrehajtása során:</span><span class="sxs-lookup"><span data-stu-id="5fa2e-185">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="5fa2e-186">[Útmutatás gyorsítótárazás](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span><span class="sxs-lookup"><span data-stu-id="5fa2e-186">[Caching Guidance](https://docs.microsoft.com/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="5fa2e-187">További tájékoztatást nyújt hogyan egy felhőalapú megoldáson, és a problémák is figyelembe kell vennie a gyorsítótár bevezetésekor adatokat képes gyorsítótárazni.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-187">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="5fa2e-188">[Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="5fa2e-188">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="5fa2e-189">A felhőalapú alkalmazásokhoz általában használja megjelenített adattárolókhoz.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-189">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="5fa2e-190">Kezelése, és biztosítja az adatok konzisztenciáját ebben a környezetben az egyik fontos szempontja, hogy a rendszer, különösen a feldolgozási mód és a rendelkezésre állási kapcsolatban felmerülő hibákat.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-190">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="5fa2e-191">A kezdés konzisztencia kapcsolatos problémákat ismerteti elosztott adatokon keresztül, és összefoglalja, hogyan alkalmazás rendelkezésre állásának adatok végleges konzisztencia is létrehozható.</span><span class="sxs-lookup"><span data-stu-id="5fa2e-191">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>
