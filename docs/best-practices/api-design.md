---
title: API-tervezési segédlet
titleSuffix: Best practices for cloud applications
description: Egy jól megtervezett webes API-k létrehozása útmutatást.
author: dragon119
ms.date: 01/12/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: b15b97de2042a0e213192dd586ffdcc4c51b1f11
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897983"
---
# <a name="api-design"></a>API-tervezés

A legtöbb modern webalkalmazás API-kat tesz elérhetővé, amelyek segítségével az ügyfelek interakcióba léphetnek az alkalmazással. Egy jól megtervezett webes API-nak a következők támogatására kell törekednie:

- **Platformfüggetlenség**. Bármelyik ügyfélnek meg kell tudnia hívni az API-t – függetlenül az API belső implementálásától. Ehhez szabványos protokollokra van szükség, valamint olyan mechanizmusra, amely által az ügyfél és a webszolgáltatás meg tud egyezni a kicserélendő adatok formátumában.

- **Szolgáltatásfejlődés**. A webes API-nak képesnek kell lennie a fejlődésre és új funkciók hozzáadására – az ügyfélalkalmazásoktól függetlenül. Az API fejlődése mellett biztosítani kell a meglévő ügyfélalkalmazások módosítás nélküli működését. Minden funkció felderíthetőnek kell lennie, hogy az ügyfélalkalmazások számára teljes körűen használható.

Ez az útmutató a webes API-k tervezése során megfontolandó problémákat ismerteti.

## <a name="introduction-to-rest"></a>A REST bemutatása

Roy Fielding 2000-ben mutatta be a REST (Representational State Transfer, reprezentáción alapuló állapotátvitel) nevű, a webes szolgáltatások tervezésére szolgáló architekturális módszert. A REST egy architekturális stílus a hipermédián alapuló elosztott rendszerek készítéséhez. A REST mindennemű mögöttes protokolltól független, és nem feltétlenül kötődik a HTTP-hez. A leggyakoribb REST-alkalmazások azonban a HTTP-protokollt használják, így ez az útmutató elsősorban a HTTP REST API-k tervezésére koncentrál.

Egy elsődleges REST HTTP protokollon keresztül előnye, hogy nyílt szabványok, és nem köti az API-t megvalósítását vagy az ügyfélalkalmazások megvalósítása sem adott. Egy REST-alapú webszolgáltatás például megírható ASP.NET-ben, az ügyfélalkalmazások pedig bármilyen nyelvet vagy eszközkészletet használhatnak, amelyekkel HTTP-kérések hozhatók létre és HTTP-válaszok elemezhetők.

Az alábbiakban a HTTP-t használó RESTful API-k fő tervezési alapelvei közül ismertetünk néhányat:

- A REST API-k *erőforrások* köré vannak szervezve. Az erőforrások olyan objektumok, adatok vagy szolgáltatások, amelyek az ügyfél által elérhetők.

- Minden erőforrás rendelkezik egy *azonosítóval*. Ez az URI, amely egyedileg azonosítja az adott erőforrást. Például egy adott ügyfélrendelés URI-ja a következő lehet:

    ```HTTP
    https://adventure-works.com/orders/1
    ```

- Az ügyfelek az erőforrások *reprezentációinak* cseréje révén lépnek interakcióba a szolgáltatásokkal. Számos webes API a JSON-t használja csereformátumként. Például, ha a fent említett URI-ra egy GET-kérés érkezik, akkor a rendszer a következő válaszüzenetet adhatja vissza:

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- A REST API-k egységes felületet használnak, amely segít az ügyfél és a szolgáltatás implementálásának különválasztásában. A REST API-k HTTP épül az egységes felületet tartalmaz szabványos HTTP-műveletek használatával az erőforrásokon végezhetők műveletek. A leggyakoribb műveletek a következők: GET, POST, PUT, PATCH és DELETE.

- A REST API-k állapot nélküli kérésmodellt használnak. A HTTP-kéréseknek függetlennek kell lenniük, és bármilyen sorrendben előfordulhatnak, ezért nem valósítható meg az átmeneti állapotadatok kérések közötti megőrzése. Az információt egyedül maguk az erőforrások tárolják, és minden kérésnek atomi műveletnek kell lennie. Ez a megkötés teszi lehetővé a webes szolgáltatások kiváló méretezhetőségét, mert nincs szükség az ügyfelek és kiszolgálók közötti affinitás megőrzésére. Bármely kiszolgáló képes kezelni bármilyen ügyféltől beérkező kérést. Mindemellett más tényezők korlátozhatják a méretezhetőséget. Számos webes szolgáltatás például egy háttérbeli adattárba ír, amelyet adott esetben nehéz lehet felskálázni. (Az [adatparticionálást](./data-partitioning.md) ismertető cikk az adattárak felskálázási stratégiáit ismerteti.)

- A REST API-kat a reprezentációban szereplő hipermédia-hivatkozások vezérlik. A következő példában egy rendelés JSON-reprezentációja látható. Hivatkozásokat tartalmaz, amelyek lekérdezik vagy frissítik a rendeléshez társított ügyfelet.

    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"https://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"https://adventure-works.com/customers/3", "action":"PUT" }
        ]
    }
    ```

Leonard Richardson 2008-ban a következő [érettségi modellt](https://martinfowler.com/articles/richardsonMaturityModel.html) javasolta a webes API-khoz:

- 0. szint: Egy URI meghatározása, és minden művelet POST kéréseket az URI felé.
- 1. szint: Hozzon létre külön URI-k az egyes erőforrásokhoz.
- 2. szint: HTTP-metódusok használatával az erőforrásokon végrehajtott műveletek definiálásához.
- 3. szint: Használjon hipermédia (HATEOAS, az alábbiakban).

A 3. szint Fielding definíciója szerint igazi RESTful API-nak felel meg. A gyakorlatban számos közzétett webes API-k szint környékén 2.

## <a name="organize-the-api-around-resources"></a>Az API erőforrások köré szervezése

Összpontosítson a webes API-k által elérhetővé tett üzleti entitásokra. Például az e-kereskedelmi rendszerekben az elsődleges entitások az ügyfelek és a rendelések lehetnek. Egy rendelés megvalósítható egy HTTP POST-kérés küldésével, amely tartalmazza a rendelési adatokat. A HTTP-válasz jelzi, hogy a rendelés sikeres volt-e. Ha lehetséges, az erőforrás-URI-k alapuljanak főneveken (az erőforrás), ne pedig igéken (az erőforráson végrehajtott műveletek).

```HTTP
https://adventure-works.com/orders // Good

