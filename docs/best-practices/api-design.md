---
title: "API-tervezési segédlet"
description: "Útmutatás jól létrehozása után API tervezték."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a>API-Tervező
[!INCLUDE [header](../_includes/header.md)]

Számos modern webes megoldás ellenőrizze webszolgáltatások webkiszolgálókon, funkcionalitással távoli ügyfél alkalmazások használatát. A művelet, amely egy webszolgáltatás-bővítmény elérhetővé teszi a webes API jelent. Tetszetős webes API-k támogatása érhető el:

* **Platform függetlenség**. Kell, hogy az API-t, hogy a webszolgáltatás biztosít anélkül, hogy az adatok vagy a műveletek API elérhetővé tévő fizikailag valósíthatók meg használatára képesek az ügyfélalkalmazások számára. Ehhez szükséges, hogy az API-t, amelyek lehetővé teszik egy ügyfél alkalmazás és szolgáltatás mely adatok formátumok használatát, és az ügyfél és a webszolgáltatás keresztül megosztott adatokat szerkezete megállapodni közös szabványokkal megfelel-e.
* **Szolgáltatás alakulása**. A webszolgáltatás fejlődnek és hozzáadása (vagy eltávolítása) funkció ügyfélalkalmazásokat től függetlenül képesnek kell lennie. Meglévő ügyfélalkalmazások továbbra is működik, a webes szolgáltatás változás által nyújtott szolgáltatásokat változatlan kell lennie. Összes funkciót is kell felderíthető, úgy, hogy az ügyfélalkalmazások teljesen használhatják azt.

Ez az útmutató célja a webes API-k tervezésekor érdemes problémákat ismertetik.

## <a name="introduction-to-representational-state-transfer-rest"></a>Bevezetés a Representational State Transfer (REST)
A saját disszertáció zárja 2000 Roy Fielding javasolt egy másik architekturális módjáról a webszolgáltatások; által elérhetővé tett műveletek szerkezetének kialakítása REST. REST egy architekturális stílus hipermédia alapján elosztott rendszerek készítéséhez. Egy elsődleges a többi modell előnye, hogy nyitva szabványok alapul, és nem köthető. a modell vagy az ügyfélalkalmazások számára bármely adott megvalósításához elérő végrehajtása. Például egy REST webszolgáltatás-bővítmény a Microsoft ASP.NET Web API használatával találhatja, és az ügyfélalkalmazások bármely nyelvet és a HTTP-kérelmek előállítására és elemezni a HTTP-válaszok eszközkészlet használatával lehet kialakítani.

> [!NOTE]
> REST ténylegesen független a mögöttes protokollt, és nem feltétlenül kötődik HTTP. Azonban a többi alapuló rendszerek leggyakoribb alkalmazása HTTP küldését és fogadását kérelmek alkalmazás protokollt használja. Ez a dokumentum elsősorban REST alapelvek leképezése rendszert úgy tervezték, hogy működik a HTTP-n keresztül.
>
>

A többi modellje egy navigációs séma képviselő objektum és a szolgáltatások a hálózaton keresztül (néven *erőforrások*). Sok rendszerek REST általában a HTTP protokoll használatával továbbítja ezeket az erőforrásokat hozzáférés kérése. Az ezekben a rendszerekben ügyfélalkalmazás kérést formája, amely azonosítja az erőforrás URI és egy (a leggyakrabban használt alatt GET, POST, PUT vagy DELETE) HTTP-metódus, amely jelzi az adott erőforrás kell elvégezni a műveletet.  A HTTP-kérelem törzsében a művelet végrehajtásához szükséges adatokat. Megértéséhez lényege, hogy a többi állapot nélküli kérelem modell határozza meg. HTTP-kérelmek függetlennek kell lenniük, és bármilyen sorrendben fordulhat elő, tartsa meg az átmeneti állapotadatokat kérelmek között kísérlet esetén nem valósítható meg.  Az egyetlen hely adatokat tároló magukat az erőforrások, és minden kérelmet egy atomi művelet kell lennie. A többi modell hatékonyan, egy adott kérelem átkerül egy erőforrást egy jól meghatározott nem átmeneti állapotból, egy másik véges állapotjelző gép valósítja meg.

> [!NOTE]
> Egyes kérelmeket a többi modell állapotmentes jellege lehetővé teszi, hogy a rendszer következő alapelvek magas szinten méretezhető kell kialakítani. Nincs szükség semmilyen egy ügyfélalkalmazást, így azokat a kérelmeket és ezeket a kérelmeket kezelő adott webkiszolgálók közötti kapcsolat megőrzéséhez.
>
>

Egy hatékony REST-modell végrehajtása során egy másik kritikus fontosságú pont, amelyre a modell hozzáférést biztosít a különböző erőforrásainak kapcsolatai megértése. Ezeket az erőforrásokat általában vannak sorolva gyűjtemények és a kapcsolatokat. Tegyük fel, hogy egy kereskedelmi rendszer gyors elemzését jeleníti meg, hogy vannak-e két gyűjtemények belül alkalmazások valószínűleg érdekelt: rendeléseket és ügyfelek. Minden sorrendje, az ügyfél azonosítására szolgál a saját egyedi kulcsot kell rendelkeznie. Lehet, hogy valami más dolga, mint a gyűjtemény rendelések eléréséhez URI */rendelések*, és ugyanígy URI-JÁNAK beolvasása az összes ügyfél lehet */customers*. Kiállító egy HTTP GET kérést a */rendelések* URI visszaadja-e egy HTTP-válasz kódolása a gyűjteményben lévő összes rendeléseket reprezentáló listáját:

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

A lent látható módon válasz egy JSON-lista szerkezetének kódolja a rendeléseket:

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
Egyes sorrendhez fetch előírja, hogy a rendelés azonosító megadása a *rendelések* erőforrás, például a */kérelmek/2*:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> Az egyszerűség kedvéért ezekben a példákban az információk megjelenítése válaszok JSON-szöveg adatokat ad vissza. Azonban nincs ok erőforrások miért nem tartalmazhat más típusú adatok támogatott HTTP-alapú, például bináris vagy titkosított információk; a HTTP-válasz a tartalomtípus kell adja meg a típusát. Emellett egy REST-modell tud visszatérni ugyanazokat az adatokat a különböző formátumokban, például az XML- vagy JSON lehet. Ebben az esetben a webszolgáltatás kell tudni tartalom végrehajtani a kérést a ügyféllel. A kérelem tartalmazhat egy *elfogadás* fejlécet, amely megadja az előnyben részesített formátumban, amely az ügyfél szeretne kapni, és a webszolgáltatás megkíséreljék-e ezt a formátumot tiszteletben Ha lehetséges.
>
>

