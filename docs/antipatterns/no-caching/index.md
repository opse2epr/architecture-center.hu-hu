---
title: Nincs gyorsítótárazás – kizárási minta
description: Ugyanazon adatok ismételt beolvasása csökkentheti a teljesítményt és a méretezhetőséget.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: f94a9f3f9166e87949a0e60af818cd89796dc3e2
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428947"
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="80cd0-103">Nincs gyorsítótárazás – kizárási minta</span><span class="sxs-lookup"><span data-stu-id="80cd0-103">No Caching antipattern</span></span>

<span data-ttu-id="80cd0-104">Ha egy számos egyidejű kérést kezelő felhőalkalmazásban ismételten beolvassa ugyanazokat az adatokat, csökkenhet a teljesítmény és a méretezhetőség.</span><span class="sxs-lookup"><span data-stu-id="80cd0-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="80cd0-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="80cd0-105">Problem description</span></span>

<span data-ttu-id="80cd0-106">Ha az adatok nincsenek gyorsítótárazva, az számos nem kívánt viselkedést okozhat, például:</span><span class="sxs-lookup"><span data-stu-id="80cd0-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="80cd0-107">Ugyanazon információ ismételt beolvasása olyan erőforrásokból, amelyeknek az elérése költséges (I/O vagy késés szempontjából).</span><span class="sxs-lookup"><span data-stu-id="80cd0-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="80cd0-108">Ugyanazon objektumok vagy adatstruktúrák ismételt felépítése a különböző kérések eredményeként.</span><span class="sxs-lookup"><span data-stu-id="80cd0-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="80cd0-109">Túl sok hívás kezdeményezése egy távoli, kvótával rendelkező szolgáltatás irányába, amely egy bizonyos korlátot túllépve szabályozza az ügyfeleket.</span><span class="sxs-lookup"><span data-stu-id="80cd0-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="80cd0-110">Ezek a problémák hosszú válaszidőkhöz, a nagyobb adattárbeli versengéshez és gyenge méretezhetőséghez vezetnek.</span><span class="sxs-lookup"><span data-stu-id="80cd0-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="80cd0-111">A következő példa az Entity Framework használatával csatlakozik egy adatbázishoz.</span><span class="sxs-lookup"><span data-stu-id="80cd0-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="80cd0-112">Minden ügyfélkérés az adatbázis hívását eredményezi, még akkor is, ha több kérés is ugyanazt az adatot olvassa be.</span><span class="sxs-lookup"><span data-stu-id="80cd0-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="80cd0-113">Az ismételt kérések költsége – az I/O terhelését és az adathozzáférési díjakat illetően – gyorsan megnövekedhet.</span><span class="sxs-lookup"><span data-stu-id="80cd0-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

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

<span data-ttu-id="80cd0-114">A teljes kódmintát [itt][sample-app] találja.</span><span class="sxs-lookup"><span data-stu-id="80cd0-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="80cd0-115">A kizárási minta jellemzően az alábbi okokból következhet be:</span><span class="sxs-lookup"><span data-stu-id="80cd0-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="80cd0-116">A gyorsítótár használatának elkerülése egyszerűbben megvalósítható, és kis terhelés esetén jól működik.</span><span class="sxs-lookup"><span data-stu-id="80cd0-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="80cd0-117">A gyorsítótárazás bonyolultabbá teszi a kódot.</span><span class="sxs-lookup"><span data-stu-id="80cd0-117">Caching makes the code more complicated.</span></span> 
- <span data-ttu-id="80cd0-118">A gyorsítótár használatának előnyei és hátrányai nem teljesen világosak.</span><span class="sxs-lookup"><span data-stu-id="80cd0-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="80cd0-119">A gyorsítótárazott adatok frissességének és pontosságának fenntartásának költségei aggályokat vetnek fel.</span><span class="sxs-lookup"><span data-stu-id="80cd0-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="80cd0-120">Az alkalmazást olyan helyszíni rendszerről migrálták, amelyen a hálózati késés nem jelentett problémát, a rendszer pedig drága, nagy teljesítményű hardveren futott, ezért a gyorsítótárazás lehetőségét nem vették figyelembe az eredeti kialakítás során.</span><span class="sxs-lookup"><span data-stu-id="80cd0-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="80cd0-121">A fejlesztők nem tudnak róla, hogy a gyorsítótárazás lehetséges egy adott helyzetben.</span><span class="sxs-lookup"><span data-stu-id="80cd0-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="80cd0-122">Előfordulhat például, hogy a fejlesztők nem gondolnak az ETagek használatára egy webes API megvalósításakor.</span><span class="sxs-lookup"><span data-stu-id="80cd0-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="80cd0-123">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="80cd0-123">How to fix the problem</span></span>

