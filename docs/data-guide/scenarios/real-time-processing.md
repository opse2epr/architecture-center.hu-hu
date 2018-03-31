---
title: "Valós idejű feldolgozására"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f1054ce5e8c2053aa4f80d8b472604125ba47187
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="real-time-processing"></a>Valós idejű feldolgozására

Valós idejű feldolgozására, amelyek rögzítve lesznek a valós idejű és a feldolgozott használatával hozzon létre valós idejű (vagy közel valós idejű) jelentések vagy automatikus válaszok minimális késéssel adatstreamek foglalkozik. Azt jelzi, például felügyeleti megoldás a valós idejű forgalom használhatja-e nagy forgalommal észleléséhez érzékelőadatait. Ezek az adatok dinamikus frissítése a torlódás megjelenítéséhez, vagy automatikusan kezdeményezni a nagy-Foglaltság sávok vagy más szolgáltatási rendszerek térképre használható.

![](./images/real-time-pipeline.png)

Valós idejű feldolgozással van definiálva a bemeneti adatok, feldolgozás nagyon rövid késési követelményekkel rendelkező unbounded adatfolyam feldolgozása, &mdash; ezredmásodperc vagy másodpercben mérve. A bejövő adatok általában érkezik egy strukturálatlan és félig strukturált formátumban, például JSON, és azonos feldolgozási követelmények vonatkoznak [kötegfeldolgozási](./batch-processing.md), de valós idejű fogyasztás támogatásához rövidebb esetenként idő feltüntetésével.

Feldolgozott adatok gyakran írni egy analitikai adattároló, amely elemzés és -megjelenítésre van optimalizálva. A feldolgozott adatok közvetlenül a elemzés és jelentéskészítési réteg elemzés, a business intelligence és a valós idejű irányítópulton a képi megjelenítés is meghatározták.

## <a name="challenges"></a>Kihívásai

A valós idejű feldolgozással megoldások nagy kihívást egyik betöltési, a folyamat, és valós időben, különösen a nagy mennyiségük üzenetek tárolásához. Feldolgozási úgy, hogy nem blokkolja az adatfeldolgozást folyamat után kell elvégezni. Az adattár támogatnia kell a nagy mennyiségű írási műveleteket. További kihívást van tudnak működni az adatokon gyorsan, például a riasztások valós időben, vagy a valós idejű (vagy közel valós idejű) Irányítópulton az adat megjelenítéséhez.

## <a name="architecture"></a>Architektúra

A valós idejű feldolgozással architektúra a következő logikai részből áll.

- **Valós idejű üzenet adatfeldolgozást.** Az architektúra tartalmaznia kell egy adatfolyam feldolgozása végfelhasználói által valós idejű üzenetek tárolásához, majd úgy. Egyszerű esetben ez a szolgáltatás egy mappa letétbe új üzenetek egyszerű adattárként találhatja. De gyakran a megoldás igényel üzenet közvetítő, például az Azure Event Hubs, amely az üzenetek pufferként. Az üzenet broker támogatnia kell a kibővített feldolgozási és megbízható kézbesítését.

- **Az adatfolyam feldolgozása.** Valós idejű üzenetek rögzítésével, miután a megoldás kell dolgozza fel őket szűrés, összesítése és egyéb előkészítése az adatok elemzése.

- **Analitikai adatokat tároló.** Adatok előkészítése az elemzési és a feldolgozott adatok szolgáljanak strukturált formátuma nem kérdezhetők le analitikai eszközeivel sok big data-megoldások tervezték. 

- **Elemzési és jelentéskészítési.** A legtöbb big data-megoldások célja az adatok elemzési és jelentéskészítési betekintést. 

## <a name="technology-choices"></a>Technológiai lehetőségek

A következő technológiákat kell az Azure-ban a valós idejű feldolgozással megoldások használata ajánlott.

### <a name="real-time-message-ingestion"></a>Valós idejű üzenet adatfeldolgozást

- **Azure Event Hubs**. Az Azure Event Hubs egy olyan üzenet üzenetsor-kezelési megoldás választásával dolgozhat fel több millió esemény üzenetek száma másodpercenként. A rögzített eseményadatok párhuzamosan több felhasználóból tudja feldolgozni.
- **Azure IoT Hub**. Azure IoT Hub biztosít az internethez csatlakozó eszközök közötti kétirányú kommunikációs, és a méretezhető üzenet-várólista kezelhető egyidejűleg több millió csatlakoztatott eszközök.
- **Apache Kafka**. Kafka egy nyílt forráskódú üzenetsor és a streamfeldolgozási alkalmazás, amely méretezhető, több üzenetet gyártók üzenetek száma másodpercenként több millió leíróját, és több felhasználóból továbbítani. Kafka érhető el az Azure HDInsight fürt típusként.

