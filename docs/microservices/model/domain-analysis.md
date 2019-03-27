---
title: A mikroszolgáltatásokat tartományelemzés
description: A mikroszolgáltatásokat tartományelemzés.
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 7f3ce8bb7d1e03d59670cc4685589ec76738d95c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298914"
---
# <a name="using-domain-analysis-to-model-microservices"></a>Tartományelemzés a mikroszolgáltatás-alapú modell használatával

A legnagyobb kihívás a mikroszolgáltatások egyik meghatározni az egyes szolgáltatások. Az általános szabály, hogy egy szolgáltatás hajtsa végre a "jó" &mdash; azonban, hogy a szabály üzembe gyakorlat tanulmányozásakor. Nincs egyetlen vehesse a műszaki folyamat, amely a "megfelelő" tervezési állítja elő. Hogy mélyen gondolja át az üzleti tartomány, követelmények és célok. Ellenkező esetben fejezheti be néhány nemkívánatos jellemzői, például a szolgáltatások, a rövid kapcsoló, vagy rosszul tervezett felületek közötti rejtett függőségek kiváló haphazard kialakítás. Ez a cikk bemutatja a mikroszolgáltatások tervezése tartományvezérelt megközelítést.

Ez a cikk egy drónos szállítási szolgáltatást használ egy futó példát. Talál további információt a forgatókönyv és a megfelelő megvalósításának hivatkozhat [Itt](../design/index.md).

## <a name="introduction"></a>Bevezetés

Mikroszolgáltatás-alapú üzleti képességekről, például az adatok elérése vagy az üzenetkezelési nem vízszintes rétegek köré kell építeni. Emellett laza összekapcsolással és magas működési kohézióval kell rendelkezniük. Mikroszolgáltatások *lazán kapcsolódó* Ha módosítható egy szolgáltatást anélkül, hogy más szolgáltatások egyszerre frissíteni kell. A mikroszolgáltatások általában *javul* hogy vannak-e egyetlen, jól meghatározott célra, például a felhasználói fiókok kezelése, vagy nyomon követi a szállítás előzményei. Egy szolgáltatás kell magába foglalja az adatait, és ügyfelek kategorizálják absztrakt. Egy ügyfél például egy drónt ütemezése anélkül, hogy az ütemezési algoritmust, vagy a drone flotta kezelésének módját részleteinek képesnek kell lennie.

Tartományvezérelt tervezési (DDD) olyan keretrendszert kínál, kerülhetnek a legtöbb, a módszer jól megtervezett mikroszolgáltatások készletét. DDD stratégiai és taktikai fázisa van két különböző. A stratégiai nnn a rendszer a nagy méretű struktúra definiálása. Stratégiai DDD segít, hogy az architektúra üzleti funkciók összpontosítanak. Taktikai DDD ismertet néhány tervezési mintát, használhatja a tartományi modell létrehozásához. Ezek a minták entitások, az összesítések és a tartományi szolgáltatások közé tartozik. E taktikai minták segítségével tervezheti meg, amelyek egyaránt lazán kapcsolódó és javul.

![Egy tartományvezérelt tervezési (DDD) folyamat ábrája](../images/ddd-process.png)

Az ebben a cikkben, a következő végigvezetjük a következő lépéseket a Drone Delivery alkalmazás való alkalmazásával:

1. Indítsa el az üzleti tartomány tudni, hogy az alkalmazás működési követelmények elemzésével. Ebben a lépésben a kimenete a tartományhoz, amely is tartománymodellek formális készletét át lesz alakítva egy informális leírását.

2. Ezt követően adja meg a *környezetek határolt* annak a tartománynak. Egyes kapcsolódó kontextusai tartalmaz olyan tartományi modell, amely egy adott nevű, a nagyobb alkalmazás jelöli.

3. A körülhatárolt kontextus belül taktikai DDD-minták meghatározására, entitás, összesítések és a tartományi szolgáltatások vonatkoznak.

4. Az előző lépésből származó eredmények segítségével azonosíthatja a mikroszolgáltatás-alapú, az alkalmazásban.

Ez a cikk ismerteti az első három lépést, amely elsősorban DDD. A következő cikkben hogy azonosítani a mikroszolgáltatás-alapú. Azonban fontos megjegyezni, hogy DDD-iteratív, folyamatban lévő folyamat. A szolgáltatás határain kőbe nem rögzített. Módon alakul ki egy alkalmazást, dönthet úgy újratervezéssel a szolgáltatás számos kisebb szolgáltatásra.

> [!NOTE]
> Ez a cikk a teljes és átfogó tartományelemzés nem jelenik meg. Hogy szándékosan tartani a példa rövid, a fő pontokat mutatja be. Ismertető háttér-DDD, javasoljuk, hogy Eric Evans *driven tervezési*, a könyvet, amely az előfizetési időszak első bevezetése. A hivatkozás egy másik jó *Implementing Domain-Driven tervezési* Vaughn Vernon szerint.

