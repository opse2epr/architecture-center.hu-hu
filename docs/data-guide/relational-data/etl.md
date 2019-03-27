---
title: Kinyerés, átalakítás és betöltés (ETL)
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 1551736d8ef3d2b82eb0a2fdb626330798ec1c65
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299251"
---
# <a name="extract-transform-and-load-etl"></a>Kinyerés, átalakítás és betöltés (ETL)

Gyakori probléma, hogy a szervezetek között, hogyan összegyűjtéséhez az adatok több adatforrásokat, többféle formátumban, és helyezze át egy vagy több adatokat tárol. A cél nem lehet adattár ugyanolyan típusú, mint a forrás, és gyakran a formátum a következő különböző kell alakú, vagy a végső rendeltetési-be való betöltés előtt törölni kell az adatokat.

Különböző eszközök, szolgáltatások és folyamatok fejlesztettek ki az évek, hogy segítse ezek a kihívások. Függetlenül attól, hogy a használt folyamat koordinálja a munkahelyi és a alkalmazni az adatok folyamaton belül átalakítását bizonyos fokú közös szükség van. A következő szakaszok kiemelnek a gyakran használt módszerek a műveletek végrehajtásához használt.

## <a name="extract-transform-and-load-etl-process"></a>A kinyerési, átalakítási és betöltési (ETL) folyamat

Kinyerési, átalakítási és betöltési (ETL) egy adatfolyamat használt különböző forrásokból származó adatok összegyűjtése, üzleti szabályok szerint az adatok átalakítása és betöltése a célként megadott adattárba. Az ETL átalakítási munka történik, egy speciális motor, és gyakran magában foglalja a használatával ideiglenesen fenntartási adatok előkészítési táblák folyamatban van átalakíthatók, és végső soron a rendeltetési betöltve.

Az Adatátalakítási, amely általában magában foglalja a különféle műveleteket, például szűrése, rendezése, összesítése, való csatlakozás a data, adattisztítást, deduplikálása és adatok érvényesítéséhez.

![Kinyerési, átalakítási-betöltési (ETL) folyamat](../images/etl.png)

Az ETL három fázisra gyakran időt takaríthat meg párhuzamosan futnak. Például adatok kibontása van folyamatban, amíg egy átalakítási folyamat sikerült dolgozik a már fogadott adatok és előkészíthető a betöltési, és a betöltési folyamat elkezdheti az előkészített adatokat a nem várja meg a teljes kinyerési folyamat befejezéséhez.

Kapcsolódó Azure-szolgáltatás:

