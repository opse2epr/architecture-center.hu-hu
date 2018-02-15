---
title: "Kinyerési, átalakítási és betöltési (ETL)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: a980c1f8aef99fc263083e5e496b1340204f7dac
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="extract-transform-and-load-etl"></a>Kinyerési, átalakítási és betöltési (ETL)

A szervezetek szembesülhetnek gyakori probléma az adatgyűjtés több adatok több formátumban adatforrásokat, és helyezze az egy vagy több adatok tárolási módja. A cél nem lehet adattár ugyanolyan típusú, mint a forráskiszolgálón, és gyakran a formátum nem egyezik, vagy alakú, vagy az azokat a végső rendeltetési hely betöltése előtt törölni kell az adatokat.

Különböző eszközök, szolgáltatások és folyamatok ezekkel a kihívásokkal fejlesztettek az évek cím segítségével. Függetlenül attól, a folyamat használja szükség van a közös koordinálja a munkahelyi és bizonyos fokú adatok átalakítása a adatok feldolgozási folyamat belül érvényesek. A következő szakaszok kiemelnek gyakran a feladatok végrehajtásához használt módszer.

## <a name="extract-transform-and-load-etl"></a>Kinyerési, átalakítási és betöltési (ETL)

Kinyerési, átalakítási és betöltési (ETL) egy adatok feldolgozási folyamat különböző forrásokból származó adatok összegyűjtése, az adatok üzleti szabályok szerint, és töltse be a cél-tárolóban. Az átalakítási munkahelyi az ETL történik, egy speciális motor, és gyakran a ideiglenesen fenntartási adatok átmeneti tárolási táblák használata, mert folyamatban van szükséges át legyenek-e, és végső soron a rendeltetési betölteni.

Az adatok átalakítása, általában akkor történik, magában foglalja a különböző műveletek, például a szűrési, rendezési, összesítése, adatok csatlakoztatása, adattisztításon, deduplikálása és az adatok.

![Kivonat-átalakítási-betöltési (ETL) folyamat](./images/etl.png)

ETL folyamat három szakaszból gyakran, időt takaríthat párhuzamosan futnak. Például adatok kibontása folyamatban van, amíg egy átalakítási folyamat is működik-e a már fogadott adatok és előkészíthető a betöltése, és egy betöltése folyamat elkezdheti az előkészített adatok a ahelyett, hogy a teljes kinyerési folyamat befejezésére történő várakozáskor végezze el.

