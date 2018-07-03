---
title: Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban
description: Az Azure-ban egy ELT munkafolyamat automatizálása az Azure Data Factory használatával
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: ffd75ba8c57a9afbc6abad61f21f738c644c9bc8
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142281"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban

Ez a referenciaarchitektúra bemutatja, hogyan hajthat végre a növekményes betöltése egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kinyerési, betöltési, átalakítási) folyamatot. Azure Data Factory használatával az ELT folyamatok automatizálásához. A folyamat növekményes helyezi át a legújabb OLTP adatokat a helyszíni SQL Server-adatbázisból az SQL Data Warehouse-bA. Tranzakciós adatok átalakításának egy táblázatos modell elemzés céljából. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

Ez az architektúra épít látható [Enterprise BI és az SQL Data Warehouse](./enterprise-bi-sqldw.md), de bizonyos funkciók, fontos vállalati adatraktározási forgatókönyvekben.

-   Automation használatával a Data Factory-folyamatot.
-   Növekményes betöltés.
-   Több adatforrás integrálásával.
-   Bináris adatot, például térinformatikai adatok és lemezképek betöltése.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

### <a name="data-sources"></a>Adatforrások

**A helyszíni SQL Server**. Az adatok helyszíni SQL Server-adatbázis található. A helyszíni környezetben, az architektúra üzembe helyezése az Azure-ban telepített SQL Server virtuális gép üzembe helyezési szkriptjei szimulálásához. [Wide World Importers OLTP-mintaadatbázist] [wwi] a forrásadatbázis szolgál.

**Külső adatok**. Adattárházak gyakran előfordul, hogy több adatforrás integrálhatja. Ez a referenciaarchitektúra egy külső adatkészlet, amely tartalmazza a város feltöltések évek szerint, és integrálja az OLTP adatbázis adataival tölti be. Használhatja ezeket az adatokat például az elemzésekhez: "Nem minden régióban eladások növekedési nagyobb vagy népességnövekedés?"

### <a name="ingestion-and-data-storage"></a>Adatfeldolgozási és az adatok tárolása

**A BLOB Storage-**. A BLOB storage lesz egy átmeneti területre a forrásadatok mielőtt betöltené azokat az SQL Data Warehouse-bA.

