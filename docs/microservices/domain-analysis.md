---
title: A mikroszolgáltatásokat tartományelemzés
description: A mikroszolgáltatásokat tartományelemzés.
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: cc265aabf401f56a1a81a630e143b2c59986a8e2
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482715"
---
# <a name="designing-microservices-domain-analysis"></a>Mikroszolgáltatások tervezése: Tartományelemzés

A legnagyobb kihívás a mikroszolgáltatások egyik meghatározni az egyes szolgáltatások. Az általános szabály, hogy egy szolgáltatás hajtsa végre a "jó" &mdash; azonban, hogy a szabály üzembe gyakorlat tanulmányozásakor. Nincs egyetlen vehesse a műszaki folyamat, amely a "megfelelő" tervezési állítja elő. Hogy mélyen gondolja át az üzleti tartomány, követelmények és célok. Ellenkező esetben fejezheti be néhány nemkívánatos jellemzői, például a szolgáltatások, a rövid kapcsoló, vagy rosszul tervezett felületek közötti rejtett függőségek kiváló haphazard kialakítás. Ebben a fejezetben azt a mikroszolgáltatások tervezése a tartományvezérelt megközelítés igénybe vehet.

Mikroszolgáltatás-alapú üzleti képességekről, például az adatok elérése vagy az üzenetkezelési nem vízszintes rétegek köré kell építeni. Emellett laza összekapcsolással és magas működési kohézióval kell rendelkezniük. Mikroszolgáltatások *lazán kapcsolódó* Ha módosítható egy szolgáltatást anélkül, hogy más szolgáltatások egyszerre frissíteni kell. A mikroszolgáltatások általában *javul* hogy vannak-e egyetlen, jól meghatározott célra, például a felhasználói fiókok kezelése, vagy nyomon követi a szállítás előzményei. Egy szolgáltatás kell magába foglalja az adatait, és ügyfelek kategorizálják absztrakt. Egy ügyfél például egy drónt ütemezése anélkül, hogy az ütemezési algoritmust, vagy a drone flotta kezelésének módját részleteinek képesnek kell lennie.

Tartományvezérelt tervezési (DDD) olyan keretrendszert kínál, kerülhetnek a legtöbb, a módszer jól megtervezett mikroszolgáltatások készletét. DDD stratégiai és taktikai fázisa van két különböző. A stratégiai nnn a rendszer a nagy méretű struktúra definiálása. Stratégiai DDD segít, hogy az architektúra üzleti funkciók összpontosítanak. Taktikai DDD ismertet néhány tervezési mintát, használhatja a tartományi modell létrehozásához. Ezek a minták entitások, az összesítések és a tartományi szolgáltatások közé tartozik. E taktikai minták segítségével tervezheti meg, amelyek egyaránt lazán kapcsolódó és javul.

![Egy tartományvezérelt tervezési (DDD) folyamat ábrája](./images/ddd-process.png)

Az ebben a fejezetben, a következő végigvezetjük a következő lépéseket a Drone Delivery alkalmazás való alkalmazásával:

1. Indítsa el az üzleti tartomány tudni, hogy az alkalmazás működési követelmények elemzésével. Ebben a lépésben a kimenete a tartományhoz, amely is tartománymodellek formális készletét át lesz alakítva egy informális leírását.

2. Ezt követően adja meg a *környezetek határolt* annak a tartománynak. Egyes kapcsolódó kontextusai tartalmaz olyan tartományi modell, amely egy adott nevű, a nagyobb alkalmazás jelöli.

3. A körülhatárolt kontextus belül taktikai DDD-minták meghatározására, entitás, összesítések és a tartományi szolgáltatások vonatkoznak.

4. Az előző lépésből származó eredmények segítségével azonosíthatja a mikroszolgáltatás-alapú, az alkalmazásban.

Ez a fejezet ismerteti az első három lépést, amely elsősorban DDD. A következő fejezetben azonosítja azt a mikroszolgáltatás-alapú. Azonban fontos megjegyezni, hogy DDD-iteratív, folyamatban lévő folyamat. A szolgáltatás határain kőbe nem rögzített. Módon alakul ki egy alkalmazást, dönthet úgy újratervezéssel a szolgáltatás számos kisebb szolgáltatásra.

> [!NOTE]
> Ez a fejezet nem célja, hogy a teljes és átfogó tartományelemzés megjelenítése. Hogy szándékosan tartani a példa rövid, annak érdekében, hogy a fő pontokat mutatja be. Ismertető háttér-DDD, javasoljuk, hogy Eric Evans *driven tervezési*, a könyvet, amely az előfizetési időszak első bevezetése. A hivatkozás egy másik jó *Implementing Domain-Driven tervezési* Vaughn Vernon szerint.