Megfelelő Azure szolgáltatás:
- [Az Azure Data Factory v2](https://azure.microsoft.com/services/data-factory/)

Más eszközök:
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="extract-load-and-transform-elt"></a>Kinyerési, betöltés és átalakítás (ELT)

Kinyerési, betöltés és átalakítás (ELT) eltér az ETL kizárólag a ahol az átalakítás történik. Az átalakítás a ELT sorban, a cél adattár következik be. Egy külön átalakító motor helyett a feldolgozási képességek a cél adattár segítségével adatok. Ez leegyszerűsíti a architektúra az átalakító motor eltávolításával a láncból. Ezt a módszert használja egy másik előnye az, hogy a cél adattár skálázás is méretezze át a ELT csővezeték teljesítmény. Azonban ELT csak akkor működik jól esetén elég erős hatékonyan átalakíthatja az adatokat a célrendszeren.

![Kivonat-betöltési-átalakítás (ELT) folyamat](./images/elt.png)

A big Data típusú adatok tartomány ELT a tipikus használati esetek tartoznak. Például előfordulhat, hogy megkezdéséhez valamennyi egybesimított fájlokba, a Hadoop elosztott fájlrendszer (HDFS) például méretezhető tárolás a forrásadatok és az Azure Data Lake Store kinyeréséhez. Technológiák, például a Spark, a Hive és a PolyBase használható az adatok lekérdezésére. A kulcs ELT pontra az, hogy az átalakítás végrehajtásához használt adattár a ugyanazt az adattárat, ahol végső soron használt adatok. Az adattároló közvetlenül a méretezhető tárolás, az adatok betöltése a saját védett tároló helyett olvassa be. Ez a megközelítés kihagyja az adatok másolása lépés ETL, szerepel, amely lehet egy nagy méretű adatkészletekhez időigényes művelet.

A gyakorlatban a cél adattár van egy [adatraktár](./data-warehousing.md) a Hadoop-fürtök (a Hive vagy Spark használatával) vagy egy SQL Data Warehouse használatával. Általában a séma, időben a strukturálatlan fájladatok az átfedett és a rendszer táblaként, engedélyezi az adatok kérdezhető le, mint a szerepel az adattárban. Ezek hivatkozunk, külső táblákra mert az adatok nem kezeli az adatokat tároló tárolja saját magát, de az egyes külső méretezhető tárolókhoz. 

Az adattár csak a Adatséma kezeli, és alkalmazza a séma olvasási. Például a Hive eszközzel Hadoop-fürtök le olyan Hive táblát, ahol az adatforrás az hatékonyan a fájlokat a HDFS elérési útját. Az SQL Data Warehouse PolyBase érhető el ugyanazt az eredményt &mdash; külsőleg maga az adatbázis tárolt adatok alapján a táblázatok létrehozásáról. Miután a forrásadatok be van töltve, levő adatok külső táblái feldolgozási az adattár képességek segítségével. A big data forgatókönyvéhez Ez azt jelenti, hogy az adattár képesnek kell lennie a nagymértékben párhuzamos feldolgozási (MPP), amely az adatok bontja kisebb csoportjai, majd továbbítja az adattömbök feldolgozása párhuzamosan több számítógépen.

A ELT folyamatának utolsó szakaszában általában az adatok átalakítására végleges formátumra, amely támogatja lekérdezések típusú hatékonyabb. Például az adatok lehetnek particionálva. Emellett ELT Parquet, amely tárolja az adatokat soros oszlopos módon és optimalizált providess indexelő optimalizált tárolási formátumok előfordulhat, hogy használja. 

Megfelelő Azure szolgáltatás:

- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [A Hive HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [Az Azure Data Factory v2](https://azure.microsoft.com/services/data-factory/)
- [A HDInsight Oozie](/azure/hdinsight/hdinsight-use-oozie-linux-mac)

Más eszközök:

- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="data-flow-and-control-flow"></a>Az adatfolyamnak és folyamata

Adatok folyamatok keretében a folyamatábrán feladatokhoz rendezett feldolgozása biztosítja. Ezeket a feladatokat a megfelelő feldolgozási sorrendben kényszerítéséhez sorrend megkötések használata. Az eltolásokat tekintheti összekötők a munkafolyamat-diagrammal, ezek a megkötések az alábbi ábrán látható módon. Minden tevékenység rendelkezik az eredménye, például sikeres, sikertelen vagy létrehozása után. A következő feladatokat nem kezdeményez a folyamatot, amíg az elődjéhez művelete befejeződött, ezek az eredmények közül.

Vezérlő adatfolyamok adatfolyamok feladatként hajtható végre. Az adatfolyam-feladathoz adatok kibontani a forrás, át legyenek-e, vagy töltse be a tárolóban. Egy adatfolyam-feladat kimenete a következő adatfolyam-feladat bemeneti, és adatok flowss párhuzamosan futtatható. Eltérően vezérlő adatfolyamok nem adhat hozzá korlátozásokat az adatfolyam a tevékenységek között. Hozzáadhat azonban egy adatokat megjelenítő, és figyelje meg az adatokat, minden feladat feldolgozása.

![Adatfolyam-az belül vezérlő egymást követő feladatok végrehajtása zajlik](./images/control-flow-data-flow.png)

A fenti ábrán vannak a folyamatábrán, melyek egyike adatfolyam-feladathoz belül több feladatot. A feladatok közül van beágyazva egy tárolót. Tárolók segítségével feladatok, így munkaegység alapot biztosítanak. Ilyen például az ismétlődő elemeket, például egy mappa vagy adatbázis utasításokban fájlokat egy gyűjteményen belül van.

Megfelelő Azure szolgáltatás:
- [Az Azure Data Factory v2](https://azure.microsoft.com/services/data-factory/)

Más eszközök:
- [SQL Server Integration Services (SSIS)](/sql/integration-services/sql-server-integration-services)

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Online tranzakciófeldolgozási (OLTP) adattároló](../technology-choices/oltp-data-stores.md)
- [Online analitikus feldolgozási (OLAP) adattároló](../technology-choices/olap-data-stores.md)
- [Az adatraktárak](../technology-choices/data-warehouses.md)
- [Vezénylési folyamat](../technology-choices/pipeline-orchestration-data-movement.md)
