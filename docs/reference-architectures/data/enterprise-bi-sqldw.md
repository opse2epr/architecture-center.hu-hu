---
title: Enterprise BI és SQL Data Warehouse
description: A relációs adatok az üzleti elemzéseket kaphat az Azure tárolja a helyszíni
author: MikeWasson
ms.date: 11/06/2018
ms.openlocfilehash: d5b680346267a17b5016b8897dc03ddcf18a7fe9
ms.sourcegitcommit: 02ecd259a6e780d529c853bc1db320f4fcf919da
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/07/2018
ms.locfileid: "51263813"
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>Enterprise BI és SQL Data Warehouse

Ez a referenciaarchitektúra valósít meg egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kinyerési, betöltési, átalakítási) folyamat, amely helyez át adatokat a helyszíni SQL Server-adatbázisból az SQL Data Warehouse-ba, és átalakítja az adatokat az elemzéshez. 

Az architektúra egy referenciaimplementációt érhető el az [GitHub][github-folder]

![](./images/enterprise-bi-sqldw.png)

**A forgatókönyv**: egy a szervezet rendelkezik a helyszíni SQL Server-adatbázisban tárolt nagy OLTP adatkészlet. A szervezet célja az SQL Data Warehouse használata a Power BI segítségével történő elemzését. 

Ez a referenciaarchitektúra egyszeri vagy igény szerinti feladat lett tervezve. Adatok áthelyezése (óránként vagy naponta) tartósan van szüksége, ha automatizált munkafolyamatokat az Azure Data Factory használatát javasoljuk. Egy referencia-architektúra, amely a Data Factory használja, lásd: [vállalati bi-ban az SQL Data Warehouse és az Azure Data Factory automatikus][adf-ra].

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

### <a name="data-source"></a>Adatforrás

**Egy SQL Server**. Az adatok helyszíni SQL Server-adatbázis található. A helyszíni környezet szimulálása, ez az architektúra üzembe helyezési szkriptjei virtuális gép létrehozása az Azure-ban telepített SQL Server. A [Wide World Importers OLTP mintaadatbázis] [ wwi] szolgál a forrásadatokat.

### <a name="ingestion-and-data-storage"></a>Adatfeldolgozási és az adatok tárolása

**A BLOB Storage-**. A BLOB storage segítségével egy átmeneti területre, másolja az adatokat, mielőtt betöltené azokat az SQL Data Warehouse-bA.

