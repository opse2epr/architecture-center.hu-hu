---
title: API-implementálási segédlet
titleSuffix: Best practices for cloud applications
description: Útmutató az API-k megvalósításához.
author: dragon119
ms.date: 07/13/2016
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: aa69746b59ddfff02381dd811caa9a7aa62d8b7b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484805"
---
# <a name="api-implementation"></a>API-implementáció

A gondosan megtervezett RESTful webes API-k meghatározzák az ügyfélalkalmazások számára elérhető erőforrásokat, kapcsolatokat és navigációs rendszereket. A webes API-k megvalósításakor és központi telepítésekor a webes API-t futtató környezet fizikai követelményeit és a webes API felépítését kell figyelembe vennie az adatok logikai szerkezete helyett. Ez az útmutató ajánlott eljárásokat tartalmaz a webes API-k megvalósításához és közzétételéhez, hogy elérhetők legyenek az ügyfélalkalmazások számára. Részletes információ a webes API-k tervezéséről: [API-tervezési segédlet](/azure/architecture/best-practices/api-design).

## <a name="processing-requests"></a>Kérelmek feldolgozása

A kéréseket kezelő kód megvalósításakor vegye figyelembe az alábbi szempontokat:

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a>A GET, PUT, DELETE, HEAD és JAVÍTÁSI műveleteknek idempotensnek kell lenniük

A kérelmeket megvalósító kódnak nem lehetnek mellékhatásai. Ha egy kérést megismétel ugyanazon az erőforráson, a kérésnek ugyanazt az állapotot kell eredményeznie. Például egy adott URI-re küldött több DELETE kérelem eredményének ugyanannak kell lennie, habár a válaszüzenetekben található HTTP-állapotkódok különbözők lehetnek. Lehetséges, hogy az első törlési kérelem a 204 (nincs tartalom) állapotkódot, a további törlési kérelmek pedig a 404 (nem található) állapotkódot adják vissza.