## <a name="analyze-the-domain"></a>A tartomány elemzése

DDD módszerével mikroszolgáltatások tervezheti meg, hogy minden szolgáltatás forms természetesen illeszkednek a egy működési üzleti igény szerint segít. Megkönnyíti, hogy ne a trap lehetővé teszik a szervezet határain vagy technológiai lehetőségek diktálni a tervező.

Kód írása előtt szüksége van egy Madártávlatból létrehozásakor a rendszer. DDD elindítja az üzleti tartomány modellezés, és hozzon létre egy *tartománymodell*. A tartományi modellben egy absztrakt modell az üzleti tartomány. Azt a mondatokból és adatait rendezi, és egy közös nyelvvel biztosít a fejlesztők és a tartomány szakértői.

Első lépésként minden az üzleti funkciók és a kapcsolatok hozzárendelése. Ez valószínűleg egy együttműködésen alapuló erőfeszítés, amely magában foglalja a tartomány-szakértők, szoftverek tervezők és más érdekelt felek. Bármely adott formalism használatához nincs szükség.  Diagram Vázlat vagy rajztáblaként rajzolni.

Töltse ki a diagramon, mivel előfordulhat, hogy megkezdése diszkrét altartományok azonosításához. Mely függvények szorosan kapcsolódó? Amelyek funkciók alapvető üzleti, és kiegészítő szolgáltatásokat biztosító? Mi az a függőségi grafikon? Ebben a kezdeti fázisban az érintett technológiákat és megvalósítási részletei nem. Ugyanakkor vegye figyelembe a hely, ahol az alkalmazás külső rendszerekkel, mint például a CRM, integrálható a fizetés feldolgozása, illetve rendszerek számlázási kell.

## <a name="drone-delivery-analyzing-the-business-domain"></a>Drone Delivery: Az üzleti tartomány elemzése

Néhány kezdeti tartomány az elemzést a Fabrikam csoport vonalkódképet hozzávetőleges vázlatot, és a Drone Delivery tartományhoz.

![A Drone Delivery tartomány diagramja](./images/ddd1.svg)

- **A szállítási** alapvető üzleti, mert a diagram közepére kerül. Minden más, a diagram létezik, ez a funkció engedélyezéséhez.
- **Felügyeleti drónos** alapvető üzleti is van. Szorosan kapcsolódik a drone felügyeleti funkciókat tartalmaz **drónos javítási** és **prediktív elemzési** előre, amikor szüksége van a drónok a karbantartás és a karbantartás.
- **DRÓN elemzési** biztosít a begyűjtés és kézbesítési becsült idő.
- **Külső szállítás** szállítási alternatív módszerek ütemezése, ha a csomag nem lehet rendszerrel szállított teljesen drónos az alkalmazás lehetővé teszi.
- **Megosztás drónos** egy lehetséges az alapvető üzleti-bővítmény. A vállalat bizonyos időszakokban drónos felesleges kapacitással rendelkezhet, és sikerült bérleti drónok, amelyek egyébként üresjárati ki. Ez a funkció az eredeti kiadásban nem.
- **Videó felügyeleti** van egy másik olyan terület, amely a vállalat előfordulhat, hogy bontsa ki az később.
- **Felhasználói fiókok**, **számlázás**, és **ügyfélszolgálatával** vannak, amelyek támogatják az alapvető üzleti altartományt.

Figyelje meg, hogy ezen a ponton a folyamat még nem végeztünk megvalósítása vagy technológiákkal kapcsolatos döntések. Néhány az alrendszerek magában foglalhatja a szoftver külső rendszerek vagy harmadik féltől származó szolgáltatásokkal. Még ha így az alkalmazásnak kell interakcióba ezen rendszerek és szolgáltatások, ezért fontos, hogy felveszi a tartományi modellben.

> [!NOTE]
> Ha egy alkalmazás egy külső rendszer függ, annak a kockázata, hogy a külső rendszer sémát vagy API-t fog memóriavesztés az alkalmazásba, végső soron veszélyeztetése az architekturális tervezés. Ez különösen igaz, amelyek nem mindig a modern ajánlott eljárásokat, és előfordulhat, hogy használja, pl. szövevényes adatsémák, elavult API-k vagy régebbi rendszerekkel. Ebben az esetben fontos a külső rendszerek és az alkalmazás között a jól definiált határok rendelkezik. Fontolja meg a [Leépítő minta](../patterns/strangler.md) vagy a [Sérülésgátló réteg minta](../patterns/anti-corruption-layer.md) erre a célra.