## <a name="scenario-drone-delivery"></a>Forgatókönyv: Drone delivery

A Fabrikam vállalat egy drónos szállítási szolgáltatást indít. A cég egy drónokból álló flottát kezel. A vállalkozások regisztrálnak a szolgáltatásban, és a felhasználók kérhetik egy termék drónos kézbesítését. Amikor az ügyfél ütemezi a felvételt, egy háttérrendszer hozzárendel egy drónt, és értesíti a felhasználót a várható szállítási időről. Amíg a kézbesítés folyamatban van, az ügyfél nyomon követheti a drón helyét, miközben a becsült érkezési időpont folyamatosan frissül.

Ez a forgatókönyv egy viszonylag összetett tartományt foglal magába. A vállalat számára problémát jelenthet a drónok ütemezése, a csomagok nyomon követése, a felhasználói fiókok felügyelete, valamint az előzményadatok tárolása és elemzése. Ezenkívül a Fabrikam gyorsan piacra szeretne lépni, majd gyors iterációkat kíván végezni, új funkciók és képességek hozzáadásával. Az alkalmazásnak felhőszinten kell üzemelnie, magas szolgáltatási szintű célkitűzéssel (service level objective, SLO). A Fabrikam emellett arra számít, hogy a rendszer különböző részeinek nagyon eltérő adattárolási és lekérdezési igényei lesznek. A megfontolandó szempontok alapján a Fabrikam azt a döntést hozta, hogy egy mikroszolgáltatási architektúrát választ a Drone Delivery alkalmazáshoz.

## <a name="analyze-the-domain"></a>A tartomány elemzése

DDD módszerével mikroszolgáltatások tervezheti meg, hogy minden szolgáltatás forms természetesen illeszkednek a egy működési üzleti igény szerint segít. Megkönnyíti, hogy ne a trap lehetővé teszik a szervezet határain vagy technológiai lehetőségek diktálni a tervező.

Kód írása előtt szüksége van egy Madártávlatból létrehozásakor a rendszer. DDD elindítja az üzleti tartomány modellezés, és hozzon létre egy *tartománymodell*. A tartományi modellben egy absztrakt modell az üzleti tartomány. Azt a mondatokból és adatait rendezi, és egy közös nyelvvel biztosít a fejlesztők és a tartomány szakértői.

Első lépésként minden az üzleti funkciók és a kapcsolatok hozzárendelése. Ez valószínűleg egy együttműködésen alapuló erőfeszítés, amely magában foglalja a tartomány-szakértők, szoftverek tervezők és más érdekelt felek. Bármely adott formalism használatához nincs szükség.  Diagram Vázlat vagy rajztáblaként rajzolni.

Töltse ki a diagramon, mivel előfordulhat, hogy megkezdése diszkrét altartományok azonosításához. Mely függvények szorosan kapcsolódó? Amelyek funkciók alapvető üzleti, és kiegészítő szolgáltatásokat biztosító? Mi az a függőségi grafikon? Ebben a kezdeti fázisban az érintett technológiákat és megvalósítási részletei nem. Ugyanakkor vegye figyelembe a hely, ahol az alkalmazás külső rendszerekkel, mint például a CRM, integrálható a fizetés feldolgozása, illetve rendszerek számlázási kell.

## <a name="example-drone-delivery-application"></a>Példa: Drone delivery alkalmazás

Néhány kezdeti tartomány az elemzést a Fabrikam csoport vonalkódképet hozzávetőleges vázlatot, és a Drone Delivery tartományhoz.

![A Drone Delivery tartomány diagramja](../images/ddd1.svg)

- **A szállítási** alapvető üzleti, mert a diagram közepére kerül. Minden más, a diagram létezik, ez a funkció engedélyezéséhez.
- **Felügyeleti drónos** alapvető üzleti is van. Szorosan kapcsolódik a drone felügyeleti funkciókat tartalmaz **drónos javítási** és **prediktív elemzési** előre, amikor szüksége van a drónok a karbantartás és a karbantartás.
- **DRÓN elemzési** biztosít a begyűjtés és kézbesítési becsült idő.
- **Külső szállítás** szállítási alternatív módszerek ütemezése, ha a csomag nem lehet rendszerrel szállított teljesen drónos az alkalmazás lehetővé teszi.
- **Megosztás drónos** egy lehetséges az alapvető üzleti-bővítmény. A vállalat bizonyos időszakokban drónos felesleges kapacitással rendelkezhet, és sikerült bérleti drónok, amelyek egyébként üresjárati ki. Ez a funkció az eredeti kiadásban nem.
- **Videó felügyeleti** van egy másik olyan terület, amely a vállalat előfordulhat, hogy bontsa ki az később.
- **Felhasználói fiókok**, **számlázás**, és **ügyfélszolgálatával** vannak, amelyek támogatják az alapvető üzleti altartományt.

