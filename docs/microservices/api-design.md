---
title: API-tervezés
description: API-k a mikroszolgáltatások tervezése
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: d85407f3092ddb5f77aacfea8def2784c4741eb9
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2017
---
# <a name="designing-microservices-api-design"></a>Mikroszolgáltatások tervezése: API-Tervező

Tervezési API fontos mikroszolgáltatások architektúra esetén, mert az összes adatcsere-szolgáltatások közötti üzenetek, illetve az API-hívásokban keresztül történik. Lehet, hogy API-k hatékony, hogy ne hozzon létre [chatty i/o](../antipatterns/chatty-io/index.md). Szolgáltatásait Önre terveztük csoportok egymástól függetlenül működik, mert API-k rendelkeznie jól meghatározott szemantikáját és versioning rendszerek, úgy, hogy a frissítések ne szakítsa egyéb szolgáltatásokat.

![](./images/api-design.png)

Fontos, hogy API kétféle megkülönböztetni:

- Nyilvános API-ügyfélalkalmazások hívható meg. 
- Háttér értekezleteire kommunikációhoz használt API-k.

E két használati esetek némileg különböző követelményekkel rendelkezik. A nyilvános API ügyfélalkalmazások, általában böngészőalkalmazásokban vagy natív mobilalkalmazások kompatibilisnek kell lennie. Az esetek többségében, amely azt jelenti, hogy a nyilvános API-t fogja használni a többi HTTP Protokollon keresztül. A háttérkiszolgáló API-kat azonban meg kell tennie a hálózati teljesítmény figyelembe. Attól függően, hogy a szolgáltatások granularitásán értekezleteire kommunikációs nagy mennyiségű hálózati forgalmat eredményezhet. Szolgáltatások vehet i/o kötött. Ezért például szerializálási sebesség és a tartalom szempontok fontosabb válik. Néhány népszerű más REST HTTP Protokollon keresztül gRPC, Apache Avro és Apache Thrift tartalmazza. Ezeket a protokollokat támogatja a bináris szerializálást, és amelyek általában hatékonyabb, mint a HTTP.

## <a name="considerations"></a>Megfontolandó szempontok

Az alábbiakban néhány dolgot gondolja át az API-k megvalósítása kiválasztásakor.

**REST vs RPC**. Vegye figyelembe a mellékhatásokkal RPC stílusú illesztőfelület és egy REST-stílusú felület használata között.

- REST-modellek erőforrásokat, amelyek természetes mód a modell express. Egy egységes felületet alapján a HTTP-műveleteket, amelyek könnyen létrejöhetnek evolvability határoz meg. Jól meghatározott szemantikáját idempotencia, hatásai és válaszkódot rendelkezik. És érvényesíti a állapot nélküli kommunikációhoz, fejleszti a méretezhetőséget. 

- RPC több célú műveletek vagy parancsok körül. RPC felületek néz helyi metódushívások, mert túlságosan chatty API-k alakíthat okozhatja. Azonban, hogy nem jelenti azt RPC chatty kell lennie. Csak akkor kell használni a figyelmet a felület tervezésekor.

Egy RESTful felület esetében a leggyakrabban használt rendszer REST JSON használatával HTTP Protokollon keresztül. Egy RPC-style felület számos népszerű keretrendszerekre, beleértve a gRPC, Apache Avro és Apache Thrift vannak.

**Hatékonyságát**. Vegye figyelembe a hatékonyság sebességét, a memória és a tartalom mérete tekintetében. A gRPC-alapú felület általában gyorsabb, mint a többi HTTP Protokollon keresztül.

**Illesztő definíciójának nyelven (IDL)**. Egy IDL határozza meg a metódusok paramétereit, és visszatérési értékek egy API segítségével. Egy IDL ügyfélkódot, a szerializálási kód és az API-JÁNAK dokumentációja létrehozásához használható. Tesztelési eszközöket, például a Postman API is képes használni IDLs. Keretrendszerek, például a gRPC Avro és Thrift saját IDL specifikációk meghatározása. HTTP Protokollon keresztül REST nincs szabványos IDL formátumban, de egy közös választott OpenAPI (korábbi nevén Swagger). Egy HTTP REST API műveletek formális definíciója nyelv használata nélkül is létrehozhat, de a kód létrehozását és tesztelését előnyeit elvesznek majd.

