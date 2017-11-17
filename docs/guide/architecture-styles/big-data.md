---
title: "Big Data típusú adatok architektúra stílus"
description: "Előnyeit, kihívást és ajánlott eljárások a Big Data-architektúrák ismerteti az Azure-on"
author: MikeWasson
ms.openlocfilehash: 4e8b58d5fa0f6a441d70e05ec7d6a0e668712563
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="big-data-architecture-style"></a>Big Data típusú adatok architektúra stílus

A big Data típusú adatok architektúra az adatfeldolgozást, feldolgozása és elemzések adatok, amelyek túl nagy vagy túl bonyolult, a hagyományos adatbázis-rendszerek kezelésére terveztek.

![](./images/big-data-logical.svg)

 Big data-megoldások általában tartalmaz, amely a következő típusú alkalmazások és szolgáltatások közül:

- A big Data típusú adatok források aktívan kötegfeldolgozási.
- A big data mozgó valós idejű feldolgozással.
- A big Data típusú adatok interaktív áttekintését.
- A prediktív elemzés és a gépi tanulás.

A big data-architektúrák vagy azok egy részét az alábbi összetevőket tartalmazza:

- **Adatforrások**: egy vagy több adatforrást kezdődnie big data-megoldások. Példák erre vonatkozóan:

    - Alkalmazásadatok tárolja, például a relációs adatbázisok.
    - Statikus fájlok, alkalmazások, például a web server naplófájlok által előállított.
    - Valós idejű adatforrások, például az IoT-eszközök.

- **Adattárolás**: adatok feldolgozási műveletek köteg általában tárolása egy elosztott fájlrendszer, amely jelentős mennyiségű különböző formátumban nagy fájlok tárolására képes. Az ilyen tároló gyakran nevezik egy *a data lake*. Ez a tároló végrehajtási lehet például az Azure Data Lake Store vagy a blob-tárolóban az Azure Storage. 

- **Kötegfeldolgozási**: mivel nagy adatkészletek, gyakran big Data típusú adatok megoldást kell feldolgozni szűrő összesített, hosszan futó kötegelt feladatok segítségével az adatfájlok és ellenkező esetben az adatok előkészítése az elemzés. Általában ezeket a feladatokat tartalmaz, amely forrásfájlok olvasása, őket, és a kimeneti új fájlok írása. A választható lehetőségek az Azure Data Lake Analytics U-SQL-feladatok futtatása, a Hive, Pig vagy egyéni térkép vagy csökkentse a feladatok egy HDInsight Hadoop-fürt használatával, vagy a HDInsight Spark-fürtben lévő Java, Scala vagy Python programokkal.

- **Valós idejű üzenet adatfeldolgozást**: Ha a megoldás a valós idejű eseményforrást tartalmaz, az architektúra tartalmaznia kell oly módon, majd adatfolyam feldolgozásra valós idejű üzenetek tárolhatja. Ez lehet egy egyszerű adattár, ahol a bejövő üzenetek eldobott datagramok feldolgozás céljából egy mappába. Sok megoldás azonban adatfeldolgozást üzenettároló puffer üzenetek, illetve kibővített feldolgozás, a megbízható és egyéb üzenet Üzenetsor-kezelés szemantikáját támogatásához szükséges. A választható lehetőségek az Azure Event Hubs, Azure IoT hub és Kafka.

- **Az adatfolyam feldolgozása**: valós idejű üzenetek rögzítésével, miután a megoldás kell dolgozza fel őket szűrés, összesítése és egyéb előkészítése az adatok elemzése. A feldolgozott adatfolyamok majd írni kimeneti fogadóként. Az Azure Stream Analytics felügyelt streamfeldolgozó termékbevételezésekor fut, amely unbounded adatfolyamok működik az SQL-lekérdezések szolgáltatáson biztosít. Nyílt forráskódú Apache adatfolyam hasonló technológiáknak köszönhetően a Storm és a Spark Streaming a HDInsight-fürtök is használható.

- **Analitikai adatokat tároló**: sok big data-megoldások adatok előkészítése az elemzés, és a feldolgozott adatok szolgáljanak strukturált formátuma nem kérdezhetők le analitikai eszközeivel. Ezeket a lekérdezéseket kiszolgálásához használt analitikus adattároló Kimball stílusú relációs adatraktár, lehet, a hagyományos üzleti intelligenciával megoldások látható módon. Azt is megteheti az adatok sikerült jelenik meg a kis késleltetésű NoSQL technológia, például a HBase, vagy egy interaktív Hive-adatbázis, amely a metaadatokat absztrakciós keresztül az elosztott adattár-adatfájlok keresztül. Az SQL Data Warehouse nagyméretű, felhőalapú adatraktárra vonatkozó felügyelt szolgáltatást nyújt. HDInsight támogatja, interaktív struktúra, a HBase és a Spark SQL, amely vonatkozó adatok elemzési célú kiszolgálásához is használható.