Figyelje meg, hogy a többi kérést kapott válasz alkalmazza, a szabványos HTTP állapotkód esetében. Például a kérelmeket, amelyek érvényes adatokat ad vissza tartalmaznia kell a HTTP-válaszkód a 200-as (OK), miközben a kérelmeket, amelyek nem található vagy nem megadott erőforrás törlése kell visszaadnia egy választ, amely tartalmazza a HTTP-állapotkód: 404-es (nem található).

## <a name="design-and-structure-of-a-restful-web-api"></a>Tervezés és RESTful webes API-k szerkezete
Az egy sikeres webes API kulcsai egyszerűség és konzisztenciáját. Egy webes API, ez a két tényező mutat megkönnyíti az ügyfél-alkalmazások, amelyeket az API-t használ.

A RESTful webes API kitettségének kapcsolódó erőforrások olyan készletét, és kezeléséről a alapvető műveleteket, amelyek lehetővé teszik az alkalmazások kezelheti ezeket az erőforrásokat, és könnyen kikereshetik közöttük összpontosít. Emiatt a tipikus RESTful webes API-k alkotó URI-k legyen objektumorientált felé érheti el az adatokat, és a HTTP által biztosított eszközökkel használja ezeket az adatokat az való működésre. Ez a módszer egy másik alaposan, hogy általában egy objektumorientált API, amely azt kell több számos alkalommal osztályok és objektumok viselkedését osztályok készlete tervezésekor a igényli. Emellett a RESTful webes API állapotmentes legyen, és adott sorrendben meghívott műveletek nem függ. A következő szakaszok összegzik a RESTful webes API-k tervezésekor érdemes pontokat.

### <a name="organizing-the-web-api-around-resources"></a>A webes API-k körül erőforrások rendszerezése
> [!TIP]
> Az URI-azonosítók egy REST-alapú webszolgáltatás által elérhetővé tett főnevek alapján (az adatokat, amelyhez hozzáférést biztosít a webes API-k) és a nem a műveleteket (az alkalmazás teendők adatokkal).
>
>

Összpontosítson az üzleti entitásokat, amely a webes API-k elérhetővé teszi. Például a webes API-k arra tervezték, hogy a korábban leírt kereskedelmi rendszer, az elsődleges entitások legyenek az ügyfelek és a rendeléseket. Folyamat, például az a folyamat egy rendelés egy HTTP POST művelet veszi a megrendelés információit, és hozzáadja azt az ügyfél rendelések listájának megadásával érheti el. A POST művelet belső, például a készlet szintjeinek ellenőrzése és az ügyfél számlázási műveleteket hajthatnak végre. A HTTP-válasz adhatja meg, hogy a megbízást sikeres volt-e. Ne feledje, hogy egy erőforrás nincs egyetlen fizikai adatelemet alapján. Például egy rendelés erőforrást is elegendő lehet belső elosztva több táblázatot egy relációs adatbázisban, de az ügyfélnek egyetlen egységként jelenik meg a sorok összesítik információk segítségével.

> [!TIP]
> Ne tervezése egy REST-felület, amely tükrözi, vagy a belső struktúra, amely azt mutatja meg az adatok függ. REST tárgya több megvalósítása egyszerű (létrehozása, beolvasása, Update, Delete) CRUD műveletek keresztül külön táblázatban egy relációs adatbázisban. REST célja üzleti entitásokat, és ezeket az entitásokat ezeket az entitásokat fizikai megvalósításához is képesek elvégezni egy alkalmazás, de nem szabad felfedni ügyfél műveletek leképezése a fizikai részleteit.
>
>

Az egyes üzleti ritkán entitások elkülönítési (bár előfordulhat, hogy létezik néhány egypéldányú objektum), de ehelyett az egyes csoportosítja a gyűjteményekbe. A többi feltételek, minden entitáshoz és minden gyűjteményben olyan erőforrások. A RESTful webes API-k egyes gyűjtemények belül a webszolgáltatás a saját URI tartozik, és egy gyűjtemény URI keresztül hajtja végre a HTTP GET kérelemre lekér egy listát azokról a gyűjtemény elemeinek. Az egyes elemekre is a saját URI-azonosító tartozik, és egy alkalmazás elküldheti az adott URI-Beállításokkal történő részleteinek lehívására szolgáló, hogy az elem egy másik HTTP GET kérést. Az URI-azonosítók gyűjteményekhez és elemeket kell szervezni hierarchikus elrendezését. A kereskedelmi rendszerben, az URI */customers* azt jelzi, a felhasználói gyűjteménybe, és */ügyfelek/5* részleteit lekéri az egyetlen ügyfél az azonosító 5 ebből a gyűjteményből. Ez a megközelítés segít távol tartani nyújt a webes API-k intuitív.

> [!TIP]
> Az URI-azonosítók; egységes elnevezési elfogadása általában segít, hogy a referencia-gyűjtemények használata többes számú főnevek az URI-azonosítók.
>
>

Is kell figyelembe venni a különböző típusú erőforrások, és hogyan akkor esetleg felfedi a társításokat közötti kapcsolatokat. Például az ügyfelek helyezhet nulla vagy több rendelés. Ezt a kapcsolatot képviselő természetes módon lenne egy URI segítségével például */customers/5/orders* ügyfél 5 a rendeléseket kereséséhez. Érdemes megfontolnia a társítás megrendelés vissza az adott ügyfélhez egy URI segítségével például képviselő */orders/99/customer* az ügyfél található rendelés 99, de ez a modell túlságosan válhat nehézkes lehet kiterjesztése valósítja meg. Jobb megoldás az, hogy hajózható kapcsolatot kapcsolódó erőforrások, például az ügyfél a HTTP-válaszüzenetnek értéket adott vissza a sorrendet le kell kérdezni törzsében. A mechanizmus navigációs engedélyezése a kapcsolódó erőforrások HATEOAS módszert használva ez az útmutató későbbi szakaszában a szakaszban részletesen ismertetett.

