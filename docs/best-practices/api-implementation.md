---
title: "API-implementálási segédlet"
description: "Útmutatás az API-k megvalósítása után."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: cc28864de36afdeed2f8a7155a307e312c3a398e
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/05/2018
---
# <a name="api-implementation"></a>API-implementáció

Gondosan tervezett RESTful webes API-k meghatározása az erőforrásokat, a kapcsolatokat és a navigációs rendszerek ügyfélalkalmazások számára elérhető. Megvalósítása és központi telepítése a webes API-k, gondolja át a fizikai követelményeinek a a webes API-t és az üzemeltetési környezetben található, amely a webes API-k összeállított ahelyett, hogy az adatok logikai szerkezetének. Ez az útmutató mutatja be gyakorlati tanácsok a webes API-k megvalósítása és közzétételi azt, hogy az ügyfélalkalmazások számára elérhető legyen. Részletes információ a webes API modell: [API tervezési útmutató](/azure/architecture/best-practices/api-design).

## <a name="processing-requests"></a>Kérelmek feldolgozása

A kód a tanúsítványigénylések bevezetésekor, vegye figyelembe a következő szempontokat.

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a>GET, PUT, DELETE, HEAD és JAVÍTÁSI műveletek idempotent kell lennie

Ezek a kérelmek megvalósító kódot kell ugyanazok a mellékhatással működő. A kérésben felett erőforrást ismétlődő olyan állapotban kell kiváltani. Például több DELETE kérelmet küld a ugyanilyen URI kell rendelkeznie a hatást, bár lehet, hogy a HTTP-állapotkód a válaszüzenetek különböző. Az első törlési kérés előfordulhat, hogy vissza állapotkód: 204 (nincs tartalom), miközben egy későbbi törlési kérés előfordulhat, hogy térjen vissza az állapotkód: 404-es (nem található).

