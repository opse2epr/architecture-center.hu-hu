---
title: Egy data pipeline vezénylési technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 76a101b76497ae2b2aacff973175bb0fe4703d9e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299219"
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>Egy data pipeline vezénylési technológia kiválasztása az Azure-ban

Legtöbb big data-megoldások állnak, ismétlődő adatfeldolgozási műveletekből, a munkafolyamatok áll. Egy folyamat koordináló rendszer olyan eszköz, amely segít a munkafolyamatok automatizálhatók. Az orchestrator feladatok ütemezése, a munkafolyamatok végrehajtásához, és koordinálja a feladatok függőségeit.

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>Mik azok a beállítások esetében az adatfolyamat előkészítéséért?

Az Azure-ban a következő szolgáltatások és eszközök felel meg az adatfolyamat előkészítéséért, átvitelvezérlés és adatáthelyezés alapvető követelményei:

- [Azure Data Factory](/azure/data-factory/)
- [A HDInsight Oozie](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

Ezek a szolgáltatások és eszközök is egymástól függetlenül használja, vagy egy hibrid megoldás létrehozásához használt együtt. Például az Integration Runtime (IR) az Azure Data Factory V2 natív módon futtathat SSIS-csomagok Azure-beli felügyelt számítási környezetben. Funkcióinak ezek a szolgáltatások között átfedés van, míg néhány fontos különbség van.

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szükség van a big data-funkciók webszolgáltatásai, és az adatok átalakítása? Általában ez azt jelenti, hogy több gigabájt terabájtra. Ha igen, majd szűkítheti a lehetőségek, amelyek a leginkább megfelelő megoldást big Data típusú adatok.

- Szükség van egy felügyelt szolgáltatás, amely nagy mennyiségű működhet? Ha igen, válasszon egy felhőalapú szolgáltatás, amely nem korlátozza a helyi feldolgozási teljesítményt.

- Néhány, az adatok források a helyszínen? Ha igen, keresse meg, amely felhőalapú és a helyszíni adatforrások vagy célok használható beállítások.

- A HDFS-fájlrendszer a Blob storage-ban tárolja a forrásadatok? Ha igen, válasszon egy beállítást, amely támogatja a Hive-lekérdezések.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek

| | Azure Data Factory | SQL Server Integration Services (SSIS) | A HDInsight Oozie
| --- | --- | --- | --- |
| Managed | Igen | Nem | Igen |
| Felhőalapú | Igen | Nincs (helyi) | Igen |
| Előfeltétel | Azure-előfizetés | SQL Server  | Azure-előfizetéssel, HDInsight-fürt |
| Felügyeleti eszközök | Azure Portal, PowerShell, CLI, .NET SDK | SSMS, PowerShell | A bash rendszerhéj, az Oozie REST API-t, az Oozie webes felhasználói felületen |
| Díjszabás | Fizessen a használat | Licencelési / szolgáltatások kell fizetnie | Külön díj nélküli felett fut, a HDInsight-fürt |

### <a name="pipeline-capabilities"></a>Folyamat képességek

| | Azure Data Factory | SQL Server Integration Services (SSIS) | A HDInsight Oozie
| --- | --- | --- | --- |
| Adatok másolása | Igen | Igen | Igen |
| Egyéni átalakítások | Igen | Igen | Igen (a MapReduce, a Pig és a Hive-feladatok) |
| Azure Machine Learning scoring | Igen | Igen (szkriptek) | Nem |
| HDInsight igény szerinti | Igen | Nem | Nem |
| Azure Batch | Igen | Nem | Nem |
| Pig, Hive, MapReduce | Igen | Nem | Igen |
| Spark | Igen | Nem | Nem |
| SSIS-csomag végrehajtása | Igen | Igen | Nem |
| Átvitelvezérlés | Igen | Igen | Igen |
| Helyszíni adatok elérése | Igen | Igen | Nem |

### <a name="scalability-capabilities"></a>Skálázhatósági képességeket.

| | Azure Data Factory | SQL Server Integration Services (SSIS) | A HDInsight Oozie
| --- | --- | --- | --- |
| Vertikális felskálázás | Igen | Nem | Nem |
| Horizontális felskálázás | Igen | Nem | Igen (a fürt munkavégző csomópontok hozzáadásával) |
| Big Data-optimalizált | Igen | Nem | Igen |

