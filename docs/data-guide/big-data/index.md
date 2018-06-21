---
title: Big data-architektúrák
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2a1336faea81470b082d4eef8e2cc53a082c63c7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
ms.locfileid: "30298855"
---
# <a name="big-data-architectures"></a>Big data-architektúrák

A big data típusú architektúrát olyan adatok betöltésére, feldolgozására és elemzésére tervezték, amelyek túl nagyok vagy összetettek lennének a hagyományos adatbázisrendszerek számára. Minden cégnél eltér, hogy milyen adatmennyiségnél kezdik el a big data használatát. Ez függ a felhasználók képességeitől és a rendelkezésre álló eszközöktől is. Egyeseknél már több száz gigabájt, másoknál csak több száz terabájt esetén van rá szükség. A big data kezelésére szolgáló eszközök folyamatosan fejlődnek, de változik az is, hogy mit értünk big data alatt. A kifejezés egyre inkább a nyers adathalmazokból fejlett elemzési módszerekkel kinyerhető értéket jelenti, és nem egyszerűen sok adatot, bár az elemzést többnyire nagy adathalmazokon szokták végezni.

Az évek során megváltozott az adatkörnyezet. Az adatok felhasználási lehetőségei és az ezekkel kapcsolatos elvárások is megváltoztak. Az adattárolás költsége jelentősen csökkent, ezért rohamosan növekszik a tárolt adatok mennyisége. Bizonyos adattípusok gyorsan gyűlnek, és állandó begyűjtést és megfigyelést igényelnek. Más adatok lassan, de hatalmas tömbökben érkeznek, gyakran több évtized előzményadatai formájában. Előfordulnak összetett elemzési problémák, vagy olyanok, amelyek megoldásához gépi tanulás szükséges. A big data-architektúrák az ilyen kihívások megoldására szolgálnak.

A big data-megoldások általában az alábbi számításifeladat-típusok legalább egyikét tartalmazzák:

* inaktív big data típusú adatforrások kötegelt feldolgozása,
* mozgásban lévő, big data típusú adatok valós idejű feldolgozása,
* big data típusú adatok interaktív feltárása,
* prediktív elemzés és gépi tanulás.

Fontolja meg a big data-architektúrák használatát a következő esetekben:

* a hagyományos adatbázisok számára túl nagy mennyiségű adat tárolása és feldolgozása,
* strukturálatlan adatok átalakítása elemzés és jelentéskészítés céljából,
* kötetlen adatstreamek rögzítése, feldolgozása és elemzése valós időben vagy kis késéssel,

## <a name="components-of-a-big-data-architecture"></a>A big data-architektúrák összetevői

A következő ábrán áttekintheti a big data-architektúrák logikai összetevőit. Lehetséges, hogy az egyes megoldások nem tartalmazzák az ábra összes elemét.

![A teljes adatfolyamat ábrája](./images/big-data-pipeline.png) 

A legtöbb big data típusú architektúra tartalmazza az alábbi összetevők egy részét vagy mindegyikét:

* **Adatforrások**. Minden big data-megoldás egy vagy több adatforrással kezdődik. Példák erre vonatkozóan:

    * Alkalmazások adattárai (pl. relációs adatbázisok).
    * Az alkalmazások által létrehozott statikus fájlok (pl. webkiszolgálók naplófájljai).
    * Valós idejű adatforrások (pl. IoT-eszközök).

* **Adattárolás**. A kötegelt feldolgozási műveletekhez használatos adatokat általában egy elosztott fájltároló tartalmazza, amely számos formátumú és nagy mennyiségű nagy méretű adatot képes tárolni. Az ilyen tárakat gyakran *data lake*-nek is nevezik. Az ilyen tárolók többek között az Azure Data Lake Store vagy az Azure Storage blobtárolóival valósíthatók meg.

* **Kötegelt feldolgozás**. Mivel az adatkészletek rendkívül nagy méretűek, a big data-megoldásoknak gyakran hosszan futó kötegelt feladatok használatával kell feldolgozniuk az adatfájlokat az adatok szűréséhez, összesítéséhez és az elemzésre való egyéb módon történő előkészítéséhez. Ezek a feladatok általában magukban foglalják az adatforrások beolvasását, feldolgozását, valamint a kimenet új fájlokba történő írását. A lehetőségek többek között az alábbiak: U-SQL-feladatok futtatása az Azure Data Lake Analyticsben; Hive-, Pig- vagy egyéni Map/Reduce-feladatok használata egy HDInsight Hadoop-fürtben; illetve Java-, Scala- vagy Python-programok használata egy HDInsight Spark-fürtben.