**Azure SQL Data Warehouse**. [Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer analytics végre, a nagy mennyiségű adat. Támogatja a nagy olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas nagy teljesítményű elemzési futtatásához. 

**Az Azure Data Factory**. [A data Factory] [adf] egy felügyelt szolgáltatás, amellyel előkészíthető és automatizálható az adatok áthelyezését és átalakítását. Ebben az architektúrában koordinálja a ELT folyamat különböző szakaszaiban.

### <a name="analysis-and-reporting"></a>Elemzés és jelentéskészítés

**Az Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely adatmodellezési képességekkel. A szemantikai modell betöltött Analysis Services.

**Power BI**. A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese. Ebben az architektúrában a szemantikai modell az Analysis Servicesben tárolt kérdezi le.

### <a name="authentication"></a>Hitelesítés

**Az Azure Active Directory** (Azure AD) hitelesíti a felhasználók, akik a Power bi-ban az Analysis Services-kiszolgálóhoz csatlakozhat.

A Data Factory használatával is az Azure AD hitelesítése az SQL Data Warehouse, egy szolgáltatásnév vagy a Felügyeltszolgáltatás-identitás (MSI). Az egyszerűség kedvéért a központi telepítésre példát az SQL Server-hitelesítést használ.

## <a name="data-pipeline"></a>Adatfolyamat

Az [Azure Data Factoryban] [adf], egy folyamatot a feladat koordinálja a tevékenységek logikus csoportosításai egy &mdash; ebben az esetben be- és az adatok átalakítása az SQL Data Warehouse-bA. 

Ez a referenciaarchitektúra egy fő folyamatot, amely gyermek folyamatok sorozatát futtatja határozza meg. Minden egyes gyermek folyamat adatokat tölt be egy vagy több adattárház táblái.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Növekményes betöltés

Automatikus ETL vagy ELT folyamat futtatásakor a leghatékonyabb csak, mivel az előző futtatása az adatok betöltéséhez. Ezt nevezzük egy *növekményes betöltés*, és ne pedig a teljes terhelés, az adatok betöltésekor. Egy növekményes betöltési végrehajtásához kell azonosítani tudja, mely adatok módosultak. A leggyakrabban használt módszer az, hogy használjon egy *magas vízjelbe beleszámított* érték, ami azt jelenti, hogy nyomon követése a legújabb értékeket néhány oszlop a forrástábla, vagy egy dátum-idő oszlop, vagy egy egyedi egész számokat tartalmazó oszlopot. 

Az SQL Server 2016 kezdve használhatja [időbeli verziózású táblák](/sql/relational-databases/tables/temporal-tables). Ezek a rendszerverzióval ellátott táblákon, hogy az adatok teljes előzményeit. Az adatbázismotor automatikusan rögzíti a külön előzménytábla minden módosítási előzményeit. Az előzményadatok lekérdezheti, ha egy FOR SYSTEM_TIME záradékot ad hozzá egy lekérdezést. Belsőleg az adatbázismotor lekérdezi az előzménytáblában, de ez átlátható az alkalmazás. 

> [!NOTE]
> Az SQL Server korábbi verzióit használhatja [adatváltozás-rögzítési](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC). Ez a megközelítés akkor időbeli verziózású táblák,-nál kevesebb kényelmes, mert rendelkezik egy különálló módosítási táblából, és kötetblokkok változásait a napló sorszáma, nem pedig egy időbélyeg. 

Historikus táblák dimenzió az adatokat, amelyek idővel hasznosak. A ténytáblák általában egy nem módosítható tranzakció, például egy rendelést a táblagépükről, ebben az esetben a rendszer verziójának előzményei tartja értelmetlen képviseli. Ehelyett a tranzakciók általában rendelkeznek egy oszlopot, amely a tranzakció dátuma, amelyek használhatók a küszöbértékek jelöli. Például a Wide World Importers OLTP adatbázis a Sales.Invoices és Sales.InvoiceLines tábla rendelkezik egy `LastEditedWhen` mező, amely alapértelmezés szerint a `sysdatetime()`. 

Itt látható az általános folyamat az ELT folyamatok:

1. A forrásadatbázis minden táblához nyomon követheti a megszakítási idő, amikor az utolsó ELT feladat futott. Ez az információ Store az adatraktárban. (A kezdeti beállítás, minden esetben vannak beállítva: 1-1-1900-hoz ".)

2. Során az adatok exportálása a lépést, a megszakítási idő tárolt eljárások a forrásadatbázis átadott paraméterként. Ezek tárolt eljárások lekérdezést, amelyet a megszakítási idő után létrehozott vagy megváltozott rekordokat. A Sales (tény) tábla a `LastEditedWhen` oszlopot használja. A dimenzió adatok rendszerverzióval ellátott historikus táblákon szolgálnak.

3. Ha az adatok migrálása befejeződött, frissítse a tábla tárolja a megszakítási idő.

Emellett akkor is hasznos, jegyezze fel a *leszármaztatási* esetében minden egyes Futtatás ELT. Egy adott rekord a leszármaztatási társítja, amely az adatok előállított rekordot a ELT, futtassa a. Az egyes ETL futtatások mindegyik táblához, megjeleníti a kezdési és befejezési lapbetöltési idők leszármaztatási új rekord jön létre. A leszármaztatási kulcsok minden egyes rekord a dimenzió és a ténytáblákat táblákban tárolódnak.

![](./images/city-dimension-table.png)

Után az adatok egy új batch tölti be az adatraktár, frissítse az Analysis Services táblázatos modellt. Lásd: [aszinkron frissítése a REST API-val](/azure/analysis-services/analysis-services-async-refresh).

## <a name="data-cleansing"></a>Adattisztító

Adattisztítás ELT folyamatának részeként kell lennie. Ez a referenciaarchitektúra egy hibás adatforrás a város population táblában, ahol egyes városoknak rendelkezik nulla population például mert nincs adat nem volt elérhető. A feldolgozás során az ELT folyamatok eltávolítja a kiválasztott városok az városa population tábla. Hajtsa végre az előkészítési táblák ahelyett, hogy a külső táblák adattisztító.

Íme a tárolt eljárást, amely a várost, a nulla population eltávolítja az városa Population táblából. (A forrásfájl annak [Itt](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).) 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>Külső adatforrások

Adattárházak gyakran több forrásból származó adatok egyesíthetők. Ez a referenciaarchitektúra egy külső adatforrás, amely tartalmazza a demográfiai adatokat tölt be. Ez az adatkészlet érhető el az Azure blob storage-ban részeként a [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) minta.

Az Azure Data Factory másolhatja, közvetlenül a blob storage használatával a [blob storage-összekötő](/azure/data-factory/connector-azure-blob-storage). Azonban az összekötő számára szükséges kapcsolati karakterlánc vagy közös hozzáférésű jogosultságkódot, így nem használható nyilvános olvasási hozzáférés a blob másolásához. Áthidaló megoldásként használhatja a PolyBase külső tábla létrehozása tárt a Blob storage, és másolja a külső táblák az SQL Data Warehouse-bA. 

## <a name="handling-large-binary-data"></a>Nagy méretű bináris adatok kezelése 

A forrás-adatbázisban, a városok táblának van egy hely oszlopot tartalmazó egy [földrajzi](/sql/t-sql/spatial-geography/spatial-types-geography) térbeli adatok típusa. Az SQL Data Warehouse nem támogatja a **földrajzi** írja be a natív módon, így ez a mező alakítja át egy **varbinary** típus betöltése során. (Lásd: [nem támogatott adattípusok megoldásai](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)

Azonban a PolyBase támogatja egy oszlop maximális mérete `varbinary(8000)`, ami azt jelenti, hogy néhány adat csonkolódik. A probléma áthidaló felosztása a adatokat adattömbökre exportálás során, és ezután szétbontani módon az adattömböket:

1. Hozzon létre egy átmeneti előkészítési táblába, az a hely oszlopban.

2. Mindegyik városhoz ossza fel a helyadatok 8000 bájtos adattömböket, ami 1 &ndash; mindegyik városhoz N sorát.

3. Az adattömbök szétbontani, használja a T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operátor sorok átalakítása oszlopokat, és majd összefűzi a minden Város oszlop értékeit.

A kihívás abban áll, hogy minden egyes az városa felosztása egy eltérő mennyiségű sor, a földrajzi adatok méretétől függően. A PIVOT operátorban működjön minden az városa ugyanannyi sort kell rendelkeznie. Működnek, a T-SQL-lekérdezés (amelyet meg lehet tekinteni [here][MergeLocation]) hajtja végre bizonyos trükköket üres értékeket tartalmazó sorok ismételt kitöltésére, hogy minden városhoz azonos számú oszlopot a pivot után. Az eredményül kapott lekérdezés elemről kiderül, hogy sokkal gyorsabb, mint a sorokat egy ismétlése egyszerre kell.

Ugyanezzel a módszerrel képadatok szolgál.

## <a name="slowly-changing-dimensions"></a>Lassan változó dimenzió

Dimenzió adatok viszonylag statikusak, de módosíthatja azt. Termék például egy másik termékkel kategóriához első hozzárendelni. Nincsenek a lassan változó dimenzió kezelése több megközelítés közül. Egy közös leképezésnek hívott technika [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), adjon hozzá egy új rekordot, amikor egy dimenzió módosítások. 

A Type 2 megközelítés megvalósításához dimenziótáblák kell további oszlopokat, amelyek egy adott rekord érvényes dátumtartományt. Is a forrásadatbázisból elsődleges kulcsok többszörözni, így a táblát egy mesterséges elsődleges kulccsal kell rendelkeznie.

Az alábbi képen látható a Dimension.City tábla. A `WWI City ID` oszlop az elsődleges kulcsot a forrásadatbázisból. A `City Key` oszlop egy az ETL-folyamat során létrehozott mesterséges kulcsot. Figyelje meg, hogy a tábla rendelkezik `Valid From` és `Valid To` oszlopot, így tartományának megadása, ha minden sor volt érvényes. Aktuális értékek rendelkezik egy `Valid To` egyenlő "9999-12-31'.

![](./images/city-dimension-table.png)

Ez a megközelítés az az előnye, megőrzi az előzményadatok, amely elemzéshez hasznos lehet. Azonban azt is jelenti az ugyanazon entitás több sor lesz. Például az alábbiakban a megfelelő rekordok `WWI City ID` = 28561:

![](./images/city-dimension-table-2.png)

Az egyes értékesítési egyedkapcsolat szeretné a tény társítása az városa dimenziótábla, egyetlen sor invoice Date megfelelő. Az ETL-folyamat részeként egy további oszlop létrehozása, amely 

A következő T-SQL-lekérdezést hoz létre egy ideiglenes táblát, amely összekapcsolja minden számlán a megfelelő várost kulcsot az városa dimenzió táblából.

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

Ez a táblázat egy oszlopot a értékesítés ténytáblában feltöltésére szolgál:

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

Ez az oszlop lehetővé teszi, hogy a Power BI lekérdezés keresse meg a megfelelő várost rekordot egy megadott értékesítési számla.

## <a name="security-considerations"></a>Biztonsági szempontok

A fokozott biztonság érdekében használhat [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) biztonságossá tétele Azure-szolgáltatási erőforrások a virtuális hálózaton. Ezzel eltávolítja ezeket az erőforrásokat, így csak a virtuális hálózatból érkező forgalom nyilvános internetkapcsolaton keresztüli hozzáférés teljes mértékben.

Ezzel a módszerrel hozhat létre egy Vnetet az Azure-ban, és hozzon létre saját Szolgáltatásvégpontok az Azure-szolgáltatásokhoz. Ezeket a szolgáltatásokat majd korlátozva vannak a forgalom a virtuális hálózatról. Is elérheti azokat a helyszíni hálózatból egy átjárón keresztül.

Vegye figyelembe a következő korlátozások vonatkoznak:

- A időben Ez a referenciaarchitektúra lett létrehozva, a virtuális hálózati Szolgáltatásvégpontok Azure Storage és Azure SQL Data warehouse-ba, de az Azure Analysis Service a nem támogatott. A legfrissebb állapotának [Itt](https://azure.microsoft.com/updates/?product=virtual-network). 

- Ha a Szolgáltatásvégpontok engedélyezve vannak az Azure Storage, a PolyBase nem adatmásolás Storage-ból az SQL Data Warehouse-bA. Nincs egy megoldás erre a problémára. További információkért lásd: [hatását a virtuális hálózati Szolgáltatásvégpontok használatával és az Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage). 

- Adatok áthelyezése a helyszínről az Azure Storage-ba, szüksége lesz a nyilvános IP-címeket a helyszíni vagy ExpressRoute. További információkért lásd: [virtuális hálózatok biztonságossá tétele Azure-szolgáltatások](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).

- Ahhoz, hogy az Analysis Services adatokat olvasni az SQL Data Warehouse, Windows virtuális gép üzembe helyezése a virtuális hálózathoz, amely tartalmazza az SQL Data Warehouse szolgáltatásvégpontot. Telepítés [Azure a helyszíni adatátjáró](/azure/analysis-services/analysis-services-gateway) a virtuális gépen. Kapcsolódjon az Azure elemzési szolgáltatás az átjárót.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra egy üzemelő példánya [GitHub] [ref-arch-adattár-mappa] érhető el. A következőket helyezi üzembe:

  * Windows virtuális gép egy helyszíni adatbázis-kiszolgáló szimulálásához. Ez magában foglalja az SQL Server 2017-ben és a kapcsolódó eszközök, Power BI Desktop együtt.
  * Azure storage-fiókkal, amely biztosít a Blob storage, az SQL Server-adatbázisból exportált adatok tárolásához.
  * Egy Azure SQL Data Warehouse-példányhoz.
  * Az Azure Analysis Services-példányt.
  * Az Azure Data Factory és a Data Factory-folyamatot a ELT-feladathoz.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a>Változók

A következő lépések tartalmazzák a felhasználó által definiált változókat. Cserélje le ezeket az Ön által meghatározott értékeket kell.

- `<data_factory_name>`. Adat-előállító nevét.
- `<analysis_server_name>`. Analysis Services-kiszolgáló neve.
- `<active_directory_upn>`. Az Azure Active Directory egyszerű felhasználónév (UPN). Például: `user@contoso.com`.
- `<data_warehouse_server_name>`. Az SQL Data Warehouse-kiszolgáló neve.
- `<data_warehouse_password>`. Az SQL Data Warehouse-rendszergazdai jelszót.
- `<resource_group_name>`. Az erőforráscsoport neve.
- `<region>`. A Azure régióban, ahol az erőforrások üzembe helyezve.
- `<storage_account_name>`. Tárfiók neve. Hajtsa végre a [elnevezési szabályok](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) tárfiókok esetében.
- `<sql-db-password>`. Az SQL Server bejelentkezési jelszót.

### <a name="deploy-azure-data-factory"></a>Az Azure Data Factory üzembe helyezése

1. Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-adattár] [ref-arch-tárház].

2. Futtassa a következő Azure CLI-paranccsal hozzon létre egy erőforráscsoportot.  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    Adja meg, amely támogatja az SQL Data Warehouse, az Azure Analysis Services és a Data Factory v2 egy régiót. Lásd: [az Azure-termékek régiók szerint](https://azure.microsoft.com/global-infrastructure/services/)

3. A következő parancs futtatásával

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

Ezután az Azure Portal használatával a hitelesítési kulcs lekérése az Azure Data Factory [integrációs modul](/azure/data-factory/concepts-integration-runtime), az alábbiak szerint:

1. Az a [az Azure Portal](https://portal.azure.com/), keresse meg a Data Factory-példányt.

2. A Data Factory panelen kattintson a **létrehozás és Monitorozás**. Ekkor megnyílik az Azure Data Factory portálon egy másik böngészőablakban.

    ![](./images/adf-blade.png)

3. Az Azure Data Factory-portálon válassza a ceruza ikonra ("Szerző"). 

4. Kattintson a **kapcsolatok**, majd válassza ki **integrációs modulok**.

5. A **sourceIntegrationRuntime**, kattintson a ceruza ikonra ("Szerkesztés").

    > [!NOTE]
    > A portálon az jelenik meg a "nem" állapot. Ez várható, amíg a helyszíni kiszolgálón telepít.

6. Keresés **Key1** , és másolja a hitelesítési kulcs értékét.

Szüksége lesz a hitelesítési kulcs a következő lépéssel.

### <a name="deploy-the-simulated-on-premises-server"></a>A szimulált helyszíni kiszolgáló üzembe helyezése

Ebben a lépésben egy szimulált helyszíni kiszolgálóként, amely tartalmazza az SQL Server 2017-ben és a kapcsolódó eszközök üzembe helyez egy virtuális Gépet. Emellett betölti a [Wide World Importers OLTP adatbázis] [wwi] SQL-kiszolgálóra.

1. Keresse meg a `data\enterprise_bi_sqldw_advanced\onprem\templates` mappában található az adattárban.

2. Az a `onprem.parameters.json` fájlt, keressen rá a `adminPassword`. Ez az a jelszó az SQL Server VM-ba való bejelentkezéshez. Cserélje le az értéket egy másik jelszót.

3. Ugyanebben a fájlban lévő keresése `SqlUserCredentials`. Ez a tulajdonság határozza meg az SQL Server fiók hitelesítő adatait. Cserélje le a jelszót egy másik értéket.

4. Ugyanebben a fájlban illessze be az Integration Runtime hitelesítési kulcsot a `IntegrationRuntimeGatewayKey` paramétert, a lent látható módon:

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

5. Futtassa a következő parancsot.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

Ebben a lépésben 20 – 30 percet is igénybe vehet. Ez magában foglalja a futó egy [DSC](/powershell/dsc/overview) szkriptet az eszközök telepítésére, és állítsa vissza az adatbázist. 

### <a name="deploy-azure-resources"></a>Az Azure-erőforrások üzembe helyezése

Ebben a lépésben az SQL Data warehouse-ba, az Azure Analysis Services és a Data Factory látja el.

1. Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-adattár] [ref-arch-tárház].

2. Futtassa a következő Azure CLI-parancsot. Cserélje le a csúcsos zárójelben látható paraméter értékét.

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - A `storageAccountName` paraméter kell követnie a [elnevezési szabályok](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) tárfiókok esetében. 
    - Az a `analysisServerAdmin` paramétert, használja az Azure Active Directory egyszerű felhasználónév (UPN).

3. Futtassa a következő Azure CLI-parancsot a tárfiók a tárelérési kulcs lekérésével. Ezt a kulcsot a következő lépésben fogja használni.

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. Futtassa a következő Azure CLI-parancsot. Cserélje le a csúcsos zárójelben látható paraméter értékét. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    A kapcsolati karakterláncok rendelkezik oszt fel kell helyettesíteni csúcsos zárójelben látható. A `<storage_account_key>`, használja az előző lépésben kapott kulcsot. A `<sql-db-password>`, a megadott SQL Server-fiókja jelszavát használja a `onprem.parameters.json` korábban már fájlba.

### <a name="run-the-data-warehouse-scripts"></a>A data warehouse parancsfájlok futtatása

1. Az a [az Azure Portal](https://portal.azure.com/), keresse meg a helyszíni virtuális gép, amelynek neve `sql-vm1`. A felhasználónév és jelszó a virtuális gép meg van adva a `onprem.parameters.json` fájlt.

2. Kattintson a **Connect** és a távoli asztal használata a virtuális Géphez való csatlakozáshoz.

3. A távoli asztali munkamenetből nyisson meg egy parancssort, és keresse meg a virtuális gépen a következő mappában:

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. Futtassa az alábbi parancsot:

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    A `<data_warehouse_server_name>` és `<data_warehouse_password>`, adatraktár-kiszolgáló nevét és a korábbi jelszót használja.

Ebben a lépésben ellenőrzéséhez használhatja az SQL Server Management Studio (SSMS) az SQL Data Warehouse-adatbázishoz való csatlakozáshoz. Az adatbázis táblasémákat kell megjelennie.

### <a name="run-the-data-factory-pipeline"></a>A Data Factory-folyamat futtatása

1. Az azonos távoli asztali munkamenetet nyissa meg egy PowerShell-ablakot.

2. Futtassa az alábbi PowerShell-parancsot. Válasszon **Igen** amikor a rendszer kéri.

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. Futtassa az alábbi PowerShell-parancsot. Adja meg, amikor a rendszer kéri az Azure hitelesítő adatait.

    ```powershell
    Connect-AzureRmAccount 
    ```

4. Futtassa a következő PowerShell-parancsokat. Cserélje le a csúcsos zárójeleket értékeit.

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
    Értékesítési összeg: összeg = ("Tény eladás" [adóval együtt összesen])
    ```

4. Repeat these steps to create the following measures:

    ```
    Évek száma: (MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber])) + 1 =
    
    Nyitó feltöltése: = CALCULATE (SUM ("Tény CityPopulation" [Population]), szűrő ('Tény CityPopulation', 'Tény CityPopulation' [YearNumber] = MIN('Fact CityPopulation'[YearNumber])))
    
    Befejezési népesség: = CALCULATE (SUM ("Tény CityPopulation" [Population]), szűrő ('Tény CityPopulation', "Tény CityPopulation" [YearNumber] = MAX('Fact CityPopulation'[YearNumber])))
    
    CAGR: = IFERROR ((([befejező Population] / [kezdő Population]) ^ (1 / [évek száma])) – 1,0)
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
