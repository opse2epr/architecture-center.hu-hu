---
title: Tartomány-elemzés mikroszolgáltatások
description: Tartomány-elemzés mikroszolgáltatások
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: c3c353a6b30507369357af4b520a51f8afc2fb8d
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/11/2018
ms.locfileid: "27765986"
---
# <a name="designing-microservices-domain-analysis"></a>Mikroszolgáltatások tervezése: tartomány elemzés 

A legnagyobb kihívást az mikroszolgáltatások egyik határozza meg az egyes szolgáltatások határait. Az általános szabály az, hogy egy szolgáltatás hajthatja végre a "jó" &mdash; azonban gondosan meg kell hogy, hogy a szabály üzembe eljárás szükséges. Nincs egyetlen gépi folyamat, amely a "jobb oldali" Tervező eredményez. A mélyen gondolja át a vállalati tartományhoz, követelmények és célok kell. Ellenkező esetben fejezheti haphazard kialakítás néhány nemkívánatos jellemzőit, például a szolgáltatások, a rövid kapcsoló vagy rosszul tervezett felületek közötti rejtett függőségek mutat. Ebben a fejezetben azt mikroszolgáltatások létrehozására az a tartomány adatvezérelt megközelítés igénybe vehet. 

Mikroszolgáltatások úgy kell megtervezni, üzleti képességeit, például az adatok elérése vagy az üzenetkezelési nem vízszintes rétegek körül. Ezenkívül laza kapcsoló és a magas működési kohézió kell rendelkezniük. Mikroszolgáltatások vannak *lazán összekapcsolt* Ha egy szolgáltatás anélkül, hogy más szolgáltatások egyszerre frissítendő bármikor módosíthatja. Egy mikroszolgáltatási van *javul* rendelkezik-e a egyetlen, jól meghatározott célú, például a felhasználói fiókok kezelése vagy küldési előzményeinek nyomon követését. A szolgáltatás kell foglalják magukban a tartomány Tudásbázis és absztrakt ezt az információt az ügyfelektől. Például egy ügyfél kell tudni egy dron ütemezése az ütemezési algoritmus vagy módjára vonatkozik. a dron járműflotta részleteinek ismerete nélkül.

Tartomány-központú kialakítás (nnn), amely szerezheti meg legtöbb keretet biztosít annak, ahogyan egy csoportjának tetszetős mikroszolgáltatások létrehozására. DDD stratégiai és taktikai fázisa van két különböző. A stratégiai nnn a rendszer a felügyeleti teendők központjaként struktúra meghatározásakor. Stratégiai DDD biztosíthatja, hogy az architektúrák továbbra is üzleti képességeit összpontosítanak. Taktikai DDD-kialakítási minta, használhatja a modell létrehozásához biztosít. Ezek a minták entitásokat, az összesítések és a tartományi szolgáltatások közé tartozik. Ezen taktikai minták segítséget mikroszolgáltatások létrehozására, amelyek lazán összekapcsolt és is javul tervezéséhez.

![](./images/ddd-process.png)

Ez a fejezet, a következő, a végigvesszük az alábbi lépéseket, azok alkalmazása a dron továbbítási alkalmazást hozhat létre: 

1. Indítsa el a vállalati tartomány, az alkalmazás működési követelmények megértése érdekében elemzése. Ez a lépés eredménye az is lehet is tartományi modellek formális készletére tartomány nem hivatalos leírását. 

2. Ezt követően adja meg a *környezetek határolt* annak a tartománynak. Minden egyes kötött környezetben a nagyobb alkalmazás adott altartomány jelölő tartománymodell tartalmazza. 

3. A kötött környezeten belül taktikai DDD minták adható meg entitások, az összesítések és a tartományi szolgáltatások alkalmazni. 
 
4. Az előző lépésből származó eredmények segítségével azonosíthatja a mikroszolgáltatások az alkalmazásban.

Ebben a fejezetben azt az első három lépést, amely elsősorban DDD fedik le. A következő fejezetben azt azonosítja a mikroszolgáltatások létrehozására. Fontos azonban megjegyezni, hogy DDD egy iteratív, folyamatban lévő folyamat. Szolgáltatás határok nem rögzített követ. Egy alkalmazás fejlődésének dönthet felosztása egymástól szolgáltatás több kisebb szolgáltatások.

