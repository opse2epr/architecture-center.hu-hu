---
title: Valós idejű feldolgozás
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1499fecc20dcbf51472b92bb588b91db0fb58a7f
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295599"
---
# <a name="real-time-processing"></a>Valós idejű feldolgozás

Valós idejű feldolgozás mutatnak, amelyek rögzítve lesznek a valós idejű és a feldolgozott használatával hozzon létre valós idejű (vagy közel valós idejű) jelentések vagy az automatikus válaszok minimális késéssel foglalkozik. Például egy valós idejű forgalmat figyelő megoldás lehet, hogy használatával érzékelőktől kapott adatok nagy forgalmú kötetek észlelése. Ezek az adatok való dinamikus frissítéséhez egy térkép torlódás megjelenítéséhez, vagy automatikusan elindítja a nagy-os foglaltságnál tart útvonalakra vagy más forgalom felügyeleti rendszerekkel használható.

![](./images/real-time-pipeline.png)

Valós idejű feldolgozás számít, ha a bemeneti adatok feldolgozásához nagyon rövid késési követelményekkel rendelkező korlátlan streameken működő stream feldolgozására &mdash; ezredmásodperc vagy másodpercben mérve. A bejövő adatok általában érkezik egy strukturálatlan és részben strukturált formátumban, például a JSON-t, és ugyanazon a feldolgozási követelmények vonatkoznak, mint [kötegelt feldolgozás](./batch-processing.md), de a valós idejű használati támogatásához rövidebb válaszidejű, optimalizált rövidebb.

Feldolgozott adatok gyakran írt egy analitikai adattár, amely elemzési és vizualizációs van optimalizálva. A feldolgozott adatokat is olvasódnak közvetlenül az elemzési és jelentéskészítési réteg elemzés, üzleti intelligencia és a valós idejű irányítópult-vizualizációból.

## <a name="challenges"></a>Problémák

Valós idejű feldolgozás megoldások a big Data típusú kihívások egyike képes feldolgozni a folyamat, és valós időben, különösen nagy mennyiségben üzenetek tárolásához. Feldolgozási oly módon, hogy nem blokkolja a betöltési folyamat kell elvégezni. Az adattár támogatnia kell a nagy mennyiségű írást. További kihívást jelent cselekedhet az adatokon, például a valós idejű riasztások létrehozásával vagy az adat megjelenítéséhez az irányítópult valós idejű (vagy közel valós idejű) van folyamatban.

## <a name="architecture"></a>Architektúra

Egy valós idejű feldolgozási architektúra a következő logikai összetevőket tartalmazza.

- **Valós idejű üzenetbetöltés.** Az architektúra tartalmaznia kell egy módja egy adatfolyam-feldolgozási fogyasztói által használt valós idejű üzenetek rögzítését és tárolását. Egyszerű esetben ez a szolgáltatás valósíthatja meg, egy egyszerű adattár, amelyben új üzenetek letétbe egy mappában. Azonban gyakran a megoldás egy közvetítő, például az Azure Event Hubs, amely pufferként működik, az üzenetek. A közvetítő támogatnia kell a kibővített feldolgozást, és megbízható kézbesítést.

- **Stream-feldolgozás.** A valós idejű üzenetek rögzítése után a megoldásnak fel kell dolgoznia, azaz szűrnie, összesítenie és egyéb módon elő kell készítenie az adatokat az elemzéshez.

- **Analitikai adattár.** Számos big data-megoldás előkészíti az adatokat az elemzésre, és ezután bocsátja a feldolgozott adatokat, hogy lekérdezhetők legyenek elemzőeszközökkel strukturált formátumban lettek kialakítva. 

- **Elemzés és jelentéskészítés.** A legtöbb big data-megoldás célja az, hogy elemzéssel és jelentéskészítéssel betekintést nyújtson az adatokba. 

## <a name="technology-choices"></a>Technológiai lehetőségek

A következő technológiákat használata akkor javasolt lehetőségek a valós idejű feldolgozás megoldások az Azure-ban.

### <a name="real-time-message-ingestion"></a>Valós idejű üzenetbetöltés

- **Azure Event Hubs**. Az Azure Event Hubs egy üzenet üzenetsor-kezelési megoldás több millió eseményt üzenetek / másodperc feldolgozására. A rögzített eseményadatok párhuzamosan több fogyasztó dolgozhassák fel.
- **Azure IoT Hub**. Az Azure IoT Hub biztosít az internethez csatlakozó eszközök közötti kétirányú kommunikációt, és skálázható üzenetsor, amely képes kezelni több millió egyszerre csatlakoztatott eszközök.
- **Az Apache Kafka**. A Kafka egy nyílt forráskódú a message queuing és adatfolyam-feldolgozó alkalmazás, amely több üzenet gyártóktól is az üzenetek másodpercenként több millió kezelni méretezhetők, és irányíthatja őket a több ügyfél. A Kafka egy HDInsight-fürt típusa, az Azure-ban érhető el.