* **Valós idejű üzenetbetöltés**. Ha a megoldás tartalmaz valós idejű forrásokat, az architektúrának lehetővé kell tennie a valós idejű üzenetek rögzítését és tárolását a streamfeldolgozáshoz. Ez lehet egy egyszerű adattár, ahol a bejövő üzenetek egy mappába kerülnek feldolgozás céljából. Számos megoldás azonban egy üzenetbetöltő tárat is igényel, amely pufferként működik az üzenetek számára, és támogatja a kibővített feldolgozást, a megbízható kézbesítést, valamint más üzenetsor-kezelési szemantikákat. A streamelési architektúra ezen részét gyakran streampufferelésnek nevezik. A lehetőségek többek között a következők: Azure Event Hubs, Azure IoT Hub és a Kafka.

* **Streamfeldolgozás**. A valós idejű üzenetek rögzítése után a megoldásnak fel kell dolgoznia, azaz szűrnie, összesítenie és egyéb módon elő kell készítenie az adatokat az elemzéshez. A rendszer ezután egy kimeneti fogadóba írja a feldolgozott streamadatokat. Az Azure Stream Analytics egy felügyelt streamfeldolgozási szolgáltatást biztosít, amely a korlátlan streameken működő, folyamatosan futó SQL-lekérdezéseken alapul. A nyílt forráskódú Apache-streamelési technológiák (pl. Storm- és Spark-streamelés) szintén használhatók a HDInsight-fürtökben.

* **Analitikus adattár**. Számos big data-megoldás előkészíti az adatokat az elemzésre, majd strukturált formátumban rendelkezésre bocsátja a feldolgozott adatokat, hogy lekérdezhetők legyenek elemzőeszközökkel. A lekérdezések kiszolgálásáért felelős analitikai adattár lehet egy Kimball-stílusú relációs adattárház, ahogy ez a legtöbb hagyományos üzletiintelligencia- (BI-) megoldásban látható. Alternatív megoldásként az adatok egy alacsony késésű NoSQL-technológián (pl. HBase) keresztül is megjeleníthetők, illetve egy interaktív Hive-adatbázisban, amely az elosztott adattárban lévő adatfájlok metaadatainak absztrakcióját tartalmazza. Az Azure SQL Data Warehouse felügyelt szolgáltatást biztosít a nagy méretű felhőalapú adattárházakhoz. A HDInsight támogatja az interaktív Hive, HBase és Spark SQL használatát, amelyekkel szintén előkészíthetők az adatok elemzésre.

* **Elemzés és jelentéskészítés**. A legtöbb big data-megoldás célja az, hogy elemzéssel és jelentéskészítéssel betekintést nyújtson az adatokba. Ahhoz, hogy a felhasználók képesek legyenek elemezni az adatokat, az architektúra tartalmazhat egy adatmodellező réteget, mint például egy többdimenziós OLAP-kockát vagy egy táblázatos adatmodellt az Azure Analysis Servicesben. Emellett a Microsoft Power BI-ban vagy Microsoft Excelben elérhető modellezési és vizualizációs technológiákkal önkiszolgáló üzletiintelligencia-megoldásokat is támogathatnak. Az elemzés és jelentéskészítés az adatszakértők vagy adatelemzők általi végzett interaktív adatfeltárással is végrehajtható. Az ilyen forgatókönyvekhez számos Azure-szolgáltatás támogat analitikus notebookokat (pl. Jupyter), így a felhasználók felhasználják a Python vagy az R terén már megszerzett tudásukat. Nagy méretű adatfeltárás esetén használhatja a Microsoft R Servert önállóan vagy a Sparkkal együtt.

* **Vezénylés**. A legtöbb big data-megoldás munkafolyamatokba foglalt, ismétlődő adatfeldolgozási műveletekből áll, amelyek átalakítják a forrásadatokat, adatokat helyeznek át több forrás és fogadó között, betöltik a feldolgozott adatokat egy analitikus adattárba, vagy továbbítják az eredményeket egyenesen egy jelentésbe vagy irányítópultba. A munkafolyamatok automatizálhatók egy vezénylési technológia (pl. Azure Data Factory vagy Apache Oozie és Sqoop) használatával.

## <a name="lambda-architecture"></a>Lambda architektúra

