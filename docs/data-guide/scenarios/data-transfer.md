---
title: Egy adatátviteli technológia kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bb0732b0f771a4c9e1a4e565875576c08484490a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="transferring-data-to-and-from-azure"></a>Azure érkező vagy oda irányuló adatátvitel

Az adatoknak az Azure-ban érkező vagy oda irányuló igényeitől függően több lehetőség áll rendelkezésre.

## <a name="physical-transfer"></a>Fizikai átviteli

Adatok átviteléhez az Azure fizikai hardver használata esetén a jó kapcsolót, ha:

- A hálózat, a lassú vagy nem megbízható.
- További hálózati sávszélesség első megfizethetetlenné.
- Szervezeti házirend vagy biztonsági nem kimenő kapcsolatok engedélyezése a bizalmas adatok meghatározásakor. 

Ha az elsődleges szempont mennyi ideig tart a adatátvitelt, érdemes lehet egy tesztet annak ellenőrzéséhez, hogy hálózati átviteli ténylegesen lassabb, mint a fizikai átviteli futtatásához.

Azure-bA fizikailag szállításához két fő lehetőség áll rendelkezésre:
- **Az Azure Import/Export**. A [Azure Import/Export szolgáltatás](/azure/storage/common/storage-import-export-service) lehetővé teszi, hogy biztonságosan átviteli nagy mennyiségű adatok Azure Blob Storage-vagy Azure fájlok által belső SATA merevlemez vagy értékpapír mobilalkalmazások számára egy Azure-adatközpontban. Ez a szolgáltatás használhatja, ha az adatok átvitele Azure Storage merevlemez-meghajtók és ezek szállított Önnek a helyszíni feltöltését.

