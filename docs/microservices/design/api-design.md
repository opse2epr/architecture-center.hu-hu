---
title: API-tervezés
description: Mikroszolgáltatás-alapú API-k tervezése
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: c2c96a340465d11c89147e991617d0c1c526e236
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299189"
---
# <a name="designing-apis-for-microservices"></a>Mikroszolgáltatás-alapú API-k tervezése

Jó API-tervezés fontos a mikroszolgáltatási architektúrákban, mert az összes adatcsere-szolgáltatások közötti üzenetek, illetve az API-hívások keresztül történik. API-kat kell lennie, ne hozzon létre hatékony [forgalmas I/O](../../antipatterns/chatty-io/index.md). Mivel a szolgáltatások egymástól függetlenül működik-e csoportok készültek, API-k rendelkeznie kell jól definiált szemantika és a verziókezelés rendszerek, úgy, hogy a frissítések nem felosztása más szolgáltatások.

![A mikroszolgáltatás-alapú API-tervezés](../images/api-design.png)

Fontos, hogy két típusú API között:

- Nyilvános API-kat, amelyek meg az ügyfélalkalmazások számára.
- Háttérbeli API-k által használt szolgáltatások közötti kommunikáció.

E két használati esetek némiképp eltérő követelmények vonatkozhatnak. Nyilvános API-t az ügyfélalkalmazások, általában böngészőalkalmazásokban vagy natív mobilalkalmazások kompatibilisnek kell lennie. A legtöbb esetben azt jelenti, hogy a nyilvános API-t a REST használata HTTP-n keresztül. A háttérrendszeri API-khoz, az azonban Önnek kell tennie a hálózati teljesítmény-fiókba. A szolgáltatások részletességétől függően közötti kommunikációt eredményezhet nagy hálózati forgalmat eredményezhet. Szolgáltatások gyorsan válhat i/o kötve. Éppen ezért több fontossá válik például a sebesség és a hasznos adat mérete szerializálási szempontok. Néhány népszerű más REST használatával HTTP protokollon keresztül gRPC Apache Avro és Apache Thrift tartalmazza. Ezeket a protokollokat támogatja a bináris szerializálási, és általában hatékonyabb, mint a HTTP.

## <a name="considerations"></a>Megfontolandó szempontok

Az alábbiakban néhány dolgot kell vennie, amikor kiválasztja, hogyan valósíthat meg egy API-t.

**A REST és RPC**. Fontolja meg egy REST-stílusú felhasználói felületén és a egy RPC stílusú felület kompromisszumokat.

- REST-modellek erőforrásokat, amelyek a természetes módon lehet a tartományi modellben express. Meghatározza egy egységes felületet alapján a HTTP-műveletek, amelyek arra ösztönzi a evolvability. Idempotens, mellékhatása és válaszkódok jól definiált szemantikával rendelkezik. És állapot nélküli kommunikációhoz, amely javítja a skálázhatóságot érvényesít.

- RPC több orientált, műveleteket, illetve parancsok körül. RPC felületek néz ki helyi metódust hívja, mert vezethet, a túlságosan forgalmas API-k tervezéséhez. Azonban ez nem jelenti azt RPC forgalmas kell lennie. Csak akkor körültekintően járjon el a felületen tervezésekor kell.

Egy RESTful interfészről a leggyakrabban használt, amely REST JSON használatával HTTP protokollon keresztül. RPC stílusú illesztőfelületet számos népszerű keretrendszereket gRPC Apache Avro és Apache Thrift is vannak.

**Hatékonyság**. Fontolja meg a hatékonyság sebességét, a memória és a hasznos adat mérete tekintetében. A gRPC felületen általában gyorsabb, mint a többi HTTP protokollon keresztül.

**Definíció nyelven (IDL) csatoló**. Egy IDL szolgál a módszereket, paraméterek, és API-KHOZ értékeit adja vissza. Egy IDL Ügyfélkód szerializálási kód és API-dokumentáció létrehozásához használható. IDLs tesztelési eszközök, például a Postman API is képes használni. Keretrendszereket, mint a gRPC Avro és Thrift saját IDL specifikációk meghatározása. HTTP-n keresztül REST nincs szabványos IDL formátumban, de egy közös megválasztása OpenAPI (korábbi nevén Swagger). Egy HTTP REST API-t egy hivatalos munkafolyamatdefiníciós nyelv használata nélkül is létrehozhat, de majd elveszíti a kód létrehozását és a tesztelés előnyeit.