<span data-ttu-id="80cd0-124">A legnépszerűbb gyorsítótárazási stratégia az *igény szerinti* vagy *gyorsítótár-feltöltő* módszer.</span><span class="sxs-lookup"><span data-stu-id="80cd0-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="80cd0-125">Olvasáskor az alkalmazás megpróbálja beolvasni az adatokat a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="80cd0-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="80cd0-126">Ha nincsenek a gyorsítótárban, az alkalmazás lekéri az adatokat az adatforrásból, majd hozzáadja őket a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="80cd0-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="80cd0-127">Íráskor az alkalmazás közvetlenül az adatforrásba írja a módosítást, a régi értéket pedig eltávolítja a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="80cd0-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="80cd0-128">Amikor legközelebb szükség van rá, a rendszer lekéri, majd hozzáadja a gyorsítótárhoz az adatokat.</span><span class="sxs-lookup"><span data-stu-id="80cd0-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="80cd0-129">Ez a módszer a gyakran módosuló adatokhoz alkalmas.</span><span class="sxs-lookup"><span data-stu-id="80cd0-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="80cd0-130">Itt van az előző példa, a [Cache-Aside][cache-aside] minta használatára frissítve.</span><span class="sxs-lookup"><span data-stu-id="80cd0-130">Here is the previous example updated to use the [Cache-Aside][cache-aside] pattern.</span></span>  

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

<span data-ttu-id="80cd0-131">A `GetAsync` metódus ezúttal nem közvetlenül az adatbázist, hanem a `CacheService` osztályt hívja meg.</span><span class="sxs-lookup"><span data-stu-id="80cd0-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="80cd0-132">A `CacheService` osztály először megkísérli lekérni az elemet az Azure Redis Cache-ből.</span><span class="sxs-lookup"><span data-stu-id="80cd0-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="80cd0-133">Ha az érték nem található a Redis Cache-ben, a `CacheService` meghív egy lambda függvényt, amelyet a hívó adott át.</span><span class="sxs-lookup"><span data-stu-id="80cd0-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="80cd0-134">A lambda függvény felelős az adat adatbázisból történő beolvasásáért.</span><span class="sxs-lookup"><span data-stu-id="80cd0-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="80cd0-135">Ez a megvalósítás leválasztja az adattárat az adott gyorsítótárazási megoldásról, valamint a `CacheService`-t elemet az adatbázisról.</span><span class="sxs-lookup"><span data-stu-id="80cd0-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span> 

## <a name="considerations"></a><span data-ttu-id="80cd0-136">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="80cd0-136">Considerations</span></span>