- **Az Azure Data mező**. [Az Azure Data mező](https://azure.microsoft.com/services/storage/databox/) van egy Microsoft által biztosított készülék, amely az Azure Import/Export szolgáltatás hasonlóan működik. A Microsoft meg egy védett, biztonságos, valamint a védett átviteli készüléknek, és kezeli a végpont logisztikai, amely a portálon keresztül nyomon követheti. Az Azure Data mező szolgáltatás egyik előnye használat megkönnyítése érdekében. Vásároljon több merevlemez-meghajtók, készítse elő azokat, és minden egyes fájlok átvitele nem szükséges. Az Azure Data mező iparágvezető a Azure partnerek számára, hogy könnyebben zökkenőmentesen kihasználja a termékeiket a felhőbe offline átviteli számos támogatja. 

## <a name="command-line-tools-and-apis"></a>A parancssori eszközök és API-k

Vegye figyelembe ezeket a beállításokat, ha azt szeretné, hogy a parancsprogram-alapú és programozott adatátvitel.

- **Azure parancssori felület (CLI)**. A [Azure CLI](/azure/hdinsight/hdinsight-upload-data#commandline) , amely lehetővé teszi, hogy az Azure-szolgáltatások kezelése, majd töltse fel az adatok Azure Storage-platformok eszköz. 

- **AzCopy**. Használja az AzCopy egy [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) vagy [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) parancssori adatok könnyen másolása és az Azure Blob, a fájl és a Table storage optimális teljesítménnyel. AzCopy támogatja az egyidejű és párhuzamossági, valamint a másolási művelet megszakadt, ha folytatni. Egyúttal gyorsabb, mint a legtöbb más beállítások. A programozott hozzáférést a [Microsoft Azure Storage adatátviteli könyvtár](/azure/storage/common/storage-use-data-movement-library) van az azcopyt működtető alapvető keretrendszert. Ez a .NET Core tárként valósul meg. 

- **PowerShell**. A [ `Start-AzureStorageBlobCopy` PowerShell-parancsmag](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0) lehetősége van a Windows rendszergazdák, akik a PowerShell segítségével.  

- **AdlCopy**. [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) lehetővé teszi az adatok másolása az Azure Storage blobs szolgáltatásban a Data Lake Store. Is használható két Azure Data Lake Store-fiókok közötti másolásához. Azonban nem használható adatok másolása Data Lake Store Storage blobs szolgáltatásban.

- **Ból a Distcp**. Ha Data Lake Store-hozzáféréssel rendelkező HDInsight-fürtöt, használhatja például a Hadoop-ökoszisztémával eszközök [ból a Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp) használatával adatait átmásolhatja egy HDInsight-fürt storage (WASB) érkező vagy oda irányuló Data Lake Store-fiók.

- **Sqoop**. [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop) Apache projektet és a Hadoop ökoszisztémájának részeként. Az előre telepítve van az összes HDInsight-fürtök. Lehetővé teszi a HDInsight-fürtök és a relációs adatbázisok például SQL, Oracle, MySQL stb közötti adatátvitelt. Sqoop kapcsolódó eszközeit, beleértve az importálás és exportálás gyűjteménye. Sqoop működik együtt a HDInsight-fürtök Azure Storage blobsba vagy a kapcsolt Data Lake Store-tároló.

- **A PolyBase**. [A PolyBase](/sql/relational-databases/polybase/get-started-with-polybase) technológia, amely a T-SQL nyelv keresztül fér hozzá az adatbázison kívül adatokat. SQL Server 2016-lehetővé teszi lekérdezéseket futtathat a külső adatokat a Hadoop vagy importálási/exportálási adatok Azure Blob Storage-ból. Az Azure SQL Data Warehouse akkor is importálási/exportálási adatok Azure Blob Storage-ból, és az Azure Data Lake Store. A PolyBase jelenleg a leggyorsabb eljárás az SQL Data Warehouse-adatok importálása a.

- **Hadoop parancssori**. Ha egy HDInsight-fürt átjárócsomópontjába lévő adatok, használhatja a `hadoop -copyFromLocal` parancs futtatásával másolja a fürthöz csatlakoztatott tárolási, például az Azure Storage-blobba vagy az Azure Data Lake Store. A Hadoop parancs használatához meg kell először a átjárócsomópontjához csatlakozik. A csatlakozás után egy fájlt tölthet tárhelyre.

## <a name="graphical-interface"></a>Grafikus felület

Vegye figyelembe az alábbiakat, ha csak néhány fájlt, vagy adatobjektumainak átadó és automatizálható a folyamat nem szükséges.

- **Az Azure Storage Explorer**. [Az Azure Tártallózó](https://azure.microsoft.com/features/storage-explorer/) platformfüggetlen eszköz, amellyel kezelheti az Azure storage-fiókok tartalmát. Ez lehetővé teszi, töltse le, kezelését és feltöltését blobokat, fájlok, várólisták, táblák és Azure Cosmos DB entitások. Segítségével az Blob storage szolgáltatással kezelheti a blobok és mappákat, valamint le- és feltöltése a BLOB storage-fiókok és a helyi rendszer és a Blob-tároló között.

- **Azure-portálon**. A Blob storage és a Data Lake Store egy webes felületet biztosít az fájlok felfedezése és a feltöltése az új fájlokat egyenként. Ez az jó választás, ha nem szeretné telepíteni az olyan eszközöket, vagy parancsok segítségével gyorsan megismerheti a fájlokat, vagy egyszerűen csak töltse fel az új adatbázisra skálázhatja kiadását.

## <a name="data-pipeline"></a>Adatfolyamat

**Az Azure Data Factory**. [Az Azure Data Factory](/azure/data-factory/) egy felügyelt szolgáltatás, ajánlott rendszeresen közti fájlátvitelre szolgáló több Azure-szolgáltatásokat, helyszíni vagy a kettő kombinációja megfelelő. Azure Data Factory használatával hozhat létre és ütemezése az adatvezérelt munkafolyamatok (más néven folyamatok), amely különálló adattároló származó adatok. feldolgozhatók és átalakíthatók az adatok különböző számítási szolgáltatások használatával (például Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics és Azure Machine Learning), Munkafolyamatokat az adatvezérelt [koordinálása](../technology-choices/pipeline-orchestration-data-movement.md) és adatmozgatást és a data transformation automatizálása.

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

Adatok átvitel forgatókönyvek esetén válassza ki az igényeinek megfelelő rendszer ezen kérdések megválaszolásával:

- Szüksége átadhat nagyon nagy mennyiségű adat, ahol az interneten keresztül kapcsolat időt vesz igénybe túl hosszú, így nem megbízható, vagy túl drága lehet? Ha igen, érdemes lehet fizikai átviteli.

- Szeretné parancsfájl-az átviteli feladatok, így az újrafelhasználható? Ha igen, válassza ki a parancssori kapcsolók és az Azure Data Factory.

- Nagyon nagy mennyiségű adat átvitele a hálózaton keresztül kell? Ha igen, kijelölés beállítási módját, amely big Data típusú adatok van optimalizálva.

- Szüksége átadhat adatokat vagy egy relációs adatbázisban a? Ha igen, válassza ki, amely egy vagy több rendszerű relációs adatbázisokat támogatja. Vegye figyelembe, hogy mindegyik lehetőség is szükséges Hadoop-fürthöz.

- Kell-e egy automatikus folyamat vagy munkafolyamat vezénylési? Ha igen, érdemes lehet az Azure Data Factory.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="physical-transfer"></a>Fizikai átviteli

| | Az Azure Import/Export szolgáltatás | Azure Data Box |
| --- | --- | --- |
| Űrlap tényező | Belső SATA merevlemez vagy értékpapír | Biztonságos, hamisíthatatlan, egyetlen hardver készülék |
| A Microsoft felügyeli szállítási logisztikai | Nem | Igen |
| Integrálható a partner termékek | Nem | Igen |
| Egyéni készülék | Nem | Igen |

### <a name="command-line-tools"></a>A parancssori eszközök

**Hadoop/HDInsight**

| | Ból a Distcp | Sqoop | Hadoop parancssori felület |
| --- | --- | --- | --- |
| A big Data típusú adatok optimalizált | Igen | Igen |  Igen |
| Relációs adatbázis másolása |  Nem | Igen | Nem |
| Relációs adatbázis másolása |  Nem | Igen | Nem |
| A Blob storage másolása |  Igen | Igen | Igen |
| A Blob storage másolása | Igen |  Igen | Nem |
| Data Lake Store-bA másolása | Igen | Igen | Igen |
| Másolja a Data Lake Store-ból | Igen | Igen | Nem |

**Other**

| | Azure CLI | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Kompatibilis platformokon | Linux, OS X, Windows | Linux, Windows | Windows | Linux, OS X, Windows | SQL Server, Azure SQL Data Warehouse | 
| A big Data típusú adatok optimalizált | Nem | Nem | Nem | Igen <sup>1</sup> | Igen <sup>2</sup> |
| Relációs adatbázis másolása | Nem | Nem | Nem | Nem | Igen | 
| Relációs adatbázis másolása | Nem | Nem | Nem | Nem | Igen | 
| A Blob storage másolása | Igen | Igen | Igen | Nem | Igen | 
| A Blob storage másolása | Igen | Igen | Igen | Igen | Igen |
| Data Lake Store-bA másolása | Nem | Nem | Igen | Igen |  Igen | 
| Másolja a Data Lake Store-ból | Nem | Nem | Igen | Igen | Igen | 


[1] AdlCopy megfelelően lett optimalizálva, az adatoknak a big Data Lake Analytics-fiók használata esetén.

[2] PolyBase [növelhető a teljesítmény](/sql/relational-databases/polybase/polybase-guide#performance) terjesztése a számítási és a Hadoop és használatával [PolyBase kibővített csoportok](/sql/relational-databases/polybase/polybase-scale-out-groups) SQL Server-példány és a Hadoop-csomópontjai közötti párhuzamos adatforgalom engedélyezéséhez.

### <a name="graphical-interface-and-azure-data-factory"></a>Grafikus felület és az Azure Data Factory

| | Azure Storage Explorer | Azure-portálon * | Azure Data Factory |
| --- | --- | --- | --- |
| A big Data típusú adatok optimalizált | Nem | Nem | Igen | 
| Relációs adatbázis másolása | Nem | Nem | Igen |
| Relációs adatbázis másolása | Nem | Nem | Igen |
| A Blob storage másolása | Igen | Nem | Igen |
| A Blob storage másolása | Igen | Nem | Igen |
| Data Lake Store-bA másolása | Nem | Nem | Igen |
| Másolja a Data Lake Store-ból | Nem | Nem | Igen |
| Töltse fel a Blob-tároló | Igen | Igen | Igen |
| Data Lake Store feltöltése | Igen | Igen | Igen |
| Adatátviteli levezényelni | Nem | Nem | Igen |
| Egyéni adatátalakítást | Nem | Nem | Igen |
| Díjszabási modell | Ingyenes | Ingyenes | Használati / kell fizetnie |

\* Azure-portálon ebben az esetben azt jelenti, hogy a Blob-tároló és a Data Lake Store feltárása web-alapú eszközökkel.