- **Elemzési és jelentéskészítési**: a legtöbb big data-megoldások célja az, hogy az adatok elemzési és jelentéskészítési betekintést. Építve a felhasználók számára az adatok elemzése, az architektúra a réteg, például a többdimenziós OLAP-kocka vagy az Azure Analysis Services táblázatos adatmodell modellezési adatokat is tartalmazhat. Az önkiszolgáló üzleti Intelligencia, a Microsoft Power bi-ban vagy a Microsoft Excel modellezési és a képi megjelenítés technológiái is használható lehet. Elemzési és jelentéskészítési is számos adatok interaktív áttekintését adatszakértőkön vagy adatelemzők formájában. Ezek a forgatókönyvek számos Azure-szolgáltatások támogatási analitikai notebookok, például a Jupyter, ezeket a felhasználókat, hogy kihasználja a meglévő ismeretei r és Python engedélyezése A nagyméretű adatok feltárása, használhatja a Microsoft R Server, vagy önálló vagy Spark.

- **Vezénylési**: a legtöbb big data-megoldások állnak ismételt adatfeldolgozási műveletek, a munkafolyamatokban, forrásadatok átalakító, beágyazott adatok áthelyezése több források és mosdók között, a feldolgozott adatok betöltése az analitikus adattár vagy küldje le az eredményeket rögtön egy jelentést és az irányítópultot. Ezek a munkafolyamatok automatizálását, használhatja az orchestration technológia ilyen Azure Data Factory vagy az Apache Oozie és a Sqoop.

Azure tartalmaz számos olyan szolgáltatás, amely a big Data típusú adatok architektúrák használható. Ezek nagyjából két kategóriába sorolhatók:

- A felügyelt szolgáltatásokat, például Azure Data Lake Store, az Azure Data Lake Analytics, az Azure Data Warehouse, az Azure Stream Analytics, a Azure Event Hubs, a Azure IoT Hub és a Azure Data Factory.
- Nyílt forráskódú technológiák többek között a HDFS, HBase, Hive, Pig, Spark, Storm, Oozie, Sqoop és Kafka Apache Hadoop platformtípus alapján. Ezek a technológiák érhetők el az Azure-on Azure HDInsight szolgáltatási.

Ezek a beállítások nincsenek kölcsönösen kizárja egymást, és sok megoldás nyílt forráskódú technológiák együttesen Azure-szolgáltatásokhoz.

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

Vegye figyelembe a architektúra stílus kell:

- A hagyományos adatbázis túl nagy kötetek tároló és a folyamat adatait.
- Az elemzési és jelentéskészítési strukturálatlan adatok átalakítása.
- Rögzítési, a folyamat, és unbounded adatstreamek valós időben, vagy a kis késleltetésű elemzése.
- Az Azure gépi tanulás vagy a Microsoft kognitív szolgáltatásokat használja.

## <a name="benefits"></a>Előnyök

- **Technológia lehetőségek**. Ön szabadon kombinálhatók Azure services és a HDInsight-fürtök, a meglévő ismeretei vagy informatikai befektetések nagybetűssé Apache technológiák felügyelt.
- **Teljesítmény párhuzamossági keresztül**. Big data-megoldások előnyeit párhuzamossági, nagy teljesítményű megoldások, amelyek a méretezhető, nagy mennyiségű adatot engedélyezése.
- **Rugalmasan méretezhető**. A big Data típusú adatok architektúra-összetevők mindegyiken kibővített kiépítés, így módosíthatja a kis és nagy munkaterhelések megoldást, és csak a használt erőforrások után kell fizetnie.
- **Meglévő megoldásaival való együttműködési**. Az összetevők a big Data típusú adatok architektúra IoT feldolgozás és a vállalati BI megoldások, amely lehetővé teszi egy integrált megoldás részeként teremtsen data-számítási feladatok is használhatók.

## <a name="challenges"></a>Kihívásai

- **Összetettsége**. Big data-megoldások számos összetevők kezeléséhez több adatforrásokból adatfeldolgozást rendkívül összetett feladat lehet. Az létrehozása, tesztelése és hibaelhárítása a big Data típusú adatok folyamatok kihívást jelenthet. Ezenkívül előfordulhat számos konfigurációs beállításait meg több rendszerben, a teljesítmény optimalizálása érdekében használni kell.
- **Skillset**. Sok big Data típusú adatok technológiák magas specializált, és keretrendszerek és a nyelveket, amelyek nem kifejezetten a általánosabb alkalmazási architektúrákban használja. Másrészt a big Data típusú adatok technológiák vannak fejlődnek új API-k, amelyek létrehozása több kiszolgálón létrehozott nyelveket. Például az Azure Data Lake Analytics U-SQL nyelv Transact-SQL és C# alapul. Hasonlóképpen SQL-alapú API-k érhetők el a Hive, a HBase és a Spark.
- **Technológia lejárat**. A big Data típusú adatok használt technológiák folyamatosan változó. Alapvető Hadoop technológiák ilyen struktúra és sertésfelmérés rögzítették, amíg feltörekvő technológiákkal, például a Spark bevezetni a jelentős változtatások és az egyes új kiadások fejlesztései. Felügyelt szolgáltatásokat, például az Azure Data Lake Analytics és az Azure Data Factory viszonylag fiatal, összeveti az egyéb Azure-szolgáltatásokkal, és valószínűleg változik, adott idő alatt.
- **Biztonság**. Big data-megoldások általában támaszkodhat központosított data lake-ben az összes statikus adatainak tárolásához. Ezek az adatok hozzáférésének biztonságossá tétele kihívást, lehet, különösen akkor, ha az adatok kell okozhatnak, és több alkalmazás és rendszerek által használt.

