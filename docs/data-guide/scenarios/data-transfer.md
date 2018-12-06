---
title: Egy adatátviteli technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: d5fbdc3a49ab16be2626b772ffd1af782963a2f0
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902680"
---
# <a name="transferring-data-to-and-from-azure"></a>Adatátvitel, és az Azure-ból

Való adatátvitel, és az Azure-beli igényeitől függően több lehetőség áll rendelkezésre.

## <a name="physical-transfer"></a>Fizikai átvitel

Fizikai hardvert használja az adatok átviteléhez az Azure-bA egy megfelelő beállítást:

- Az hálózat, a lassú vagy nem megbízható.
- További hálózati sávszélesség első költségekkel járnának.
- Biztonsági vagy a szervezeti szabályzatok nem kimenő kapcsolatok engedélyezése bizalmas adatok esetén. 

Ha az elsődleges szempont, akkor mennyi ideig tart az adatátvitel, érdemes ellenőrizni, hogy hálózati átvitelt ténylegesen lassabb, mint a fizikai átviteli a vizsgálat futtatása.

Adatok az Azure fizikailag szállítására két fő lehetőség van:
- **Az Azure Import/Export**. A [Azure Import/Export szolgáltatás](/azure/storage/common/storage-import-export-service) segítségével biztonságosan nagy mennyiségű adatot továbbít az Azure Blob Storage vagy az Azure Files belső SATA HDD vagy Sdd szállításával az Azure-adatközpontba. Adatátvitel Azure Storage-ból merevlemezekre, és ezek szállításra Önnek betöltésére, a helyszínen is használhatja ezt a szolgáltatást.

