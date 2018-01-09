---
title: "API-tervezési segédlet"
description: "Útmutatás hozzon létre egy jól kidolgozott webes API-t."
author: dragon119
ms.date: 01/12/2018
pnp.series.title: Best Practices
ms.openlocfilehash: f0813c18da03b9deeabbf529a560c60e8ce579d8
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/05/2018
---
# <a name="api-design"></a>API-tervezés

A legtöbb modern webalkalmazások teszi közzé az API-k, amellyel az ügyfelek számára az alkalmazással. Tetszetős webes API-k támogatása érhető el:

* **Platform függetlenség**. Bármely ügyfél kell tudni hívja az API-t, függetlenül az API hogyan a belsőleg történik. Ehhez szükséges szabványos protokollok segítségével, és amelyek az ügyfél egy olyan mechanizmus rendelkezik, és a webszolgáltatás hozzájárulhat az exchange-adatok formátumát.

* **Szolgáltatás alakulása**. A webes API-k fejlődnek, és egymástól függetlenül funkciókkal az ügyfélalkalmazásokból képesnek kell lennie. Az API-t fejlődésének meglévő ügyfélalkalmazások továbbra is módosítás nélkül működik. Minden funkciója felderíthető, kell, hogy az ügyfélalkalmazások teljesen használhatják azt.

Ez az útmutató a webes API-k tervezésekor érdemes problémákat ismerteti.

## <a name="introduction-to-rest"></a>REST bemutatása

2000 Roy Fielding egy architekturális módjáról webes szolgáltatások tervezése, javasolt a Representational State Transfer (REST). REST egy architekturális stílus hipermédia alapján elosztott rendszerek készítéséhez. REST független a mögöttes protokollt, és nem feltétlenül kötődik HTTP. Azonban a leggyakoribb REST-alkalmazása HTTP használata a protokoll, és ez az útmutató elsősorban a HTTP REST API-k tervezésekor.

Egy elsődleges REST HTTP Protokollon keresztül előnye, hogy az nyitott szabványok használ, és nem köthető az API-t vagy az ügyfélalkalmazások végrehajtásának egyetlen megvalósítása sem. Például egy REST-alapú webszolgáltatás sikerült írni az ASP.NET, és ügyfélalkalmazások használhat bármilyen nyelven vagy eszközkészlet, amely HTTP-kérelmek előállítására és elemezni a HTTP-válaszokat.

Az alábbiakban néhány fő tervezési alapelvek RESTful API-k, HTTP-n keresztül:

- REST API-k ellátására van kialakítva *erőforrások*, amelyeket bármilyen típusú objektum, adatokat vagy egy szolgáltatást, az ügyfél által elérhető. 

- Egy erőforrás rendelkezik egy *azonosítója*, ez az URI, amely egyedileg azonosítja az adott erőforrás. Például egy adott felhasználói sorrendben URI-JÁNAK lehet: 
 
    ```http
    http://adventure-works.com/orders/1
    ```
 
- Kommunikáljanak az ügyfelek egy szolgáltatás kicserélésével *felelősséget* erőforrások. Számos webes API-k használata JSON-ban az exchange-formátumban. Például egy GET kérelmet a fent felsorolt URI előfordulhat, hogy térjen vissza az adott válasz törzse:

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- REST API-k használata, ami segít a használata leválasztja az ügyfél és a szolgáltatás megvalósítások egységes felületet. A HTTP REST API-k lett felépítve, az egységes felületet tartalmaz szabványos HTTP-n keresztül műveletek erőforrások meg műveleteket elvégezni. A leggyakoribb műveletek esetében, GET, POST, PUT, javítás és törlése. 

- REST API-k állapot nélküli kérelem használ. HTTP-kérelmek függetlennek kell lenniük, és bármilyen sorrendben fordulhat elő, a kérelmek között átmeneti állapotadatokat megőrzi esetén nem valósítható meg. Az egyetlen hely adatokat tároló magukat az erőforrások, és minden kérelmet egy atomi művelet kell lennie. Ennél a határértéknél lehetővé teszi, hogy a webes szolgáltatások kell kiválóan méretezhető, mert nincs szükség semmilyen ügyfelek és kiszolgálók közötti kapcsolat megőrzéséhez. Bármely kiszolgáló kezelni tud a bármely ügyfél kérelmet. Említett, más tényezőktől korlátozhatja a méretezhetőségét. Számos webes szolgáltatás például egy háttérbeli adattára, amely bővítő nehéz lehet írni. (A cikk [Adatparticionálás](./data-partitioning.md) adattárat horizontális stratégiák ismerteti.)

- REST API-k, amelyek szerepelnek a ábrázolását hipermédia hivatkozások alakítják. Például a következő látható sorrendben JSON-ábrázolását. Beolvasni vagy frissíteni az ügyfél az ahhoz társított mutató hivatkozásokat tartalmaz. 
 
    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"PUT" } 
        ]
    } 
    ```


Leonard Richardson 2008, a következő javasolt [lejárat modell](https://martinfowler.com/articles/richardsonMaturityModel.html) Web API-kat:

- 0. szint: Egy URI megadása, és minden műveletet POST-kérésnél ezt az URI.
- 1. szint: Hozzon létre külön URI-azonosítók az egyes erőforrásokra vonatkozó.
- 2. szint: HTTP használata módszerek meghatározása az erőforrásokon végrehajtott műveletek.
- 3. szint: Hipermédia (HATEOAS, az alábbiakban) használja.

3 szintje Fielding tartozó definíciója szerint valóban RESTful API felel meg. A gyakorlatban sok a közzétett webes API-k tartoznak valahol 2. szint körül.  

## <a name="organize-the-api-around-resources"></a>Az API-t körül erőforrások rendszerezése

Összpontosítson az üzleti entitásokat, amely a webes API-k elérhetővé teszi. Például az e-kereskedelmi rendszer elsődleges entitások lehet az ügyfelek és a rendeléseket. Rendelés megvalósítható úgy, hogy a rendelés információt tartalmazó HTTP POST-kérelmet küld. A HTTP-válasz azt jelzi, hogy a megbízást sikeres volt-e. Ha lehetséges, erőforrás URI-azonosítók alapuljon főnevek (erőforrás) és a műveleteket (az erőforrás-műveletek). 

```HTTP
http://adventure-works.com/orders // Good