- [Az Azure Data Factory v2-ben](https://azure.microsoft.com/services/data-factory/)

Más eszközök:

- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="extract-load-and-transform-elt"></a>A kinyerési, betöltési és átalakítási (ELT)

A kinyerési, betöltési és átalakítási (ELT) eltér a ETL kizárólag a Ha az átalakítás történik. A ELT folyamat az átalakítást a célzott adattár történik. Egy külön átalakító motor helyett a célzott adattár feldolgozási funkcióinak használhatók alakíthat át adatokat. Ez az architektúra az átalakító motor eltávolítása az adatcsatornából egyszerűbbé teszi. Ez a megközelítés egy másik előnye, hogy a célzott adattár skálázási is méretezhető ELT folyamat teljesítményét. Azonban ELT csak akkor működik jól Ha elég erős az adatok átalakításához hatékonyan-e a célrendszeren.

![Kinyerési-terhelés-átalakítási (ELT) folyamat](../images/elt.png)

A big Data típusú adatok tartomány ELT a tipikus használati esetek tartoznak. Például előfordulhat, hogy lépésként a forrásadatok egybesimított fájlokba, például a Hadoop elosztott fájlrendszer (HDFS) méretezhető storage-ban vagy az Azure Data Lake Store az összes kibontása. Technológiák, például a Spark, Hive és a PolyBase majd az adatok lekérdezéséhez használható. A lényeg az ELT, hogy az átalakítás végrehajtásához használt adattárak, végső soron használt adatok ugyanabban az adattárban. Az adattár közvetlenül a méretezhető tárhely, ahelyett, hogy az adatok betöltése a saját szellemi tulajdont képező storage-bA olvassa be. Ez a módszer az adatok másolása lépés szerepel az ETL-kihagyja az is lehet, amely egy nagy méretű adatkészletekhez időigényes művelet.

A gyakorlatban a célzott adattár van egy [adatraktár](./data-warehousing.md) egy Hadoop-fürtöt (a Hive- vagy Spark) és a egy SQL Data Warehouse használatával. Általában egy sémát az egybesimított fájl adatokon lekérdezéskor átfedésben és táblázatként, az adatokat lehet lekérdezni, mint bármely más táblázat a data Store engedélyezése tárolja. Ezek neve, a külső táblákra, mert az adatok nem találhatók az adatok által felügyelt tárolási tárolására magát, de néhány külső tárhelyen, méretezhető.

Az adattár csak kezeli a séma az adatok, és alkalmazza a séma olvasása. Például a Hive használatával egy Hadoop-fürt le egy Hive-táblába, ahol az adatforrás lényegében a fájlokat a HDFS elérési útját. Az SQL Data Warehouse PolyBase ugyanaz az eredmény érhető el &mdash; külsőleg magában az adatbázisban tárolt adatokon, tábla létrehozása. Az adatok betöltése után a külső táblák adatok lehet feldolgozni az adattár képességeit. A big data jellegű alkalmazási Ez azt jelenti, hogy az adattár nagymértékben párhuzamos feldolgozási (MPP), amely működésképtelenné válik az adatok szeletekre, és párhuzamosan több gép között osztja el a feldolgozási az adattömbök képesnek kell lennie.

Utolsó szakaszában az ELT folyamatok általában, hogy az adatok átalakítása a végleges formátumra, amely a típusú támogatott lekérdezések esetében hatékonyabb. Például előfordulhat, hogy lehet particionálni az adatokat. ELT is, Parquet, amely tárolja az adatokat soralapú Oszlopalapú módon és optimalizált providess indexelő tárolás tartalomfolyamokat használhatja.

Kapcsolódó Azure-szolgáltatás:

- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight Hive-val](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Az Azure Data Factory v2-ben](https://azure.microsoft.com/services/data-factory/)
- [A HDInsight Oozie](/azure/hdinsight/hdinsight-use-oozie-linux-mac)

Más eszközök:

- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="data-flow-and-control-flow"></a>Az adatfolyam és vezérlés folyamata

Adatfolyamatok kontextusában átvitelvezérlés biztosítja a szabályos feldolgozása feladatokhoz. Ezeket a feladatokat a megfelelő feldolgozási sorrendben kényszerítéséhez prioritású korlátozások szolgálnak. Ezek a korlátozások a munkafolyamat-diagramokon összekötőkként az alábbi képen látható módon is felfoghatók. Minden tevékenység rendelkezik egy serkenti az eredményt, például sikeres, sikertelen vagy befejezését. Bármely ezt követő feladat nem kezdeményezni, amíg az elődjéhez, ezeket a kimeneteket egyik befejeződött.

Vezérlő folyamatok adatfolyamok feladatként hajtható végre. Egy adatfolyam-feladathoz, a adatok egy forrásból kinyert, alakította át, vagy betöltése egy adattárba. Egy adatfolyam-feladat kimenete a következő adatfolyam-feladat bemenete, és adatokat flowss is futtatható egyszerre. Ellentétben a control flow nem adhat hozzá korlátozásokat az adatfolyam a tevékenységek között. Tekintse át az adatokat az egyes tevékenységek által feldolgozás során az adatokat megjelenítő azonban hozzáadhatja.

![A Control Flow belül feladatként végrehajtása az adatfolyam](../images/control-flow-data-flow.png)

A fenti ábrán vannak átvitelvezérlés, amelyek egyike egy adatfolyam-feladat belül több feladat. Az egyik feladat van beágyazva egy tárolót. Tárolók struktúrát feladatokhoz, így az adott munkaegység használható. Ilyen például az ismétlődő elemeket, például egy mappa vagy adatbázis utasításokban a gyűjteményen belül van.

Kapcsolódó Azure-szolgáltatás:

- [Az Azure Data Factory v2-ben](https://azure.microsoft.com/services/data-factory/)

Más eszközök:

- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Online tranzakciófeldolgozási (OLTP) adattárak](./online-transaction-processing.md#oltp-in-azure)
- [Online analitikus feldolgozási (OLAP) adattárak](./online-analytical-processing.md#olap-in-azure)
- [Az adatraktárak](./data-warehousing.md)
- [Vezénylési folyamat](../technology-choices/pipeline-orchestration-data-movement.md)

## <a name="next-steps"></a>További lépések

A következő referenciaarchitektúrákat teljes körű ELT folyamatok megjelenítése az Azure-ban:

- [Vállalati bi-ban az Azure SQL Data Warehouse-ban](../../reference-architectures/data/enterprise-bi-sqldw.md)
- [Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban](../../reference-architectures/data/enterprise-bi-adf.md)