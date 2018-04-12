---
title: Big data-architektúrák
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2a1336faea81470b082d4eef8e2cc53a082c63c7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="big-data-architectures"></a>Big data-architektúrák

A big data típusú architektúrát olyan adatok betöltésére, feldolgozására és elemzésére tervezték, amelyek túl nagyok vagy összetettek lennének a hagyományos adatbázisrendszerek számára. A küszöbérték, amellyel a szervezetek írja be a big Data típusú adatok tartomány eltér attól függően, hogy a felhasználók és az eszközök képességeit. Az egyes azzal járhat több száz gigabájt adatok, míg mások több száz terabájt jelent. Munkavégzés big data készletek eszközök előzetes, mivel ezáltal a big Data típusú adatok jelentése. Nagyobb ez a kifejezés vonatkozik, az érték kinyerése a speciális elemzés keresztül adatkészletek ahelyett, hogy szigorúan csak az adatok mérete bár az ezekben az esetekben Igen tekintélyes lehet általában.

Az évek során az adatok fekvő megváltozott. Milyen akkor teheti meg, vagy tegye, várhatóan adatokkal megváltozott. A tárolók költségét jelentősen, csökkent, amíg az összegyűjtött adatok azt jelenti, hogy növekvő tartja. Néhány adat érkezik egy gyors lépést, folyamatosan gyűjti és megfigyelt igényelnek. Egyéb adat érkezik lassabb, de a nagyon nagy méretű adattömböket, gyakran előzményadatoknak évtizedekben formájában. Előfordulhat, hogy egy speciális elemzésekre probléma, vagy egy gépi tanulás igénylő szemben. Ezek a kihívások, amely a big Data típusú adatok architektúrák pozícionálni oldja meg.

A big data-megoldások általában az alábbi számításifeladat-típusok legalább egyikét tartalmazzák:

* inaktív big data típusú adatforrások kötegelt feldolgozása,
* mozgásban lévő, big data típusú adatok valós idejű feldolgozása,
* big data típusú adatok interaktív feltárása,
* prediktív elemzés és gépi tanulás.

Vegye figyelembe a big Data típusú adatok architektúrák kell:

* a hagyományos adatbázisok számára túl nagy mennyiségű adat tárolása és feldolgozása,
* strukturálatlan adatok átalakítása elemzés és jelentéskészítés céljából,
* kötetlen adatstreamek rögzítése, feldolgozása és elemzése valós időben vagy kis késéssel,

## <a name="components-of-a-big-data-architecture"></a>A big Data típusú adatok architektúra összetevői

Az alábbi ábrán a logikai összetevők, amelyek felelnek meg a big Data típusú adatok architektúra. Egyes megoldások nem tartalmazhat minden elem ezen a diagramon.

![Általános adatok folyamat diagramja](./images/big-data-pipeline.png) 

A legtöbb big data típusú architektúra tartalmazza az alábbi összetevők egy részét vagy mindegyikét:

* **Adatforrások**. Első lépésként minden big data-megoldások egy vagy több adatforrást. Példák erre vonatkozóan:

    * Alkalmazások adattárai (pl. relációs adatbázisok).
    * Az alkalmazások által létrehozott statikus fájlok (pl. webkiszolgálók naplófájljai).
    * Valós idejű adatforrások (pl. IoT-eszközök).

* **Adattárolás**. Adatok kötegelt feldolgozási műveletek általában tárolása egy elosztott fájlrendszer, amely jelentős mennyiségű különböző formátumban nagy fájlok tárolására képes. Az ilyen tárakat gyakran *data lake*-nek is nevezik. Az ilyen tárolók többek között az Azure Data Lake Store vagy az Azure Storage blobtárolóival valósíthatók meg.

* **Kötegfeldolgozási**. Mivel nagy adatkészletek, gyakran a big Data típusú adatok megoldás adatok fájlokat a hosszan futó kötegelt feladatok szűréséhez, összesített és ellenkező esetben az adatok előkészítése az elemzés kell feldolgozni. Ezek a feladatok általában magukban foglalják az adatforrások beolvasását, feldolgozását, valamint a kimenet új fájlokba történő írását. A lehetőségek többek között az alábbiak: U-SQL-feladatok futtatása az Azure Data Lake Analyticsben; Hive-, Pig- vagy egyéni Map/Reduce-feladatok használata egy HDInsight Hadoop-fürtben; illetve Java-, Scala- vagy Python-programok használata egy HDInsight Spark-fürtben.

