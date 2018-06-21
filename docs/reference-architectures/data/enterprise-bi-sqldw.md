---
title: Vállalati BI, az SQL Data Warehouse szolgáltatással
description: A relációs adatok üzleti dcu használata Azure tárolt helyszíni
author: alexbuckgit
ms.date: 04/13/2018
ms.openlocfilehash: b5e5aa32fc9cc8c7b8b5a42c9a4fc3e0216b2f72
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
ms.locfileid: "31019107"
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>Vállalati BI, az SQL Data Warehouse szolgáltatással
 
A referencia-architektúrában megvalósítja az [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kivonat-betöltési-átalakítási) folyamatot, amely adatokat a helyszíni SQL Server-adatbázist az SQL Data Warehouse pedig átalakítja a vonatkozó adatok elemzési célú. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

**A forgatókönyv**: egy szervezet rendelkezik a nagy OLTP adatkészletet, a helyszíni SQL Server adatbázis tárolja. A szervezet szeretné használni az SQL Data Warehouse elemzésére a Power BI használatával. 

A referencia-architektúrában egyszeri vagy igény szerinti feladatok tervezték. Ha az adatok áthelyezése (óránként vagy naponta) tartósan van szüksége, azt javasoljuk, Azure Data Factory egy automatizált munkafolyamat meghatározásához.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

**Egy SQL Server**. Az adatok a helyszíni SQL Server-adatbázisban található. A helyszíni környezetben, az az architektúra biztosítása a telepített SQL Server Azure virtuális gép üzembe helyezési parancsfájlok szimulálásához. 

**BLOB Storage**. A BLOB storage segítségével egy átmeneti területre, másolja az adatokat, az SQL Data Warehouse betöltése előtt.