https://adventure-works.com/create-order // Avoid
```

Egy erőforrásnak nem szükséges egyetlen fizikai adatelemen alapulnia. Például egy rendelési erőforrás esetében előfordulhat, hogy belsőleg implementálják egy relációs adatbázis több táblájaként, de az ügyfél számára egyetlen egységként kell bemutatni. Kerülje az olyan API-k létrehozását, amelyek egyszerűen csak tükrözik az adatbázis belső szerkezetét. A REST célja, hogy modellt képezzen az entitásokról és a műveletekről, amelyeket egy alkalmazás elvégezhet az adott entitásokon. Az ügyfél előtt nem szerencsés felfedni a belső implementáció részleteit.

Az entitások gyakran gyűjteményekbe vannak csoportosítva (rendelések, ügyfelek). A gyűjtemény a gyűjtemény elemétől különálló erőforrást képez, így rendelkeznie kell saját URI-val. A következő URI jelölheti például a rendelések gyűjteményét:

```HTTP
https://adventure-works.com/orders
```

Egy HTTP GET-kérés a gyűjtemény URI-jának történő küldése lekéri a gyűjteményben szereplő elemek listáját. A gyűjtemény minden egyes eleméhez is egyedi URI tartozik. Egy, az elem URI-jának küldött HTTP GET-kérés visszaadja az elem részletes adatait.

Az URI-k elnevezésekor használjon egységes elnevezési módszert. Általában segít, ha hivatkozásgyűjtemények használatakor többes számú főneveket ad meg URI-ként. Javasolt a gyűjteményekhez és elemekhez tartozó URI-k hierarchiába rendezése. Például a `/customers` az ügyfélgyűjteményhez tartozó elérési út, a `/customers/5` pedig az 5-ös azonosítóval rendelkező ügyfélhez tartozó elérési út. Ez a megközelítés segít abban, hogy a webes API hosszabb távon is könnyen használható maradjon. Ezenkívül számos webes API-keretrendszer képes a kérések paraméteralapú URI-elérésiutak alapján történő irányítására, így meghatározhat egy útvonalat a következő elérési úthoz: `/customers/{id}`.

Vegye figyelembe a különböző típusú erőforrások közötti kapcsolatokat, valamint a társítások elérhetővé tételének módját. Például a `/customers/5/orders` az 5-ös azonosítójú ügyfél összes rendelését reprezentálhatja. De megközelítheti a kérdést az ellenkező irányból is. Ábrázolhatja a rendeléstől az ügyfélig is a társítást egy olyan URI-val, mint a következő: `/orders/99/customer`. Ha azonban túlzottan kibővíti ezt a modellt, az nehézkessé teszi az implementálását. Jobb megoldás, ha navigálható hivatkozásokkal szolgál, amelyek a HTTP-válaszüzenet törzsében lévő, társított forrásokra mutatnak. Ezt a mechanizmust a későbbiekben, [a HATEOAS-megközelítés a kapcsolódó erőforrásokhoz való navigálás engedélyezéséhez történő használatát](#using-the-hateoas-approach-to-enable-navigation-to-related-resources) ismertető szakaszban mutatjuk be részletesen.

Összetettebb rendszereken vonzónak tűnhet olyan URI-k biztosítása, amelyek lehetővé teszik az ügyfél számára a több szintnyi kapcsolatok közötti navigálást, például: `/customers/1/orders/99/products`. Azonban ezt az összetettségi szintet nehéz lehet fenntartani, illetve rugalmatlan, ha az erőforrások kapcsolatai később módosulnak. Törekedjen inkább az URI-k viszonylagos egyszerűségének megőrzésére. Ha egy alkalmazás hivatkozással rendelkezik egy erőforrásra, akkor célszerű biztosítani, hogy a hivatkozás által rá lehessen keresni az adott erőforráshoz kapcsolódó elemekre is. Az előző lekérdezés lecserélhető a következő URI-val: `/customers/1/orders`. Így a rendszer kiadja az 1-es ügyfélhez tartozó összes rendelést, majd a `/orders/99/products` segítségével a jelen rendelésben szereplő termékekre kereshet.

> [!TIP]
> Kerülje a következőnél összetettebb erőforrás-URI-k megkövetelését: *collection/item/collection*.

Egy másik tényező, hogy minden webes kérés terheli a webkiszolgálót. Minél több a kérés, annál nagyobb a terhelés. Ezért lehetőleg kerülje a „forgalmas” webes API-kat, amelyek kis méretű erőforrásokat tesznek elérhetővé nagy számban. Ilyen API esetén szükség lehet egy olyan ügyfélalkalmazásra, amely több kérést küld ki, hogy minden szükséges adatot megtalálhasson. Ehelyett inkább denormalizálja az adatokat, és ötvözze a kapcsolódó információkat nagyobb erőforrásokban, amelyek egy kéréssel lekérhetők. E megközelítés használatakor azonban fontos vigyázni arra, hogy ne olvasson be túl sok olyan adatot, amire az ügyfélnek nincs szüksége. A nagy objektumok lekérése növelheti a kérés késését, és további sávszélességgel kapcsolatos költséget okozhat. A teljesítményre vonatkozó rossz példákkal kapcsolatos további információért lásd a [forgalmas I/O](../antipatterns/chatty-io/index.md) és a [felesleges beolvasások](../antipatterns/extraneous-fetching/index.md) témakörét.

Kerülje a webes API-k és az alapul szolgáló adatforrások közötti függőségek bevezetését. Például ha az adatok egy relációs adatbázisban vannak tárolva, akkor nem szükséges a webes API-nak minden táblát erőforrások gyűjteményeként elérhetővé tennie. Valójában az egy rossz kialakítás volna. Tekintsen inkább a webes API-kra az adatbázis absztrakciójaként. Ha szükséges, vezessen be egy leképezési réteget az adatbázis és a webes API között. E módon az ügyfélalkalmazások függetlenítve lesznek az alapul szolgáló adatbázisséma módosításaitól.

Végül pedig előfordulhat az is, hogy nem lehetséges minden olyan művelet leképezése, amelyet egy webes API implementál egy meghatározott erőforráson. Az ilyen *nem erőforrás* típusú forgatókönyveket kezelheti HTTP-kéréseken keresztül, amelyek meghívnak egy függvényt, az eredményeket pedig HTTP-válaszüzenetként adják vissza. Például egy webes API, amely egyszerű számológépes műveleteket valósít meg – pl. a hozzáadást és a kivonást –, megadhat olyan URI-kat, amelyek ezeket a műveleteket pszeudo-erőforrásként teszik közzé, és lekérdezési sztringet használhat a szükséges paraméterek meghatározására. Például egy, az URI-hoz beérkező GET-kérés (*/add?operand1=99&operand2=1*) olyan válaszüzenetet adna vissza, amelynek a törzse a 100-as értéket tartalmazza. Azonban csak módjával használja az ilyen típusú URI-kat.

## <a name="define-operations-in-terms-of-http-methods"></a>Műveletek meghatározása HTTP-metódusok keretében

A HTTP-protokoll számos olyan metódust határoz meg, amely szemantikai jelentést rendel hozzá egy adott kéréshez. A RESTful webes API-k által használt gyakoribb HTTP-metódusok a következők:

- **GET**: lekéri az erőforrás reprezentációját a megadott URI-n keresztül. A válaszüzenet törzse tartalmazza a kért erőforrás részleteit.
- **POST**: egy új erőforrást hoz létre a megadott URI-n. A kérésüzenet törzse tartalmazza az új erőforrás részleteit. Vegye figyelembe, hogy a POST olyan műveletek aktiválására is használható, amelyek nem hoznak létre erőforrásokat.
- **PUT**: A megadott URI-n létrehoz egy új erőforrást, vagy cseréli a meglévőt. A kérésüzenet törzse meghatározza a létrehozni vagy frissíteni kívánt erőforrást.
- **PATCH**: egy erőforrás részleges frissítését hajtja végre. A kérés törzse megadja az erőforrásra alkalmazni kívánt módosításokat.
- **DELETE**: eltávolítja az erőforrást a megadott URI-n.

Egy adott kérés hatása függ attól, hogy az erőforrás egy gyűjtemény része vagy egy egyéni elem. A következő táblázat az e-kereskedelmi példa használatával összefoglalja a legtöbb RESTful-implementáció által elfogadott közös szabályokat. Vegye figyelembe, hogy e kérések közül nem feltétlenül implementálható mindegyik – ez az adott forgatókönyvtől is függ.

| **Erőforrás** | **POST** | **GET** | **PUT** | **DELETE** |
| --- | --- | --- | --- | --- |
| /customers |Új ügyfél létrehozása |Az összes ügyfél beolvasása |Ügyfelek tömeges frissítése |Az összes ügyfél eltávolítása |
| /customers/1 |Hiba |1-es ügyfél adatainak beolvasása |1-es ügyfél adatainak frissítése, ha létezik |1-es ügyfél eltávolítása |
| /customers/1/orders |Új rendelés létrehozása az 1-es ügyfélhez |Az 1-es ügyfél minden rendelésének beolvasása |1-es ügyfél rendeléseinek tömeges frissítése |1-es ügyfél összes rendelésének eltávolítása |

Előfordulhat, hogy a POST, PUT és PATCH közötti különbségek elsőre nem egyértelműek.

- A POST-kérések egy erőforrást hoznak létre. A kiszolgáló hozzárendel az új erőforráshoz egy URI-t, és elküldi az illető URI-t az ügyfélnek. A REST-modellben gyűjtemények esetében gyakran alkalmaznak POST-kéréseket. Az új erőforrás hozzáadódik a gyűjteményhez. A POST-kérés az adatok egy meglévő erőforráshoz való elküldésére is használható (azok feldolgozása céljából), anélkül, hogy bármilyen új erőforrást kellene létrehozni.

- A PUT-kérések létrehoznak egy erőforrást, *vagy* frissítenek egy meglévő erőforrást. Az ügyfél határozza meg az erőforrás URI-ját. A kérés törzse tartalmazza az erőforrás teljes reprezentációját. Ha ez az URI már tartalmaz egy erőforrást, azt a rendszer lecseréli. Ellenkező esetben egy új erőforrás jön létre, ha a kiszolgáló támogatja ezt. A PUT-kéréseket leggyakrabban olyan erőforrások esetében alkalmazzák, amelyek egyedi elemek – például egy meghatározott ügyfél –, nem pedig gyűjteményeknél. Előfordulhat, hogy a kiszolgáló támogatja a frissítéseket, de a PUT-kérésen keresztüli létrehozást nem. A PUT-kérésen keresztüli létrehozás támogatottsága attól függ, hogy az ügyfél közérthetően tud-e URI-t hozzárendelni egy erőforráshoz, mielőtt az létrejönne. Ha nem, használja a POST-kérést erőforrások létrehozásához, a PUT- vagy a PATCH-kérést pedig a frissítéshez.

- A PATCH-kérés *részleges frissítést* hajt végre egy meglévő erőforráson. Az ügyfél határozza meg az erőforrás URI-ját. A kérés törzse *módosítások* egy halmazát határozza meg, amelyeket az erőforráson kell alkalmazni. Ez a PUT használatánál hatékonyabb megoldás lehet, mert az ügyfél csak a módosításokat küldi el, nem pedig az erőforrás teljes reprezentációját. Tulajdonképpen a PATCH is létrehozhat egy új erőforrást („null” erőforráshoz tartozó frissítések megadásával), ha a kiszolgáló támogatja ezt.

A PUT-kéréseknek idempotensnek kell lenniük. Ha egy ügyfél egyazon PUT-kérést többször is elküld, az eredményeknek mindig meg kell egyezniük (ugyanaz az erőforrás ugyanazon értékekkel lesz módosítva). A POST- és a PATCH-kérések esetén nem garantált, hogy idempotensek lesznek.

## <a name="conform-to-http-semantics"></a>HTTP-szemantikának való megfelelés

Ez a szakasz a HTTP-specifikációknak megfelelő API-k tervezésekor jellemző szempontokat ismerteti. Nem fed le azonban minden lehetséges részletet vagy forgatókönyvet. Ha kétségei vannak, tekintse meg a HTTP-specifikációkat.

### <a name="media-types"></a>Adathordozó-típusok

Mint fentebb említettük, az ügyfelek és a kiszolgálók erőforrások reprezentációit cserélik ki egymás között. Például egy POST-kérés esetén a kérés törzse tartalmazza a létrehozandó erőforrás reprezentációját. Egy GET-kérés esetén a válasz törzse tartalmazza a lekért erőforrás reprezentációját.

A HTTP-protokoll használatával az *adathordozók típusai*, más néven a MIME-típusok használatán keresztül adja meg a rendszer a formátumokat. A nem bináris adatok esetén a legtöbb webes API támogatja a JSON (media type = application/json) és esetlegesen az XML (media type = application/xml) formátum használatát.

A kérés Content-Type fejléce megadja a reprezentáció formátumát. Az alábbiakban egy példát láthat egy POST-kérésre, amely JSON-adatokat tartalmaz:

```HTTP
POST https://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

