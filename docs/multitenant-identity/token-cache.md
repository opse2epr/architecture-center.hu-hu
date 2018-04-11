---
title: Gyorsítótár számára a hozzáférést jogkivonatok egy több-bérlős alkalmazásban
description: A háttérrendszer webes API hívásához gyorsítótárazási hozzáférési jogkivonatok
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: cffc15686ef9d77fafb40982efdbcd4a79f5aaf2
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2017
---
# <a name="cache-access-tokens"></a><span data-ttu-id="e56f9-103">Hozzáférési jogkivonatok gyorsítótárazása</span><span class="sxs-lookup"><span data-stu-id="e56f9-103">Cache access tokens</span></span>

<span data-ttu-id="e56f9-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="e56f9-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="e56f9-105">Akkor is elérheti az OAuth jogkivonat, mert ahhoz szükséges, hogy a jogkivonat végpontjához HTTP-kérelem viszonylag drága.</span><span class="sxs-lookup"><span data-stu-id="e56f9-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="e56f9-106">Ezért is érdemes gyorsítótár jogkivonatokat, amikor csak lehetséges.</span><span class="sxs-lookup"><span data-stu-id="e56f9-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="e56f9-107">A [Azure AD Authentication Library] [ ADAL] (ADAL) automatikusan gyorsítótárazza az Azure AD, beleértve a frissítési jogkivonatokat kapott jogkivonatok.</span><span class="sxs-lookup"><span data-stu-id="e56f9-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="e56f9-108">Adal-t egy alapértelmezett token gyorsítótár-megvalósítással biztosít.</span><span class="sxs-lookup"><span data-stu-id="e56f9-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="e56f9-109">Azonban ez a token gyorsítótár natív ügyfél-alkalmazásokhoz készült, és **nem** webalkalmazások alkalmas:</span><span class="sxs-lookup"><span data-stu-id="e56f9-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="e56f9-110">Ez nem egy statikus példányt, és nem többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="e56f9-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="e56f9-111">Mivel az összes felhasználó tokenek az ugyanazon szótár kísérhet, nem méretezhető nagy a felhasználók számára.</span><span class="sxs-lookup"><span data-stu-id="e56f9-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="e56f9-112">Egy farm webkiszolgáló között nem lehet megosztani.</span><span class="sxs-lookup"><span data-stu-id="e56f9-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="e56f9-113">Ehelyett inkább egy egyéni jogkivonatok gyorsítótárát, amely az ADAL származik `TokenCache` osztály azonban egy kiszolgálói környezet alkalmas, és biztosít a kívánatos jogkivonatokat a különböző felhasználók elkülönítését.</span><span class="sxs-lookup"><span data-stu-id="e56f9-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="e56f9-114">A `TokenCache` osztály tárol tokenek kibocsátó, az erőforrás, az ügyfél-azonosító és a felhasználói indexelik dictionary.</span><span class="sxs-lookup"><span data-stu-id="e56f9-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="e56f9-115">Egy egyéni jogkivonat gyorsítótára kell írni a ehhez a szótárhoz egy biztonsági, például a Redis gyorsítótár.</span><span class="sxs-lookup"><span data-stu-id="e56f9-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="e56f9-116">A Dejójáték felmérések alkalmazásban a `DistributedTokenCache` osztály megvalósít a jogkivonatok gyorsítótárát.</span><span class="sxs-lookup"><span data-stu-id="e56f9-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="e56f9-117">Ez a megvalósítás a [IDistributedCache] [ distributed-cache] absztrakciós az ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="e56f9-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="e56f9-118">Ezzel a módszerrel bármely `IDistributedCache` egy biztonsági tár végrehajtására használható.</span><span class="sxs-lookup"><span data-stu-id="e56f9-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="e56f9-119">Alapértelmezés szerint a felmérések alkalmazást használja a Redis gyorsítótár.</span><span class="sxs-lookup"><span data-stu-id="e56f9-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="e56f9-120">Egypéldányos webkiszolgáló, használhatja az ASP.NET Core [memórián belüli gyorsítótárral][in-memory-cache].</span><span class="sxs-lookup"><span data-stu-id="e56f9-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="e56f9-121">(Ez történik akkor is helyben fut az alkalmazás fejlesztése során jó választás.)</span><span class="sxs-lookup"><span data-stu-id="e56f9-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="e56f9-122">`DistributedTokenCache`kulcs/érték párok biztonsági tárolójában tárolja a gyorsítótárazott adatokat.</span><span class="sxs-lookup"><span data-stu-id="e56f9-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="e56f9-123">A kulcs a felhasználói Azonosítót és az ügyfél-azonosító, így a biztonsági tár tárolja a felhasználó vagy Windows-ügyfélen egyedi kombinációjához külön gyorsítótárazott adatokat.</span><span class="sxs-lookup"><span data-stu-id="e56f9-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Jogkivonat gyorsítótára](./images/token-cache.png)

