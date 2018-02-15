---
title: "A folyamat vezénylési technológiát kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 17aeb871bc815793295ed610795e5e83de72c637
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>A folyamat vezénylési technológiát kiválasztása az Azure-ban

A big data-megoldások ismételt adatfeldolgozási műveletek, a munkafolyamatok beágyazott áll. A feldolgozási sor koordináló rendszer például olyan eszköz, amely segítségével automatizálhatja a ezek a munkafolyamatok. Az orchestrator feladatok ütemezése, munkafolyamatok hajtható végre, és összehangolják a tevékenységek között függőségek.

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>Mik a adatok folyamat előkészítése lehetőségei?

Az Azure, a következő szolgáltatások és eszközök core követelményeknek megfelelő folyamat vezénylési, folyamatábrán és adatmozgás:

- [Azure Data Factory](/azure/data-factory/)
- [A HDInsight Oozie](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

Ezek a szolgáltatások és eszközök is egymástól függetlenül használja, vagy hibrid megoldás létrehozásához használt együtt. Például a integrációs futásidejű (IR) az Azure Data Factory V2 natív módon végrehajtható SSIS-csomagok felügyelt Azure számítási környezetben. Amíg funkcióinak ezen szolgáltatások között átfedés van, néhány fontos különbség van.

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Szükséges big Data típusú adatok képességek az áttelepítése, valamint az adatok átalakítása? Általában ez azt jelenti, hogy az adatok több terabájt gigabájttól. Ha igen, majd szűkítheti a beállítások az adott big Data típusú adatok a legalkalmasabb.

- Szükség van egy felügyelt szolgáltatás, amely működhet léptékű? Ha igen, válassza ki a felhőalapú szolgáltatások, amelyek nem korlátozza a helyi feldolgozási kapacitás.

- Az adatok források a helyszínen közé tartoznak a? Ha igen, keresse meg, amely felhőalapú és a helyszíni adatforrások vagy célok használható beállítások.

- Egy HDFS fájlrendszer a Blob Storage tárolóban tárolja a forrásadatok? Ha igen, válassza ki, amely támogatja a Hive-lekérdezéseket.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="general-capabilities"></a>Általános képességek

| | Azure Data Factory | SQL Server Integration Services (SSIS) | A HDInsight Oozie
| --- | --- | --- | --- |
| Managed | Igen | Nem | Igen |
| Felhőalapú | Igen | Nem található (helyi) | Igen |
| Előfeltétel | Azure-előfizetés | SQL Server  | Azure-előfizetéssel, HDInsight-fürt |
| Felügyeleti eszközök | Azure-portálon PowerShell parancssori felület, a .NET SDK | SSMS, PowerShell | Bash rendszerhéjat, Oozie REST API-t Oozie webes felhasználói felület |
| Díjszabás | Használati / kell fizetnie | Licencelési / funkciók díj ellenében | Nem kell külön fizetni felett fut, a HDInsight-fürt |

### <a name="pipeline-capabilities"></a>Feldolgozási sor képességek

| | Azure Data Factory | SQL Server Integration Services (SSIS) | A HDInsight Oozie
| --- | --- | --- | --- |
| Adatok másolása | Igen | Igen | Igen |
| Egyéni átalakítások | Igen | Igen | Igen (MapReduce, a Pig és a Hive-feladatok) |
| Az Azure Machine Learning, pontozási | Igen | Igen (parancsfájl-kezelési) | Nem |
| A HDInsight-igény | Igen | Nem | Nem |
| Azure Batch | Igen | Nem | Nem |
| A Pig, a Hive, MapReduce | Igen | Nem | Igen |
| Spark | Igen | Nem | Nem |
| SSIS-csomagot | Igen | Igen | Nem |
| Átvitelvezérlés | Igen | Igen | Igen |
| Helyszíni adatok elérése | Igen | Igen | Nem |

### <a name="scalability-capabilities"></a>Méretezhetőség képességek

| | Azure Data Factory | SQL Server Integration Services (SSIS) | A HDInsight Oozie
| --- | --- | --- | --- |
| Vertikális felskálázás | Igen | Nem | Nem |
| Horizontális felskálázás | Igen | Nem | Igen (a fürt feldolgozó csomópontok hozzáadásával) |
| A big Data típusú adatok optimalizált | Igen | Nem | Igen |

