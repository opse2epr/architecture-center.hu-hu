---
title: Hibrid ETL a meglévő helyszíni SSIS és az Azure Data Factory
description: Hibrid ETL a meglévő helyszíni SQL Server Integration Services (SSIS)-telepítések és az Azure Data Factory
author: alhieng
ms.date: 9/20/2018
ms.openlocfilehash: c4c0cfd63ef1d6c620eb36e16622ad9ffb7b5d80
ms.sourcegitcommit: 16bc6a91b6b9565ca3bcc72d6eb27c2c4ae935e4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/28/2018
ms.locfileid: "52552608"
---
# <a name="hybrid-etl-with-existing-on-premises-ssis-and-azure-data-factory"></a>Hibrid ETL a meglévő helyszíni SSIS és az Azure Data Factory

Szervezetek számára, amelyek az SQL Server-adatbázisok migrálása a felhőbe is vegye figyelembe, áttekintse költségmegtakarítást, teljesítménynövekedést, további rugalmasságot és méretezhető. Azonban finomhangolásával meglévő kinyerési, átalakítási és betöltési (ETL) folyamat készült SQL Server Integration Services (SSIS) lehet egy áttelepítési roadblock. Egyéb esetben az adatok betöltése a folyamathoz az szükséges összetett logikát és/vagy az adott eszköz összetevői, amelyek még nem támogatott az Azure Data Factory v2 (ADF). A gyakran használt SSIS többek között az intelligens keresési és intelligens csoportosítási átalakítások, a módosítási adatok rögzítése (CDC), a lassú módosítása dimenziók (. SCD) és a Data Quality Services (DQS).

Megkönnyítése érdekében egy lift-and-shift áttelepítését egy meglévő SQL database, egy hibrid ETL megközelítés lehet, hogy a leginkább megfelelő lehetőséget. A hibrid megközelítés ADF használja elsődleges vezénylési motorként, de továbbra is kihasználhatja a meglévő SSIS-csomagok az adatok és a helyszíni erőforrásokkal. Ez a megközelítés az ADF SQL Server beépített modul (IR) használatával egy lift-and-shift meglévő adatbázisok engedélyezése a felhőbe meglévő kódot és az SSIS-csomagok használata során.

Ebben a példaforgatókönyvben a szervezeteknek, amelyek adatbázisokat a felhőbe áthelyezni, és az ADF használatával motorként az elsődleges felhő alapú ETL során a meglévő SSIS-csomagok beépítése az új felhőalapú munkafolyamat releváns. Számos szervezet rendelkezik jelentős befektettek SSIS ETL csomagok adott feladatok fejlesztése. Ezek a csomagok újraírását ijesztőnek tűnhet. Meglévő kódot csomagok számát is, függőségekkel rendelkeznek a helyi erőforrások, a felhőbe történő áttelepítés megakadályozza.

ADF lehetővé teszi, hogy az ügyfelek kihasználhatják a meglévő ETL-csomagjaikat, eközben helyszíni ETL fejlesztési további befektetése jelenti. Ebben a példában ismerteti a lehetséges alkalmazási helyzetei kihasználva a meglévő SSIS-csomagok használata Azure Data Factory v2 új felhőalapú munkafolyamat részeként.

## <a name="potential-use-cases"></a>A lehetséges alkalmazási helyzetek

SSIS hagyományosan ETL eszközben dolgozhat átalakítását és betöltését számos SQL Server data szakemberek számára volt. Egyes esetekben SSIS adott funkció díjával vagy külső csatlakoztatás összetevők használták felgyorsíthatja a fejlesztési folyamatot. Csere vagy átalakítását, ezeket a csomagokat nem lehet olyan beállítás, amely megakadályozza, hogy a felhasználók saját adatbázisaikat a felhőbe való migrálás. Ügyfelek kis hatású megközelítés a meglévő adatbázisok migrálása a felhőbe, és kihasználni a meglévő SSIS-csomagok keres.

Számos lehetséges helyi használati esetek az alábbiak:

* Egy adatbázis-elemzés hálózati útválasztó naplók betöltése.
* Foglalkoztatás adatok az emberi erőforrások előkészítése az analitikai jelentés.
* Termék- és értékesítési adatok betöltése az értékesítési előrejelzésére egy data warehouse-bA.
* Be- vagy működési tárolók vagy adatok adattárházakkal, amelyeknél a Pénzügy és könyvelés automatizálása.

## <a name="architecture"></a>Architektúra

![Azure Data Factory használatával az ETL-folyamattal hibrid architektúra áttekintése][architecture-diagram]

1. Az adat-előállító Data származtatható Blob storage-ból.
2. A Data Factory-folyamatot meghív egy tárolt eljárást az SSIS-feladatok végrehajtásához a helyileg üzemeltetett az integrált modulon keresztül.
3. Az adatok előkészítése az alárendelt használati az adattisztító feladatok végrehajtása.
4. Miután az adattisztító feladat sikeresen befejeződik, egy másolási tevékenység a tiszta adatok betöltése az Azure-bA hajtja végre.
5. Adatok tisztítása majd betöltött táblák az SQL Data warehouse.

### <a name="components"></a>Összetevők