http://adventure-works.com/create-order // Avoid
```

Egy erőforrás nem rendelkezik egyetlen fizikai adatelemet alapján. Például egy rendelés erőforrás előfordulhat, hogy kell belső, mint egy relációs adatbázisban több táblázat implementálva, de egyetlen egységként az ügyfél számára. Ne hozzon létre olyan API-k, amelyek egyszerűen tükrözik az adatbázis belső szerkezete. REST célja a modell entitásokat és egy alkalmazás entitásainak végezhető műveletek. Egy ügyfél nem szabad felfedni a belső megvalósításához.

Entitások gyakran gyűjteményekbe vannak csoportosítva együtt (rendelések, az ügyfelek). A gyűjtemény egy külön erőforrás elemet a elem a gyűjteményben, és rendelkeznie kell a saját URI. A következő URI Azonosítót jelölheti például a rendelések gyűjteménye: 

```HTTP
http://adventure-works.com/orders
```

A gyűjtemény elemeinek listáját egy HTTP GET kérést küld a gyűjtemény URI kéri le. A gyűjtemény minden elemén is saját egyedi URI tartozik. A cikk URI HTTP GET kérelemre, hogy az elem részletes adatait adja vissza. 

Az URI-azonosítók egységes elnevezési fogad el. Általában segít, hogy a referencia-gyűjtemények használata többes számú főnevek az URI-azonosítók. Ajánlott egy URI-k rendszerezéséhez gyűjteményekhez és elemek hierarchiába. Például `/customers` elérési útját a felhasználók gyűjteményhez, és `/customers/5` elérési útját a felhasználói Azonosítóval rendelkező megegyezik az 5. Ez a megközelítés segít távol tartani nyújt a webes API-k intuitív. Is, számos webes API-keretrendszerek irányíthatja a kérelmet, meghatározhatja, hogy az elérési útvonal paraméteres URI elérési utak alapján `/customers/{id}`.

Figyelembe venni a különböző típusú erőforrások, és hogyan akkor esetleg felfedi a társításokat közötti kapcsolatokat. Például a `/customers/5/orders` előfordulhat, hogy az összes ügyfél 5 rendelések képviseli. Sikerült is keresse meg az ellenkező irányba, majd megjelenítik például a társítás megrendelés vissza az ügyfél egy URI-azonosítójú `/orders/99/customer`. Azonban túl, amennyiben ez a modell kibővítése válhat nehézkes lehet megvalósítani. Jobb megoldás, hogy hajózható, a HTTP-válasz üzenet törzsét kapcsolódó forrásokra mutató hivatkozásokat biztosít. A mechanizmus a szakaszban részletesen ismertetett [a HATEOAS megközelítéssel engedélyezése navigációs a kapcsolódó erőforrások később](#using-the-hateoas-approach-to-enable-navigation-to-related-resources).

Összetettebb rendszereken URI-azonosítók, amelyek lehetővé teszik több szintet kapcsolatokat, például a navigálni ügyfél így tempting lehet `/customers/1/orders/99/products`. Azonban ez a szint összetettségi nehéz lehet karbantartása és rugalmatlan, ha erőforrásainak kapcsolatai később módosíthatja. Ehelyett próbálnia úgy elhelyezni az URI-azonosítók viszonylag egyszerű. Az alkalmazás legalább egy erőforráshoz, amennyiben ez a hivatkozás segítségével található erőforráshoz kapcsolódó cikkek lehetővé kell tenni. Az előző lekérdezés az URI azonosító lehet cserélni `/customers/1/orders` ügyfél 1, a rendeléseket kereséséhez, majd `/orders/99/products` sorrendje a termékek kereséséhez.

> [!TIP]
> Kerülni, hogy az erőforrás URI-azonosítók eddigieknél még bonyolultabbá kelljen *gyűjtemény/elemgyűjtemény*.

Egy másik tényező, hogy minden webes kérésnek ugyanazok a terhelést a webkiszolgálón. A további kérelmeket, annál nagyobb terhelés. Ezért lehetőleg kerülje a "chatty" webes API-k kis erőforrások nagy számú visszaállítását. Ilyen az API-k szükség lehet egy ügyfélalkalmazás található összes adat, amely igényli több kérés küldése. Ehelyett érdemes denormalize az adatokat, és nagyobb erőforrásokat lehet beolvasni a kapcsolódó információk egyesíthető egy kérelemhez. Azonban kell ezt a módszert használja a terhelést növelni az, hogy az ügyfél nem szükséges adatok beolvasása elleni elosztása érdekében. Nagy objektumok beolvasása növelheti a Tiltás késése a kérelmet, és fel Önnek további sávszélességgel kapcsolatos költségek. A teljesítmény antipatterns kapcsolatos további információkért lásd: [Chatty i/o](../antipatterns/chatty-io/index.md) és [idegen beolvasása](../antipatterns/extraneous-fetching/index.md).

Ne vezet be, a webes API-t és az alapjául szolgáló adatforrásai közötti függőségek. Például ha az adatok egy relációs adatbázisban, a webes API-t nem szükséges teszi közzé minden tábla olyan erőforrások gyűjteménye, mint. Amely valójában valószínűleg rossz kialakítást. Ehelyett gondol a webes API-t, az adatbázis absztrakciós. Ha szükséges, akkor az adatbázis és a webes API-k közötti leképezést rétegben. Ily módon ügyfélalkalmazások különítve a módosításokat az alapul szolgáló adatbázis rendszerhez.

Végül hogy előfordulhat, hogy nem lehet megfeleltetni minden műveletet a webes API-k egy adott erőforrás által megvalósított. Például kezelheti *nem erőforrás-* forgatókönyvek keresztül, amely egy függvényt, és térjen vissza az eredményeket egy HTTP-válaszüzenetnek, HTTP-kérelmekre. Például egy webes API-ja, amely megvalósítja az egyszerű számológép műveletek, mint hozzáadása, és kivonás biztosítani, hogy ezek a műveletek pszeudo erőforrásként közzétételére, és használja a lekérdezési karakterláncot határozza meg a paraméter kötelező URI. Például egy GET kérelem URI */ add? operand1 99 & operand2 = = 1* alakítanák vissza válaszüzenetet 100 értéket tartalmazó szervezethez. Azonban csak ezek forma közül választhat URI-azonosítók módjával.

## <a name="define-operations-in-terms-of-http-methods"></a>Adja meg a HTTP-metódus műveletek

A HTTP protokoll számos módszer, amely a szemantikai jelentés hozzárendelése egy kérelem határozza meg. A legtöbb RESTful webes API-k által használt közös HTTP megoldások a következők:

* **ELSŐ** lekéri a megjelenítése az erőforrás a megadott URI-t. A szervezet a válaszüzenet a kért erőforrás részleteit tartalmazza.
* **POST** hoz létre egy új erőforrást a megadott URI-t. A kérelem üzenet törzsét részletesen bemutatja az új erőforrás. Vegye figyelembe, hogy POST is használható, amelyek ténylegesen ne hozzon létre erőforrások műveleteket indítsanak.
* **PUT** hoz létre, vagy a felváltja az erőforrás a megadott URI-t. A kérelem üzenet törzsét határozza meg az erőforrás létrehozása vagy frissítése.
* **JAVÍTÁS** erőforrás részleges frissítést hajt végre. A kérelem törzsében megadja a módosítások alkalmazásához az erőforráshoz.
* **Törlés** eltávolítja az erőforrás a megadott URI-t.

Egy adott kérelem hatásának függ-e az erőforrást a gyűjtemény vagy egy egyéni elemet. A következő táblázat összefoglalja az kereskedelmi példát a RESTful megvalósítások által elfogadott közös szabályok. Vegye figyelembe, hogy ezek a kérelmek nem az összes is elegendő lehet; az adott forgatókönyv függ.

| **Erőforrás** | **POST** | **GET** | **A PUT** | **TÖRLÉSE** |
| --- | --- | --- | --- | --- |
| /Customers |Hozzon létre egy új ügyfél |Összes ügyfél beolvasása |Tömeges frissítés ügyfelek |Távolítsa el az összes ügyfél számára |
| / ügyfelek/1 |Hiba |Ügyfél 1 adatainak beolvasása |Az ügyfél 1 részleteinek frissítése, ha létezik |Távolítsa el az ügyfél 1 |
| /Customers/1/orders |Hozzon létre egy új ahhoz, hogy az ügyfél 1 |Ügyfél 1 összes rendelés beolvasása |Tömeges frissítés rendelések ügyfél 1 |Távolítsa el az összes rendelés ügyfél 1 |

A POST, PUT és javítás közötti különbségek zavaró lehet.

- Egy POST kérést hoz létre egy erőforrást. A kiszolgáló rendeli hozzá az új erőforrás URI, és elküldi az ügyfélnek, hogy URI. A többi modellben, gyakran a POST kérelmek gyűjteményére vonatkoznak. Az új erőforrás van hozzá a gyűjteményhez. Egy POST kérést küldje el az adatokat a feldolgozáshoz erőforrással, bármely új erőforrás létrehozása nélkül is használható.

- Egy PUT-kérelmekben létrehoz egy erőforrás *vagy* frissíti a meglévő erőforrás. Az ügyfél határozza meg az erőforrás URI. A kérelem törzsében tartalmazza a teljes megjelenítése az erőforrás. Ha ez az URI-azonosítójú erőforrás már létezik, a rendszer lecseréli. Ellenkező esetben egy új erőforrást jön létre, ha a kiszolgáló támogatja, így. PUT kérelmek leggyakrabban elemek, például a gyűjtemények helyett egy adott ügyfélhez erőforrásokhoz is vonatkozik. A kiszolgáló támogathatja a frissítéseket, de nem keresztül PUT létrehozása. Hogy keresztül PUT létrehozását támogatja-e függ, hogy az ügyfél készítéséhez rendelhet egy URI-t egy erőforrást ahhoz, hogy létezik-e. Ha nem, használja POST erőforrások és a PUT vagy frissíteni a javítás létrehozásához.

- A PATCH kérésnek hajt végre egy *részleges frissítés* erőforrással. Az ügyfél határozza meg az erőforrás URI. A kérelem törzsében kumulatív *módosítások* alkalmazása az erőforrás. Ez attól az esettől, PUT, hatékonyabb lehet, mert az ügyfél csak akkor küldi el a módosításokat, nem az erőforrás teljes ábrázolását. Technikailag javítás is létrehozhat egy új erőforrást (megadásával "null" erőforráshoz frissítéseit), ha a kiszolgáló támogatja ezt. 

PUT kérelmek idempotent kell lennie. Ha egy ügyfél elküldte a PUT kérésben többször, az eredmények mindig meg kell egyeznie (az azonos erőforrás módosulnak ugyanazokat az értékeket). POST és a PATCH kéréseknek nem garantált, hogy az idempotent lehet.

## <a name="conform-to-http-semantics"></a>HTTP szemantikáját felel meg

Ez a szakasz ismerteti az API-k, amely megfelel-e a HTTP-specifikáció tervezéséhez jellemző szempontokat. Azt azonban nem fedi le, minden lehetséges részlet vagy a forgatókönyvet. A kétségei vannak, tekintse meg a HTTP-specifikációk.

### <a name="media-types"></a>Adathordozó-típusok

Amint azt korábban említettük, az ügyfelek és kiszolgálók erőforrások ábrázolásai exchange. Például egy POST kérelemben a kérés törzsében tartalmazza a megjelenítése az erőforrás létrehozásához. GET kérést az adott válasz törzsének tartalmazza a lehívott erőforrás ábrázolása.

A HTTP protokoll használatával formátumban vannak megadva *adathordozók típusairól*, más néven a MIME-típusok. A nem bináris adatok nagy webes API-k támogatása JSON (adathordozó-típus az application/json =) és az esetlegesen XML (adathordozó-típus = application/xml). 

A Content-Type fejléc a következő olyan kérésre vagy válaszra megadja a ábrázolását formátumát. Íme egy példa egy POST kérést JSON-adatokat tartalmazó:

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

Ha a kiszolgáló nem támogatja az adathordozó típusát, az HTTP-állapotkód 415 (nem támogatott adathordozó-típust) kell visszaadnia.

Egy ügyfél kérelmet tartalmazhat egy Accept fejlécet, amely tartalmazza az ügyfél elfogadja a kiszolgálóról a válaszüzenetben adathordozó-típusok listája. Példa:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

A kiszolgáló nem található olyan, az adathordozó típusú szerepel a listában, ha az HTTP-állapotkódot (elfogadható) 406 kell visszaadnia. 

### <a name="get-methods"></a>Módszerek beolvasása

Egy sikeres GET metódus általában a HTTP-állapotkód 200 (OK) adja vissza. Ha az erőforrás nem található, a módszer 404-es (nem található) kell visszaadnia.

### <a name="post-methods"></a>POST metódus

Ha a POST metódussal hoz létre egy új erőforrást, HTTP-állapotkód (létrehozva) 201 adja vissza. A helyet megjelölő fejlécet a válasz URI-azonosítója az új erőforrást tartalmazza. Az adott válasz törzsének tartalmazza a megjelenítése az erőforrás.

Ha a metódus egyes feldolgozási does, de nem hoz létre egy új erőforrást, a módszer térjen vissza a 200-as HTTP-állapotkód:, és a művelet eredményét szerepeljenek az adott válasz törzse. Másik lehetőségként Nincs visszaadandó eredmény, ha a metódus HTTP-állapotkód: 204 (nincs tartalom) a választörzs adhat vissza.

Ha az ügyfél érvénytelen adatokat elhelyezi a kérelmet, a kiszolgáló HTTP-állapotkód: 400 (hibás kérés) kell visszaadnia. Az adott válasz törzsének további információt a hiba vagy URI, amely további részleteket tartalmaz egy hivatkozást is tartalmaz.

### <a name="put-methods"></a>PUT metódust

Egy PUT metódust hoz létre egy új erőforrást, egyébként HTTP-állapotkód (létrehozva) 201 csakúgy, mint a POST metódussal. Ha a metódus frissíti a meglévő erőforrás, 200-as (OK) vagy a (nincs tartalom) 204 adja vissza. Bizonyos esetekben nem esetleg meglévő erőforrás frissítése. Ebben az esetben érdemes lehet HTTP-állapotkód 409 (Ütközés) ad vissza. 

Vegye figyelembe, hogy csak egy erőforráshoz egy gyűjtemény frissítések kötegelhető tömeges HTTP PUT műveletek végrehajtása. A PUT kérés URI-azonosítója a gyűjteményt kell megadnia, és a kérés törzsében adja meg a módosítani kívánt erőforrások részleteit. Ez a megközelítés segíthet chattiness csökkentéséhez és a teljesítmény javításához.

### <a name="patch-methods"></a>JAVÍTÁS módszerek

A javítás kérelemmel az ügyfél frissítéseit küld valamelyik meglévő erőforrására formájában egy *javítás dokumentum*. A kiszolgáló feldolgozza a javítás dokumentum, a frissítés végrehajtásához. A javítás dokumentum nem ismerteti a teljes erőforrás csak meghatározott módosítások alkalmazásához. A javítás metódus specification ([RFC 5789](https://tools.ietf.org/html/rfc5789)) nem adja meg a javítás dokumentumok formátumát. A formátum kell következtethető ki a kérelemben szereplő adathordozó-típus.

JSON-ja valószínűleg a leggyakrabban használt adatformátum; Ez a webes API-k. Nincsenek a két fő JSON-alapú javítás formátum, úgynevezett *JSON javítás* és *JSON egyesítési javítás*.

JSON egyesítési javítás rendszer némileg egyszerűbb. A javítás dokumentum ugyanazt a szerkezetet az eredeti JSON-erőforrásként rendelkezik, de csak a mezőket, amelyeknek meg kell új és módosított részét tartalmazza. Emellett egy mező megadásával törölhető `null` a mezőérték a javítás dokumentumban. (Ez azt jelenti, egyesítési javítás nincs megfelelő, ha az eredeti erőforrás explicit null értékeket veheti fel.)

Tegyük fel például, hogy az eredeti erőforrás van a következő JSON-Megjelenítés:

```json
{ 
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

Íme egy JSON egyesítési javítást ehhez az erőforráshoz:

```json
{ 
    "price":12,
    "color":null,
    "size":"small"
}
```

Ez alapján a kiszolgálót frissíteni a "price", "szín" törlés, és adja hozzá a "mérete". "Name" és "kategória" nem módosulnak. A pontos JSON egyesítési javításának, lásd: [RFC 7396](https://tools.ietf.org/html/rfc7396). A médiatípus JSON egyesítési javítás "application/egyesítési-javítás + json".

Egyesítési javítás nincs megfelelő, ha az eredeti erőforrást tartalmazhat explicit null értékeket, különleges szerinti miatt `null` a javítás dokumentumban. Emellett a javítás dokumentum nem ad meg, hogy a kiszolgáló vonatkozzon-e a frissítések sorrendje. Amely lehet, vagy előfordulhat, hogy nem számít, attól függően, hogy az adatok és a tartomány. JSON-javítás definiált [RFC 6902](https://tools.ietf.org/html/rfc6902), rugalmasabb. Azt adja meg a módosítások alkalmazásához műveletek sorozata. A műveletek hozzáadása, eltávolítása, cserélje le, másolja és tesztelni (érték érvényesítése). A médiatípus JSON javítás "application/json-javítás + json".

Az alábbiakban néhány tipikus hiba feltételeket, amelyek feldolgozása közben egy javítás, valamint a megfelelő HTTP-állapotkód szembesülhet.

| Hiba az állapot | HTTP-állapotkód: |
|-----------|------------|
| A javítás dokumentum formátuma nem támogatott. | 415 (nem támogatott adathordozó-típus) |
| A dokumentum nem megfelelően formázott javítás. | 400 (hibás kérés) |
| A javítás dokumentum érvényes, de az erőforrás jelenlegi állapotában nem lehet alkalmazni a módosításokat. | 409 (Ütközés)

### <a name="delete-methods"></a>Módszerek törlése

A törlési művelet végrehajtása sikeres, ha a webkiszolgálón, amely jelzi, hogy a folyamat sikeresen már kezelt, de, hogy az adott válasz törzsének tartalmazza-e további információ nem HTTP-állapotkód: 204, kell válaszolnia. Ha az erőforrás nem létezik, a webkiszolgáló (nem található) HTTP 404 adhat vissza.

### <a name="asynchronous-operations"></a>Az aszinkron műveletek

Néha egy POST, PUT, javítását vagy DELETE művelet befejezéséhez feldolgozására valamennyit lehet szükség. Várja meg a művelet befejeződése után az ügyfél választ küld, ha nem fogadható el késést okozhat. Ha igen, vegye figyelembe, így a művelet aszinkron. Térjen vissza a HTTP-állapotkód: 202 (elfogadható) jelzi a kérés elfogadva feldolgozásra, de nem végzi el. 

Olyan végponttal, amely adja vissza egy aszinkron kérés állapotát, így az ügyfél állapota figyelhető a állapot végpont lekérdezésével kell tenni. Vegye fel a helyet megjelölő fejlécet a 202 válasz az állapot-végpont URI. Példa:

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

Az ügyfél végponti GET kérést küld, ha a válasz tartalmaznia kell a kérés aktuális állapotát. Az opcionális, is lehetnek egy becsült ideje a befejezési vagy összekötőelemre kattintva szakítsa meg a műveletet. 

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345"
}
```

Ha az aszinkron művelet hoz létre egy új erőforrást, az állapot végpont kell visszatérési állapotkód 303 (lásd az egyéb) a művelet befejeződése után. A 303 válaszként helyet megjelölő fejlécet, amely az új erőforrás URI a következők:

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

További információkért lásd: [többi aszinkron műveletek](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/).

## <a name="filter-and-paginate-data"></a>Szűrés és megjelenítheti az adatokat

Egyetlen URI keresztül erőforrások gyűjteményét az ilyen alkalmazások nagy mennyiségű adat beolvasása, amikor szükség az információkat csak egy részét is járhat. Tegyük fel, hogy ügyfélalkalmazás meg kell keresnie az összes rendelés helyköltséggel keresztül megadott értékkel. Előfordulhat, hogy beolvasni a összes rendelést a */rendelések* URI és szűréssel ezeket a rendeléseket az ügyféloldalon. Egyértelműen a folyamat nem nagyon hatékony. A webes API-t futtató kiszolgálón hulladékok azt a hálózati sávszélesség és a feldolgozási kapacitást.

Ehelyett az API lehetővé tehetik a szűrőt, mint a lekérdezési karakterláncban az URI átadásakor */rendelések? minCost = n*. A webes API-k felelős majd elemzése és kezelése a `minCost` paraméter a lekérdezési karakterláncot, és a kiszolgáló oldalán a szűrt eredmények visszaadása. 

Gyűjtemény-erőforrások keresztül küldött GET-kérések esetleg adhat vissza nagyszámú elemet. Akkor tervezzen a webes API bármely egyetlen kérelem által visszaadott adatok korlátozásához. Vegye figyelembe, hogy adja meg az elemek beolvasása és a kezdő eltolása maximális számát a gyűjteményhez lekérdezési karakterláncok támogató. Példa:

```
/orders?limit=25&offset=50
```

Is érdemes lehet olyan szolgáltatásmegtagadási támadások megelőzése érdekében a visszaadott elemek számának felső korlátja. Segít az ügyfélalkalmazások, GET kérések többoldalas adatokat is tartalmaznia kell a metaadatok valamilyen visszaadó jelző érhető el a gyűjteményben lévő erőforrások teljes száma. Emellett érdemes valamilyen más intelligens lapozás stratégiák; További információkért lásd: [API tervezési megjegyzések: intelligens lapozás](http://bizcoder.com/api-design-notes-smart-paging)

Egy hasonló stratégia segítségével van lehívása, azáltal, hogy egy rendezési paraméter, amely a mezőnév értékeként, például a módon rendezheti az adatokat */rendelések? rendezési = ProductID*. Azonban ez a megközelítés is negatív hatással a gyorsítótár, mert a lekérdezési karakterlánc paramétereket kulcsként számos gyorsítótár megvalósítását által a gyorsítótárazott adatokat az erőforrás-azonosító részét képezik.

Bővítheti ezt a módszert a mezők vissza minden elemhez korlátozni, ha egyes elemek nagy mennyiségű adatot tartalmaz. Például használhatja a lekérdezési karakterlánc paraméterként, amely a mezőket, vesszővel tagolt listáját fogadja el, mint */rendelések? mezők ProductID, mennyiség =*. 

Adjon meg a lekérdezési karakterláncok a választható paramétereket jelentéssel bíró alapértelmezett értékeket. Például a `limit` 10 paraméter és a `offset` paraméter 0 értékre tördelési, ha a rendezési paraméter értéke az erőforrás kulcsa, amennyiben rendezést alkalmazza, és állítsa be a `fields` paramétert az erőforrás található összes mezőhöz Ha Ön támogatja a leképezések.

## <a name="support-partial-responses-for-large-binary-resources"></a>Részleges válasz támogatása nagy bináris erőforrások

Egy erőforrás tartalmazhatnak nagy bináris mezőkön, például a fájlok vagy képeket. Problémák megoldásához okozott megbízhatatlanná és időszakos kapcsolatokhoz és javíthatja a válaszidejét, érdemes lehet engedélyezni az ezekhez az erőforrásokhoz az adattömbök kérhető. Ehhez az szükséges, a webes API-t kell támogatja az elfogadás-tartományokat fejlécet nagy erőforrások GET kérelmek. Ezt a fejlécet, az azt jelzi, hogy a GET műveletet részleges kérelmeket támogatja. Az ügyfélalkalmazás kérelmezheti GET, amely egy adott erőforrás használata, megadott bájttartomány részét adja vissza. 

Emellett vegye fontolóra a HEAD HTTP kérések. Egy HEAD kérelem hasonlít GET kérés, azzal a különbséggel, hogy csak a HTTP-fejléceket az erőforrást, egy üres üzenettörzs leíró adja vissza. Egy ügyfélalkalmazás adhatnak ki olyan HEAD kérelem annak megállapításához, hogy egy erőforrást beolvasni a részleges GET kérelmek használatával. Példa:

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

Íme egy példa válaszüzenet: 

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

A Content-Length fejlécet ad a erőforrás teljes méretét, és az elfogadás-tartományokat fejléc azt jelzi, hogy, hogy a megfelelő GET műveletet támogatja-e a részleges eredményt. Az ügyfélalkalmazás ezen információk használatával kéri le a képet a kisebb csoportjai. Az első kérelem első 2500 bájtok beolvassa a tartomány fejléc használatával:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

A hibaválasz-üzenet jelzi, hogy a részleges válasz HTTP-állapotkód 206 vissza. A Content-Length fejlécet az üzenet törzsében (nem az erőforrás mérete) visszaadott bájtok tényleges számát adja meg, és a Content-Range fejléc azt jelzi, hogy melyik erőforrás része azt (0-2499 kívüli 4580. bájt):

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

Az ügyfélalkalmazás egy későbbi kérés kérheti le az erőforrás további része.

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a>Ahhoz, hogy a kapcsolódó erőforrások navigációs HATEOAS használata

Az elsődleges összefüggések mögött REST egyik, hogy lehetővé kell tenni az erőforrások teljes készletét megnyitása anélkül, hogy az URI-séma előzetes ismerete. Minden HTTP GET kérést kell visszaadnia az információk szükségesek ahhoz, hogy a közvetlenül a kért objektum keresztül a válaszban szereplő hivatkozások kapcsolódó erőforrás megkeresése, és azt is rendelkezniük kell az elérhető műveletek leíró adatokat minden egyes ezeket az erőforrásokat. Ez az elv HATEOAS, vagy az alkalmazás motor állapot Hypertext néven ismert. A rendszer gyakorlatilag egy véges állapotjelző gép, és az egyes irányuló kérelemre adott válasz tartalmazza a szükséges áthelyezése egy másik; adatok nincs más információ szükséges kell lennie.

> [!NOTE]
> Jelenleg nincsenek szabványok vagy meghatározhatja, hogyan kell a HATEOAS elv modell specifikációk. Az ebben a szakaszban szereplő példák bemutatják, egy lehetséges megoldás.
>
>

Például egy megrendelés és az ügyfél közötti kapcsolat kezelése, sorrendben ábrázolását sikerült mutató hivatkozásokat tartalmaznak, amelyek azonosítják az elérhető műveletek az ügyfél a rendelés. A lehetséges megjelenítése a következő: 

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"GET",
      "types":["text/xml","application/json"] 
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"DELETE",
      "types":[]
    }]
}
```

Ebben a példában a `links` tömböt tartalmaz hivatkozásokat tartalmaz. Mindegyik hivatkozás egy műveletet a kapcsolódó entitás jelöli. Minden hivatkozás szerepel a kapcsolatban ("ügyfél"), az URI (`http://adventure-works.com/customers/3`), a HTTP-metódus, és a támogatott MIME-típusok. Ez az ügyfélalkalmazás képesnek kell lennie a művelet a meghívni kívánt összes információt. 

A `links` tömböt is tartalmaz önmagára hivatkozó az erőforrással kapcsolatos információkat saját magát, amely beolvasása. Ezek a kapcsolat rendelkezik *önkiszolgáló*.

A visszaadott hivatkozások készlete változhat, attól függően, hogy az erőforrás állapotát. Ez mit jelent a "motor alkalmazásállapot." alatt hypertext az

## <a name="versioning-a-restful-web-api"></a>Versioning egy RESTful webes API

Nagyon valószínű, hogy a webes API statikus marad. Üzleti követelmények módosítása az új gyűjtemények erőforrások vehetők, erőforrásainak kapcsolatai változhat, és erőforrásokat az adatok szerkezete, előfordulhat, hogy módosítani kell. Amíg a webes API-k új vagy eltérő követelmények kezelésére frissítése egy viszonylag egyszerű folyamat, meg kell fontolnia, hogy a változások rendelkezik majd a webes API-t használó ügyfélalkalmazások által okozott hatások. A probléma oka, hogy a fejlesztő tervezésével és megvalósításával webes API-k, hogy API keresztül teljes körű vezérléssel rendelkezik, bár a fejlesztői nincs szabályozhatják, előfordulhat, hogy kialakítani, harmadik féltől származó szervezetek működő távoli ügyfél-alkalmazással azonos szinten. Az elsődleges imperatív a meglévő ügyfél alkalmazások is működjenek, miközben lehetővé teszi az új ügyfél alkalmazások új szolgáltatásait és az erőforrások kihasználását változatlan.

Versioning lehetővé teszi, hogy a webes API-k jelzi a következő szolgáltatásokat és erőforrásokat, amely azt mutatja, és egy ügyfélalkalmazás kérelmezheti irányított egy szolgáltatás vagy az erőforrás egy adott verziójához. A következő szakaszok ismertetik, amelyek mindegyikének saját előnyei és vonzatai több különböző módon.

### <a name="no-versioning"></a>Semmilyen verziószámozási
Ez a legegyszerűbb módszere az, és egyes belső API-k elfogadható. Nagy változások sikerült jeleníthető meg új erőforrások vagy mutató új hivatkozásokat.  Tartalom hozzáadása a meglévő erőforrásokkal nem találhatók használhatatlanná tévő változást, ügyfélalkalmazások, amelyek nem számított a tartalom egyszerűen figyelmen kívül hagyja azt.

Például az URI kérelem *http://adventure-works.com/customers/3* visszaadja-e részletes adatait tartalmazó egyetlen ügyfél `id`, `name`, és `address` az ügyfélalkalmazás által várt mezők :

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Az egyszerűség kedvéért az itt látható példa válaszok nem HATEOAS mutató hivatkozásokat tartalmaznak.
>
>

Ha a `DateCreated` mező hozzáadódik a séma, a felhasználói erőforrás, akkor a válasz néz ki:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Meglévő ügyfélalkalmazások továbbra is megfelelően működik, ha azok figyelmen kívül hagyása ismeretlen mezők, amíg új ügyfélalkalmazások is kell megtervezni, hogy ez az új mező kezelésére képes. Azonban a sémában az erőforrások több legradikálisabb változások történnek (például eltávolítása, vagy mezők átnevezése) vagy a erőforrásainak kapcsolatai módosítani majd ezek jelenthet akár jelentős változásokat, amelyek meggátolják a meglévő ügyfél alkalmazások megfelelően működik-e . Ezekben az esetekben érdemes lehet a következő módszerek egyikét.

### <a name="uri-versioning"></a>URI versioning
Minden alkalommal, amikor módosítja a webes API-t vagy módosítja a sémát, az erőforrások hozzáadása egy verziószámot minden erőforrás URI. A korábban meglévő URI-k továbbra is működnek, mint korábban tartó erőforrások visszatér az eredeti séma.

Az előző példában kiterjesztése, ha a `address` mező új alárendelt mezőkbe tartalmazó minden részét képezi a cím szerkezete (például `streetAddress`, `city`, `state`, és `zipCode`), az erőforrás ezen verziója lehet egy verziószámot, például a http://adventure-works.com/v2/customers/3 tartalmazó URI keresztül közzétett:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

A verziókezelő mechanizmus nagyon egyszerű, de a kiszolgáló a kérés útválasztási a megfelelő végpontnak függ. Azonban, a webes API Miután kiforrottá válik, több lépés keresztül kezelése nehézkessé válhat, és a kiszolgáló rendelkezik egy különböző verziói számának támogatásához. Is, egy purist szempontjából, minden esetben az ügyfélalkalmazások számára vannak a azonos adatok beolvasása (ügyfél 3), így az URI nem valóban szabad verziójától függően eltérnek. Ez a séma HATEOAS végrehajtását is növeli az összes hivatkozás kell ahhoz, hogy a verziószám szerepeljen az URI-azonosítók szerint.

### <a name="query-string-versioning"></a>Lekérdezési karakterlánc versioning
Ahelyett, hogy így több URI-k, megadhatja a verziót az erőforrás egy paraméterrel, a lekérdezési karakterláncot, mint a HTTP-kérelem fűzött belül *http://adventure-works.com/customers/3?version=2*. A version paramétert kell alapértelmezés szerint egy jelentéssel bíró értékre, például 1, ha ki van hagyva régebbi ügyfélalkalmazások.

Ez a megközelítés azzal az szemantikai előnye, hogy ugyanaz az erőforrás mindig lekért ugyanilyen URI, de a kódot, amely kezeli a kérelmet a lekérdezési karakterlánc elemzése és a megfelelő HTTP-választ küldött függ. Ez a megközelítés is szenved az azonos komplikációk URI versioning mechanizmusként HATEOAS megvalósításához.

> [!NOTE]
> Egyes régebbi webböngészők és a webes proxykat rendszer gyorsítótárazza a választ az URI lekérdezési karakterláncot tartalmazó kérelmeket. Ez kedvezőtlen hatással lehet webes alkalmazásokhoz, amelyek a webes API-k használják, és futtatni egy webes böngésző belül, amely teljesítmény.
>
>

### <a name="header-versioning"></a>Fejléc versioning
Ahelyett, hogy hozzáfűzi a lekérdezési karakterlánc paraméterként a verziószámot, egy egyéni fejlécet, amely jelzi a verziót az erőforrás sikertelen végrehajtása. Ez a módszer igényli, hogy, hogy az ügyfélalkalmazás a megfelelő fejlécet ad a kéréseit, bár a kódot az ügyfél kérelmet kezelő használhatja az alapértelmezett értéket (verzió: 1), ha meg van adva a verziófejléc. Az alábbi példák használatára egy elnevezésű egyéni fejlécet *egyéni-fejléc*. A fejléc értékének webes API verzióját adja meg.

1. verzió:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

2. verzió:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Vegye figyelembe, hogy az előző két megközelítés a HATEOAS végrehajtási igényel a együtt megfelelő egyéni szereplő esetleges hivatkozások.

### <a name="media-type-versioning"></a>Az adathordozó típusa versioning
Amikor egy ügyfél-alkalmazás egy HTTP GET kérést küld egy webkiszolgálóhoz történő azt kell kötni, is képes kezelni az Accept fejlécet, használatával, ez az útmutató korábbi szakaszaiban ismertetett tartalom formátuma. Gyakran a célja a *elfogadás* fejléc, hogy lehetővé tegyék az ügyfélalkalmazás adhatja meg, hogy a választörzs XML, JSON vagy valamilyen más általános formátumú, mely az ügyfél képes legyen-e. Azonban úgy is, amely tartalmazza az információt, amely alapján az ügyfélalkalmazás annak jelzésére, hogy melyik erőforrás verzióját is megfelelő egyéni adathordozó-típust határoznak meg. A következő példa bemutatja egy olyan kérelmet, amely meghatározza egy *elfogadás* értékű fejléc *application/vnd.adventure-works.v1+json*. A *vnd.adventure-works.v1* elem határozza meg, hogy a webkiszolgáló, hogy az visszaküldje-e az 1 verziót az erőforrás, amíg a *json* elem megadhatja, hogy az adott válasz törzsének formátuma JSON:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

A kódot, a kérelem feldolgozása felelős a *elfogadás* fejléc és a lehető érvényesítenie azt (az ügyfélalkalmazás adhatnak meg több formátumok a *elfogadás* fejléc, ebben az esetben a webalkalmazás-kiszolgáló is válassza ki a legmegfelelőbb az adott válasz törzsének formátumát). A webkiszolgáló megerősíti, hogy a válasz törzsében szereplő adatok formátumának a Content-Type fejléc használatával:

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Az Accept fejlécet bármely ismert adathordozó típusa nem határoz meg, ha a webkiszolgáló nem sikerült létrehozni egy HTTP 406 (elfogadható) válaszüzenetet vagy egy üzenetet, amelyben egy alapértelmezett az adathordozó típusát adja vissza.

Ez a megközelítés késései referenciavegyületeknek a lehető legtisztább versioning mechanizmusok és adatmodelljeinek természetes HATEOAS, ami magában foglalhatja a kapcsolódó adatokra vonatkozó MIME-típusát a források hivatkozásait.

> [!NOTE]
> Amikor kiválaszt egy versioning stratégia, is érdemes a gyakorolt hatása a teljesítményre, különösen gyorsítótárazása a webkiszolgálón. Az URI versioning és a lekérdezési karakterlánc versioning sémák áll a gyorsítótár-barát amennyiben ugyanazokhoz az adatokhoz minden alkalommal, amikor a azonos URI-lekérdezés-karakterlánc kombinációja hivatkozik.
>
> A fejléc versioning és médiatípus versioning mechanizmusok használatához általában szükség van további logikát vizsgálja meg az értékeket az egyéni fejlécet vagy az Accept fejlécet. A nagyméretű környezetben sok ügyfél különböző verzióit egy webes API-t használ a kiszolgálóoldali gyorsítótár duplikált adatok jelentős mennyiségű eredményezhet. Lehet, hogy a probléma akut, ha egy ügyfél-alkalmazás egy webkiszolgálón, amely megvalósítja a gyorsítótárazás, és, hogy csak továbbítja a kérést a webkiszolgáló Ha azt jelenleg nem vonatkozik a kért adatok másolatát gyorsítótárában proxyn keresztül kommunikál.
>
>

## <a name="open-api-initiative"></a>Nyissa meg az API kezdeményezés
A [nyitott API kezdeményezésére](https://www.openapis.org/) egy iparági konzorcium normalizálnia REST API leírása több szállító hozta létre. A kezdeményezés részeként a Swagger 2.0-s specifikációnak volt a OpenAPI Specification (OAS) neve, és nyissa meg az API kezdeményezésére alapján.

Érdemes lehet OpenAPI elfogadják a Web API-k. Néhány megfontolandó szempontok:

- Az OpenAPI specifikációt tartalmaz egy opinionated iránymutatásokat meg, hogyan kell megtervezni a REST API-t. Amely a való együttműködés előnye is van, de további figyelmet igényel, az API-t a specifikációnak megfelelő tervezésekor.
- OpenAPI elősegíti a szerződés-első megközelítés ahelyett, hogy egy megvalósítási-first-módszert is. Adategyezmény-első azt jelenti, hogy alakítson ki az API-t először szerződést (a felület), és jegyezze meg a szerződés megvalósító kódot. 
- Eszközök, például a Swagger API-szerződések hozhat létre ügyfélkódtárai vagy dokumentációját. Lásd például: [ASP.NET API segítségével weblapok a Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>További információ
* [Microsoft REST API-irányelvek](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md). Részletes információkat találhat a nyilvános REST API-k tervezésekor.
* [A többi Cookbook](http://restcookbook.com/). Bevezetés a RESTful API-k létrehozása.
* [Webes API ellenőrzőlista](https://mathieu.fenniak.net/the-api-checklist/). Megfontolandó tervezésével és megvalósításával webes API-k hasznos listáját.
* [Nyissa meg az API kezdeményezésére](https://www.openapis.org/). Dokumentáció és megvalósítási részleteket nyílt API-t.
