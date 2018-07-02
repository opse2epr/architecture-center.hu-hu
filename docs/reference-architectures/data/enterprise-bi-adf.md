---
title: Az SQL-adatraktár és az Azure Data Factory automatizált vállalati BI
description: Egy Azure ELT munkafolyamat automatizálhatják az Azure Data Factory használatával
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: 36239ce88fa2a80a865a8883e2729d9b7b094268
ms.sourcegitcommit: d5db5b8ed7429f056130096d0ef4b249b564599a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/01/2018
ms.locfileid: "37141613"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>Az SQL-adatraktár és az Azure Data Factory automatizált vállalati BI

A referencia-architektúrában bemutatja, hogyan hajthat végre a növekményes betöltése egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kivonat-betöltési-átalakítási) folyamat. Az Azure Data Factory segítségével automatizálhatja a ELT folyamat. A feldolgozási sor Növekményesen helyez a legújabb OLTP adatokat a helyszíni SQL Server-adatbázist az SQL Data Warehouse. Tranzakciós adatok alakul egy táblázatos modell elemzés céljából. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

Ez az architektúra látható egy épít [az SQL Data Warehouse szolgáltatással vállalati BI](./enterprise-bi-sqldw.md), néhány fontos vállalati adatraktározási forgatókönyvekben szolgáltatásokkal bővíti, de.

-   A Data Factory használatával folyamatának Automation.
-   Növekményes betöltése.
-   Több adatforrást integrálásával.
-   Bináris adatok, például a földrajzi adatok és a képek betöltését.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

### <a name="data-sources"></a>Adatforrások

**Helyszíni SQL Server**. Az adatok a helyszíni SQL Server-adatbázisban található. A helyszíni környezetben, az az architektúra biztosítása a telepített SQL Server Azure virtuális gép üzembe helyezési parancsfájlok szimulálásához. [Wide World Importers OLTP-mintaadatbázist] [wwi] szolgál a forrás-adatbázisként.

**Külső adatok**. Az adatraktárak egy általános forgatókönyv integrálásához több adatforrás. A referencia-architektúrában betölt egy külső adathalmaz évente város feltöltések tartalmazza, amely integrálható a az OLTP adatbázis adatait. Használhatja ezeket az adatokat, mint az elemzések: "Nem minden régióban értékesítési növekedés nagyobb vagy populáció növekedésének?"

### <a name="ingestion-and-data-storage"></a>Adatfeldolgozást és adattárolási

**BLOB Storage**. A BLOB storage szolgál egy átmeneti területre a forrásadatok azt az SQL Data Warehouse betöltése előtt.

