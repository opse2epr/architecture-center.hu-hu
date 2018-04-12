---
title: Kötegelt feldolgozás
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d6843bf4e20c3eb26e61cfa09300ad533e969c2e
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
---
# <a name="batch-processing"></a>Kötegelt feldolgozás

Big Data típusú adatok forgatókönyve az inaktív adatok kötegelt feldolgozása. Ebben a forgatókönyvben az adatok tárolására, vagy az alkalmazás saját magát vagy egy vezénylési munkafolyamat betöltődik. Az adatfeldolgozás majd egy parallelized feladat, és a vezénylési munkafolyamat is kezdeményezheti helyben. A feldolgozási tartalmazhatják több iteratív lépés előtt az átalakítani kívánt eredmények be vannak töltve az analitikus adatok tárolóba, amely lekérdezhetők, elemzés és a jelentéskészítési összetevőket.

Például a webkiszolgáló naplói mappába másolt előfordulhat, hogy, és majd feldolgozásra éjszaka a webkiszolgáló tevékenységének napi jelentések készítése.

![](./images/batch-pipeline.png)

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Kötegfeldolgozási lehetőségeket, egyszerű adatátalakítást jóval összetettebb ETL (extract-átalakítási-betöltés)-feldolgozási folyamat különböző használatban van. A big Data típusú adatok környezetben kötegfeldolgozási előfordulhat, hogy hatnak a nagyon nagy méretű adatkészletekhez, ahol a számítási jelentős időt vesz igénybe. (Lásd például: [Lambda architektúra](../big-data/index.md#lambda-architecture).) Kötegfeldolgozási általában további interaktív adatkutatási vezet, a modellezési használatra kész adatokat biztosít a gépi tanulás, vagy a adatokat ír az adattároló, amely elemzés és -megjelenítésre van optimalizálva.

Egy példa a köteges feldolgozás van átalakítása egyszerű, félig strukturált CSV- vagy JSON-fájlok számos olyan formátumra sematizált és strukturált, amely készen áll a további lekérdezése. Általában az adatok alakul a nyers formátumok adatfeldolgozást (például a fürt megosztott kötetei szolgáltatás) használatos bináris formátum, amelyek több performant lekérdezése, mert az adat tárolása oszlopos formátumot, és gyakran biztosítanak az indexek és az adatok beágyazott statisztikája.

## <a name="challenges"></a>Problémák

- **Adatok formázása és kódolás**. A legbonyolultabb problémák hibakeresési fordulhat elő, ha a fájlok használata érvénytelen formátumú vagy kódolást. Például forrásfájlok előfordulhat, hogy az UTF-16 és UTF-8 kódolását, vagy váratlan elválasztó karaktert (hely és a lapon) tartalmaz, vagy használni nem várt karakter. Egy másik közös: tartalmazó lapokat, szóközök és vesszővel válassza el egymástól, amelyeket az elválasztó karakterként szöveg mezők. Adatok betöltése és értelmezése logika vajon elég rugalmas az észlelésére, és ezek a problémák kezelésére kell lennie.

- **Időszeletek megvalósításában.** Gyakran a forrásadatok el van helyezve a mappahierarchiában, amely tükrözi a feldolgozási windows év, hónap, nap, óra, és így tovább szerint vannak rendezve. Bizonyos esetekben adatok előfordulhat, hogy az ügyfélszámítógépekre érkeznek késői. Tegyük fel például, hogy a webkiszolgáló nem, és a naplókat. március 7. március 9-ig feldolgozásra a mappában nem végződhet. Ezek csak figyelmen kívül hagyja, mert túl későn fontosságúak? Az alárendelt feldolgozási logikai kezelni tud a soron rekordok?

## <a name="architecture"></a>Architektúra

A kötegelt architektúrákban rendelkezik a következő logikai összetevők, a fenti ábrán is látható.

- **Adattárolás.** Általában egy elosztott szolgáltatásfájl-tároló, amely a különböző formátumokban nagy fájlok jelentős mennyiségű tára szolgálhatnak. Általános az ilyen tároló gyakran nevezik data lake. 

- **Kötegfeldolgozási.** A big Data típusú adatok nagy mennyiségű jellege gyakran azt jelenti, hogy a megoldások kell feldolgozni az adatfájlok hosszan futó kötegelt feladatok segítségével szűréséhez, összesített és ellenkező esetben az adatok előkészítése az analysis. Ezek a feladatok általában magukban foglalják az adatforrások beolvasását, feldolgozását, valamint a kimenet új fájlokba történő írását. 

- **Analitikai adatokat tároló.** Adatok előkészítése az elemzési és a feldolgozott adatok szolgáljanak strukturált formátuma nem kérdezhetők le analitikai eszközeivel sok big data-megoldások tervezték. 

- **Elemzési és jelentéskészítési.** A legtöbb big data-megoldások célja az adatok elemzési és jelentéskészítési betekintést. 

- **Vezénylési.** Kötegfeldolgozási, általában néhány vezénylési szükség, áttelepítése vagy az adatait átmásolja az adatokat tároló, a batch-feldolgozás, analitikai adatok, és jelentéskészítés rétegek.

## <a name="technology-choices"></a>Technológiai lehetőségek

A következő technológiákat lehetőségek kötegfeldolgozási megoldások Azure-ban a használata ajánlott.

### <a name="data-storage"></a>Adattárolás

- **Az Azure Storage-Blob tárolók**. Meglévő Azure üzleti folyamatok már igénybe az Azure blob-tároló, így ez nagy adattároló jó választás.
- **Az Azure Data Lake Store**. Azure Data Lake Store bármilyen méretű fájlt, és a széles körű biztonsági beállítások, az gyakorlatilag korlátlan tárterületet biztosít, így jól funkcionálnak a rendkívül nagy méretű big data-megoldások, amely egy központi tárolóban igényel a heterogén formátumú adatok.

További információkért lásd: [adattárolás](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Kötegelt feldolgozás

- **U-SQL**. U-SQL Ez a lekérdezés feldolgozása az Azure Data Lake Analytics nyelvét. SQL deklaratív természetét a C# eljárási bővítési egyesíti, és kihasználja a párhuzamos végrehajtás engedélyezése a nagy méretű adatok hatékony kezelésére.
- **Hive**. Hive a legtöbb Hadoop terjesztéseket, beleértve a HDInsight a támogatott SQL-szerű nyelven. Feldolgozott adatok a HDFS-kompatibilis áruházból, például az Azure blob storage és az Azure Data Lake Store használható.
- **A Pig**. A Pig egy deklaratív nagy adatfeldolgozási Hadoop terjesztések, beleértve a HDInsight-ben használt nyelvhez. Különösen célszerű strukturálatlan és félig strukturált adatok feldolgozására.
- **Spark**. A Spark-motor számos különböző nyelveken, többek között a Java, Scala és Python nyelven írt kötegfeldolgozási programok támogatja. Spark párhuzamosan adatfeldolgozásra történő egy felosztott architektúrában több munkavégző csomópontokhoz használ.

További információkért lásd: [kötegfeldolgozási](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Analitikai adatokat tároló

- **Az SQL Data Warehouse**. Az SQL Data Warehouse egy olyan felügyelt szolgáltatás SQL Server adatbázis technológiák alapján, és nagy méretű adatraktározás számítási feladatainál támogatása optimalizált.
- **Spark SQL**. Spark SQL az API-k, amely támogatja a létrehozása dataframes és -táblázatot, amely az SQL-szintaxis használatával lehet lekérdezni a Spark épül.
- **A HBase**. A HBase a kis késleltetésű NoSQL tárolóban, vagy félig strukturált adatok lekérdezése egy nagy teljesítményű, rugalmas lehetőséget is kínál.
- **Hive**. Hasznosak lehetnek a köteges feldolgozás alatt álló, Hive elméleti szinten hasonló egy tipikus relációs adatbázis-kezelő rendszer egy adatbázis-architektúra lehetőséget biztosít. Fejlesztések a Hive-lekérdezések teljesítményét innovációinak keresztül, például a Tez-motor és, hogy Hive táblák használható hatékonyan forrásként bizonyos esetekben elemzési lekérdezések Stinger kezdeményezésére közepét.

További információkért lásd: [analitikai adattárolókhoz](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Elemzések és jelentéskészítés

- **Az Azure Analysis Services**. Sok big data-megoldások hagyományos vállalati üzleti intelligencia architektúrák emulálni (más néven a kocka) központi online analitikus feldolgozási (OLAP) adatmodell-ot a jelentéseket, irányítópultok, és interaktív "részletekbe menően" Analysis-alapú lehet. Az Azure Analysis Services többdimenziós és táblázatos modellek az igénynek a kielégítése létrehozását támogatja.
- **Power BI**. A Power BI lehetővé teszi, hogy az adatelemzők, az adatok interaktív képi megjelenítéseket készíthet, az OLAP-modell, vagy közvetlenül az analitikus adatok áruházból adatmodellekben alapján.
- **A Microsoft Excel**. Microsoft Excel a világ a legszélesebb körben használt szoftverek alkalmazáshoz, és számos olyan adatok elemzése és a képi megjelenítés lehetőségeket kínál. Adatelemzők használható Excel dokumentum adatmodelleket építhetnek analitikai adatok áruházakból, illetve OLAP adatmodellekben interaktív kimutatások és diagramokat az adatok lekérése.

További információkért lásd: [elemzési és jelentéskészítési](../technology-choices/analysis-visualizations-reporting.md).

### <a name="orchestration"></a>Adat-előkészítés

- **Az Azure Data Factory**. Az Azure Data Factory folyamatok használható tevékenységek, ismétlődő historikus windows az ütemezett sorrendje határozza meg. Ezek a tevékenységek adatok másolási műveleteket, valamint az igény szerinti HDInsight-fürtök; Hive, Pig, MapReduce vagy Spark feladatok is kezdeményezhető. U-SQL feladatok Azure dátum Lake Analytics; és tárolt eljárások az Azure SQL Data Warehouse vagy az Azure SQL Database.
- **Oozie** és **Sqoop**. Oozie szolgáló feladat automation motor az Apache Hadoop-ökoszisztéma és adatok másolási műveleteket, valamint a Hive, Pig és MapReduce feladatokat dolgoz fel adatokat és a Sqoop feladatok, másolja az adatokat HDFS és SQL-adatbázisok között kezdeményezheti használható.

További információkért lásd: [orchestration-feldolgozási folyamat](../technology-choices/pipeline-orchestration-data-movement.md)