* **Valós idejű üzenet adatfeldolgozást**. Ha a megoldás a valós idejű eseményforrást tartalmaz, az architektúra adatfolyam feldolgozásra valós idejű üzenetek tárolásához, majd úgy tartalmaznia kell. Ez lehet egy egyszerű adattár, ahol a bejövő üzenetek egy mappába kerülnek feldolgozás céljából. Számos megoldás azonban egy üzenetbetöltő tárat is igényel, amely pufferként működik az üzenetek számára, és támogatja a kibővített feldolgozást, a megbízható kézbesítést, valamint más üzenetsor-kezelési szemantikákat. Ez a része egy adatfolyam-továbbítási architektúra gyakran nevezik adatfolyam pufferelés. A választható lehetőségek az Azure Event Hubs, Azure IoT Hub és Kafka.

* **Az adatfolyam feldolgozása**. Valós idejű üzenetek rögzítésével, miután a megoldás kell dolgozza fel őket szűrés, összesítése és egyéb előkészítése az adatok elemzése. A rendszer ezután egy kimeneti fogadóba írja a feldolgozott streamadatokat. Az Azure Stream Analytics egy felügyelt streamfeldolgozási szolgáltatást biztosít, amely a korlátlan streameken működő, folyamatosan futó SQL-lekérdezéseken alapul. A nyílt forráskódú Apache-streamelési technológiák (pl. Storm- és Spark-streamelés) szintén használhatók a HDInsight-fürtökben.

* **Analitikai adatokat tároló**. Sok big data-megoldások adatok előkészítése analysis, és a feldolgozott adatok szolgáljanak strukturált formátuma nem kérdezhetők le analitikai eszközeivel. A lekérdezések kiszolgálásáért felelős analitikai adattár lehet egy Kimball-stílusú relációs adattárház, ahogy ez a legtöbb hagyományos üzletiintelligencia- (BI-) megoldásban látható. Alternatív megoldásként az adatok egy alacsony késésű NoSQL-technológián (pl. HBase) keresztül is megjeleníthetők, illetve egy interaktív Hive-adatbázisban, amely az elosztott adattárban lévő adatfájlok metaadatainak absztrakcióját tartalmazza. Az Azure SQL Data Warehouse felügyelt szolgáltatást biztosít a nagy méretű felhőalapú adattárházakhoz. A HDInsight támogatja az interaktív Hive, HBase és Spark SQL használatát, amelyekkel szintén előkészíthetők az adatok elemzésre.

* **Elemzési és jelentéskészítési**. A legtöbb big data-megoldások célja az adatok elemzési és jelentéskészítési betekintést. Ahhoz, hogy a felhasználók képesek legyenek elemezni az adatokat, az architektúra tartalmazhat egy adatmodellező réteget, mint például egy többdimenziós OLAP-kockát vagy egy táblázatos adatmodellt az Azure Analysis Servicesben. Emellett a Microsoft Power BI-ban vagy Microsoft Excelben elérhető modellezési és vizualizációs technológiákkal önkiszolgáló üzletiintelligencia-megoldásokat is támogathatnak. Az elemzés és jelentéskészítés az adatszakértők vagy adatelemzők általi végzett interaktív adatfeltárással is végrehajtható. Az ilyen forgatókönyvekhez számos Azure-szolgáltatás támogat analitikus notebookokat (pl. Jupyter), így a felhasználók felhasználják a Python vagy az R terén már megszerzett tudásukat. Nagy méretű adatfeltárás esetén használhatja a Microsoft R Servert önállóan vagy a Sparkkal együtt.

* **Vezénylési**. Megoldásokat ismételt adatfeldolgozási műveletek, a munkafolyamatokban, forrásadatok átalakító, beágyazott alkotják a big Data típusú adatok közötti több források és felül, a feldolgozott adatok betöltése az analitikus adatok tárolóba, vagy az eredményeket a egyenes leküldéses áthelyezése egy jelentést és az irányítópultot. A munkafolyamatok automatizálhatók egy vezénylési technológia (pl. Azure Data Factory vagy Apache Oozie és Sqoop) használatával.

## <a name="lambda-architecture"></a>Lambda architektúrája