Ha a kiszolgáló nem támogatja az adathordozó típusát, akkor a rendszer a 415-ös HTTP-állapotkódot adja vissza („Nem támogatott adathordozó-típus”).

Az ügyfél kérése tartalmazhat egy Accept fejlécet, amely tartalmazza az ügyfél által a kiszolgáló válaszüzenetében elfogadott adathordozó-típusok listáját. Példa:

```HTTP
GET https://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

Ha a kiszolgáló nem egyezik a listában szereplő adathordozó-típusok egyikével sem, a rendszer a 406-os HTTP-állapotkódot adja vissza („Nem elfogadható”).

### <a name="get-methods"></a>GET-metódusok

Egy sikeres GET-metódus általában a 200-as HTTP-állapotkódot („OK”) adja vissza. Ha az erőforrás nem található, a metódus a 404-es („Nem található”) kódot adja vissza.

### <a name="post-methods"></a>POST-metódusok

Ha egy POST-metódus új erőforrást hoz létre, akkor a rendszer a 201-es HTTP-állapotkódot („Létrehozva”) adja vissza. Az új erőforrás URI-ját a válasz Location fejléce tartalmazza. A válasz törzse tartalmazza az erőforrás reprezentációját.

Ha a metódus feldolgozást végez, de nem hoz létre új erőforrást, előfordulhat, hogy a 200-as HTTP-állapotkódot adja vissza, majd beleveszi a művelet eredményét az adott válasz törzsébe. Egyébként, ha nincs visszaadandó eredmény, a metódus a 204-es HTTP-állapotkódot („Nincs tartalom”) adja vissza, választörzs nélkül.

Ha az ügyfél érvénytelen adatokat helyez a kérésbe, a kiszolgáló a 400-as HTTP-állapotkódot („Hibás kérés”) adja vissza. A választörzs tartalmazhat további információt a hibáról vagy egy URI-hivatkozást, amely további részleteket tesz elérhetővé.

### <a name="put-methods"></a>PUT-metódusok

Ha egy PUT-metódus új erőforrást hoz létre, akkor a 201-es HTTP-állapotkódot („Létrehozva”) adja vissza, pontosan úgy, mint a POST-metódus. Ha a metódus meglévő erőforrást frissít, akkor vagy a 200-as („OK”), vagy a 204-es („Nincs tartalom”) állapotkódot adja vissza. Bizonyos esetekben nem lehetséges egy meglévő erőforrás frissítése. Ilyenkor érdemes lehet a 409-es HTTP-állapotkódot („Ütközés”) visszaadni.

Fontolja meg a tömeges HTTP PUT-műveletek implementálását, amelyek képesek egy gyűjtemény több erőforrására vonatkozó frissítések kötegelt feldolgozására. A PUT-kérésnek meg kell határoznia a gyűjtemény URI-ját, a kérés törzsének pedig a módosítandó erőforrások részleteit. Ez a megközelítés segíthet a forgalom csökkentésében és a teljesítmény javításában.

### <a name="patch-methods"></a>PATCH-metódusok

A PATCH-kérés használatával az ügyfél frissítések halmazát küldi egy meglévő erőforrásnak egy *javítási dokumentum* formájában. A kiszolgáló feldolgozza a javítási dokumentumot a frissítés végrehajtásához. A javítási dokumentum nem ismerteti a teljes erőforrást, csak az alkalmazandó módosításokat. A PATCH-metódus specifikációi ([RFC 5789](https://tools.ietf.org/html/rfc5789)) nem határozzák meg a javítási dokumentumok formátumát. A formátumot a kérésben szereplő adathordozó-típusból kell kikövetkeztetni.

A JSON valószínűleg a leggyakrabban használt adatformátum a webes API-k esetében. Két fő JSON-alapú javítási formátum létezik: a *JSON-javítás* és az *egyesített JSON-javítás*.

Az egyesített JSON-javítás rendszere némileg egyszerűbb. A javítási dokumentum szerkezete ugyanaz, mint az eredeti JSON-erőforrásé, de csak a módosítandó vagy hozzáadandó mezők alkészletét tartalmazza. Emellett egy mező törölhető, ha a javítási dokumentum egy mezőjénél a `null` értéket adják meg. (Ez azt jelenti, hogy az egyesített javítás nem megfelelő, ha az eredeti erőforrás explicit nullértékeket is tartalmazhat.)

Tegyük fel például, hogy az eredeti erőforrás a következő JSON-reprezentációval rendelkezik:

```json
{
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

Az alábbiakban egy lehetséges egyesített JSON-javítást talál ehhez az erőforráshoz:

```json
{
    "price":12,
    "color":null,
    "size":"small"
}
```

Ez jelzi, hogy a kiszolgáló frissítéséhez `price`, törlése `color`, és adja hozzá `size` &mdash; `name` és `category` nem módosulnak. Az egyesített JSON-javítás pontos részleteiért lásd az [RFC 7396](https://tools.ietf.org/html/rfc7396)-ot. Az adathordozó típusát, a JSON-javítás rendszere `application/merge-patch+json`.

Az egyesített javítás a javítási dokumentumban szereplő `null` különleges jelentése miatt nem megfelelő, ha az eredeti erőforrás tartalmazhat explicit nullértékeket. Emellett a javítási dokumentum nem adja meg a kiszolgáló által alkalmazandó frissítések sorrendjét. Az adatoktól és a tartománytól függ, hogy ez számít-e. Az [RFC 6902](https://tools.ietf.org/html/rfc6902) által definiált JSON-javítás rugalmasabb. Alkalmazandó műveletek sorozataként adja meg a módosításokat. A műveletek a következők lehetnek: hozzáadás, eltávolítás, csere, másolás és tesztelés (az értékek érvényesítéséhez). Az adathordozó típusát, a JSON-javítás `application/json-patch+json`.

Az alábbiakban a PATCH-kérés feldolgozása közben jellemzően előforduló hibafeltételeket ismertetjük, valamint a hozzájuk tartozó HTTP-állapotkódokat.

| Hibafeltétel | HTTP-állapotkód |
|-----------|------------|
| A javítás dokumentumformátuma nem támogatott. | 415 (Nem támogatott adathordozó-típus) |
| Nem megfelelően formázott javítási dokumentum. | 400 (Hibás kérés) |
| A javítási dokumentum érvényes, de az erőforrás jelenlegi állapotában nem lehet alkalmazni a módosításokat. | 409 (Ütközés)

### <a name="delete-methods"></a>DELETE-metódusok

Ha a törlési művelet sikeres, a webkiszolgálónak a 204-es HTTP-állapotkóddal kell válaszolnia, amely jelzi, hogy a rendszer már sikeresen kezelte a folyamatot, de a válasz törzse nem tartalmaz további információt. Ha az erőforrás nem létezik, a webkiszolgáló HTTP 404 (Nem található) hibát adhat vissza.

### <a name="asynchronous-operations"></a>Aszinkron műveletek

Előfordulhat, hogy egy POST-, PUT-, PATCH- vagy DELETE-művelet feldolgozásához hosszabb időre van szükség. Ha megvárja a művelet befejezését, mielőtt válaszol az ügyfélnek, az elfogadhatatlan mértékű késést okozhat. Ilyenkor fontolja meg a művelet aszinkronná tételét. Adja vissza a 202-es HTTP-állapotkódot („Elfogadva”), ha jelezni kívánja, hogy a kérés el lett fogadva feldolgozásra, de az még nem lett befejezve.

Elérhetővé kell tennie egy végpontot, amely egy aszinkron kérés állapotát adja vissza, így az ügyfél monitorozhatja az állapotot az állapotvégpont lekérdezésével. Vegye fel az állapotvégpont URI-ját a 202-es válasz Location fejlécébe. Példa:

```HTTP
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

Ha az ügyfél GET-kérést küld erre a végpontra, a válasznak tartalmaznia kell a kérés aktuális állapotát. Tartalmazhatja emellett a befejezés becsült idejét vagy egy hivatkozást, amellyel visszavonható a művelet.

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

Ha az aszinkron művelet egy új erőforrást hoz létre, az állapotvégpontnak a 303-as állapotkódot („Lásd másik helyen”) kell visszaadnia a művelet befejezése után. A 303-as válaszba vegyen bele egy Location fejlécet, amely megadja az új erőforrás URI-ját:

```HTTP
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

További információkért lásd az [aszinkron műveletek végrehajtását a REST-ben](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/).

## <a name="filter-and-paginate-data"></a>Adatok szűrése és oldalakra bontása

Az erőforrások gyűjteménye egyetlen URI-val való közzététele ahhoz vezethet, hogy az alkalmazások hatalmas mennyiségű adatokat kérnek le olyankor is, amikor az információnak csak egy részletére van szükség. Tegyük fel például, hogy az ügyfélalkalmazásnak meg kell keresnie az összes olyan rendelést, amelynek a költsége meghalad egy bizonyos értéket. Ilyenkor előfordulhat, hogy minden rendelést lekér az */orders* URI-ból, majd az ügyféloldalon szűri a találatokat. Egyértelmű, hogy ez a folyamat nem túl hatékony. A hálózati sávszélesség és a webes API-t üzemeltető kiszolgáló feldolgozási teljesítménye szempontjából nagyon pazarló megoldás.

Ehelyett az API engedélyezheti egy szűrő megadását az URI lekérdezési sztringjében. Például: */orders?minCost=n*. A webes API felelős elemzéséért és kezeléséért a `minCost` paraméter a lekérdezési karakterlánc és a kiszolgálóoldali szűrt eredmények visszaadása.

A GET-kérések gyűjtemény-erőforrások esetében nagy számú elemet is visszaadhatnak. Tervezzen olyan webes API-t, amely korlátozza az egyetlen kérés által visszaadott adatok mennyiségét. Fontolja meg olyan lekérdezési sztringek támogatását, amelyek megadják a beolvasható elemek maximális számát és egy, a gyűjteményre vonatkozó kezdőértéket (ofszetet). Példa:

```HTTP
/orders?limit=25&offset=50
```

Szintén megfontolandó egy felső határérték meghatározása a visszaadott elemek számára vonatkozóan, így megakadályozhatja a szolgáltatásmegtagadásos (DoS-) támadásokat. Segítheti az ügyfélalkalmazások működését, ha azon GET-kérések, amelyek többoldalas adatokat adnak vissza, szintén tartalmazzák a metaadatokat valamilyen formában, amelyek jelzik az adott gyűjteményben lévő elérhető erőforrások teljes számát.

Hasonló stratégiát alkalmazhat az adatok szűrésére azok lekérésekor, ha egy olyan rendezési paraméterrel szolgál, amely a mezők nevét veszi fel értékként. Például: */orders?sort=ProductID*. Ez a megközelítés azonban negatív hatással lehet a gyorsítótárazásra, mert a lekérdezési sztring paraméterei szerepelnek az erőforrás-azonosítóban, amelyet számos gyorsítótárazási implementáció kulcsként használ a gyorsítótárazott adatokhoz történő hozzáféréshez.

Bővítheti ezt a módszert úgy, hogy korlátozza az elemenként visszaadott mezők számát, ha az egyes elemek nagy mennyiségű adatot tartalmaznak. Például használhat egy olyan lekérdezésisztring-paramétert, amely vesszővel elválasztott mezőket fogad. Például: */orders?fields=ProductID,Quantity*.

A lekérdezési sztringekban minden választható paraméternek adjon közérthető alapértelmezett értékeket. Például állítsa a `limit` paramétert 10-es értékre, az `offset` paramétert pedig 0-ra, ha oldalakra bontást szeretne megvalósítani. Állítsa a rendezési paramétert az erőforrás kulcsának megfelelőre, ha szeretne rendezést megvalósítani. Végül adja meg a `fields` paramétert az erőforrás összes mezőjénél, ha támogatja a leképezéseket.

## <a name="support-partial-responses-for-large-binary-resources"></a>Részleges válaszok támogatása nagyméretű bináris erőforrásokhoz

Egy erőforrás tartalmazhat nagyméretű bináris mezőket, például fájlokat vagy képeket. A megbízhatatlan és időszakos kapcsolatok okozta problémák megoldásához és a válaszidő javításához érdemes lehet engedélyezni az ilyen erőforrások adattömbökként való lekérését. Ehhez szükséges, hogy a webes API támogassa az Accept-Ranges fejlécet a nagyméretű erőforrásokra vonatkozó GET-kérések esetében. Ez a fejléc jelzi, hogy a GET-művelet támogatja a részleges kéréseket. Az ügyfélalkalmazás küldhet olyan GET-kéréseket, amelyek egy adott erőforrás alkészletét adják vissza, és bájttartományként vannak megadva.

Emellett az ilyen erőforrások esetében fontolja meg a HTTP HEAD-kérések implementálását. A HEAD-kérések hasonlítanak a GET-kérésekre, azzal a különbséggel, hogy csak az erőforrást leíró HTTP-fejléceket adják vissza, üres üzenettörzzsel. Az ügyfélalkalmazások kiadhatnak olyan HEAD-kérést, amely megállapítja, hogy részleges GET-kérésekkel kell-e lekérni egy erőforrást. Példa:

```HTTP
HEAD https://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

Az alábbiakban egy példát láthat a válaszüzenetre:

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

A Content-Length fejléc megadja az erőforrás teljes méretét, az Accept-Ranges fejléc pedig jelzi, hogy a kapcsolódó GET-művelet támogatja-e a részleges eredményeket. Az ügyfélalkalmazás az információk használatával kisebb adattömbökben kérheti le a képet. Az első kérés az első 2500 bájtot olvassa be a Range fejléc használatával:

```HTTP
GET https://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

A válaszüzenet jelzi a részleges választ a 206-os HTTP-állapotkód visszaadásával. A Content-Length fejléc határozza meg az üzenet törzse által visszaadott bájtok tényleges számát (nem az erőforrás méretét), a Content-Range fejléc pedig azt jelzi, hogy ez az erőforrás melyik része (0–2499 a 4580 bájtból):

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

Az ügyfélalkalmazás egy későbbi kérése beolvashatja az erőforrás további részét.

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a>HATEOAS használata a kapcsolódó erőforrásokhoz való navigáció engedélyezéséhez

A REST használatának egyik elsődleges célja, hogy az URI-séma előzetes ismeretének szükségessége nélkül lehetséges legyen a navigálás az erőforrások teljes készletében. Minden HTTP GET-kérésnek vissza kell adnia a kért objektumhoz közvetlenül kapcsolódó erőforrások megkereséséhez szükséges információt a válaszban lévő hiperhivatkozások által, valamint rendelkezniük kell az egyes erőforrásokon elérhető műveleteket leíró adatokkal is. Ez az elv HATEOAS (Hypertext as the Engine of Application State, azaz az alkalmazásállapot motorját adó hiperszöveg) néven ismert. A rendszer gyakorlatilag egy véges állapotú gép, és az egyes kérésekre adott válaszok tartalmazzák az egyik állapotból a másikba való mozgáshoz szükséges adatokat. Más információra nincs szükség.

> [!NOTE]
> Jelenleg nincsenek szabványok vagy specifikációk a HATEOAS-elv modellezésére vonatkozóan. A jelen szakaszban szereplő példák egy lehetséges megoldást mutatnak be.

Például egy megrendelés és egy ügyfél közötti kapcsolat kezeléséhez a rendelés reprezentációja hivatkozásokat tartalmazhat, amelyek azonosítják a rendeléshez tartozó ügyfél számára elérhető műveleteket. Az alábbiakban látható egy lehetséges reprezentáció:

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"https://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3",
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3",
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"https://adventure-works.com/orders/3",
      "action":"DELETE",
      "types":[]
    }]
}
```

Ebben a példában a `links` tömb hivatkozások halmazát tartalmazza. Mindegyik hivatkozás egy kapcsolódó entitáson elvégzendő műveletet jelöl. Minden hivatkozás adatai tartalmazzák a kapcsolatot („customer”), az URI-t (`https://adventure-works.com/customers/3`), a HTTP-metódust és a támogatott MIME-típusokat. Ez minden információ, amelyre az ügyfélalkalmazásnak szüksége van a művelet meghívásához.

