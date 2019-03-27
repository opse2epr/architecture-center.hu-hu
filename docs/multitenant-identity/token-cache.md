---
title: Gyorsítótár hozzáférési jogkivonatai egy több-bérlős alkalmazásban
description: A háttérbeli webes API-k hívásához hozzáférési jogkivonatok gyorsítótárazása.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: 608a244eb0842da33af32457418d837b7f0a75c2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298422"
---
# <a name="cache-access-tokens"></a><span data-ttu-id="ee158-103">Hozzáférési jogkivonatok gyorsítótárazása</span><span class="sxs-lookup"><span data-stu-id="ee158-103">Cache access tokens</span></span>

<span data-ttu-id="ee158-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="ee158-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="ee158-105">Ez viszonylag drága kaphat OAuth hozzáférési token, mert a HTTP-kérést, hogy a jogkivonat-végpont szükséges.</span><span class="sxs-lookup"><span data-stu-id="ee158-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="ee158-106">Ezért, fontos gyorsítótár jogkivonatok, amikor csak lehetséges.</span><span class="sxs-lookup"><span data-stu-id="ee158-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="ee158-107">A [Azure AD Authentication Library] [ ADAL] (ADAL) automatikusan gyorsítótárazza az Azure ad-ből, többek között a frissítési biztonsági jogkivonat kapott jogkivonatok.</span><span class="sxs-lookup"><span data-stu-id="ee158-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="ee158-108">Adal-t biztosít egy alapértelmezett tokengyorsítótárral implementációja.</span><span class="sxs-lookup"><span data-stu-id="ee158-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="ee158-109">Azonban ez a token gyorsítótár natív ügyfélalkalmazások szól, és van **nem** webalkalmazások alkalmas:</span><span class="sxs-lookup"><span data-stu-id="ee158-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="ee158-110">Egy statikus példány, de nem alkalmasak a többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="ee158-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="ee158-111">Mivel az összes felhasználó tokenek az ugyanazon szótár lépnek, nem méretezhető nagyszámú felhasználó.</span><span class="sxs-lookup"><span data-stu-id="ee158-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="ee158-112">Egy farm webkiszolgálók között is megoszthatja.</span><span class="sxs-lookup"><span data-stu-id="ee158-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="ee158-113">Ehelyett is meg kell valósítania az ADAL származó egyéni tokengyorsítótárral `TokenCache` osztály azonban egy kiszolgálói környezet számára megfelelő, és a jogkivonatok különböző felhasználók elkülönítését kívánatos szinten biztosítja.</span><span class="sxs-lookup"><span data-stu-id="ee158-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="ee158-114">A `TokenCache` osztály tartalmazó jogkivonatok, kibocsátó, erőforrás, ügyfél-azonosító és felhasználó által indexelt tárolja.</span><span class="sxs-lookup"><span data-stu-id="ee158-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="ee158-115">Egy egyéni tokengyorsítótárral tento slovník kell írni a háttértárban, például a Redis Cache-gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="ee158-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="ee158-116">A Tailspin Surveys-alkalmazás a `DistributedTokenCache` osztálya határozza meg a jogkivonatok gyorsítótárát.</span><span class="sxs-lookup"><span data-stu-id="ee158-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="ee158-117">Ez a megvalósítás a [IDistributedCache] [ distributed-cache] absztrakciós az ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="ee158-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="ee158-118">Ezzel a módszerrel bármilyen `IDistributedCache` végrehajtására használhatók a háttértárban.</span><span class="sxs-lookup"><span data-stu-id="ee158-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="ee158-119">Alapértelmezés szerint a Surveys alkalmazás használja a Redis Cache-gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="ee158-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="ee158-120">Egypéldányos webkiszolgálókhoz, használhatja az ASP.NET Core [memórián belüli gyorsítótár][in-memory-cache].</span><span class="sxs-lookup"><span data-stu-id="ee158-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="ee158-121">(Ez történik akkor is jó választás az alkalmazást helyileg futtatja a fejlesztés során.)</span><span class="sxs-lookup"><span data-stu-id="ee158-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="ee158-122">`DistributedTokenCache` a gyorsítótár adatait tárolja, kulcs-érték párokat a háttértárban.</span><span class="sxs-lookup"><span data-stu-id="ee158-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="ee158-123">A kulcs a felhasználói azonosító és az ügyfél-Azonosítót, így a háttértárban minden ügyfél-vagy felhasználói egyéni kombinációja külön gyorsítótár adatait tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="ee158-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Jogkivonatok gyorsítótárát](./images/token-cache.png)

<span data-ttu-id="ee158-125">Felhasználó által particionálása a háttértárban.</span><span class="sxs-lookup"><span data-stu-id="ee158-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="ee158-126">Minden egyes HTTP-kérelem a tokenek számára, hogy a felhasználó a háttértárban olvasása és betölti a `TokenCache` szótárban.</span><span class="sxs-lookup"><span data-stu-id="ee158-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="ee158-127">A Redis a háttértárban használatos, ha minden kiszolgálópéldány kiszolgálófarm olvasást/írást, az azonos gyorsítótárába, és ez a megközelítés méretezhető sok felhasználó.</span><span class="sxs-lookup"><span data-stu-id="ee158-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="ee158-128">Gyorsítótárazott jogkivonatok titkosítása</span><span class="sxs-lookup"><span data-stu-id="ee158-128">Encrypting cached tokens</span></span>

