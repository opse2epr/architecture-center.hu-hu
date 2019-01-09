---
title: A kötegelt feldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 11/03/2018
ms.openlocfilehash: 0c6392fb0a921e95f2704696fb2447ac5ec6f4c0
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111325"
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Egy köteg feldolgozása az Azure-ban technológia kiválasztása

Big data-megoldások gyakran használnak a hosszan futó kötegelt feladatok szűréséhez, összesítéséhez és ellenkező esetben az adatok elemzésre való előkészítését. Ezek a feladatok általában magukban, skálázható tárolók (például a HDFS, az Azure Data Lake Store és az Azure Storage) az adatforrások beolvasását, feldolgozását és méretezhető tárhely az új fájlok írása a kimenet.

A legfontosabb követelmény, ilyen köteg feldolgozása motorok, horizontális felskálázás számításokat, annak érdekében, hogy nagy mennyiségű adat kezelésére képes. Valós idejű feldolgozás eltérően azonban kötegelt feldolgozás kellene rendelkeznie az késleltetések (az adatbetöltés és az eredmény kiszámítása közötti idő), amely a mérték a óra.

## <a name="technology-choices-for-batch-processing"></a>Technológiai lehetőségek a kötegelt feldolgozáshoz

### <a name="azure-sql-data-warehouse"></a>Azure SQL Data Warehouse

[Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer analytics végre, a nagy mennyiségű adat. Támogatja a nagy olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas nagy teljesítményű elemzési futtatásához. Amikor nagy mennyiségű (több mint 1 TB) adat rendelkezik, és a egy párhuzamosság kihasználó elemzési számítási feladatot futtat, vegye figyelembe az SQL Data warehouse-bA.

### <a name="azure-data-lake-analytics"></a>Azure Data Lake Analytics

[A Data Lake Analytics](/azure/data-lake-analytics/data-lake-analytics-overview) szolgáltatás egy igény szerinti elemzési feladatokat végző szolgáltatás. Nagyon nagy méretű adatkészleteket az Azure Data Lake Store tárolja elosztott feldolgozásához van optimalizálva.

- A nyelvek: [U-SQL](/azure/data-lake-analytics/data-lake-analytics-u-sql-get-started) (beleértve a Python, R, és C# bővítmények).
- Integrálható az Azure Data Lake Store, az Azure Storage blobok, az Azure SQL Database és SQL Data warehouse-bA.
- Díjszabási modell Feladatonkénti.

### <a name="hdinsight"></a>HDInsight

HDInsight egy olyan felügyelt Hadoop-szolgáltatás. Üzembe helyezése és kezelése az Azure-beli Hadoop-fürtök használatát. Használhatja a kötegelt feldolgozáshoz, [Spark](/azure/hdinsight/spark/apache-spark-overview), [Hive](/azure/hdinsight/hadoop/hdinsight-use-hive), [Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started), [MapReduce](/azure/hdinsight/hadoop/hdinsight-use-mapreduce).

- A nyelvek: R, Python, Java-, Scala-, SQL
- Az Active Directory, az Apache Ranger-alapú hozzáférés-vezérlés Kerberos-hitelesítés
- A Hadoop-fürt teljes hozzáférést biztosít

### <a name="azure-databricks"></a>Azure Databricks

[Az Azure Databricks](/azure/azure-databricks/) egy Apache Spark-alapú elemzési platform. Felfoghatók úgy, mint "Spark szolgáltatás." A Spark használata az Azure platformon legegyszerűbb módja.

- A nyelvek: R, Python, Java, a Scala, a Spark SQL
- A fürt gyors kezdési idejének, auto-lezárást, automatikus méretezést.
- A Spark-fürt kezeli az Ön számára.
- Beépített integráció az Azure Blob Storage, Azure Data Lake Storage (ADLS), Azure SQL Data Warehouse (az SQL DW) és egyéb szolgáltatásokat. Lásd: [adatforrások](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html).
- Felhasználó hitelesítése az Azure Active Directoryval.
- Webes [notebookok](https://docs.azuredatabricks.net/user-guide/notebooks/index.html) együttműködés és az adatok feltárásához.
- Támogatja a [fürtök GPU-kompatibilis](https://docs.azuredatabricks.net/user-guide/clusters/gpu.html)

### <a name="azure-distributed-data-engineering-toolkit"></a>Elosztott adatok az Azure mérnöki eszközkészlet

A [elosztott adatok mérnöki eszközkészlet](https://github.com/azure/aztk) (AZTK) egy olyan eszköz, üzembe helyezés igény szerinti Spark az Azure-ban a Docker-fürtökön.

AZTK nem áll az Azure-szolgáltatások. Inkább egy ügyféloldali eszköz felülettel CLI és a Python SDK-t, az Azure Batch-alapú. Ez a beállítás révén, a legtöbb szabályozhatja az infrastruktúra egy Spark-fürt üzembe helyezésekor.

- A saját Docker-rendszerkép használata.
- Alacsony prioritású virtuális gépek használata egy 80 %-os kedvezményt.
- Vegyes üzemmódú fürtök, amelyek mind az alacsony prioritású, és dedikált virtuális gépek használata.
- Beépített támogatás az Azure Blob Storage és az Azure Data Lake-kapcsolathoz.

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szeretne egy felügyelt szolgáltatás, hanem a saját kiszolgálók kezelése?

- Biztosan hozhat létre a kötegelt feldolgozási logika deklaratív vagy adatműveleteket?

- Elvégzi a kötegelt feldolgozás a szolgáltatás? Ha igen, fontolja meg a beállításokat, amelyek lehetővé teszik, hogy a fürt automatikusan leáll, vagy amelynek díjszabási modell száma batch-feladatot.

- Szükség van relációs adattárak és a kötegelt feldolgozás, a referenciaadatok kereshet például lekérdezéséhez? Ha igen, fontolja meg a beállításokat, amelyek lehetővé teszik a külső relációs adattárak lekérdezését.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek

<!-- markdownlint-disable MD033 -->

| | Azure Data Lake Analytics | Azure SQL Data Warehouse | HDInsight | Azure Databricks |
| --- | --- | --- | --- | --- | --- |
| A felügyelt szolgáltatás | Igen | Igen | Igen <sup>1</sup> | Igen |
| Relációs adattároló | Igen | Igen | Nem | Nem |
| Díjszabási modell | Egy batch-feladat | A fürt óra | A fürt óra | Databricks-egység<sup>2</sup> + a fürt óra |

[1] a kézi konfigurálás és a méretezést.

[2] egy Databricks egységek (DBU) óránkénti feldolgozási egységet jelent.

### <a name="capabilities"></a>Funkciók

| | Azure Data Lake Analytics | SQL Data Warehouse | A Spark HDInsight | HDInsight Hive-val | HDInsight Hive LLAP-val | Azure Databricks |
| --- | --- | --- | --- | --- | --- | --- |
| Automatikus skálázás | Nem | Nem | Nem | Nem | Nem | Igen |
| Horizontális felskálázás részletessége  | Feladatonként | Fürtönként | Fürtönként | Fürtönként | Fürtönként | Fürtönként |
| Memóriában történő gyorsítótárazás az adatok | Nem | Igen | Igen | Nem | Igen | Igen |
| Külső relációs áruházakból származó lekérdezése | Igen | Nem | Igen | Nem | Nem | Igen |
| Hitelesítés  | Azure AD | SQL / Azure AD | Nem | Az Azure AD<sup>1</sup> | Az Azure AD<sup>1</sup> | Azure AD |
| Naplózás  | Igen | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen |
| Sorszintű biztonság | Nem | Nem | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> | Nem |
| Támogatja a tűzfalak | Igen | Igen | Igen | Igen <sup>2</sup> | Igen <sup>2</sup> | Nem |
| Dinamikus adatmaszkolás | Nem | Nem | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> | Nem |

<!-- markdownlint-enable MD033 -->

[1] használata szükséges egy [tartományhoz csatlakoztatott HDInsight-fürt](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] támogatással [egy Azure virtuális hálózaton belüli](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