- <span data-ttu-id="80cd0-137">Ha a gyorsítótár nem érhető el, például átmeneti hiba miatt, ne jelezzen hibát az ügyfélnek.</span><span class="sxs-lookup"><span data-stu-id="80cd0-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="80cd0-138">Ehelyett olvassa be az adatot az eredeti adatforrásból.</span><span class="sxs-lookup"><span data-stu-id="80cd0-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="80cd0-139">Vegye azonban figyelembe, hogy a gyorsítótár helyreállítása közben az eredeti adattárat eláraszthatják a kérések, ami időtúllépéseket és sikertelen kapcsolatokat eredményez.</span><span class="sxs-lookup"><span data-stu-id="80cd0-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="80cd0-140">(Eleve ez az egyik fő ok a gyorsítótár használatára.) Az adatforrás túlterhelésének elkerülése érdekében használjon az [Áramköri megszakító mintához][circuit-breaker] hasonló technikát.</span><span class="sxs-lookup"><span data-stu-id="80cd0-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="80cd0-141">A nem statikus adatokat gyorsítótárazó alkalmazásokat úgy kell megtervezni, hogy támogassák a későbbi konzisztenciát.</span><span class="sxs-lookup"><span data-stu-id="80cd0-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="80cd0-142">A webes API-k esetében támogatható az ügyféloldali gyorsítótárazás, ha elhelyez egy gyorsítótár-vezérlő fejlécet a kérés- és válaszüzenetekben, valamint ETageket használ az objektumok verzióinak azonosítására.</span><span class="sxs-lookup"><span data-stu-id="80cd0-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="80cd0-143">További információ: [API-megvalósítás][api-implementation].</span><span class="sxs-lookup"><span data-stu-id="80cd0-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="80cd0-144">Nem szükséges egész entitásokat gyorsítótáraznia.</span><span class="sxs-lookup"><span data-stu-id="80cd0-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="80cd0-145">Ha egy entitás legnagyobbrészt statikus, de egy kis része gyakran változik, akkor a statikus elemeket gyorsítótárazza, a dinamikus elemeket pedig kérje le az adatforrásból.</span><span class="sxs-lookup"><span data-stu-id="80cd0-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="80cd0-146">Ez a módszer segíthet csökkenteni az adatforráson végzett I/O-műveletek mennyiségét.</span><span class="sxs-lookup"><span data-stu-id="80cd0-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="80cd0-147">Bizonyos esetekben hasznos lehet gyorsítótárazni a rövid életű ideiglenes adatokat is.</span><span class="sxs-lookup"><span data-stu-id="80cd0-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="80cd0-148">Vegyünk például egy olyan eszközt, amely folyamatosan állapotfrissítéseket küld.</span><span class="sxs-lookup"><span data-stu-id="80cd0-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="80cd0-149">Érdemes lehet a gyorsítótárba helyezni a beérkező információt, és egyáltalán nem írni állandó tárolóba.</span><span class="sxs-lookup"><span data-stu-id="80cd0-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>  

- <span data-ttu-id="80cd0-150">Az adatok elavulásának megakadályozása érdekében számos gyorsítótárazási megoldás támogatja a lejárati idő konfigurálását. A rendszer így automatikusan eltávolítja az adatokat a gyorsítótárból a megadott idő elteltével.</span><span class="sxs-lookup"><span data-stu-id="80cd0-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="80cd0-151">A lejárati időt a forgatókönyvnek megfelelően érdemes beállítani.</span><span class="sxs-lookup"><span data-stu-id="80cd0-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="80cd0-152">A nagymértékben statikus adatok hosszabb ideig a gyorsítótárban maradhatnak, mint az ideiglenes, gyorsan elavuló adatok.</span><span class="sxs-lookup"><span data-stu-id="80cd0-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="80cd0-153">Ha a gyorsítótárazási megoldás nem biztosít beépített lejárati időt, akkor szükséges lehet egy olyan háttérfolyamat megvalósítása, amely időnként kiüríti a gyorsítótárat, hogy ne tudjon korlátlanul nőni.</span><span class="sxs-lookup"><span data-stu-id="80cd0-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span> 