További információkért lásd: [valós idejű üzenetbetöltés](../technology-choices/real-time-ingestion.md).

### <a name="data-storage"></a>Adattárolás

- **Az Azure Storage-Blobtárolók** vagy **Azure Data Lake Store**. Bejövő valós idejű adatokat általában egy közvetítő (lásd fent) van rögzítve, de bizonyos esetekben érdemes új fájlok mappa megfigyelése és feldolgozni azokat, mivel a létrehozott vagy frissített teheti. Ezenkívül a számos valós idejű feldolgozás megoldás statikus referenciaadatokra vonatkoznak, amely tárolható egy fájltároló a streamelt adatokat használ. Végül a file storage is használható célhelyként rögzített valós idejű adatok archiválása, vagy további kötegelt feldolgozás alatt álló egy [lambda architektúra](../big-data/index.md#lambda-architecture).

További információkért lásd: [adattárolás](../technology-choices/data-storage.md).

### <a name="stream-processing"></a>Stream-feldolgozás

- **Azure Stream Analytics**. Az Azure Stream Analytics is örökös-lekérdezések futtatásához az adatok a kötetlen adatfolyamokat. Ezeket a lekérdezéseket a storage vagy az üzenet közvetítők, szűrő és az adatok historikus windows-alapú, és az eredmények írása, tárolás, adatbázisok, például fogadóként vagy közvetlenül a Power BI-jelentések összesítés adatstreamek felhasználását. Stream Analytics használ egy SQL-alapú lekérdezőnyelvet, amely támogatja a historikus és térinformatikai entitástörzséhez, és a JavaScript használatával is kiterjeszthető.
- **A Storm**. Az Apache Storm egy adatfolyam-feldolgozás egy spoutok topológiát használó nyílt forráskódú keretrendszer és boltok feldolgozható, feldolgozásához és az eredményeket a valós idejű streamelési adatforrásokhoz. Egy Azure HDInsight-fürtön Storm kiépítéséhez és valósítsanak meg topológiákat Java nyelven vagy C#.
- **Spark-Streamelés**. Az Apache Spark egy nyílt forráskódú elosztott platformot, az általános adatfeldolgozás. A Spark biztosít a Spark Streamelési API-t, amelyben is írhat kódot bármilyen nyelven támogatott Spark, beleértve a Java, a Scala és a Python. A Spark 2.0 vezette be a Spark strukturált Streamelési API-t, amely egyszerűbb, egységesebb és programozási modellt biztosít. Spark 2.0-ból egy Azure HDInsight-fürtön érhető el.

További információkért lásd: [Stream feldolgozási](../technology-choices/stream-processing.md).

### <a name="analytical-data-store"></a>Analitikai adattár

- **Az SQL Data Warehouse**, **HBase**, **Spark**, vagy **Hive**. Feldolgozott valós idejű adatok tárolhatók egy relációs adatbázisban, például Azure SQL Data Warehouse, egy NoSQL-tárolót, például a HBase, vagy nem található fájlok elosztott storage melyik Spark vagy a Hive táblák definiált és kérdezhető le.

További információkért lásd: [analitikus adattárak](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Elemzések és jelentéskészítés

- **Az Azure Analysis Services**, **Power bi-ban**, és **Microsoft Excel**. Feldolgozott valós idejű adatokat egy analitikai adattár tárolt korábbi jelentéskészítési és elemzési ugyanúgy, mint a feldolgozott adatok is használható. Ezenkívül a Power BI segítségével valós idejű (vagy közel valós idejű) jelentéseket és vizualizációkat elég alacsony késés esetén analitikai adatforrásokból származó, vagy közvetlenül az adatfolyam-feldolgozó kimenete az egyes esetekben közzététele.

További információkért lásd: [elemzések és jelentéskészítés](../technology-choices/analysis-visualizations-reporting.md).

A tisztán valós idejű megoldás a feldolgozási vezénylési a legtöbb kezeli a üzenetbetöltés és adatfolyam-feldolgozó összetevőinek. Azonban a lambda architektúra kötegelt feldolgozások és a valós idejű feldolgozás egyesítő, szükség lehet egy vezénylési keretrendszert, például az Azure Data Factory vagy Apache Oozie és Sqoop segítségével kezelheti a batch-munkafolyamatok rögzített valós idejű adatokat.

## <a name="next-steps"></a>További lépések

A következő referencia-architektúra látható egy teljes körű stream-feldolgozási folyamat:

- [Az Azure Stream Analytics feldolgozási Stream](../../reference-architectures/data/stream-processing-stream-analytics.md)