* [A BLOB storage-] [ docs-blob-storage] tárolja a fájlokat és a egy Data Factoryben az adatok forrásaként.
* [SQL Server Integration Services] [ docs-ssis] tartalmazza a helyszíni ETL feladat jellemző számítási feladatok végrehajtására szolgáló csomagok.
* [Az Azure Data Factory] [ docs-data-factory] a felhőalapú vezénylési motor, amely beolvassa az adatokat több forrásból származó és ötvözi, koordinálja és betölti az adatokat egy adattárházba.
* [Az SQL Data Warehouse] [ docs-sql-data-warehouse] központosítja standard ANSI SQL-lekérdezések használatával egyszerűen hozzáférhetnek a felhőbeli adatok.

### <a name="alternatives"></a>Alternatív megoldások

A Data Factory adatokat tisztítására eljárások egyéb technológiák, például egy Databricks-jegyzetfüzet, Python-szkriptet vagy egy virtuális gépen futó SSIS-példánya használatával implementált tudta meghívni. [Telepíti a fizetős verzióra, vagy egyéni összetevők az Azure-SSIS integrációs modul licencelt](/azure/data-factory/how-to-develop-azure-ssis-ir-licensed-components) egy működőképes alternatívája lehet a hibrid megközelítés lehet.

## <a name="considerations"></a>Megfontolandó szempontok

A beépített modul (IR) két modell támogatja: saját üzemeltetésű integrációs Modult vagy az Azure-ban üzemeltetett integrációs Először határozza meg, a két lehetőség között. Helyi tároló költséghatékonyabb, de több terhelést jelenthet a karbantartási és felügyeleti. További információkért lásd: [helyi integrációs modul](/azure/data-factory/concepts-integration-runtime#self-hosted-integration-runtime). Ha segítségre van használatáról integrációs modul meghatározása [használandó integrációs modul meghatározása](/azure/data-factory/concepts-integration-runtime#determining-which-ir-to-use).

Az Azure-ban üzemeltetett megoldás meg kell határoznia, mekkora teljesítményre van szükség az adatok feldolgozásához. Az Azure-ban üzemeltetett konfiguráció lehetővé teszi a virtuális gép méretének kiválasztásához a konfigurációs lépések részeként. Virtuálisgép-méret kiválasztásával kapcsolatban további tudnivalókért lásd: [virtuális gépek teljesítményével kapcsolatos megfontolások](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

A döntés akkor sokkal egyszerűbb, ha már rendelkezik meglévő SSIS-csomagok, például az adatforrások vagy a fájlokat, amelyek nem érhetők el az Azure-ból a helyszíni függőséggel rendelkező. Ebben az esetben egyetlen lehetősége a saját üzemeltetésű Ez a megközelítés a vezénylési motorként, meglévő csomagok újraírása nélkül használhatja a felhőt a legnagyobb rugalmasságot biztosít.

Végső soron a célja a feldolgozott adatok áthelyezése a felhőbe további pontosítás vagy egyéb adatokkal, a felhőben tárolt csoportba foglalása. A tervezési folyamat részeként a nyomon követheti, az ADF folyamatokban használt tevékenységek száma. További információkért lásd: [az Azure Data Factory folyamatai és tevékenységei](/azure/data-factory/concepts-pipelines-activities).

## <a name="pricing"></a>Díjszabás

Az Azure Data Factory egy költséghatékony megoldás készítheti elő az adatáthelyezést a felhőben. A költségek a számos tényező befolyásolja alapul.

* Folyamat végrehajtások száma
* A folyamaton belül használt entitások/tevékenységek száma
* Szám figyelési műveletek
* (Azure-ban üzemeltetett integrációs modul vagy a saját üzemeltetésű integrációs modul) integrációs futtatások száma

Az ADF használ a fogyasztás alapú számlázáshoz. Költség folyamat-végrehajtás és a figyelés során, ezért csak felmerült. A végrehajtás alapvető folyamat akár 50 cent és a figyelés akár 25 cent lenne költség. A [Azure költségkalkulátor](https://azure.microsoft.com/pricing/calculator/) segítségével hozzon létre egy pontosabb becslést, az adott számítási feladatok alapján.

Ha egy hibrid ETL számítási feladatot futtat, figyelembe kell venni az SSIS-csomagok üzemeltetéséhez használt virtuális gép költségét. A virtuális gép és a egy D1v2 mérete alapján ez a költség (1 mag, 3,5 GB RAM, 50 GB-os lemezt) való E64V3 (64 mag, 432 GB RAM, 1600 GB-os lemezt).  Ha szüksége további útmutatást a kiválasztott virtuális gép mérete megfelelő, tekintse meg [virtuális gépek teljesítményével kapcsolatos megfontolások](/azure/cloud-services/cloud-services-sizes-specs#performance-considerations).

## <a name="next-steps"></a>További lépések

* Tudjon meg többet [Azure Data Factory](https://azure.microsoft.com/services/data-factory/).
* Ismerkedés az Azure Data Factory a következő a [részletes oktatóanyag](/azure/data-factory/#step-by-step-tutorials).
* [Az Azure-SSIS integrációs modul Azure-beli adat-előállító üzembe helyezése](/azure/data-factory/tutorial-deploy-ssis-packages-azure).

<!-- links -->
[architecture-diagram]: ./media/architecture-diagram-hybrid-etl-with-adf.png
[small-pricing]: https://azure.com/e/
[medium-pricing]: https://azure.com/e/
[large-pricing]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-blob-storage]: /azure/storage/blobs/
[docs-data-factory]: /azure/data-factory/introduction
[docs-resource-groups]: /azure/azure-resource-manager/resource-group-overview
[docs-ssis]: /sql/integration-services/sql-server-integration-services
[docs-sql-data-warehouse]: /azure/sql-data-warehouse/sql-data-warehouse-overview-what-is