<span data-ttu-id="e56f9-125">A biztonsági tár felhasználó particionálva van.</span><span class="sxs-lookup"><span data-stu-id="e56f9-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="e56f9-126">Az egyes HTTP-kérelem a jogkivonatok az adott felhasználó olvasni a biztonsági tár és betölti a `TokenCache` szótárban.</span><span class="sxs-lookup"><span data-stu-id="e56f9-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="e56f9-127">Ha a Redis szolgál a biztonsági tár kiszolgálófarm összes server-példány olvasása/írása az azonos gyorsítótárába, és sok felhasználó méretezi ezt a módszert használja.</span><span class="sxs-lookup"><span data-stu-id="e56f9-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="e56f9-128">Gyorsítótárazott jogkivonatok titkosítása</span><span class="sxs-lookup"><span data-stu-id="e56f9-128">Encrypting cached tokens</span></span>
<span data-ttu-id="e56f9-129">A jogkivonatok bizalmas adatokat, mert azok egy felhasználó erőforrásokhoz való hozzáférés engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="e56f9-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="e56f9-130">(Továbbá eltérően a jelszó, nem csak tárolhatja a token kivonatát.) Ezért kiemelten fontos annak védelme jogkivonatok illetéktelen kezekbe.</span><span class="sxs-lookup"><span data-stu-id="e56f9-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="e56f9-131">A Redis-biztonsági gyorsítótár jelszóval védett, de ha valaki beszerzi a jelszót, azok az lehetett beolvasni a gyorsítótárazott hozzáférési jogkivonatok mindegyikét.</span><span class="sxs-lookup"><span data-stu-id="e56f9-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="e56f9-132">Emiatt a `DistributedTokenCache` minden, ami ír a biztonsági tár titkosítja.</span><span class="sxs-lookup"><span data-stu-id="e56f9-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="e56f9-133">Titkosítási történik, az ASP.NET Core segítségével [adatvédelem] [ data-protection] API-k.</span><span class="sxs-lookup"><span data-stu-id="e56f9-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="e56f9-134">Ha telepíti az Azure-webhelyek, a titkosítási kulcsok biztonsági másolat a hálózati tároláshoz és az összes gép szinkronizálását (lásd: [felügyeleti és-élettartam][key-management]).</span><span class="sxs-lookup"><span data-stu-id="e56f9-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="e56f9-135">Alapértelmezés szerint kulcsok nem titkosítását, ha fut az Azure-webhelyek, azonban úgy is [engedélyezheti a titkosítást, X.509 tanúsítvánnyal][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="e56f9-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>
> 
> 

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="e56f9-136">DistributedTokenCache végrehajtása</span><span class="sxs-lookup"><span data-stu-id="e56f9-136">DistributedTokenCache implementation</span></span>
<span data-ttu-id="e56f9-137">A `DistributedTokenCache` származik-az ADAL [TokenCache] [ tokencache-class] osztály.</span><span class="sxs-lookup"><span data-stu-id="e56f9-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="e56f9-138">A konstruktor a `DistributedTokenCache` osztály egy kulcs az aktuális felhasználó hoz létre, és betölti a gyorsítótár a biztonsági store-ból:</span><span class="sxs-lookup"><span data-stu-id="e56f9-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

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

<span data-ttu-id="e56f9-139">A kulcs hozta létre hozzáfűzésével a felhasználói Azonosítót és az ügyfél-azonosítót.</span><span class="sxs-lookup"><span data-stu-id="e56f9-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="e56f9-140">Mindkét esetben készít a jogcímek a felhasználó megtalálható `ClaimsPrincipal`:</span><span class="sxs-lookup"><span data-stu-id="e56f9-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

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

<span data-ttu-id="e56f9-141">A gyorsítótár adatainak betöltése olvasni a szerializált blob a biztonsági és hívás `TokenCache.Deserialize` gyorsítótárazott adatokat a blob alakítani.</span><span class="sxs-lookup"><span data-stu-id="e56f9-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

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

<span data-ttu-id="e56f9-142">Amikor a gyorsítótár-hozzáféréshez adal-t, akkor következik be egy `AfterAccess` esemény.</span><span class="sxs-lookup"><span data-stu-id="e56f9-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="e56f9-143">Ha a gyorsítótárazott adatokat megváltozott, a `HasStateChanged` tulajdonság értéke igaz.</span><span class="sxs-lookup"><span data-stu-id="e56f9-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="e56f9-144">Ebben az esetben frissíteni a biztonsági tár a változásnak, és utána állítsa be `HasStateChanged` false értékre.</span><span class="sxs-lookup"><span data-stu-id="e56f9-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

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

<span data-ttu-id="e56f9-145">TokenCache küld két esemény:</span><span class="sxs-lookup"><span data-stu-id="e56f9-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="e56f9-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="e56f9-146">`BeforeWrite`.</span></span> <span data-ttu-id="e56f9-147">Hívni a azonnal adal-t a gyorsítótárba ír.</span><span class="sxs-lookup"><span data-stu-id="e56f9-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="e56f9-148">Ezzel a feldolgozási stratégia megvalósításához</span><span class="sxs-lookup"><span data-stu-id="e56f9-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="e56f9-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="e56f9-149">`BeforeAccess`.</span></span> <span data-ttu-id="e56f9-150">Hívni a azonnal ADAL beolvassa a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="e56f9-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="e56f9-151">Itt is betöltheti a gyorsítótárban a legújabb verzióra.</span><span class="sxs-lookup"><span data-stu-id="e56f9-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="e56f9-152">Ebben az esetben döntöttünk nem két eseményekből kezelése érdekében.</span><span class="sxs-lookup"><span data-stu-id="e56f9-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="e56f9-153">Párhuzamossági, az utolsó írás wins.</span><span class="sxs-lookup"><span data-stu-id="e56f9-153">For concurrency, last write wins.</span></span> <span data-ttu-id="e56f9-154">Ez OK, mert, hogy jogkivonatok tárolják egymástól függetlenül minden felhasználót + ügyfél, így ütközés csak akkor fordulhat elő, ha ugyanazon felhasználó kellett két egyidejű munkamenetek.</span><span class="sxs-lookup"><span data-stu-id="e56f9-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="e56f9-155">Olvasását, a gyorsítótár halasztása minden kérelemnél betölteni azt.</span><span class="sxs-lookup"><span data-stu-id="e56f9-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="e56f9-156">Kérések rövid élettartamú használata.</span><span class="sxs-lookup"><span data-stu-id="e56f9-156">Requests are short lived.</span></span> <span data-ttu-id="e56f9-157">Adott időpont módosításakor a gyorsítótár lekérdezi a következő kérés felveszi az új érték.</span><span class="sxs-lookup"><span data-stu-id="e56f9-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="e56f9-158">[**Tovább**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="e56f9-158">[**Next**][client-assertion]</span></span>

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