> [!NOTE]
> A cikk [idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathan Oliver blogjában áttekintést idempotencia, és hogyan vonatkozik az felügyeleti műveleteket.
>

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a>Hozzon létre új erőforrások utáni műveletek kell nem rendelkezik egymástól független hatásai

Hozzon létre egy új erőforrást egy POST kérést olyan, ha a kérelem eredő korlátozódik az új erőforrás (és valószínűleg bármely közvetlenül kapcsolódó erőforrások esetén valamilyen kapcsolat érintett) például egy kereskedelmi rendszerben POST kérelmet létrehoz egy új rendelést, az ügyfél is előfordulhat, hogy módosítani kell a készlet szintek és számlázási adatokat, de nem módosíthatja a sorrend nem közvetlenül kapcsolódó információkat vagy mellékhatásokkal bármely más – a rendszer általános állapotát.

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a>Kerülje a chatty POST, PUT és DELETE műveletek végrehajtása

Támogatja a POST, PUT és DELETE kérelmek felett erőforrást gyűjtemények. Egy POST kérést is tartalmazza az új erőforrások több, és adja hozzá az összes ugyanaz a gyűjtemény, PUT-kérelmekben lecserélheti egy gyűjtemény-erőforrások teljes készletét, és a törlési kérelem távolíthatja el egy teljes gyűjteményt.

Kötegelt kérelmekben teszi lehetővé az OData-támogatást ASP.NET Web API 2 szerepel. Egy ügyfélalkalmazás becsomagolhatja a több webes API-kérelmek és küldje el a kiszolgáló a egyetlen HTTP-kérelmek, és egyetlen HTTP-válasz tartalmazza a válaszok az egyes kérelmek fogadására. További információ [kötegelt támogatást bevezeti a webes API- és webes API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx).

### <a name="follow-the-http-specification-when-sending-a-response"></a>A válasz küldésekor, hajtsa végre a HTTP-specifikáció 

Egy webes API-t, hogy a megfelelő HTTP-állapotkód: ahhoz, hogy az ügyfél határozza meg, hogyan legyen kezelve az eredmény, a megfelelő HTTP-fejlécek, hogy az ügyfél megértette az eredmény, és az ügyfél számára, hogy megfelelően formázott törzs tartalmazó üzenetek kell visszaadnia. az eredmény elemzése. 

Például a POST műveletet kell visszaadnia állapotkód (létrehozva) 201 és a válaszüzenetet a helyet megjelölő fejlécet a válaszüzenet bele kell foglalni az újonnan létrehozott erőforrás URI.

### <a name="support-content-negotiation"></a>Támogatja a tartalom egyeztetését

A válasz üzenet törzsét tartalmazhat adatok többféle formátumúak. Például egy HTTP GET kérést sikerült elküldeni JSON, vagy XML-formátuma. Az ügyfél elküldte a kérelmet, azt is adja meg az Accept fejlécet, amely meghatározza az adatok formátumok, amelyet kezelni tud. Ezek a formátumok adathordozó-típusok vannak megadva. Ügyfél, amely kibocsát egy GET kérelmet, amely lekéri a lemezkép megadhatja például, egy Accept fejlécet, amely az ügyfél képes kezelni, például a "kép/jpeg, kép/gif, kép vagy png" média-típusok listája.  A webes API-t az eredményt adja vissza, ha azt kell adatok formázásához médiatípust egyikének használatával, és adja meg a formátum a válasz a Content-Type fejléc.

Ha az ügyfél nem adott meg az Accept fejlécet, majd formátumának a használatára nem ésszerű alapértelmezett a választörzset. Tegyük fel az ASP.NET Web API-keretrendszer alapértelmezés szerint az JSON szöveges adatok.

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a>Tartalmaznak egy HATEOAS stílusú navigációs és az erőforrások felderítési támogatása

A HATEOAS megközelítés lehetővé teszi az ügyfél keresse meg és felderíthetik az erőforrásokat egy kezdeti kiindulási pontot. Ez az tartalmazó URI-azonosítók; hivatkozások használatával érhető el Amikor egy ügyfél megszerezni egy erőforrást HTTP GET kérelmet ad ki, a válasz URI-azonosítók, amelyek lehetővé teszik egy ügyfélalkalmazás bármely közvetlenül kapcsolódó erőforrások kereshetők meg gyorsan kell tartalmaznia. Például a webes API-k, amely az elektronikus kereskedelmi megoldás támogatja, az ügyfél esetleg elhelyezett megrendelést. Ha egy ügyfél-alkalmazás adatait ügyfél olvas be, a válasz hivatkozások, amelyek lehetővé teszik az ügyfélalkalmazás küldeni HTTP GET kérelmeket, amelyek ezeket a rendeléseket kell tartalmaznia. Emellett HATEOAS stílusú hivatkozások le kell írnia a többi művelet (POST, PUT, DELETE, és így tovább), hogy minden kapcsolódó erőforrás által támogatott minden egyes kérelem teljesítéséhez a hozzá tartozó URI együtt. Ez a megközelítés a további részletes leírását lásd [API tervezési][api-design].

Jelenleg nincsenek HATEOAS végrehajtásának meghatározó szabványok nem, de a következő példában látható módon egy lehetséges módszer. Ebben a példában a részletek megtalált ügyfél HTTP GET kérelemre adja vissza egy választ, amely az adott ügyfélhez tartozó rendelések hivatkozó HATEOAS mutató hivatkozásokat tartalmaznak:

```HTTP
GET http://adventure-works.com/customers/2 HTTP/1.1
Accept: text/json
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"CustomerID":2,"CustomerName":"Bert","Links":[
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

Ebben a példában a felhasználói adatok által képviselt a `Customer` osztály a következő kódrészletben látható. A HATEOAS hivatkozások tartják a `Links` gyűjteménytulajdonság:

```csharp
public class Customer
{
    public int CustomerID { get; set; }
    public string CustomerName { get; set; }
    public List<Link> Links { get; set; }
    ...
}

public class Link
{
    public string Rel { get; set; }
    public string Href { get; set; }
    public string Action { get; set; }
    public string [] Types { get; set; }
}
```

A HTTP GET művelet lekérdezi a felhasználói adatok tárolási és szerkezetek egy `Customer` objektumot, és majd feltölti a `Links` gyűjtemény. Az eredmény válaszüzenetet JSON formátumúak. Mindegyik hivatkozás a következő mezőket tartalmazza:

* A visszaküldött objektum és a kapcsolat által ismertetett objektum közötti kapcsolat. Ebben az esetben "önkiszolgáló" azt jelzi, hogy a hivatkozás egy hivatkozást az objektum (hasonlóan egy `this` sok objektumorientált nyelvű mutató), "rendelések" Ez a név az ahhoz kapcsolódó információkat tartalmazó gyűjtemény.
* A hivatkozás (`Href`) által URI formájában hivatkozás alatt leírt objektumhoz.
* A HTTP-kérelem típusa (`Action`), amely ezt az URI lehet küldeni.
* Adatok formátuma (`Types`), amely kell adni a HTTP kérelem, vagy adhatók vissza a válaszban, a kérelem típusától függően.

A példa HTTP-válasz HATEOAS hivatkozások arra utal, hogy egy ügyfél-alkalmazás a következő műveleteket hajthat végre:

* Az URI azonosító a HTTP GET kérelemre `http://adventure-works.com/customers/2` az ügyfél (újra) részleteinek beolvasása. Az adatok adhatók vissza XML-vagy JSON-NÁ.
* Az URI egy HTTP PUT-kérelmet `http://adventure-works.com/customers/2` módosításához az ügyfél részletes adatait. Az új adatokat a kérelemüzenet x-www-form-urlencoded formátumban kell megadni.
* Az URI egy HTTP DELETE kérelmet `http://adventure-works.com/customers/2` az ügyfél törli. A kérelem nem minden további információkra, vagy vissza adatokat a válasz üzenet törzsében.
* Az URI azonosító a HTTP GET kérelemre `http://adventure-works.com/customers/2/orders` az ügyfél a rendeléseket kereséséhez. Az adatok adhatók vissza XML-vagy JSON-NÁ.
* Az URI egy HTTP PUT-kérelmet `http://adventure-works.com/customers/2/orders` hozzon létre egy új ahhoz, hogy az ügyfél számára. Az adatok a kérelemüzenet x-www-form-urlencoded formátumban kell megadni.

## <a name="handling-exceptions"></a>Kivételek kezelése

Ha egy művelet nem kezelt kivételt jelez, vegye figyelembe a következő szempontokat.

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a>Rögzítheti a kivételeket, és az ügyfelek számára értelmes választ

A HTTP-művelet megvalósító kódot kell biztosítania az átfogó kivétel kezelése és terjesztése a keretrendszer nem kezelt kivételek helyett. Kivétel nem teszi lehetővé a művelet sikeres, ha vissza a válaszüzenetben található függvénynek adható át a kivételt, de egy beszédes leírást, a hiba a kivételt okozó tartalmaznia kell. A kivétel is tartalmaznia kell minden esetben egyszerűen adatszolgáltató állapotkódja 500 a megfelelő HTTP-állapotkód helyett. Például, ha a felhasználó kérésére, amely megsérti a korlátozás (például egy ügyfél, amely rendelkezik a nyitott rendelések törlésére tett kísérlet) adatbázis frissítése miatt, térjen vissza állapot kód 409 (Ütközés) és az ütközés okát jelző üzenet törzse. Ha bizonyos más feltétel Ez a beállítás a kérelem elérhetetlen, állapotkód: 400 (hibás kérés) térhet vissza. A HTTP-állapotkódok teljes listáját megtalálhatja a [állapotkódok definícióit](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) lap a W3C-webhelyen.

A Kódpélda trapek különböző feltételeket, és egy megfelelő választ ad vissza.

```csharp
[HttpDelete]
[Route("customers/{id:int}")]
public IHttpActionResult DeleteCustomer(int id)
{
    try
    {
        // Find the customer to be deleted in the repository
        var customerToDelete = repository.GetCustomer(id);

        // If there is no such customer, return an error response
        // with status code 404 (Not Found)
        if (customerToDelete == null)
        {
                return NotFound();
        }

        // Remove the customer from the repository
        // The DeleteCustomer method returns true if the customer
        // was successfully deleted
        if (repository.DeleteCustomer(id))
        {
            // Return a response message with status code 204 (No Content)
            // To indicate that the operation was successful
            return StatusCode(HttpStatusCode.NoContent);
        }
        else
        {
            // Otherwise return a 400 (Bad Request) error response
            return BadRequest(Strings.CustomerNotDeleted);
        }
    }
    catch
    {
        // If an uncaught exception occurs, return an error response
        // with status code 500 (Internal Server Error)
        return InternalServerError();
    }
}
```

> [!TIP]
> Ne adjon meg információt, hogy egy támadó megpróbálta behatoljanak az API-érdemes lehet.
  
Sok webkiszolgáló trapfeltételek hiba magukat a webes API elérése előtti. Például ha a webhely hitelesítést, és a felhasználó nem adja meg a megfelelő hitelesítési adatokat, a webkiszolgáló kell válaszolnia állapotkód 401 (nem engedélyezett). Amennyiben az ügyfél hitelesítése megtörtént, a kód végezheti a saját ellenőrzi, hogy ellenőrizze, hogy az ügyfél férnek hozzá a kért erőforrás kell lennie. Ha a hitelesítés sikertelen, térjen vissza állapotkód 403 (tiltott).
 
### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a>Kivételek következetesen kezelnek, és a hibákkal kapcsolatos adatok

Egységes módon kezelje a kivételeket, vegye fontolóra egy globális hibakezelési stratégia a teljes webes API-k között. Is tartalmazniuk kell a naplózás, amely részletesen az egyes kivétel; rögzíti hiba Ez a hiba napló részletes információkat is tartalmazhat, mindaddig, amíg azt nem elérhetővé válik az interneten keresztül az ügyfelek számára. 

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a>Hibák az ügyféloldali és kiszolgálóoldali hibák megkülönböztetésére

A HTTP protokoll különböztet miatt az ügyfélalkalmazás (a HTTP 4xx állapotkódok) előforduló hibákat, és a kiszolgálón (a HTTP 5xx állapotkódok) szülőmappához által okozott hibákat. Győződjön meg arról, hogy a válasz hibaüzeneteket az egyezmény tiszteletben.

## <a name="optimizing-client-side-data-access"></a>Ügyféloldali adatelérési optimalizálása
Például a webkiszolgáló és az ügyfélalkalmazások elosztott környezetben a hálózati érintő elsődleges forrásokból. Ez működhet, és jelentős szűk keresztmetszet, különösen akkor, ha az ügyfélalkalmazás gyakran kérelmeket küld vagy fogad adatokat. Ezért akkor érhető el a hálózaton keresztül zajló kommunikációról forgalom csökkentése érdekében. A kód lekéri és adatok karbantartása bevezetésekor, vegye figyelembe a következő szempontokat:

### <a name="support-client-side-caching"></a>Támogatja az ügyféloldali gyorsítótárazás

A HTTP 1.1 protokoll támogatja a gyorsítótárazást az ügyfeleken és a köztes kiszolgálókon keresztül, amely egy kérelem webproxykiszolgálókra irányítja a Cache-Control fejléc használatát. Amikor egy ügyfél-alkalmazás egy HTTP GET kérést küld a webes API-t, a válasz tartalmazhat egy Cache-Control-fejlécet, amely jelzi, hogy az adatokat a választörzs biztonságosan gyorsítótárazhatók az ügyfél vagy egy közbenső kiszolgálón, amelyen keresztül a kérelem volt irányíthatja, és mennyi ideig, mielőtt azt lejárati és elavult veszi figyelembe. A következő példa bemutatja a HTTP GET kérelemre, és a sérülésre adott válasz, amely tartalmazza a Cache-Control fejléc:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

Ebben a példában a Cache-Control-fejlécet határozza meg, hogy az adatok vissza kell lejárt 600 másodperc után csak egyetlen ügyfél számára megfelelő és nem kell más ügyfelek által használt megosztott gyorsítótárával tárolni (Ez *titkos*). Megadhatja a Cache-Control fejléc *nyilvános* helyett *titkos* ebben az esetben az adatok megosztott gyorsítótárával tárolhatja, vagy megadhatja azt *no-tároló* ebben az esetben az adatok kell **nem** kell az ügyfél gyorsítótárába. Az alábbi példakód bemutatja, hogyan hozható létre a válaszüzenetet Cache-Control fejléc:

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...
        // Create a Cache-Control header for the response
        var cacheControlHeader = new CacheControlHeaderValue();
        cacheControlHeader.Private = true;
        cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
        ...

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            CacheControlHeader = cacheControlHeader
        };
        return response;
    }
    ...
}
```

Ez a kód egy egyéni használ `IHttpActionResult` osztályt `OkResultWithCaching`. Ez az osztály lehetővé teszi, hogy a tartományvezérlő beállítása a gyorsítótár fejléc tartalma:

```csharp
public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
{
    public OkResultWithCaching(T content, ApiController controller)
        : base(content, controller) { }

    public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
        : base(content, contentNegotiator, request, formatters) { }

    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }

    public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response;
        try
        {
            response = await base.ExecuteAsync(cancellationToken);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;
        }
        catch (OperationCanceledException)
        {
            response = new HttpResponseMessage(HttpStatusCode.Conflict) {ReasonPhrase = "Operation was cancelled"};
        }
        return response;
    }
}
```

> [!NOTE]
> A HTTP protokollt is meghatároz. a *no-cache* fejlécéhez Cache-Control direktíva. Ahelyett, hogy megtévesztően Ez a direktíva nem jelent "nem gyorsítótárazzák" helyett "kísérelje meg újra érvényesítését a gyorsítótárban lévő adatokkal a kiszolgálóval való visszaküldés előtt."; de az adatok továbbra is gyorsítótárazható, de a rendszer minden alkalommal, amikor ellenőrzi segítségével győződjön meg arról, hogy az aktuális továbbra is.
>
>

Gyorsítótár kezelése feladata az ügyfélalkalmazás vagy köztes server, de ha a megfelelően megvalósított is sávszélességet, és a jobb teljesítmény érdekében szükség már nemrég beolvasása fetch adatokhoz.

A *maximális-életkora* a Cache-Control fejléc értéke csak egy útmutató, és nem biztosítja, hogy a megfelelő adatokat nem módosítja a megadott időszakban. A webes API-t kell beállítania a maximális életkora attól függően, hogy a várt illékonyság az adatok a megfelelő értéket. Ha ez az időszak lejár, az ügyfél törölnie kell az objektumot a gyorsítótárból.

> [!NOTE]
> A legtöbb modern böngésző támogatja, ügyféloldali gyorsítótárazás a megfelelő a cache-control fejlécek hozzáadásával a kérelmekre, és az eredmények a fejlécek vizsgálata leírtak szerint. Egyes régebbi böngészők azonban nem gyorsítótárazhatják az egy URL-címet, amely tartalmazza a lekérdezési karakterlánc által visszaadott értékeket. Ez a nem általában probléma valósítja meg a saját gyorsítótár felügyeleti stratégiájának a Microsofttól protokollon alapuló egyéni ügyfélalkalmazások esetében.
>
> Néhány régebbi proxyk ugyanilyen viselkedést, és előfordulhat, hogy gyorsítótárazzák URL-címek alapján a lekérdezési karakterláncokat tartalmazó kérelmeket. Ez az egyedi ügyfél-alkalmazások például olyan proxyn keresztül a webkiszolgálóhoz való csatlakozáshoz problémát okozhat.
>

### <a name="provide-etags-to-optimize-query-processing"></a>Adja meg a lekérdezés feldolgozása optimalizálása ETag-EK

Ha egy ügyfélalkalmazás olvas be olyan objektum, a válaszüzenetet is tartalmazhatnak egy *ETag* (Entitáscímke). Az ETag, amely jelzi, az erőforrás; verziója nem átlátszó karakterláncra minden alkalommal, amikor egy erőforrást módosítja az Etag is módosul. Az ügyfélalkalmazás az ETag gyorsítótárazza az adatokat részeként. Az alábbi példakód bemutatja, hogyan adja hozzá az egy ETag HTTP GET kérelemre adott válasz részeként. Ezt a kódot használja a `GetHashCode` metódus az objektum létrehozása egy numerikus érték, amely azonosítja az objektum (ha szükséges, bírálja felül ezt a módszert és saját olyan algoritmussal, például az MD5 kivonatoló készítése):

```csharp
public class OrdersController : ApiController
{
    ...
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...