**Azure SQL Data Warehouse**. [Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer analytics végre, a nagy mennyiségű adat. Támogatja a nagy olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas nagy teljesítményű elemzési futtatásához. 

### <a name="analysis-and-reporting"></a>Elemzés és jelentéskészítés

**Az Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely adatmodellezési képességekkel. Analysis Services használatával, amely a felhasználó lekérdezheti a szemantikai modell létrehozása. Analysis Services különösen hasznos a forgatókönyvekben BI irányítópult. Ebben az architektúrában az Analysis Services a szemantikai modell feldolgozása a data warehouse-ból adatokat olvasó, és hatékonyan szolgál az irányítópult-lekérdezéseket. Gyorsabb lekérdezés-feldolgozás replikái horizontális felskálázásával rugalmas egyidejűségi is támogatja.

Jelenleg az Azure Analysis Services rendszerbeli táblázatos modellek, de nem többdimenziós modellek támogatja. Táblázatos modellek használata relációs modellezési hoz létre (táblák és oszlopok), mivel a többdimenziós modellek használata OLAP modellezési szerkezeteket (kockákat, dimenziók és mértékek). Ha többdimenziós modellek van szüksége, használja az SQL Server Analysis Services (SSAS). További információkért lásd: [táblázatos és többdimenziós megoldások összehasonlítása](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).

**Power BI**. A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese. Ebben az architektúrában a szemantikai modell az Analysis Servicesben tárolt kérdezi le.

### <a name="authentication"></a>Hitelesítés

**Az Azure Active Directory** (Azure AD) hitelesíti a felhasználók, akik a Power bi-ban az Analysis Services-kiszolgálóhoz csatlakozhat.

## <a name="data-pipeline"></a>Adatfolyamat
 
Ez a referenciaarchitektúra használja a [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) mintaadatbázis adatforrásként. Az adatfolyamatok rendelkezik a következő szakaszokban:

1. Az adatok exportálása az SQL Serverről egybesimított fájlokba (bcp segédprogram).
2. Az egybesimított fájlok másolása az Azure Blob Storage (AzCopy).
3. Adatok betöltése az SQL Data warehouse-ba (PolyBase).
4. Alakítsa át az adatokat egy csillag sémában (T-SQL).
5. A szemantikai modell betöltése az Analysis Services (SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> A lépéseket 1 &ndash; 3, fontolja meg a Redgate Data Platform Studio használatával. A Data Platform Studio a legmegfelelőbb kompatibilitási javításokat és optimalizálásokat alkalmazza, így ez a leggyorsabb mód az SQL Data Warehouse használatának megkezdésére. További információkért lásd: [adatok betöltése a Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate). 

A következő szakaszok ismertetik ezen szakaszokban részletesebben.

### <a name="export-data-from-sql-server"></a>Adatok exportálása az SQL Serverről

A [bcp](/sql/tools/bcp-utility) (tömeges másolási funkciójával) segédprogram egy gyorsan hozhat létre SQL-táblákban strukturálatlan szöveges fájlok. Ebben a lépésben az oszlopokat, amelyeket szeretne exportálni, választja, de nem alakíthatja át az adatokat. Bármely adatátalakítások az SQL Data Warehouse történjen.

**Javaslatok**

Ha lehetséges ütemezheti az éles környezetben az erőforrás-versengés minimalizálása érdekében csúcsidőn adatkinyerés. 

Kerülje a BCP használatával fut a kiszolgálón. Ehelyett futtassa azt egy másik gépről. Írás a fájlok egy helyi meghajtóra. Győződjön meg arról, hogy rendelkezik-e elegendő i/o-erőforrások kezeli az egyidejű írások. A legjobb teljesítmény érdekében dedikált gyors tárolóeszközöket exportálja a fájlokat.

A hálózati átvitel menti az exportált adatok Gzip formátumban tömörített felgyorsíthatja. Azonban az adatraktárban való betöltésének a tömörített fájlok, lassabb, mint a kibontott fájlok betöltése, így a kompromisszummal jár a gyorsabb hálózati átvitel helyett a gyorsabb betöltése között. Ha úgy dönt, a Gzip tömörítést szeretne használni, ne hozzon létre egy egyetlen Gzip fájl. Ehelyett az adatok felosztása több tömörített fájl.

### <a name="copy-flat-files-into-blob-storage"></a>Egybesimított fájlok másolása a blob storage-bA

A [AzCopy](/azure/storage/common/storage-use-azcopy) segédprogram készült nagy teljesítményű másolása az adatok Azure blob storage-bA.

**Javaslatok**

A storage-fiók létrehozása az adatok helye közelében található régiókban. A tárfiók és az SQL Data Warehouse-példányhoz ugyanabban a régióban helyezhet üzembe. 

Az AzCopy ugyanarra a gépre, amely az éles számítási feladatok nem futtatható, mert a Processzor- és i/o-felhasználás megzavarhatja az éles környezetbeli számítási feladatokra. 

A feltöltés először használatával megtekintheti a feltöltési sebesség például teszteléséhez. Az AzCopy a /NC beállítás használatával adja meg az egyidejű másolási műveletek száma. Indítsa el az alapértelmezett értékkel, majd ezt a beállítást, a teljesítmény hangolására kísérletezhet. Alacsony sávszélességű környezetben túl sok az egyidejű művelet túlterhelhetik futó a hálózati kapcsolat, és megakadályozza a műveletek sikeres befejezését.  

Az AzCopy helyez át adatokat tároló a nyilvános interneten keresztül. Ha ez nem elég gyors, érdemes beállítani egy [ExpressRoute](/azure/expressroute/) kapcsolatcsoport. Az ExpressRoute egy szolgáltatása, amely az adatok egy dedikált privát kapcsolaton keresztül irányítja az Azure-bA. Egy másik lehetőség, ha túl lassú, a hálózati kapcsolatot, fizikailag tehetnek elérhetővé az adatokat egy Azure-adatközpontba lemezen. További információkért lásd: [, és az Azure-ból történő adatátvitel](/azure/architecture/data-guide/scenarios/data-transfer).

A másolási művelet során AzCopy létrehoz egy ideiglenes journal-fájlt, amely lehetővé teszi az AzCopy segítségével indítsa újra a műveletet, ha azt (például hálózati hiba) miatt beolvasása megszakítva. Győződjön meg arról, hogy nincs elegendő lemezterület az adatbázisnapló-fájlok tárolására. A /Z beállítás használatával adja meg, ahol a naplófájlok íródnak.

### <a name="load-data-into-sql-data-warehouse"></a>Adatok betöltése az SQL Data Warehouse-ba

Használat [PolyBase](/sql/relational-databases/polybase/polybase-guide) betölteni a blob storage-ból a data warehouse-bA. A PolyBase úgy tervezték, kihasználhatja az SQL Data Warehouse, ami lehetővé teszi az adatok betöltése az SQL Data Warehouse leggyorsabban MPP (nagymértékben párhuzamos feldolgozási) architektúráját. 

Az adatok betöltése két lépésből áll:

1. Hozzon létre az adatok külső táblák egy készlete. A külső tábla esetében, amely az adatraktár-on kívül tárolt adatokat tábla definícióját &mdash; ebben az esetben a átalánydíjjal fájlok blob storage-ban. Ez a lépés nem helyezi át adatokat a warehouse-bA.
2. Átmeneti tárolási táblákat hozhat létre, és betöltheti az adatokat az átmeneti tárolási táblákba. Ebben a lépésben másolja az adatokat az adattárház.

**Javaslatok**

Amikor nagy mennyiségű (több mint 1 TB) adat rendelkezik, és a egy párhuzamosság kihasználó elemzési számítási feladatot futtat, vegye figyelembe az SQL Data warehouse-bA. Az SQL Data Warehouse nem jó megoldás lehet OLTP számítási feladatokat vagy kisebb adatkészletek (< 250GB). Adatkészletek kisebb, mint 250 GB-os fontolja meg az Azure SQL Database vagy SQL Server. További információkért lásd: [adattárházak](../../data-guide/relational-data/data-warehousing.md).

Hozzon létre az előkészítési táblák halomtáblák, amelyek nem lesznek indexelve. A lekérdezéseket, amely a termelési táblákat hozhat létre teljes tábla beolvasásával, eredményez, így az előkészítési táblák indexelése nem ok.

A PolyBase automatikusan kihasználja a párhuzamosság látható az adatraktárban. A betöltési teljesítmény arányosan növekszik, ahogy a Dwu növelését. A legjobb teljesítmény érdekében használjon egy egyetlen betöltési művelet. Nem jár a bemeneti adatok kompatibilitástörő adattömbökbe és több egyidejű betöltések futtatására teljesítmény előnnyel.

A PolyBase gzip formátumban tömörített fájlok olvashatja. Azonban csak egyetlen olvasó szolgál egy tömörített fájl, mert a fájl kibontása egy egyszálas működésre. Ezért kerülje a nagy tömörített fájl betöltése. Ehelyett az adatok felosztása több tömörített fájlok, párhuzamosság kihasználása érdekében. 

Vegye figyelembe a következő korlátozások vonatkoznak:

- A PolyBase támogatja egy oszlop maximális mérete `varchar(8000)`, `nvarchar(4000)`, vagy `varbinary(8000)`. Ha olyan adat, amely meghaladja a ezeket a korlátokat, az egyik lehetőség az felosztása az adatok adattömbökre exportálni, és ezután szétbontani importálás után az adattömböket. 

- A polybase egy rögzített sor lezárójele \n vagy új sor. Ez problémákat okozhat, ha a forrásadatok soremelés karakter jelenik meg.

- A forrás sémát nem támogatottak az SQL Data Warehouse adattípusokat tartalmazhat.

Ezek a korlátozások megkerülő megoldásként létrehozhat egy tárolt eljárást, amely végrehajtja a szükséges átalakítás. Ez a tárolt eljárás hivatkozhat a bcp futtatásakor. Másik lehetőségként [Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) automatikusan átalakítja az SQL Data Warehouse nem támogatott adattípusokat.

További információkért tekintse át a következő cikkeket:

- [Ajánlott eljárások az adatok betöltéséhez az Azure SQL Data Warehouse-bA](/azure/sql-data-warehouse/guidance-for-loading-data).
- [A sémák áttelepítése az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [Útmutató az SQL Data Warehouse tábla adattípusok definiálása](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>Az adatok átalakítása

Az adatok átalakításához, és helyezze át a termelési táblákba. Ebben a lépésben az adatok átalakításának ténytáblák, alkalmas szemantikai modellezés és dimenziótáblák csillag sémában.

Hozzon létre fürtözött oszlopcentrikus indexek, ami a legjobb általános lekérdezési teljesítményt termelési táblákba. Az Oszlopcentrikus indexek sok rekordot vizsgálata lekérdezés számára vannak optimalizálva. Az Oszlopcentrikus indexek nem is végrehajtásához egyszeres keresések (amelyek egyetlen sor van, keresése). Gyakori egyszeres keresések végrehajtására van szüksége, ha egy tábla egy nem fürtözött indexet is hozzáadhat. Egyszeres keresések futtathatja, jelentősen gyorsabban használatával egy nem fürtözött indexet. Azonban a singleton kereséseket is általában kevésbé gyakori a data warehouse-forgatókönyvek, mint az OLTP-munkaterhelések. További információkért lásd: [indexelés a táblák az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).

> [!NOTE]
> Nem támogatja a fürtözött oszlopcentrikus táblák `varchar(max)`, `nvarchar(max)`, vagy `varbinary(max)` adattípusokat. Ebben az esetben érdemes lehet egy halmot vagy fürtözött index. Előfordulhat, hogy ezek az oszlopok helyezzen egy különálló táblában.

Mivel az adatbázist nem nagyon nagy méretű, nem partícióval rendelkező létrehozott replikált táblák. A termelési számítási feladatokhoz elosztott táblák használata valószínűleg javítható a lekérdezési teljesítmény. Lásd: [útmutatást az Azure SQL Data Warehouse elosztott táblák tervezése](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute). Példa szkriptjeit futtassa a lekérdezéseket a statikus [erőforrásosztály](/azure/sql-data-warehouse/resource-classes-for-workload-management).

### <a name="load-the-semantic-model"></a>A szemantikai modell betöltése

Az adatok betöltése az Azure Analysis Services táblázatos modellbe. Ebben a lépésben az SQL Server Data Tools (SSDT) használatával hoz létre egy olyan szemantikai modell. Egy modellt a Power BI Desktop-fájlból való importálásával is létrehozhat. Mivel az SQL Data Warehouse nem támogatja a külső kulcsok, hozzá kell adnia a kapcsolatokat a szemantikai modell úgy, hogy a táblák között is csatlakozhat.

### <a name="use-power-bi-to-visualize-the-data"></a>Az adatok megjelenítése a Power BI használatával

A Power BI csatlakozik az Azure Analysis Services támogatja két lehetőség közül választhat:

- Az importálás. Az adatok a Power BI-modellben van importálva.
- Élő kapcsolat. Data Analysis Services-ről kéri le.

Javasoljuk, hogy élő kapcsolat, mert nincs szükség a Power BI-modellben az adatok másolását. Emellett a DirectQuery használata biztosítja, hogy eredmények mindig konzisztensek legyenek a legújabb forrásadatokat. További információkért lásd: [csatlakozás a Power bi-JAL](/azure/analysis-services/analysis-services-connect-pbi).

**Javaslatok**

Ne közvetlenül az adatraktáron irányítópult-lekérdezések futtatása a BI-ban. A BI-irányítópultok szükséges nagyon alacsony válaszidők, amely közvetlen, előfordulhat, hogy az adatraktár-lekérdezéseket az nem felel meg. Emellett az irányítópult frissítése vonja le a rendszer a lekérdezést, ami hatással lehet a teljesítmény számát. 

Az Azure Analysis Services célja annak érdekében, hogy a lekérdezés, BI-irányítópult, ezért az ajánlott eljárás lekérdezés Analysis Services a Power bi-BÓL.

## <a name="scalability-considerations"></a>Méretezési szempontok

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

Az SQL Data Warehouse a számítási erőforrások igény szerinti horizontálisan. A lekérdezési motor optimalizálja a lekérdezések párhuzamos feldolgozásra, a számítási csomópontok száma alapján, és szükség esetén csomópontok között helyez át adatokat. További információkért lásd: [kezelés compute az Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="analysis-services"></a>Analysis Services

A termelési számítási feladatokhoz, javasoljuk, hogy a Standard csomag az Azure Analysis Services, mivel támogatja a particionálást és a DirectQuery. A szinteken belül a példány mérete határozza meg, a memória és a feldolgozási teljesítményt. A feldolgozási teljesítményt mértékegysége a lekérés-feldolgozó egység (qpu-ra). Válassza ki a megfelelő méret a QPU-használat figyeléséhez. További információkért lásd: [kiszolgáló metrikáinak monitorozása](/azure/analysis-services/analysis-services-monitor).

Nagy terhelés alatt lekérdezési teljesítmény is válnak teljesítményűre lekérdezés párhuzamosság miatt. Analysis Services horizontálisan készletet hoz létre egy replikák a lekérdezések feldolgozásához, hogy egyidejűleg több lekérdezést is elvégezhető. Az adatmodell mindig feldolgozási munka az elsődleges kiszolgálón történik. Alapértelmezés szerint az elsődleges kiszolgáló lekérdezéseket is kezeli. Igény szerint is kijelölhet feldolgozás, kizárólag az elsődleges kiszolgálót, hogy a lekérdezési készletből kezeli az összes lekérdezés. Ha nagy feldolgozási követelmények, meg kell külön a feldolgozás, a lekérdezési készletből. Ha a lekérdezési terhelés, és viszonylag könnyű feldolgozása, az elsődleges kiszolgálón is felvehet a lekérdezési készletből. További információkért lásd: [kibővített Azure Analysis Services](/azure/analysis-services/analysis-services-scale-out). 

Ami felesleges feldolgozási mennyisége csökkentése érdekében fontolja meg a partíciók osztani a táblázatos modell logikai részekre. Mindegyik partíció külön-külön lehet feldolgozni. További információkért lásd: [partíciók](/sql/analysis-services/tabular-models/partitions-ssas-tabular).

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="ip-whitelisting-of-analysis-services-clients"></a>Az Analysis Services-ügyfelek IP-engedélyezési

Fontolja meg az Analysis Services tűzfala szolgáltatást az ügyfél IP-címeket. Ha engedélyezve van, a tűzfal blokkolja a tűzfalszabályok az összes ügyfélkapcsolatokat. Az alapértelmezett szabályok engedélyezési lista a Power BI szolgáltatásban, de a szükség esetén letilthatja a Ez a szabály. További információkért lásd: [korlátozására Azure Analysis Services az új tűzfal képességet nyújtanak](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/).

### <a name="authorization"></a>Engedélyezés

Az Azure Analysis Services az Azure Active Directory (Azure AD) segítségével hitelesítheti a felhasználókat, akik csatlakozik egy Analysis Services-kiszolgálóhoz. Korlátozhatja, hogy milyen adatokat egy adott felhasználó, megtekintheti a szerepkörök létrehozásával, és ezután az Azure AD-felhasználók és csoportok hozzárendelése ezeket a szerepköröket. Az egyes szerepkörökhöz a következőket teheti: 

- Táblák vagy oszlopok az egyes védelmét. 
- A szűrőkifejezések alapján egyedi sorok védelmét. 

További információkért lásd: [adatbázis-szerepkörök és a felhasználók kezelése](/azure/analysis-services/analysis-services-database-users).

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A telepítés, és futtassa a referenciaimplementációt, kövesse a lépéseket a [GitHub információs][github-folder]. A következőket helyezi üzembe:

  * Windows virtuális gép egy helyszíni adatbázis-kiszolgáló szimulálásához. Ez magában foglalja az SQL Server 2017-ben és a kapcsolódó eszközök, Power BI Desktop együtt.
  * Azure storage-fiókkal, amely biztosít a Blob storage, az SQL Server-adatbázisból exportált adatok tárolásához.
  * Egy Azure SQL Data Warehouse-példányhoz.
  * Az Azure Analysis Services-példányt.


## <a name="next-steps"></a>További lépések

- Azure Data Factory használatával az ELT folyamatok automatizálásához. Lásd: [az automatikus vállalati bi-ban az SQL Data Warehouse és az Azure Data Factory] [adf = ra].

<!-- links -->

[adf-ra]: ./enterprise-bi-adf.md
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database