A `links` tömb a lekért erőforrásra vonatkozó, önmagára hivatkozó információt is tartalmaz. Ezek a *self*-kapcsolattal rendelkeznek.

A visszaadott hivatkozások halmaza az erőforrás állapotától függően módosulhat. Ezt értjük az alatt, hogy a hiperszöveg az „alkalmazásállapot motorja”.

## <a name="versioning-a-restful-web-api"></a>Webes RESTful API verziókezelése

Nagyon valószínűtlen, hogy egy webes API statikus marad. Az üzleti követelmények módosulásával erőforrás-gyűjtemények vehetők fel, az egyes erőforrások kapcsolatai változhatnak, valamint előfordulhat, hogy az erőforrások adatszerkezetét is módosítani kell. Bár a webes API-k az új vagy eltérő követelmények teljesítésének érdekében történő frissítése viszonylag egyszerű folyamat, meg kell fontolnia, hogy a változások miképpen fognak kihatni a webes API-t használó ügyfélalkalmazásokra. A probléma oka, hogy a fejlesztő, aki megtervezi és implementálja a webes API-t, teljes körű vezérléssel rendelkezik az API fölött, de nem bír ugyanekkora irányítással az ügyfélalkalmazások felett, amelyeket lehet, hogy egy távolról üzemelő harmadik fél hozott létre. Az elsődleges cél lehetővé tenni, hogy a meglévő ügyfélalkalmazások továbbra is változtatás nélkül működjenek, miközben új ügyfélalkalmazások is élhessenek az új funkciók és erőforrások előnyeivel.