## <a name="define-bounded-contexts"></a>Adja meg a kapcsolódó kontextusokat

A tartományi modell tartalmazza semmilyen felelősséget a világ valós hálózata &mdash; felhasználók, drónok, csomagokat és így tovább. De ez nem jelenti azt, hogy a rendszer minden része kell használni az azonos reprezentációinak ugyanazokat a műveleteket.

Például alrendszereket, a drone javítása és a prediktív elemzés kezelni kell, amelyek sok fizikai jellemzők drónok, például a karbantartási előzményei, távolság, kor, modellszám, teljesítményt nyújt és stb. De amikor egy kézbesítési ütemezni, azt nem érdeklik ezeket a dolgokat. Az ütemezési alrendszer csak tudnia kell, hogy rendelkezésre áll-e egy drónt, és a felvétel és kézbesítési DRÓN.

Ha létrehoz egy egyetlen modellt mindkét alrendszereket megpróbáltuk, szükségtelenül összetett lenne. Azt is válnak nehezebb a modellhez való idővel, mert több csapat dolgozik a különböző alrendszereket teljesítéséhez szüksége lesz a módosításokat. Ezért érdemes gyakran külön modellek, amelyek ugyanazon valós entitás (ebben az esetben egy drónt) két különböző környezetekben tervezéséhez. Minden modell tartalmazza, csak a szolgáltatások és az attribútumokat, amelyek fontosak az adott környezeten belül.

Ekkor nyernek a DDD fogalmát *környezetek határolt* play kerül. A körülhatárolt kontextus egyszerűen a határt, a tartományban, ahol egy adott modell vonatkozik. Az előző ábrán megnézzük, hogy csoportosíthatja funkció e a különböző függvényeket oszt egy szakterületi modell szerint.

![Kapcsolódó kontextusokat ábrája](./images/ddd2.svg)

Kapcsolódó kontextusokat, amelyek nem feltétlenül különítve egymástól. Ezen az ábrán a folytonos vonal a kapcsolódó kontextusokat csatlakozás helyeken, ahol két kapcsolódó kontextusokat interaktív képviseli. A szállítási címhez tartozó felhasználói fiókoknak az ügyfelek adatainak lekérése, és a flotta a drónok ütemezése Drónos felügyeleti függ.

