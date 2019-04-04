---
title: Architektúrastílusok
titleSuffix: Azure Application Architecture Guide
description: Gyakori architektúrastílusok felhőalkalmazásokhoz.
layout: LandingPage
ms.topic: landing-page
ms.service: architecture-center
ms.subservice: reference-architecture
ms.date: 08/30/2018
ms.openlocfilehash: 6895ffc9c73dac29c27e7d8df68550c94f68b10a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346535"
---
# <a name="architecture-styles"></a>Architektúrastílusok

Az *architektúrastílus* olyan architektúrák családja, amelyeknek bizonyos tulajdonságai megegyeznek. Az [N szintű][n-tier] például egy gyakori architektúrastílus. Újabban a [mikroszolgáltatás-architektúrák][microservices] is egyre népszerűbbé válnak. Az architektúrastílusok használatához nincs szükség adott technológiákra, egyes technológiák azonban kifejezetten alkalmasak bizonyos architektúrákhoz. A tárolók például természetesen illeszkednek a mikroszolgáltatásokhoz.

Azonosítottuk a felhőalkalmazásokban gyakran alkalmazott architektúrastílusokat. Az egyes stílusokkal foglalkozó cikkek a következőkre térnek ki:

- Az adott stílus leírása és logikai diagramja.
- A stílus alkalmazási területeire vonatkozó ajánlások.
- Előnyök, problémák és ajánlott eljárások.
- Az üzembe helyezés javasolt módja a megfelelő Azure-szolgáltatások használatával.

## <a name="a-quick-tour-of-the-styles"></a>A stílusok gyors bemutatása

Ez a szakasz röviden bemutatja az általunk azonosított architektúrastílusokat, valamint általános áttekintést ad az alkalmazási területeiket illetően. További részleteket a kapcsolódó témakörökben olvashat.

### <a name="n-tier"></a>N szintű

<!-- markdownlint-disable MD033 -->

<img src="./images/n-tier-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

Az **[N szintű][n-tier]** egy hagyományos architektúra vállalati alkalmazások számára. A függőségek kezelése érdekében az alkalmazás *szintekre* van osztva, amely szintekhez logikai függvények vannak hozzárendelve, például a megjelenítés, az üzleti logika vagy az adatok hozzáférése. Az egyes szintek csak az alattuk lévő szinteket tudják meghívni. Az ilyen vízszintes szintelrendezés azonban problémákat okozhat. Adott esetben nehézkes lehet az alkalmas valamely részét úgy módosítani, hogy az ne érintse az alkalmazás egészét. Így a gyakori frissítések problémásnak bizonyulhatnak, ami lelassítja az új funkciók hozzáadásának tempóját.

Az N szintű architektúra természetes választás az eleve szintekre osztott architektúrával rendelkező meglévő alkalmazások migrálásához. Ezért az N szintű architektúra a szolgáltatásként kínált infrastruktúra (IaaS-) megoldások, illetve az IaaS-megoldások és felügyelt szolgáltatások kombinációját használó alkalmazások esetében a leggyakoribb.

### <a name="web-queue-worker"></a>Webüzenetsor-feldolgozó

<!-- markdownlint-disable MD033 -->

<img src="./images/web-queue-worker-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

A tisztán PaaS-megoldások esetében érdemes lehet alkalmazni egy **[Webüzenetsor-feldolgozó](./web-queue-worker.md)** architektúrát. Az ilyen stílusú architektúrákban az alkalmazás egy HTTP-kéréseket feldolgozó webes előtérrendszerrel és egy, a processzorigényes feladatokat és a hosszabb átfutású műveleteket végrehajtó háttérfeldolgozóval rendelkezik. Az előtér egy aszinkron üzenetsor segítségével kommunikál a feldolgozóval.

A Webüzenetsor-feldolgozó architektúra néhány erőforrás-igényes feladattal rendelkező, viszonylag egyszerű tartományok esetében jelent megfelelő megoldást. Az N szintűhöz hasonlóan ez az architektúratípus is könnyen áttekinthető. A felügyelt szolgáltatások használata megkönnyíti az üzembe helyezést és az üzemeltetést. Az összetettebb tartományok esetében azonban a függőségek kezelése nehézkesnek bizonyulhat. Az előtérrendszer és a feldolgozó hatalmas, monolitikus összetevőkké nőhetik ki magukat, amelyeket nehéz karbantartani és frissíteni. Ez, ahogy az N szintű architektúra esetében, itt is csökkentheti a frissítések gyakoriságát, és korlátozhatja a fejlesztést.

### <a name="microservices"></a>Mikroszolgáltatások

<!-- markdownlint-disable MD033 -->

<img src="./images/microservices-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