**Szerializálási**. Hogyan objektumok szerializálva vannak a hálózaton? A választható lehetőségek szöveges formátumban (elsősorban JSON) vagy bináris formátumban, például a protokoll puffer. Bináris formátuma nem általában gyorsabb, mint a szöveges formátumú. Azonban JSON előnye is van tekintetében együttműködési képesség, mivel a legtöbb nyelv és keretrendszer támogatja a JSON-szerializálást. Némelyik szerializálási formátum van szükség egy rögzített sémájába, és néhány fájl fordítása. Ebben az esetben szüksége lesz a felépítési folyamat funkcióját beépítse ezt a lépést. 

**Keretrendszer és a nyelvi támogatás**. A HTTP szinte minden keretrendszer és a nyelv támogatott. gRPC, Avro és Thrift összes van szalagtárak C++, C#, Java és Python. Thrift és gRPC is támogatja a nyissa meg. 

**Kompatibilitás és együttműködés**. Ha például a gRPC protokoll, szükség lehet a fordítási réteg protokoll a nyilvános API és a háttérrendszer között. A [átjáró](./gateway.md) is, hogy a művelet végrehajtására szolgál(nak). Ha szolgáltatás rácsvonal használ, fontolja meg, mely protokollok kompatibilisek-e a szolgáltatás háló. Például linkerd rendelkezik HTTP Thrift és gRPC beépített támogatása. 

Az alapkonfiguráció javasoljuk, hogy REST módot HTTP Protokollon keresztül, ha a bináris protokoll által van szüksége. HTTP Protokollon keresztül REST különleges függvénytárak igényel. Létrehoz minimális kapcsolási, mert a hívó nem kell egy ügyfél helyettes a szolgáltatással való kommunikációra. Nincs gazdag ökoszisztéma eszközök sémadefiníciók, tesztelése és figyelése a RESTful HTTP-végpontok támogatásához. Végezetül HTTP összeegyeztethető webböngésző-ügyfelek számára, így nem kell az ügyfél és a háttérkiszolgáló közötti protokoll fordítási rétegben. 

Azonban ha REST HTTP Protokollon keresztül, inkább hajtsa végre a teljesítmény és terheléses tesztelés korai szakaszában a fejlesztési folyamat, ellenőrizze, hogy elég esetén a forgatókönyv segítségével végzi.

## <a name="restful-api-design"></a>RESTful API-Tervező

Nincsenek számos olyan forrás RESTful API-k tervezéséhez. Íme néhány hasznos előfordulhat:

- [API-Tervező](../best-practices/api-design.md) 

- [API-implementáció](../best-practices/api-implementation.md) 