**Szerializálási**. Hogyan objektumok szerializálva vannak a hálózaton keresztül? A lehetőségek között (elsősorban JSON) formátumú szöveges és bináris formátumban, például a protokoll puffer. Bináris formátumok általában gyorsabb, mint a szöveges formátumban. Azonban JSON, együttműködési képességgel rendelkezik, előnyöket, mert a legtöbb nyelvet és keretrendszert támogatja a JSON-szerializálást. Némelyik szerializációs formátum van szükség rögzített sémát, és néhány fájl fordítása. Ebben az esetben szüksége ebben a lépésben a build-folyamatoknak építsen be.

**Keretrendszer és a nyelvi támogatás**. HTTP szinte minden keretrendszer és nyelv támogatott. gRPC, Avro és Thrift összes rendelkezik a könyvtárak a C++, C#, Java és Python. Thrift és gRPC is támogatja a Go.

**Kompatibilitás és együttműködés**. Ha például a gRPC protokoll, szükség lehet a nyilvános API-t és a háttérrendszer közötti protokoll fordítási rétegben. A [átjáró](./gateway.md) hajthat végre a kívánt függvényt. Ha egy szolgáltatás háló használ, fontolja meg, mely protokollok kompatibilisek-e a szolgáltatás háló. Például a linkerd HTTP Thrift és gRPC beépített támogatással rendelkezik.

Az alapkonfiguráció ajánljuk, hogy válassza ki a REST HTTP-n keresztül, kivéve, ha egy bináris protokoll által nyújtott van szüksége. HTTP-n keresztül REST speciális szalagtár van szükség. Hoz létre a minimális kapcsolási hívó nem kell egy ügyfél helyettes a szolgáltatással való kommunikációra. Nincs gazdag rendszereit eszközök támogatásához sémadefiníciók, tesztelése és figyelése a RESTful HTTP-végpontokat. Végül HTTP esetén webböngésző-ügyfelek számára, így Önnek nem kell egy protokoll fordítási réteg az ügyfél és a háttérrendszer között.

Azonban ha REST HTTP-n keresztül, kell hajtsa végre a teljesítmény és terheléses tesztelés, megállapítani, hogy e, illetve a forgatókönyv elvégzi a fejlesztési folyamat korai szakaszaiban.

## <a name="restful-api-design"></a>RESTful API-tervezés

Számos forrásanyag RESTful API-k tervezéséhez. Íme néhány hasznos lehet:

- [API-tervezés](../../best-practices/api-design.md)

- [API-implementáció](../../best-practices/api-implementation.md)