        var hashedOrder = order.GetHashCode();
        string hashedOrderEtag = $"\"{hashedOrder}\"";
        var eTag = new EntityTagHeaderValue(hashedOrderEtag);

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            ...,
            ETag = eTag
        };
        return response;
    }
    ...
}
```

A webes API-k által közzétett válaszüzenetet így néz ki:

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
ETag: "2147483648"
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

> [!TIP]
> Biztonsági okokból nem engedélyezett bizalmas adatok vagy gyorsítótárazható hitelesített (HTTPS) kapcsolaton keresztül által visszaadott adatokat.
>
>

Egy ügyfélalkalmazás is későbbi GET kérést erőforrást beolvasásához bármikor, és ha az erőforrás változott (rendelkezik egy eltérő ETag) nem használhatók fel a gyorsítótárazott verzió és az új verzió kerüljön a gyorsítótárba. Ha egy erőforrás nagy, és az ügyfélnek küldött sávszélesség jelentős időt igényel, ugyanazok az adatok lehívása ismétlődő kérelmek nem elég hatékony válhat. Ez elleni, a HTTP protokoll meghatározza, hogy a GET kérelmek egy webes API-t támogató kell optimalizálása a következő folyamat:

* Az ügyfél az If-None-Match HTTP-fejlécekben hivatkozott erőforrás-jelenleg gyorsítótárazott verziójának ETag tartalmazó GET kérést hoz létre:

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```
* A webes API-t a GET műveletet szerzi be a jelenlegi ETag a kért adatok (2. sorrendje a fenti példában), és összehasonlítja az értéket az If-None-Match fejléc.
* Ha a jelenlegi ETag a kért adatok megegyezik a kérés megadott ETag, az erőforrás nem változott, és a webes API-t egy HTTP-válasz 304 (nem módosított) állapotkódot és egy üres üzenettörzs kell visszaadnia.
* A kért adatok jelenlegi ETag nem egyezik a kérelem által biztosított, ha az adatok változásairól, és a webes API-t kell visszaadnia egy HTTP-válasz az új adatokkal az üzenettörzs és 200-as (OK) állapotkódot.
* Ha a kért adatok már nem létezik a webes API-t az állapotkód: 404-es (nem található), egy HTTP-válasz kell visszaadnia.
* Az ügyfél az állapotkód: használja a gyorsítótár karbantartása. Ha az adatok nem változott (állapotkód 304), akkor az objektum gyorsítótárazott maradhat, és az ügyfélalkalmazás továbbra is az az objektum jelenlegi verzióját használja. Ha az adatok megváltozott (állapotkód 200), majd a gyorsítótárazott objektum kellene, és az újat kerülnek. Ha már nem érhető el az adatokat (állapotkód: 404), majd az objektum el kell távolítani a gyorsítótárból.

> [!NOTE]
> Ha a válasz fejléce a Cache-Control fejléc-tároló nem tartalmaz, akkor az objektum mindig el kell távolítani a gyorsítótárból, függetlenül a HTTP-állapotkód:.
>

Az alábbi kódot a `FindOrderByID` metódus terjeszteni az If-None-Match fejléc támogatja. Figyelje meg, hogy ha az If-None-Match fejléc hiányzik, a megadott sorrendben mindig olvassa:

```csharp
public class OrdersController : ApiController
{
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        try
        {
            // Find the matching order
            Order order = ...;

            // If there is no such order then return NotFound
            if (order == null)
            {
                return NotFound();
            }

            // Generate the ETag for the order
            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Create the Cache-Control and ETag headers for the response
            IHttpActionResult response;
            var cacheControlHeader = new CacheControlHeaderValue();
            cacheControlHeader.Public = true;
            cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
            var eTag = new EntityTagHeaderValue(hashedOrderEtag);

            // Retrieve the If-None-Match header from the request (if it exists)
            var nonMatchEtags = Request.Headers.IfNoneMatch;

            // If there is an ETag in the If-None-Match header and
            // this ETag matches that of the order just retrieved,
            // then create a Not Modified response message
            if (nonMatchEtags.Count > 0 &&
                String.CompareOrdinal(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
            {
                response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NotModified,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }
            // Otherwise create a response message that contains the order details
            else
            {
                response = new OkResultWithCaching<Order>(order, this)
                {
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }

            return response;
        }
        catch
        {
            return InternalServerError();
        }
    }
...
}
```

Ebben a példában a szerződés magában foglalja egy további egyéni `IHttpActionResult` osztályt `EmptyResultWithCaching`. Ez az osztály egyszerűen funkcionál csomagolásának egy `HttpResponseMessage` objektum, amely nem tartalmaz egy adott válasz törzse:

```csharp
public class EmptyResultWithCaching : IHttpActionResult
{
    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }
    public HttpStatusCode StatusCode { get; set; }
    public Uri Location { get; set; }

    public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response = new HttpResponseMessage(StatusCode);
        response.Headers.CacheControl = this.CacheControlHeader;
        response.Headers.ETag = this.ETag;
        response.Headers.Location = this.Location;
        return response;
    }
}
```

> [!TIP]
> Ebben a példában az adatok ETag az adatok, az alapul szolgáló adatforrásban kivonatolásával jön létre. Az ETag más módon számítható ki, ha a folyamat további is lehet optimalizálni, és az adatok csak szükség lehet lekérni az adatforrásból, amennyiben a változás.  Ezt a módszert akkor különösen akkor hasznos, ha az adatok mérete nagy, vagy az adatforrás eléréséhez szükséges eredményezhet jelentős késés (például, ha az adatforrás egy távoli adatbázishoz).
>

### <a name="use-etags-to-support-optimistic-concurrency"></a>ETag-EK használatával támogatja az egyidejű hozzáférések optimista

Korábban a gyorsítótárazott adatok frissítések engedélyezéséhez a HTTP protokollt támogatja az egyidejű hozzáférések optimista stratégia. Ha után beolvasása, és a gyorsítótár egy erőforrást, az ügyfélalkalmazás ezt követően módosítsa vagy távolítsa el az erőforrás PUT vagy DELETE kérelmet küld, azt az ETag hivatkozó If-Match fejléc bele kell foglalni. A webes API-t használhatja ezeket az információkat határozza meg, hogy az erőforrás már megtörtént egy másik felhasználó lekérdezés óta, és egy megfelelő választ az ügyfélalkalmazás küldése az alábbiak szerint:

* Az ügyfél hoz létre egy új részletes adatainak megadása az erőforrás és az ETag az If-Match HTTP-fejlécekben hivatkozott erőforrás-jelenleg gyorsítótárazott verziójának tartalmazó PUT-kérelmekben. A következő példa bemutatja, amely sorrendben frissíti a PUT-kérelmekben:

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
* A webes API-t a PUT-műveletével szerzi be a jelenlegi ETag a kért adatok (1. sorrendje a fenti példában), és összehasonlítja az értéket az If-Match fejlécben.
* Ha a jelenlegi ETag a kért adatok megegyezik a kérés megadott ETag, az erőforrás nem változott, és a webes API-k végre kell hajtania a frissítést, egy üzenetet, amelyben a HTTP-állapotkód: 204 (nincs tartalom) ad vissza, ha sikeres. A válasz a Cache-Control és a frissített verziót az erőforrás ETag fejlécek tartalmazhatnak. A válasz mindig tartalmaznia kell a helyre fejlécet tartalmazta, amely az újonnan frissített erőforrás URI hivatkozik.
* Ha a kért adatok jelenlegi ETag nem egyezik meg a kérés megadott ETag, majd az adatok egy másik felhasználó óta megváltozott lehívása és a webes API-t kell visszaadnia egy HTTP-válasz egy üres üzenet szövege és annak 412 (előfeltétel be egy állapotkód: sikertelen).
* Ha frissítenie kell az erőforrás már nem létezik a webes API-t az állapotkód: 404-es (nem található), egy HTTP-válasz kell visszaadnia.
* Az ügyfél a gyorsítótár karbantartása az állapot kód- és válaszfejlécekről használ. Ha az adatok frissítése (állapotkód: 204), akkor az objektum is maradniuk gyorsítótárazott (a Cache-Control fejléc nem adja meg a nem-tároló), de az ETag frissíteni kell. Ha az adatokat egy másik felhasználó módosította (állapotkód 412) módosult, vagy nem található (állapotkód: 404), majd a gyorsítótárazott objektum kellene.

A következő példakód a PUT művelet az rendelések tartományvezérlő megvalósítását mutatja:

```csharp
public class OrdersController : ApiController
{
    [HttpPut]
    [Route("api/orders/{id:int}")]
    public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
    {
        try
        {
            var baseUri = Constants.GetUriFromConfig();
            var orderToUpdate = this.ordersRepository.GetOrder(id);
            if (orderToUpdate == null)
            {
                return NotFound();
            }

            var hashedOrder = orderToUpdate.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Retrieve the If-Match header from the request (if it exists)
            var matchEtags = Request.Headers.IfMatch;

            // If there is an Etag in the If-Match header and
            // this etag matches that of the order just retrieved,
            // or if there is no etag, then update the Order
            if (((matchEtags.Count > 0 &&
                String.CompareOrdinal(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                matchEtags.Count == 0)
            {
                // Modify the order
                orderToUpdate.OrderValue = order.OrderValue;
                orderToUpdate.ProductID = order.ProductID;
                orderToUpdate.Quantity = order.Quantity;

                // Save the order back to the data store
                // ...

                // Create the No Content response with Cache-Control, ETag, and Location headers
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Private = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                hashedOrder = order.GetHashCode();
                hashedOrderEtag = $"\"{hashedOrder}\"";
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                var location = new Uri($"{baseUri}/{Constants.ORDERS}/{id}");
                var response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NoContent,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag,
                    Location = location
                };

                return response;
            }

            // Otherwise return a Precondition Failed response
            return StatusCode(HttpStatusCode.PreconditionFailed);
        }
        catch
        {
            return InternalServerError();
        }
    }
    ...
}
```

> [!TIP]
> Az If-Match fejléc nem teljesen kötelező, és ha nincs megadva a webes API mindig megpróbálja frissíteni a megadott sorrendben, esetleg egy másik felhasználó által végrehajtott frissítés mappába felülírja. Elveszett frissítése problémák elkerülése érdekében mindig adja meg az If-Match fejléc.
>
>

## <a name="handling-large-requests-and-responses"></a>Nagy kérelmeit és válaszait kezelése
Lehetnek olyan alkalmak, amikor egy ügyfél alkalmazást kell elküldeni vagy fogadni a több mérete (MB) lehet adatokat kérelmek ki (vagy nagyobb) mérete. Várakozás a adatmennyiség továbbított okozhat az ügyfélalkalmazás válaszol. Ha jelentős mennyiségű adatot tartalmazó kérelmek kezeléséhez van szüksége, vegye figyelembe a következő szempontokat:

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a>Kérelmeit és válaszait, például a nagyméretű objektumok optimalizálása

Bizonyos erőforrások nagy objektumok vagy nagy mezők, például képeket vagy más típusú bináris adatok szerepelhetnek. A webes API támogatnia kell a folyamatos átviteli optimalizált feltöltése és ezekkel az erőforrásokkal való letöltésének engedélyezése.

A HTTP protokoll biztosítja a darabolásos átviteli kódolás mechanizmus nagyméretű objektumok egy ügyfél adatfolyamként történő küldéséhez. Az ügyfél egy nagy objektumot egy HTTP GET kérést küld, a webes API-t is küldhet a válasz vissza darabonkénti *adattömbök* HTTP-kapcsolaton keresztül. A válaszban adatok hossza nem lehet, hogy kezdetben ismert (Ez lehet, hogy hozható létre), ezért a webes API-t futtató kiszolgálót az egyes adattömbök, amely meghatározza a Transfer-Encoding válaszüzenetet küldött e: darabolásos Content-Length fejlécet helyett fejléc. Az ügyfélalkalmazás kaphat az egyes adattömbök viszont, hogy a teljes válasz kialakításához. Az adatok átvitele befejeződött, ha a kiszolgáló küld vissza a végső rendszer mérete nulla. 

Egyetlen kérelem kapcsolódását eredményezheti egy nagy objektumot, amely jelentős erőforrásokat használ fel. Ha a folyamatos átviteli folyamat során a webes API-k határozza meg, hogy a kérelemben adatok mennyisége túllépte néhány elfogadható határai, akkor megszakítja a műveletet, és vissza válaszüzenetet állapotkóddal 413 (kérelem entitás túl nagy).

Minimalizálhatja a HTTP-tömörítés a hálózaton keresztül továbbított nagy objektumok méretét. Ez a megközelítés segít csökkentheti a hálózati forgalom és a kapcsolódó hálózati késés, de az ügyfél és a webes API-t futtató kiszolgáló további feldolgozását igénylő. Például egy ügyfélalkalmazást, amely a tömörített adatok fogadására vár tartalmazhatnak elfogadás-kódolással: gzip kérelem fejléce (más algoritmusok is megadható adattömörítés). Ha a kiszolgáló támogatja a tömörítést azt kell válaszolnia a gzip-formátumban a üzenet szövege és a tartalom-kódolás tárolt tartalmat: gzip válaszfejlécet.

Kombinálhatja a kódolt tömörítés, amelynél adatfolyamos; először előtt streaming az adatok tömörítésére, és adja meg a gzip tartalmának kódolását és darabolásos átviteli kódolás a üzenetfejlécben. Ne feledje, hogy olyan webkiszolgálók (például az Internet Information Server) beállítható úgy, hogy automatikusan tömöríti a HTTP-válaszokat, függetlenül attól, hogy a webes API-k tömöríti-e az adatokat, vagy nem.

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a>Részleges válasz az ügyfelek, amelyek nem támogatják az aszinkron műveletek végrehajtása

Aszinkron streaming alternatívájaként ügyfélalkalmazás is igényelhetnek adattömböket, részleges válasz néven nagy objektumok adatait. Az ügyfélalkalmazás megszerezni az objektum adatait egy HTTP HEAD-kérést küld. Ha a webes API támogatja a részleges válasz, ha a HEAD kérelem, amely tartalmaz egy elfogadás-tartományok és a Content-Length fejlécet, amely az objektum teljes méretét válaszüzenetet válaszolnia kell, de az üzenet törzsét üresnek kell lennie. Az ügyfélalkalmazás ezen információk használatával hozható létre több GET kérelem által megadott bájttartomány fogadására. A webes API HTTP-állapot 206 (részleges tartalom), a Content-Length fejlécet, amely meghatározza a válaszüzenetet, és a Content-Range fejléc, amely jelzi, mely része (például bájt 4000 üzenettörzsbeli adatok tényleges mennyiségét válaszüzenetet kell visszaadnia. 8000) az az objektum az adatok jelöli.

Részletesen ismerteti a HTTP HEAD-kérelmek és a részleges válasz [API tervezési][api-design].

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a>Ne küldjön szükségtelen 100-ügyfélalkalmazások található állapotüzenetek folytatása

Egy ügyfél-alkalmazás, amely arra készül, hogy a nagy mennyiségű adatot küldeni előfordulhat, hogy meghatározásához először a kiszolgáló tényleges hajlandó a kérést. Az adatküldés előtt az ügyfélalkalmazás is elküldhetik a egy várt HTTP-kérelem: 100-továbbra is a fejlécet, a Content-Length fejlécet, amely az adatokat, de egy üres üzenettörzs méretét jelzi. Ha a kiszolgáló hajlandó kezelni a kérést, azt egy üzenet, amely meghatározza a HTTP-állapot – 100 (Folytatás) kell válaszolnia. Az ügyfélalkalmazás majd a folytatáshoz, és a teljes, beleértve az adatokat az üzenet törzsében kérelem küldése.