Ha az alkalmazás összetettebb tartománnyal rendelkezik, érdemes megfontolni a **[Mikroszolgáltatások][microservices]** architektúra használatát. A mikroszolgáltatás-alkalmazásokat sok kisebb, független szolgáltatás alkotja. Mindegyik szolgáltatás egyetlen üzleti képességet valósít meg. A szolgáltatások lazán vannak összekapcsolva, és API-egyezmények útján kommunikálnak.

Az egyes szolgáltatások kialakításával kisebb, fókuszált fejlesztőcsapatok foglalkozhatnak. Az egyes szolgáltatások anélkül vezethetők be, hogy ehhez nagy fokú koordinációra lenne szükség a csapatok között, ami elősegíti a gyakori frissítéseket. A mikroszolgáltatási architektúra kiépítése és felügyelete összetettebb feladat, mint akár az N szintű, akár a webüzenetsor-feldolgozó architektúráé. A használatához megalapozott fejlesztési és DevOps-kultúra szükséges. Ha azonban jól van megvalósítva, ez a stílus hamarabb megjelentethető kiadásokat, gyorsabb fejlesztéseket és rugalmasabb architektúrát biztosít.

### <a name="cqrs"></a>CQRS

<!-- markdownlint-disable MD033 -->

<img src="./images/cqrs-sketch.svg" style="float:left; margin-top:6px;"/>

<!-- markdownlint-enable MD033 -->

A **[CQRS](./cqrs.md)** (Command and Query Responsibility Segregation, azaz parancskiadási és lekérdezési felelősségek elkülönítése) stílus az olvasási és írása műveleteket külön modellekbe rendeli. Ezáltal elszigeteli a rendszer adatokat frissítő részeit az adatokat olvasó részeitől. Ezen túlmenően az adatolvasásra egy olyan, tényleges táblán alapuló nézetben kerülhet sor, amely fizikailag elkülönül az írható adatbázistól. Ennek köszönhetően az olvasási és írási számítási feladatok egymástól függetlenül méretezhetők, és a tényleges táblán alapuló nézet a lekérdezésekre optimalizálható.

A CQRS használata akkor a legcélszerűbb, ha egy nagyobb architektúrának csak egy alrendszerén használják. Általában nem érdemes a teljes alkalmazáson bevezetni, mivel szükségtelenül bonyolult rendszert eredményez. Az olyan együttműködési tartományok esetében érdemes használni, ahol több felhasználó dolgozik ugyanazokkal az adatokkal.

### <a name="event-driven-architecture"></a>Eseményvezérelt architektúra

<!-- markdownlint-disable MD033 -->

<img src="./images/event-driven-sketch.svg" style="float:left; margin-top:6px;"/>

Az **[Eseményvezérelt architektúrák](./event-driven.md)** egy közzétételi-feliratkozási (pub-sub) modellt alkalmaznak, amelyben az adatalkotók közzétesznek eseményeket, az adatfelhasználók pedig feliratkoznak rájuk. Az adatalkotók függetlenek az adatfelhasználóktól, valamint az adatfelhasználók is egymástól.

Az eseményvezérelt architektúra használata olyan alkalmazások esetében javasolt, amelyek nagy mennyiségű adatot olvasnak be és dolgoznak fel nagyon alacsony késleltetéssel. Ilyenek például az IoT-megoldások. Ez az architektúrastílus akkor is hasznos, ha különböző alrendszereknek különböző típusú feldolgozási feladatokat kell elvégeznie ugyanazokon az eseményadatokon.

<br />

<!-- markdownlint-enable MD033 -->

### <a name="big-data-big-compute"></a>Big Data, Big Compute

A **[Big Data](./big-data.md)** és a **[Big Compute](./big-compute.md)** olyan specializált architektúrastílusok, amelyek bizonyos egyedi profilokhoz illeszkednek. A Big Data architektúra egy nagyon nagy adatkészletet adattömbökké darabol, és a teljes készletet párhuzamosan dolgozza fel elemzési és jelentéskészítési célból. A Big Compute vagy más néven nagy teljesítményű feldolgozás (high-performance computing, HPC) párhuzamosan végez egyidejű számításokat nagyon nagy számú (több ezer) magon. Az alkalmazási tartomány lehet szimuláció, modellezés és 3D renderelés.

## <a name="architecture-styles-as-constraints"></a>Az architektúrastílus mint korlátozás

Az architektúrastílusok korlátozzák az alkalmazások kialakítását, beleértve a felhasználható elemek körét és a köztük fennálló kapcsolatokat. A korlátozások az elérhető lehetőségek leszűkítésével formálják az architektúrák „alakját”. Ha egy architektúra igazodik egy adott stílus korlátaihoz, bizonyos kívánatos tulajdonságok alakulnak ki.

A mikroszolgáltatások korlátai például a következők:

- Minden szolgáltatás egyetlen felelősséget jelöl.
- Mindegyik szolgáltatás független a többitől.
- Az adatok csak az azokat birtokló szolgáltatás számára érhetők el. A szolgáltatások nem osztoznak az adatokon.

A korlátok betartásával egy olyan rendszert kapunk, amelyben a szolgáltatások egymástól függetlenül helyezhetők üzembe, a hibák elszigetelten fordulnak elő, lehetséges a gyakori frissítés, és könnyen vezethetők be új technológiák az alkalmazásba.

Az egyes architektúrastílusok kiválasztása előtt mindenképp érdemes megismerni az adott stílus mögött húzódó irányelveket és a rá jellemző korlátokat. Ennek elmulasztása esetén előfordulhat, hogy egy olyan kialakítás valósul meg, amely a felszínen ugyan megfelel az adott stílusnak, azonban nem sikerül kiaknázni a stílusban rejlő minden lehetőséget. Az is fontos, hogy gyakorlatiasan közelítsük meg a kérdést. Néha jobb feloldani valamely korlátot, mint mereven ragaszkodni az architektúra tisztaságához.

Az alábbi tábla összefoglalja, hogyan kezelik az egyes stílusok a függőségeket, és hogy melyik milyen típusú alkalmazási tartományokhoz alkalmas.

| Architektúrastílus | Függőségkezelés | Alkalmazási tartomány típusa |
|--------------------|------------------------|-------------|
| N szintű | Alhálózatokba tagolt vízszintes szintek | Hagyományos üzleti tartományok. Ritkán kerül sor frissítésekre. |
| Webüzenetsor-feldolgozó | Előtér- és háttérfeladatok, aszinkron üzenetküldéssel leválasztva. | Néhány erőforrás-igényes feladattal rendelkező, viszonylag egyszerű tartományok. |
| Mikroszolgáltatások | Függőlegesen (funkcionálisan) elkülönített szolgáltatások, amelyek egymást API-kon keresztül hívják meg. | Összetett tartomány. Gyakori frissítések. |
| CQRS | Írás és olvasás elkülönítése. A séma és a méret külön optimalizálható. | Együttműködési tartományok, ahol több felhasználó dolgozik ugyanazokkal az adatokkal. |
| Eseményvezérelt architektúra | Adatalkotó/adatfelhasználó. Alrendszerenként független nézet. | IoT és valósidejű rendszerek |
| Big Data | Hatalmas adatkészlet felosztása kisebb tömbökre. A helyi adatkészletek párhuzamos feldolgozása. | Kötegelt és valós idejű adatelemzés. Prediktív elemzés ML használatával. |
| Big Compute| Adatok kiosztása több ezer magra. | Nagy számításigényű tartományok, például szimuláció. |

## <a name="consider-challenges-and-benefits"></a>A kihívások és az előnyök mérlegelése

A korlátozások kihívásokat hordoznak magukban, ezért fontos tisztában lenni az előnyökkel és hátrányokkal az egyes stílusok alkalmazásakor. Vajon az architektúrastílus előnyei túlsúlyban vannak az esetleges kihívásokkal szemben *az adott altartomány és a kapcsolódó kontextus esetén*?

Néhány problématípus, amelyeket érdemes mérlegelni az architektúrastílus kiválasztásakor:

- **Összetettség**. Az adott tartomány valóban megköveteli egy összetett architektúra használatát? Vagy ellenkezőleg, esetleg túl egyszerű a választott stílus a tartományhoz? Ha ez a helyzet, egy „[nagy sárgolyó][ball-of-mud]” lesz a végeredmény (egy átláthatatlan rendszer), mivel az architektúra nem segít a függőségek egyértelmű kezelésében.

- **Aszinkron üzenetkezelés és végleges konzisztencia**. Az aszinkron üzenetkezelés használatával a szolgáltatások szétválaszthatók, növelhető a megbízhatóság (mivel az üzenetek újraküldhetőek) és a méretezhetőség. Ez azonban olyan problémákhoz vezethet, mint a mindig-egyszer szemantika és a végleges konzisztencia.

- **Szolgáltatások közötti kommunikáció**. Az alkalmazások külön szolgáltatásokra való lebontásakor fennáll annak a kockázata, hogy a szolgáltatások közötti kommunikáció elfogadhatatlan mértékű késést vagy a hálózat túlterhelését okozza (például a mikroszolgáltatási architektúrákban).

- **Kezelhetőség**. Milyen bonyolult az alkalmazás felügyelete, a monitorozás, a frissítések telepítése stb.?

[ball-of-mud]: https://en.wikipedia.org/wiki/Big_ball_of_mud
[microservices]: ./microservices.md
[n-tier]: ./n-tier.md