- <span data-ttu-id="80cd0-154">A gyorsítótárazás a külső adatforrásból érkező adatok tárolása mellett a komplex számítások eredményeinek mentésére is alkalmas.</span><span class="sxs-lookup"><span data-stu-id="80cd0-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="80cd0-155">Előbb azonban meg kell határozni, hogy az alkalmazás valóban a processzorhoz van-e kötve.</span><span class="sxs-lookup"><span data-stu-id="80cd0-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="80cd0-156">Hasznos lehet előkészíteni a gyorsítótárat az alkalmazás indításakor.</span><span class="sxs-lookup"><span data-stu-id="80cd0-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="80cd0-157">Töltse fel a gyorsítótárat azokkal az adatokkal, amelyekre a legnagyobb valószínűséggel lesz szükség.</span><span class="sxs-lookup"><span data-stu-id="80cd0-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="80cd0-158">Minden esetben olyan kialakítást válasszon, amely észleli a sikeres és sikertelen gyorsítótárbeli kereséseket.</span><span class="sxs-lookup"><span data-stu-id="80cd0-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="80cd0-159">Ezekből az információkból gyorsítótárazási szabályokat alakíthat ki, amelyek meghatározzák például azt, hogy mely adatokat kell gyorsítótárazni, illetve meddig kell megtartani őket, mielőtt elavulnak.</span><span class="sxs-lookup"><span data-stu-id="80cd0-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="80cd0-160">Ha a gyorsítótárazás hiánya szűk keresztmetszetet jelent, akkor a hozzáadása olyan mértékben növelheti a kérések számát, hogy a webes kezelőfelület túlterhelését okozhatja.</span><span class="sxs-lookup"><span data-stu-id="80cd0-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="80cd0-161">Előfordulhat, hogy az ügyfelek HTTP 503 (A szolgáltatás nem érhető el) hibákat kapnak.</span><span class="sxs-lookup"><span data-stu-id="80cd0-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="80cd0-162">Ez arra utal, hogy a kezelőfelület horizontális felskálázást igényel.</span><span class="sxs-lookup"><span data-stu-id="80cd0-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="80cd0-163">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="80cd0-163">How to detect the problem</span></span>

<span data-ttu-id="80cd0-164">A következő lépésekkel határozhatja meg, hogy a gyorsítótárazás hiánya okoz-e teljesítményproblémákat:</span><span class="sxs-lookup"><span data-stu-id="80cd0-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="80cd0-165">Tekintse át az alkalmazás kialakítását.</span><span class="sxs-lookup"><span data-stu-id="80cd0-165">Review the application design.</span></span> <span data-ttu-id="80cd0-166">Készítsen leltárt az alkalmazás által használt összes adattárolóról.</span><span class="sxs-lookup"><span data-stu-id="80cd0-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="80cd0-167">Mindegyik esetében állapítsa meg, hogy az alkalmazás használ-e gyorsítótárat.</span><span class="sxs-lookup"><span data-stu-id="80cd0-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="80cd0-168">Ha lehetséges, állapítsa meg az adatok változásának gyakoriságát.</span><span class="sxs-lookup"><span data-stu-id="80cd0-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="80cd0-169">Elsőként olyan adatokat célszerű gyorsítótárazni, amely ritkán változnak, valamint a gyakran olvasott statikus referenciaadatokat.</span><span class="sxs-lookup"><span data-stu-id="80cd0-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span> 

2. <span data-ttu-id="80cd0-170">Alakítsa ki az alkalmazást, és monitorozza az élő rendszert annak megállapításához, hogy az alkalmazás milyen gyakorisággal kér le adatokat vagy számít ki információkat.</span><span class="sxs-lookup"><span data-stu-id="80cd0-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="80cd0-171">Készítse el az alkalmazás profilját tesztkörnyezetben az adathozzáférési műveletekhez vagy gyakran elvégzett számításokhoz kapcsolódó terhelés részletes metrikáinak rögzítéséhez.</span><span class="sxs-lookup"><span data-stu-id="80cd0-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="80cd0-172">Végezzen terhelési tesztet tesztkörnyezetben annak megállapításához, hogy a rendszer hogyan reagál átlagos, illetve nagy terhelés alatt.</span><span class="sxs-lookup"><span data-stu-id="80cd0-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="80cd0-173">A terhelési tesztnek az éles környezetben megfigyelt, valósághű számítási feladatok mellett megfigyelt adat-hozzáférési mintákat.</span><span class="sxs-lookup"><span data-stu-id="80cd0-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span> 

5. <span data-ttu-id="80cd0-174">Vizsgálja meg a teszt alapját képező adattárolók adat-hozzáférési statisztikáit, és ellenőrizze, milyen gyakran ismétlődnek az ugyanazon adatokra vonatkozó kérések.</span><span class="sxs-lookup"><span data-stu-id="80cd0-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span> 