A könyv *Domain-Driven Design*, Eric Evans számos olyan tartományi modell integritásának fenntartása, amikor interaktálnak az egy másik körülhatárolt kontextus mintákat ismerteti. Mikroszolgáltatások fő alapelveit egyik célja, hogy a szolgáltatások jól meghatározott API-kon keresztül kommunikálnak. Ez a módszer felel meg a két minta, hogy Evans meghívja a gazdagépen nyissa meg a szolgáltatási és közzétett nyelvi. A megnyitott gazdagép szolgáltatás lényege, hogy egy alrendszer határozza meg a hivatalos protokoll (API) kommunikálni más alrendszerek. Közzétett nyelvi kibővíti ezt az elképzelést tegye közzé az API-t egy űrlapon, amellyel más csapatokkal az ügyfelek írása. A fejezetben [API-tervezés](./api-design.md), megbeszélhetünk használatával [OpenAPI-specifikáció](https://www.openapis.org/specification/repo) (korábban Swagger néven) JSON vagy a YAML formátumú kifejezett meghatározásához a REST API-k, nyelvtől független felület leírását.

Tartson a folyamatban a többi fogunk koncentrálni, amelyet szállítási keretében.

## <a name="tactical-ddd"></a>Taktikai DDD

A stratégiai DDD fázisban ki az üzleti tartomány végzett, és definiálása, amelyet a tartománymodellek környezetét. Taktikai DDD van, a tartomány modellek nagyobb pontossággal meghatározása során. A taktikai minták körülhatárolt kontextus egy egyetlen belül érvényesek. A mikroszolgáltatási architektúrákban tudjuk különösen az entitás- és összesített minták iránt. Ezek a minták alkalmazásának segítségünkre lesz a természetes határokat a szolgáltatások, az alkalmazás azonosításához (lásd: [következő fejezet](./microservice-boundaries.md)). Általános szabály a mikroszolgáltatások nem kisebb, mint az összesítést, és nem haladhatja meg a körülhatárolt kontextus kell lennie. Első lépésként vizsgáljuk meg a taktikai minták. Ezután azt fogja azokat alkalmazni a korlátozott szállítási környezetben a Drone Delivery alkalmazást.

### <a name="overview-of-the-tactical-patterns"></a>A taktikai minták áttekintése

Ez a témakör röviden ismerteti a taktikai DDD mintáit, így ha már ismeri a DDD, valószínűleg ezt a szakaszt kihagyhatja. A minták fejezet 5 részletesen ismertetett &ndash; 6 Eric Evans címjegyzék-alkalmazásával, valamint a *Implementing Domain-Driven tervezési* Vaughn Vernon szerint.

![Taktikai tartományvezérelt tervezési mintákat bemutató ábra](./images/ddd-patterns.png)

**Entitások**. Egy entitás egy olyan objektum, amely továbbra is fennáll, idővel egyedi azonosítóval rendelkező. Ha például egy banki alkalmazás, ügyfelek és partnerek lenne entitások.

- Egy entitás egy egyedi azonosítót a rendszer, amely segítségével kereshet, vagy kérje le az entitást tartalmaz. Ez nem jelenti azt közvetlenül a felhasználóknak mindig megkapja a azonosítóját. Ez lehet egy GUID Azonosítót vagy egy elsődleges kulcs egy adatbázisban.
- Az identitás több kapcsolódó kontextusokat magában foglalhat, és előfordulhat, hogy rutinszerű túl az alkalmazás teljes élettartama. Például bankszámla számokat vagy a kormányzat által kibocsátott azonosítók vannak kötve egy adott alkalmazás teljes élettartama.
- Egy entitás attribútumai idővel változhat. Például egy személy nevét vagy címét változhat, de azok továbbra is ugyanazt az embert.
- Egy entitás tartalmazhat más entitásokra mutató hivatkozásokat.

**Objektumok érték**. Egy érték objektumnak nincs identitás. Csak a hozzá tartozó attribútumok értékeinek van definiálva. Érték objektumokat is nem módosíthatók. Érték objektum frissítése, mindig hozzon létre egy új példányt a régi helyett. Érték objektumoknak is lehetnek, amely magába foglalja az üzleti logikát módszereket, de azokat a módszereket az objektum állapota nem mellékhatásai legyen. Tipikus érték objektumok például színek, dátumok és időpontok és pénznemek.

**Összesítések**. Összesítés konzisztencia határ egy vagy több entitás körül határozza meg. Az összesítés pontosan egy entitás, a legfelső szintű. Keresési történik a gyökérszintű entitás-azonosítója használatával. Minden más entitás aggregált a legfelső szintű gyermek, és a legfelső szintű következő mutatók hivatkozik.

Az összesítés célja, hogy tranzakciós invariants modell. A való világból dolgot kell összetett webhelyek közötti kapcsolatok. Ügyfelek rendelések létrehozása rendelések tartalmazzák a termékek, termékek szállítók rendelkezik, és így tovább. Ha az alkalmazás módosítja a több kapcsolódó objektumot, hogy nem garantálja a konzisztenciát? Hogyan tudjuk invariants nyomon követheti, és kényszeríteni azokat?  

A hagyományos alkalmazások gyakran használt adatbázis-tranzakciók konzisztencia érvényesítéséhez. Az elosztott alkalmazásokban azonban ez gyakran nem megvalósítható. Egyetlen üzleti tranzakciók több adattárral, magában foglalhat, vagy előfordulhat, hogy hosszú ideig futó, vagy külső szolgáltatásokat is igénybe vehet. Végső soron van az alkalmazáshoz, az adatok réteg nem a szükséges tartomány invariants kényszerítésére. Ez milyen összesítések úgy van kialakítva, hogy a modell.

> [!NOTE]
> Összesítés tartalmazhatnak egyetlen entitást, gyermekentitások nélkül. Ez teszi összesítés a tranzakciós határt.

**Tartomány- és alkalmazásszolgáltatások**. A DDD-terminológiában a szolgáltatás olyan objektum, amely egy logikai bármely állapotban tartása nélkül. Evans különböztet *tartományi szolgáltatások*, amely magába foglalja az üzleti logikát, és *alkalmazásszolgáltatások*, biztosító műszaki funkciókat, például felhasználói hitelesítés és a egy SMS küldése az üzenet. Tartományi szolgáltatások gyakran használják az eszközmodell viselkedésének kiterjedő több entitás.

> [!NOTE]
> Az előfizetési időszak *szolgáltatás* szoftverfejlesztési túl van terhelve. Itt a definíció nem kapcsolódik közvetlenül mikroszolgáltatási.

**Tartományi események**. Tartományi események segítségével a rendszer egyéb részeivel értesítése, ha olyan esemény történik. Ahogy a neve is sugallja, tartományi események valamit a tartományon belül kell jelenti. Például "egy rekordot egy táblában illesztett" nem egy tartományi esemény. "Egy kézbesítési meg lett szakítva" az tartomány esemény. Tartományi események olyan gyakran a mikroszolgáltatási architektúrákban. Mikroszolgáltatás-alapú vannak osztva, és ne osszon adattárak, mert a tartomány események koordinálnak egymással a mikroszolgáltatások lehetőséget biztosíthat. A fejezet [szolgáltatások közötti kommunikáció](./interservice-communication.md) aszinkron üzenetkezelést részletesebben ismerteti.

Nincsenek néhány egyéb DDD minták nem szerepel itt, beleértve az előállítók, az adattárak és a modulok. Ezek lehet hasznos, amikor azt fontolgatja, mikroszolgáltatások mintákat, de kevésbé fontos a határok között mikroszolgáltatások tervezése során.

## <a name="drone-delivery-applying-the-patterns"></a>Drone delivery: A minták alkalmazásának

Kezdődik meg, amely a szállítási határolt környezetet kell kezelni a forgatókönyvekkel.

- Egy ügyfél egy termék drónos áru, egy drónos szállítási szolgáltatásra regisztrált üzleti kérhetnek.
- A küldő egy címkét (vonalkód vagy RFID) helyezi a csomagot hoz létre.
- Egy drónt folytattuk a munkát, és a egy csomag forráshelyéről továbbítására a célhelyen.
- Amikor az ügyfél ütemezi a kézbesítési, akkor a rendszer egy útvonal-információkat, az időjárási viszonyok és az előzményadatok alapján DRÓN biztosít.
- Ha a drone útban, a felhasználó nyomon követheti az aktuális hely és a legújabb DRÓN.
- Mindaddig, amíg egy drónt rendelkezik mértékének a csomag, az ügyfél egy kézbesítési vonhatja vissza.
- Az ügyfél értesítést kap a kézbesítési befejezésekor.
- A küldő kézbesítési megerősítő használói kérhetnek az ügyfél egy aláírás vagy az ujját nyomtatási formájában.
- Felhasználók kereshet egy befejezett szállítás előzményei.

Az ezekben az esetekben a fejlesztői csapat az alábbi azonosított **entitások**.

- Kézbesítés
- Csomag
- Drónos
- Fiók
- Megerősítés
- Értesítés
- Címke

Az első négy, Tartalomkézbesítési, csomag, Drónos fiók jsou összes **összesítések** , amelyek az tranzakció-konzisztencia határokat. Megerősítések és értesítések gyermekentitások a szállítások, és a címkék olyan gyermekentitások csomagok.

A **érték-objektumok** Ez a kialakítás tartalmazzák a helyet, DRÓN, PackageWeight és PackageSize.

Mutatja be, ez a kézbesítési összesítés UML diagramját. Figyelje meg, hogy rendelkezik-e más összesítéseket, beleértve a fiókot, a csomag és a Drone mutató hivatkozásokat.

![A kézbesítési összesítés UML ábrája](./images/delivery-entity.png)

Nincsenek a két tartomány esemény:

- Bár egy drónt útban, a Drone entitás küld DroneStatus események, amelyek ismertetik a drón helyét és állapotát (átvitel közben, szállítási).

- A kézbesítési entitás DeliveryTracking eseményeket, amikor módosul egy szállítási küld. Ezek közé tartozik a DeliveryCreated, DeliveryRescheduled, DeliveryHeadedToDropoff és DeliveryCompleted.

Figyelje meg, hogy ezek az események leírása dolog, amelyek fontosak lehetnek a tartomány modellen belül. Adja meg valamit a tartományhoz, és nem kapcsolódik egy adott programozási nyelv szerkezet.

A fejlesztői csapat egy további terület, amely nem fér el eligazíthatja bármelyikével az eddig leírt azonosítani. A rendszer egyes részei kell koordinálja az összes ütemezés vagy egy tartalomkézbesítési frissítése lépéseit. Ezért a fejlesztői csapat hozzáadott két **tartományi szolgáltatások** terve: egy *Scheduler* , hogy koordinálja a lépéseket, és a egy *felügyelő* , amely figyeli az egyes állapota lépés annak észleléséhez, hogy olyan lépéseket a sikertelen vagy túllépte az időkorlátot. Ez a kapcsolat egy változata a [Feladatütemező ügynök felügyeleti mintájának](../patterns/scheduler-agent-supervisor.md).

![A módosított tartományi modellben ábrája](./images/drone-ddd.png)

> [!div class="nextstepaction"]
> [Mikroszolgáltatások határainak azonosítása](./microservice-boundaries.md)