Nagy adathalmazok használata esetén hosszú időbe telhet az ügyfél által igényelt lekérdezések futtatása. Ezek a lekérdezések nem futtathatók valós időben, és gyakran olyan algoritmusok használatát igénylik, mint a [MapReduce](https://en.wikipedia.org/wiki/MapReduce), amely párhuzamosan fut a teljes adathalmazon. A rendszer a nyers adatoktól elkülönítve tárolja, az eredményeket, és lekérdezésekhez használja fel őket.

A módszer egyik hátránya, hogy késést eredményez – ha a feldolgozás órákat vesz igénybe, akkor a lekérdezés több órás adatokat ad eredményül. Az ideális megoldás a valós idejű, talán kissé pontatlanabb eredmények beszerzése, amelyeket aztán a kötegelt elemzés eredményeivel együtt lehet használni.

Az először Nathan Marz által kidolgozott **lambda architektúra** két adatfolyam létrehozásával kínál megoldást az ilyen problémákra. A rendszerbe érkező összes adat ezen a két útvonalon halad végig:

* A **kötegelt réteg** (ritka elérésű útvonal) nyers formában tárolja az összes beérkező adatot, és kötegelt feldolgozást végez rajta. Ennek eredményét **kötegelt nézetben** tárolja a rendszer.

* A **gyors réteg** (gyakori elérésű útvonal) valós időben elemzi az adatokat. A réteg késése minimális, ám az eredmények kevésbé pontosak.

A kötegelt réteg a **kiszolgálórétegbe** továbbítja az eredményeket, amely indexeli a kötegelt nézetet a hatékonyabb lekérdezés érdekében. A gyors réteg növekményes frissítésekkel frissíti a kiszolgálóréteget a legújabb adatok alapján.

![Lambda architektúra ábrája](./images/lambda.png)

A gyakori elérésű útvonalon keresztül érkező adatfolyamot korlátozzák a gyors réteg késési követelményei, hogy a lehető leggyorsabban fel lehessen dolgozni. Ez azt jelenti, hogy az adatok a lehető leggyorsabban készen állnak, azonban kevésbé pontosak. Vegyünk például egy IoT-forgatókönyvet, amelyben sok hőmérséklet-érzékelő küld telemetriai adatokat. A gyors réteg egy változó időablakban beérkező adatokat dolgoz fel. 

A ritka elérésű útvonalon végighaladó adatoknak azonban nem kell megfelelniük ugyanazoknak az alacsony késésre vonatkozó követelményeknek. Ez nagy pontosságú számítást biztosít nagy adathalmazokon, ami nagyon időigényes lehet. 

A gyakori és a ritka elérésű útvonalak idővel az analitikai ügyfélalkalmazásban találkoznak. Ha az ügyfélnek aktuális, de potenciálisan kevésbé pontos adatokat kell megjelenítenie, a gyakori elérésű útvonal eredményeit kéri le. Ellenkező esetben a ritka elérésű útvonalról érkező eredményeket választja, amelyek kevésbé aktuálisak, de pontosabbak. Másként fogalmazva a gyakori elérésű útvonal viszonylag kis időtartam adataival rendelkezik, de ezek az eredmények később frissíthetők a ritka elérésű útvonal pontosabb adataival.

A kötegelt rétegben tárolt nyers adatok nem módosíthatók. A rendszer a beérkező adatokat hozzáfűzi a meglévő adatokhoz, és sosem írja felül a korábbi adatokat. Egy adott adat értékének változása új, időbélyeggel ellátott eseményrekordként jelenik meg. Ez lehetővé teszi az újraszámítást a begyűjtött adatok bármelyik időpontban aktuális értékével. Nagyon fontos, hogy a kötegelt nézet újraszámítható legyen a nyers adatokból, mert így hozhatók létre új nézetek, ahogy fejlődik a rendszer. 

## <a name="kappa-architecture"></a>Kappa architektúra

A lambda architektúra hátránya az összetettség. A feldolgozó logika két helyen jelenik meg &mdash; a ritka és a gyakori elérésű útvonalon&mdash;, és különböző keretrendszereket használ. Ez a számítási logika megkétszerezését jelenti, és a két útvonal architektúrájának összetett kezelését igényli.

A **kappa architektúrát** Jay Kreps dolgozta ki a lambda architektúra alternatívájaként. Az alapvető céljai ugyanazok, mint a lambda architektúrának, de van egy fontos különbség: minden adat egyetlen útvonalon fut végig egy streamfeldolgozó rendszer használatával. 

![Kappa architektúra ábrája](./images/kappa.png)

Némileg hasonlít a lambda architektúra kötegelt rétegéhez, mivel az eseményadatok nem módosíthatók, és a rendszer minden adatot begyűjt, nem csak egy részüket. Az adatok eseménystreamként töltődnek be egy elosztott és hibatűrő, egységes naplóba. A rendszer sorba rendezi az eseményeket, és csak új esemény hozzáfűzésekor módosítja az aktuális állapotukat. A lambda architektúra gyors rétegéhez hasonlóan az események feldolgozása a bemeneti streamben, a megőrzés pedig valós idejű nézetként történik. 

Ha a teljes adathalmazon kell újraszámítást végezni (ami a lambda architektúra kötegelt rétegének felel meg), csak újra le kell játszani a streamet, általában párhuzamos módon, hogy a számítás időben elkészüljön.

## <a name="internet-of-things-iot"></a>Eszközök internetes hálózata (IoT)

Az eszközök internetes hálózata (IoT) gyakorlatilag minden eszközt magában foglal, amely rendelkezik internetkapcsolattal. Ebbe beletartoznak a számítógépek, az okostelefonok, az okosórák, az intelligens termosztátok, hűtőszekrények, az internetkapcsolattal rendelkező gépkocsik, a szívritmust ellenőrző implantátumok és minden más, ami internetkapcsolattal rendelkezik, és adatokat fogad vagy küld. Az ilyen kapcsolattal rendelkező eszközök száma és az általuk gyűjtött adatok mennyisége napról napra növekszik. Ezek az adatok gyakran rendkívül korlátozott, néha nagy késésű környezetekből származnak. Más esetekben az adatokat alacsony késésű környezetből küldi több ezer vagy több millió eszköz, ami gyors adatbetöltést és -feldolgozást igényel. Ezen korlátozások és egyedi követelmények kezelése megfelelő tervezést tesz szükségessé.

Az eseményvezérelt architektúrák központi szerepet játszanak az IoT-megoldásokban. Az alábbi ábrán egy Iot-megoldás lehetséges logikai architektúrája látható. Az ábra az architektúra eseménystreamelési összetevőit hangsúlyozza ki.

![IoT-architektúra](../../guide/architecture-styles/images/iot.png)

A **felhőátjáró** a felhő határán olvassa be az eszközeseményeket egy megbízható, alacsony késésű üzenetkezelési rendszert használva.

Az eszközök közvetlenül a felhőátjárónak vagy egy **helyi átjárón** keresztül küldhetik el az eseményeket. A helyi átjáró egy speciális, általában az eszközökkel egy helyen található eszköz vagy szoftver, amely fogadja az eseményeket, majd továbbítja őket a felhőátjárónak. A helyi átjáró a nyers eszközesemények előfeldolgozására is képes, olyan feladatok végrehajtásával, mint a szűrés, az összesítés vagy a protokollátalakítás.

A beolvasást követően az események egy vagy több **streamfeldolgozón** haladnak át, amelyek továbbíthatják az adatokat (például egy tárolóba), vagy elemzést és más feldolgozási műveleteket végezhetnek.

Az alábbiakban a feldolgozás néhány gyakori típusát ismertetjük. (A felsorolás semmiképpen sem teljes.)

- Eseményadatok írása offline tárolóba archiválás vagy kötegelt elemzés céljából.

- Működő elérési út elemzése, vagyis az eseménystream (közel) valós idejű elemzése a rendellenességek észlelése, adott időtartamokra jellemző minták felismerése vagy riasztások aktiválása céljából, ha egy adott helyzet áll elő a streamben. 

- Az eszközöktől származó nem telemetriaüzenetek különleges típusainak, például az értesítéseknek és a riasztásoknak a kezelése. 

- Gépi tanulás.

A szürke dobozok az IoT-rendszer azon összetevőit jelölik, amelyek nem kapcsolódnak közvetlenül az eseménystreameléshez, a teljesség igénye miatt azonban az ábra részét képezik.

- Az **eszközjegyzék** az üzembe helyezett eszközök adatbázisa, amely az eszközök azonosítóját és rendszerint az eszközök metaadatait, például a helyüket tartalmazza.

- Az **üzembe helyezési API** egy általános külső felület az új eszközök üzembe helyezéséhez és regisztrálásához.

- Egyes IoT-megoldások lehetővé teszik **parancs- és vezérlő üzenetek** küldését az eszközöknek.

Kapcsolódó Azure-szolgáltatások:

- [Azure IoT Hub](https://azure.microsoft.com/services/iot-hub/)
- [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/)
- [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/)  

További információ az IoT Azure-ban történő használatáról: [Azure IoT-referenciaarchitektúra](https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/).