## <a name="example-diagnosis"></a><span data-ttu-id="80cd0-175">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="80cd0-175">Example diagnosis</span></span>

<span data-ttu-id="80cd0-176">Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.</span><span class="sxs-lookup"><span data-stu-id="80cd0-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="80cd0-177">Az alkalmazás kialakítása és az élő rendszer monitorozása</span><span class="sxs-lookup"><span data-stu-id="80cd0-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="80cd0-178">Alakítsa ki és monitorozza az alkalmazást, hogy információkhoz jusson a felhasználók által az alkalmazás működése közben küldött kérésekről.</span><span class="sxs-lookup"><span data-stu-id="80cd0-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span> 

<span data-ttu-id="80cd0-179">A következő képen a [New Relic][NewRelic] által egy terhelési teszt során rögzített monitorozási adatok láthatók.</span><span class="sxs-lookup"><span data-stu-id="80cd0-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="80cd0-180">Ebben az esetben az egyetlen végrehajtott HTTP GET művelet a következő: `Person/GetAsync`.</span><span class="sxs-lookup"><span data-stu-id="80cd0-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="80cd0-181">Éles környezetben azonban, ha ismeri az egyes kérések relatív gyakoriságát, információhoz juthat arról, hogy mely erőforrásokat érdemes gyorsítótárazni.</span><span class="sxs-lookup"><span data-stu-id="80cd0-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![A New Relic által ábrázolt kiszolgálói kérések a CachingDemo alkalmazáshoz][NewRelic-server-requests]

<span data-ttu-id="80cd0-183">Ha mélyebb elemzésre van szükség, használhat profilkészítőt az alacsony szintű teljesítményadatok tesztkörnyezetben (nem üzemi környezetben) történő rögzítéséhez.</span><span class="sxs-lookup"><span data-stu-id="80cd0-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="80cd0-184">Különösen figyeljen az I/O-kérések mennyiségére, valamint a memória és a processzor kihasználtságára vonatkozó metrikákra.</span><span class="sxs-lookup"><span data-stu-id="80cd0-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="80cd0-185">Ezek a metrikák egy adattárolóra vagy szolgáltatásra irányuló, nagy mennyiségű kérést mutathatnak, vagy ismételt folyamatokat, amelyek ugyanazon számítást végzik el.</span><span class="sxs-lookup"><span data-stu-id="80cd0-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span> 

### <a name="load-test-the-application"></a><span data-ttu-id="80cd0-186">Az alkalmazás terheléstesztje</span><span class="sxs-lookup"><span data-stu-id="80cd0-186">Load test the application</span></span>

<span data-ttu-id="80cd0-187">A következő diagram a mintaalkalmazás terhelési tesztjének eredményeit ábrázolja.</span><span class="sxs-lookup"><span data-stu-id="80cd0-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="80cd0-188">A terhelési teszt legfeljebb 800 felhasználó által egyidejűleg végzett, jellemző művelettípusok lépéses terhelését szimulálja.</span><span class="sxs-lookup"><span data-stu-id="80cd0-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span> 