**Azure SQL Data Warehouse**. [Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer készült elemzés végrehajtásához, nagy. Támogatja a jelentős olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas a nagy teljesítményű analytics futtatásához. 

**Az Azure Data Factory**. [Adat-előállító] [adf] koordinálja és automatizálja az adatmozgás és adatok átalakítása felügyelt szolgáltatás. Ebben az architektúrában koordinálja a ELT folyamat különböző szakaszaiban.

### <a name="analysis-and-reporting"></a>Elemzési és jelentéskészítési

**Az Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely a modellezési képességekkel adatokat biztosít. A szemantikai modell Analysis Services-bA betöltött.

**Power BI**. A Power BI eszközcsomagot jelent üzleti analytics üzleti elemzések készítése adatok elemzésére. Ebben az architektúrában lekéri az Analysis Servicesben tárolt a szemantikai modellben.

### <a name="authentication"></a>Hitelesítés

**Az Azure Active Directory** (az Azure AD) hitelesíti a Power BI Analysis Services-kiszolgálóhoz csatlakozó felhasználók számára.

Adat-előállító segítségével is használja az Azure AD az SQL Data Warehouse egy egyszerű vagy egy felügyelt szolgáltatás identitásának (MSI) használatával hitelesíteni. Az egyszerűség kedvéért központi telepítésre példát a SQL Server-hitelesítést használ.

## <a name="data-pipeline"></a>Adatfolyamat

[Azure Data Factory] [adf], a folyamat feladat koordinálja tevékenységek logikai csoportosítása &mdash; ebben az esetben be- és az SQL Data Warehouse-adatok átalakítása. 

A referencia-architektúrában definiál egy fő folyamat, amely gyermek folyamatok sorozatát futtatja. Minden gyermek csővezeték adatokat tölt be egy vagy több data warehouse tábláiba.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Növekményes betöltése

Egy automatikus ETL vagy ELT folyamat futtatásakor csak a megváltozott, mióta az előző futtatása adatok betöltésére. Ez a lehetőség egy *növekményes terhelés*, és nem a betölti az összes adat egy teljes terhelése. Hajtsa végre egy növekményes terhelés, kell azonosítani tudja, melyik adatok változásairól. A leggyakrabban használt módszer az, hogy használja a *magas vízjel alapján* érték, amely azt jelenti, hogy a legutóbbi értékét a forrástáblában, dátum és idő oszlop vagy egy egyedi egészszám-oszloppal néhány oszlop nyomon követése. 

SQL Server 2016-os verziótól kezdődően használhatja [ideiglenes táblák](/sql/relational-databases/tables/temporal-tables). Ezek a rendszerverzióval ellátott táblákon, amely az adatmódosítások teljes megőrzése. Az adatbázis-kezelő automatikusan egy külön tábla minden változás előzményadatait rögzíti. A korábbi adatok lekérheti a lekérdezés egy FOR SYSTEM_TIME záradék hozzáadásával. Belsőleg az adatbázismotor lekérdezi a előzménytábla, ez azonban átlátható az alkalmazás. 

> [!NOTE]
> Az SQL Server korábbi verzióit használhatja [adatváltozás-rögzítés](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC). Ezt a módszert nem ideiglenes táblák, mint kisebb kényelmes, mert van egy különálló módosítási táblából, és kötetblokkok változásait időbélyeg helyett olyan naplósorszámot. 

A historikus táblák hasznosak dimenzió adatokat, amelyek idővel megváltozhat. A ténytáblák általában egy módosítható tranzakció, például egy pénztári, ebben az esetben megőrzi a rendszer verziójának előzményei értelmetlen jelölik. Ehelyett tranzakciók általában kell egy olyan oszlopot, amely a tranzakció dátuma, amely a vízjel értékként használható jelöli. Például a Wide World Importers OLTP adatbázismodell a Sales.Invoices és Sales.InvoiceLines kell egy `LastEditedWhen` mező, amely alapértelmezés szerint az `sysdatetime()`. 

Az általános folyamat a ELT adatcsatorna itt található:

1. Minden táblához adatbázisában, amikor a legutóbbi ELT feladat futtatta levágási idejének nyomon követésére. Az adatraktárban tárolja az adatait. (A kezdeti beállítás mindig lesz állítva "1-1-1900".)

2. Során az adatok exportálása a lépést, a megadott idő objektumnak átadott paraméterként adatbázisában tárolt eljárások. A tárolt eljárások lekérdezés megváltozott vagy a megadott idő után létrehozott bejegyzésekhez. Az értékesítési ténytábla a `LastEditedWhen` oszlopot használja. A dimenzió adatok rendszerverzióval ellátott historikus táblákon szolgálnak.

3. Az adatáttelepítés végeztével frissítse a tábla tárolja a megadott időpontban.

Is célszerű jegyezze fel a *Leszármaztatás* az egyes ELT futtatásához. Egy adott rekord a Leszármaztatás társítja a rekord, futtassa a ELT előállított adatokat. Minden táblának megjelenítő kezdő és záró betöltési idők minden egyes ETL futtatásához egy új Leszármaztatás rekord jön létre. A rekordokban Leszármaztatás kulcsokat a dimenzió és a ténytáblákat táblákban tárolódnak.

![](./images/city-dimension-table.png)

Az adatraktár egy új köteg adatok tölti be, miután frissítése az Analysis Services táblázatos modell. Lásd: [aszinkron frissítse a REST API-val](/azure/analysis-services/analysis-services-async-refresh).

## <a name="data-cleansing"></a>Adatok tisztítása

Adatok tisztítására a ELT folyamat részének kell lennie. A referencia-architektúrában egy hibás adat forrása a város feltöltési táblában, ahol egyes városokat rendelkeznek nulla feltöltési esetleg mert adatot nem volt elérhető. A feldolgozás során az a ELT-feldolgozási folyamat ezen városokat eltávolítja a város feltöltési táblából. Az előkészítési táblák, nem pedig külső táblák tisztításokat végezhet.

Ez a tárolt eljárás, amely nulla feltöltési rendelkező városokat eltávolítja a város feltöltési táblából. (A forrásfájl található [Itt](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).) 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>Külső adatforrások

Az adatraktárak gyakran több forrásból származó adatok összesítése. A referencia-architektúrában betölti demográfiai adatait tartalmazó külső adatforráshoz. Ez az adatkészlet érhető el az Azure blob storage részeként a [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) minta.

Az Azure Data Factory közvetlenül átmásolhatja blob storage használata a [blob-tároló összekötő](/azure/data-factory/connector-azure-blob-storage). Azonban az összekötő számára szükséges kapcsolati karakterláncot vagy a közös hozzáférésű jogosultságkód, ezért nem használható nyilvános olvasási hozzáférés a blob másolása. Megoldás a PolyBase használatával létrehoz egy külső táblát a Blob storage keresztül, és másolja a külső táblákhoz az SQL Data Warehouse. 

## <a name="handling-large-binary-data"></a>Nagy bináris adatok kezelése 

A forrásadatbázis városokat táblához birtokló hely oszlop egy [geográfiai](/sql/t-sql/spatial-geography/spatial-types-geography) térbeli adatok típusa. Az SQL Data Warehouse nem támogatja a **geográfiai** írja be a natív módon, így ez a mező alakítja át a **varbinary** típus betöltése során. (Lásd: [nem támogatott adattípusú megoldásai](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)

Ugyanakkor támogatja a PolyBase az oszlop maximális méretet `varbinary(8000)`, ami azt jelenti, hogy néhány adat csonkolódik. A megoldás a probléma, hogy feloszthatja az adatok adattömbökbe exportálás során, és majd állítják vissza a adattömböket, az alábbiak szerint:

1. A hely oszlopban egy ideiglenes előkészítési tábla létrehozása.

2. Mindegyik városhoz ossza fel a helyadatok 8000 bájtos adattömböket, ami azt eredményezi, 1 &ndash; mindegyik városhoz N sorát.

3. Az adattömbök elhelyezkedését, használja a T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) sorok átalakítás oszlopokba, és ezután az egyes város oszlopértékeit összefűzésére operátor.

A probléma az, hogy minden egyes város felosztása sorok, a földrajzi adatok méretétől függően különböző számú. A PIVOT operátorban működjön minden város azonos számú sort kell rendelkeznie. -Re, a T-SQL-lekérdezések (amelyet meg lehet tekinteni [here][MergeLocation]) létezik néhány ütés üres értékeket tartalmazó sorok ismételt kitöltésére úgy, hogy minden város azonos számú oszlopot a PowerPivot után. Az eredményül kapott lekérdezés elemről kiderül, hogy sokkal gyorsabb, mint a sorokat egy ismétlése egyszerre kell.

Ugyanezt a megközelítést képadatok szolgál.

## <a name="slowly-changing-dimensions"></a>Lassú a dimenziók módosítása

Dimenzió adatok viszonylag statikus, de ez módosítható. A termék például egy másik termékkel kategóriához beolvasása rendelik. Nincsenek lassan módosítása a dimenziók kezelésével több megközelítés közül. Egy közös technika, úgynevezett [típus 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), adjon hozzá egy új rekordot, amikor egy dimenzió módosítások-hoz. 

Ahhoz, hogy a típus 2 megközelítés implementálása a dimenziótáblák kell további oszlopokat, amelyek egy adott rekord érvényes dátumtartományt. Is a forrásadatbázisból elsődleges kulcsok fog lehet duplikálni, így a dimenziótáblában egy mesterséges elsődleges kulccsal kell rendelkeznie.

A következő kép bemutatja a Dimension.City tábla. A `WWI City ID` oszlop az elsődleges kulcsot a forrás-adatbázisból. A `City Key` oszlop, mint egy mesterséges kulcs az ETL-folyamat során. Is figyelje meg, hogy a tábla tartalmaz `Valid From` és `Valid To` oszlop, adja meg a tartomány minden egyes sorára volt érvényes. Aktuális értékei egy `Valid To` egyenlő "9999-12-31'.

![](./images/city-dimension-table.png)

Ez a megközelítés előnye megőrzi az előzményadatok, amely elemzés igen hasznos lehet. Azonban azt is jelenti a ugyanaz az entitás több sort lesz. Például az alábbiakban a megfelelő rekordok `WWI City ID` = 28561:

![](./images/city-dimension-table-2.png)

Minden egyes forgalmi tény szeretné, hogy a tény társítandó város dimenzió tábla, egy sorban a számla dátumnak megfelelő. Az ETL-folyamat részeként egy további oszlop létrehozása, amely 

A következő T-SQL-lekérdezés ideiglenes táblát hoz létre minden egyes számla társítja a megfelelő város kulcsot a város dimenzió táblából.

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

Ez a táblázat az értékesítési ténytábla oszlopát feltölti a következőkre használható:

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

Ez az oszlop lehetővé teszi, hogy a Power BI-lekérdezést a helyes város bejegyzés egy adott értékesítési számla.

## <a name="security-considerations"></a>Biztonsági szempontok

Használhatja a fokozott biztonság érdekében [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) csak a virtuális hálózat az Azure-szolgáltatások erőforrások biztonságossá tételére. Teljes távolítja el ezeket az erőforrásokat, átengedi a forgalmat a virtuális hálózat csak a nyilvános Internet-hozzáférést.

Ezt a módszert használja hozzon létre egy Vnetet az Azure-ban, és létrehozhat saját Szolgáltatásvégpontok az Azure-szolgáltatásokhoz. Ezek a szolgáltatások majd az adott virtuális hálózati forgalom korlátozódik. Is elérhető azokat a helyi hálózatról átjárón keresztül.

Vegye figyelembe a következő korlátozások vonatkoznak:

- Időben a referencia-architektúrában jött létre, Szolgáltatásvégpontok támogatott Azure tárolási és az Azure SQL Data Warehouse, de nem Azure Analysis Service VNet. A legfrissebb állapotának [Itt](https://azure.microsoft.com/updates/?product=virtual-network). 

- Ha Szolgáltatásvégpontok engedélyezve vannak az Azure Storage, a PolyBase az SQL Data Warehouse nem lehet másolni adatok a tároló. A megoldás a probléma van. További információkért lásd: [VNet Szolgáltatásvégpontok használata az Azure storage hatását](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage). 

- Az Azure Storage helyez át adatokat a helyszíni, akkor nyilvános IP-címek engedélyezési listája a helyszíni vagy ExpressRoute. További információkért lásd: [virtuális hálózatok védelmét biztosító Azure-szolgáltatások](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).

- Ahhoz, hogy az Analysis Services adatokat olvasni az SQL Data Warehouse, telepítse a Windows virtuális Gépet a virtuális hálózat, amely tartalmazza az SQL Data Warehouse szolgáltatásvégpontot. Telepítés [Azure a helyszíni adatátjáró](/azure/analysis-services/analysis-services-gateway) a virtuális Gépet. Az Azure Analysis service majd csatlakozni a data gateway.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A központi telepítés a referenciaarchitektúra [GitHub] [ref-architektúrája-tárház-mappa] érhető el. A következőket helyezi üzembe:

  * Windows virtuális gép egy helyi adatbázis-kiszolgáló szimulálásához. SQL Server 2017 és a kapcsolódó eszközök, valamint Power BI Desktop tartalmaz.
  * Egy Azure storage-fiók, amely Blob-tároló az SQL Server adatbázisból exportált adatok tárolásához.
  * Egy Azure SQL Data Warehouse-példányhoz.
  * Egy Azure Analysis Services-példányon.
  * Az Azure Data Factory és a Data Factory-folyamathoz, ELT feladat.

### <a name="prerequisites"></a>Előfeltételek

1. Klónozás, elágazás vagy az [Azure hivatkozás architektúrák] [ref-architektúrája-tárház] GitHub tárház zip-fájl letöltése.

2. Telepítse az [Azure építőelemeket] [azbb-wiki] (azbb).

3. A Power BI Desktop kapcsolatos további információkért lásd: első lépések a Power BI Desktop.

    ```bash
    az login  
    ```

### <a name="variables"></a>Változók

A következő lépések bizonyos felhasználói változók tartalmazzák. Szüksége lesz az Ön által meghatározott értékeket cseréli le.

- `<data_factory_name>`. Adat-előállító neve.
- `<analysis_server_name>`. Analysis Services-kiszolgáló neve.
- `<active_directory_upn>`. Az Azure Active Directory egyszerű felhasználónév (UPN). Például: `user@contoso.com`.
- `<data_warehouse_server_name>`. Az SQL Data Warehouse-kiszolgáló neve.
- `<data_warehouse_password>`. Az SQL Data Warehouse rendszergazdai jelszót.
- `<resource_group_name>`. Az erőforráscsoport neve.
- `<region>`. Az Azure-régió, hova szeretné telepíteni az erőforrásokat.
- `<storage_account_name>`. Tárfiók neve. Hajtsa végre a [elnevezési szabályait](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) Storage-fiókok.
- `<sql-db-password>`. SQL Server bejelentkezési jelszót.

### <a name="deploy-azure-data-factory"></a>Az Azure Data Factory telepítése

1. Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-tárház] [ref-architektúrája-tárház].

2. A következő parancsot az Azure parancssori felület futtatásával hozzon létre egy erőforráscsoportot.  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    Adja meg, amely támogatja az SQL Data Warehouse, az Azure Analysis Services és a Data Factory v2 egy régiót. Lásd: [régiónként Azure termékek](https://azure.microsoft.com/global-infrastructure/services/)

3. A következő parancsot

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

Ezt követően az Azure portál használatával a hitelesítési kulcs lekérése az Azure Data Factory [integrációs futásidejű](/azure/data-factory/concepts-integration-runtime), az alábbiak szerint:

1. Az a [Azure Portal](https://portal.azure.com/), keresse meg a Data Factory-példány.

2. A Data Factory paneljén kattintson **Szerző & figyelő**. Az Azure Data Factory portálon ekkor megnyílik egy másik böngészőablakban.

    ![](./images/adf-blade.png)

3. Az Azure Data Factory-portálon válassza ki a ceruza ikonra ("Szerző"). 

4. Kattintson a **kapcsolatok**, majd válassza ki **integrációs futtatókörnyezetek**.

5. A **sourceIntegrationRuntime**, kattintson a ceruza ikonra ("Edit").

    > [!NOTE]
    > A portálon jelennek meg a "nem érhető el" állapotának. Ez egy várható üzenet, amíg a helyszíni kiszolgáló központi telepítése.

6. Található **Key1** , és másolja a hitelesítési kulcs értékét.

Szüksége lesz a hitelesítési kulcs a következő lépéssel.

### <a name="deploy-the-simulated-on-premises-server"></a>A referencia-architektúrában kapcsolatos további információkért látogasson el a GitHub-tárházban.

Ebben a lépésben egy virtuális Gépet, amely tartalmazza az SQL Server 2017 és a kapcsolódó eszközök szimulált helyszíni kiszolgálóként központilag telepíti. Azt is betölti a [Wide World Importers OLTP adatbázis] [wwi] az SQL-kiszolgálóra.

1. Keresse meg a `data\enterprise_bi_sqldw_advanced\onprem\templates` a tárház mappát.

2. Az a `onprem.parameters.json` fájlt, keresse meg `adminPassword`. Ez az a jelszót az SQL Server Virtuálisgép jelentkezni. Cserélje le az érték egy másik jelszót.

3. A ugyanazt a fájlt, keresse meg a `SqlUserCredentials`. Ez a tulajdonság határozza meg az SQL Server fiók hitelesítő adatait. A jelszó cseréje egy másik értéket.

4. Ugyanebben a fájlban, illessze be azokat az integrációs futásidejű hitelesítési kulcs a `IntegrationRuntimeGatewayKey` paraméter, a lent látható módon:

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. A következő parancsot.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

Ez a lépés a 20-30 percet is igénybe vehet. Ez magában foglalja a fut egy [DSC](/powershell/dsc/overview) parancsprogramot, az eszközök telepítése, majd állítsa vissza az adatbázist. 

### <a name="deploy-azure-resources"></a>Azure-erőforrások telepítése

Ez a lépés látja el az SQL Data Warehouse, az Azure Analysis Services és a Data Factory.

1. Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-tárház] [ref-architektúrája-tárház].

2. A következő Azure CLI parancsot. Cserélje le a paraméterértékeket csúcsos zárójelek látható.

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - A `storageAccountName` paraméter kell követnie a [elnevezési szabályait](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) Storage-fiókok. 
    - Az a `analysisServerAdmin` paraméter, használja az Azure Active Directory egyszerű felhasználónév (UPN).

3. A következő parancsot az Azure parancssori felület a hozzáférési kulcsot a tárfiók eléréséhez. Ezt a kulcsot a következő lépésben fogja használni.

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. A következő Azure CLI parancsot. Cserélje le a paraméterértékeket csúcsos zárójelek látható. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    A kapcsolati karakterláncokkal rendelkezik karakterláncrészletek szög zárójelbe kell cserélni. A `<storage_account_key>`, a kulcs, amely az előző lépésben kapott. A `<sql-db-password>`, a megadott SQL Server-fiókja jelszavát használja a `onprem.parameters.json` korábban fájlt.

### <a name="run-the-data-warehouse-scripts"></a>A data warehouse parancsfájlok futtatása

1. Az a [Azure Portal](https://portal.azure.com/), a helyszíni virtuális Gépet, amelynek neve található `sql-vm1`. A felhasználónév és jelszó a virtuális gép számára meg van adva a `onprem.parameters.json` fájlt.

2. Kattintson a **Connect** és a távoli asztal használatával csatlakoztassa a virtuális Gépet.

3. A távoli asztali munkamenetből nyisson meg egy parancssort, és keresse meg a virtuális Gépen a következő mappát:

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. Futtassa az alábbi parancsot:

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    A `<data_warehouse_server_name>` és `<data_warehouse_password>`, az adatraktár-kiszolgáló nevét és a korábbi jelszó használata.

Ebben a lépésben ellenőrzéséhez használhatja az SQL Server Management Studio (SSMS) az SQL Data Warehouse-adatbázishoz való kapcsolódáshoz. Az adatbázis táblasémákat kell megjelennie.

### <a name="run-the-data-factory-pipeline"></a>Futtassa a Data Factory-folyamat

1. A azonos a távoli asztal munkamenetet nyissa meg egy PowerShell-ablakot.

2. Futtassa az alábbi PowerShell-parancsot. Válasszon **Igen** megjelenésekor.

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. Futtassa az alábbi PowerShell-parancsot. Adja meg, amikor a rendszer kéri az Azure hitelesítő adatait.

    ```powershell
    Connect-AzureRmAccount 
    ```

4. Futtassa a következő PowerShell-parancsokat. Cserélje le a csúcsos zárójelek értékeket.

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    Összes értékesítés: = SUM ('Tény értékesítés' [összes, beleértve a adó])
    ```

4. Repeat these steps to create the following measures:

    ```
    Évek száma: (MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber])) + 1 =
    
    Feltöltési megkezdése: = CALCULATE (SUM ("Tény CityPopulation" [feltöltési]), szűrő ("Tény CityPopulation", "Tény CityPopulation" [YearNumber] = MIN('Fact CityPopulation'[YearNumber])))
    
    Befejezési feltöltési: = CALCULATE (SUM ("Tény CityPopulation" [feltöltési]), szűrő ("Tény CityPopulation", "Tény CityPopulation" [YearNumber] = MAX('Fact CityPopulation'[YearNumber])))
    
    CAGR: = IFERROR ((([a feltöltési befejező] / [kezdő feltöltési]) ^ (1 / [évek száma]))-1,0)
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