A verziókezelés lehetővé teszi a webes API-k számára, hogy jelezzék az elérhetővé tett szolgáltatásokat és erőforrásokat, az ügyfélalkalmazások pedig küldhetnek olyan kéréseket, amelyek egy szolgáltatás vagy erőforrás meghatározott verziójára vonatkoznak. A következő szakaszokban számos különféle megközelítést ismertetünk, az előnyeikkel és a hátrányaikkal együtt.

### <a name="no-versioning"></a>Nincs verziókezelés

Ez a legegyszerűbb megközelítés, és egyes belső API-k esetében elfogadható. A nagy változások megjeleníthetők új erőforrásokként vagy hivatkozásokként. A tartalom meglévő erőforrásokhoz való hozzáadása nem biztos, hogy alapvető változást jelent, mivel az ügyfélalkalmazások, amelyek nem számítanak erre a tartalomra, egyszerűen figyelmen kívül hagyják azt.

Például a kérés URI-ra `https://adventure-works.com/customers/3` egyetlen ügyfél részleteit adja vissza `id`, `name`, és `address` ügyfélalkalmazás által várt mezők:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Az egyszerűség kedvéért a jelen szakaszban bemutatott példaválaszok nem tartalmaznak HATEOAS-hivatkozásokat.

Ha a `DateCreated` mező hozzá van adva az ügyfélerőforrás sémájához, akkor a válasz a következőképpen néz ki:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