Összetettebb rendszereken előfordulhat többféle típusú entitás, és azok tempting URI-azonosítók, amelyek lehetővé teszik több szintet kapcsolatokat, például a navigálni ügyfélalkalmazás biztosításához */customers/1/orders/99/products* számára Szerezze be az ügyfél 1 elhelyezett 99 sorrendben termékek listáját. Azonban ez a szint összetettségi nehéz lehet karbantartása és rugalmatlan, ha erőforrásainak kapcsolatai később módosíthatja. Ehelyett kell törekedni tartsa URI-azonosítók viszonylag egyszerű. Figyelembe kell vennie, hogy az alkalmazás legalább egy erőforráshoz, amennyiben lehetővé kell tenni ezt a hivatkozást használata erőforráshoz kapcsolódó elemek kereséséhez. Az előző lekérdezés az URI azonosító lehet cserélni */customers/1/orders* található összes ügyfél 1 rendelés, majd lekérdezheti és az URI */orders/99/products* (feltéve, sorrendben sorrendje a termékek kereséséhez 99 helyezte ügyfél 1).

> [!TIP]
> Kerülni, hogy az erőforrás URI-azonosítók eddigieknél még bonyolultabbá kelljen *gyűjtemény/elemgyűjtemény*.
>
>

Egy másik és megfontolandó szempont, hogy minden webes kérésnek ugyanazok a kiszolgáló terhelése, de minél nagyobb a kérelmek száma nagyobb a terhelést. Meg kell próbálni határozza meg az erőforrások "chatty" webes API-k kis erőforrások nagy számú visszaállítását elkerülése érdekében. Ilyen az API-k több kérelmek elküldése a szükséges adatok ügyfélalkalmazás lehet szükség. Akkor lehet hasznos adatok denormalize és egyesítése egyetlen kérelmet lekérhetők nagyobb erőforrások együtt történő kapcsolódó információk. Azonban kell ezt a módszert használja a terhelést növelni az adatok, amelyek nem gyakran szüksége lehet az ügyfél beolvasása elleni elosztása érdekében. Nagy objektumok beolvasása növelje a Tiltás késése a kérelmet, és fel Önnek további sávszélességgel kapcsolatos költségek származik sok előnye az, ha a további adatok nem gyakran használják.

Ne vezet be, a webes API-t a struktúra, típus vagy hely alapul szolgáló adatforrások közötti függőségek. Például ha az adatok egy relációs adatbázisban található, a webes API nem kell olyan erőforrások gyűjteménye, mint minden tábla közzétételét. Gondolja át, hogy a webes API-t, az adatbázis absztrakciós, és szükség esetén szükség leképezési réteg az adatbázis és a webes API-k között. Ily módon, ha megváltozik a Tervező vagy az adatbázis végrehajtásának (például a védelemről egy relációs adatbázisban, egy nosql-alapú denormalizált tárolórendszer, például egy dokumentum-adatbázis normalizált táblázatokat gyűjteményét tartalmazó) ügyfélalkalmazások megfelelően a módosítások.

> [!TIP]
> Egy webes API ösztönzője adatok forrása nem kell a tárolóban; Ez lehet egy másik szolgáltatás vagy sor üzleti alkalmazás vagy akár egy örökölt alkalmazást futtató helyi szervezeten belül.
>
>

Végül hogy előfordulhat, hogy nem lehet megfeleltetni minden műveletet a webes API-k egy adott erőforrás által megvalósított. Például kezelheti *nem erőforrás-* forgatókönyvek meghívni egy adott funkciót, és térjen vissza az eredményeket egy HTTP-válaszüzenetnek, HTTP GET kérelmek keresztül. A webes API-ja, amely megvalósítja az egyszerű számológép stílusú műveletek, mint hozzáadása, és kivonás teszi közzé a pszeudo erőforrásként ezeket a műveleteket, és a lekérdezési karakterlánc szükséges paramétereket felhasználását URI azonosítók biztosítani. Például egy GET kérelem URI */ add? operand1 99 & operand2 = = 1* képes vissza válaszüzenetet 100 értéket tartalmazó szervezethez, és az URI kérelmek *kivonás /? operand1 = 50 & operand2 = 20* 30 értéket tartalmazó törzsű sikerült vissza válaszüzenetet. Azonban csak ezek forma közül választhat URI-azonosítók módjával.

### <a name="defining-operations-in-terms-of-http-methods"></a>HTTP-metódus műveletek meghatározása
A HTTP protokoll számos módszer, amely a szemantikai jelentés hozzárendelése egy kérelem határozza meg. A legtöbb RESTful webes API-k által használt közös HTTP megoldások a következők:

* **ELSŐ**, beolvasni az erőforrás a megadott URI-t egy példányát. A szervezet a válaszüzenet a kért erőforrás részleteit tartalmazza.
* **POST**, új erőforrás létrehozása a megadott URI-t. A kérelem üzenet törzsét részletesen bemutatja az új erőforrás. Vegye figyelembe, hogy POST is használható, amelyek ténylegesen ne hozzon létre erőforrások műveleteket indítsanak.
* **PUT**, és cserélje le a megadott URI azonosító helyen található erőforrás frissítése. A kérelem üzenet törzsét határozza meg, a módosítani kívánt erőforrás és az alkalmazni kívánt értékeket.
* **Törlés**, hogy távolítsa el az erőforrás a megadott URI-t.

> [!NOTE]
> A HTTP protokollt is meghatároz más kevesebb mint a gyakran használt módszerek, például a javítás, amellyel szelektív frissítést lehessen egy erőforrást, HEAD, amely kérhet egy erőforrás leírása, beállítások, amely lehetővé teszi a rájuk vonatkozó információ beszerzéséhez egy ügyfél-információt kérő a kommunikációs lehetőségekről, a kiszolgáló és a NYOMKÖVETÉSI, amely lehetővé teszi a tesztelés és diagnosztikai célokra használhat kérelmekkel kapcsolatos információ az ügyfél által támogatott.
>
>