- **Az Azure Data Box**. [Az Azure Data Box](https://azure.microsoft.com/services/storage/databox/) egy Microsoft által biztosított berendezés, amely az Azure Import/Export szolgáltatás hasonlóan működik. A Microsoft jogvédett, biztonságos, és feltörhetetlen berendezést letöltésként érhető el, és kezeli a teljes körű logisztika, amely a portálon keresztül nyomon követheti. Az Azure Data Box szolgáltatás egyik előnye a könnyű használatra. Nem kell több merevlemezeket vásárolni, készítse elő azokat, és minden egyes fájlok átvitele. Az Azure Data Box-szolgáltatás piacvezető Azure-partnerek könnyebb zökkenőmentesen használhatják a termékeiket a felhőbe offline átviteli számos támogatja. 

## <a name="command-line-tools-and-apis"></a>Parancssori eszközök és API-k

Vegye figyelembe ezeket a beállításokat, ha azt szeretné, hogy parancsfájlok és programozás alapú adatátvitelt.

- **Azure parancssori felület (CLI)**. A [Azure CLI-vel](/azure/hdinsight/hdinsight-upload-data#commandline) egy platformfüggetlen eszköz, amely lehetővé teszi az Azure-szolgáltatások kezelését, és az Azure Storage-adatok feltöltése. 

- **Az AzCopy**. Az AzCopy használata egy [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) vagy [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) parancssori az optimális teljesítmény érdekében az Azure Blob, fájl és Table storage szolgáltatásba vagy onnan egyszerűen adatmásolás céljából. AzCopy támogatja az egyidejűség és a párhuzamosság és másolási műveletek során megszakad folytatódhasson. Emellett akkor is gyorsabb, mint a legtöbb más lehetőségek. Programozott hozzáférés a [a Microsoft Azure Storage adatátviteli könyvtár](/azure/storage/common/storage-use-data-movement-library) az azcopyt működtető alapvető keretrendszer. .NET Core kódtárként nyújtja. 

- **PowerShell**. A [ `Start-AzureStorageBlobCopy` PowerShell-parancsmag](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0) lehetőség van a Windows rendszergazdák számára, aki a PowerShell segítségével.  

- **Az AdlCopy**. [Az AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) lehetővé teszi, hogy az adatok másolása az Azure Storage-Blobokból a Data Lake Store-bA. Is használható két Azure Data Lake Store-fiókok közötti adatokat másolja. Azonban nem használható az adatok másolása az Data Lake Store a Storage-blobokat.

- **A Distcp**. Ha a Data Lake Store-hozzáféréssel rendelkező HDInsight-fürt, például a Hadoop-ökoszisztéma eszközök használhatja [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp) másolhat adatokat egy HDInsight-fürt storage (WASB) és a egy Data Lake Store-fiókba.

- **Sqoop**. [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) egy Apache-projecttel, a Hadoop-ökoszisztéma része. Ez előtelepítve minden HDInsight-fürtön. Lehetővé teszi egy HDInsight-fürtöt és például az SQL, Oracle, MySQL és egyéb relációs adatbázisok közötti adatátvitelt. Sqoop kapcsolódó eszközöket, beleértve az importálás és exportálás gyűjteménye. Sqoop együttműködik a HDInsight-fürtöket az Azure Storage BLOB vagy a Data Lake Store csatolt storage használatával.

- **A PolyBase**. [A PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) technológiája, amely az adatbázison kívül adatokhoz fér hozzá a a T-SQL nyelv használatával. Az SQL Server 2016 lehetővé teszi a Hadoop a külső adatok lekérdezések futtatására vagy Azure Blob Storage-ból adatok importálása és exportálása. Az Azure SQL Data Warehouse akkor is importálási/exportálási adatok Azure Blob Storage-ból és az Azure Data Lake Store. A PolyBase jelenleg a leggyorsabb eljárás a adatok importálása az SQL Data Warehouse-bA.

- **Hadoop parancssor**. Ha egy HDInsight-fürt fő csomópontjának lévő adatok, használhatja a `hadoop -copyFromLocal` parancsot, hogy az adatok másolása a fürt csatlakoztatott tárolókba, például az Azure Storage-blobból vagy Azure Data Lake Store. Annak érdekében, hogy a Hadoop parancsot használja, először csatlakoznia kell a fő csomópontot. Ha csatlakoztatva van, feltölthet egy fájlt a storage.

## <a name="graphical-interface"></a>Grafikus felület

Vegye figyelembe a következő beállításokat, ha csak át néhány fájlt, vagy objektumok, és automatizálja a folyamatot nem kell.

- **Az Azure Storage Explorer**. [Az Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) platformfüggetlen eszköz, amely lehetővé teszi az Azure-tárfiókok tartalmának kezeléséhez. Lehetővé teszi, hogy feltöltése, letöltése és kezelése a blobok, fájlok, üzenetsorok, táblák és az Azure Cosmos DB-entitásokat. Használhatja a Blob storage-blobok és mappák kezelése, valamint feltöltése és tölthet le blobokat a helyi fájlrendszerben és a Blob storage vagy tárfiókok között.

- **Az Azure portal**. A Blob storage és a Data Lake Store egy webes felületet biztosít az fájlok felfedezése és a feltöltése az új fájlokat egyenként. Ez a beállítást akkor jó választás, ha nem szeretné, hogy telepítse minden olyan eszközt, vagy nem ad ki parancsokat gyorsan megismerheti a fájlokat, vagy egyszerűen csak töltse fel újakat néhány.

## <a name="data-pipeline"></a>Adatfolyamat

**Az Azure Data Factory**. [Az Azure Data Factory](/azure/data-factory/) legjobb felügyelt szolgáltatást rendszeresen közti fájlátvitelre szolgáló Azure-szolgáltatások, a helyszíni vagy a kettő kombinációját számos olyan. Azure Data Factory használatával hozhat létre és ütemezhet, adatvezérelt munkafolyamatokat (folyamatokat is nevezik) különböző adattárolókból adatokat beolvasó. feldolgozhatók és átalakíthatók az adatok különböző számítási szolgáltatások használatával (például Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics és Azure Machine Learning), Az adatvezérelt munkafolyamatokat hozhat létre [replikálásával segít a vállalatnak](../technology-choices/pipeline-orchestration-data-movement.md) és automatizálja az adatok áthelyezését és átalakítását.

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Adatok átvitele forgatókönyvek esetén válassza ki a az igényeinek megfelelő rendszert ezen kérdések megválaszolásával:

- Szükséges átviteli nagyon nagy mennyiségű adatot, ahol az interneten keresztül kapcsolat állna túl hosszú, így nem megbízható hálózathatáron vagy túl költséges lehet? Ha igen, érdemes lehet fizikai átvitel.

- Parancsfájl-az átviteli feladatok, így azok újrafelhasználható inkább? Ha igen, válassza ki a parancssori kapcsolók és az Azure Data Factory.

- Szükséges egy nagyon nagy mennyiségű adat átvitele a hálózaton keresztül? Ha igen, válassza ki az olyan beállítás, amely a big Data típusú adatok van optimalizálva.

- Szükséges-adatok áthelyezésénél relációs adatbázisból? Ha igen, válasszon olyan beállítás, amely egy vagy több relációs adatbázisokat támogatja. Vegye figyelembe, hogy a beállítások egy része emellett szükséges egy Hadoop-fürtöt.

- Szükség van egy automatizált adatfeldolgozási folyamat vagy munkafolyamat vezénylési? Ha igen, fontolja meg az Azure Data Factoryban.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="physical-transfer"></a>Fizikai átvitel

| | Azure Import/Export szolgáltatás | Azure Data Box |
| --- | --- | --- |
| Helyigény | Belső SATA HDD vagy Sdd | Biztonságos, hamisíthatatlan, egyetlen hardver készülék |
| A Microsoft felügyeli a szállítási címhez tartozó logisztika | Nem | Igen |
| Partnerek termékei integrálható | Nem | Igen |
| Egyéni készülék | Nem | Igen |

### <a name="command-line-tools"></a>Parancssori eszközök

**Hadoop/HDInsight**

| | A Distcp | Sqoop | Hadoop-CLI |
| --- | --- | --- | --- |
| Big Data-optimalizált | Igen | Igen |  Igen |
| Relációs adatbázis másolása |  Nem | Igen | Nem |
| Relációs adatbázis másolása |  Nem | Igen | Nem |
| Másolja a Blob storage |  Igen | Igen | Igen |
| A Blob storage-ból | Igen |  Igen | Nem |
| Data Lake Store másolása | Igen | Igen | Igen |
| A Data Lake Store másolása | Igen | Igen | Nem |

**Egyéb**

| | Azure CLI | AzCopy | PowerShell | Az AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Kompatibilis platformon | Linux, OS X, Windows | Linux, Windows | Windows | Linux, OS X, Windows | Az SQL Server, az Azure SQL Data warehouse-bA | 
| Big Data-optimalizált | Nem | Nem | Nem | Igen <sup>1</sup> | Igen <sup>2</sup> |
| Relációs adatbázis másolása | Nem | Nem | Nem | Nem | Igen | 
| Relációs adatbázis másolása | Nem | Nem | Nem | Nem | Igen | 
| Másolja a Blob storage | Igen | Igen | Igen | Nem | Igen | 
| A Blob storage-ból | Igen | Igen | Igen | Igen | Igen |
| Data Lake Store másolása | Nem | Nem | Igen | Igen |  Igen | 
| A Data Lake Store másolása | Nem | Nem | Igen | Igen | Igen | 


[1] az AdlCopy a Data Lake Analytics-fiók használatakor a big Data típusú adatok átvitele van optimalizálva.

[2] PolyBase [növelhető a teljesítmény](/sql/relational-databases/polybase/polybase-guide#performance) számítási leküldése a Hadoop és a használatával [PolyBase kibővített csoportok](/sql/relational-databases/polybase/polybase-scale-out-groups) az SQL Server példányai és a Hadoop-csomópontjai közötti párhuzamos adatátvitel engedélyezése.

### <a name="graphical-interface-and-azure-data-factory"></a>Grafikus felület és az Azure Data Factory

| | Azure Storage Explorer | Az Azure portal * | Azure Data Factory |
| --- | --- | --- | --- |
| Big Data-optimalizált | Nem | Nem | Igen | 
| Relációs adatbázis másolása | Nem | Nem | Igen |
| Relációs adatbázis másolása | Nem | Nem | Igen |
| Másolja a Blob storage | Igen | Nem | Igen |
| A Blob storage-ból | Igen | Nem | Igen |
| Data Lake Store másolása | Nem | Nem | Igen |
| A Data Lake Store másolása | Nem | Nem | Igen |
| Feltöltése a Blob storage | Igen | Igen | Igen |
| A Data Lake Store feltöltése | Igen | Igen | Igen |
| Összehangolhatja a adatátvitel | Nem | Nem | Igen |
| Egyéni adatátalakítások | Nem | Nem | Igen |
| Díjszabási modell | Ingyenes | Ingyenes | Fizessen a használat |

\* Az Azure portal ebben az esetben azt jelenti, a webes adatvizsgálati eszközök használata a Blob storage és a Data Lake Store.