A meglévő ügyfélalkalmazások továbbra is megfelelően működhetnek, ha figyelmen kívül tudják hagyni az ismeretlen mezőket, az új ügyfélalkalmazások pedig megtervezhetők úgy, hogy képesek legyenek kezelni az új mezőket. Azonban ha radikálisabb változások történnek az erőforrások sémájában (például mezők eltávolítása vagy átnevezése), vagy az erőforrások közötti kapcsolatok módosulnak, akkor ez akár jelentős változásokat is jelenthet, amelyek meggátolják a meglévő ügyfélalkalmazások megfelelő működését. Ilyen esetekben érdemes lehet megfontolni a következő módszerek egyikét.

### <a name="uri-versioning"></a>URI-verziókezelés

Minden alkalommal, amikor módosítja a webes API-t vagy az erőforrások sémáját, minden erőforrás esetében hozzáad egy verziószámot az URI-hoz. A korábban meglévő URI-k továbbra is úgy működnek, mint korábban, vagyis visszaadják az eredeti sémájukhoz igazodó erőforrásokat.

Az előző példában kiterjesztése, ha a `address` mezőt tartalmazó minden részét képezi a cím alárendelt mezőkbe van átstrukturálása (például `streetAddress`, `city`, `state`, és `zipCode`), az erőforrás e verziója lehet egy URI-t, amely tartalmaz egy verziószámot, mint például keresztül közzétett `https://adventure-works.com/v2/customers/3`:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Ez a verziókezelő mechanizmus nagyon egyszerű, de függ attól, hogy a kiszolgáló a megfelelő végpontra irányítja-e a kérést. Azonban nehézkessé válhat, ahogy a webes API egyre kiforrottabb lesz, és a kiszolgálónak különböző verziókat kell támogatnia egyidejűleg. Ha az egyszerűség felől közelítjük meg a kérdést, az ügyfélalkalmazások minden esetben ugyanazt az adatot (3-as ügyfél) kérdezik le, ezért az URI-nak sem kellene verziónként eltérőnek lennie. Ez a séma a HATEOAS implementálását is bonyolultabbá teszi, mivel az összes hivatkozásnak tartalmaznia kell a verziószámot a hozzájuk tartozó URI-kban.