Egy adott kérelem hatásának függ-e az erőforrást, amely érvényben van a gyűjtemény vagy egy egyéni elemet. A következő táblázat összefoglalja az kereskedelmi példát a RESTful megvalósítások által elfogadott közös szabályok. Vegye figyelembe, hogy ezek a kérelmek nem az összes is elegendő lehet; az adott forgatókönyv függ.

| **Erőforrás** | **POST** | **GET** | **A PUT** | **TÖRLÉSE** |
| --- | --- | --- | --- | --- |
| /Customers |Hozzon létre egy új ügyfél |Összes ügyfél beolvasása |Tömeges frissítés vevők (*Ha megvalósított*) |Távolítsa el az összes ügyfél számára |
| / ügyfelek/1 |Hiba |Ügyfél 1 adatainak beolvasása |A adatainak frissítéséhez ügyfél 1 Ha létezik, vagy visszatérési hiba |Távolítsa el az ügyfél 1 |
| /Customers/1/orders |Hozzon létre egy új ahhoz, hogy az ügyfél 1 |Ügyfél 1 összes rendelés beolvasása |Tömeges frissítés rendelések ügyfél 1 (*Ha megvalósított*) |Távolítsa el az összes rendelés ügyfél 1 (*Ha megvalósított*) |

A GET és DELETE kérelmek célját viszonylag egyszerű, de a cél és a POST és a PUT kérelmek hatásait kapcsolatos zavarok hatóköre van.

Egy POST kérést hozzon létre egy új erőforrást a kérés törzsében megadott adatokkal. A többi modell gyakran alkalmaz POST-kérésnél az erőforrásokat, amelyek a gyűjtemények; az új erőforrás van hozzá a gyűjteményhez.

> [!NOTE]
> Azt is megadhatja a POST kérelmek, amelyek indul el, hogy bizonyos funkciók (és, amely nem feltétlenül vissza adatokat), és az ilyen típusú kérelmet gyűjteményekre alkalmazhatók. Használhat például egy POST kérést a időrendben átadása egy bérlista-feldolgozó szolgáltatás, és a számított adók vissza válaszként.
>
>

Egy PUT-kérelmekben célja, hogy módosítsa a meglévő erőforrás. Ha a megadott erőforrás nem létezik, a PUT kérés visszaadhatja hiba (néhány esetben az lehet, hogy ténylegesen létrehozása az erőforrás). PUT kérelmek leggyakrabban érvényesek erőforrásokhoz (például egy adott ügyfélhez vagy a sorrendben), az egyes elemek bár alkalmazhatók a gyűjteményekhez, bár ez általában kevésbé van megvalósítva. Vegye figyelembe, hogy a PUT kérelmek idempotent, mivel a POST kérelmek nem; Ha egy alkalmazás elküldi a azonos PUT kérés többször az eredmények mindig meg kell egyeznie (az azonos erőforrás módosulnak ugyanazokat az értékeket), de ha egy alkalmazás ismétlődik a azonos POST kérelem eredménye több erőforrás létrehozása.

> [!NOTE]
> Szigorúan fogalmazva egy HTTP PUT-kérelmet meglévő erőforrás lecseréli a kérés törzsében megadott erőforrás. Ha nem módosítja a kiválasztott erőforrás-tulajdonságokat, de más tulajdonságok változatlanul hagyja kívánja, majd ezt kell végrehajtani egy javítás HTTP-kérelem használatával. Számos RESTful megvalósítását azonban ez a szabály enyhíteni, és mindkét szituációkban PUT használható.
>
>

### <a name="processing-http-requests"></a>HTTP-kérelmek feldolgozása
Egy ügyfélalkalmazás a webkiszolgálóról a HTTP-kérelmekre, és a megfelelő válaszüzenetek által tartalmazott adatokkal sikerült jelenik meg a különböző formátumokban (vagy az adathordozók típusairól). Például az adatokat, amely meghatározza egy ügyfél vagy a rendelés részleteit XML, JSON vagy valamilyen más kódolt és tömörített formátumú lehet megadni. A RESTful webes API támogatnia kell a különböző típusú kérést ügyfél alkalmazás által kért.

Amikor egy ügyfél-alkalmazás, amely visszaadja az adatokat egy üzenet törzsében kérelmet küld, az adathordozók típusairól is képes kezelni az elfogadás a kérelem fejlécében határozhatja meg. Az alábbi kód bemutatja a HTTP GET kérelemre, amely lekéri 2 rendelés részleteit, és az eredményt a adódik vissza (az az ügyfél továbbra is kell vizsgálni az adathordozó-típusát a visszaadott adatok formátumának ellenőrzése a válaszban szereplő adatok) JSON-kérelmek :

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

Ha a webkiszolgálón az adathordozó-típus, megválaszolhatja azt egy választ, amely magában foglalja a Content-Type fejléc, amely megadja az adatok formátumát az üzenet törzsében:

> [!NOTE]
> A maximális való együttműködés az adathordozót az elfogadás és a Content-Type fejléc a hivatkozott típusok kell lennie, hanem néhány egyéni médiatípus felismerni MIME-típusok.
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

A webalkalmazás-kiszolgáló nem támogatja a kért adathordozó-típust, ha más formátumban is küldheti az adatokat. Minden esetben meg kell határoznia azt az adathordozó típusát (pl. *az application/json*) a Content-Type fejléc. A feladata az ügyfélalkalmazás elemzi a válaszüzenetet, és az eredményeket az üzenettörzs megfelelően értelmezni.

Ne feledje, hogy ebben a példában a webalkalmazás-kiszolgáló sikeresen kéri le a kért adatokat úgy, hogy a válaszfejlécet vissza 200 állapotkódot sikert jelez. Ha nem egyező adatokat, ehelyett az kell visszaadnia (nem található) 404 állapotkódot, és a válasz üzenet törzsét további információkat is tartalmazhat. Ez az információ formátuma van megadva a Content-Type fejléc a következő példában látható módon:

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