> [!NOTE]
> Jonathan Oliver blogjának [idempotenciamintákról](https://blog.jonathanoliver.com/idempotency-patterns/) szóló cikke áttekintést nyújt idempotenciáról, és hogy az hogyan kapcsolódik az adatkezelési műveletekhez.

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a>Az új erőforrásokat létrehozó POST műveleteknek nem lehetnek független mellékhatásai

Ha egy POST kérés célja egy új erőforrás létrehozása, a kérés csak az új erőforrásra lehet hatással (valamint esetleg a közvetlenül kapcsolódó erőforrásokhoz, ha vannak ilyenek). Például egy elektronikus kereskedelmi rendszerben egy ügyfél számára egy új erőforrást létrehozó POST kérés módosíthatja a készletszinteket és létrehozhat számlázási adatokat, de nem módosíthatja a rendeléshez nem közvetlenül kapcsolódó információkat, és nem lehet semmilyen más mellékhatása a rendszer általános állapotára.

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a>A POST, PUT és DELETE műveletek ne legyenek hosszadalmasak

Biztosítson támogatást a POST, PUT és DELETE kérések teljes erőforrás-gyűjteményeken való végrehajtásához. Egy POST kérés tartalmazhatja több új erőforrás részleteit is, amelyeket ugyanahhoz a gyűjteményhez ad hozzá. A PUT kérések egy gyűjtemény teljes erőforráskészletét lecserélhetik, míg a DELETE kérések egy teljes gyűjteményt eltávolíthatnak.

Az ASP.NET Web API 2-ben található OData-támogatás lehetővé teszi a kérések kötegelését. Az ügyfélalkalmazások becsomagolhatnak több webes API-kérést és egyetlen HTTP-kérésben küldhetik el őket a kiszolgálóra, majd egyetlen HTTP-választ kapnak vissza, amely az összes kérésre vonatkozó választ tartalmazza. További információ: [A kötegelés támogatásának bevezetése a webes API-ban és a webes API ODatában](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/).

### <a name="follow-the-http-specification-when-sending-a-response"></a>Kövesse a HTTP-specifikációkat válasz küldésekor

A webes API-nak olyan üzeneteket kell visszaadnia, amelyek tartalmazzák a megfelelő HTTP-állapotkódot, amelyek alapján az ügyfél el tudja dönteni, hogyan kezelje az eredményt. Emellett tartalmazniuk kell még a megfelelő HTTP-fejléceket, hogy az ügyfél értse az eredmény jellegét, valamint egy megfelelően formázott törzset, amely alapján az ügyfél elemezheti az eredményt.

Például a POST műveletnek a 201 (Létrehozva) állapotkódot kell visszaadnia, a válaszüzenetnek pedig bele kell foglalnia kell az újonnan létrehozott erőforrás URI-jét a válaszüzenet Location fejlécébe.

### <a name="support-content-negotiation"></a>A tartalomegyeztetés támogatása

A válaszüzenetek törzse többféle formátumú adatokat tartalmazhat. Például egy HTTP GET kérés JSON vagy XML formátumú adatokat is visszaadhat. Az ügyfél által küldött kérés tartalmazhat egy Accept fejlécet, amelyben megadja, hogy milyen adatformátumokat tud kezelni. Ezek a formátumok médiatípusként vannak megadva. Egy ügyfél kibocsát egy képet lekérő GET kérést megadhatja például, egy Accept fejlécet, amely felsorolja az ügyfél képes kezelni, például adathordozó-típusok `image/jpeg, image/gif, image/png`. Amikor a webes API visszaadja az eredményt, az adatokat a felsorolt médiatípusok egyikével kell formáznia, és meg kell adnia a formátumot a válasz Content-Type fejlécében.

Ha az ügyfél nem adott meg Accept fejlécet, akkor az API egy kézenfekvő alapértelmezett formátumot használ a választörzsben. Az ASP.NET webes API-keretrendszer például alapértelmezés szerint a JSON formátumot használja a szöveges adatokhoz.

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a>Adjon meg hivatkozásokat a HATEOAS stílusú navigáció és az erőforrás-felderítés támogatásához

A HATEOAS módszer lehetővé teszi az ügyfelek számára, hogy egy kezdeti kiindulási pontról navigáljanak és derítsék fel az erőforrásokat. Ez URI-ket tartalmazó hivatkozásokkal történik. Amikor egy ügyfél kiad egy HTTP GET kérést egy erőforrás beszerzéséhez, akkor a válasznak tartalmaznia olyan URI-kat kell tartalmaznia, amelyekkel az ügyfélalkalmazás gyorsan megtalálhat bármilyen közvetlenül kapcsolódó erőforrást. Például vegyünk egy elektronikus kereskedelmi megoldást támogató webes API-t, amelyben egy ügyfél számos megrendelést adott le. Amikor egy ügyfélalkalmazás lekéri az ügyfél adatait, a válasznak olyan hivatkozásokat kell tartalmaznia, amelyek alapján az ügyfélalkalmazás el tudja küldeni a HTTP GET kéréseket a rendelés lekéréséhez. Emellett a kérések végrehajtásához HATEOAS stílusú hivatkozásokkal le kell írnia az egyes hivatkozott erőforrások által támogatott egyéb műveleteket (POST, PUT, DELETE stb.) a megfelelő URI-kkel. A módszer részletesebb leírását lásd: [API-tervezés][api-design].

Jelenleg nincsenek a HATEOAS megvalósítására vonatkozó szabványok, de a következő példa bemutat egy lehetséges módszert. Ebben a példában egy HTTP GET kérés, amely megkeresi az ügyfél adatait, egy olyan választ ad vissza, amely az ügyfél rendeléseire mutató HATEOAS-hivatkozásokat tartalmaz.

```HTTP
GET https://adventure-works.com/customers/2 HTTP/1.1
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
    "href":"https://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"https://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"https://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"https://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"https://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

Ebben a példában a felhasználói adatokat a `Customer` osztály jelöli a következő kódrészletben. A HATEOAS-hivatkozásokat a `Links` gyűjteménytulajdonság tartalmazza:

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

A HTTP GET művelet lekérdezi a felhasználói adatokat a tárolóból, és létrehoz egy `Customer` objektumot, majd feltölti a `Links` gyűjteményt. Az eredményt egy JSON-válaszüzenet formájában adja vissza a rendszer. Mindegyik hivatkozás a következő mezőket tartalmazza:

- A visszaadott objektum és hivatkozásban leírt objektum közötti kapcsolat. Ebben az esetben `self` azt jelzi, hogy a hivatkozás magára az objektumra egy hivatkozást (hasonlóan egy `this` számos objektumorientált nyelv mutatójához), és `orders` a kapcsolódó rendelési adatokat tartalmazó gyűjtemény neve.
- A hivatkozás által leírt objektum hiperhivatkozása (`Href`) egy URI formájában.
- Az URI felé küldhető HTTP-kérelem típusa (`Action`).
- Azon adatok formátuma (`Types`), amelyeket meg kell adni a HTTP-kérelemben, vagy amelyeket a válasz visszaadhat, a kérelem típusától függően.

A példa HTTP-válaszban található HATEOAS-hivatkozások azt jelzik, hogy egy ügyfélalkalmazás a következő műveleteket hajthatja végre:

- Egy HTTP GET kérelem a `https://adventure-works.com/customers/2` URI felé, amely (újra) lekéri az ügyfél adatait. Ezek az adatok XML vagy JSON formátumban adhatók vissza.
- Egy HTTP PUT kérelem a `https://adventure-works.com/customers/2` URI felé, amely módosítja az ügyfél adatait. Az új adatokat a kérésüzenetben x-www-form-urlencoded formátumban kell megadni.
- Egy HTTP DELETE kérelem a `https://adventure-works.com/customers/2` URI felé, amely törli az ügyfelet. A kérelem nem vár semmilyen további információt, és nem ad vissza semmilyen adatot a válaszüzenet törzsében.
- Egy HTTP GET kérelem a `https://adventure-works.com/customers/2/orders` URI felé, amely lekéri az ügyfél összes rendelését. Ezek az adatok XML vagy JSON formátumban adhatók vissza.
- Egy HTTP PUT kérelem a `https://adventure-works.com/customers/2/orders` URI felé, amely létrehoz egy új rendelést az ügyfélhez. Az adatokat a kérésüzenetben x-www-form-urlencoded formátumban kell megadni.

## <a name="handling-exceptions"></a>Kivételek kezelése

Ha egy művelet nem kezelt kivételt jelez, vegye figyelembe a következő szempontokat.

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a>Kivételek rögzítése és jelentéssel bíró válasz visszaadása az ügyfeleknek

A HTTP-műveleteket megvalósító kódnak átfogó kivételkezelést kell biztosítania ahelyett, hogy engedné a nem kezelt kivételek propagálását a keretrendszerbe. Ha egy kivétel megakadályozza a művelet sikeres elvégzését, a kivétel visszaadható a válaszüzenetben, de ez esetben jelentéssel bíró leírást is kell adnia a kivételt okozó hibáról. A kivételnek tartalmaznia a megfelelő HTTP-állapotkódot is ahelyett, hogy minden esetben egyszerűen az 500-as állapotkódot adná vissza. Ha például egy felhasználó kérés egy olyan frissítést vált ki, amely megsért egy korlátozást (például megkísérel törölni egy nyitott rendelésekkel rendelkező ügyfelet), akkor a 409 (Ütközés) állapotkódot kell visszaadni, valamint az üzenet törzsében jelezni kell ütközés okát. Ha valamilyen más körülmény akadályozza a kérés teljesítését, akkor a 400 (Hibás kérelem) állapotkód is visszaadható. A HTTP-állapotkódok teljes listáját megtalálhatja a W3C webhely [Állapotkódok definíciói](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) lapján.

A kódpélda rögzíti a különböző körülményeket, és megfelelő választ ad vissza.

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
> Ne adjon meg olyan információt, amelyet egy támadó felhasználhat az API-ba való behatoláshoz.
  
Sok webkiszolgáló maga rögzíti a hibák körülményeit, mielőtt a hibák elérnék a webes API-t. Ha például egy webhelyen hitelesítés van konfigurálva, és a felhasználó nem adja meg a megfelelő hitelesítési adatokat, a webkiszolgálónak a 401 (Nem engedélyezett) állapotkódot kell visszaadnia. Az ügyfél hitelesítése után a kód a saját ellenőrzéseivel győződhet meg róla, hogy az ügyfélnek hozzáféréssel kell rendelkeznie a kért erőforráshoz. Ha a hitelesítés sikertelen, a 403 (Tiltott) állapotkódot kell visszaadni.

### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a>A kivételek egységes kezelése és a hibákkal kapcsolatos adatok naplózása

A kivételek egységes kezelése érdekében érdemes egy globális hibakezelési stratégiát bevezetnie a teljes webes API-n. Emellett érdemes hibanaplózással rögzítenie az egyes kivételek összes részletét. A hibanapló bármilyen részletes információt tartalmazhat azzal a feltétellel, hogy az ügyfelek a weben keresztül nem férhetnek hozzá.

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a>Az ügyféloldali és kiszolgálóoldali hibák megkülönböztetése

A HTTP-protokoll megkülönbözteti az ügyfélalkalmazás által okozott (HTTP 4xx állapotkódú) és a kiszolgáló hibái miatt előforduló (HTTP 5xx állapotkódú) hibákat. Ügyeljen arra, hogy a hibákkal kapcsolatos válaszüzenetekben betartsa ezt a konvenciót.

## <a name="optimizing-client-side-data-access"></a>Ügyféloldali adatelérés optimalizálása

Az elosztott környezetekben, például amelyekben egy webkiszolgáló és ügyfélalkalmazások találhatók, az egyik elsődleges hibaforrás a hálózat. Ez jelentős szűk keresztmetszeteket okozhat, különösen akkor, ha egy ügyfélalkalmazás gyakran küld kérelmeket vagy fogad adatokat. Ezért törekedni kell a hálózaton keresztül zajló forgalom minimalizálására. Az adatok lekérését és karbantartását kezelő kód megvalósításakor vegye figyelembe az alábbi szempontokat:

### <a name="support-client-side-caching"></a>Az ügyféloldali gyorsítótárazás támogatása

A HTTP 1.1 protokoll támogatja a gyorsítótárazást az ügyfeleken és köztes kiszolgálókon, amelyeken egy Cache-Control fejléc irányítja a kérelmeket. Amikor egy ügyfélalkalmazás egy HTTP GET kérést küld a webes API-nak, a válasz tartalmazhat egy Cache-Control fejlécet, amely jelzi, hogy a válasz törzsében található adatok biztonságosan gyorsítótárazhatók-e az ügyfélnél vagy egy köztes kiszolgálón, amelyen a kérelem áthaladt, valamint megadja, hogy az adatok mennyi idő után járnak le és válnak elavulttá. A következő példa bemutat egy HTTP GET kérelmet és a rá adott választ, amely tartalmaz egy Cache-Control fejlécet:

```HTTP
GET https://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

Ebben a példában a Cache-Control fejléc megadja, hogy a visszaadott adatok 600 másodperc után járnak le, csak egyetlen ügyfél számára megfelelők és tilos őket más ügyfelek által is használt megosztott gyorsítótárakban tárolni (vagyis *private* (bizalmas) adatokról van szó). A Cache-Control fejléc a *private* helyett megadhat *public* (nyilvános) beállítást is, amely esetben az adatok tárolhatók egy megosztott gyorsítótárban, vagy megadhatja a *no-store* (nincs tárolás) értéket, amely esetben az adatok **nem** gyorsítótárazhatók az ügyfélben. Az alábbi példakód bemutatja, hogyan hozható létre egy Cache-Control fejléc a válaszüzenetben:

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

Ez a kód egy `OkResultWithCaching` nevű egyéni `IHttpActionResult` osztályt használ. Ez az osztály lehetővé teszi a vezérlőnek a gyorsítótárfejléc tartalmának beállítását:

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
> A HTTP-protokoll a *no-cache* direktívát is meghatározza a Cache-Control fejléchez. Megtévesztő módon ez nem azt jelenti, hogy az adatokat nem lehet gyorsítótárazni, hanem hogy a gyorsítótárazott adatokat ellenőriztetni kell a kiszolgálóval a visszaadás előtt – tehát az adatok gyorsítótárazhatók, de minden használatkor ellenőrizni kell, hogy még aktuálisak-e.

A gyorsítótár kezelése az ügyfélalkalmazás vagy a köztes kiszolgáló feladata, de ha megfelelően van megvalósítva, akkor a használatával sávszélesség takarítható meg és növelhető a teljesítmény, mivel nincs szükség a már beszerzett adatok újbóli lekérésére.

A Cache-Control fejléc *max-age* (maximális kor) értéke csak tájékoztató jellegű, és nem garantálja, hogy a vonatkozó adatok időközben nem változnak. A webes API-nak úgy kell beállítania a max-age értékét, hogy tükrözze az adatok várható érvényességét. Ha ez az időszak lejár, az ügyfélnek törölnie kell az objektumot a gyorsítótárból.

> [!NOTE]
> A legtöbb modern böngésző támogatja az ügyféloldali gyorsítótárazást azáltal, hogy az ismertetett módon gyorsítótár-vezérlő fejléceket ad hozzá a kérelmekhez, és megvizsgálja az eredmények fejlécét. Egyes régebbi böngészők azonban nem gyorsítótárazzák az olyan URL-címekről visszakapott adatokat, amelyek lekérdezési sztringet tartalmaznak. Ez általában nem okoz problémát az olyan egyéni ügyfélalkalmazások számára, amelyek az itt tárgyalt protokollon alapuló saját gyorsítótár-kezelési stratégiával rendelkeznek.
>
> Egyes régebbi proxyk ugyanígy viselkednek, és lehetséges, hogy nem gyorsítótárazzák a lekérdezési sztringekat tartalmazó URL-címeken alapuló kérelmeket. Ez az olyan egyedi ügyfélalkalmazások számára jelenthet problémát, amelyek egy ilyen proxyn keresztül csatlakoznak egy webkiszolgálóhoz.

### <a name="provide-etags-to-optimize-query-processing"></a>A lekérdezés feldolgozásának optimalizálása ETagek megadásával

Amikor egy ügyfélalkalmazás lekér egy objektumot, a válaszüzenet egy *ETaget* (entitáscímkét) is tartalmazhat. Az ETag egy olyan átlátszatlan sztring, amely egy erőforrás verzióját jelzi, és az erőforrás minden változásakor módosul. Az ügyfélalkalmazásnak az adatokkal együtt az ETaget is gyorsítótáraznia kell. Az alábbi példakód bemutatja, hogyan lehet hozzáadni egy ETaget egy HTTP GET kérelemre adott válaszhoz. Ez a kód a `GetHashCode` metódussal létrehoz egy numerikus értéket, amely azonosítja az objektumot (ha szükséges, felülírhatja ezt a metódust, és létrehozhat egy saját kivonatot az MD5 vagy más algoritmus segítségével):

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

A webes API által közzétett válaszüzenet így néz ki:

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
> Biztonsági okokból ne engedélyezze a bizalmas adatok és a hitelesített (HTTPS-) kapcsolaton keresztül visszaadott adatok gyorsítótárazását.

Egy ügyfélalkalmazás egy újabb GET kéréssel bármikor újra lekérheti ugyanazt az erőforrást, és ha az erőforrás változott (ha más az ETagje), akkor a gyorsítótárazott verziót el kell távolítani, és az új verziót kell hozzáadni a gyorsítótárhoz. Ha egy erőforrás nagy méretű, és az ügyfélnek való visszaküldése jelentős sávszélességet igényel, akkor nem hatékony több kérelmet küldeni ugyanannak az adatnak a lekérése céljából. A HTTP-protokoll ennek elkerülése érdekében a következő folyamatot határozza meg a GET kérelmek optimalizálásához, amelyet a webes API-knak támogatniuk kell:

- Az ügyfél létrehoz egy GET kérelmet, amely az erőforrás jelenleg gyorsítótárazott verziójának ETagjét tartalmazza, és egy If-None-Match HTTP-fejlécben hivatkozik rá:

    ```HTTP
    GET https://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```

- A webes API GET művelete lekéri a kért adatok jelenlegi ETag címkéjét (2. rendelés a fenti példában), és összehasonlítja az értéket az If-None-Match fejlécben találhatóval.

- Ha a kért adatok jelenlegi ETagje megegyezik a kérés által megadott ETaggel, akkor az erőforrás nem változott, és a webes API-nak a HTTP-válaszban egy üres üzenettörzset és a 304 (Nem módosított) állapotkódot kell visszaadnia.

- Ha a kért adatok jelenlegi ETagje nem egyezik a kérés által megadott ETaggel, akkor az adatok változtak, és a webes API-nak a HTTP-válaszban az üzenettörzsben meg kell adnia az új adatokat és a 200 (OK) állapotkódot.

- Ha a kért adatok már nem léteznek, akkor a webes API-nak egy 404 (Nem található) állapotkódú HTTP-választ kell visszaadnia.

- Az ügyfél az állapotkódot használja a gyorsítótár karbantartásához. Ha az adatok nem változtak (304-es állapotkód), akkor az objektum a gyorsítótárban maradhat, és ügyfélalkalmazás tovább használhatja az objektum jelenlegi verzióját. Ha az adatok megváltoztak (200-as állapotkód), akkor a gyorsítótárazott objektumot el kell távolítani, és egy újat kell beszúrni helyette. Ha az adatok már nem érhetők el (404-es állapotkód), akkor az objektumot el kell távolítani a gyorsítótárból.

> [!NOTE]
> Ha a válasz fejléce a no-store Cache-Control fejlécet tartalmazza, akkor az objektumot mindenképp el kell távolítani a gyorsítótárból, a HTTP-állapotkódtól függetlenül.

Az alábbi kód az If-None-Match fejléc támogatásához kibővített `FindOrderByID` metódust mutatja. Figyelje meg, hogy ha az If-None-Match fejléc hiányzik, a rendszer mindig lekéri a megadott rendelést:

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

Ebben a példában egy `EmptyResultWithCaching` nevű másik egyéni `IHttpActionResult` osztály is található. Ez az osztály egyszerű csomagolásként szolgál egy `HttpResponseMessage` objektum körül, amely nem tartalmaz választörzset:

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
> Ebben a példában az adatok ETagje az alapul szolgáló adatforrásból lekért adatok kivonatolásával jön létre. Ha az ETag más módon is kiszámítható, akkor a folyamat tovább optimalizálható, és az adatokat csak akkor kell lekérni az adatforrásból, ha változtak. Ez a módszer különösen hasznos, ha az adatok mérete nagy, vagy az adatforrás elérése jelentős késést eredményezhet (például ha az adatforrás egy távoli adatbázis).

### <a name="use-etags-to-support-optimistic-concurrency"></a>Optimista párhuzamosság támogatása ETagek használatával

A korábban gyorsítótárazott adatok frissítésének engedélyezéséhez a HTTP-protokoll támogat egy optimista párhuzamossági stratégiát. Ha egy erőforrás beolvasása és gyorsítótárazása után az ügyfélalkalmazás elküld egy PUT vagy DELETE kérést az erőforrás módosítása vagy eltávolítása céljából, akkor a kérésnek tartalmaznia kell az ETagre hivatkozó If-Match fejlécet. A webes API ezen információk alapján határozza meg, hogy az erőforrást a lekérése óta módosította-e egy másik felhasználó, majd az alábbiak szerint visszaküldi a megfelelő választ az ügyfélalkalmazásnak:

- Az ügyfél létrehoz egy PUT kérelmet, amely tartalmazza az erőforrás új adatait és az erőforrás jelenleg gyorsítótárazott verziójának ETagjét, amelyre egy If-Match HTTP-fejléc hivatkozik. A következő példa egy rendelést frissítő PUT kérelmet mutat be:

    ```HTTP
    PUT https://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```

- A webes API PUT művelete lekéri a kért adatok jelenlegi ETag címkéjét (1. rendelés a fenti példában), és összehasonlítja az értéket az If-Match fejlécben találhatóval.

- Ha a kért adatok jelenlegi ETagje megegyezik a kérés által megadott ETaggel, akkor az erőforrás nem változott, és a webes API-nak el kell végeznie a frissítést, majd a sikeres frissítés után egy 204 (Nem módosított) HTTP-állapotkódú üzenetet kell visszaadnia. A válasz tartalmazhatja az erőforrás frissített verziójának Cache-Control és ETag fejléceit. A válasznak mindig tartalmaznia kell a Location fejlécet, amely az újonnan frissített erőforrás URI-jére hivatkozik.

- Ha a kért adatok jelenlegi ETagje nem egyezik a kérés által megadott ETaggel, akkor a lekérés óta egy másik felhasználó módosította az adatokat, a webes API-nak pedig a HTTP-válaszban egy üres üzenettörzset és a 412 (Nem teljesül az előfeltétel) állapotkódot kell visszaadnia.

- Ha a frissítendő erőforrás már nem létezik, akkor a webes API-nak egy 404 (Nem található) állapotkódú HTTP-választ kell visszaadnia.

- Az ügyfél az állapotkóddal és a válaszfejlécekkel tartja karban a gyorsítótárat. Ha az adatok frissültek (204-es állapotkód), akkor az objektum maradhat a gyorsítótárban (amennyiben a Cache-Control fejléc nem a no-store értéket tartalmazza), de az ETaget frissíteni kell. Ha az adatokat egy másik felhasználó módosította (412-es állapotkód) vagy nem találhatók (404-es állapotkód), akkor a gyorsítótárazott objektumot el kell vetni.

A következő példakód az Orders vezérlő PUT műveletének megvalósítását mutatja be:

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
> Az If-Match fejléc használata teljesen választható, és ha elhagyja, a webes API mindig megpróbálja frissíteni a megadott rendelést, akár vakon felülírva egy másik felhasználó által végrehajtott frissítést. Az elveszett frissítések miatti problémák elkerüléséhez mindig adjon meg egy If-Match fejlécet.

## <a name="handling-large-requests-and-responses"></a>Nagyméretű kérelmek és válaszok kezelése

Előfordulhat, hogy egy ügyfélalkalmazásnak olyan kérelmeket kell kiállítania vagy olyan adatokat kell fogadnia, amelyek mérete több MB (vagy még nagyobb). Ha az ügyfélalkalmazás ekkora mennyiségű adat átvitelére várakozik, előfordulhat, hogy nem válaszol. Ha jelentős mennyiségű adatot tartalmazó kérelmeket kezel, vegye figyelembe a következő szempontokat:

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a>Nagyméretű objektumokat tartalmazó kérelmek és válaszok optimalizálása

Bizonyos erőforrások nagyméretű objektumok lehetnek, vagy nagy mezőket, például képeket vagy más típusú bináris adatokat tartalmazhatnak. A webes API-nak támogatnia kell a streamelést az ilyen erőforrások fel- és letöltésének optimalizálásához.

A HTTP-protokoll biztosítja a darabolásos átvitel kódolási mechanizmusát, amellyel a nagyméretű adatobjektumok visszastreamelhetők egy ügyfélhez. Amikor az ügyfél elküld egy nagy objektumra vonatkozó HTTP GET kérést, a webes API képes a választ darabonkénti *adattömbök* formájában visszaküldeni egy HTTP-kapcsolaton keresztül. A válaszban található adatok hossza előfordulhat, hogy kezdetben ismeretlen lehet (Előfordulhat, hogy lehet létrehozott), így a webes API-t futtató kiszolgáló küldjön egy válaszüzenetet az egyes adattömbök, amely meghatározza a Transfer-Encoding: Darabolásos fejléc helyett egy Content-Length fejlécet. Az ügyfélalkalmazás egymás után kaphatja meg az egyes adattömböket, amelyekből felépíti a teljes választ. Az adatok átvitele akkor ér véget, amikor a kiszolgáló visszaküld egy nulla méretű végső tömböt.

Egyetlen kérelem is eredményezhet egy olyan nagyméretű objektumot, amely jelentős erőforrásokat használ fel. Ha a streamelési folyamat során a webes API azt állapítja meg, hogy egy kérelem adatmennyisége túllép valamilyen elfogadható korlátot, akkor megszakíthatja a műveletet, és visszaadhat egy válaszüzenetet a 413 (A kérelemegység túl nagy) állapotkóddal.

A hálózaton keresztül továbbított nagyméretű objektumok mérete HTTP-tömörítéssel minimalizálható. Ez a módszer segít csökkenteni a hálózati forgalmat és a kapcsolódó hálózati késést, viszont az ügyfélnek és a webes API-t futtató kiszolgálónak egyaránt további feldolgozást kell végeznie. Például egy tömörített adatok fogadását váró ügyfélalkalmazás belefoglalhat a kérésébe egy „Accept-Encoding: gzip” fejlécet (más adattömörítési algoritmusok is megadhatók). Ha a kiszolgáló támogatja a tömörítést, akkor a válaszban meg kell adnia a gzip formátumú tartalmat az üzenettörzsben, valamint az „Accept-Encoding: gzip” válaszfejlécet.

A tömörítés kombinálható a streameléssel: a streamelés előtt tömörítse az adatokat, és adja meg a gzip tartalomkódolást és a darabolt átviteli kódolást az üzenetfejlécekben. Vegye figyelembe, hogy egyes webkiszolgálók (például az Internet Information Server) beállíthatók úgy, hogy automatikusan tömörítsék a HTTP-válaszokat, függetlenül attól, hogy a webes API tömöríti-e az adatokat.

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a>Részleges válaszok megvalósítása az aszinkron műveleteket nem támogató ügyfelek felé

Az aszinkron streamelés alternatívájaként az ügyfélalkalmazás explicit módon megadhatja, hogy a nagy objektumok esetében tömbökben kéri az adatokat. Ezeket a tömböket részleges válaszoknak hívják. Az ügyfélalkalmazás egy HTTP HEAD kérést küld az objektum adatainak lekéréséhez. Ha a webes API támogatja a részleges válaszokat, akkor a HEAD kérésre adott válaszüzenetének tartalmaznia kell egy Accept-Ranges fejlécet és egy Content-Length fejlécet, amely az objektum teljes méretét jelzi, de üzenet törzsének üresnek kell lennie. Az ügyfélalkalmazás ezen információk alapján hoz létre több GET kérést, amelyekben megadja a fogadni kívánt bájttartományt. A webes API válaszüzenetének tartalmaznia kell a 206 (Részleges tartalom) HTTP-állapotkódot; egy Content-Length fejlécet, amely meghatározza, hogy a válaszüzenet törzsében mennyi adat található, valamint egy Content-Range fejlécet, amely jelzi, hogy az adatok az objektum mely részét képezik (például a 4000. bájttól a 8000. bájtig).

A HTTP HEAD-kérelmek és a részleges válaszok bővebb leírása: [API-tervezés][api-design].

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a>Kerülje a szükségtelen 100-Continue állapotüzenetek küldését az ügyfélalkalmazásban

Ha egy ügyfélalkalmazás nagy mennyiségű adatot készül küldeni egy kiszolgálóra, akkor először megállapíthatja, hogy a kiszolgáló egyáltalán hajlandó-e fogadni a kérést. Az adatküldés előtt az ügyfélalkalmazás elküldhet egy várt a HTTP-kérelem: 100-fejléc, egy Content-Length fejlécet, amely azt jelzi, hogy az adatokat, de az üzenettörzse üres mérete továbbra is. Ha a kiszolgáló hajlandó kezelni a kérést, akkor a válaszüzenetében a 100 (Folytatás) HTTP-állapotkódot kell megadnia. Az ügyfélalkalmazás ezután elküldheti a teljes kérést, az üzenet törzsében megadott adatokkal.

Az IIS használatával üzemeltetett szolgáltatás, a HTTP.sys illesztő automatikusan észleli, és kezeli a várt: 100-továbbra is fejléceket, mielőtt továbbítaná a kéréseket a webalkalmazásnak. Ez azt jelenti, hogy az alkalmazás kódjában valószínűleg nem lesznek láthatók ezek a fejlécek, és feltételezhető, hogy az IIS már kiszűrte az olyan üzeneteket, amelyeket nem megfelelőnek vagy túl nagynak talált.

A .NET-keretrendszer használatával hoz létre ügyfélalkalmazásokat, ha minden POST és PUT üzeneteket fogja először elküldheti az üzeneteket a várt: 100-továbbra is a fejlécek alapértelmezés szerint. A kiszolgálóoldalhoz hasonlóan a .NET-keretrendszer transzparens módon kezeli a folyamatot. Ez a folyamat azonban azzal jár, hogy minden POST és PUT kérés teljesítése két üzenetváltást igényel a kiszolgálóval, még a kis méretű kérések is. Ha az alkalmazás nem küld nagy mennyiségű adatot tartalmazó kérelmeket, akkor letilthatja ezt a funkciót, ha a `ServicePointManager` osztállyal hoz létre `ServicePoint` objektumokat az ügyfélalkalmazásban. A `ServicePoint` objektum a kiszolgálón lévő erőforrásokat azonosító URI-k séma- és gazdagéptöredékei alapján kezeli az ügyfél által a kiszolgálóval létrehozott kapcsolatokat. Ezután a `ServicePoint` objektum `Expect100Continue` tulajdonságát false (hamis) értékre állíthatja. A séma- és gazdagéptöredékeivel megegyező URI-keresztül az ügyfél által végrehajtott minden későbbi POST és PUT kérés a `ServicePoint` objektumot várt nélkül lesz elküldve: 100-fejlécek továbbra is. A következő kód bemutatja, hogyan konfigurálható egy `ServicePoint` objektum, amely az URI-knek küldött összes kérést `http` sémával és `www.contoso.com` gazdagéppel konfigurálja.

```csharp
Uri uri = new Uri("https://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

A statikus is beállíthat `Expect100Continue` tulajdonságát a `ServicePointManager` osztály ezt követően az összes adja meg ennek a tulajdonságnak az alapértelmezett érték a létrehozott [ServicePoint](/dotnet/api/system.net.servicepoint) objektumokat.

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a>Tördelés támogatása olyan kéréseknél, amelyek esetlegesen nagy számú objektumot adhatnak vissza

Ha egy gyűjtemény sok erőforrást tartalmaz, egy GET kérés eljuttatása a megfelelő URI-hez jelentős mértékű feldolgozást igényelhet a webes API-t üzemeltető kiszolgálón, ami hatással lehet a teljesítményre és nagy mennyiségű hálózati forgalommal járhat, ez pedig a késések növekedéséhez vezet.

Az ilyen esetek kezeléséhez a webes API-nak támogatnia kell az olyan lekérdezési sztringekat, amelyek lehetővé teszik, hogy az ügyfélalkalmazás pontosítsa a kérelmeket, vagy könnyebben kezelhető, különálló blokkokra (vagy lapokra) tagolva kérje le az adatokat. Az alábbi kód az `Orders` vezérlő `GetAllOrders` metódusát mutatja be. Ez a metódus a rendelések részleteit kéri le. Ha a metódus korlátozás nélküli lenne, előfordulhat, hogy nagy mennyiségű adatot adna vissza. A `limit` és az `offset` paraméter célja, hogy az adatok mennyiségét egy kisebb részhalmazra csökkentse, ebben az esetben alapértelmezés szerint csak az első 10 rendelésre:

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

Egy ügyfélalkalmazás kiadhat olyan kérelmet, amely 30 rendelést kér le 50-es eltolástól kezdve a `https://www.adventure-works.com/api/orders?limit=30&offset=50` URI használatával.

> [!TIP]
> Ne engedélyezze az ügyfélalkalmazásoknak olyan lekérdezési sztringek megadását, amelyek 2000 karakternél hosszabb URI-t eredményeznek. Számos webes ügyfél és kiszolgáló nem tudja kezelni az ilyen hosszú URI-kat.

## <a name="maintaining-responsiveness-scalability-and-availability"></a>A válaszképesség, a skálázhatóság és a rendelkezésre állás fenntartása

Egy adott webes API-t számos ügyfélalkalmazás használhat a világ bármely részén. Fontos, hogy a webes API úgy legyen megvalósítva, hogy válaszképes legyen nagy mértékű terhelés alatt is, hogy skálázható legyen a nagymértékben ingadozó számítási feladatok kezeléséhez, és hogy garantáltan rendelkezésre álljon az üzleti szempontból kulcsfontosságú műveleteket végző ügyfelek számára. Mindezen követelmények teljesítése érdekében vegye figyelembe a következő szempontokat:

### <a name="provide-asynchronous-support-for-long-running-requests"></a>A hosszan futó kérelmek aszinkron támogatásának biztosítása

Az olyan kérelmeket, amelyek feldolgozása hosszú időbe telhet, a kérelmet küldő ügyfél blokkolása nélkül kell feldolgozni. A webes API elvégezhet néhány kezdeti ellenőrzést a kérelem érvényesítéséhez, elindíthat egy külön feladatot a munka elvégzéséhez, majd visszaadhat egy 202 (Elfogadva) HTTP-kódú válaszüzenetet. A feladat futhat aszinkron módon a webes API feldolgozófolyamatának részeként, vagy kiszervezhető egy háttérfeladatba.

A webes API-nak emellett biztosítania kell egy mechanizmust, amely visszaadja a feldolgozás eredményét az ügyfélalkalmazásnak. Ennek érdekében megadhat az ügyfélalkalmazásoknak egy lekérdezési mechanizmust, amellyel időnként lekérdezhetik, hogy a feldolgozás befejeződött-e, és lekérhetik az eredményt, vagy engedélyezhetjük, hogy a webes API egy értesítést küldjön a művelet befejezése után.

Egyszerűen megvalósíthat egy lekérdezési mechanizmust, ha a következőképpen megad egy virtuális erőforrásként szolgáló *lekérdezési* URI-t:

1. Az ügyfélalkalmazás elküldi a kezdeti kérelmet a webes API-nak.
2. A webes API tárolja a kérelemmel kapcsolatos információkat egy táblatárolóban vagy a Microsoft Azure Cache-ben tárolt táblában, és létrehoz egy egyedi kulcsot a bejegyzéshez, valószínűleg egy GUID formájában.
3. A webes API elindítja a feldolgozást egy külön feladatként. A webes API a táblában a feladat állapotát *Fut* értékre állítja.
4. A webes API visszaad egy válaszüzenetet a 202 (Elfogadva) HTTP-állapotkóddal és az üzenet törzsében a táblabejegyzés GUID-azonosítójával.
5. A feladat befejezése után a webes API tárolja az eredményeket a táblában, és a feladat állapotát *Befejezve* értékre állítja. Vegye figyelembe, hogy ha a feladat meghiúsul, a webes API a sikertelenségről is tárolhat információkat, az állapotot pedig *Sikertelen* értékre állíthatja.
6. A feladat futása közben az ügyfél továbbra is végezhet saját feldolgozást. Eközben időközönként elküldhet egy kérelmet a */polling/{guid}* URI-nek, ahol a *{guid}* az a globálisan egyedi azonosító, amelyet a webes API a 202-es válaszüzenetben visszaadott.
7. A */polling/{guid}* URI-hez tartozó webes API lekérdezi a vonatkozó feladat állapotát a táblából, és visszaad egy válaszüzenetet a 200 (OK) HTTP-állapotkóddal és az állapottal (*Fut*, *Befejezve* vagy *Sikertelen*). Ha a feladat befejeződött vagy meghiúsult, a válaszüzenet tartalmazhatja a feldolgozás eredményét vagy a sikertelenség okával kapcsolatban elérhető információkat is.

Az értesítések megvalósításának lehetőségei a következők:

- Aszinkron válaszok leküldése az ügyfélalkalmazásoknak az Azure Notification Hub használatával. További információ: [Azure Notification Hubs – felhasználók értesítése](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).
- A Comet modell használata egy állandó hálózati kapcsolat fenntartásához az ügyfél és a webes API-t üzemeltető kiszolgáló között, és a kapcsolat használata a kiszolgálóról az ügyfélhez visszaküldött üzenetek leküldéséhez. Ehhez az MSDN magazin [Egyszerű Comet-alkalmazás Microsoft .NET-keretrendszerben való létrehozását](https://msdn.microsoft.com/magazine/jj891053.aspx) ismertető cikkében talál egy példát.
- Adatok valós idejű leküldése a kiszolgálóról az ügyfél felé állandó kapcsolaton keresztül a SignalR használatával. A SignalR az ASP.NET-webalkalmazásokhoz NuGet-csomagként érhető el. További információt az [ASP.NET SignalR](https://www.asp.net/signalr) webhelyen talál.

### <a name="ensure-that-each-request-is-stateless"></a>Az egyes kérelmek állapotnélküliségének biztosítása

Minden egyes kérelmet atominak kell tekinteni. Egy ügyfélalkalmazás különböző időpontokban elküldött kérelmei között nem állhatnak fenn függőségek. Ez a módszer segíti a méretezhetőséget, és a webszolgáltatás példányai több kiszolgálóra is telepíthetők. Az ügyfélkérések bármelyik példányhoz irányíthatók, és az eredmények mindig azonosak. A megoldás hasonlóképpen javítja a rendelkezésre állást is: ha egy webkiszolgáló meghibásodik, a kérések átirányíthatók egy másik példányhoz az Azure Traffic Managerrel, amíg a kiszolgáló újraindul, és mindez semmilyen negatív hatással nincs az ügyfélalkalmazásra.

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a>Ügyfelek nyomon követése és szabályozás alkalmazása a szolgáltatásmegtagadási támadások esélyének csökkentése érdekében

Ha egy adott ügyfél nagy számú kérést küld egy adott időn belül, előfordulhat, hogy kisajátítja a szolgáltatást, és befolyásolja a további ügyfelek teljesítményét. Ennek elkerülése érdekében a webes API monitorozhatja az ügyfélalkalmazások hívásait. Ez az összes bejövő kérelem IP-címének rögzítésével vagy az összes hitelesített hozzáférés naplózásával is történhet. Ezen adatok alapján korlátozható az erőforrásokhoz való hozzáférés. Ha egy ügyfél túllép egy adott korlátozását, a webes API visszaadhat egy válaszüzenetet az 503 (A szolgáltatás nem érhető el) állapotkóddal és egy Retry-After fejléccel, amelyben megadja, hogy az ügyfél mikor küldheti el a következő kérelmet anélkül, hogy a rendszer elutasítaná. Ez a stratégia segíthet megakadályozni, hogy bizonyos ügyfelek szolgáltatásmegtagadásos (DOS) támadásokkal akadályozzák a rendszer működését.

### <a name="manage-persistent-http-connections-carefully"></a>Állandó HTTP-kapcsolatok gondos kezelése

A HTTP-protokoll támogatja az állandó HTTP-kapcsolatokat, ahol elérhetők. A HTTP 1.0 specifikáció bevezette a „Connection:Keep-Alive” fejlécet, amellyel az ügyfélalkalmazások jelezhetik a kiszolgálónak, hogy a későbbi kérelmeket ugyanazon a kapcsolaton keresztül is el tudják küldeni új kapcsolatok megnyitása helyett. A kapcsolat automatikusan lezárul, ha az ügyfél nem használja újra egy bizonyos időtartamon belül, amelyet a gazdagép határoz meg. Ez az Azure-szolgáltatások által használt HTTP 1.1 alapértelmezett viselkedése, így az üzenetekbe nem szükséges Keep-Alive fejléceket belefoglalni.

A kapcsolat nyitva tartása segíthet növelni a válaszkészséget a késés és a hálózati torlódás csökkentésével, de gyengítheti a skálázhatóságot, mivel a szükségtelen kapcsolatok felesleges nyitva tartása korlátozza a további ügyfelek csatlakozási lehetőségeit. Emellett ha az ügyfélalkalmazás mobileszközön fut, csökkentheti az eszköz akkumulátor-élettartamát is: ha az alkalmazás csak alkalmanként kérelmeket küld a kiszolgáló felé, a kapcsolat nyitva tartása gyorsabban lemerítheti az akkumulátort. Annak biztosításához, hogy a HTTP 1.1 ne tegyen állandóvá egy kapcsolatot, az ügyfél egy Connection:Close fejléc használatával felülírhatja az alapértelmezett viselkedést. Ha egy kiszolgáló rendkívül nagy mennyiségű ügyfelet kezel, akkor a válaszaiban a Connection:Close fejléc használatával hasonló módon lezárhatja a kapcsolatokat, így kiszolgáló-erőforrásokat takaríthat meg.

> [!NOTE]
> Az állandó HTTP-kapcsolatok funkciója teljes mértékben választható, és arra szolgál, hogy csökkentse a kommunikációs csatornák ismétlődő létrehozásával járó hálózati terhelést. Sem a webes API, sem az ügyfélalkalmazás nem függhet egy állandó HTTP-kapcsolat elérhetőségétől. Ne használjon állandó HTTP-kapcsolatokat a Comet stílusú értesítési rendszerek megvalósításához. Ehelyett használjon szoftvercsatornákat (vagy webes szoftvercsatornákat, ha elérhetők) a TCP-rétegben. Végül vegye figyelembe, hogy a Keep-Alive fejlécek csak korlátozottan használhatók, ha az ügyfélalkalmazás egy proxyn keresztül kommunikál a kiszolgálóval: ez esetben csak az ügyfél és a proxy kapcsolata lesz állandó.

## <a name="publishing-and-managing-a-web-api"></a>Webes API-k közzététele és kezelése

Ahhoz, hogy egy webes API elérhető legyen az ügyfélalkalmazások számára, a webes API-t telepíteni kell egy gazdakörnyezetben. Ez a környezet általában egy webkiszolgáló, bár valamilyen más típusú gazdafolyamat is lehet. A webes API-k közzétételekor vegye figyelembe a következő szempontokat:

- Minden kérelmet hitelesíteni és engedélyezni kell, és be kell tartatni a megfelelő szintű hozzáférés-vezérlést.
- A kereskedelmi webes API-k válaszidejére különböző minőségi garanciák vonatkozhatnak. Fontos, hogy a gazdakörnyezet skálázható legyen, ha a terhelés idővel nagymértékben változhat.
- Szükség lehet a kérelmek mérésére pénzügyi okokból.
- Lehetséges, hogy a webes API felé irányuló forgalmat korlátozni kell, valamint a kvótájukat felhasználó adott ügyfelekre vonatkozó szabályozást kell bevezetni.
- A szabályozási követelmények előírhatják az összes kérelem és válasz naplózását.
- A rendelkezésre állás biztosítása érdekében szükség lehet a webes API-t futtató kiszolgáló állapotának monitorozására, és adott esetben újraindítására.

Ezeket a problémákat célszerű elkülöníteni a műszaki kérdésektől a webes API-k megvalósításakor. Emiatt érdemes lehet létrehozni egy [előtérrendszert](https://en.wikipedia.org/wiki/Facade_pattern), amely külön folyamatként fut, és a webes API-hoz irányítja a kérelmeket. Az előtérrendszer felügyeleti műveleteket biztosíthat, és továbbíthatja az érvényesített kéréseket a webes API felé. Az előtérrendszer használata számos funkcionális előnnyel is jár, többek között:

- Több webes API integrációs pontjaként szolgál.
- különböző technológiák használatával átalakítja az üzeneteket és lefordítja a kommunikációs protokollokat az ügyfelek számára.
- Gyorsítótárazza a kérelmeket és válaszokat a webes API-t futtató kiszolgáló terhelésének csökkentése érdekében.

## <a name="testing-a-web-api"></a>Webes API tesztelése

A webes API-kat ugyanolyan alaposan kell tesztelni, mint minden más szoftvert. Érdemes egységteszteket működésének létrehozása.

A webes API-k jellegét elérhetővé teszi a saját további követelmények alapján, hogy megfelelően működik-e. Fordítson különös figyelmet a következőkre:

- Ellenőrizze az összes útvonalat, hogy a megfelelő műveleteket hívják-e meg. Különösen figyeljen a váratlanul visszaadott 405 (Nem engedélyezett metódus) HTTP-állapotkódra, mivel ez azt jelezheti, hogy egy útvonal és a rajta keresztül elküldhető HTTP-metódusok (GET, POST, PUT, DELETE) nem egyeznek.

    Küldjön HTTP-kérelmeket olyan útvonalakra, amelyeket nem támogatják őket, például egy POST kérelmet egy adott erőforrásra (a POST kérelmeket csak erőforrás-gyűjteményekre lehet küldeni). Ezekben az esetekben az *egyetlen* érvényes válasz a 405 (Nem engedélyezett) állapotkód lehet.

- Ellenőrizze, hogy az összes útvonal megfelelően védett-e, és megtörténnek-e a megfelelő hitelesítési és engedélyezési ellenőrzések.

  > [!NOTE]
  > Bizonyos biztonsági intézkedésekért, például a felhasználók hitelesítésért valószínűleg nem a webes API, hanem a gazdakörnyezet felel, de ettől függetlenül még szükséges a biztonsági vizsgálatok elvégzése a telepítési folyamat részeként.
  >
  >

- Tesztelje az egyes műveletek kivételkezelését, és ellenőrizze, hogy a műveletek megfelelő és jelentéssel bíró HTTP-válaszokat adnak-e vissza az ügyfélalkalmazásnak.

- Ellenőrizze, hogy a kérelem- és válaszüzenetek szabályosak-e. Ha például egy HTTP POST kérelem x-www-form-urlencoded formátumban tartalmazza egy új erőforrás adatait, győződjön meg arról, hogy a vonatkozó művelet megfelelően elemzi az adatokat, létrehozza az erőforrásokat, és visszaad egy választ, amely tartalmazza az új erőforrás részleteit, köztük a megfelelő Location fejlécet.

- Ellenőrizze az összes hivatkozást és URI-t a válaszüzenetekben. Például egy HTTP POST üzenetnek az újonnan létrehozott erőforrás URI-jét kell visszaadnia. Minden HATEOAS-hivatkozásnak érvényesnek kell lennie.

- Győződjön meg arról, hogy a minden művelet a megfelelő állapotkódokat adja vissza a különböző bemeneti kombinációkra. Példa:

  - Ha egy lekérdezés sikeres, a 200 (OK) állapotkódot kell visszaadnia.
  - Ha egy erőforrás nem található, a műveletnek a 404 (Nem található) HTTP-állapotkódot kell visszaadnia.
  - Ha az ügyfél elküld egy kérelmet, amely sikeresen töröl egy erőforrást, a 204 (Nincs tartalom) állapotkódnak kell megjelennie.
  - Az ügyfél elküld egy kérelmet, amely egy új erőforrást hoz létre, ha az állapotkód: 201 (létrehozva) kell lennie.

Figyeljen a válaszokban a váratlan állapotkódokra az 5xx tartományban. Ezekkel az üzenetekkel a gazdakiszolgáló általában azt jelzi, hogy nem tudott teljesíteni egy érvényes kérelmet.

- Tesztelje az ügyfélalkalmazás által megadható különböző fejléc-kombinációkat, és győződjön meg arról, hogy a webes API a várt információt adja vissza a válaszüzenetekben.

- Tesztelje a lekérdezési sztringekat. Ha egy műveletnek választható paraméterei is lehetnek (például tördelési kérések), tesztelje a paraméterek különböző kombinációit és sorrendjeit.

- Ellenőrizze, hogy az aszinkron műveletek sikeresen befejeződnek-e. Ha a webes API támogatja streamelést a nagyméretű bináris objektumokat (például videókat vagy hangfájlokat) visszaadó kérelmeknél, akkor ügyeljen arra, hogy az ügyfélkérelmek ne legyenek blokkolva az adatok streamelése közben. Ha a webes API ciklikus lekérdezéseket használ a hosszan futó adatmódosítási műveletekhez, ellenőrizze, hogy a műveletek menet közben megfelelően jelentik-e az állapotukat.

Emellett hozzon létre és futtasson teljesítményteszteket, amelyekkel ellenőrizheti, hogy a webes API kielégítően működik-e nagy terhelés alatt. Webes teljesítmény- és terheléses tesztelési projekteket a Visual Studio Ultimate használatával hozhat létre. További információ: [Alkalmazások teljesítményének tesztelése közzététel előtt](https://msdn.microsoft.com/library/dn250793.aspx).

## <a name="using-azure-api-management"></a>Az Azure API Management használata

Az Azure-ban érdemes az [Azure API Management](/azure/api-management//services/api-management/) szolgáltatást használnia a webes API-k közzétételéhez és felügyeletéhez. Ezzel az eszközzel létrehozhat egy olyan szolgáltatást, amely egy vagy több webes API előtér-szolgáltatásaként funkcionál. Maga a szolgáltatás egy méretezhető webszolgáltatás, amelyet az Azure felügyeleti portálján hozhat létre és konfigurálhat. A szolgáltatással a következőképpen tehet közzé és felügyelhet egy webes API-t:

1. Telepítse az API-t egy webhelyre, Azure-felhőszolgáltatásba vagy Azure-beli virtuális gépre.

2. Csatlakoztassa az API Management szolgáltatást a webes API-hoz. A felügyeleti API URL-címére küldött kérelmeket a webes API-ban lévő URI-kre képezi le a rendszer. Egy API Management szolgáltatás több webes API-hoz is irányíthatja a kérelmeket. Ezáltal több webes API-t összesíthet egyetlen felügyeleti szolgáltatásban. Hasonlóképp egy webes API-ra több API Management szolgáltatás is hivatkozhat, ha a különböző alkalmazások számára elérhető funkciókat korlátozni vagy particionálni kell.

     > [!NOTE]
     > A HTTP GET kérések válaszainak részeként létrehozott HATEOAS-hivatkozásokban található URI-knek az API Management szolgáltatás URL-címére kell hivatkozniuk, és nem a webes API-t üzemeltető webkiszolgálóra.

3. Minden webes API-hoz adja meg a közzétett HTTP-műveleteket, valamint a műveletek által bemenetként felhasználható opcionális paramétereket. Azt is beállíthatja, hogy az API Management szolgáltatás gyorsítótárazza-e a webes API-tól kapott választ, így optimalizálhatja az ugyanazon adatokra irányuló ismétlődő kérelmeket. Rögzítse az egyes műveletek által adható HTTP-válaszok részleteit. Ezek az adatok a fejlesztői dokumentációk létrehozására szolgálnak, ezért fontos hogy pontosak és teljesek legyenek.

    A műveleteket meghatározhatja manuálisan az Azure felügyeleti portáljának varázslóival, vagy importálhatja őket egy fájlból, amely WADL vagy Swagger formátumban tartalmazza a definíciókat.

4. Konfigurálja az API Management szolgáltatás és a webes API-t üzemeltető webkiszolgáló közötti kommunikáció biztonsági beállításait. Az API Management szolgáltatás jelenleg támogatja a tanúsítványokkal végzett alapszintű hitelesítést és kölcsönös hitelesítést, valamint az OAuth 2.0 felhasználói hitelesítést.

5. Hozzon létre egy terméket. A termék a közzététel egysége: azokat a webes API-kat kell hozzáadnia, amelyeket korábban csatlakoztatott a kezelési szolgáltatáshoz. A termék közzététele után a webes API-k elérhetővé válnak a fejlesztők számára.

    > [!NOTE]
    > A termék közzététele előtt megadhat felhasználói csoportokat, amelyek hozzáférhetnek a termékhez, és felhasználókat adhat a csoportokhoz. Ezzel szabályozható, hogy a webes API-t mely fejlesztők és alkalmazások használhatják. Ha a webes API-t jóvá kell hagyni, a hozzáférés előtt a fejlesztőknek kérelmet kell küldeniük a termék adminisztrátorának. A rendszergazda engedélyezheti vagy megtagadhatja a fejlesztő hozzáférését. A már engedélyezett fejlesztők is blokkolhatók, ha a körülmények megváltoznak.

6. Konfiguráljon szabályzatokat minden webes API-hoz. A szabályzatok meghatározzák például, hogy a tartományok közötti hívások engedélyezettek-e, hogy hogyan zajlik az ügyfelek hitelesítése, hogy az XML és JSON formátumok közötti átalakítás transzparens módon történik-e, hogy egy adott IP-címtartomány hívásai korlátozva legyenek-e, hogy a hívások sebessége korlátozott legyen-e, valamint a használati kvótákat. A szabályzatok alkalmazhatók globálisan az egész termékre, a termékben lévő adott webes API-ra, vagy a webes API-k egyes műveleteire.

További információ: [az API Management dokumentációja](/azure/api-management/).

> [!TIP]
> Az Azure az Azure Traffic Manager segítségével teszi lehetővé a feladatátvétel és a terheléselosztás megvalósítását, valamint egy webhely különböző földrajzi helyeken üzemeltetett példányai közötti késés csökkentését. Az Azure Traffic Manager az API Management szolgáltatással együtt használható: az API Management az Azure Traffic Manager segítségével irányítja egy webhely példányaira küldött kérelmek útválasztását. További információ: [A Traffic Manager forgalomirányítási módszerei](/azure/traffic-manager/traffic-manager-routing-methods/).
>
> Ha ebben a struktúrában egyéni DNS-neveket használ a webhelyekhez, konfigurálja mindegyik webhely megfelelő CNAME rekordját úgy, hogy az Azure Traffic Manager webhely DNS-nevére mutasson.

## <a name="supporting-client-side-developers"></a>Ügyféloldali fejlesztők támogatása

Az ügyfélalkalmazások fejlesztőinek általában információkra van szükségük a webes API elérésével kapcsolatban. Emellett dokumentációra is szükségük van a paraméterekről, adattípusokról, visszatérési típusokról és visszatérési kódokról, amelyek leírják a webszolgáltatás és az ügyfélalkalmazás között küldött különböző kérelmeket és válaszokat.

### <a name="document-the-rest-operations-for-a-web-api"></a>A webes API-k REST-műveleteinek dokumentálása

Az Azure API Management szolgáltatás tartalmaz egy fejlesztői portált, amely leírja a webes API-k által elérhetővé tett REST-műveleteket. A közzétett termékek megjelennek a portálon. A fejlesztők ezen a portálon regisztrálhatnak hozzáférésért, majd a rendszergazda jóváhagyhatja vagy visszautasítja a kérelmüket. Ha a fejlesztőnek jóváhagyták a kérelmét, kap egy előfizetési kulcsot, amellyel hitelesítheti az általa fejlesztett ügyfélalkalmazások hívásait. Ezt a kulcsot meg kell adni minden webes API-hívással, máskülönben a rendszer elutasítja a hívást.

A portál a következőket is tartalmazza:

- A termék dokumentációja, a termék által elérhetővé tett műveletek listája, a szükséges paraméterek és a különböző visszaadható válaszok. Vegye figyelembe, hogy ezek az információk a „Webes API közzététele a Microsoft Azure API Management szolgáltatással” című szakaszban található lista 3. lépésében megadott információ alapján jönnek létre.
- Kódrészletek, amelyek bemutatják a műveletek meghívását számos nyelvből, köztük a JavaScript, C#, Java, Ruby, Python, és PHP nyelvekből.
- Egy fejlesztői konzol, amely lehetővé teszi a fejlesztőknek a termékműveletek HTTP-kérelmek küldésével való tesztelését, valamint az eredmények megtekintését.
- Egy oldal, ahol a fejlesztők jelenthetik a talált hibákat és problémákat.

Az Azure felügyeleti portálja lehetőséget nyújt a fejlesztői portál testreszabására: a portál stíluselemei és elrendezése a saját vállalat márkajegyeihez igazíthatók.

### <a name="implement-a-client-sdk"></a>Ügyféloldali SDK megvalósítása

Ha olyan ügyfélalkalmazást hoz létre, amely REST-kérelmek meghívásával ér el egy webes API-t, nagy mennyiségű kódot kell írni az egyes kérelmek létrehozásához, megfelelő formázásához és a webszolgáltatást futtató kiszolgálóra való elküldéséhez, valamint a válasz elemzéséhez, a kérelem sikerességének vagy sikertelenségének megállapításához és a visszaadott adatok kinyeréséhez. Ha el kívánja szigetelni az ügyfélalkalmazást ezektől a problémáktól, megadhat egy SDK-t, amely becsomagolja a REST-felületet, és egy jobban használható metódusgyűjteménybe foglalja ezeket a részleteket. Az ügyfélalkalmazások felhasználhatják ezeket a metódusokat, amelyek transzparens módon átalakítják a hívásokat REST-kérelmekké, majd a válaszokat visszaalakítják a metódusok által visszaadott értékekké. Ez egy elterjedt megoldás, amelyet számos szolgáltatás használ, köztük az Azure SDK is.

Egy ügyféloldali SDK létrehozása jelentős vállalkozás, mivel egységes megvalósítást és alapos tesztelést igényel. Azonban a folyamat nagy része gépesíthető, és sok beszállító kínál olyan eszközöket, amelyekkel számos feladat automatizálható.

## <a name="monitoring-a-web-api"></a>Webes API monitorozása

Attól függően, hogy a webes API hogyan lett közzétéve és üzembe helyezve, monitorozhatja az API-t közvetlenül, vagy felhasználási és állapotadatokat gyűjthet az API Management szolgáltatáson áthaladó forgalom elemzésével.

### <a name="monitoring-a-web-api-directly"></a>Webes API közvetlen monitorozása

Ha a webes API megvalósításához az ASP.NET Web API-sablont (akár webes API-projektként, akár egy Azure-felhőszolgáltatáson belüli webes szerepkörként) és a Visual Studio 2013-at használta, akkor az ASP.NET Application Insights segítségével adatokat gyűjthet a webes API rendelkezésre állásáról, teljesítményéről és használatáról. Az Application Insights egy olyan csomag, amely transzparens módon nyomon követi és rögzíti a kérelmek és válaszok adatait, ha a webes API a felhőben lett üzembe helyezve. A csomag telepítése és konfigurálása után nem szükséges módosítani a webes API kódját a használatához. Ha a webes API-t egy Azure-webhelyen telepíti, a rendszer minden forgalmat megvizsgál, és a következő statisztikákat gyűjti össze:

- kiszolgáló válaszideje,
- kiszolgáló kérelmeinek száma és az egyes kérelmek részletei,
- a leglassabb kérelmek átlagos válaszidő alapján,
- a sikertelen kérelmek részletei,
- a különböző böngészők és felhasználói ügynökök által kezdeményezett munkamenetek száma,
- a leggyakrabban megtekintett lapok (ez nem elsősorban a webes API-khoz, hanem a webalkalmazásokhoz hasznos).
- a webes API-hoz hozzáférő különböző felhasználói szerepkörök.

Ezeket az adatokat valós időben követheti az Azure felügyeleti portálján. Létrehozhat webteszteket is, amelyek a webes API állapotát monitorozzák. A webtesztek rendszeres kérelmeket küldenek egy webes API megadott URI-jére, és rögzítik a válaszokat. Meghatározhatja, mi számít sikeres válasznak (például a 200-as HTTP-állapotkód), és beállíthatja, hogy ha a kérelem nem ezt a választ adja vissza, a rendszer küldjön riasztást egy rendszergazdának. Szükség esetén a rendszergazda újraindíthatja a webes API-t futtató kiszolgálót, ha meghibásodott.

További információ: [Application Insights – Az ASP.NET használatának első lépései](/azure/application-insights/app-insights-asp-net/).

### <a name="monitoring-a-web-api-through-the-api-management-service"></a>Webes API monitorozása az API Management szolgáltatáson keresztül

Ha az API Management szolgáltatással tette közzé a webes API-t, az Azure felügyeleti portál API Management oldala tartalmaz egy irányítópultot, amelyen áttekinthető a szolgáltatás általános teljesítménye. Az Elemzés oldalon részletes elemzések találhatók a termék használatáról. Ez az oldal a következő lapokat tartalmazza:

- **Használat** – Ez a lap információkat közöl az API-hívások számáról és az ezek kezeléséhez igénybe vett sávszélességről. A használati adatok szűrhetők termék, API és művelet szerint.
- **Állapot** – Ezen a lapon megtekinthetők az API-kérelmek eredményei (a visszaadott HTTP-állapotkódok), a gyorsítótárazási szabályzat hatékonysága, az API válaszideje és a szolgáltatás válaszideje. Az állapotadatok szintén szűrhetők termék, API és művelet szerint.
- **Tevékenység** – Ez a lap szöveges összegzést tartalmaz a sikeres, sikertelen és blokkolt hívásokról, az átlagos válaszidőről, valamint az egyes termékek, webes API-k és műveletek válaszidejéről. Emellett az oldal az egyes fejlesztők által indított hívások számát is felsorolja.
- **Áttekintés** – Ez a lap a teljesítményadatok összegzését tartalmazza, beleértve a legtöbb API-hívást indító fejlesztőket, és az e hívásokat fogadó termékeket, webes API-kat és műveleteket.

Ezen adatok alapján lehet megállapítani, hogy egy adott webes API vagy művelet szűk keresztmetszetet okoz-e, majd szükség esetén skálázhatja a gazdakörnyezetet további kiszolgálókkal. Emellett az is megállapítható, ha egy vagy több alkalmazás aránytalanul nagy mennyiségű erőforrást használ, majd a megfelelő szabályzatokkal kvóták állíthatók be és korlátozható a hívások gyakorisága.

> [!NOTE]
> A közzétett termékek részleteinek módosításai azonnal hatályba lépnek. Például egy webes API-hoz hozzáadhat vagy eltávolíthat egy műveletet anélkül, hogy újra közzé kellene tennie a webes API-t tartalmazó terméket.

## <a name="more-information"></a>További információ

- Az [ASP.NET – webes API OData](https://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) példákat és további információkat tartalmaz az OData webes API-k ASP.NET használatával való megvalósításáról.
- [Kötegelés támogatásának a webes API-t és a webes API Odatában](https://blogs.msdn.microsoft.com/webdev/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata/) ismerteti, hogyan lehet kötegelt műveleteket megvalósítani egy webes API OData használatával.
- [Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) blogján Jonathan Oliver idempotenciáról, és hogyan kapcsolódik az adatkezelési műveletekhez áttekintést nyújt.
- A W3C webhely [Állapotkód-definíciókat](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) ismertető lapján megtalálható a HTTP-állapotkódok teljes listája és a leírásaik.
- A [Háttérfeladatok WebJobs-feladatokkal való futtatásáról](/azure/app-service-web/web-sites-create-web-jobs/) szóló cikk útmutatást és példákat tartalmaz a háttérbeli műveletek WebJobs használatával való végrehajtásához.
- [Értesítse a felhasználókat az Azure Notification Hubs](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) mutatja be az ügyfélalkalmazások aszinkron válaszok leküldése az Azure Notification Hub használatával.
- Az [API Management](https://azure.microsoft.com/services/api-management/) leírja, hogyan lehet közzétenni egy terméket, amely felügyelt és biztonságos hozzáférést biztosít egy webes API-hoz.
- [Az Azure API Management REST API-referencia](https://msdn.microsoft.com/library/azure/dn776326.aspx) azt ismerteti, hogyan használható az API Management REST API egyéni felügyeleti alkalmazások létrehozásához.
- [A TRAFFIC Manager útválasztási módszerei](/azure/traffic-manager/traffic-manager-routing-methods/) összefoglalja, hogyan Azure Traffic Manager segítségével el a kérelmek terhelése egy webes API-t futtató webhely több példánya között.
- Az [Application Insights – Az ASP.NET első lépéseit](/azure/application-insights/app-insights-asp-net/) ismertető cikk részletes információkat nyújt az Application Insights ASP.NET webes API-projektben történő telepítéséhez és konfigurálásához.

<!-- links -->

[api-design]: ./api-design.md