## <a name="best-practices"></a>Ajánlott eljárások

- **Használja ki a párhuzamos végrehajtás**. A nagy adatfeldolgozási technológiák okozott terhelés elosztásához több feldolgozóegység között. Ehhez az szükséges, hogy statikus adatfájlokat létrehozni, és feloszthatók formátumban tárolja. Az elosztott fájlrendszerek például a HDFS optimalizálhatja olvasási és írási teljesítményt, és több fürtcsomóponton párhuzamosan, ami csökkenti az általános feladat alkalommal végzi el a tényleges feldolgozását.

- **Adatok particionálása**. Kötegfeldolgozási általában akkor fordul elő, egy ismétlődő ütemezés szerint &mdash; például weekly vagy monthly. Partíció adatok és fájlok adatszerkezetek, például a táblákat, amelyek megfelelnek a feldolgozási ütemtervet historikus időszakok alapján. Amely egyszerűbbé teszi a adatfeldolgozást és a feladatütemezés, és megkönnyíti a hibák elhárítása. Is a Hive használt táblák particionálása, U-SQL vagy az SQL-lekérdezések jelentősen növelheti a lekérdezések teljesítményét.

- **Olvassa el séma szemantikáját alkalmazása**. Használatával a data lake lehetővé teszi több formátumú fájlok tárolási kombinálását, hogy strukturált, félig strukturált vagy strukturálatlan. Használjon *olvassa el séma* szemantikáját, amely egy séma projekt telepítését az adatok, az adatok feldolgozása, nem amikor az adatok tárolása közben. Ez rugalmasságot alkot a megoldásba történő, és megakadályozza, hogy a szűk keresztmetszetek adatfeldolgozást okozta adatérvényesítést és típus ellenőrzése során.

- **Folyamat adatok helyben**. Hagyományos BI-megoldásai gyakran használnak egy kinyerési, átalakítási és betöltési (ETL) folyamat áthelyezni az adatokat az adatraktárban. Nagyobb kötetek adatokat, és nagyobb többféle formátumúak a big data-megoldások általában ETL, változatait, kinyerési, átalakítási például használja, és (TEL) történik. Ezt a módszert használja az adatok végrehajtva az elosztott adattár, átalakítása a szükséges struktúra az átalakított adatok áthelyezése az analitikus adatok importálása előtt.

- **Kihasználtsági és idő költségek egyenleg**. A kötegelt feldolgozásra, fontos, hogy két tényezőket kell figyelembe venni: a számítási csomópontok egység költségét és perc költségeinek azokat a csomópontokat, a feladat befejezéséhez. Például egy kötegelt négy fürtcsomópontra nyolc óráig is eltarthat. Azonban, előfordulhat, hogy kapcsolja, hogy a feladat összes négy csomópont használ, csak az első két órában, és azt követően, csak két csomópontra szükség. Ebben az esetben a teljes feladat két csomópont nőne a feladatok teljes idő, de nőhet nem, így a teljes költség lenne kisebb. Az egyes üzleti forgatókönyvek kezeléséhez a fürterőforrások magasabb költségeinek előnyös lehet hosszabb feldolgozási időt.

- **Külön fürterőforrások**. A HDInsight-fürtök való telepítésekor, akkor lesz általában jobb teljesítmény érdekében úgy külön fürterőforrások az egyes alkalmazások és szolgáltatások. Például bár a Spark-fürtök struktúra, ha végre kell hajtania a széles körű feldolgozását a Hive és a Spark, érdemes lehet külön dedikált Spark és a Hadoop fürtök telepítése. Ehhez hasonlóan használatakor a HBase és a Storm kis késleltetésű adatfolyam feldolgozása és a Hive kötegelt feldolgozásra, érdemes lehet külön fürtök a Storm, a HBase és a Hadoop.

- **Adatfeldolgozást levezényelni**. Bizonyos esetekben a meglévő üzleti alkalmazások írhat adatforrás adatfájljainak kötegfeldolgozási közvetlenül az Azure storage-blob tárolók, ahol azok képes használni a HDInsight vagy az Azure Data Lake Analytics. Azonban gyakran kell levezényelni a adatfeldolgozást helyszíni vagy külső forrásból származó adatok a data lake-be. Ennek érdekében a központilag kezelhető és kiszámítható módon használhat egy vezénylési munkafolyamat vagy folyamat, például az Azure Data Factory vagy Oozie, által támogatott.

- **Megtisztítás a bizalmas adatok korai**. Az adatok adatfeldolgozást munkafolyamat elkerülése érdekében a data Lake tárolja, a folyamat korai szakaszaiban kell megtisztítás a bizalmas adatokat.