- [Microsoft REST API-irányelvek](https://github.com/Microsoft/api-guidelines)

Az alábbiakban néhány figyelembe kell venni bizonyos szem előtt tartani.

- Figyeljen az API-t szivárgás lépett fel, belső megvalósítás részletei vagy egy belső adatbázisséma egyszerűen tükrözik. Az API-t a tartomány kell modell. Szolgáltatások között, és amikor új funkciókat ad hozzá, nem csak, mert néhány kódot átkerült vagy egy adatbázis tábláinak normalizált ideális csak módosítsa. 

- Ügyfél, például a mobilalkalmazás- és asztali böngészőben, különböző típusú különböző hasznos méretek vagy interakció minták lehet szükség. Érdemes lehet a [Frontends minta háttérkiszolgálókon](../patterns/backends-for-frontends.md) külön háttérkiszolgálókon létrehozása az egyes ügyfelekhez, optimális illesztőfelület visszaállítását, hogy az ügyfélszámítógépek számára.

- A mellékhatással működő műveleteket fontolja meg az idempotent minősítené, és azok PUT módszerként megvalósítását. Amely lehetővé teszi a biztonságos újrapróbálkozások, és javíthatja a rugalmasságot. A fejezetek [adatfeldolgozást és a munkafolyamat](./ingestion-workflow.md#idempotent-vs-non-idempotent-operations) és [kommunikációs Interservice](./interservice-communication.md) probléma részletesebben ismertetik.

- HTTP-metódus lehet aszinkron szemantikáját, amikor a metódus azonnal visszaadja a választ, de a szolgáltatás aszinkron módon végrehajtja a műveletet. Ebben az esetben a metódus visszaadja-e egy [HTTP 202](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) válaszkód, ami azt jelenti, hogy a kérés elfogadva feldolgozásra, de a feldolgozás nem fejeződött be.

## <a name="mapping-rest-to-ddd-patterns"></a>REST leképezése DDD minták

Például érték objektum, entitás vagy Összesítés minták úgy tervezték, hogy az objektumok bizonyos megkötéseknek helyez a modell. Sok beszélgetéseket, DDD a minta van modellezve, objektumorientált (OO) nyelvi fogalmak például konstruktorok vagy tulajdonság beolvasókat és Setter elemek használatával. Például *objektumok érték* nem módosítható elvileg. Egy OO programozási nyelven szeretné kényszeríti Ez a konstruktor értékeket rendel, és a tulajdonságok csak olvasható:

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

Ezek rendezi a kódolási eljárások különösen fontosak, egy hagyományos egységes alkalmazás felépítése közben. A nagyméretű kódbázis, sok alrendszereket használhatja a `Location` objektum, ezért fontos az objektum helyes működése érvényesítését. 

Egy másik példa a tárház mintát, amely biztosítja, hogy az alkalmazás egyéb elemeinek közvetlen olvasási vagy írási műveleteket ad ki az adatok tárolására ne:

![](./images/repository.png)

Mikroszolgáltatások architektúra esetén azonban szolgáltatások ne ossza meg az azonos kódbázisra és ne ossza meg adattárolókhoz. Ehelyett kommunikáció API-k segítségével. Vegye figyelembe az esetet, ahol a Feladatütemező szolgáltatás kéri le a dron szolgáltatás egy dron kapcsolatos információkat. A dron szolgáltatás a belső egy dron kód kifejezett-modellhez tartozik. De az ütemező, hogy nem látható. Ehelyett azt vissza lekérdezi egy *ábrázolását* dron entitás &mdash; lehet, hogy egy JSON-objektum a HTTP-válasz.

![](./images/ddd-rest.png)

A Feladatütemező szolgáltatás nem módosítható a dron szolgáltatás belső modelljei vagy a dron szolgáltatás adattár írni. Amely azt jelenti, hogy a kód a dron szolgáltatást kisebb kitett terület, a hagyományos monolit kód képest. Ha a dron szolgáltatás helye definiálja, amely hatóköre korlátozódik &mdash; nincs más szolgáltatás közvetlenül fog használni az osztály. 

Ezen okok miatt ez az útmutató nem összpontosítson, nagy a kódolási eljárások, mivel taktikai DDD mintázatokat kapcsolódnak. De változik, hogy is modellezhető DDD mintázatokat REST API-k segítségével számos. 

Példa:

- Összesíti a természetes leképezése *erőforrások* többi. Például a kézbesítési összesítés volna elérhetővé tehető erőforrásként a kézbesítési API.

- Aggregátumok konzisztencia határok. Aggregátumok műveletek soha nem hagyja összesítő inkonzisztens állapotú. Ezért ne API-k, amelyek lehetővé teszik egy ügyfél segítségével kezelheti a belső állapotot, az összesítés létrehozásakor. Ehelyett alkalmazást coarse-grained API aggregátumok erőforrásként visszaállítását.

- Az entitásnak egyedi azonosítói. A többi erőforrások URL-címek formájában egyedi azonosítói rendelkeznek. Hozzon létre erőforrás URL-címek, amelyek megfelelnek egy entitás tartomány identitása. Lehet, hogy a leképezés URL-cím a tartomány identitása nem átlátszó ügyfélnek.

- Gyermek entitások az összesítés a legfelső szintű entitásból útvonalon elérhető. Ha követi [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) alapelvek, gyermek entitások elérhető a szülőentitás ábrázolását a kapcsolaton keresztül. 

- Érték objektumok nem módosíthatók, mert frissítések megtörténik a teljes érték objektum cseréli. Többi PUT vagy javítás kéréseken keresztüli frissítések végrehajtása. 

- A tárház lehetővé teszi, hogy az ügyfelek lekérdezés, hozzáadhat és eltávolíthat objektumok egy gyűjtemény, absztrakt módon megjelenítve az alapul szolgáló adattár részleteit. A többi egy gyűjtemény lehet különböző erőforrás, a gyűjtemény lekérdezése, illetve új entitásokat a gyűjteményhez való hozzáadása a.

Az API-k tervezésekor gondolniuk hogyan azok express a modell, nem csupán azokat az adatokat a modellt, de is az üzleti tevékenységre és a megkötések az adatokon belül.

| DDD koncepció | REST-megfelelője | Példa | 
|-------------|-----------------|---------|
| Összesítés | Erőforrás | `{ "1":1234, "status":"pending"... }` | 
| Identitás | URL | `http://delivery-service/deliveries/1` |
| Gyermek entitások | Hivatkozások | `{ "href": "/deliveries/1/confirmation" }` |
| Frissítési érték objektumok | PUT vagy rendszerhez – javítás | `PUT http://delivery-service/deliveries/1/dropoff` |
| Tárház | Gyűjtemény | `http://delivery-service/deliveries?status=pending` |


## <a name="api-versioning"></a>API-versioning

Az API-k egy szolgáltatás és az ügyfelek, illetve az adott szolgáltatás fogyasztóknak között. Ha megváltoztatja az API-k, fennáll a kockázata, az API-t, a függő ügyfelek megtörje van hogy külső ügyfelek vagy egyéb mikroszolgáltatások-e azokat. Emiatt tanácsos API végrehajtott módosítások számának minimalizálása érdekében. Az alapul szolgáló implementáció változásai gyakran, nem igényel módosításokat az az API-t. Reálisan nézve azonban egy bizonyos ponton érdemes adja hozzá az új funkciók és egy meglévő API módosítását igénylő új képességek.

Lehetséges, ellenőrizze API, amikor megváltozik a visszamenőlegesen kompatibilis. Például ne eltávolítása egy mezőt a modellből az ügyfelek, a mező nem várt, amely érvénytelenné mert. Amikor felvettek egy mezőt nem törik kompatibilitási, mert az ügyfelek a válasz nem ismerniük mezőket figyelmen kívül hagyja. A szolgáltatás azonban kezelni kell az esetben, ha egy régebbi ügyfél kihagyja az új mező a kérelemben. 

Az API egyezmény versioning támogatja. Ha API-t használhatatlanná tévő változást, akkor egy új API-verzió. Továbbra is támogatja a korábbi verziót, és engedélyezheti, hogy válassza ki, melyik verzióját hívására ügyfelek. Többféle módszer. Egyik egyszerűen teszi közzé a mindkét verziója ugyanazt a szolgáltatást. Egy másik lehetőség, hogy a szolgáltatás-mellé két verzióit, és átirányíthatja kérelmek egyik vagy a más verziót, HTTP-útválasztási szabályok alapján. 

![](./images/versioning1.svg)

Nincs több verziót, fejlesztői idejét, a tesztelés és a működési munkaterhelés támogató költséget. Ezért célszerű minél gyorsabban érvényteleníthető régi változatainak helyes. Belső API-k esetében az API-t birtokló csoport használható együtt más csapatokkal az segítséget nyújthatnak az új verziójára való áttérést. Ez akkor, ha egy csoportközi irányítás folyamatot, akkor lehet hasznos. Külső (nyilvános) API-k esetében azok nehezebben érvényteleníthető az API-verziót, különösen akkor, ha az API-t natív ügyfélalkalmazások vagy harmadik felek által felhasznált. 

A szolgáltatás megvalósítási változásokat esetén hasznos, ha szeretné, hogy a módosítás verziójával címkét. A verzió nyújtó fontos információkról, amikor kapcsolatos hibák elhárítása. A kiváltó okának elemzése tudni, hogy pontosan melyik verzióját a service hívták nagyon hasznos lehet. Érdemes lehet [szemantikai versioning](https://semver.org/) szolgáltatás verzióihoz. Szemantikai versioning használ egy *jelentős. KISEBB. JAVÍTÁS* formátumban. Azonban az ügyfelek csak válassza az API-k által a fő verziószáma, vagy esetleg a alverziója esetén jelentős (de nem törhető) módosítások alverziót között. Ez azt jelenti akkor ésszerű ügyfelek válassza az 1-es és 2-es verziójának az API-k között, de nem 2.1.3 verzió kiválasztása. Ha engedélyezi a ekkora szintű részletességű, kockázatok, nem kell egy elterjedése verziók támogatják. 

Az API-versioning további leírását lásd: [Versioning egy RESTful webes API](../best-practices/api-design.md#versioning-a-restful-web-api).

> [!div class="nextstepaction"]
> [Adatfeldolgozást és a munkafolyamat](./ingestion-workflow.md)