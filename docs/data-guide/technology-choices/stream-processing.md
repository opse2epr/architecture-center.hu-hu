---
title: A streamfeldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: fd93418c62b584e79f229e9f42703d148aeb0eca
ms.sourcegitcommit: e9d9e214529edd0dc78df5bda29615b8fafd0e56
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/28/2018
ms.locfileid: "37091063"
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>A streamfeldolgozási az Azure-ban technológia kiválasztása

Ez a cikk az Azure-ban a valós idejű streamfeldolgozó technológia lehetőségeket hasonlítja össze.

Valós idejű stream feldolgozási üzenetek üzenetsorból vagy a fájlalapú tárolást használ fel, az üzenetek feldolgozásához és az eredmény egy másik üzenet-várólista, szolgáltatásfájl-tároló vagy adatbázisba továbbítja. Feldolgozási tartalmazhatják lekérdezése, szűrési és üzenetek összesítése. A streamfeldolgozási motorok egy végtelen adatfolyamok adatok felhasználását, és minimális késéssel eredményt képesnek kell lennie. További információkért lásd: [valós idejű feldolgozására](../big-data/real-time-processing.md).

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>Mik azok a beállítások a valós idejű feldolgozásra technológia kiválasztásakor?
Azure-ban a valós idejű feldolgozással támogatja core követelményeknek megfelelő összes a következő adatokat tárolja:
- [Azure Stream Analytics](/azure/stream-analytics/)
- [HDInsight Spark Streamelési](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [Az Apache Spark on Azure Databricks](/azure/azure-databricks/)
- [HDInsight Storm](/azure/hdinsight/storm/apache-storm-overview)
- [Azure Functions](/azure/azure-functions/functions-overview)
- [Azure App Service WebJobs](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

Valós idejű feldolgozással forgatókönyvek esetén az igényeinek megfelelő szolgáltatás kiválasztása a kérdések megválaszolásával megkezdéséhez:

- Szeretné a deklaratív vagy imperatív megközelítés szerzői streamfeldolgozó logikákat?

- Szüksége van historikus feldolgozást vagy Ablakozó beépített támogatást?

- Az adatok mellett az Avro, JSON vagy CSV formátumú nem érkeznek? Ha igen, akkor fontolja meg a beállítások bármilyen egyéni kód használatával formátumot támogatja.

- 1 GB/s meghaladja a feldolgozási méretezése kell? Ha igen, fontolja meg a beállításokat, a fürtméret méretezést. 

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit. 

### <a name="general-capabilities"></a>Általános képességek

| | Azure Stream Analytics | HDInsight Spark Streamelési | Apache Spark az Azure Databricksben | HDInsight Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Programozható | A Stream analytics lekérdezési nyelv, JavaScript | Scala, Python, Java | Scala, Python, Java, R | A Java, a C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| Programozási paradigma | Deklaratív | Deklaratív és imperatív | Deklaratív és imperatív | Imperatív | Imperatív | Imperatív |    
| Díjszabási modell | [Adatfolyam-egységek](https://azure.microsoft.com/pricing/details/stream-analytics/) | Fürt óránként | [Databricks egység](https://azure.microsoft.com/pricing/details/databricks/) | Fürt óránként | Egy függvény végrehajtása és erőforrás-felhasználás | App service csomag óránként |  

### <a name="integration-capabilities"></a>Integrációs funkciók

| | Azure Stream Analytics | HDInsight Spark Streamelési | Apache Spark az Azure Databricksben | HDInsight Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Bemenetek | Az Azure Event Hubs, Azure IoT Hub, Azure Blob-tároló  | Az Event Hubs, IoT-központot, Kafka, HDFS, Storage blobs szolgáltatásban, az Azure Data Lake Store  | Az Event Hubs, IoT-központot, Kafka, HDFS, Storage blobs szolgáltatásban, az Azure Data Lake Store  | Az Event Hubs, IoT-központot, Storage blobs szolgáltatásban, az Azure Data Lake Store  | [Támogatott kötések](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | A Service Bus, Tárüzenetsort, Storage blobs szolgáltatásban, az Event Hubs, Webhookokkal, DB, Cosmos-fájlok |
| Fogadók esetében |  Az Azure Data Lake Store, az Azure SQL-adatbázis, Storage blobs szolgáltatásban, az Event Hubs, Power BI-ban a Table Storage, Service Bus-üzenetsorok, Service Bus-Témaköröktől, Cosmos DB, az Azure Functions  | HDFS, Kafka, Storage blobs szolgáltatásban, az Azure Data Lake Store, Cosmos DB | HDFS, Kafka, Storage blobs szolgáltatásban, az Azure Data Lake Store, Cosmos DB | Event Hubs, a Service Bus Kafka | [Támogatott kötések](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | A Service Bus, Tárüzenetsort, Storage blobs szolgáltatásban, az Event Hubs, Webhookokkal, DB, Cosmos-fájlok | 

### <a name="processing-capabilities"></a>Feldolgozási képességek

| | Azure Stream Analytics | HDInsight Spark Streamelési | Apache Spark az Azure Databricksben | HDInsight Storm | Azure Functions | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | --- | 
| Beépített historikus Ablakozó/támogatása | Igen | Igen | Igen | Igen | Nem | Nem |
| A bemeneti adatok formátumok | Az Avro, JSON vagy CSV, UTF-8 kódolású | Olyan egyéni kód használatával formátumban | Olyan egyéni kód használatával formátumban | Olyan egyéni kód használatával formátumban | Olyan egyéni kód használatával formátumban | Olyan egyéni kód használatával formátumban |
| Méretezhetőség | [Lekérdezési partíciók](/azure/stream-analytics/stream-analytics-parallelization) | Fürt méretét, amelyet | Határolt Databricks méretezési fürtkonfiguráció | Fürt méretét, amelyet | 200 függvény app példányok párhuzamos feldolgozása | Amelyet az app service csomag kapacitás | 
| Késő érkezés és nem megfelelő sorrendben eseménykezelést | Igen | Igen | Igen | Igen | Nem | Nem |

Lásd még:

- [A valós idejű üzenet adatfeldolgozást technológia kiválasztása](./real-time-ingestion.md)
- [Az Apache Storm és az Azure Stream Analytics összehasonlítása](/azure/stream-analytics/stream-analytics-comparison-storm)
- [Valós idejű feldolgozás](../big-data/real-time-processing.md)