### <a name="query-string-versioning"></a>Lekérdezésisztring-verziókezelés

Több URI megadása helyett megadhat az erőforrás verziója egy paraméterrel a lekérdezési karakterláncban, mint például a HTTP-kérelem hozzáfűzi `https://adventure-works.com/customers/3?version=2`. A verzióparamétert alapértelmezés szerint egy közérthető értékre kell állítani, például az 1 értékre, ha a régebbi ügyfélalkalmazások nem használják azt.

Ez a megközelítés azzal a szemantikai előnnyel rendelkezik, hogy ugyanazt az erőforrást a rendszer mindig ugyanabból az URI-ból kéri le, de függ attól a programkódtól, amely kezeli a lekérdezési sztring elemzésére vonatkozó kérést, és visszaküldi a megfelelő HTTP-választ. E megközelítés a HATEOAS implementálása terén ugyanazzal a hátránnyal rendelkezik, mint az URI-verziókezelési mechanizmus.

> [!NOTE]
> Egyes régebbi webböngészők és webes proxyk nem gyorsítótárazzák a válaszokat olyan kérésekre, amelyek URI-jai lekérdezési sztringet is tartalmaznak. Ez kedvezőtlen hatással lehet az olyan webes alkalmazások teljesítményére, amelyek webes API-t használnak, illetve amelyek például egy böngészőből futnak.

