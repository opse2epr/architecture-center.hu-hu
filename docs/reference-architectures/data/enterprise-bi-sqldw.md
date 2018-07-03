---
title: Enterprise BI és SQL Data Warehouse
description: A relációs adatok az üzleti elemzéseket kaphat az Azure tárolja a helyszíni
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: e3542e40b4b6d1f604f93bb21528f34ba7f22fc6
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142335"
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>Enterprise BI és SQL Data Warehouse

Ez a referenciaarchitektúra valósít meg egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kinyerési, betöltési, átalakítási) folyamat, amely helyez át adatokat a helyszíni SQL Server-adatbázisból az SQL Data Warehouse-ba, és átalakítja az adatokat az elemzéshez. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

**A forgatókönyv**: egy a szervezet rendelkezik a helyszíni SQL Server-adatbázisban tárolt nagy OLTP adatkészlet. A szervezet célja az SQL Data Warehouse használata a Power BI segítségével történő elemzését. 

Ez a referenciaarchitektúra egyszeri vagy igény szerinti feladat lett tervezve. Adatok áthelyezése (óránként vagy naponta) tartósan van szüksége, ha automatizált munkafolyamatokat az Azure Data Factory használatát javasoljuk. Egy referencia-architektúra, amely a Data Factory használja, lásd: [vállalati bi-ban az SQL Data Warehouse és az Azure Data Factory automatikus](./enterprise-bi-adf.md).

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

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo-folder]. A következőket helyezi üzembe:

  * Windows virtuális gép egy helyszíni adatbázis-kiszolgáló szimulálásához. Ez magában foglalja az SQL Server 2017-ben és a kapcsolódó eszközök, Power BI Desktop együtt.
  * Azure storage-fiókkal, amely biztosít a Blob storage, az SQL Server-adatbázisból exportált adatok tárolásához.
  * Egy Azure SQL Data Warehouse-példányhoz.
  * Az Azure Analysis Services-példányt.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-server"></a>A szimulált helyszíni kiszolgáló üzembe helyezése

Egy virtuális Gépet először egy szimulált helyszíni kiszolgálóként, amely tartalmazza az SQL Server 2017-ben és a kapcsolódó eszközök helyezünk üzembe. Ez a lépés is betölti a [Wide World Importers OLTP adatbázis] [ wwi] az SQL Server.

1. Keresse meg a `data\enterprise_bi_sqldw\onprem\templates` mappában található az adattárban.