Figyelje meg, hogy ezen a ponton a folyamat még nem végeztünk megvalósítása vagy technológiákkal kapcsolatos döntések. Néhány az alrendszerek magában foglalhatja a szoftver külső rendszerek vagy harmadik féltől származó szolgáltatásokkal. Még ha így az alkalmazásnak kell interakcióba ezen rendszerek és szolgáltatások, ezért fontos, hogy felveszi a tartományi modellben.

> [!NOTE]
> Ha egy alkalmazás egy külső rendszer függ, annak a kockázata, hogy a külső rendszer sémát vagy API-t fog memóriavesztés az alkalmazásba, végső soron veszélyeztetése az architekturális tervezés. Ez különösen igaz, amelyek nem mindig a modern ajánlott eljárásokat, és előfordulhat, hogy használja, pl. szövevényes adatsémák, elavult API-k vagy régebbi rendszerekkel. Ebben az esetben fontos a külső rendszerek és az alkalmazás között a jól definiált határok rendelkezik. Fontolja meg a [Leépítő minta](../../patterns/strangler.md) vagy a [Sérülésgátló réteg minta](../../patterns/anti-corruption-layer.md) erre a célra.

## <a name="define-bounded-contexts"></a>Adja meg a kapcsolódó kontextusokat

A tartományi modell tartalmazza semmilyen felelősséget a világ valós hálózata &mdash; felhasználók, drónok, csomagokat és így tovább. De ez nem jelenti azt, hogy a rendszer minden része kell használni az azonos reprezentációinak ugyanazokat a műveleteket.

Például alrendszereket, a drone javítása és a prediktív elemzés kezelni kell, amelyek sok fizikai jellemzők drónok, például a karbantartási előzményei, távolság, kor, modellszám, teljesítményt nyújt és stb. De amikor egy kézbesítési ütemezni, azt nem érdeklik ezeket a dolgokat. Az ütemezési alrendszer csak tudnia kell, hogy rendelkezésre áll-e egy drónt, és a felvétel és kézbesítési DRÓN.

Ha létrehoz egy egyetlen modellt mindkét alrendszereket megpróbáltuk, szükségtelenül összetett lenne. Azt is válnak nehezebb a modellhez való idővel, mert több csapat dolgozik a különböző alrendszereket teljesítéséhez szüksége lesz a módosításokat. Ezért érdemes gyakran külön modellek, amelyek ugyanazon valós entitás (ebben az esetben egy drónt) két különböző környezetekben tervezéséhez. Minden modell tartalmazza, csak a szolgáltatások és az attribútumokat, amelyek fontosak az adott környezeten belül.

Ekkor nyernek a DDD fogalmát *környezetek határolt* play kerül. A körülhatárolt kontextus egyszerűen a határt, a tartományban, ahol egy adott modell vonatkozik. Az előző ábrán megnézzük, hogy csoportosíthatja funkció e a különböző függvényeket oszt egy szakterületi modell szerint.

![Kapcsolódó kontextusokat ábrája](../images/ddd2.svg)

Kapcsolódó kontextusokat, amelyek nem feltétlenül különítve egymástól. Ezen az ábrán a folytonos vonal a kapcsolódó kontextusokat csatlakozás helyeken, ahol két kapcsolódó kontextusokat interaktív képviseli. A szállítási címhez tartozó felhasználói fiókoknak az ügyfelek adatainak lekérése, és a flotta a drónok ütemezése Drónos felügyeleti függ.

A könyv *Domain-Driven Design*, Eric Evans számos olyan tartományi modell integritásának fenntartása, amikor interaktálnak az egy másik körülhatárolt kontextus mintákat ismerteti. Mikroszolgáltatások fő alapelveit egyik célja, hogy a szolgáltatások jól meghatározott API-kon keresztül kommunikálnak. Ez a módszer felel meg a két minta, hogy Evans meghívja a gazdagépen nyissa meg a szolgáltatási és közzétett nyelvi. A megnyitott gazdagép szolgáltatás lényege, hogy egy alrendszer határozza meg a hivatalos protokoll (API) kommunikálni más alrendszerek. Közzétett nyelvi kibővíti ezt az elképzelést tegye közzé az API-t egy űrlapon, amellyel más csapatokkal az ügyfelek írása. A cikk [API-k tervezése a mikroszolgáltatásokat](../design/api-design.md), megbeszélhetünk használatával [OpenAPI-specifikáció](https://www.openapis.org/specification/repo) (korábban Swagger néven) kifejezett meghatározásához a REST API-k, nyelvtől független felület leírása JSON vagy a YAML formátumú.

Tartson a folyamatban a többi fogunk koncentrálni, amelyet szállítási keretében.

## <a name="next-steps"></a>További lépések

Miután befejezte egy tartományelemzés, a következő lépés az taktikai DDD, a tartomány modellek nagyobb pontossággal meghatározásához a alkalmazni.

> [!div class="nextstepaction"]
> [Taktikai DDD](./tactical-ddd.md)