Ha nagyon nagy adatkészletekkel dolgozik, a rendezés ügyfelek igénylő lekérdezések futtatása hosszú időt is igénybe vehet. Ezeket a lekérdezéseket valós időben nem hajtható végre, és gyakran igényelnek, mint az algoritmusok [MapReduce](https://en.wikipedia.org/wiki/MapReduce) párhuzamosan működik, amelyek között a teljes adatkészletet. Az eredmények majd elkülöníti a nyers adatok és használható a lekérdezésre.

Egyik ezt a megközelítést hátránya, hogy késés okozna &mdash; feldolgozási néhány órát vesz igénybe, ha egy lekérdezést lehet, hogy adja vissza az eredményeket, amelyek több órával korábbi. Ideális esetben egyes eredményt valós idejű (lehet, hogy adatvesztés pontosság), és ezekkel az eredményekkel, a kötegelt elemzés eredményeinek egyesíteni szeretné.

A **lambda architektúra**, először Nathan Marz által javasolt, hozzon létre két elérési útnak adatfolyamok kezeli ezt a problémát. A rendszerbe érkező összes adat végighalad két elérési utak:

* A **kötegelt réteg** (cold elérési utat) a nyers formátumban tárolja az összes bejövő adat és kötegfeldolgozási végez az adatokat. Az eredmény a feldolgozási tárolja egy **nézet kötegelt**.

* A **sebesség réteg** (gyakran használt adatok elérési útja) elemzi az adatokat valós időben. Ez a réteg végzi a kis késéssel, csökkenti a pontossága.

A kötegelt réteg hírcsatornák be egy **szolgáló réteg** , amely indexeli a kötegelt nézet hatékony lekérdezése. A sebesség réteg frissíti a szolgáló réteg növekményes frissítések legutóbbi adatokon alapulnak.

![Lambda architektúra diagramja](./images/lambda.png)

Adatok bemenetként szolgál a gyakran használt adatok elérési útja korlátozza a sebesség réteg által meghatározott késésre vonatkozó követelmény, hogy a lehető leggyorsabban dolgozható. Gyakran ehhez a kompromisszummal jár, helyett adatok, amelyek a lehető leggyorsabban készen áll a pontossági szint. Vegye figyelembe például az IoT-forgatókönyveknél, ahol nagyszámú hőmérséklet-érzékelő telemetriai adatokat küldenek. A sebesség réteg mozgó időkerete pedig a bejövő adatok feldolgozásához használható. 

A cold elérési összekapcsolását, másrészt nincs ugyanazok a kis késleltetésű vonatkoznak. Ez lehetővé teszi pontosságot számításhoz nagy méretű adatkészletekhez, amelyek akkor nagyon időigényes. 

Végül a és meleg elérési utak: az elemzés ügyfélalkalmazás összevonva. Ha az ügyfelet valós időben jeleníti meg az időben, de a potenciálisan kevésbé pontos adatokat, akkor az eredmény a gyakran használt adatok elérési fogják beszerezni. Ellenkező esetben kiválaszt eredmények a cold elérési kevesebb időben, de még pontosabb adatok megjelenítéséhez. Más szóval a gyakran használt adatok rendelkezik egy viszonylag kis ablakban időt, amely után az eredményeket a cold elérési pontosabb adataival frissítésre vonatkozó adatokat.

A nyers adatok a kötegelt rétegben tárolt nem módosítható. A meglévő adatok mindig fűz hozzá a bejövő adatok, és a korábbi adatok soha nem felülírja a rendszer. Egy adott datum értékének módosításait, egy új időbélyegzővel eseményrekord tárolódnak. Ez lehetővé teszi bármely pontján recomputation időben gyűjtött adatokat az előzmények között. Számítsa újra az eredeti nyers adatok a kötegelt nézet olyan fontos, mert a rendszer fejlődésének létrehozni új nézetek lehetővé teszi. 

## <a name="kappa-architecture"></a>Kappa architektúrája

A lambda architektúrájának hátránya, hogy összetettsége. Két különböző helyen számításokat megjelenik &mdash; a cold és működés közbeni útvonalak &mdash; különböző keretrendszerek használatával. Ennek eredménye duplikálja számítási programot, és mindkét elérési utak architektúrája kezelése összetettsége.

A **kappa architektúra** ahelyett, hogy a lambda architektúra által Jay Kreps volt javasolt. A lambda architektúrája megegyezik, azonban fontos különbség van a ugyanazon alapvető célok: a streamfeldolgozó rendszerek használatával egyetlen elérési összes adatfolyamok. 

![Kappa architektúra diagramja](./images/kappa.png)

A lambda architektúra kötegelt réteghez, néhány Hasonlóságok van, hogy az eseményadatok nem módosítható, és az összes összegyűjtött helyett egy részét. Az adatok az elosztott az események streamjét és hibatűrő egyesített hibanapló van okozhatnak. Ezek az események rendezett, és az esemény aktuális állapotát módosítja a csak fűzött alatt álló új esemény. Egy lambda architektúra sebesség réteg hasonló, minden esemény feldolgozása a bemeneti adatfolyam végre és valós idejű nézetként maradnak. 

Ha számítsa újra a teljes adatkészletet (a batch réteg funkciója lambda kifejezésben egyenértékű) van szüksége, akkor egyszerűen visszajátszásos az adatfolyam használata jellemzően párhuzamossági időben számítási befejezéséhez.

## <a name="internet-of-things-iot"></a>Eszközök internetes hálózata (IoT)

Gyakorlati szempontból az eszközök internetes hálózatát (IoT) jelöli az internethez csatlakozó bármilyen eszközről. Tartalmazzák a számítógép, mobiltelefon, intelligens figyelési, intelligens termosztát, intelligens hűtő, csatlakoztatott autó, szív figyelési sorolni az, és bármely más, csatlakozik az internethez és küld vagy fogad adatokat. A csatlakoztatott eszközök száma nő minden nap, azok gyűjtött adatok mennyisége nem. Gyakran ez folyik adatgyűjtés erősen korlátozott, egyes esetekben a nagy késleltetésű környezetekben. Más esetekben adatküldést kis késleltetésű környezetekben a több ezer vagy eszközöket, gyorsan viszi be az adatokat, és ennek megfelelően feldolgozni több millió. Ezért megfelelő tervezés szükséges ezen korlátozások és egyedi követelmények kezelésére.

Az eseményvezérelt architektúrák központi szerepet játszanak az IoT-megoldásokban. Az alábbi ábrán egy Iot-megoldás lehetséges logikai architektúrája látható. Az ábra az architektúra eseménystreamelési összetevőit hangsúlyozza ki.

![IoT-architektúra](../../guide/architecture-styles/images/iot.png)

A **felhőátjáró** a felhő határán olvassa be az eszközeseményeket egy megbízható, alacsony késésű üzenetkezelési rendszert használva.

Az eszközök közvetlenül a felhőátjárónak vagy egy **helyi átjárón** keresztül küldhetik el az eseményeket. A mező az átjáró az speciális eszközt vagy szoftvert, az eszközök általában közös elhelyezésű fogadó eseményeket, és továbbítja őket a felhőátjáróhoz. A helyi átjáró a nyers eszközesemények előfeldolgozására is képes, olyan feladatok végrehajtásával, mint a szűrés, az összesítés vagy a protokollátalakítás.

A beolvasást követően az események egy vagy több **streamfeldolgozón** haladnak át, amelyek továbbíthatják az adatokat (például egy tárolóba), vagy elemzést és más feldolgozási műveleteket végezhetnek.

Az alábbiakban a feldolgozás néhány gyakori típusát ismertetjük. (A felsorolás semmiképpen sem teljes.)

- Eseményadatok írása offline tárolóba archiválás vagy kötegelt elemzés céljából.

- Működő elérési út elemzése, vagyis az eseménystream (közel) valós idejű elemzése a rendellenességek észlelése, adott időtartamokra jellemző minták felismerése vagy riasztások aktiválása céljából, ha egy adott helyzet áll elő a streamben. 

- Kezeli a különleges típusú eszközök, például az értesítések és riasztások nontelemetry üzeneteit. 

- Gépi tanulás.

A szürke dobozok az IoT-rendszer azon összetevőit jelölik, amelyek nem kapcsolódnak közvetlenül az eseménystreameléshez, a teljesség igénye miatt azonban az ábra részét képezik.

- Az **eszközjegyzék** az üzembe helyezett eszközök adatbázisa, amely az eszközök azonosítóját és rendszerint az eszközök metaadatait, például a helyüket tartalmazza.

- Az **üzembe helyezési API** egy általános külső felület az új eszközök üzembe helyezéséhez és regisztrálásához.

- Egyes IoT-megoldások lehetővé teszik **parancs- és vezérlő üzenetek** küldését az eszközöknek.

Kapcsolódó Azure-szolgáltatások:

- [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/)
- [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/)
- [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/)  

További tudnivalók az Azure IoT ehhez beolvassa a [Azure IoT-referenciaarchitektúra](https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/).