2. Az a `onprem.parameters.json` fájlt, cserélje le a tartozó értékeket `adminUsername` és `adminPassword`. Értékeit is módosíthatja a `SqlUserCredentials` szakaszban felel meg a felhasználónevet és jelszót. Megjegyzés: a `.\\` a felhasználónév tulajdonságban előtaggal.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. Futtatás `azbb` a helyszíni kiszolgáló üzembe helyezése az alább látható módon.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

    Adja meg, amely támogatja az SQL Data Warehouse és az Azure Analysis Services egy régiót. Lásd: [az Azure-termékek régiók szerint](https://azure.microsoft.com/global-infrastructure/services/)

4. Az üzembe helyezés eltarthat 20 – 30 percet, amely tartalmazza a futó a [DSC](/powershell/dsc/overview) szkriptet az eszközök telepítésére, és állítsa vissza az adatbázist. Az üzembe helyezés az Azure Portalon ellenőrizze az erőforrások az erőforráscsoportban lévő áttekintésével. Megtekintheti a `sql-vm1` virtuális gép és az összes kapcsolódó erőforrás.

### <a name="deploy-the-azure-resources"></a>Az Azure-erőforrások üzembe helyezése

Ebben a lépésben együtt egy Storage-fiókot látja el az SQL Data Warehouse és az Azure Analysis Services esetén. Ha azt szeretné, ezt a lépést az előző lépés párhuzamosan is futtathatja.

1. Keresse meg a `data\enterprise_bi_sqldw\azure\templates` mappában található az adattárban.

2. Futtassa a következő Azure CLI-paranccsal hozzon létre egy erőforráscsoportot. Egy másik erőforráscsoportban található, mint az előző lépésben üzembe helyezése, de válassza ki az ugyanabban a régióban. 

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

3. Futtassa a következő Azure CLI-parancsot az Azure-erőforrások üzembe helyezéséhez. Cserélje le a csúcsos zárójelben látható paraméter értékét. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<server_name>" \
     "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="user@contoso.com"
    ```

    - A `storageAccountName` paraméter kell követnie a [elnevezési szabályok](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) tárfiókok esetében.
    - Az a `analysisServerAdmin` paramétert, használja az Azure Active Directory egyszerű felhasználónév (UPN).

4. Az üzembe helyezés az Azure Portalon ellenőrizze az erőforrások az erőforráscsoportban lévő áttekintésével. A storage-fiók, Azure SQL Data Warehouse-példányhoz és Analysis Services-példányt kell megjelennie.

5. Az Azure portal használatával a tárfiók hozzáférési kulcs lekérése. A megnyitáshoz válassza ki a tárfiókot. A **Beállítások** területen válassza a **Hozzáférési kulcsok** elemet. Másolja az elsődleges kulcs értékét. Ez a következő lépésben fogja használni.

### <a name="export-the-source-data-to-azure-blob-storage"></a>Az adatok exportálása az Azure Blob storage 

Ebben a lépésben fog futtatni egy PowerShell-parancsprogram, amely a BCP segítségével exportál az SQL database egybesimított fájlokba, a virtuális gépen, és ezután másolja ki ezeket a fájlokat az Azure Blob Storage-bA az AzCopy használatával.

1. A távoli asztal használatával csatlakozhat a szimulált helyszíni virtuális gép.

2. Bejelentkezés a virtuális géppel, futtassa az alábbi parancsokat a PowerShell-ablakból.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    Az a `Destination` paramétert, cserélje le `<storage_account_name>` a korábban létrehozott tárfiók nevét. Az a `StorageAccountKey` paramétert, a hozzáférési kulcsot használja a tárfiók.

3. Az Azure Portalon győződjön meg arról, hogy a forrásadatok a Blob storage másolták ellenőrizheti, hogy a tárfiók, a Blob szolgáltatás kiválasztásával, és nyissa meg a `wwi` tároló. Megjelenik a mintametódus használ táblák listáját `WorldWideImporters_Application_*`.

### <a name="run-the-data-warehouse-scripts"></a>A data warehouse parancsfájlok futtatása

1. A távoli asztali munkamenetből indítsa el az SQL Server Management Studio (SSMS). 

2. Connect to SQL Data Warehouse

    - Kiszolgáló típusa: adatbázis-kezelő
    
    - Kiszolgálónév: `<dwServerName>.database.windows.net`, ahol `<dwServerName>` amikor központilag telepítette az Azure-erőforrások megadott név. Ez a név kaphat az Azure Portalról.
    
    - Hitelesítés: SQL Server-hitelesítés. Amikor központilag telepítette az Azure-erőforrások, a megadott hitelesítő adatokat a `dwAdminLogin` és `dwAdminPassword` paramétereket.

2. Keresse meg a `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` mappa a virtuális gépen. Ebben a mappában számsorrendben, a parancsfájlok végrehajtásának `STEP_1` keresztül `STEP_7`.

3. Válassza ki a `master` adatbázisban az ssms-ben, és nyissa meg a `STEP_1` parancsfájlt. Módosítsa az értéket a következő sort a jelszót, majd hajtsa végre a parancsprogramot.

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. Válassza ki a `wwi` adatbázisban az ssms-ben. Nyissa meg a `STEP_2` szkriptet, és hajtsa végre a parancsfájlt. Ha hibaüzenetet kap, ellenőrizze, hogy a szkript futtatásakor a `wwi` adatbázist, és nem `master`.

5. Nyisson meg egy új kapcsolatot az SQL Data Warehouse használatával a `LoaderRC20` megadott felhasználónevet és a jelszót a `STEP_1` parancsfájlt.

6. Ez a kapcsolat használatával nyissa meg a `STEP_3` parancsfájlt. A parancsfájlban állítsa be a következő értékeket:

    - Titkos kulcs: A hozzáférési kulcsot használja a tárfiók.
    - HELY: A tárfiók nevére az alábbiak szerint alkalmazza: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`.

7. Ugyanazt a kapcsolatot, használja a szkriptek végrehajtása `STEP_4` keresztül `STEP_7` egymás után. Győződjön meg arról, hogy minden parancsprogram a következő futtatása előtt sikeresen lefutott.

Az SMSS, kell megjelennie egy készletét `prd.*` a táblák a `wwi` adatbázis. Annak ellenőrzéséhez, hogy létrejött-e az adatokat, a következő lekérdezés futtatásával: 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

## <a name="build-the-analysis-services-model"></a>Az Analysis Services-modell létrehozása

Ebben a lépésben létrehoz egy táblázatos modellhez, amely adatokat importál az adatraktárba. A modell telepíti majd az Azure Analysis Servicesbe.

1. A távoli asztali munkamenetből indítsa el az SQL Server Data Tools 2015.

2. Válassza a **File** (Fájl) > **New** (Új) > **Project** (Projekt) lehetőséget.

3. Az a **új projekt** párbeszédpanelen, a **sablonok**válassza **üzleti intelligencia** > **Analysis Services**  >  **Analysis Services rendszerbeli táblázatos projekt**. 

4. Adja a projektnek, és kattintson a **OK**.

5. Az a **táblázatos modell tervezője** párbeszédpanelen válassza **integrált munkaterület** és **kompatibilitási szint** való `SQL Server 2017 / Azure Analysis Services (1400)`. Kattintson az **OK** gombra.

6. Az a **Táblázatosmodell-tallózóban** ablakban kattintson a jobb gombbal a projektre, majd válassza ki **importálás adatforrásból**.

7. Válassza ki **Azure SQL Data Warehouse** kattintson **Connect**.

8. A **kiszolgáló**, adja meg az Azure SQL Data Warehouse-kiszolgáló teljes nevét. A **adatbázis**, adja meg `wwi`. Kattintson az **OK** gombra.

9. A következő párbeszédpanelen válasszon **adatbázis** hitelesítést, és adja meg az Azure SQL Data Warehouse-felhasználónevet és jelszót, majd kattintson **OK**.

10. Az a **kezelő** párbeszédpanelen jelölje be a jelölőnégyzeteket **prd. CityDimensions**, **prd. DateDimensions**, és **prd. SalesFact**. 

    ![](./images/analysis-services-import.png)

11. Kattintson a **terhelés**. Feldolgozás befejeződése után kattintson a **Bezárás**. Meg kell jelennie az adatok egy táblázatos nézetben.

12. Az a **Táblázatosmodell-tallózóban** ablakban kattintson a jobb gombbal a projektre, majd válassza ki **modellnézet** > **diagramnézet**.

13. Húzza a **[prd. SalesFact]. [WWI város ID]**  mezőt a **[prd. CityDimensions]. [WWI város ID]**  mező egy kapcsolat létrehozásához.  

14. Húzza a **[prd. SalesFact]. [Invoice Date Key]**  mezőt a **[prd. DateDimensions]. [Date]**  mező.  
    ![](./images/analysis-services-relations.png)

15. Az a **fájl** menüben válassza a **összes mentése**.  

16. A **Megoldáskezelőben**, kattintson a jobb gombbal a projektre, és válassza ki **tulajdonságok**. 

17. A **kiszolgáló**, adja meg az Azure Analysis Services-példány URL-CÍMÉT. Ezt az értéket kaphat az Azure Portalról. A portálon, válassza ki az Analysis Services-erőforrást, az Áttekintés ablaktábláján kattintson, és keresse meg a **kiszolgálónév** tulajdonság. Hasonló lesz `asazure://westus.asazure.windows.net/contoso`. Kattintson az **OK** gombra.

    ![](./images/analysis-services-properties.png)

18. A **Megoldáskezelőben**, kattintson a jobb gombbal a projektre, és válassza ki **telepítés**. Ha a rendszer kéri, jelentkezzen be Azure. Feldolgozás befejeződése után kattintson a **Bezárás**.

19. Az Azure Portalon az Azure Analysis Services-példány részleteinek megtekintéséhez. Győződjön meg arról, hogy a modell a modellek listája látható.

    ![](./images/analysis-services-models.png)

## <a name="analyze-the-data-in-power-bi-desktop"></a>Elemezheti az adatokat a Power BI Desktopban

Ebben a lépésben használandó Power BI-jelentés létrehozása az adatokból az Analysis Servicesben.

1. A távoli asztali munkamenetből indítsa el a Power BI Desktopban.

2. Kattintson az üdvözlő Scren **adatok lekérése**.

3. Válassza ki **Azure** > **Azure Analysis Services-adatbázis**. Kattintson a **csatlakoztatása**

    ![](./images/power-bi-get-data.png)

4. Adja meg az Analysis Services-példány URL-CÍMÉT, majd kattintson a **OK**. Ha a rendszer kéri, jelentkezzen be Azure.

5. Az a **kezelő** párbeszédpanelen bontsa ki a táblázatos projektet, üzembe helyezett, válassza ki a létrehozott, és kattintson a létrehozott modellt **OK**.

2. Az a **Vizualizációk** panelen válassza a **-ig halmozott sávdiagram** ikonra. A jelentés nézetben méretezze át a Vizualizáció nagyobb legyen.

6. Az a **mezők** ablaktáblán bontsa ki a **prd. CityDimensions**.

7. A csomóponthúzási **prd. CityDimensions** > **WWI város azonosító** , a **tengely** is.

8. A csomóponthúzási **prd. CityDimensions** > **Város** , a **jelmagyarázat** is.

9. Az a **mezők** ablaktáblán bontsa ki a **prd. SalesFact**.

10. A csomóponthúzási **prd. SalesFact** > **kivételével adó** , a **érték** is.

    ![](./images/power-bi-visualization.png)

11. A **Látványelemszint szűrői**válassza **WWI város azonosító**.

12. Állítsa be a **szűrőtípus** való `Top N`, és állítsa be **elemek megjelenítése** való `Top 10`.

13. A csomóponthúzási **prd. SalesFact** > **kivételével adó** , a **idővel** is

    ![](./images/power-bi-visualization2.png)

14. Kattintson a **szűrő alkalmazása**. A Vizualizáció a felső 10 összes értékesítés város szerint jeleníti meg.

    ![](./images/power-bi-report.png)

A Power BI Desktop kapcsolatos további információkért lásd: [Ismerkedés a Power BI Desktop](/power-bi/desktop-getting-started).

## <a name="next-steps"></a>További lépések

- Ez a referenciaarchitektúra kapcsolatos további információkért látogasson el a [GitHub-adattár][ref-arch-repo-folder].
- További információ a [az Azure építőelemei][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[wwi]: /sql/sample/world-wide-importers/wide-world-importers-oltp-database