### <a name="header-versioning"></a>Fejléc-verziókezelés

A verziószám lekérdezésisztring-paramétereként való feltüntetése helyett használhat egy egyéni fejlécet, amely jelzi az erőforrás verzióját. E módszerhez szükség van arra, hogy az ügyfélalkalmazás a megfelelő fejlécet adja hozzá minden kéréshez, ugyanakkor az ügyfélkérést kezelő programkód használhat alapértelmezett értéket (1-es verzió), ha a verziófejléc ki van hagyva. A következő példákban nevű egyéni fejlécet *Custom-Header*. A fejléc értéke a webes API verzióját adja meg.

1-es verzió:

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

2-es verzió:

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Vegye figyelembe, hogy akárcsak az előző két megközelítésnél, a HATEOAS implementálásához fel kell tüntetni a megfelelő egyéni fejlécet minden hivatkozásban.

### <a name="media-type-versioning"></a>Adathordozótípus-verziókezelés

Amikor egy ügyfélalkalmazás HTTP GET-kérést küld egy webkiszolgálónak, akkor közölnie kell az általa kezelt tartalomformátumokat egy Accept fejléc használatával, ahogy ezt az útmutató korábbi szakaszaiban is ismertettük. Az *Accept* fejléc célja sokszor az, hogy lehetővé tegye az ügyfélalkalmazás számára annak meghatározását, hogy a választörzs XML, JSON vagy valamilyen más gyakori formátummal bír, amelynek elemzésére az ügyfél képes. Lehetséges azonban egyéni adathordozó-típusok meghatározása is. Ezek olyan információt tartalmaznak, amely alapján az ügyfélalkalmazás képes jelezni, melyik erőforrás-verzióra számít. A következő példa bemutat egy olyan kérést, amely megad egy *Accept* fejlécet a következő értékkel: *application/vnd.adventure-works.v1+json*. A *vnd.adventure-works.v1* elem jelzi a webkiszolgáló felé, hogy annak az erőforrás 1-es verzióját kell visszaadnia, míg a *json* elem azt határozza meg, hogy a választörzs formátumának JSON-nak kell lennie:

```HTTP
GET https://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

A kérést feldolgozó kód felelős az *Accept* fejléc feldolgozásáért, valamint annak figyelembevételéért (az ügyfélalkalmazás több formátumot is megadhat az *Accept* fejlécben, ebben az esetben a webkiszolgáló kiválaszthatja a legmegfelelőbbet a választörzshöz). A webkiszolgáló a Content-Type fejléc használatával megerősíti az adatformátumot a válasz törzsében:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Ha az Accept fejléc nem határoz meg egy ismert adathordozó-típust sem, a webkiszolgáló létrehozhat egy HTTP 406 („Nem elfogadható”) válaszüzenetet, vagy visszaadhat egy üzenetet az alapértelmezett adathordozó-típussal.

E megközelítést tartják a legtisztább verziókezelési mechanizmusnak, és jól használható a HATEOAS-szal is, amely tartalmazhatja a kapcsolódó adatok MIME-típusát az erőforrás-hivatkozásokban.

> [!NOTE]
> Amikor kiválaszt egy verziókezelési stratégiát, érdemes megfontolnia a teljesítményre gyakorolt hatást is, különösen a webkiszolgáló gyorsítótárazását illetően. Az URI-verziókezelés és a lekérdezésisztring-verziókezelés sémája gyorsítótárral kompatibilis, tekintve, hogy minden alkalommal ugyanaz az URI és lekérdezési sztring vonatkozik ugyanarra az adatra.
>
> A fejléc- és adathordozótípus-verziókezelési mechanizmus használatához általában szükség van további logikára, amely az egyéni fejlécben vagy az Accept fejlécben lévő értékeket vizsgálja meg. Nagyméretű környezetben számos ügyfél a webes API-k eltérő verzióját használja, ez pedig jelentős mennyiségű duplikált adatot eredményezhet a kiszolgálóoldali gyorsítótárban. A probléma akuttá válhat, ha az ügyfélalkalmazások gyorsítótárazást használó proxyn keresztül kommunikálnak a webkiszolgálókkal. A proxy csak akkor továbbítja a kérést a webkiszolgáló felé, ha a gyorsítótár jelenleg nem tartalmazza a kért adat másolatát.

## <a name="open-api-initiative"></a>Open API-kezdeményezés

Az [Open API kezdeményezést](https://www.openapis.org/) egy iparági konzorcium hozta létre annak érdekében, hogy a REST API-leírásokat az összes gyártó esetében szabványossá tegyék. A kezdeményezés részeként a Swagger 2.0-s specifikációt átnevezték OpenAPI-specifikációra (OAS), és az Open API-kezdeményezés részévé tették azt.

Érdemes lehet az OpenAPI-t használnia a webes API-jaihoz. Néhány megfontolandó szempont:

- Az OpenAPI-specifikáció véleményekkel ellátott iránymutatásokat tartalmaz a REST API-k megtervezéséről. Az együttműködési képesség terén több előnnyel rendelkezik, de nagyobb odafigyelést is igényel az API a specifikációnak megfelelő megtervezésekor.

- Az OpenAPI elsősorban az egyezményre fókuszáló megközelítést segíti elő a megvalósításra fókuszáló megoldások ellenében. Az egyezményre fókuszáló megközelítés azt jelenti, hogy először megtervezi az API-egyezményt (a felületet), és ezt követően írja meg a kódot, amely az egyezményt implementálja.

- Az olyan eszközök, mint például a Swagger, képesek ügyfélkódtárakat vagy dokumentációt létrehozni API-egyezményekből. Példákért lásd [a Swaggert használó ASP.NET webes API-k súgóoldalait](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>További információ

- [Microsoft REST API-irányelvek](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md). Részletes javaslatok a nyilvános REST API-k tervezéséhez.

- [Webes API-k ellenőrzőlistája](https://mathieu.fenniak.net/the-api-checklist/). A webes API-k tervezése és implementálása során megfontolandó szempontok hasznos listája.

- [Open API-kezdeményezés](https://www.openapis.org/). Az Open API dokumentációja és az implementálás részletei.
