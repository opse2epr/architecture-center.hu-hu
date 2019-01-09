---
title: Kötegelt feldolgozás
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 50e50ae121fda7ceb9dd298b8a072bd7cc4053d9
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114181"
---
# <a name="batch-processing"></a>Kötegelt feldolgozás

Big Data típusú adatok gyakran előfordul az inaktív adatok kötegelt feldolgozására. Ebben a forgatókönyvben a forrásadatok betöltött adatok tárolására, az alkalmazás saját maga vagy egy vezénylési munkafolyamat. Az adatok feldolgozása majd helyben egy párhuzamos feladatot, amely a vezénylési munkafolyamat által is kezdeményezhető. A feldolgozás tartalmazhat több iteratív lépést előtt az átalakított eredmények töltődnek be egy analitikus adattárba, amely lekérdezhetők, elemzési és jelentéskészítési összetevőket.

Például a webkiszolgáló naplóinak előfordulhat, hogy egy mappába másolta és feldolgozása történhet az majd éjjelente napi jelentések webes tevékenység.

![A kötegelt feldolgozási folyamat ábrája](./images/batch-pipeline.png)

## <a name="when-to-use-this-solution"></a>Ez a megoldás használata

Kötegelt feldolgozás esetben egyszerű adatátalakítások egy teljes (kinyerési, átalakítási, betöltési) ETL-folyamat különféle használatban van. A big data-környezetben a kötegelt feldolgozás előfordulhat, hogy hatnak a rendkívül nagy méretű adatkészleteket, ahol a számítási jelentős időt vesz igénybe. (Lásd a [Lambda architektúra](../big-data/index.md#lambda-architecture).) Kötegelt feldolgozás általában vezet, további interaktív feltárása, a modellezési használatra kész adatokat biztosít a machine Learning, vagy az adatokat ír egy adattár, amely elemzési és vizualizációs van optimalizálva.

Egy példa a kötegelt feldolgozás olyan formátumra sematikus strukturált és strukturálatlan, amely készen áll a további lekérdezési formálja át a strukturálatlan, részben strukturált CSV vagy JSON-fájlok nagy készletét. Általában az adatok lesz konvertálva (például a CSV-) támogatunk használt nyers formátumokból bináris formátum, amelyek több nagy teljesítményű lekérdezéshez, mivel tárolja az adatokat oszlopos formátumot, és gyakran biztosítanak az indexek és az adatok beágyazott statisztikája.

## <a name="challenges"></a>Problémák

- **Adatok formázása és a kódolási**. A legnehezebb problémák hibakeresése fordulhat elő, ha a fájlok az váratlan formátumúak vagy kódolást. Például forrásfájlok előfordulhat, hogy használja az UTF-16 és UTF-8 kódolást, tartalmaz váratlan elválasztó karaktereket (szóköz vagy lap), vagy váratlan karaktereket. Egy másik gyakori példa szöveges mezők, amelyek tartalmazzák a lapok, szóközök és vesszővel válassza el egymástól, amelyeket az elválasztó karakterként értelmez. Adatok betöltése és elemzési logika kellően rugalmas ahhoz, hogy ezek a problémák észlelésével és kell lennie.

- **Replikálásával segít a vállalatnak időszeletek**. Gyakran a forrásadatok kerül a mappahierarchiában, amely tükrözi a feldolgozási windows, év, hónap, nap, óra és így tovább szerint vannak rendezve. Bizonyos esetekben adatokat az késői lehet, hogy érkezik. Tegyük fel, hogy egy webkiszolgáló meghibásodik, és a naplókat. március 7. végül nem a feldolgozáshoz. március 9. amíg a mappát. Vannak, csak figyelmen kívül, mert azok már túl késő? Az alárendelt feldolgozási logika out soron kívüli rekordok képes kezelni?

## <a name="architecture"></a>Architektúra

A kötegelt feldolgozási architektúra rendelkezik a következő logikai összetevőit, a fenti ábrán látható.

- **Adattárolás**. Általában egy elosztott fájltároló, amely számos formátumú és nagy nagy mennyiségű vagy a. Általános az ilyen típusú tároló gyakran nevezik data lake.

- **Kötegelt feldolgozás**. A nagy mennyiségű jellegét a big Data típusú adatok gyakran azt jelenti, hogy megoldásokat kell feldolgozniuk az adatfájlokat, hosszan futó kötegelt feladatok használatával történő szűréséhez, összesítéséhez és ellenkező esetben az adatok elemzésre való előkészítését. Ezek a feladatok általában magukban foglalják az adatforrások beolvasását, feldolgozását, valamint a kimenet új fájlokba történő írását.

- **Analitikus adattár**. Számos big data-megoldás előkészíti az adatokat az elemzésre, és ezután bocsátja a feldolgozott adatokat, hogy lekérdezhetők legyenek elemzőeszközökkel strukturált formátumban lettek kialakítva.

- **Elemzés és jelentéskészítés**. A legtöbb big data-megoldás célja az, hogy elemzéssel és jelentéskészítéssel betekintést nyújtson az adatokba.

- **Vezénylés**. A kötegelt feldolgozásra, általában néhány vezénylési szükség áttelepítése vagy az adatok másolása az adatok tárolóba, kötegelt feldolgozás, analitikai adattár, és a reporting rétegek.

## <a name="technology-choices"></a>Technológiai lehetőségek

A következő technológiákat használata akkor javasolt lehetőségek a kötegelt feldolgozási megoldásokat az Azure-ban.

### <a name="data-storage"></a>Adattárolás

- **Az Azure Storage-Blobtárolók**. Számos meglévő Azure üzleti folyamat már használatba az Azure blob storage, ami megfelelő választás az olyan big data store-ban.
- **Az Azure Data Lake Store**. Az Azure Data Lake Store bármilyen méretű fájlt, és a széles körű biztonsági beállítások, az gyakorlatilag korlátlan tárhelyet kínál, így a megfelelő választás az olyan rendkívül nagy méretű big data-megoldás, amely egy központi tárolóban igényel a heterogén formátumú adatokat.

További információkért lásd: [adattárolás](../technology-choices/data-storage.md).

<!-- markdownlint-disable MD024 -->

### <a name="batch-processing"></a>Kötegelt feldolgozás

<!-- markdownlint-enable MD024 -->

- **U-SQL**. U-SQL az Azure Data Lake Analytics által használt nyelv Lekérdezésfeldolgozás. Eljárási bővíthetőségét kombinálja az SQL deklaratív jellegét a C#, és kihasználja a párhuzamosság engedélyezéséhez a hatékony feldolgozás az adatok nagy mennyiségben.
- **Hive-**. Hive egy SQL-szerű nyelv által támogatott legtöbb Hadoop disztribúciók, például a HDInsight. Bármely HDFS-kompatibilis többek között az Azure blob storage és az Azure Data Lake Store áruházból származó adatok feldolgozására használható.
- **A Pig**. A Pig egy deklaratív big data típusú adatfeldolgozást számos Hadoop disztribúciók, például a HDInsight-ben használt nyelvhez. Ez különösen hasznos, amely strukturálatlan vagy részben strukturált adatok feldolgozására.
- **A Spark**. A Spark-motort támogatja a kötegelt feldolgozás programot, számos nyelvet – például a Java, Scala és Python nyelven írt. Spark párhuzamosan dolgozza fel az adatokat, egy felosztott architektúrában több munkavégző csomóponton használ.

További információkért lásd: [kötegelt feldolgozás](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Analitikai adattár

- **Az SQL Data Warehouse**. Az Azure SQL Data Warehouse az SQL Server adatbázis-technológiákon alapuló, és nagy léptékű adattárház számítási feladatok támogatására optimalizált felügyelt szolgáltatás.
- **A Spark SQL**. Spark SQL API-k, Spark, amely támogatja a dataframes és táblákat, amelyek az SQL-szintaxissal lekérdezhetők épülő.
- **A HBase**. A HBase, kis késésű NoSQL-alapú tárolót, amely a strukturált és részben strukturált adatok lekérdezéséhez egy nagy teljesítményű és rugalmas lehetőséget kínál.
- **Hive-**. Azonfelül hasznos a kötegelt feldolgozáshoz, Hive kínál egy adatbázis-architektúrát, amely egy tipikus relációsadatbázis-kezelő rendszer tárolókéhoz hasonló. Fejlesztések azzal kapcsolatban, a Hive-lekérdezés teljesítményének innovációkat keresztül, mint a Tez-motor és Stinger kezdeményezési középérték, hogy Hive-táblákat is hatékonyan használható forrásként az elemzési lekérdezéseknél, bizonyos esetekben.

További információkért lásd: [analitikus adattárak](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Elemzések és jelentéskészítés

- **Az Azure Analysis Services**. Számos big data-megoldás a hagyományos vállalati üzleti intelligencia architektúrák emulációjához fel (gyakran nevezik adatkocka) egy olyan központi online elemzésfeldolgozási (OLAP) modell a jelentéseket, irányítópultokat, és interaktív "részletekbe menően vizsgálhatja az" elemzés alapján. Az Azure Analysis Services támogatja a táblázatos modellek erre.
- **Power BI**. A Power BI lehetővé teszi, hogy az adatelemzők interaktív vizualizációkat adatmodellek az OLAP-modell vagy közvetlenül egy analitikai adattár létrehozása.
- **A Microsoft Excel**. A Microsoft Excel a leggyakrabban használt alkalmazások a világ egyik, és számos olyan adatok elemzési és vizualizációs funkciókat kínál. Az adatelemzők az Excel dokumentum adatmodellek analitikai adatokat hozhat létre, vagy adatokat lekérni az OLAP-adatmodelleket, interaktív kimutatásokat és diagramokat.

További információkért lásd: [elemzések és jelentéskészítés](../technology-choices/analysis-visualizations-reporting.md).

### <a name="orchestration"></a>Adat-előkészítés

- **Az Azure Data Factory**. Az Azure Data Factory-folyamatok ismétlődő historikus windows az ütemezett tevékenységek sorozatát meghatározására használható. Ezek a tevékenységek kezdeményezheti a másolási műveletek, valamint igény szerinti HDInsight-fürtök; Hive, Pig, MapReduce vagy Spark-feladatok Az Azure dátum Lake Analyticsben; U-SQL-feladatok és az Azure SQL Data Warehouse vagy az Azure SQL Database tárolt eljárásokat.
- **Az Oozie** és **Sqoop**. Oozie szolgáló feladat automatizálási motor az Apache Hadoop-ökoszisztéma és kezdeményezése a másolási műveletek, valamint a Hive, Pig és MapReduce-feladatok adatok feldolgozását és másolhat adatokat a HDFS- és SQL-adatbázisok közötti Sqoop feladatok is használható.

További információkért lásd: [vezénylési folyamat](../technology-choices/pipeline-orchestration-data-movement.md)