![A teljesítményterhelési teszt eredményei a gyorsítótárazás nélküli forgatókönyv esetében][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="80cd0-190">A másodpercenként végzett sikeres tesztek száma eléri a maximumot, emiatt pedig a további kérések lelassulnak.</span><span class="sxs-lookup"><span data-stu-id="80cd0-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="80cd0-191">Az átlagos tesztelési idő a terheléssel arányosan, egyenletesen növekszik.</span><span class="sxs-lookup"><span data-stu-id="80cd0-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="80cd0-192">A válaszidőszintek nem változnak, ahogy a felhasználói terhelés eléri a maximumot.</span><span class="sxs-lookup"><span data-stu-id="80cd0-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="80cd0-193">Az adathozzáférési statisztikák vizsgálata</span><span class="sxs-lookup"><span data-stu-id="80cd0-193">Examine data access statistics</span></span>

<span data-ttu-id="80cd0-194">Az adathozzáférési statisztikák és az adattároló által szolgáltatott egyéb információk hasznosak lehetnek például annak megállapításában, hogy melyik lekérdezések ismétlődnek a leggyakrabban.</span><span class="sxs-lookup"><span data-stu-id="80cd0-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="80cd0-195">A Microsoft SQL Serverben például a `sys.dm_exec_query_stats` felügyeleti nézet statisztikai információkkal szolgál a legutóbb végrehajtott lekérdezésekről.</span><span class="sxs-lookup"><span data-stu-id="80cd0-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="80cd0-196">Az egyes lekérdezések szövege elérhető a `sys.dm_exec-query_plan` nézetben.</span><span class="sxs-lookup"><span data-stu-id="80cd0-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="80cd0-197">Egy erre alkalmas eszköz, például az SQL Server Management Studio használatával futtathatja a következő SQL-lekérdezést annak megállapításához, milyen gyakran végez a rendszer lekérdezéseket.</span><span class="sxs-lookup"><span data-stu-id="80cd0-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="80cd0-198">Az eredmények között látható `UseCount` oszlop jelzi, hogy az egyes lekérdezések milyen gyakran futnak.</span><span class="sxs-lookup"><span data-stu-id="80cd0-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="80cd0-199">A következő képen látható, hogy a harmadik lekérdezés több mint 250 000 alkalommal futott, vagyis lényegesen többször, mint bármely más lekérdezés.</span><span class="sxs-lookup"><span data-stu-id="80cd0-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![A SQL Server Management Server dinamikus felügyeleti nézeteinek lekérdezési eredményei][Dynamic-Management-Views]

<span data-ttu-id="80cd0-201">Itt látható az SQL-lekérdezés, amely ennyi adatbázis-kérés benyújtását eredményezi:</span><span class="sxs-lookup"><span data-stu-id="80cd0-201">Here is the SQL query that is causing so many database requests:</span></span> 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="80cd0-202">Ez az Entity Framework által a korábban bemutatott `GetByIdAsync` metódusban létrehozott lekérdezés.</span><span class="sxs-lookup"><span data-stu-id="80cd0-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="80cd0-203">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="80cd0-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="80cd0-204">A gyorsítótár megvalósítása után ismételje meg a terhelési teszteket, majd hasonlítsa össze az eredményeket a korábbi, gyorsítótár nélküli terhelési tesztekével.</span><span class="sxs-lookup"><span data-stu-id="80cd0-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="80cd0-205">Itt láthatók a gyorsítótár mintaalkalmazáshoz való hozzáadása után végzett teszt eredményei.</span><span class="sxs-lookup"><span data-stu-id="80cd0-205">Here are the load test results after adding a cache to the sample application.</span></span>

![A teljesítményterhelési teszt eredményei a gyorsítótáras forgatókönyv esetében][Performance-Load-Test-Results-Cached]

<span data-ttu-id="80cd0-207">A sikeres tesztek mennyisége így is tetőzik, de magasabb felhasználói terhelésnél.</span><span class="sxs-lookup"><span data-stu-id="80cd0-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="80cd0-208">A kérések mennyisége e terhelés esetében a korábbinál sokkal magasabb.</span><span class="sxs-lookup"><span data-stu-id="80cd0-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="80cd0-209">Az átlagos tesztelési idő továbbra is nő a terheléssel, viszont a maximális válaszidő 0,05 ezredmásodperc. A korábban mért 1 ezredmásodperchez képest ez &mdash;20-szoros&times; javulást jelent.</span><span class="sxs-lookup"><span data-stu-id="80cd0-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span> 

## <a name="related-resources"></a><span data-ttu-id="80cd0-210">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="80cd0-210">Related resources</span></span>

- <span data-ttu-id="80cd0-211">[API-implementáció – Ajánlott eljárások][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="80cd0-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="80cd0-212">[Gyorsítótár-feltöltési minta][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="80cd0-212">[Cache-Aside Pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="80cd0-213">[Gyorsítótárazás – Ajánlott eljárások][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="80cd0-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="80cd0-214">[Áramköri megszakítási minta][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="80cd0-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

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