További információkért lásd: [valós idejű üzenet adatfeldolgozást](../technology-choices/real-time-ingestion.md).

### <a name="data-storage"></a>Adattárolás

- **Az Azure Storage-Blob tárolók** vagy **az Azure Data Lake Store**. Bejövő valós idejű adatok általában bekerül az üzenet közvetítő (lásd fent), de bizonyos esetekben érdemes lehet az új fájlok mappa megfigyelése és dolgozza fel őket, a létrehozott vagy frissített tehet. Emellett számos valós idejű feldolgozással megoldás statikus referenciaadatok, amely egy fájltároló tárolhatja a streamelési adatok egyesítése. Végezetül a file storage használható célhelyként rögzített valós idejű adatokat tartalmaznak, vagy további feldolgozás alatt álló kötegben egy [lambda architektúra](../concepts/big-data.md#lambda-architecture).

További információkért lásd: [adattárolás](../technology-choices/data-storage.md).

### <a name="stream-processing"></a>Az adatfolyam feldolgozása

- **Azure Stream Analytics**. Az Azure Stream Analytics egy unbounded adatfolyamba való is örökös lekérdezéseinek futtatásához. Ezeket a lekérdezéseket adatstreamek tároló- és üzenet brókerek, szűrő és az adatok historikus windows-alapú, és az eredmények írási, mosdók például a tárolási, adatbázisok, vagy közvetlenül a Power BI jelentéseket összesítést használ.
- **A Storm**. Apache Storm egy nyílt forráskódú keretrendszer, amely használja a spoutokkal kapcsolatban-topológia adatfolyam feldolgozásra és felhasználását, boltokhoz feldolgozásához, és az eredményeket a valós idejű streamelési adatforrásokból. Az Azure HDInsight-fürtök Storm kiépítéséhez és valósítsanak meg topológiákat Java vagy C#.
- **Spark Streaming**. Apache Spark egy nyílt forráskódú általános adatfeldolgozás elosztott platform. Spark biztosít a Spark Streamelési API-t, amelyben a kód bármilyen nyelven támogatott Spark, többek között a Java, Scala és Python írhat. Spark 2.0 rendszerben jelent meg a Spark strukturált Streaming API, ami egy egyszerűbb, egységesebb és programozási modellt biztosít. Spark 2.0 Azure HDInsight-fürtök érhető el.

További információkért lásd: [adatfolyam feldolgozása](../technology-choices/stream-processing.md).

### <a name="analytical-data-store"></a>Analitikai adatokat tároló

- **Az SQL Data Warehouse**, **HBase**, **Spark**, vagy **Hive**. Feldolgozott valós idejű adatok tárolhatók ilyen Azure SQL Data Warehouse, egy NoSQL-tároló, például a HBase, vagy a fájlok tárolási mely Spark vagy Hive táblák definiált és lekérdezett elosztott relációs adatbázis.

További információkért lásd: [analitikai adattárolókhoz](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Elemzés és jelentéskészítés

- **Az Azure Analysis Services**, **a Power BI**, és **Microsoft Excel**. Feldolgozott valós idejű adatok, amelyek az analitikus adatok tárolása a feldolgozott adatokat azonos módon használható korábbi jelentéskészítés és elemzés céljára. Emellett a Power BI segítségével teszik közzé a valós idejű (vagy közel valós idejű) jelentések és a képi megjelenítés esetén elég alacsony késleltetésű analitikai adatforrásokból származó, vagy bizonyos esetekben a streamfeldolgozási kimeneti-ről.

További információkért lásd: [elemzési és jelentéskészítési](../technology-choices/analysis-visualizations-reporting.md).

A probléma pusztán valós idejű megoldás a feldolgozási vezénylési a legtöbb kezeli az üzenet adatfeldolgozást és feldolgozó összetevőinek adatfolyam. Azonban, hogy kötegfeldolgozási és valós idejű feldolgozással lambda architektúra esetén szükség lehet rögzített valós idejű adatok kötegelt munkafolyamatok kezelését az orchestration keretrendszert, például az Azure Data Factory vagy az Apache Oozie és a Sqoop használatával.