Rendelés 222 nem létezik, ezért a válaszüzenetet néz ki:

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

Amikor egy alkalmazás egy erőforrás frissítése egy HTTP PUT-kérelmet küld, azt határozza meg az erőforrás URI, és adja meg a kérelem üzenet törzsét módosítani adatait. Akkor kell is használatával adja meg az adatok formátumát az a Content-Type fejléc. Egy általános szöveges információ használt formátuma *alkalmazás/x-www-form-urlencoded*, amely magában foglalja a elválasztott név/érték párok halmaza a & karakter. A következő példa bemutatja, egy HTTP PUT kérelmet, amely 1 sorrendben adatainak módosítása:

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

Sikeres a módosítás esetén azt kell ideális válaszol egy HTTP 204-es állapotkód, amely azt jelzi, hogy a folyamat sikeresen kezelve van azonban, hogy az adott válasz törzsének tartalmazza-e további információ. A hely egy fejléc a következő a válasz URI-azonosítója az újonnan frissített erőforrás tartalmazza:

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> Ha egy HTTP PUT kérelemüzenet dátum-és szerepel, ellenőrizze, hogy a webszolgáltatás dátumok fogad-e, és időpontokban formátuma a következő az ISO 8601 szabványnak.
>
>

Ha frissítenie kell az erőforrás nem létezik, a webalkalmazás-kiszolgáló nem talált választ válaszolhassanak, a fentebb leírt módon. Azt is megteheti Ha a kiszolgáló tényleges hoz létre az objektum az sikerült visszaadnia állapotát kódok HTTP 200 (OK) vagy HTTP 201-es (létrehozva) és az adott válasz törzsének tartalmazhatnak az adatokat az új erőforrás. Ha a kérelemben Content-Type fejlécében a webkiszolgáló nem tudja kezelni adatok formátuma nem határoz meg, azt kell válaszolnia HTTP-állapotkód 415 (nem támogatott adathordozó-típust).

> [!TIP]
> Vegye figyelembe, hogy csak egy erőforráshoz egy gyűjtemény frissítések kötegelhető tömeges HTTP PUT műveletek végrehajtása. A PUT kérés URI-azonosítója a gyűjteményt kell megadnia, és a kérés törzsében adja meg a módosítani kívánt erőforrások részleteit. Ez a megközelítés segíthet chattiness csökkentéséhez és a teljesítmény javításához.
>
>

A formátuma, hogy hozzon létre új erőforrásokat egy HTTP POST-kérelem hasonlóak a PUT kérelmek; az üzenettörzs tartalmazza az új erőforrás lehet hozzáadni. Azonban az URI általában határozza meg, a gyűjteményt, amelyhez az erőforrás hozzá kell adni. Az alábbi példa létrehoz egy új rendelést, és hozzáadja azt a rendeléseket gyűjteményhez:

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

Ha a kérelem sikeres, a webkiszolgáló egy HTTP-állapotkód (létrehozva) 201 üzenet kódot kell válaszolnia. A helyet megjelölő fejlécet az újonnan létrehozott erőforrás URI kell tartalmaznia, és a választörzs tartalmaznia kell egy példányát az új erőforrás; a Content-Type fejléc megadja az adatok formátumát:

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> A PUT vagy POST kérelem által megadott adat érvénytelen, ha a webkiszolgáló egy üzenetet, amelyben a HTTP-állapotkód: 400 (hibás kérés) kell válaszolnia. Ez az üzenet törzsét a kérelem és a várt formátumok a problémával kapcsolatos további információkat is tartalmazhat, vagy tartalmazhat egy hivatkozást egy URL-címet, amely további adatokat.
>
>

Eltávolít egy erőforrást, egy HTTP DELETE kérelmet egyszerűen biztosít, az erőforrás URI törölhető. Az alábbi példa megpróbálja sorrend 99 eltávolítása:

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

A törlési művelet végrehajtása sikeres, ha a webkiszolgáló válaszolnia kell arról, hogy a folyamat sikeresen kezelve van azonban, hogy az adott válasz törzsének tartalmazza-e további információ nem (Ez ugyanaz a válasz egy sikeres által visszaadott HTTP-állapotkód: 204, PUT művelet, de az erőforrás helye fejléc nélküli már nem létezik.) Akkor is a DELETE kérelmek HTTP-állapotkód 200 (OK) vagy 202 (elfogadható) vissza, ha a törlés aszinkron módon történik.

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

Ha az erőforrás nem található, a webkiszolgáló helyette 404-es (nem található) üzenet kell visszaadnia.

> [!TIP]
> Ha egy gyűjtemény összes erőforrása kell törölni, engedélyezése egy HTTP DELETE kérelmet kell adni az URI-azonosítója egy alkalmazást az egyes erőforrások pedig eltávolítása a gyűjteményből újraindítás helyett a gyűjteményben.
>
>

### <a name="filtering-and-paginating-data"></a>Szűrés és paginating adatok
Meg kell tartani az URI-k egyszerű és intuitív endeavor. Az ilyen egyetlen erőforrások gyűjteményét URI ezzel segít, de nagy mennyiségű adat beolvasása, amikor szükség az információkat csak egy részhalmazát alkalmazások vezethet. A nagy forgalom létrehozása hatással van, nem csak teljesítményét és méretezhetőségét, a webkiszolgáló, de is hátrányosan befolyásolhatja a a figyelt alkalmazások az adatokat kér-e.

Például ha rendelések sorrendje fizetett tartalmazhat, egy ügyfélalkalmazást, amelyet a költségekkel keresztül megadott értékkel rendelkező összes rendelés beolvasása módosítania kell beolvasni az összes rendelést a */rendelések* URI és szűréssel ezeket a rendeléseket helyileg. Egyértelműen a folyamat nem nagyon hatékony; a webes API-t futtató kiszolgálón hulladékok azt a hálózati sávszélesség és a feldolgozási kapacitást.

Lehet, hogy egyik megoldást egy URI-séma biztosításához */kérelmek/ordervalue_greater_than_n* ahol  *n*  rendelés ár, de a díjak csak korlátozott számú kivételével megközelítéssel van nem célszerű. Emellett lekérdezés rendelések más feltétel alapján kell, ha fejezheti be alatt szembesül hosszú listáját tartalmazó URI-azonosítók valószínűleg nem intuitív nevű biztosítása.

