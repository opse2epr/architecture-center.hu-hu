---
title: "Gyorsítótár számára a hozzáférést jogkivonatok egy több-bérlős alkalmazásban"
description: "A háttérrendszer webes API hívásához gyorsítótárazási hozzáférési jogkivonatok"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: ed0def244f3229bbd3fdd0976574d13a345f9aee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="cache-access-tokens"></a>Gyorsítótár hozzáférési jogkivonatok

[![GitHub](../_images/github.png) példakód][sample application]

Akkor is elérheti az OAuth jogkivonat, mert ahhoz szükséges, hogy a jogkivonat végpontjához HTTP-kérelem viszonylag drága. Ezért is érdemes gyorsítótár jogkivonatokat, amikor csak lehetséges. A [Azure AD Authentication Library] [ ADAL] (ADAL) automatikusan gyorsítótárazza az Azure AD, beleértve a frissítési jogkivonatokat kapott jogkivonatok.

Adal-t egy alapértelmezett token gyorsítótár-megvalósítással biztosít. Azonban ez a token gyorsítótár natív ügyfél-alkalmazásokhoz készült, és *nem* webalkalmazások alkalmas:

* Ez nem egy statikus példányt, és nem többszálú futtatásra.
* Mivel az összes felhasználó tokenek az ugyanazon szótár kísérhet, nem méretezhető nagy a felhasználók számára.
* Egy farm webkiszolgáló között nem lehet megosztani.

Ehelyett inkább egy egyéni jogkivonatok gyorsítótárát, amely az ADAL származik `TokenCache` osztály azonban egy kiszolgálói környezet alkalmas, és biztosít a kívánatos jogkivonatokat a különböző felhasználók elkülönítését.

A `TokenCache` osztály tárol tokenek kibocsátó, az erőforrás, az ügyfél-azonosító és a felhasználói indexelik dictionary. Egy egyéni jogkivonat gyorsítótára kell írni a ehhez a szótárhoz egy biztonsági, például a Redis gyorsítótár.

A Dejójáték felmérések alkalmazásban a `DistributedTokenCache` osztály megvalósít a jogkivonatok gyorsítótárát. Ez a megvalósítás a [IDistributedCache] [ distributed-cache] absztrakciós az ASP.NET Core. Ezzel a módszerrel bármely `IDistributedCache` egy biztonsági tár végrehajtására használható.

* Alapértelmezés szerint a felmérések alkalmazást használja a Redis gyorsítótár.
* Egypéldányos webkiszolgáló, használhatja az ASP.NET Core [memórián belüli gyorsítótárral][in-memory-cache]. (Ez történik akkor is helyben fut az alkalmazás fejlesztése során jó választás.)

`DistributedTokenCache`kulcs/érték párok biztonsági tárolójában tárolja a gyorsítótárazott adatokat. A kulcs a felhasználói Azonosítót és az ügyfél-azonosító, így a biztonsági tár tárolja a felhasználó vagy Windows-ügyfélen egyedi kombinációjához külön gyorsítótárazott adatokat.

![Jogkivonat gyorsítótára](./images/token-cache.png)

A biztonsági tár felhasználó particionálva van. Az egyes HTTP-kérelem a jogkivonatok az adott felhasználó olvasni a biztonsági tár és betölti a `TokenCache` szótárban. Ha a Redis szolgál a biztonsági tár kiszolgálófarm összes server-példány olvasása/írása az azonos gyorsítótárába, és sok felhasználó méretezi ezt a módszert használja.

## <a name="encrypting-cached-tokens"></a>Gyorsítótárazott jogkivonatok titkosítása
A jogkivonatok bizalmas adatokat, mert azok egy felhasználó erőforrásokhoz való hozzáférés engedélyezése. (Továbbá eltérően a jelszó, nem csak tárolhatja a token kivonatát.) Ezért kiemelten fontos annak védelme jogkivonatok illetéktelen kezekbe. A Redis-biztonsági gyorsítótár jelszóval védett, de ha valaki beszerzi a jelszót, azok az lehetett beolvasni a gyorsítótárazott hozzáférési jogkivonatok mindegyikét. Emiatt a `DistributedTokenCache` minden, ami ír a biztonsági tár titkosítja. Titkosítási történik, az ASP.NET Core segítségével [adatvédelem] [ data-protection] API-k.

> [!NOTE]
> Ha telepíti az Azure-webhelyek, a titkosítási kulcsok biztonsági másolat a hálózati tároláshoz és az összes gép szinkronizálását (lásd: [felügyeleti és-élettartam][key-management]). Alapértelmezés szerint kulcsok nem titkosítását, ha fut az Azure-webhelyek, azonban úgy is [engedélyezheti a titkosítást, X.509 tanúsítvánnyal][x509-cert-encryption].
> 
> 

## <a name="distributedtokencache-implementation"></a>DistributedTokenCache végrehajtása
A `DistributedTokenCache` származik-az ADAL [TokenCache] [ tokencache-class] osztály.

A konstruktor a `DistributedTokenCache` osztály egy kulcs az aktuális felhasználó hoz létre, és betölti a gyorsítótár a biztonsági store-ból:

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

A kulcs hozta létre hozzáfűzésével a felhasználói Azonosítót és az ügyfél-azonosítót. Mindkét esetben készít a jogcímek a felhasználó megtalálható `ClaimsPrincipal`:

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

A gyorsítótár adatainak betöltése olvasni a szerializált blob a biztonsági és hívás `TokenCache.Deserialize` gyorsítótárazott adatokat a blob alakítani.

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

Amikor a gyorsítótár-hozzáféréshez adal-t, akkor következik be egy `AfterAccess` esemény. Ha a gyorsítótárazott adatokat megváltozott, a `HasStateChanged` tulajdonság értéke igaz. Ebben az esetben frissíteni a biztonsági tár a változásnak, és utána állítsa be `HasStateChanged` false értékre.

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

TokenCache küld két esemény:

* `BeforeWrite`. Hívni a azonnal adal-t a gyorsítótárba ír. Ezzel a feldolgozási stratégia megvalósításához
* `BeforeAccess`. Hívni a azonnal ADAL beolvassa a gyorsítótárból. Itt is betöltheti a gyorsítótárban a legújabb verzióra.

Ebben az esetben döntöttünk nem két eseményekből kezelése érdekében.

* Párhuzamossági, az utolsó írás wins. Ez OK, mert, hogy jogkivonatok tárolják egymástól függetlenül minden felhasználót + ügyfél, így ütközés csak akkor fordulhat elő, ha ugyanazon felhasználó kellett két egyidejű munkamenetek.
* Olvasását, a gyorsítótár halasztása minden kérelemnél betölteni azt. Kérések rövid élettartamú használata. Adott időpont módosításakor a gyorsítótár lekérdezi a következő kérés felveszi az új érték.

[**Következő**][client-assertion]

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