> [!NOTE]
> Ez a fejezet nem célja, hogy a teljes, átfogó tartomány elemzésre megjelenítése. A Microsoft szándékosan tartani a példa rövid, ahhoz, hogy bemutassa a főbb pontjai. További háttere DDD, azt javasoljuk, Eric Evans *Domain-Driven tervezési*, a címjegyzék-alkalmazásával, amely először a kifejezés. Egy másik a helyes hivatkozás *Implementing Domain-Driven tervezési* Vaughn Vernon által. 

## <a name="analyze-the-domain"></a>A tartomány elemzése

Egy DDD megközelítéssel segítséget mikroszolgáltatások kialakítani, hogy minden szolgáltatás űrlapok működési üzleti igény a természetes megfelelően. Segítségével elkerülhetők a trap lehetővé teszik a szervezet határain kívülre vagy technológia döntések előírják a tervező.

Előtt kódot írnia kell egy Madártávlatból létrehozásakor a rendszer. DDD indítja el a vállalati tartomány modellezési és létrehozása egy *tartománymodell*. A modell egy absztrakt modell az üzleti tartomány. Az kigyűjti tartomány Tudásbázis rendezi és a közös nyelvi biztosít a fejlesztők és a tartomány szakértői. 

Első lépésként leképezési összes üzleti funkciók és a kapcsolatokat. Ez valószínűleg egy együttműködési erőfeszítés, amely magában foglalja a tartomány szakértők szoftver mérnökök és egyéb érintett felek. Nem kell minden egyes formalism használja.  A diagram Vázlat vagy Faliújságra megrajzolásához.

Írja be a diagram, előfordulhat, hogy kezdené diszkrét altartományok azonosításához. Mely függvények szorosan kapcsolódó? Funkciók core az üzleti tevékenységre vonatkozóan, és kiegészítő szolgáltatásokat nyújt? Mi az a függőségi grafikon? A kezdeti fázisában nem érintett technológiák vagy megvalósítás részletei. Említett, vegye figyelembe a hely, ahol az alkalmazás lesz integrálni kell külső rendszerekkel, például a CRM, feldolgozás, vagy számlázási rendszerek fizetési. 

## <a name="drone-delivery-analyzing-the-business-domain"></a>Dron kézbesítési: Elemzése a vállalati tartományhoz.

Néhány kezdeti tartomány az elemzést a Fabrikam team le egy durva ábrát, amely a dron kézbesítési tartomány ábrázol.

![](./images/ddd1.svg) 

- **Szállítási** helyezkedik el a diagramon közepére, mert az üzleti core. Minden más, az ábrán létezik, és ennek a funkciónak.
- **Felügyeleti dron** egyben core az üzleti tevékenységre vonatkozóan. Szorosan kapcsolódó dron kezelési funkciókat tartalmaz **dron javítási** és **prediktív elemzési** előre jelezni, amikor dronok kell karbantartása. 
- **Európai elemzés** biztosít a felvételi és kézbesítési becsült idő. 
- **Külső szállítására** szállítására alternatív módszerek ütemezése, ha a csomag nem szállítják teljesen dron által az alkalmazás lehetővé teszi.
- **Megosztás dron** a fő üzleti lehetséges bővítménye. A vállalat esetleg felesleges dron kapacitás egyes órában, és sikerült bérleti dronok, amelyek egyébként üresjárati ki. Ez a funkció nem lesz az eredeti kiadását.
- **Figyelőrendszer** van egy másik olyan területen, amely a vállalat később lehet, hogy kibővítése.
- **Felhasználói fiókok**, **számlázás**, és **ügyfélszolgálatával** , amely támogatja a fő üzleti altartományok vannak.
 
Figyelje meg, hogy ezen a ponton a folyamat során is még nem jutott megvalósítási és technológiák kapcsolatos döntések meghozatala. Néhány az alrendszerek magában foglalhatja külső szoftver rendszerek vagy harmadik féltől származó szolgáltatással. Még ha így az alkalmazást kell együttműködhet ezen rendszerek és a szolgáltatások, ezért fontos, a tartomány modell felvenni. 

> [!NOTE]
> Ha egy alkalmazás olyan külső rendszertől függ, nincs a veszélye, hogy a külső rendszer Adatséma vagy API fog szivárgás lépett fel az alkalmazásba a architekturális tervezési végső soron veszélyeztetése. Ez különösen igaz a régebbi rendszerekre, amelyek nem mindig a modern ajánlott eljárásokat, és előfordulhat, hogy csavart adatok sémák vagy elavult API-k. Ebben az esetben fontos rendelkezik a külső rendszerekből és az alkalmazás jól meghatározott határt. Érdemes lehet a [Strangler mintát](../patterns/strangler.md) vagy a [elleni sérülés réteg mintát](../patterns/anti-corruption-layer.md) erre a célra.