- [A Microsoft REST API-irányelvek](https://github.com/Microsoft/api-guidelines)

Az alábbiakban néhány konkrét szempontot figyelembe kell venni.

- Figyelje, hogy az API-k, amelyek nyilvánosságra kerüljenek a belső implementáció részleteit, vagy egyszerűen csak tükrözik az egy belső adatbázissémát. Az API-t a tartományt kell modellezniük. Szolgáltatások közötti szerződés, és új funkciók hozzáadásakor, nem csak, mert néhány kódot refactored, vagy egy adatbázistábla soraihoz normalized ideális csak módosítsa.

- Különböző típusú ügyfelet, például a mobilalkalmazás- és asztali webböngésző, a másik hasznos adat mérete vagy interakció minták lehet szükség. Fontolja meg a [háttérrendszerek és Előtérrendszerek minta](../../patterns/backends-for-frontends.md) külön háttérrendszerek létrehozásához, hogy az ügyfél számára tegye elérhetővé a optimális illesztőfelületet, minden ügyfél számára.

- A mellékhatással műveletek fontolja meg a minősítené idempotens, és azok megvalósítását, PUT-metódusok. Amely lehetővé teszi a biztonságos újrapróbálkozás, és javíthatja a rugalmasság. A cikk [szolgáltatások közötti kommunikáció](./interservice-communication.md) probléma részletesebben tárgyalják.

- HTTP-metódusok aszinkron szemantikát, ahol a metódus azonnal visszatér a választ, de a szolgáltatás aszinkron módon végrehajtja a műveletet is rendelkezhet. Ebben az esetben adja vissza a metódus egy [HTTP 202](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) válaszkód, ami azt jelenti, hogy a kérés elfogadva feldolgozásra, de a feldolgozása még nem fejeződött be.

## <a name="mapping-rest-to-ddd-patterns"></a>REST leképezése DDD-minták

Entitás, aggregációs és érték objektum hasonló minták célja, hogy bizonyos korlátozások az objektumok elhelyezése a tartományi modellben. DDD számos vitafórumokban a minták modellezése eltér az objektum-orientált (Fájlmegosztásain) nyelvi fogalmak például konstruktorok vagy tulajdonság beolvasókat és beállítókat használatával. Ha például *érték-objektumok* is lehet a nem módosítható. Egy Fájlmegosztásain programozási nyelven, akkor ennek kényszerítése a hozzárendelése az értékeket a konstruktorban, és a tulajdonságok csak olvasható:

```ts
export class Location {
    readonly latitude: number;
    readonly longitude: number;

    constructor(latitude: number, longitude: number) {
        if (latitude < -90 || latitude > 90) {
            throw new RangeError('latitude must be between -90 and 90');
        }
        if (longitude < -180 || longitude > 180) {
            throw new RangeError('longitude must be between -180 and 180');
        }
        this.latitude = latitude;
        this.longitude = longitude;
    }
}
```

A kódolási módszereket is számos különösen fontosak, egy hagyományos monolitikus alkalmazások készítése során. A nagy méretű kódbázis esetén számos alrendszerek használatakor előfordulhat, hogy a `Location` objektumot, ezért fontos helyes működése kényszerítése az objektumhoz.

Egy másik példa a tárház mintát, amely biztosítja, hogy az alkalmazás egyéb részei közvetlen olvasási vagy írási, vagy az adatok tárolására ne:

![Egy Drónt tárház ábrája](../images/repository.png)

A mikroszolgáltatási architektúra azonban szolgáltatások nem adjuk ki azonos kódbázisra és nem megosztásához adattárakban. Ehelyett kommunikáció API-kon keresztül. Fontolja meg az eset, ahol a Scheduler szolgáltatás kér a Drone szolgáltatásból egy drónt kapcsolatos információkat. A Drone szolgáltatásnak van egy drónt, ki, a kód, a belső modelljét. Azonban az ütemező nem látható. Ehelyett azt visszakap egy *ábrázolás* a drone entitás &mdash; HTTP-választ talán egy JSON-objektumában.

![A Drone szolgáltatás ábrája](../images/ddd-rest.png)

A Scheduler szolgáltatás nem lehet módosítani a Drone szolgáltatás belső modelljei, vagy a Drone szolgáltatás adattár írni. Ez azt jelenti, hogy a kódot, amely megvalósítja a Drone szolgáltatás rendelkezik egy kisebb elérhető felület, a hagyományos mikroszolgáltatásokká kódokkal képest. Ha egy helyen az osztály a Drone szolgáltatás, a hatókör osztály korlátozódik &mdash; egyetlen más szolgáltatás közvetlenül fog felhasználni a osztály.

Ebből kifolyólag ez az útmutató nem összpontosítani nagy részét a kódolási módszereket is, mivel a taktikai DDD-minták kapcsolódnak. De azt tapasztaltuk, hogy is modellezheti számos, a DDD-minták – REST API-kon keresztül.

Példa:

- Természetesen a térkép összesítések *erőforrások* a REST-ben. Például a Tartalomkézbesítési összesítés lenne kell által elérhetővé tett erőforrásként a kézbesítési API.

- Összesítések konzisztencia határok. Műveletek az összesítések soha nem hagyja aggregátum inkonzisztens állapotban. Ezért kerülje létrehozása API-k, amelyek lehetővé teszik az ügyfél segítségével kezelheti a belső állapotot az összesítést. Ehelyett alkalmazást coarse-grained API-k, amelyek összesítések erőforrásként teszik közzé.

- Az entitásnak egyedi azonosítói. REST, az erőforrások URL-címek formájában egyedi azonosítók rendelkeznek. Erőforrás URL-címek, amelyek megfelelnek egy entitás tartomány identitás létrehozása. Előfordulhat, hogy a tartomány identitás URL-címről a hozzárendelést átlátszatlan ügyfélnek.

- Gyermekentitások az összesítést a gyökérszintű entitás navigálva érhető el. Ha követi [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) elvek, gyermekentitások is ügyfélszolgálatán keresztül érhető el a szülőentitás reprezentációja hivatkozásokat.

- Érték objektumok nem módosíthatók, mert a frissítések a teljes értéket objektum lecserélésével történik. REST, a megvalósítása a PUT vagy PATCH-kérések frissítéseit.

- Egy adattár lehetővé teszi, hogy az ügyfelek lekérdezés hozzáadása vagy eltávolítása az objektumok egy gyűjteményben paltformfüggetlen az alapul szolgáló adattár részletei. REST egy gyűjtemény lehet egy egyedi erőforrást, módszerekkel, a gyűjtemény lekérdezése vagy új entitásokat ad hozzá a gyűjteményhez.

Az API-k tervezésekor gondolja át hogyan, express, hogy a tartományi modellben, nem csak az adatokat a modellben, de is az üzleti műveletek és a megkötések az adatokon belül.

| DDD fogalma | REST-megfelelője | Példa |
|-------------|-----------------|---------|
| Összesítés | Erőforrás | `{ "1":1234, "status":"pending"... }` |
| Identitás | URL-cím | `https://delivery-service/deliveries/1` |
| Gyermekentitások | Hivatkozások | `{ "href": "/deliveries/1/confirmation" }` |
| Érték objektumok | PUT vagy PATCH | `PUT https://delivery-service/deliveries/1/dropoff` |
| Adattár | Gyűjtemény | `https://delivery-service/deliveries?status=pending` |

## <a name="api-versioning"></a>API-verziószámozás

Egy API-ját egy szolgáltatás és -ügyfelek vagy ügyfél között. Ha egy API megváltozik, akkor annak a kockázata, az API-t, a függő ügyfelek használhatatlanná tévő azokat, hogy-e külső ügyfelek vagy egyéb mikroszolgáltatás-alapú. Emiatt tanácsos API módosításai számának csökkentése érdekében. Az alapul szolgáló implementációs változásait gyakran, nem szükséges módosítania kellene az API-t. Reálisan azonban bizonyos ponton célszerű új funkciók vagy módosítása egy meglévő API-t igénylő új képességek hozzáadása.

Lehetséges, ellenőrizze API minden alkalommal, amikor megváltozik a visszamenőlegesen kompatibilis. Például ne eltávolítása egy mezőt a modellből, mert, hogy az ügyfelek, amelyek hatással vannak a mező nincs, azzal. Mező hozzáadása tilos megszüntetnie kompatibilitási, mert az ügyfél figyelmen kívül hagyja a válasz nem ismerik mezőket. A szolgáltatás azonban a helyzet, amelyben egy régebbi ügyfél az áttekinthetőség kedvéért kihagyja az új mezőt a kérelem kell kezelnie.

Verziókezelés támogatja az API-szerződés. Ha egy API-t használhatatlanná tévő változást, vezessen be egy új API-verzió. Továbbra is támogatja a korábbi verziót, és megadásakor az ügyfelek mely meghívásához-verziónak a kiválasztása. Nincsenek ehhez többféleképpen. Egyik egyszerűen elérhetővé mindkét verziót, egyazon szolgáltatáson belül. Egy másik lehetőség, hogy a szolgáltatás egymás melletti két verzióját futtassa, és irányíthatja a kérelmeket az egyik vagy a más verziót, HTTP-útválasztási szabályok alapján.

![Verziókezelés](../images/versioning.png)

Nincs több verzió, fejlesztői idő, a tesztelés és üzemeltetési terheit támogató díjat fizetni. Ezért, fontos a lehető leggyorsabban kivezetjük a régi verziók. A belső API-k az API-t birtokló csoport más csapatokkal, amelyekkel az új verzióra történő migrálása is dolgozhat. Ez akkor, ha a fejlesztőcsapatok közötti cégirányítási folyamat kellene hasznos. A külső (nyilvános) API-k azok nehezebben kivezetjük az API-verziót, különösen akkor, ha az API a natív ügyfélalkalmazások vagy harmadik fél által felhasznált.

Ha a szolgáltatás megvalósítása módosításait, hasznos címkézése a módosítás verzióját. A verzió fontos információkat tartalmaz a hibák elhárítása során. A kiváltó okok elemzését tudni, hogy pontosan a szolgáltatás melyik verziója lett meghívva nagyon hasznos lehet. Fontolja meg [Szemantikus verziószámozást](https://semver.org/) esetében. Szemantikus verziószámozást használ egy *jelentős. KISEBB. JAVÍTÁS* formátumban. Azonban az ügyfelek csak válassza a fő verziószáma, vagy esetleg a alverzió API van-e jelentős (de nem kompatibilitástörő) változások alverziót között. Más szóval, akkor ésszerű-ügyfelek 1-es és 2-es verziójú API-választhat, de nem 2.1.3-verziónak a kiválasztása. Ha engedélyezi a részletesség szintjét, kockázati, nem kell egy elterjedése verziókat támogatja.

Az API-k verziókezelése további leírását lásd: [Versioning webes RESTful API](../../best-practices/api-design.md#versioning-a-restful-web-api).

## <a name="next-steps"></a>További lépések

Ismerje meg, a határ ügyfélalkalmazások és mikroszolgáltatások közötti API-átjáró használatával.

> [!div class="nextstepaction"]
> [API-átjárókat](./gateway.md)
