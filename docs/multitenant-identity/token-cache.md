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
# <a name="cache-access-tokens"></a>Hozzáférési jogkivonatok gyorsítótárazása

[![GitHub](../_images/github.png) Mintakód][sample application]

Ez viszonylag drága kaphat OAuth hozzáférési token, mert a HTTP-kérést, hogy a jogkivonat-végpont szükséges. Ezért, fontos gyorsítótár jogkivonatok, amikor csak lehetséges. A [Azure AD Authentication Library] [ ADAL] (ADAL) automatikusan gyorsítótárazza az Azure ad-ből, többek között a frissítési biztonsági jogkivonat kapott jogkivonatok.

Adal-t biztosít egy alapértelmezett tokengyorsítótárral implementációja. Azonban ez a token gyorsítótár natív ügyfélalkalmazások szól, és van **nem** webalkalmazások alkalmas:

* Egy statikus példány, de nem alkalmasak a többszálú futtatásra.
* Mivel az összes felhasználó tokenek az ugyanazon szótár lépnek, nem méretezhető nagyszámú felhasználó.
* Egy farm webkiszolgálók között is megoszthatja.

Ehelyett is meg kell valósítania az ADAL származó egyéni tokengyorsítótárral `TokenCache` osztály azonban egy kiszolgálói környezet számára megfelelő, és a jogkivonatok különböző felhasználók elkülönítését kívánatos szinten biztosítja.

A `TokenCache` osztály tartalmazó jogkivonatok, kibocsátó, erőforrás, ügyfél-azonosító és felhasználó által indexelt tárolja. Egy egyéni tokengyorsítótárral tento slovník kell írni a háttértárban, például a Redis Cache-gyorsítótárhoz.

A Tailspin Surveys-alkalmazás a `DistributedTokenCache` osztálya határozza meg a jogkivonatok gyorsítótárát. Ez a megvalósítás a [IDistributedCache] [ distributed-cache] absztrakciós az ASP.NET Core. Ezzel a módszerrel bármilyen `IDistributedCache` végrehajtására használhatók a háttértárban.

* Alapértelmezés szerint a Surveys alkalmazás használja a Redis Cache-gyorsítótárhoz.
* Egypéldányos webkiszolgálókhoz, használhatja az ASP.NET Core [memórián belüli gyorsítótár][in-memory-cache]. (Ez történik akkor is jó választás az alkalmazást helyileg futtatja a fejlesztés során.)

`DistributedTokenCache` a gyorsítótár adatait tárolja, kulcs-érték párokat a háttértárban. A kulcs a felhasználói azonosító és az ügyfél-Azonosítót, így a háttértárban minden ügyfél-vagy felhasználói egyéni kombinációja külön gyorsítótár adatait tartalmazza.

![Jogkivonatok gyorsítótárát](./images/token-cache.png)

Felhasználó által particionálása a háttértárban. Minden egyes HTTP-kérelem a tokenek számára, hogy a felhasználó a háttértárban olvasása és betölti a `TokenCache` szótárban. A Redis a háttértárban használatos, ha minden kiszolgálópéldány kiszolgálófarm olvasást/írást, az azonos gyorsítótárába, és ez a megközelítés méretezhető sok felhasználó.

## <a name="encrypting-cached-tokens"></a>Gyorsítótárazott jogkivonatok titkosítása

Jogkivonatok olyan bizalmas adatokat, mert, a felhasználó erőforrásokhoz való hozzáférést. (Ezenkívül ellentétben a felhasználó jelszava, nem csak tárolhatja a jogkivonat kivonata.) Ezért rendkívül fontos jogkivonatok védelmét az illetéktelen kezekbe kerüljenek. A Redis-alapú gyorsítótár jelszóval védett, de ha valaki megszerzi a jelszót, azok az sikerült beolvasni minden, a gyorsítótárazott hozzáférési jogkivonatok. Éppen ezért a `DistributedTokenCache` mindent, ami a háttértárban ír titkosítja. Titkosítási történik az ASP.NET Core használatával [adatvédelem] [ data-protection] API-k.

> [!NOTE]
> Ha telepíti az Azure webhelyek, a titkosítási kulcsok biztonsági másolat a hálózati tároláshoz és szinkronizálja az összes gép (lásd: [felügyeleti és-élettartam][key-management]). Alapértelmezés szerint kulcsok nem titkosíthatók az Azure webhelyek fut, de Ön is [engedélyezze a titkosítást, X.509 tanúsítvánnyal][x509-cert-encryption].

## <a name="distributedtokencache-implementation"></a>DistributedTokenCache megvalósítása

A `DistributedTokenCache` származik az ADAL [TokenCache] [ tokencache-class] osztály.

A konstruktor a `DistributedTokenCache` osztály létrehoz egy kulcsot az aktuális felhasználó számára, és a gyorsítótár tölt be a háttértárban:

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

A kulcs jön létre elkülönített változó összefűzésével előállítjuk a felhasználói Azonosítót és az ügyfél-azonosítót. Mindkét megnyílik a jogcímeket a felhasználó megtalálható `ClaimsPrincipal`:

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

A gyorsítótár-adatok betöltése, olvassa el a szerializált blob a háttértárban, majd hívást `TokenCache.Deserialize` alakítható át a blob adatok gyorsítótárazásához.

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

Minden alkalommal, amikor adal-t a gyorsítótár eléréséhez, aktiválódik egy `AfterAccess` esemény. Ha az adatok gyorsítótárazása megváltozott, a `HasStateChanged` tulajdonság igaz értékű. Ebben az esetben a háttértárban a változás tükrözése érdekében frissíteni, és állítsa `HasStateChanged` hamis értékre.

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

TokenCache két olyan eseményeket küld:

* `BeforeWrite`. Nevű azonnal, mielőtt az adal-t a gyorsítótárba ír. Ezzel egy párhuzamossági stratégiát megvalósítása
* `BeforeAccess`. Nevű azonnal, mielőtt az adal-t beolvassa a gyorsítótárból. Itt is betöltheti a gyorsítótár a legújabb verzió beszerzéséhez.

Ebben az esetben azt úgy döntött, hogy nem ezeket az eseményeket két kezelni.

* Egyidejűségi, az utolsó írás a wins. Ez nem jelent gondot, mert jogkivonatok tárolják egymástól függetlenül minden felhasználó + ügyfél, így ütközés csak történnek, ha ugyanaz a felhasználó két egyidejű munkamenetek.
* Az olvashatóság érdekében a gyorsítótár halasztása minden kérelemnél töltünk be. Kérelmek rövid élettartamú. Ha a gyorsítótár az adott lekérdezi módosítanak, a következő kérelmet kiesik az új értéket.

[**Tovább**][client-assertion]

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