## <a name="define-bounded-contexts"></a>A kötött környezetek meghatározása

A modell tartalmazza a világ valós dolgot ábrázolásai &mdash; felhasználók, dronok, csomagokat és így tovább. Azonban, hogy nem jelenti azt, hogy a rendszer minden részét kell használni a azonos felelősséget ugyanazokat a műveleteket. 

Például dron javítása és a prediktív elemzési alrendszerek kell meghatároznia sok fizikai jellemzők dronok, például a karbantartási előzmények, távolság, kor, teljesítményt nyújt, és a stb. De amikor egy kézbesítési ütemezését, azt nem érdeklik ezeket a beállításokat. Az ütemezési alrendszer csak tudnia kell, hogy rendelkezésre áll-e egy dron, és az Európai a felvételi és kézbesítési. 

Nem sikerült mindkét alrendszereket egyetlen modell létrehozása, ha feleslegesen összetett lenne. Azt is válna nehezebb a modell a verzióinformációk, mert módosításokat kell több csapat dolgozik a különböző alrendszereket kielégítéséhez. Ezért érdemes gyakran külön modellek, amelyek megfelelnek az azonos valós entitás (ebben az esetben egy dron) kétszer tervezéséhez. Egyes modellek csak funkcióit és attribútumokat, amelyek az adott környezetben megfelelő tartalmazza.

Ez akkor, ha a DDD fogalmát *környezetek kötve* játékba származik. A kötött környezetben egyszerűen a határt, a tartományban, ahol egy adott modell alkalmazza. Az előző ábrán megnézi, azt csoportosíthatja, hogy különböző függvényeket egytartományos modell megosztja a szolgáltatás. 

![](./images/ddd2.svg) 
 
A kötött környezetek, amelyek nem feltétlenül elkülönülnek egymástól. Ezen a diagramon a csatlakozás a kötött környezetek folytonos vonal képviselő helyen, ahol két kötött környezetben interakciót. Szállítási például felhasználói fiókokat az ügyfelek kapcsolatos információkat, és ütemezni a járműflotta a dronok dron felügyeleti függ.