**Azure SQL Data Warehouse**. [Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer készült elemzés végrehajtásához, nagy. Támogatja a jelentős olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas a nagy teljesítményű analytics futtatásához. 

**Az Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely a modellezési képességekkel adatokat biztosít. Analysis Services segítségével hozzon létre egy, a felhasználók lekérdezheti a szemantikai modellben. Analysis Services különösen fontos olyan BI-irányítópult forgatókönyvekben. Ebben az architektúrában Analysis Services olvassa be az adatokat az adatraktárban a szemantikai modell feldolgozásához, és hatékonyan működik az irányítópult lekérdezések. Rugalmas párhuzamossági gyorsabb a lekérdezés feldolgozása a replikák kiterjesztése által is támogatja.

Jelenleg Azure Analysis Services rendszerbeli táblázatos modellek, de nem többdimenziós modellek támogatja. A táblázatos modellek használata relációs modellezési hoz létre (táblákat és oszlopokat), míg többdimenziós modellek használó OLAP modellezési szerkezetek (kockák, dimenziókat és mértékeket). Ha többdimenziós modellek van szüksége, használja az SQL Server Analysis Services (SSAS). További információkért lásd: [többdimenziós vagy táblázatos megoldások összehasonlítása](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas).

**Power BI**. A Power BI eszközcsomagot jelent üzleti analytics üzleti elemzések készítése adatok elemzésére. Ebben az architektúrában lekéri az Analysis Servicesben tárolt a szemantikai modellben.

**Az Azure Active Directory** (az Azure AD) hitelesíti a Power BI Analysis Services-kiszolgálóhoz csatlakozó felhasználók számára.

## <a name="data-pipeline"></a>Data Pipeline
 
A referencia-architektúrában használja a [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) mintaadatbázis adatforrásként. Az adatok csővezeték rendelkezik a következő szakaszt:

1. Az adatok exportálása az SQL Serverről egybesimított fájlokba (bcp segédprogram).
2. Másolja a strukturálatlan fájlokat az Azure Blob Storage-(ba AzCopy).
3. Az adatok betöltése az SQL Data warehouse-ba (PolyBase).
4. Adatok átalakítása a csillagsémát (T-SQL).
5. A szemantikai modell betöltése az Analysis Services (SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> A lépéseket 1 &ndash; 3, fontolja meg a Redgate adatok Platform Studio használatát. A Data Platform Studio a legmegfelelőbb kompatibilitási javításokat és optimalizálásokat alkalmazza, így ez a leggyorsabb mód az SQL Data Warehouse használatának megkezdésére. További információkért lásd: [adatok betöltését az Redgate adatok Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate). 

A következő szakaszok ismertetik a szakaszok részletesebben.

### <a name="export-data-from-sql-server"></a>Exportál adatokat az SQL Server

A [bcp](/sql/tools/bcp-utility) (tömeges másolási funkciójával) segédprogram használata egyszerű szöveges fájlt létrehozni az SQL-táblák gyors módja. Ebben a lépésben válassza az oszlopot, amely kíván exportálni, de nem átalakíthatja az adatokat. Bármely adatátalakítást az SQL Data Warehouse megtörténik.

**Javaslatok**

Ha lehetséges ütemezni a csúcsidőn, minimálisra csökkentése érdekében az éles környezetben Erőforrásverseny adatok kinyerése. 

Kerülje a bcp fut a kiszolgálón. Ehelyett futtassa azt egy másik számítógépről. Egy helyi meghajtó a fájlok írni. Győződjön meg arról, hogy rendelkezik-e kezelni az egyidejű írási műveletek elegendő i/o-erőforrások. A legjobb teljesítmény érdekében a fájlok exportálása dedikált gyors tárolóeszközöket.

A hálózati átvitel felgyorsíthatja az exportált adatok mentése Gzip tömörített formátumban. Tömörített fájlok betöltését az adatraktárba azonban alacsonyabb, mint a kibontott fájlok betöltése, így gyorsabban betöltése vagy gyorsabb hálózati átviteli között egy kompromisszumot. Ha úgy dönt, hogy a Gzip-tömörítés, ne hozzon létre egy Gzip fájl. Ehelyett az adatok felosztása több tömörített fájl.

### <a name="copy-flat-files-into-blob-storage"></a>A blob-tároló egybesimított fájlok másolása

A [AzCopy](/azure/storage/common/storage-use-azcopy) segédprogram készült tartozó nagyteljesítményű másolására az adatok Azure blob-tárolóba.

**Javaslatok**

A storage-fiók létrehozása az adatok helye közeli régióban. Telepítse a tárfiók és az SQL Data Warehouse-példányokhoz ugyanabban a régióban. 

AzCopy ugyanazon a számítógépen, amelyen a termelési számítási feladatokhoz, nem futtatható, mert a Processzor- és i/o-felhasználás megzavarhatja a termelési munkaterhelés. 

A feltöltés először megtudhatja, mi a feltöltési sebességét például tesztelése. AzCopy /NC beállítás használatával adja meg az egyidejű másolási műveletek számát. Indítsa el az alapértelmezett értékkel, majd ezt a beállítást, a teljesítmény hangolására kísérletezhet. Alacsony sávszélességű környezetben túl sok egyidejű művelet ne terhelje tovább a hálózati kapcsolatot, és megakadályozza a művelet sikeres befejezését.  

AzCopy adatok áthelyezése a nyilvános interneten keresztül a tárolási. Ha ez nem elég gyors, érdemes beállítani egy [ExpressRoute](/azure/expressroute/) körön. ExpressRoute olyan szolgáltatás, amely továbbítja a dedikált titkos kapcsolaton keresztül az adatok az Azure-bA. Egy másik lehetőség, ha a hálózati kapcsolat túl lassú, egy fizikailag küldje el az adatokat egy Azure-adatközpontban lemezen lévő. További információkért lásd: [Azure érkező vagy oda irányuló adatátvitel](/azure/architecture/data-guide/scenarios/data-transfer).

A másolási műveletek során AzCopy hoz létre egy ideiglenes napló fájlt, amely lehetővé teszi az AzCopy segítségével indítsa újra a műveletet, ha azt lekérése megszakadt (például egy hálózati hiba miatt). Győződjön meg arról, hogy nincs elegendő szabad lemezterület az adatbázisnapló-fájlok tárolására. A /Z kapcsoló használatával adja meg, ahová az adatbázisnapló-fájlok készültek.

### <a name="load-data-into-sql-data-warehouse"></a>Adatok betöltése az SQL Data Warehouse-ba

Használjon [PolyBase](/sql/relational-databases/polybase/polybase-guide) tölthető be a fájlokat a blob-tároló az adatraktárba. A PolyBase célja, hogy kihasználja a (nagymértékben párhuzamos feldolgozás) MPP architektúra az SQL Data Warehouse, így az adatok betöltése az SQL Data Warehouse a leggyorsabban. 

Az adatok betöltése két lépésből áll:

1. Az adatok külső táblái létrehozása. A külső tábla esetében mutat, az adatraktár-on kívüli adatkészletekhez tábla definícióját &mdash; ebben az esetben a simán fájlok, a blob Storage tárolóban. Ez a lépés nem helyezi át adatokat az adatraktárba.
2. Átmeneti tárolási táblák létrehozása, és az adatok betöltése az átmeneti tárolási táblákba. Ez a lépés másolja az adatokat az adatraktárba.

**Javaslatok**

Fontolja meg az SQL Data Warehouse, ha nagy mennyiségű adat (több mint 1 TB-os) és egy analytics munkaterhelés, amelyek előnyösek, párhuzamossági futnak. Az SQL Data Warehouse nincs remekül beválik, ha az OLTP-munkaterhelések vagy kisebb adatkészletek (< 250GB). Az adatkészlet kevesebb mint 250GB fontolja meg az Azure SQL Database vagy az SQL Server. További információkért lásd: [adatraktározással](../../data-guide/relational-data/data-warehousing.md).

Az előkészítési táblák létrehozása halommemória táblákat, amelyek indexelése. A lekérdezések, amely a termelési táblák létrehozása teljes táblázatbeolvasás eredményez, így nem ok arra, hogy az előkészítési táblák index.

A PolyBase automatikusan kihasználja párhuzamossági az adatraktár. A betöltési teljesítmény arányosan, ahogy a dwu-k növeli. A legjobb teljesítmény érdekében használjon egy egyetlen betöltési művelethez. Nincs a bemeneti adatok ossza adattömböket, és több egyidejű terhelések futtató teljesítmény előnye.

A PolyBase Gzip tömörített fájlok olvashatja. Azonban csak egyetlen olvasó szolgál egy tömörített fájl, mert a fájl kibontása nem egyszálas működésre. Ezért ne egyetlen nagy tömörített fájl betöltésekor. Ehelyett az adatok felosztása több tömörített fájl párhuzamossági kihasználása érdekében. 

Vegye figyelembe a következő korlátozások vonatkoznak:

- A PolyBase támogatja a oszlop maximális méretet `varchar(8000)`, `nvarchar(4000)`, vagy `varbinary(8000)`. Ha rendelkezik, amelynek hossza meghaladja a működés felső korlátjának adatokat, egy lehetőség egy feloszthatja az adatok adattömbökbe exportálni, és az adattömbök majd később az importálás után. 

- A polybase egy rögzített méretű sor lezárójele \n vagy új sor. Ez problémát okozhat, ha a forrásadatok szerepelhet soremelés karaktereket.

- A forrás adatsémája nem támogatott az SQL Data Warehouse adattípusok is tartalmazhatnak.

Ezekkel a korlátozásokkal megkerüléséhez tárolt eljárás, amely elvégzi a szükséges átalakítások is létrehozhat. Ez a tárolt eljárás hivatkozzon a bcp futtatásakor. Másik lehetőségként [Redgate adatok Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate) automatikusan átalakítja az adattípust, amelyet az SQL Data Warehouse nem támogatottak.

További információkért tekintse át a következő cikkeket:

- [Ajánlott eljárások az adatok betöltése az Azure SQL Data Warehouse](/azure/sql-data-warehouse/guidance-for-loading-data).
- [Az SQL Data Warehouse a sémák áttelepítése](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [Útmutató az SQL Data Warehouse táblák adattípusok definiálása](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>Az adatok átalakítása

Átalakíthatja az adatokat, és helyezze át a termelési táblákat. Ebben a lépésben az adatokat a dimenziótáblák és ténytáblák, megfelelő-e szemantikai modellezési csillagséma alakul.

Hozzon létre fürtözött oszlopcentrikus indexek – amelyek nyújtja a legjobb átfogó lekérdezési teljesítményt a termelési táblákat. Oszlopcentrikus indexek sok rekord vizsgálhatják lekérdezések vannak optimalizálva. Oszlopcentrikus indexek nem végezze el a is egypéldányos keresések (amely keresése van, egy sorban). Ha gyakori egypéldányos végrehajtott keresési műveletek elvégzéséhez szüksége, adhat hozzá egy nem fürtözött index táblázat. Egypéldányos keresések futtatható jelentősen gyorsabb egy nem fürtözött index. Azonban az Egypéldányos végzett keresések nem általában ritkább data warehouse forgatókönyvekben mint az OLTP-munkaterhelések. További információkért lásd: [táblák az SQL Data Warehouse indexelő](/azure/sql-data-warehouse/sql-data-warehouse-tables-index).

> [!NOTE]
> Nem támogatja a fürtözött oszloptárindexű táblákat `varchar(max)`, `nvarchar(max)`, vagy `varbinary(max)` adattípusokat. Ebben az esetben érdemes lehet halmot vagy fürtözött index. Előfordulhat, hogy ezek az oszlopok bocsátott különálló táblában.

Mivel a mintaadatbázis nem nagyon nagy, nem partíciókkal rendelkező létrehozott replikált táblák. A termelési számítási feladatokhoz elosztott táblák használata valószínűleg javítható a teljesítmény-küszöbérték. Lásd: [útmutató az Azure SQL Data Warehouse elosztott táblák tervezése](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute). A példa parancsfájlok a lekérdezések futtatása használatával statikus [erőforrásosztály](/azure/sql-data-warehouse/resource-classes-for-workload-management).

### <a name="load-the-semantic-model"></a>A szemantikai modell betöltése

Adatok betöltése az Azure Analysis Services táblázatos modell. Ebben a lépésben hoz létre egy szemantikai adatmodell SQL Server Data Tools (SSDT). A modell a Power BI Desktop-fájlból való importálásával is létrehozhat. Az SQL Data Warehouse nem támogatja a külső kulcsokat, hozzá kell adnia a kapcsolat a szemantikai modell, hogy a táblák között kapcsolhatja össze.

### <a name="use-power-bi-to-visualize-the-data"></a>Az adatok megjelenítése a Power BI használatával

A Power BI Azure Analysis Services történő kapcsolódáshoz támogatja a két lehetőség közül választhat:

- Importálja. Az adatok importálása a Power BI-modellbe.
- Élő kapcsolatot. Az adatok közvetlenül Analysis Services lekért.

Élő kapcsolatot azt javasoljuk, mert nincs szükség az adatok másolásának a Power BI-modellbe. Is a DirectQuery biztosítja, hogy az eredmények mindig a legújabb forrásadatok összhangban. További információkért lásd: [csatlakozás a Power BI](/azure/analysis-services/analysis-services-connect-pbi).

**Javaslatok**

Küszöbölhető BI irányítópult lekérdezések közvetlenül az adatraktáron. Üzleti INTELLIGENCIA irányítópultok szükséges nagyon alacsony válaszidejét, amely közvetlen, előfordulhat, hogy a lekérdezések írásában, az adatraktár nem elégíti ki. Is az irányítópult frissítésekor fog, csökkenti az egyidejű lekérdezések, amelyek jelentős hatással lehet a teljesítmény. 

Az Azure Analysis Services egy BI-irányítópult lekérdezés követelményeinek kezelésére, ezért az ajánlott eljárás lekérdezés Analysis Services a Power BI-ból terveztek.

## <a name="scalability-considerations"></a>Méretezhetőségi kérdései

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

Az SQL Data Warehouse ki lehet terjeszteni a számítási erőforrásokat szükség esetén kérheti. A lekérdezési motor optimalizálja a lekérdezések alapján a számítási csomópontok száma párhuzamos feldolgozásra, és mozgatja az adatokat szükség szerint a csomópontok között. További információkért lásd: [kezelése számítási az Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="analysis-services"></a>Analysis Services

A termelési számítási feladatokhoz ajánlott Standard csomagra Azure Analysis Services, mert támogatja a particionálás és a DirectQuery. A réteg belül példány határozza meg a memória és a feldolgozási kapacitást. Feldolgozási kapacitása révén a lekérdezés feldolgozása egységek (QPUs) mérik. A QPU megfigyeléséhez válassza ki a megfelelő méretű. További információkért lásd: [server metrikát](/azure/analysis-services/analysis-services-monitor).

Nagy terhelés teljesítmény-küszöbérték is válnak romlik a lekérdezés párhuzamosság miatt. Ki lehet terjeszteni Analysis Services a lekérdezések feldolgozásához replikák készletét létrehozásával, így további lekérdezések egyidejűleg végrehajtható. Az adatmodell mindig feldolgozási munka az elsődleges kiszolgálón történik. Alapértelmezés szerint az elsődleges kiszolgáló lekérdezéseket is kezeli. Szükség esetén is kijelölhet feldolgozás kizárólag, az elsődleges kiszolgálót, hogy a lekérdezés készlet az összes lekérdezés kezeli. Ha nagy feldolgozási követelményeket, a lekérdezés készletből feldolgozási kell egymástól. Ha magas lekérdezésekkel, és viszonylag könnyű feldolgozása, az elsődleges kiszolgálón is felvehet a lekérdezés-készlet. További információkért lásd: [Azure Analysis Services kibővített](/azure/analysis-services/analysis-services-scale-out). 

Rövidítse le a felesleges feldolgozási, fontolja meg a partíciók használatával a táblázatos modell olyan részekre való felosztásához logikai. Mindegyik partíció külön-külön lehet feldolgozni. További információkért lásd: [partíciók](/sql/analysis-services/tabular-models/partitions-ssas-tabular).

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="ip-whitelisting-of-analysis-services-clients"></a>Az Analysis Services ügyfelek IP-engedélyezése

Fontolja meg az Analysis Services-tűzfal szolgáltatás engedélyezett ügyfél IP-címek használatát. Ha engedélyezve van, a tűzfal blokkolja az tűzfalszabályok minden ügyfél-kommunikációhoz. Az alapértelmezett szabályok engedélyezési listája a Power BI-ban, de a szükség esetén letilthatja a Ez a szabály. További információkért lásd: [Azure Analysis Services korlátozására a tűzfal az új képesség](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/).

### <a name="authorization"></a>Engedélyezés

Az Azure Analysis Services egy Analysis Services-kiszolgálóhoz csatlakozó felhasználók hitelesítéséhez az Azure Active Directory (Azure AD). Korlátozhatja, hogy milyen adatokat egy adott felhasználó megtekintheti, szerepkörök létrehozásával, és majd ezen szerepkörök hozzárendelése az Azure AD-felhasználókat vagy csoportokat. Az egyes szerepkörökhöz a következő műveletek végezhetők el: 

- Táblák, illetve az egyes oszlopok védheti. 
- Az egyes sorok szűrőkifejezéseket alapján védelme. 

További információkért lásd: [adatbázis-szerepkörök és a felhasználók kezelése](/azure/analysis-services/analysis-services-database-users).

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo-folder]. A következőket helyezi üzembe:

  * Windows virtuális gép egy helyi adatbázis-kiszolgáló szimulálásához. SQL Server 2017 és a kapcsolódó eszközök, valamint Power BI Desktop tartalmaz.
  * Egy Azure storage-fiók, amely Blob-tároló az SQL Server adatbázisból exportált adatok tárolásához.
  * Egy Azure SQL Data Warehouse-példányhoz.
  * Egy Azure Analysis Services-példányon.

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [Azure referencia architektúra] [ ref-arch-repo] GitHub-tárházban.

2. Telepítse a [Azure építőelemeket] [ azbb-wiki] (azbb).

3. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs használatával, és az utasításokat követve.

  ```bash
  az login  
  ```

### <a name="deploy-the-simulated-on-premises-server"></a>A szimulált helyszíni kiszolgáló központi telepítése

Először teheti elérhetővé a virtuális gépek szimulált a helyszíni kiszolgálónak, ide tartozik az SQL Server 2017 és a kapcsolódó eszközök. Ez a lépés is betölti a minta [Wide World Importers OLTP adatbázis](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) az SQL-kiszolgálóra.

1. Keresse meg a `data\enterprise-bi-sqldw\onprem\templates` a fenti előfeltételek letöltött tárház mappát.

2. Az a `onprem.parameters.json` fájlt, cserélje le a értékeinek `adminUsername` és `adminPassword`. Szereplő értékeket is módosíthatja a `SqlUserCredentials` szakasz felel meg a felhasználónevet és jelszót. Megjegyzés: a `.\\` a userName tulajdonság-előtagot.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. Futtatás `azbb` a helyszíni kiszolgáló telepítésének alább látható módon.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p onprem.parameters.json --deploy
    ```

4. A központi telepítés eltarthat 20-30 percet, mely tartalmazza, hogy fut a [DSC](/powershell/dsc/overview) parancsprogramot, az eszközök telepítése, majd állítsa vissza az adatbázist. A központi telepítést, az Azure portálon ellenőrizze az erőforrások az erőforráscsoportban megtekintésével. Megjelenik a `sql-vm1` virtuális gép és a kapcsolódó erőforrások.

### <a name="deploy-the-azure-resources"></a>Az Azure-erőforrások telepítése

Ebben a lépésben kiépítését Azure SQL Data warehouse-bA és az Azure Analysis Services, és a Storage-fiók. Ha azt szeretné, ebben a lépésben az előző lépés párhuzamosan is futtathatja.

1. Keresse meg a `data\enterprise-bi-sqldw\azure\templates` a fenti előfeltételek letöltött tárház mappát.

2. A következő parancsot az Azure parancssori felület futtatásával hozzon létre egy erőforráscsoportot, cseréje a zárójeles paraméterek vannak megadva. Vegye figyelembe, hogy telepítene egy másik erőforráscsoportban található, mint a helyszíni kiszolgáló, az előző lépésben használt. 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. A következő parancsot az Azure parancssori felület telepítése az Azure-erőforrások, a mag cseréje a zárójeles paraméterek vannak megadva. A `storageAccountName` paraméter kell követnie a [elnevezési szabályait](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) Storage-fiókok. Az a `analysisServerAdmin` paraméter, használja az Azure Active Directory egyszerű felhasználónév (UPN).

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<server_name>" "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="user@contoso.com"
    ```

4. A központi telepítést, az Azure portálon ellenőrizze az erőforrások az erőforráscsoportban megtekintésével. Meg kell jelennie egy tárfiókot, Azure SQL Data Warehouse-példányokhoz és Analysis Services-példányon.

5. Az Azure portál segítségével a tárfiók elérési kulcsának lekérése. A megnyitáshoz válassza ki a tárfiókot. A **Beállítások** területen válassza a **Hozzáférési kulcsok** elemet. Másolja az elsődleges kulcs értéke. A következő lépésben még szüksége lesz rájuk.

### <a name="export-the-source-data-to-azure-blob-storage"></a>Az adatok exportálása az Azure Blob storage 

Ebben a lépésben futtatja egy PowerShell-parancsfájlt, amely a bcp segítségével exportál az SQL-adatbázis egybesimított fájlokba, a virtuális Gépre, és az AzCopy segítségével ezeket a fájlokat az Azure Blob Storage másolhatja.

1. A távoli asztal segítségével csatlakoztassa a szimulált helyszíni virtuális Gépet.

2. Jelentkezzen be a virtuális gép, a következő parancsokat a PowerShell-ablakban.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    Az a `Destination` paraméter, cserélje le `<storage_account_name>` nevű a korábban létrehozott tárfiókot. Az a `StorageAccountKey` paraméter, a hozzáférési kulcsot a tárfiók használja.

3. Az Azure portálon győződjön meg arról, hogy az adatok másolja a Blob storage lépjen a tárfiókhoz, a Blob szolgáltatás kiválasztásával, és az megnyitása a `wwi` tároló. Végrehajtásával kerüli meg a táblák listáját kell megjelennie `WorldWideImporters_Application_*`.

### <a name="execute-the-data-warehouse-scripts"></a>A data warehouse parancsfájlok végrehajtása

1. A távoli asztali munkamenetből indítsa el az SQL Server Management Studio (SSMS). 

2. Connect to SQL Data Warehouse

    - Kiszolgáló típusa: adatbázis-kezelő
    
    - Kiszolgáló neve: `<dwServerName>.database.windows.net`, ahol `<dwServerName>` amikor központilag telepítette az Azure-erőforrások megadott név. Ez a név lekérheti az Azure-portálon.
    
    - Hitelesítési: SQL Server-hitelesítést. Amikor központilag telepítette az Azure-erőforrások, a megadott hitelesítő adatokat használja a `dwAdminLogin` és `dwAdminPassword` paraméterek.

2. Keresse meg a `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` mappa a virtuális Gépen. Ebben a mappában számsorrendben, végrehajtja a a parancsfájlok `STEP_1` keresztül `STEP_7`.

3. Válassza ki a `master` adatbázisban az SSMS és nyissa meg a `STEP_1` parancsfájl. Módosítsa az értéket a következő sort a jelszót, majd futtassa a parancsfájlt.

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. Válassza ki a `wwi` adatbázisban az SSMS. Nyissa meg a `STEP_2` parancsfájlt, és futtassa a parancsfájlt. Ha hibaüzenetet kap, győződjön meg arról, szemben a parancsfájlt futtatja a `wwi` adatbázis, és nem `master`.

5. Az SQL Data Warehouse egy új kapcsolat megnyitásához használja a `LoaderRC20` felhasználói és a jelszót a `STEP_1` parancsfájl.

6. Ezt a kapcsolatot, nyissa meg a `STEP_3` parancsfájl. A parancsfájlban állítsa be a következő értékeket:

    - Titkos kulcs: A hozzáférési kulcsot a tárfiók használja.
    - HELYE: A tárfiók nevét használja az alábbiak szerint: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`.

7. Az azonos kapcsolaton keresztül végrehajtani a parancsfájlok `STEP_4` keresztül `STEP_7` egymás után. Győződjön meg arról, hogy minden parancsprogram a következő futtatása előtt sikeresen befejeződik.

A SMSS, megjelenik egy `prd.*` a táblázatot a `wwi` adatbázis. A következő lekérdezés futtatásával ellenőrizheti, hogy létrejött-e az adatok: 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

### <a name="build-the-azure-analysis-services-model"></a>Az Azure Analysis Services-modell létrehozása

Ebben a lépésben létrehoz egy táblázatos modell, amellyel különféle adatok, az adatraktárból. A modell telepíti majd Azure Analysis Services.

1. A távoli asztali munkamenetből indítsa el az SQL Server Data Tools 2015.

2. Válassza a **File** (Fájl) > **New** (Új) > **Project** (Projekt) lehetőséget.

3. Az a **új projekt** párbeszédpanelen, a **sablonok**, jelölje be **üzleti intelligencia** > **Analysis Services**  >  **Analysis Services táblázatos projekt**. 

4. Nevet a projektnek, és kattintson a **OK**.

5. A a **táblázatos modellek tervezőjében** párbeszédablakban válassza **integrált munkaterület** és **kompatibilitási szintje** való `SQL Server 2017 / Azure Analysis Services (1400)`. Kattintson az **OK** gombra.

6. Az a **táblázatos modell Explorer** ablak, kattintson jobb gombbal a projektre, és válassza ki **importálás az adatforrás**.

7. Válassza ki **Azure SQL Data Warehouse** kattintson **Connect**.

8. A **Server**, adja meg az Azure SQL Data Warehouse kiszolgáló teljesen minősített neve. A **adatbázis**, adja meg `wwi`. Kattintson az **OK** gombra.

9. A következő párbeszédpanelen válassza ki a **adatbázis** hitelesítés és az Azure SQL Data Warehouse-felhasználónevet és jelszót adjon meg, és kattintson a **OK**.

10. Az a **Navigator** párbeszédpanelen jelölje be a jelölőnégyzeteket **prd. CityDimensions**, **prd. DateDimensions**, és **prd. SalesFact**. 

    ![](./images/analysis-services-import.png)

11. Kattintson a **terhelés**. Ha feldolgozása befejeződött, kattintson a **Bezárás**. Meg kell jelennie az adatok táblázatos nézetben.

12. Az a **táblázatos modell Explorer** ablak, kattintson jobb gombbal a projektre, és válassza ki **modell nézet** > **diagramnézet**.

13. Húzza a **[prd. SalesFact]. [Város WWI azonosítója]**  mezőről az **[prd. CityDimensions]. [Város WWI azonosítója]**  mező közötti kapcsolat kialakításához.  

14. Húzza a **[prd. SalesFact]. [Számlán dátum kulcs]**  mezőről az **[prd. DateDimensions]. [Dátum]**  mező.  
    ![](./images/analysis-services-relations.png)

15. Az a **fájl** menüben válasszon **összes mentése**.  

16. A **Megoldáskezelőben**, kattintson jobb gombbal a projektre, és válassza ki **tulajdonságok**. 

17. A **Server**, adja meg az Azure Analysis Services-példány URL-CÍMÉT. Ez az érték lekérheti az Azure portálon. A portálon, válassza ki az Analysis Services-erőforrást, az Áttekintés ablaktábláján kattintson, és keresse meg a **kiszolgálónév** tulajdonság. Hasonló lesz `asazure://westus.asazure.windows.net/contoso`. Kattintson az **OK** gombra.

    ![](./images/analysis-services-properties.png)

18. A **Megoldáskezelőben**, kattintson jobb gombbal a projektre, és válassza ki **telepítés**. Ha a rendszer kéri, jelentkezzen be Azure. Ha feldolgozása befejeződött, kattintson a **Bezárás**.

19. Az Azure-portálon az Azure Analysis Services-példány részletes adatainak megtekintése. Győződjön meg arról, hogy a modell megjelenik-e a modellek listája.

    ![](./images/analysis-services-models.png)

### <a name="analyze-the-data-in-power-bi-desktop"></a>A Power BI Desktop adatok elemzése

Ebben a lépésben szüksége lesz az Power BI-jelentés létrehozása az Analysis Services az adatokból.

1. A távoli asztali munkamenetből indítsa el a Power BI Desktop.

2. Kattintson az üdvözlő Scren **adatok beolvasása**.

3. Válassza ki **Azure** > **Azure Analysis Services-adatbázis**. Kattintson a **csatlakozás**

    ![](./images/power-bi-get-data.png)

4. Adja meg az Analysis Services-példány URL-CÍMÉT, majd kattintson a **OK**. Ha a rendszer kéri, jelentkezzen be Azure.

5. Az a **Navigator** párbeszédpanelen bontsa ki a központilag telepített táblázatos projektet, válassza ki a létrehozott, és kattintson a modell **OK**.

2. Az a **képi megjelenítések** ablaktáblán válassza előbb a **halmozott sáv diagrammá** ikonra. A jelentés nézetben a képi megjelenítés abba, hogy nagyobb átméretezése.

6. Az a **mezők** ablaktáblában bontsa ki a **prd. CityDimensions**.

7. A csomóponthúzási **prd. CityDimensions** > **WWI város azonosító** számára a **tengely well**.

8. A csomóponthúzási **prd. CityDimensions** > **Város** számára a **jelmagyarázat** is.

9. A mezők ablaktáblában bontsa ki a **prd. SalesFact**.

10. A csomóponthúzási **prd. SalesFact** > **kivételével adó** számára a **érték** is.

    ![](./images/power-bi-visualization.png)

11. A **Visual szint szűrők**, jelölje be **WWI város azonosító**.

12. Állítsa be a **szűrőtípus** való `Top N`, és állítsa be **elemek megjelenítése** való `Top 10`.

13. A csomóponthúzási **prd. SalesFact** > **kivételével adó** számára a **által érték** is

    ![](./images/power-bi-visualization2.png)

14. Kattintson a **szűrőt**. A képi megjelenítés a felső 10 összes értékesítés városonként jeleníti meg.

    ![](./images/power-bi-report.png)

A Power BI Desktop kapcsolatos további információkért lásd: [első lépések a Power BI Desktop](/power-bi/desktop-getting-started).

## <a name="next-steps"></a>További lépések

- A referencia-architektúrában kapcsolatos további információkért látogasson el a [GitHub-tárházban][ref-arch-repo-folder].
- További tudnivalók a [Azure építőelemeket][azbb-repo].

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