Jobb stratégia adatok szűrése, hogy adja meg a szűrési feltételeket a lekérdezési karakterláncban a webes API-hoz, például a átadott */rendelések? ordervaluethreshold = n*. Ebben a példában a megfelelő műveletet, a webes API-t a felelős a elemzése és kezelése a `ordervaluethreshold` paraméter a lekérdezési karakterláncot, és a szűrt eredmények visszaadni a HTTP-válasz.

Néhány egyszerű HTTP GET kérelmek keresztül gyűjtemény-erőforrások potenciálisan visszaadhatja nagyszámú elemet. A folyamatban, akkor lehetősége elleni tervezzen a webes API-t bármely egyetlen kérelem által visszaadott adatok korlátozásához. Lehet ezt elérni azáltal támogatja a lekérdezési karakterláncok, amelyek lehetővé teszik a kérhető elemek maximális számát adja meg (amely maga képezhetik egy felső korlátot szolgáltatásmegtagadási támadások megelőzése érdekében), és a kezdő eltolása a gyűjteményhez. Például a lekérdezési karakterlánc URI azonosítójában */rendelések? korlát 25 & eltolás = = 50* kezdődő, és a rendeléseket gyűjteményében található 50 sorrendje 25 rendelések kell beolvasni. Mivel az adatok szűrése, a művelet, amely a GET kérelmet a webes API felelős elemzése és kezelése a `limit` és `offset` paraméterek a lekérdezési karakterláncban. Segít az ügyfélalkalmazások, GET kérések többoldalas adatokat is tartalmaznia kell a metaadatok valamilyen visszaadó jelző érhető el a gyűjteményben lévő erőforrások teljes száma. Emellett érdemes valamilyen más intelligens lapozás stratégiák; További információkért lásd: [API tervezési megjegyzések: intelligens lapozás](http://bizcoder.com/api-design-notes-smart-paging)

Hajtsa végre az adatok rendezése, lehívása; hasonló stratégia biztosíthatja, hogy egy rendezési paraméter, amely a mezőnév értékeként, például a */rendelések? rendezési = ProductID*. Azonban ügyeljen arra, hogy ez a megközelítés lehet gyorsítótárazást (a lekérdezési karakterlánc paramétereket kulcsként számos gyorsítótár megvalósítását által a gyorsítótárazott adatokat az erőforrás-azonosító részét képezik) káros hatással.

Bővítheti ezt a módszert a korlátot (projekt) mezők adott vissza, ha csak egyetlen elemet tartalmazza-e nagy mennyiségű adatot. Például használhatja a lekérdezési karakterlánc paraméterként, amely a mezőket, vesszővel tagolt listáját fogadja el, mint */rendelések? mezők ProductID, mennyiség =*.

> [!TIP]
> Adjon meg a lekérdezési karakterláncok a választható paramétereket jelentéssel bíró alapértelmezett értékeket. Például a `limit` 10 paraméter és a `offset` paraméter 0 értékre tördelési, ha a rendezési paraméter értéke az erőforrás kulcsa, amennyiben rendezést alkalmazza, és állítsa be a `fields` paramétert az erőforrás található összes mezőhöz Ha Ön támogatja a leképezések.
>
>

### <a name="handling-large-binary-resources"></a>Nagy bináris erőforrások kezelése
Egyetlen nagy bináris mezőkön, például a fájlok vagy a képek is tartalmazhat. Megoldásához, az átviteli problémák megbízhatatlanná és időszakos kapcsolatok által okozott és javításához válaszidejét, próbáljon műveleteket, amelyek lehetővé teszik az ezekhez az erőforrásokhoz az ügyfélalkalmazás által adattömbök kérhető. Ehhez az szükséges, a webes API-t kell támogatja az elfogadás-tartományokat fejlécet nagy erőforrások GET kérelmek, és ideális a HTTP HEAD kérések megvalósításához. Az elfogadás-tartományokat fejléc azt jelzi, hogy támogatja-e a GET műveletet részleges eredményt, és, hogy egy ügyfél-alkalmazás elküldheti a GET-kérésekhez megadott bájttartomány erőforrás részhalmazát adja vissza. Egy HEAD kérelem hasonlít a GET kérés, azzal a különbséggel, hogy csak egy fejlécet tartalmazta, amely ismerteti az erőforrás- és egy üres üzenettörzs adja vissza. Egy ügyfélalkalmazás adhatnak ki olyan HEAD kérelem annak megállapításához, hogy egy erőforrást beolvasni a részleges GET kérelmek használatával. A következő példa bemutatja, hogy beolvassa a termék lemezképére vonatkozó információ HEAD kérelem:

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

A válaszüzenetet, amely tartalmazza az erőforrás (4580 bájt) mérete fejlécet, valamint az, hogy a megfelelő GET műveletet támogatja-e a részleges eredmények elfogadás-tartományokat fejléc tartalmazza:

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

Az ügyfélalkalmazás ezen információk használatával hozható létre a kisebb csoportjai képének beolvasása GET műveletek sorozata. Az első kérelem első 2500 bájtok beolvassa a tartomány fejléc használatával:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

A hibaválasz-üzenet jelzi, hogy a részleges válasz HTTP-állapotkód 206 vissza. A Content-Length fejlécet az üzenet törzsében (nem az erőforrás mérete) visszaadott bájtok tényleges számát adja meg, és a Content-Range fejléc azt jelzi, hogy melyik erőforrás része azt (0-2499 kívüli 4580. bájt):

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

Az ügyfélalkalmazás egy későbbi kérés egy megfelelő tartományon fejléc használatával lekérhető az erőforrás fennmaradó:

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

A megfelelő eredmény üzenetet kell kinéznie:

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a>A HATEOAS módszer segítségével lehetővé a navigációt kapcsolódó erőforrások
Az elsődleges összefüggések mögött REST egyik, hogy lehetővé kell tenni az erőforrások teljes készletét megnyitása anélkül, hogy az URI-séma előzetes ismerete. Minden HTTP GET kérést kell visszaadnia az információk szükségesek ahhoz, hogy a közvetlenül a kért objektum keresztül a válaszban szereplő hivatkozások kapcsolódó erőforrás megkeresése, és azt is rendelkezniük kell az elérhető műveletek leíró adatokat minden egyes ezeket az erőforrásokat. Ez az elv HATEOAS, vagy az alkalmazás motor állapot Hypertext néven ismert. A rendszer gyakorlatilag egy véges állapotjelző gép, és az egyes irányuló kérelemre adott válasz tartalmazza a szükséges áthelyezése egy másik; adatok nincs más információ szükséges kell lennie.

> [!NOTE]
> Jelenleg nincsenek szabványok vagy meghatározhatja, hogyan kell a HATEOAS elv modell specifikációk. Az ebben a szakaszban szereplő példák bemutatják, egy lehetséges megoldás.
>
>

Tegyük fel a vásárlók és a rendeléseket, közötti kapcsolat kezelése a kívánt sorrendjét a válaszban visszaadott adatok kell tartalmaznia, és az ügyfél a rendelést, amely a végrehajtható műveletek azonosító hivatkozás formájában URI ügyfél.

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

A hibaválasz-üzenet törzsében egy `links` tömb (kiemelt a kódpéldát), amely megadja a kapcsolat jellege (*ügyfél*), az ügyfél az URI (*http://adventure-works.com/ az ügyfelek/3*), hogyan lehet lekérni a felhasználói adatait (*beolvasása*), és a MIME-típusok, amely a webalkalmazás-kiszolgáló támogatja az adatok beolvasása (*text/xml* és *az application/json*). Ez az összes adatot, amelyet a ügyfélalkalmazás beolvashatja az ügyfél részletes adatait. Emellett a hivatkozások tömb hivatkozások is végrehajtható, például a PUT (az ügyfél és a formátuma, hogy a webkiszolgáló az ügyféltől vár módosítandó), és törölje a többi művelet.

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

A teljesség kedvéért a hivatkozások tömb is beolvasása erőforrás önmagára hivatkozó információkat kell tartalmaznia. Ezeket a hivatkozásokat az előző példából kimaradtak, de a következő kódban vannak kiemelve. Figyelje meg, hogy ezeket a hivatkozásokat, a kapcsolat *önkiszolgáló* azt jelzi, hogy ez az erőforrás a művelet által visszaadott hivatkozás használatban van:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

Ezzel a megközelítéssel hatékony ügyfélalkalmazások lekérni, és ez az információ elemezni kell készíteni.

## <a name="versioning-a-restful-web-api"></a>Versioning egy RESTful webes API
Nagyon valószínű, hogy az összes olyan helyzetekben, hogy a webes API statikus marad a legegyszerűbb. Üzleti követelmények módosítása az új gyűjtemények erőforrások vehetők, erőforrásainak kapcsolatai változhat, és erőforrásokat az adatok szerkezete, előfordulhat, hogy módosítani kell. Amíg a webes API-k új vagy eltérő követelmények kezelésére frissítése egy viszonylag egyszerű folyamat, meg kell fontolnia, hogy a változások rendelkezik majd a webes API-t használó ügyfélalkalmazások által okozott hatások. A probléma oka, hogy a fejlesztő tervezésével és megvalósításával webes API-k, hogy API keresztül teljes körű vezérléssel rendelkezik, bár a fejlesztői nincs szabályozhatják, előfordulhat, hogy kialakítani, harmadik féltől származó szervezetek működő távoli ügyfél-alkalmazással azonos szinten. Az elsődleges imperatív a meglévő ügyfél alkalmazások is működjenek, miközben lehetővé teszi az új ügyfél alkalmazások új szolgáltatásait és az erőforrások kihasználását változatlan.

Versioning lehetővé teszi, hogy a webes API-k jelzi a következő szolgáltatásokat és erőforrásokat, amely azt mutatja, és egy ügyfélalkalmazás kérelmezheti irányított egy szolgáltatás vagy az erőforrás egy adott verziójához. A következő szakaszok ismertetik, amelyek mindegyikének saját előnyei és vonzatai több különböző módon.

### <a name="no-versioning"></a>Semmilyen verziószámozási
Ez a legegyszerűbb módszere az, és egyes belső API-k elfogadható. Nagy változások sikerült jeleníthető meg új erőforrások vagy mutató új hivatkozásokat.  Tartalom hozzáadása a meglévő erőforrásokkal nem találhatók használhatatlanná tévő változást, ügyfélalkalmazások, amelyek nem számított a tartalom egyszerűen figyelmen kívül hagyja azt.

Például az URI kérelem *http://adventure-works.com/customers/3* visszaadja-e részletes adatait tartalmazó egyetlen ügyfél `id`, `name`, és `address` az ügyfélalkalmazás által várt mezők :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> Az egyszerűség és egyértelműség céljából az itt látható példa válaszok nem HATEOAS mutató hivatkozásokat tartalmaznak.
>
>

Ha a `DateCreated` mező hozzáadódik a séma, a felhasználói erőforrás, akkor a válasz néz ki:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Meglévő ügyfélalkalmazások továbbra is megfelelően működik, ha azok figyelmen kívül hagyása ismeretlen mezők, amíg új ügyfélalkalmazások is kell megtervezni, hogy ez az új mező kezelésére képes. Azonban a sémában az erőforrások több legradikálisabb változások történnek (például eltávolítása, vagy mezők átnevezése) vagy a erőforrásainak kapcsolatai módosítani majd ezek jelenthet akár jelentős változásokat, amelyek meggátolják a meglévő ügyfél alkalmazások megfelelően működik-e . Ezekben az esetekben érdemes lehet a következő módszerek egyikét.

### <a name="uri-versioning"></a>URI versioning
Minden alkalommal, amikor módosítja a webes API-t vagy módosítja a sémát, az erőforrások hozzáadása egy verziószámot minden erőforrás URI. A korábban meglévő URI-k továbbra is működnek, mint korábban tartó erőforrások visszatér az eredeti séma.

Az előző példában kiterjesztése, ha a `address` mező új alárendelt mezőkbe tartalmazó minden részét képezi a cím szerkezete (például `streetAddress`, `city`, `state`, és `zipCode`), az erőforrás ezen verziója lehet egy verziószámot, például a http://adventure-works.com/v2/customers/3 tartalmazó URI keresztül közzétett:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

A verziókezelő mechanizmus nagyon egyszerű, de a kiszolgáló a kérés útválasztási a megfelelő végpontnak függ. Azonban, a webes API Miután kiforrottá válik, több lépés keresztül kezelése nehézkessé válhat, és a kiszolgáló rendelkezik egy különböző verziói számának támogatásához. Is, egy purist szempontjából, minden esetben az ügyfélalkalmazások számára vannak a azonos adatok beolvasása (ügyfél 3), így az URI nem valóban szabad verziójától függően eltérnek. Ez a séma HATEOAS végrehajtását is növeli az összes hivatkozás kell ahhoz, hogy a verziószám szerepeljen az URI-azonosítók szerint.

### <a name="query-string-versioning"></a>Lekérdezési karakterlánc versioning
Ahelyett, hogy így több URI-k, megadhatja a verziót az erőforrás egy paraméterrel, a lekérdezési karakterláncot, mint a HTTP-kérelem fűzött belül *http://adventure-works.com/customers/3?version=2*. A version paramétert kell alapértelmezés szerint egy jelentéssel bíró értékre, például 1, ha ki van hagyva régebbi ügyfélalkalmazások.

Ez a megközelítés azzal az szemantikai előnye, hogy ugyanaz az erőforrás mindig lekért ugyanilyen URI, de a kódot, amely kezeli a kérelmet a lekérdezési karakterlánc elemzése és a megfelelő HTTP-választ küldött függ. Ez a megközelítés is szenved az azonos komplikációk URI versioning mechanizmusként HATEOAS megvalósításához.

> [!NOTE]
> Egyes régebbi webböngészők és a webalkalmazás-proxy nem gyorsítótárazhatják az választ adjon az URL-CÍMÉT egy lekérdezési karakterláncot tartalmazó kérelmeket. Ez kedvezőtlen hatással lehet webes alkalmazásokhoz, amelyek a webes API-k használják, és futtatni egy webes böngésző belül, amely teljesítmény.
>
>

### <a name="header-versioning"></a>Fejléc versioning
Ahelyett, hogy hozzáfűzi a lekérdezési karakterlánc paraméterként a verziószámot, egy egyéni fejlécet, amely jelzi a verziót az erőforrás sikertelen végrehajtása. Ez a módszer igényli, hogy, hogy az ügyfélalkalmazás a megfelelő fejlécet ad a kéréseit, bár a kódot az ügyfél kérelmet kezelő használhatja az alapértelmezett értéket (verzió: 1), ha meg van adva a verziófejléc. Az alábbi példák használatára egy elnevezésű egyéni fejlécet *egyéni-fejléc*. A fejléc értékének webes API verzióját adja meg.

1. verzió:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

2. verzió:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Vegye figyelembe, hogy az előző két megközelítés a HATEOAS végrehajtási igényel a együtt megfelelő egyéni szereplő esetleges hivatkozások.

### <a name="media-type-versioning"></a>Az adathordozó típusa versioning
Amikor egy ügyfél-alkalmazás egy HTTP GET kérést küld egy webkiszolgálóhoz történő azt kell kötni, is képes kezelni az Accept fejlécet, használatával, ez az útmutató korábbi szakaszaiban ismertetett tartalom formátuma. Gyakran a célja a *elfogadás* fejléc, hogy lehetővé tegyék az ügyfélalkalmazás adhatja meg, hogy a választörzs XML, JSON vagy valamilyen más általános formátumú, mely az ügyfél képes legyen-e. Azonban úgy is, amely tartalmazza az információt, amely alapján az ügyfélalkalmazás annak jelzésére, hogy melyik erőforrás verzióját is megfelelő egyéni adathordozó-típust határoznak meg. A következő példa bemutatja egy olyan kérelmet, amely meghatározza egy *elfogadás* értékű fejléc *application/vnd.adventure-works.v1+json*. A *vnd.adventure-works.v1* elem határozza meg, hogy a webkiszolgáló, hogy az visszaküldje-e az 1 verziót az erőforrás, amíg a *json* elem megadhatja, hogy az adott válasz törzsének formátuma JSON:

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

A kódot, a kérelem feldolgozása felelős a *elfogadás* fejléc és a lehető érvényesítenie azt (az ügyfélalkalmazás adhatnak meg több formátumok a *elfogadás* fejléc, ebben az esetben a webalkalmazás-kiszolgáló is válassza ki a legmegfelelőbb az adott válasz törzsének formátumát). A webkiszolgáló megerősíti, hogy a válasz törzsében szereplő adatok formátumának a Content-Type fejléc használatával:

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
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

- A OpenAPI megadását a opinionated irányelvek állítja be a hogyan egy REST API-t úgy kell megtervezni, előre. Amely a való együttműködés előnye is van, de további figyelmet igényel, az API-t a specifikációnak megfelelő tervezésekor.
- OpenAPI elősegíti a szerződés-első megközelítés ahelyett, hogy egy megvalósítási-first-módszert is. Adategyezmény-első azt jelenti, hogy alakítson ki az API-t először szerződést (a felület), és jegyezze meg a szerződés megvalósító kódot. 
- Eszközök, például a Swagger API-szerződések hozhat létre ügyfélkódtárai vagy dokumentációját. Lásd például: [ASP.NET API segítségével weblapok a Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).

## <a name="more-information"></a>További információ
* A [Microsoft REST API-irányelvek](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) tartalmaz részletes információkat találhat a nyilvános REST API-k tervezéséhez.
* A [RESTful Cookbook](http://restcookbook.com/) RESTful API-k létrehozása bemutatása tartalmazza.
* A [webes API ellenőrzőlista](https://mathieu.fenniak.net/the-api-checklist/) megfontolandó tervezésével és megvalósításával webes API-k hasznos listáját tartalmazza.
* A [nyitott API kezdeményezésére](https://www.openapis.org/) helyet, a nyílt API-t minden kapcsolódó dokumentáció és megvalósítási részleteit tartalmazza.