Ha az IIS használatával az üzemeltetett szolgáltatás, a HTTP.sys illesztő automatikusan észleli és kezeli a várt: 100-fejlécek folytatni a kérelmek a webes alkalmazásba való továbbítása előtt. Ez azt jelenti, hogy nem valószínű, hogy ezek a fejlécek, az alkalmazás kódjában, és akkor feltételezheti, hogy az IIS már szűri, akkor megfelelően konfiguráltnak ítéli alkalmatlan vagy túl nagy üzeneteket.

Ha a .NET-keretrendszer használatával hoz létre az ügyfélalkalmazások, akkor minden POST és a PUT üzenetek először a várt üzenetet szeretne küldeni: 100-alapértelmezés szerint továbbra is a fejléceket. Csakúgy, mint a kiszolgálóoldali, a folyamat kezeli transzparens módon a .NET-keretrendszer. Ez a folyamat azonban a kiszolgálóra, még akkor is a kis-kéréseket két üzenetváltások utak számát, amely minden POST és a PUT kérés eredményez. Ha az alkalmazás kérelmek nagy mennyiségű adatot nem küld, ez a szolgáltatás segítségével letilthatja a `ServicePointManager` osztály létrehozása `ServicePoint` az ügyfélalkalmazás az objektumok. A `ServicePoint` objektum kezeli a kapcsolatokat, az ügyfél által az alapján a rendszer és a gazdagép darabjai URI, amely azonosítja a kiszolgálón. Ezután beállíthatja a `Expect100Continue` tulajdonsága a `ServicePoint` objektum false értékre. URI, amely megfelel a rendszer és a gazdagép darabjai keresztül az ügyfél által küldött összes további POST és a PUT kérelmek a `ServicePoint` objektum rendszer anélkül küldi el várt: 100-fejlécek folytatni. A következő kód bemutatja, hogyan konfigurálhatja a `ServicePoint` objektum összes tervet az URI-azonosítók küldött kérelmeket a konfigurált `http` és `www.contoso.com`.

```csharp
Uri uri = new Uri("http://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

Azt is beállíthatja a statikus `Expect100Continue` tulajdonsága a `ServicePointManager` adhatja meg ennek a tulajdonságnak az alapértelmezett érték az összes azt követő osztály `ServicePoint` objektumok. További információkért lásd: [ServicePoint osztály](https://msdn.microsoft.com/library/system.net.servicepoint.aspx).

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a>Tördelési támogatja, amelyek nagy számú objektum térhetnek vissza kérelmeknél

Ha egy gyűjtemény sok erőforrást tartalmaz, GET kérés kiadása a hozzá tartozó URI képes jelentős feldolgozási teljesítményét webes API-t üzemeltető kiszolgálón eredményez, és hálózati forgalom, ami jelentős mennyiségű készítése nagyobb késéseket.

Ezekben az esetekben kezelni, a webes API-k támogatnia kell a lekérdezési karakterláncok, amelyek lehetővé teszik az ügyfélalkalmazás kérelmek finomíthatja vagy adatlehívás könnyebben kezelhető, különálló tömb (vagy lapok). Az alábbi kódot a `GetAllOrders` metódust a `Orders` vezérlő. Ez a módszer lekéri a rendelés részleteit. Ez a metódus lett megkötés nélkül, kapcsolódását visszaadhatja nagy mennyiségű adatot. A `limit` és `offset` paraméterek célja, hogy csökkentse az adatok mennyisége kisebb részhalmazát, ez esetben csak az első 10 rendeléseket alapértelmezés szerint:

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders")]
    [HttpGet]
    public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    {
        // Find the number of orders specified by the limit parameter
        // starting with the order specified by the offset parameter
        var orders = ...
        return orders;
    }
    ...
}
```

Egy ügyfélalkalmazás kiadhatnak egy URI segítségével 50 eltolástól kezdve 30 kérelmek lekérésére irányuló kérelem `http://www.adventure-works.com/api/orders?limit=30&offset=50`.

> [!TIP]
> Kerülje a megadásához, amelyek URI, amely az érték 2000-nél több karakter hosszú lekérdezési karakterláncok ügyfélalkalmazások engedélyezését. Számos webalkalmazás-ügyfelek és kiszolgálók ilyen hosszú URI-azonosítók nem tudja kezelni.
>
>

## <a name="maintaining-responsiveness-scalability-and-availability"></a>Reakcióidőt, a méretezhetőség és a rendelkezésre állás fenntartása
Az azonos web API futtató bárhol a világon sok ügyfélalkalmazások előfordulhat, hogy lesz szükség. Fontos biztosítania, hogy a webes API-t kell lennie a magas különböző munkaterhelések támogatásához, és rendelkezésre állás biztosításához az üzleti szempontból kulcsfontosságú műveleteket az ügyfelek méretezhető, nagy terhelés alatt válaszkészsége fenntartásához. Ha ezek a követelmények teljesítéséhez módjának meghatározása, vegye figyelembe a következő szempontokat:

### <a name="provide-asynchronous-support-for-long-running-requests"></a>Aszinkron támogatást nyújt a hosszan futó kérelmek

A kérelmeket, amelyek feldolgozása hosszú időbe telhet, hogy a kérés elküldése megtörtént az ügyfél blokkolása nélkül kell elvégezni. A webes API-k hajthatják végre a kérelem ellenőrzéséhez kezdeményezni egy külön feladat a munkájuk elvégzéséhez, és térjen vissza a HTTP-kód 202 (elfogadható) válaszüzenetet néhány kezdeti ellenőrzése. A feladat futtathat a webes API részeként aszinkron feldolgozás, vagy a háttérfeladat sikerült kiszervezett.

A webes API-t kell is egy olyan mechanizmus biztosítása az ügyfélalkalmazás találatot feldolgozását. Ennek érdekében azáltal, hogy a rendszeres időközönként lekérdezés alkalmazásokat egy olyan ügyfél lekérdezési mechanizmus e feldolgozása befejeződik, és az eredményt vagy engedélyezése a webes API-k elküldeni egy értesítést, ha a művelet befejeződött.

Egy egyszerű lekérdezési módszer biztosításával valósíthat meg egy *lekérdezési* URI, amely úgy működik, mint egy virtuális erőforrást a következő módon:

1. Az ügyfélalkalmazás a kezdeti kérést küld a webes API-t.
2. A webes API-t a kérelem kapcsolatos információkat tárolja a table storage vagy a Microsoft Azure Cache tárolt tábla, és állít elő ez a bejegyzés egyedi kulcs, valószínűleg egy GUID formájában.
3. A webes API-k indít el egy külön feladat feldolgozásával. A webes API-t a feladat állapotát rögzíti a táblázat *futtató*.
4. A webes API-t adja vissza válaszüzenetet HTTP-állapotkód: 202 (elfogadható), és a táblabeli GUID-azonosítója az üzenet törzsét.
5. A feladat befejezése után a webes API-t tárolja az eredményeket a táblában, és a feladat állapotának beállítása *Complete*. Ne feledje, hogy a feladat sikertelen lesz, ha a webes API-k sikerült is a hibával kapcsolatos információkat tárolja az állapot beállítása *sikertelen*.
6. A feladat futása közben, az ügyfél továbbra is a saját feldolgozási végrehajtása. Azt is rendszeresen kérelmet küld az URI */polling/ {guid}* ahol *{guid}* a globálisan egyedi Azonosítót ad vissza a 202 válaszüzenetben a webes API-t.
7. A webes API-t, a */polling/ {guid}* URI a táblázatban a megfelelő feladat állapotának lekérdezése, és adja vissza egy válaszüzenetben HTTP-állapotkód: 200 (OK) ebben az állapotban (*futtató*, *Teljes*, vagy *sikertelen*). Ha a feladat befejeződött, vagy sikertelen volt, a válaszüzenetet is hozzáadhat az eredmények a feldolgozás vagy bármely információ elérhető a hiba okát.

Az értesítések végrehajtási lehetőségek a következők:

- Az Azure Notification Hub használatával, majd az aszinkron válaszok ügyfélalkalmazások számára. További információkért lásd: [Azure Notification Hubs – felhasználók értesítése](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).
- Az ügyfél biztonsági a üstökös modell használatával szeretné megőrizni az ügyfél és a webes API-t futtató kiszolgáló közötti hálózati kapcsolatot, és a kapcsolattal a leküldéses üzenetek a kiszolgálóról. Az MSDN újság cikk [egy egyszerű üstökös alkalmazást a Microsoft .NET-keretrendszer összeállítása](https://msdn.microsoft.com/magazine/jj891053.aspx) egy példa megoldást ismertet.
- SignalR segítségével adatokat küldeni valós idejű a webkiszolgáló az ügyfél hálózati kapcsolaton keresztül. SignalR érhető el az ASP.NET-webalkalmazások számára történő NuGet-csomagként. További információ található a [ASP.NET SignalR](http://signalr.net/) webhelyet.

### <a name="ensure-that-each-request-is-stateless"></a>Gondoskodjon arról, hogy az egyes kérelmek állapot nélküli

Minden egyes kérelem atomi figyelembe venni. Egy ügyfélalkalmazás egy kérelmet, és bármely későbbi ugyanaz az ügyfél által küldött kérelmeket közötti függőségek kell lennie. Ez a megközelítés segít a méretezhetőségi; a webszolgáltatás példánya több kiszolgálót is telepíthető. Ügyfélkérések irányítható, ezek a példányok, és az eredmények mindig azonosnak kell lennie. Ez is növeli a rendelkezésre állási hasonló okból; Ha egy web server sikertelen kérelmeket egy másik példányhoz átirányítható (Azure Traffic Manager használatával) során a kiszolgáló újraindítása nincs sérültnek hatása az ügyfélalkalmazások számára.

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a>Nyomon követheti az ügyfelek és a sávszélesség-szabályozás megvalósítása esélyét csökkentheti a Szolgáltatásmegtagadási támadások

Ha egy adott ügyfél nagy számú kérést küld egy adott időn belül, előfordulhat, hogy se sajátíthassa ki a szolgáltatás és a teljesítményét a más ügyfelekkel. A probléma orvoslása érdekében webes API-k nyomon követését az IP-cím, az összes beérkező kérelem vagy -naplózás minden hitelesített hozzáférést figyelheti ügyfélalkalmazások hívásait. Ezek az információk használatával korlátozható az erőforrásokhoz való hozzáférést. Ha egy ügyfél meghaladja a meghatározott korláttal, a webes API-k vissza válaszüzenetet 503-as (a szolgáltatás nem érhető el) állapotú, és tartalmaznak egy újrapróbálkozási után fejlécet, amely meghatározza, amikor az ügyfél feladateseményeket küldhet a következő kérés nélkül alatt elutasította. Ezt a stratégiát segíthet a veszélyét annak, hogy a szolgáltatás megtagadása (DOS) támadás lévő ügyfelek, a rendszer leállása csökkentése érdekében.

### <a name="manage-persistent-http-connections-carefully"></a>Állandó HTTP-kapcsolatok gondosan kezelése

A HTTP protokoll azok állandó HTTP-kapcsolatokat támogatja. A HTTP 1.0 leírásnak fel a kapcsolatot: Keep-Alive fejléc, amely lehetővé teszi egy ügyfélalkalmazást, annak jelzésére, hogy az újakat nyitása helyett későbbi kérelmek küldéséhez használhatja ugyanazt a kapcsolatot a kiszolgálóval. Ha az ügyfél nem használja fel a kapcsolatot a gazdagép által megadott időn belül automatikusan bezáródik a kapcsolatot. Ez a viselkedés a HTTP 1.1 Azure-szolgáltatások által használt, így nem szükséges belefoglalni a fejlécek életben tartási üzenetek az alapértelmezett beállítás.

Csökkenti a késést és a hálózati torlódás válaszkészsége javítása érdekében kapcsolatot nyitva tartása segítséget, de azok méretezhetőség sértő szükségtelen kapcsolatok hosszabb, mint más egyidejű ügyfelek képességét korlátozása szükséges nyitva tartja Csatlakoztassa. Azt is befolyásolhatja az eszközakkumulátor élettartamának, ha az ügyfél-alkalmazás fut egy mobileszközön; Ha az alkalmazás csak alkalmi kérelmeket küld a kiszolgáló, nyitott kapcsolat fenntartása okozhat az akkumulátor gyorsabban kiürítése. Győződjön meg arról, hogy a kapcsolat nem jön létre a HTTP 1.1 állandó, az ügyfél az alapértelmezett viselkedés felülbírálásához kapcsolat: Close fejlécet is tartalmaznia. Hasonlóképpen ha a kiszolgáló által kezelt nagyon nagy mennyiségű ügyfelet része lehet kapcsolat: Close fejléc a válaszüzenetek, amely kell zárja le a kapcsolatot, és kiszolgáló-erőforrás mentése.

> [!NOTE]
> Állandó HTTP-kapcsolatok csak választható újdonsága a ismételten a kommunikációs csatornát létrehozó társított hálózati terhelésének csökkentése érdekében. A webes API-t és az ügyfélalkalmazás nem függ egy állandó HTTP-kapcsolat rendelkezésre állása. Ne használjon állandó HTTP-kapcsolatok megvalósításához üstökös stílusú értesítési rendszere; Ehelyett sockets (vagy websocket elemek, ha elérhető) használatára kell a TCP-rétegben. Végül vegye figyelembe a Keep-Alive fejléc korlátozott használat van, ha egy ügyfél-alkalmazás; proxyn keresztülmenő kiszolgálóval kommunikálva csak az ügyfél és a proxy kapcsolódási állandó lesz.
>
>

## <a name="publishing-and-managing-a-web-api"></a>Közzététele és webes API-k kezelése
Webes API-k akkor válik elérhetővé az ügyfélalkalmazások, a webes API-t telepíteni kell egy gazdagép-környezetben. Ebben a környezetben az általában a webkiszolgáló, bár lehet, hogy más típusú gazdafolyamat. A webes API közzétételekor, a következő szempontokat kell figyelembe vennie:

* Összes kérelem kell hitelesítenie és engedélyeznie, és a megfelelő szintű hozzáférés-vezérlés kényszerítettnek kell lennie.
* Kereskedelmi webes API-k válaszidők vonatkozó különböző szolgáltatásminőségi garanciák célpontjává válhat. Fontos méretezhető, hogy a gazdagép-környezetben, így ha a terhelést időbeli nagymértékben változhat.
* Elképzelhető, hogy szükséges mérési kérelmekre jeleníthetnek célokra.
* Szabályozza a forgalmat a webes API-hoz, és a kvóták kimerített adott ügyfelek sávszélesség-szabályozás megvalósításához szükség lehet.
* Szabályozási követelmények előfordulhat, hogy engedélyezhetik naplózása és az összes kérelmeit és válaszait naplózását.
* Rendelkezésre állás biztosítása érdekében, akkor lehet szükség a webes API-t futtató kiszolgáló állapotának figyelésére, és szükség esetén indítsa újra.

Akkor célszerű lehet használata leválasztja ezeket a problémákat a műszaki problémák, a webes API-k végrehajtására vonatkozó. Emiatt érdemes lehet létrehozni egy [homlokzati](http://en.wikipedia.org/wiki/Facade_pattern), és külön folyamatban futó kérelmek irányítja a webes API-hoz. A homlokzati biztosíthat a felügyeleti műveleteket, és előre érvényesíteni a kéréseket a webes API-hoz. Egy homlokzati használatával művelettel is sok működési előnyöket, többek között:

* Több Web API-k integrációs pontjaként működött.
* Üzenetek átalakítása, és a kommunikációs protokollokat fordítása ügyfelek különböző technológiák használatával készültek.
* Gyorsítótárazás kérelmeit és válaszait csökkentése érdekében betölteni a webes API-t futtató kiszolgálón.

## <a name="testing-a-web-api"></a>A webes API tesztelése
Egy webes API-t, mint minden más olyan szoftver alaposan kell vizsgálni. Érdemes lehet egység tesztet végrehajtva ellenőrizze a webes API-k jellege működésére létrehozása számos lehetőséget kínál a saját további követelmények ellenőrzése, hogy megfelelően működik-e. Meg kell különös figyelmet fordítani az alábbi szempontokat:

* Az összes útvonal ellenőrzése, hogy azok a megfelelő műveleteket hívhatnak meg a teszteléséhez. Különösen ügyeljen a visszaadott HTTP-állapotkód: (a metódus nem engedélyezett) 405 váratlanul Ez azt jelzi, hogy az útvonal és a HTTP-metódus (GET, POST, PUT, DELETE), amely képes továbbítani, hogy az útvonal eltérést.

    HTTP-kérelmeket küldjön útvonalakat, amelyek nem támogatják őket, például egy adott erőforrás POST kérelem elküldése (a POST kérelmek csak küldjön el erőforrás-gyűjtemények). Ezekben az esetekben az egyetlen érvényes válasz *kell* állapotkód 405 (nem engedélyezett) lehet.
* Győződjön meg arról, hogy az összes útvonal megfelelően védettek, és a megfelelő hitelesítési és engedélyezési ellenőrzés alatt áll.

  > [!NOTE]
  > Például a felhasználó hitelesítési olyan vonatkozásai feltehetően az ezzel a webes API-k, hanem a gazdagép-környezetben, de továbbra is szükséges, hogy biztonsági vizsgálatok tartalmazzák a telepítési folyamat részeként.
  >
  >
* Minden egyes művelet által végrehajtott kivételkezelést tesztelése, és győződjön meg arról, hogy egy megfelelő és a jelentéssel bíró HTTP-válasz küld vissza az ügyfélalkalmazást.
* Ellenőrizze, hogy szabályos és válasz üzeneteket. Például ha egy HTTP POST-kérelmet tartalmazza az adatokat egy új erőforrás x-www-form-urlencoded formátumban, győződjön meg arról, hogy a megfelelő műveletet megfelelően elemzi az adatokat, az erőforrásokat, és az új erőforrás részleteit tartalmazó választ többek között a megfelelő helyet megjelölő fejlécet.
* Ellenőrizze az összes hivatkozásokat és URI-azonosítók a válaszüzenetek. Például egy HTTP POST üzenet az újonnan létrehozott erőforrás URI kell visszaadnia. Minden HATEOAS kapcsolat érvényesnek kell lennie.

* Győződjön meg arról, hogy a minden művelet bemeneti különböző kombinációkban megfelelő állapotkódjai adja vissza. Példa:

  * Sikeres lekérdezés esetén visszaadja-e állapotkód 200 (OK)
  * Ha egy erőforrás nem található, a művelet HTTP-állapotkód: 404-es (nem található) kell visszaadnia.
  * Ha az ügyfél elküldi a kérelmeket, amelyek sikeresen töröl egy erőforrást, az állapotkód: 204 (nincs tartalom) kell lennie.
  * Az ügyfél által létrehozott egy új erőforrást kérelmet küld, ha az állapotkódnak kell-e a 201-es (létrehozva)

Figyelje, hogy váratlan állapotkódok 5xx tartományban vannak. Ezek az üzenetek általában jelenti a kiszolgáló által annak jelzésére, hogy az eszköz nem egy érvényes kérelem teljesítéséhez.

* A különböző kérelem fejléc kombinációit, amelyek egy ügyfélalkalmazást, adja meg, és győződjön meg arról, hogy a webes API-t adja vissza a várt információkat válaszüzenetek tesztelése.
* Lekérdezési karakterláncok tesztelése. Egy műveletet a választható paraméterek: (például tördelési kérések) is igénybe vehet, ha tesztelni a különböző kombinációkban és a paraméterek sorrendjét.
* Győződjön meg arról, hogy az aszinkron műveletek sikeresen befejeződnek. Ha a webes API támogatja az adatfolyam-kérelmek nagyméretű bináris objektumok (például a video- vagy hang) visszaadó, győződjön meg arról, hogy ügyfélkérelmek nem tiltanak közben az adatok továbbítja adatfolyamként. Ha a webes API-k megvalósítja a hosszan futó adatmódosítási műveletek lekérdezése, győződjön meg arról, hogy a műveletek jelentett állapotuk megfelelően azok a folytatáshoz.

Inkább hozzon létre, és teljesítmény tesztek futtatása annak ellenőrzése, hogy a webes API-k cselekedjenek kielégítően működik. Webalkalmazás teljesítmény- és tesztelése a projekt betöltésekor Visual Studio Ultimate használatával hozhat létre. További információkért lásd: [teljesítmény vizsgálat futtatása előtt egy alkalmazás](https://msdn.microsoft.com/library/dn250793.aspx).

## <a name="using-azure-api-management"></a>Az Azure API Management használata 

A Azure-érdemes [Azue API Management](https://azure.microsoft.com/documentation/services/api-management/) közzétételét és kezelését egy webes API-t. Ez a funkció segítségével is létrehozhat olyan szolgáltatás, amely egy vagy több Web API-k egy homlokzati funkcionál. A szolgáltatás pedig egy méretezhető webszolgáltatás, amelyet hozhat létre és konfigurálja az Azure felügyeleti portál használatával. Ez a szolgáltatás segítségével közzétehet és kezelhet a webes API-k az alábbiak szerint:

1. Egy webhely, az Azure cloud service vagy Azure virtuális gép központi telepítése a webes API-t.
2. Az API management szolgáltatás kapcsolódni a webes API-t. A felügyeleti API URL-címre küldött kérelmeket a webes API URI-azonosítók van leképezve. Az azonos API management szolgáltatás irányíthatja a kérelmek egynél több webes API-hoz. Ez lehetővé teszi, hogy több webes API-k az egyetlen felügyeleti szolgáltatást összesíteni. Ehhez hasonlóan az azonos web API lehet hivatkozni több API management szolgáltatás Ha korlátozhatja, vagy a különböző alkalmazások funkcióinak partícióazonosító kell.

   > [!NOTE]
   > Az URI-azonosítók a HTTP GET kérelmek válasz részeként létrehozott HATEOAS hivatkozások hivatkoznia kell az API management szolgáltatás és nem a webes API-t üzemeltető webkiszolgáló URL-CÍMÉT.
   >
   >
3. Minden webes API-t, adja meg, amely a webes API-t és a választható paraméterek: egy művelet által végrehajtható elérhetővé teszi a HTTP-műveletek bemeneti adatként. Is beállítható, hogy az API management szolgáltatás kell gyorsítótárazza a választ, a webes API ismétlődő kérelmek az adatok optimalizálása kapott. A minden művelet hozhat létre a HTTP-válasz részletes adatait rögzíti. Az adatok létrehozásához a dokumentáció a fejlesztők számára, ezért fontos, hogy a rendszer pontos és teljes.

   Megadhat műveletek az Azure felügyeleti portál által szolgáltatott varázslók segítségével manuálisan, vagy importálhatja azokat a definíciók WADL vagy Swagger formátumú tartalmazó fájlt.
4. A biztonsági beállítások az API management szolgáltatás és a webkiszolgáló a webes API-k közötti kommunikáció céljára. Az API management szolgáltatás jelenleg az egyszerű hitelesítés és a kölcsönös hitelesítést tanúsítványokkal, és az OAuth 2.0 felhasználói engedélyezési támogatja.
5. Hozzon létre egy termék. A termék a egység kiadvány; a webes API-t a kezelési szolgáltatást, hogy a termék korábban csatolva hozzá. A termék meg van nyitva, a webes API-k elérhetővé válnak a fejlesztők számára.

   > [!NOTE]
   > Közzététel a termék, mielőtt-felhasználócsoportokat, amelyek a termék elérheti és felhasználók hozzáadása az ezeket a csoportokat is meghatározhatja. A felügyelhető a fejlesztők és a webes API-t használó alkalmazásokban. Ha a webes API jóvá, nem tudnak hozzáférni az előtt egy fejlesztő kérést kell küldenie a termék rendszergazdájának. A rendszergazda is hozzáférés engedélyezése vagy megtagadása a fejlesztőnek. Meglévő fejlesztők is blokkolható, ha olyan körülmények között módosíthatja.
   >
   >
6. Minden webes API-szabályzatok konfigurálása Házirendek szabályozására, például hogy a tartományok közötti hívások engedélyezni kell, hogyan hitelesíti az ügyfeleket, hogy XML és JSON adatok formázza transzparens módon, hogy egy adott IP-címtartomány hívásait korlátozza-e szempontok használati kvóták, és korlátozhatja a hívás sebessége. Házirendek globálisan alkalmazhatók a teljes termék, a termék egy webes API-t vagy webes API-k az egyes műveletek között.

További információkért lásd: a [API Management dokumentációja](/azure/api-management/). 

> [!TIP]
> Azure biztosít az Azure Traffic Manager, ami lehetővé teszi a feladatátvételi és terheléselosztási megvalósítása, és a késés csökkentésére különböző földrajzi helyeken található üzemeltetett webhely több példánya között. Az API Management szolgáltatással; együtt használhatja az Azure Traffic Manager az API Management Service egy webhely keresztül Azure Traffic Manager-példányokban kívánja irányíthatja a kérelmeket.  További információkért lásd: [Traffic Manager útválasztási metódusok](/azure/traffic-manager/traffic-manager-routing-methods/).
>
> Ez a struktúra a webhelyek, az egyéni DNS-nevek használatakor konfigurálnia kell a megfelelő CNAME rekordot a DNS-nevével, az Azure Traffic Manager webhely ponthoz minden webhelyhez.
>

## <a name="supporting-client-side-developers"></a>Ügyféloldali fejlesztők támogatása
A fejlesztők ügyfélalkalmazások általában hozhat létre a web API, és a paraméterek, a adattípusok, a visszatérési típusok és a visszatérési kódokat, amelyek bemutatják a különböző kérelmek és között a webes válaszok vonatkozó dokumentációt elérésével ismereteket igényel szolgáltatás és az ügyfélalkalmazást.

### <a name="document-the-rest-operations-for-a-web-api"></a>A webes API-k REST műveleteinek dokumentálása
Az Azure API Management Service tartalmaz, amely leírja a REST-műveletek webes API-k által elérhetővé tett egy fejlesztői portálján. A termék is közzé lett téve jelenik meg ezen a portálon. A fejlesztők is használhatja e portált Feliratkozás a hozzáférést. a rendszergazda ezután jóváhagyhatja vagy visszautasítja a kérelmet. Ha a fejlesztői jóvá van hagyva, rendelt való hitelesítéshez szükséges hívások az ügyfélalkalmazások számára, ők fejlesztik ki az előfizetés kulcsa. Ezt a kulcsot meg kell adni minden webes API-hívással egyébként elutasításra kerül.

Ezen a portálon is biztosítja:

* Műveletek a érheti el, a szükséges paraméterek és a különböző válaszokat visszaadható listázása a termék dokumentációját. Vegye figyelembe, hogy ezek az információk jönnek létre a 3. lépésben a közzétételi listájában megadott adatok egy webes API-t a Microsoft Azure API Management Service szakasz segítségével.
* Az kódrészletek, amelyek bemutatják, hogyan több nyelven, beleértve a JavaScript, C#, Java, Ruby, Python, és a PHP műveleteket hívhatnak meg.
* A fejlesztők konzol, amely lehetővé teszi a fejlesztők a termékben lévő egyes műveletek tesztelése, és az eredmények megtekintése a HTTP-kérelem küldése.
* Egy oldal, ahol a fejlesztői jelenthetik-e bármilyen problémák vagy a talált problémákat.

Az Azure felügyeleti portálra lehetővé teszi a fejlesztői portálján stíluselemekkel és az elrendezés felel meg a szervezet branding módosítása testreszabását.

### <a name="implement-a-client-sdk"></a>Egy ügyfél SDK megvalósítása
Egy ügyfélalkalmazás létrehozása, amely meghívja a többi hozzáférés kérése a webes API megköveteli, hogy minden kérelmet létrehozni és formázza megfelelően, a webszolgáltatást futtató kiszolgálón a kérelem elküldése és kidolgozására válasz elemzése kód jelentős mennyiségű írása a kérelem e sikeres vagy sikertelen volt, és minden visszaadott adatok kinyerése érdekében. Az ügyfélalkalmazás az a problémák elleni védekezésben, hogy a REST-felületet burkolja és kivonatolja a kevésbé fontos részletek belül több működési készlete módszerek SDK biztosíthat. Egy ügyfélalkalmazás ezek a módszerek, amely transzparens módon hívások konvertálása a REST-kérelmek és alakítsa át a válaszok metódus visszatérési értékek programba. Ez a sok szolgáltatásokat, például az Azure SDK által megvalósított közös technika.

Jelentős vállalkozás egy ügyféloldali SDK létrehozása esetén, mert van egységesen kell végrehajtani, alaposan tesztelni. Azonban a folyamat nagy része gépi tehető, és sok szállítók eszközöket, amelyek a automatizálhat ezek a feladatok fogja tartalmazni.

## <a name="monitoring-a-web-api"></a>Webes API-k figyelése
Attól függően, hogy hogyan közzététele és telepítése a webes API-t is figyelheti a webes API-k közvetlenül, vagy használati és adatokat gyűjthet a forgalomhoz, amely az API Management szolgáltatáson keresztül elemzésével.

### <a name="monitoring-a-web-api-directly"></a>Webes API-k közvetlenül figyelése
Ha a webes API-t az ASP.NET Web API-sablont (vagy a Web API-projektet, vagy egy Azure-felhőszolgáltatásban a webes szerepkör) és a Visual Studio 2013 megvalósítását, gyűjthet rendelkezésre állását, teljesítményét és használati adatok ASP.NET Application Insights segítségével. Az Application Insights egy csomagot, amely transzparens módon nyomon követi, és információt kérelmeit és válaszait rögzíti, amikor a webes API-t a rendszer a felhő; Ha a csomag telepítve és konfigurálva van, nem kell módosítani kell a kódot a webes API-t használja. A webes API-t az Azure-webhelyre történő telepítésekor az összes forgalom vizsgálják, és a következő adatokat gyűjti az adatokat:

* Kiszolgáló válaszideje.
* Kiszolgáló kérések számát és az egyes kérelmek részletei.
* A felső leglassabb kérelmek átlagos válaszidő tekintetében.
* A sikertelen kérelmek részleteit.
* A munkamenetek számának a különböző böngészők és a felhasználó által kezdeményezett ügynökök.
* A legtöbb gyakran megtekintett lapok (elsősorban akkor hasznos, webes alkalmazásokhoz, hanem a webes API-khoz).
* A különböző felhasználói szerepköröket a webes API-k eléréséhez.

Valós idejű az Azure felügyeleti portálon megtekintheti ezeket az adatokat. Webtests, amely a webes API-k állapotának figyelésére is létrehozhat. A webtesztben rendszeres kérést küld a megadott URI-azonosító található a webes API-t, és a válasz rögzíti. A sikeres válasz (például a HTTP-állapotkód 200) definícióját is megadhat, és ha a kérelem nem ad vissza ezt a választ egy rendszergazda küldendő riasztás elrendezheti. Ha szükséges, a rendszergazda újraindíthatja a kiszolgáló, a webes API-t futtató, ha sikertelen volt.

További információkért lásd: [ASP.NET Application Insights – első lépések](/azure/application-insights/app-insights-asp-net/).

### <a name="monitoring-a-web-api-through-the-api-management-service"></a>Az API Management szolgáltatáson keresztül webes API-k figyelése
Ha a webes API-t az API Management szolgáltatással közzétett, az API Management oldalon, az Azure felügyeleti portálra tartalmaz egy irányítópultot, amely lehetővé teszi az általános teljesítményt, a szolgáltatás megtekintését. Az Analytics lap lehetővé teszi a részletekbe menően tárhatják fel a termék felhasználási módjának részleteit. Ezen a lapon az alábbi lapokat tartalmazza:

* **Használati**. Ezen a lapon API-hívások és a sávszélesség adott idő alatt az adott hívások kezelésére használt információkat tartalmaz. Szűrheti a használat részleteiről termék, az API és a művelet.
* **Állapotfigyelő**. Ezen a lapon megtekintheti az API-kérelmek (a HTTP-állapotkódok visszaadott) eredményeit, a gyorsítótárazási házirendet, az API válaszidő és a szolgáltatás válaszidejének hatékonyságát teszi lehetővé. Ebben az esetben szűrheti állapotadatok termék, az API és a művelet.
* **Tevékenység**. Ezen a lapon a sikeres hívás, sikertelen hívás, letiltott hívások, átlagos válaszidő és minden egyes termék, a webes API és a működéséhez válaszidők számú szöveg összegzését tartalmazza. Ezen a lapon is találhatók minden egyes fejlesztő felé indított hívások száma.
* **Egy pillanat alatt**. Ezen a lapon a teljesítményadatokat, beleértve a fejlesztők feladata, hogy a legtöbb API-hívások, és a termékek, a webes API-k és a fogadott hívásokat a műveletek összegzését jeleníti meg.

Ezek az információk segítségével határozza meg, hogy egy adott webes API-t vagy a művelet hatására a szűk keresztmetszetek, és ha szükséges méretezni a gazdagép-környezetben, és adjon hozzá további kiszolgálókat. Akkor is megállapítása, hogy egy vagy több alkalmazást le aránytalanul nagy mennyiségű erőforrást használ, és a megfelelő házirendekkel kvótáit és hívás díjszabás korlátozza.

> [!NOTE]
> Módosíthatja a közzétett termék adatait, és a módosítások azonnal. Például vegye fel vagy távolítsa el a művelet a webes API-k anélkül, hogy újra közzé a terméket, amely tartalmazza a webes API-t.
>
>

## <a name="more-information"></a>További információ
* [Az ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) példák és további információkat tartalmaz az OData webes API-k végrehajtási ASP.NET használatával.
* [Introducing kötegelt támogatása a webes API- és webes API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) kötegműveletek megvalósítását a webes API-k használatával OData ismerteti.
* [Idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathan Oliver blogjában áttekintést idempotencia, és hogyan vonatkozik az felügyeleti műveleteket.
* [Állapotkódok definícióit](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) a W3C a webhely HTTP-állapotkódok és azok leírásait tartalmazza a teljes listáját tartalmazza.
* [Háttérfeladatok futtatása a webjobs-feladatok](/azure/app-service-web/web-sites-create-web-jobs/) útmutatást és példákat nyújt WebJobs használata a háttérbeli műveletek végrehajtásához.
* [Az Azure Notification Hubs – felhasználók értesítése](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) bemutatja, hogyan használja az Azure Notification Hub leküldéses aszinkron válaszok ügyfélalkalmazások számára.
* [Az API Management](https://azure.microsoft.com/services/api-management/) egy webes API-k felügyelt és biztonságos hozzáférést biztosító termék közzétételét ismerteti.
* [Az Azure API Management REST API-referencia](https://msdn.microsoft.com/library/azure/dn776326.aspx) az API Management REST API használata egyéni felügyeleti alkalmazásokat hozhatnak létre.
* [A TRAFFIC Manager útválasztási metódusok](/azure/traffic-manager/traffic-manager-routing-methods/) összefoglalja, hogyan Azure Traffic Manager segítségével terheléselosztásához kérelmek egy webes API-t futtató webhely több példánya között.
* [Az Application Insights – első lépések ASP.NET](/azure/application-insights/app-insights-asp-net/) telepítését és konfigurálását egy ASP.NET Web API-projektet az Application Insights részletes információkat tartalmaz.


<!-- links -->

[api-design]: ./api-design.md