A könyv *tartomány vezérelt tervezési*, Eric Evans karbantartásához tartománymodell integritását, amikor hatással van egy másik kötött környezetben az több mintákat ismerteti. A fő alapelvek mikroszolgáltatások egyik, a szolgáltatások kommunikálni a jól meghatározott API-k segítségével. Ez a megközelítés megfelel két mintát, hogy Evans meghívja a gazdaszolgáltatás megnyitása és közzétett nyelv. A nyitott Host szolgáltatás lényege, hogy egy alrendszer határozza meg a formális protokoll (API) más alrendszerek kommunikálni. Közzétett nyelvi ezt az elképzelést kiterjeszti közzététele az API-t, amely más csapatokkal használja a ügyfelek űrlap. A fejezet a [API tervezési](./api-design.md), arról lesz szó használatával [OpenAPI Specification](https://www.openapis.org/specification/repo) (korábbi néven Swagger) JSON- vagy YAM formátumban kifejezett REST API-k, nyelvtől független felület leírásainak meghatározásához.

Ez út további tárgyaljuk, amelyet szállítási környezetében. 

## <a name="tactical-ddd"></a>Taktikai DDD

DDD stratégiai fázisában végzett el a vállalati tartományhoz, és meghatározása, amelyet a tartomány modellek környezeteket. Taktikai DDD van, a tartomány modellek pontosabb definiálásakor. Taktikai mintázatokat a egyetlen kötött környezeten belül érvényesek. Mikroszolgáltatások architektúra esetén azt is különösen az entitás és összesített kombinációját. Ezeket a mintákat alkalmazása segíthet számunkra a természetes határok az alkalmazás a szolgáltatásokhoz azonosítása (lásd: [következő fejezet](./microservice-boundaries.md)). Általános szabály egy mikroszolgáltatási összesítő kifejezés nem kisebb, és nem lehet nagyobb a kötött környezetben kell lennie. Először igazolnia áttekinti a taktikai minták. Ezután azt fogja alkalmazni azokat a szállítási, amelyet a környezetben, a dron továbbítási alkalmazást hozhat létre. 

### <a name="overview-of-the-tactical-patterns"></a>A taktikai minták áttekintése

Ez a témakör röviden összefoglalva a taktikai DDD mintákra, így ha már ismeri a DDD, ez a szakasz valószínűleg kihagyhatja. A minták fejezetek 5 részletesen ismerteti a &ndash; 6 Eric Evans címjegyzék-alkalmazásával, valamint a *Implementing Domain-Driven tervezési* Vaughn Vernon által. 

![](./images/ddd-patterns.png)

**Entitások**. Egy entitás egyedi azonosítóval, amely adott idő alatt továbbra is fennáll objektum. Például egy banki alkalmazás, ügyfél és a fiókok lenne entitások. 

- Egy entitás egyedi azonosítóval rendelkezik, a rendszer, amely kereshető vagy bejegyzés lekérdezésére használható. Ez a nem jelenti azt, a azonosítója mindig ki van téve közvetlenül a felhasználók számára. Lehet, hogy a GUID-nak vagy az adatbázis elsődleges kulcs. 
- Az identitás előfordulhat, hogy több kötött környezet span, és előfordulhat, hogy rutinszerű túl az élettartama. Például banki számokat vagy azonosító kormánya által kiállított nem kötődnek, egy adott alkalmazás élettartamát.
- Egy entitás attribútumait idővel megváltozhat. Például egy személy nevét vagy címét változhatnak, ugyanakkor továbbra is ugyanaz a személy. 
- Egy entitás egyéb entitások mutató hivatkozásokat is tárolhatják.
 
**Objektumok érték**. Egy érték objektumnak nincs identitás. Az attribútum értéke csak által van definiálva. Érték objektumok is nem módosíthatók. Egy érték objektum frissítéséhez, mindig egy új példányának létrehozása lecseréli a régi kiszolgálóéval. Érték objektumoknak is lehetnek, amellyel ágyazható be a tartomány logika módszerek, de azokat a módszereket az objektum állapota nem mellékhatásokkal kell. Tipikus érték objektumok például színek, dátumok és idők és pénznemek. 

**Aggregátumok**. Összesítő kifejezés határozza meg a konzisztencia-határ egy vagy több entitás körül. Az összesítő pontosan egy entitás a legfelső szintű. Keresési történik, a legfelső szintű entitás azonosítója. Minden más entitás aggregált a legfelső szintű gyermek, és a legfelső szintű következő mutatók hivatkoznak. 

Összesítő célja a modellhez tartozó tranzakciós invariants. A valós életben dolgot kapcsolatok összetett webhelyek rendelkeznek. Az ügyfelek létre rendelések rendelések tartalmazzák a termékek, termékek szállítók rendelkezik, és így tovább. Ha az alkalmazás több kapcsolódó objektumot sem módosítja, hogy nem garantálja a konzisztencia? Hogyan azt invariants nyomon követheti, és kényszeríteni azokat?  

Hagyományos alkalmazások gyakran használt adatbázis-tranzakció konzisztencia érvényesítéséhez. Az elosztott alkalmazásokban azonban ez gyakran nem valósítható meg. Egyetlen üzleti tranzakció több adattárolókhoz, előfordulhat, hogy span lehet, hogy hosszú ideig futó, vagy külső szolgáltatásokat tartalmazhatnak. Végső soron is az alkalmazáshoz, nem a réteget, a tartomány szükséges invariants kényszerítésére. Ez milyen összesítések úgy van kialakítva, hogy a modell.

> [!NOTE]
> Összesítő állhat egyetlen entitás, gyermek entitások nélkül. Mi teszi összesítő a tranzakciós határ.

**Tartomány-és alkalmazás**. A DDD terminológiát a szolgáltatás olyan olyan objektum, amely megvalósítja az egyes logikai bármely állapotban tartása nélkül. Evans különböztet *tartományi szolgáltatások*, amely foglalják magukban a tartomány programot, és *alkalmazásszolgáltatások*, műszaki funkciókat, például a felhasználói hitelesítést és az SMS küldő nyújt üzenet. Tartományi szolgáltatások több entitás kiterjedő modellként viselkedő gyakran szolgálnak. 

> [!NOTE]
> A kifejezés *szolgáltatás* a szoftver fejlesztési túl van terhelve. Itt nem közvetlenül kapcsolódik mikroszolgáltatások létrehozására.

**Kapcsolódó**. Tartomány események segítségével a rendszer egyéb részeivel értesítése, ha olyan esemény történik. A neve alapján, mint a kapcsolódó valamit a tartományon belül kell jelenti. Például "rekord tábla illesztett" az nem tartományi esemény. "Egy kézbesítési meg lett szakítva" az tartomány esemény. Tartomány események gyakran mikroszolgáltatások architektúra. Mikroszolgáltatások küld, és ne ossza meg adattárolókhoz, mert a tartomány események teszik lehetővé az egymással koordinálására mikroszolgáltatások létrehozására. A fejezet [kommunikációs Interservice](./interservice-communication.md) aszinkron üzenetkezelési részletesebben ismerteti.
 
Nincsenek néhány más DDD minták itt nem látható előállítók, a tárolóhelyekkel és a modulok. Ha egy mikroszolgáltatási webkiszolgálókból hasznos mintái is lehetnek, ugyanakkor kevesebb vonatkozó mikroszolgáltatási közötti határokat tervezésekor.

## <a name="drone-delivery-applying-the-patterns"></a>Kézbesítési dron: a minta alkalmazása

Először a forgatókönyvek, amelyet a szállítási, amelyet a környezetben kell kezelni.

- Az ügyfél kérhet egy dron a dron kézbesítési szolgáltatás regisztrált üzleti termékek átvételéhez.
- A küldő a címke (vonalkóddal vagy RFID) helyezhető el a csomagot hoz létre. 
- Egy dron átvételéhez, és a csomag forráshelyéről kézbesíthet a célhelyen.
- Amikor egy ügyfél egy kézbesítési ütemezi, a rendszer biztosítja az Európai műszaki engedély az útvonal-információkat, a időjárási feltételek és a korábbi adatok alapján. 
- Ha a dron útban, a felhasználó nyomon követheti az aktuális hely és a legújabb Európai. 
- A csomag egy dron rendelkezik felvenni, amíg az ügyfél egy kézbesítési szüntetheti meg.
- Az ügyfél a kézbesítési befejezésekor értesítést kap.
- A küldő kézbesítési megerősítő kérhetnek az ügyfél egy aláírás vagy a nyomtatási ujját formájában.
- Felhasználók befejezett kézbesítési előzményeinek is kereshet.

A fenti forgatókönyvek a fejlesztői csapat a következő azonosított **entitások**.

- Kézbesítési
- Csomag
- Dron
- Fiók
- Megerősítés
- Értesítés
- Címke

Az első négy, a kézbesítés, a csomag, a dron és a fiók, nincs minden **összesítések** , amelyek egységessége határok jelölik. Visszaigazolások és értesítések gyermek entitásai számára kézbesítések, és címkék alárendelt entitások csomagok. 

A **objektumok érték** Ez a kialakítás például a hely, az Európai, a PackageWeight és a PackageSize. 

Mutatja be, ez a kézbesítési összesítés UML ábrázoló diagram. Figyelje meg, hogy rendelkezik-e további összesítést, beleértve a fiók, a csomag és a dron hivatkozik.

![](./images/delivery-entity.png)

Két kapcsolódó van:

- Míg egy dron útban, a dron entitás elküldi a dron hely és az állapot (üzenetsoroktól, szállítási) leíró DroneStatus események.

- A szállítási entitás DeliveryTracking az eseményeket egy szállítási megváltozásakor küldi. Ezek közé tartozik a DeliveryCreated, DeliveryRescheduled, DeliveryHeadedToDropoff és DeliveryCompleted. 

Figyelje meg, hogy ezek az események leírása dolgot bíró belül a modell. Írja le, valamit a tartománnyal kapcsolatos információk, és egy adott programozási nyelv szerkezet nem kötődik.

A fejlesztői csapat azonosított működését, ami nem fér el szépen minden eddigi leírt entitások egy további területén. A rendszer egyes részei kell koordinálja ütemezés vagy frissítésekor. a kézbesítési részt lépéseket. Ezért, a fejlesztői csapat hozzá két **tartományi szolgáltatások** terve: egy *Feladatütemező* koordináló a lépéseket, és egy *felügyelő* , amely az egyes állapotának figyelése lépés annak észleléséhez, hogy olyan lépéseket a sikertelen vagy túllépte az időkorlátot. Ez a módosítás a [Feladatütemező ügynök felügyelő mintát](../patterns/scheduler-agent-supervisor.md).

![](./images/drone-ddd.png)

> [!div class="nextstepaction"]
> [Mikroszolgáltatási határok azonosítása](./microservice-boundaries.md)