<span data-ttu-id="ee158-129">Jogkivonatok olyan bizalmas adatokat, mert, a felhasználó erőforrásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="ee158-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="ee158-130">(Ezenkívül ellentétben a felhasználó jelszava, nem csak tárolhatja a jogkivonat kivonata.) Ezért rendkívül fontos jogkivonatok védelmét az illetéktelen kezekbe kerüljenek.</span><span class="sxs-lookup"><span data-stu-id="ee158-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="ee158-131">A Redis-alapú gyorsítótár jelszóval védett, de ha valaki megszerzi a jelszót, azok az sikerült beolvasni minden, a gyorsítótárazott hozzáférési jogkivonatok.</span><span class="sxs-lookup"><span data-stu-id="ee158-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="ee158-132">Éppen ezért a `DistributedTokenCache` mindent, ami a háttértárban ír titkosítja.</span><span class="sxs-lookup"><span data-stu-id="ee158-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="ee158-133">Titkosítási történik az ASP.NET Core használatával [adatvédelem] [ data-protection] API-k.</span><span class="sxs-lookup"><span data-stu-id="ee158-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="ee158-134">Ha telepíti az Azure webhelyek, a titkosítási kulcsok biztonsági másolat a hálózati tároláshoz és szinkronizálja az összes gép (lásd: [felügyeleti és-élettartam][key-management]).</span><span class="sxs-lookup"><span data-stu-id="ee158-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="ee158-135">Alapértelmezés szerint kulcsok nem titkosíthatók az Azure webhelyek fut, de Ön is [engedélyezze a titkosítást, X.509 tanúsítvánnyal][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="ee158-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="ee158-136">DistributedTokenCache megvalósítása</span><span class="sxs-lookup"><span data-stu-id="ee158-136">DistributedTokenCache implementation</span></span>

<span data-ttu-id="ee158-137">A `DistributedTokenCache` származik az ADAL [TokenCache] [ tokencache-class] osztály.</span><span class="sxs-lookup"><span data-stu-id="ee158-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="ee158-138">A konstruktor a `DistributedTokenCache` osztály létrehoz egy kulcsot az aktuális felhasználó számára, és a gyorsítótár tölt be a háttértárban:</span><span class="sxs-lookup"><span data-stu-id="ee158-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="ee158-139">A kulcs jön létre elkülönített változó összefűzésével előállítjuk a felhasználói Azonosítót és az ügyfél-azonosítót.</span><span class="sxs-lookup"><span data-stu-id="ee158-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="ee158-140">Mindkét megnyílik a jogcímeket a felhasználó megtalálható `ClaimsPrincipal`:</span><span class="sxs-lookup"><span data-stu-id="ee158-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="ee158-141">A gyorsítótár-adatok betöltése, olvassa el a szerializált blob a háttértárban, majd hívást `TokenCache.Deserialize` alakítható át a blob adatok gyorsítótárazásához.</span><span class="sxs-lookup"><span data-stu-id="ee158-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="ee158-142">Minden alkalommal, amikor adal-t a gyorsítótár eléréséhez, aktiválódik egy `AfterAccess` esemény.</span><span class="sxs-lookup"><span data-stu-id="ee158-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="ee158-143">Ha az adatok gyorsítótárazása megváltozott, a `HasStateChanged` tulajdonság igaz értékű.</span><span class="sxs-lookup"><span data-stu-id="ee158-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="ee158-144">Ebben az esetben a háttértárban a változás tükrözése érdekében frissíteni, és állítsa `HasStateChanged` hamis értékre.</span><span class="sxs-lookup"><span data-stu-id="ee158-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="ee158-145">TokenCache két olyan eseményeket küld:</span><span class="sxs-lookup"><span data-stu-id="ee158-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="ee158-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="ee158-146">`BeforeWrite`.</span></span> <span data-ttu-id="ee158-147">Nevű azonnal, mielőtt az adal-t a gyorsítótárba ír.</span><span class="sxs-lookup"><span data-stu-id="ee158-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="ee158-148">Ezzel egy párhuzamossági stratégiát megvalósítása</span><span class="sxs-lookup"><span data-stu-id="ee158-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="ee158-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="ee158-149">`BeforeAccess`.</span></span> <span data-ttu-id="ee158-150">Nevű azonnal, mielőtt az adal-t beolvassa a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="ee158-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="ee158-151">Itt is betöltheti a gyorsítótár a legújabb verzió beszerzéséhez.</span><span class="sxs-lookup"><span data-stu-id="ee158-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="ee158-152">Ebben az esetben azt úgy döntött, hogy nem ezeket az eseményeket két kezelni.</span><span class="sxs-lookup"><span data-stu-id="ee158-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="ee158-153">Egyidejűségi, az utolsó írás a wins.</span><span class="sxs-lookup"><span data-stu-id="ee158-153">For concurrency, last write wins.</span></span> <span data-ttu-id="ee158-154">Ez nem jelent gondot, mert jogkivonatok tárolják egymástól függetlenül minden felhasználó + ügyfél, így ütközés csak történnek, ha ugyanaz a felhasználó két egyidejű munkamenetek.</span><span class="sxs-lookup"><span data-stu-id="ee158-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="ee158-155">Az olvashatóság érdekében a gyorsítótár halasztása minden kérelemnél töltünk be.</span><span class="sxs-lookup"><span data-stu-id="ee158-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="ee158-156">Kérelmek rövid élettartamú.</span><span class="sxs-lookup"><span data-stu-id="ee158-156">Requests are short lived.</span></span> <span data-ttu-id="ee158-157">Ha a gyorsítótár az adott lekérdezi módosítanak, a következő kérelmet kiesik az új értéket.</span><span class="sxs-lookup"><span data-stu-id="ee158-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="ee158-158">[**Tovább**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="ee158-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
