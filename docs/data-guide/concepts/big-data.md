---
title: "Big data-architektúrák"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2a1336faea81470b082d4eef8e2cc53a082c63c7
ms.sourcegitcommit: 023d88e781f7fe64c62b247d876441ee40921b1b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/28/2018
---
# <a name="big-data-architectures"></a>Big data-architektúrák

A big data típusú architektúrák kialakításuknak köszönhetően képesek kezelni az olyan adatok bevitelét, feldolgozását és elemzését, amelyek túl nagyok vagy túl összetettek a hagyományos adatbázisrendszerek számára. A küszöbérték, amellyel a szervezetek írja be a big Data típusú adatok tartomány eltér attól függően, hogy a felhasználók és az eszközök képességeit. Az egyes azzal járhat több száz gigabájt adatok, míg mások több száz terabájt jelent. Munkavégzés big data készletek eszközök előzetes, mivel ezáltal a big Data típusú adatok jelentése. Nagyobb ez a kifejezés vonatkozik, az érték kinyerése a speciális elemzés keresztül adatkészletek ahelyett, hogy szigorúan csak az adatok mérete bár az ezekben az esetekben Igen tekintélyes lehet általában.

Az évek során az adatok fekvő megváltozott. Milyen akkor teheti meg, vagy tegye, várhatóan adatokkal megváltozott. A tárolók költségét jelentősen, csökkent, amíg az összegyűjtött adatok azt jelenti, hogy növekvő tartja. Néhány adat érkezik egy gyors lépést, folyamatosan gyűjti és megfigyelt igényelnek. Egyéb adat érkezik lassabb, de a nagyon nagy méretű adattömböket, gyakran előzményadatoknak évtizedekben formájában. Előfordulhat, hogy egy speciális elemzésekre probléma, vagy egy gépi tanulás igénylő szemben. Ezek a kihívások, amely a big Data típusú adatok architektúrák pozícionálni oldja meg.

Big data-megoldások általában tartalmaz, amely a következő típusú alkalmazások és szolgáltatások közül:

* A big Data típusú adatok források aktívan kötegfeldolgozási.
* A big data mozgó valós idejű feldolgozással.
* A big Data típusú adatok interaktív áttekintését.
* A prediktív elemzés és a gépi tanulás.

Vegye figyelembe a big Data típusú adatok architektúrák kell:

* A hagyományos adatbázis túl nagy kötetek tároló és a folyamat adatait.
* Az elemzési és jelentéskészítési strukturálatlan adatok átalakítása.
* Rögzítési, a folyamat, és unbounded adatstreamek valós időben, vagy a kis késleltetésű elemzése.

## <a name="components-of-a-big-data-architecture"></a>A big Data típusú adatok architektúra összetevői

Az alábbi ábrán a logikai összetevők, amelyek felelnek meg a big Data típusú adatok architektúra. Egyes megoldások nem tartalmazhat minden elem ezen a diagramon.

![Általános adatok folyamat diagramja](./images/big-data-pipeline.png) 

A big data-architektúrák vagy azok egy részét az alábbi összetevőket tartalmazza:

* **Adatforrások**. Első lépésként minden big data-megoldások egy vagy több adatforrást. Példák erre vonatkozóan:

    * Alkalmazásadatok tárolja, például a relációs adatbázisok.
    * Statikus fájlok, alkalmazások, például a web server naplófájlok által előállított.
    * Valós idejű adatforrások, például az IoT-eszközök.

* **Adattárolás**. Adatok kötegelt feldolgozási műveletek általában tárolása egy elosztott fájlrendszer, amely jelentős mennyiségű különböző formátumban nagy fájlok tárolására képes. Az ilyen tároló gyakran nevezik egy *a data lake*. Ez a tároló végrehajtási lehet például az Azure Data Lake Store vagy a blob-tárolóban az Azure Storage.

* **Kötegfeldolgozási**. Mivel nagy adatkészletek, gyakran a big Data típusú adatok megoldás adatok fájlokat a hosszan futó kötegelt feladatok szűréséhez, összesített és ellenkező esetben az adatok előkészítése az elemzés kell feldolgozni. Általában ezeket a feladatokat tartalmaz, amely forrásfájlok olvasása, őket, és a kimeneti új fájlok írása. A választható lehetőségek az Azure Data Lake Analytics U-SQL-feladatok futtatása, a Hive, Pig vagy egyéni térkép vagy csökkentse a feladatok egy HDInsight Hadoop-fürt használatával, vagy a HDInsight Spark-fürtben lévő Java, Scala vagy Python programokkal.

* **Valós idejű üzenet adatfeldolgozást**. Ha a megoldás a valós idejű eseményforrást tartalmaz, az architektúra adatfolyam feldolgozásra valós idejű üzenetek tárolásához, majd úgy tartalmaznia kell. Ez lehet egy egyszerű adattár, ahol a bejövő üzenetek eldobott datagramok feldolgozás céljából egy mappába. Sok megoldás azonban adatfeldolgozást üzenettároló puffer üzenetek, illetve kibővített feldolgozás, a megbízható és egyéb üzenet Üzenetsor-kezelés szemantikáját támogatásához szükséges. Ez a része egy adatfolyam-továbbítási architektúra gyakran nevezik adatfolyam pufferelés. A választható lehetőségek az Azure Event Hubs, Azure IoT Hub és Kafka.

