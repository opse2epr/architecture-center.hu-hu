---
title: Egy adatfolyam-feldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 2e0d142bc5cd462703ef1ca4530a2104efdf3be3
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902627"
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Egy adatfolyam-feldolgozó az Azure-ban technológia kiválasztása

Ez a cikk összehasonlítja a technológiai lehetőségek a valós idejű adatfolyam-feldolgozó az Azure-ban.

Valós idejű adatfolyam-feldolgozás fogadja az üzeneteket az üzenetsor vagy a fájlalapú tárolást, az üzenetek feldolgozása és továbbítása az eredmény egy másik üzenet-várólista, tárolását és adatbázis. Feldolgozási tartalmazhat lekérdezése, szűréssel és összesíti az üzeneteket. Stream motorok feldolgozási egy végtelen streameket, az adatok és az eredmények minimális késéssel képesnek kell lennie. További információkért lásd: [valós idejű feldolgozás](../big-data/real-time-processing.md).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Mik azok a beállítások, valós idejű feldolgozásra technológia kiválasztásakor?
Az Azure-ban mind a következő adattárakat felel meg a valós idejű feldolgozás támogató alapvető követelményeknek:
- [Azure Stream Analytics](/azure/stream-analytics/)
- [HDInsight Spark-Streamelés](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [Az Apache Spark az Azure Databricksben](/azure/azure-databricks/)
- [HDInsight a Storm](/azure/hdinsight/storm/apache-storm-overview)
- [Azure Functions](/azure/azure-functions/functions-overview)
- [Azure App Service WebJobs](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

A valós idejű feldolgozás esetben kiválasztása az igényeinek megfelelő szolgáltatás által a kérdések megválaszolása kezdete:

- Részesíti előnyben a deklaratív vagy imperatív módszer az Authoring Tool streamfeldolgozó logikákat?

- Szükség van beépített módon támogatja a historikus feldolgozás vagy ablakkezelési?

- Az adatok mellett az avro-hoz, JSON vagy CSV formátumban nem érkezik? Ha igen, fontolja meg a beállítások minden olyan egyéni kóddal formátumot támogatják.

- Van szüksége az adatok feldolgozását, 1 GB/s méretezését? Ha igen, fontolja meg a beállításokat, amelyeket a fürt méretének méretezést. 

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek. 

### <a name="general-capabilities"></a>Általános képességek

| | Azure Stream Analytics | HDInsight Spark Streamelési | Apache Spark az Azure Databricksben | HDInsight Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Programozhatóság | Stream analytics lekérdezési nyelvet, JavaScript | Scala, Python, Java | Scala, Python, Java, az R | A Java,C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Programozási költségei | Deklaratív | Deklaratív és az imperatív | Deklaratív és az imperatív | Imperatív | Imperatív | Imperatív |    
| Díjszabási modell | [A folyamatos átviteli egységek](https://azure.microsoft.com/pricing/details/stream-analytics/) | Fürt óránként | [Databricks-egységek](https://azure.microsoft.com/pricing/details/databricks/) | Fürt óránként | Egy függvény végrehajtási és az erőforrás-felhasználás | App service csomag óránként |  

### <a name="integration-capabilities"></a>Integrációs funkciók

| | Azure Stream Analytics | HDInsight Spark Streamelési | Apache Spark az Azure Databricksben | HDInsight Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Bemenetek | Az Azure Event Hubs, Azure IoT Hub, az Azure Blob storage  | Az Event Hubs, az IoT Hub, a Kafka, HDFS, Storage-Blobokkal, az Azure Data Lake Store  | Az Event Hubs, az IoT Hub, a Kafka, HDFS, Storage-Blobokkal, az Azure Data Lake Store  | Az Event Hubs, az IoT Hub, a Storage-Blobokkal, az Azure Data Lake Store  | [Támogatott kötések](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | A Service Bus, tároló-üzenetsorok, Storage-Blobokkal, az Event Hubs, Webhookok, Cosmos DB-fájlok |
| fogadóként |  Az Azure Data Lake Store, az Azure SQL Database, Storage-Blobokkal, az Event Hubs, a Power bi-ban, Table Storage, Service Bus-üzenetsorok, Service Bus-témakörök, Cosmos DB, az Azure Functions  | HDFS, a Kafka, a Storage-Blobokkal, a Azure Data Lake Store, a Cosmos DB | HDFS, a Kafka, a Storage-Blobokkal, a Azure Data Lake Store, a Cosmos DB | Az Event Hubs, Service Bus, a Kafka | [Támogatott kötések](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | A Service Bus, tároló-üzenetsorok, Storage-Blobokkal, az Event Hubs, Webhookok, Cosmos DB-fájlok | 

### <a name="processing-capabilities"></a>Feldolgozási képességek

| | Azure Stream Analytics | HDInsight Spark Streamelési | Apache Spark az Azure Databricksben | HDInsight Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Beépített historikus ablakkezelési/támogatása | Igen | Igen | Igen | Igen | Nem | Nem |
| A bemeneti adatok formátumok | Az Avro, JSON vagy CSV, UTF-8 kódolású | Tetszőleges méretű, egyéni kód használatával | Tetszőleges méretű, egyéni kód használatával | Tetszőleges méretű, egyéni kód használatával | Tetszőleges méretű, egyéni kód használatával | Tetszőleges méretű, egyéni kód használatával |
| Méretezhetőség | [Lekérdezési partíciók](/azure/stream-analytics/stream-analytics-parallelization) | Amelyet a fürt mérete | Databricks-fürt méretezés konfigurálása, amelyet | Amelyet a fürt mérete | Akár 200 függvény alkalmazáspéldány párhuzamos feldolgozása | Amelyet az app service kapacitás megtervezése | 
| Késedelmes beérkezés és sorrendben eseménykezelést | Igen | Igen | Igen | Igen | Nem | Nem |

Lásd még:

- [Egy valós idejű üzenetek feldolgozási technológia kiválasztása](./real-time-ingestion.md)
- [Valós idejű feldolgozására](../big-data/real-time-processing.md)