* **Az adatfolyam feldolgozása**. Valós idejű üzenetek rögzítésével, miután a megoldás kell dolgozza fel őket szűrés, összesítése és egyéb előkészítése az adatok elemzése. A feldolgozott adatfolyamok majd írni kimeneti fogadóként. Az Azure Stream Analytics felügyelt streamfeldolgozó termékbevételezésekor fut, amely unbounded adatfolyamok működik az SQL-lekérdezések szolgáltatáson biztosít. Nyílt forráskódú Apache adatfolyam hasonló technológiáknak köszönhetően a Storm és a Spark Streaming a HDInsight-fürtök is használható.

* **Analitikai adatokat tároló**. Sok big data-megoldások adatok előkészítése analysis, és a feldolgozott adatok szolgáljanak strukturált formátuma nem kérdezhetők le analitikai eszközeivel. Ezeket a lekérdezéseket kiszolgálásához használt analitikus adattároló Kimball stílusú relációs adatraktár, lehet, a hagyományos üzleti intelligenciával megoldások látható módon. Azt is megteheti az adatok sikerült jelenik meg a kis késleltetésű NoSQL technológia, például a HBase, vagy egy interaktív Hive-adatbázis, amely a metaadatokat absztrakciós keresztül az elosztott adattár-adatfájlok keresztül. Az SQL Data Warehouse nagyméretű, felhőalapú adatraktárra vonatkozó felügyelt szolgáltatást nyújt. HDInsight támogatja, interaktív struktúra, a HBase és a Spark SQL, amely vonatkozó adatok elemzési célú kiszolgálásához is használható.

* **Elemzési és jelentéskészítési**. A legtöbb big data-megoldások célja az adatok elemzési és jelentéskészítési betekintést. Építve a felhasználók számára az adatok elemzése, az architektúra a réteg, például a többdimenziós OLAP-kocka vagy az Azure Analysis Services táblázatos adatmodell modellezési adatokat is tartalmazhat. Az önkiszolgáló üzleti Intelligencia, a Microsoft Power bi-ban vagy a Microsoft Excel modellezési és a képi megjelenítés technológiái is használható lehet. Elemzési és jelentéskészítési is számos adatok interaktív áttekintését adatszakértőkön vagy adatelemzők formájában. Ezek a forgatókönyvek számos Azure-szolgáltatások támogatási analitikai notebookok, például a Jupyter, ezeket a felhasználókat, hogy kihasználja a meglévő ismeretei r és Python engedélyezése A nagyméretű adatok feltárása, használhatja a Microsoft R Server, vagy önálló vagy Spark.

* **Vezénylési**. Megoldásokat ismételt adatfeldolgozási műveletek, a munkafolyamatokban, forrásadatok átalakító, beágyazott alkotják a big Data típusú adatok közötti több források és felül, a feldolgozott adatok betöltése az analitikus adatok tárolóba, vagy az eredményeket a egyenes leküldéses áthelyezése egy jelentést és az irányítópultot. Ezek a munkafolyamatok automatizálását, használhatja az orchestration technológia ilyen Azure Data Factory vagy az Apache Oozie és a Sqoop.

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

Eseményvezérelt architektúrák olyan központi helyet foglalnak el az IoT-megoldások. Az alábbi ábra a IoT egy lehetséges logikai architektúráját mutatja be. A diagram az esemény-adatfolyam-összetevők a architektúra emeli ki.

![IoT-architektúra](../../guide/architecture-styles/images/iot.png)

A **felhőátjáróhoz** ingests eszköz események, a felhő határt, egy megbízható, alacsony késésű rendszer üzenetküldési használatával.

Eszközök el tudja küldeni események közvetlenül az átjáró, vagy egy **mező átjáró**. A mező az átjáró az speciális eszközt vagy szoftvert, az eszközök általában közös elhelyezésű fogadó eseményeket, és továbbítja őket a felhőátjáróhoz. A mező átjáró is előfordulhat, hogy előfeldolgozása nyers eszköz események, funkciókat, például a szűrést, összesítési vagy protokoll átalakítás végrehajtásához.

Bevitel, után események halad át egy vagy több **adatfolyam-processzorok** , amelyek az adatok (például, hogy a tároló) útvonalát, vagy hajtsa végre a kapcsolódó elemzések és egyéb feldolgozási.

Az alábbiakban néhány gyakori hibatípusokat feldolgozása. (A lista létrehozási biztosan nem teljes.)

- Írás a esemény cold tárhely, az archiválás vagy kötegelt elemzés.

- Elérési út analytics közbeni, az esemény-adatfolyam elemzésekor a (mellett) valós idejű, rendellenességek észlelését ismeri fel az minták keresztül működés közbeni idő windows vagy aktiváltak riasztásokat, ha egy adott feltétel lép fel az adatfolyam. 

- Kezeli a különleges típusú eszközök, például az értesítések és riasztások nontelemetry üzeneteit. 

- Gépi tanulás.

A listák, amelyek az egyes elemek árnyékolt szürke összetevőket az IoT-rendszer nem közvetlenül kapcsolódó adatfolyam-esemény, de ide tartoznak a teljesség megjelenítése.

- A **eszközbeállításjegyzékben** kiépített eszközt, beleértve az eszköz azonosítóját, és általában eszköz metaadatait, például a hely egy adatbázis.

- A **API kiépítés** egy közös külső felület üzembe helyezési és új eszközök regisztrációja.

- Bizonyos IoT-megoldás lehetővé teszi **parancs és a vezérlő üzenetek** eszközökre küldendő.

Azure-szolgáltatásokhoz kapcsolódó:

- [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/)
- [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/)
- [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/)  

További tudnivalók az Azure IoT ehhez beolvassa a [Azure IoT-referenciaarchitektúra](https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/).


