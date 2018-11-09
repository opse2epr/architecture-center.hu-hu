---
title: Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban
description: Az Azure-ban egy ELT munkafolyamat automatizálása az Azure Data Factory használatával
author: MikeWasson
ms.date: 11/06/2018
ms.openlocfilehash: 39089d80047b584ac590d285097020212ab72911
ms.sourcegitcommit: 02ecd259a6e780d529c853bc1db320f4fcf919da
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/07/2018
ms.locfileid: "51263729"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban

Ez a referenciaarchitektúra bemutatja, hogyan hajthat végre a növekményes betöltése egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kinyerési, betöltési, átalakítási) folyamatot. Azure Data Factory használatával az ELT folyamatok automatizálásához. A folyamat növekményes helyezi át a legújabb OLTP adatokat a helyszíni SQL Server-adatbázisból az SQL Data Warehouse-bA. Tranzakciós adatok átalakításának egy táblázatos modell elemzés céljából. 

Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![](./images/enterprise-bi-sqldw-adf.png)

Ez az architektúra épít látható [Enterprise BI és az SQL Data Warehouse](./enterprise-bi-sqldw.md), de bizonyos funkciók, fontos vállalati adatraktározási forgatókönyvekben.

-   Automation használatával a Data Factory-folyamatot.
-   Növekményes betöltés.
-   Több adatforrás integrálásával.
-   Bináris adatot, például térinformatikai adatok és lemezképek betöltése.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

### <a name="data-sources"></a>Adatforrások

**A helyszíni SQL Server**. Az adatok helyszíni SQL Server-adatbázis található. A helyszíni környezetben, az architektúra üzembe helyezése az Azure-ban telepített SQL Server virtuális gép üzembe helyezési szkriptjei szimulálásához. A [Wide World Importers OLTP mintaadatbázis] [ wwi] a forrásadatbázis szolgál.

**Külső adatok**. Adattárházak gyakran előfordul, hogy több adatforrás integrálhatja. Ez a referenciaarchitektúra egy külső adatkészlet, amely tartalmazza a város feltöltések évek szerint, és integrálja az OLTP adatbázis adataival tölti be. Használhatja ezeket az adatokat például az elemzésekhez: "Nem minden régióban eladások növekedési nagyobb vagy népességnövekedés?"

### <a name="ingestion-and-data-storage"></a>Adatfeldolgozási és az adatok tárolása

**A BLOB Storage-**. A BLOB storage lesz egy átmeneti területre a forrásadatok mielőtt betöltené azokat az SQL Data Warehouse-bA.

**Azure SQL Data Warehouse**. [Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer analytics végre, a nagy mennyiségű adat. Támogatja a nagy olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas nagy teljesítményű elemzési futtatásához. 

**Az Azure Data Factory**. [A Data Factory] [ adf] egy felügyelt szolgáltatás, amellyel előkészíthető és automatizálható az adatok áthelyezését és átalakítását. Ebben az architektúrában koordinálja a ELT folyamat különböző szakaszaiban.

### <a name="analysis-and-reporting"></a>Elemzés és jelentéskészítés

**Az Azure Analysis Services**. [Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely adatmodellezési képességekkel. A szemantikai modell betöltött Analysis Services.

**Power BI**. A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese. Ebben az architektúrában a szemantikai modell az Analysis Servicesben tárolt kérdezi le.

### <a name="authentication"></a>Hitelesítés

**Az Azure Active Directory** (Azure AD) hitelesíti a felhasználók, akik a Power bi-ban az Analysis Services-kiszolgálóhoz csatlakozhat.

A Data Factory használatával is az Azure AD hitelesítése az SQL Data Warehouse, egy szolgáltatásnév vagy a Felügyeltszolgáltatás-identitás (MSI). Az egyszerűség kedvéért a központi telepítésre példát az SQL Server-hitelesítést használ.

## <a name="data-pipeline"></a>Adatfolyamat

A [Azure Data Factory][adf], egy folyamatot a feladat koordinálja a tevékenységek logikus csoportosításai egy &mdash; ebben az esetben be- és az adatok átalakítása az SQL Data Warehouse-bA. 

Ez a referenciaarchitektúra egy fő folyamatot, amely gyermek folyamatok sorozatát futtatja határozza meg. Minden egyes gyermek folyamat adatokat tölt be egy vagy több adattárház táblái.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>Növekményes betöltés

Automatikus ETL vagy ELT folyamat futtatásakor a leghatékonyabb csak, mivel az előző futtatása az adatok betöltéséhez. Ezt nevezzük egy *növekményes betöltés*, és ne pedig a teljes terhelés, az adatok betöltésekor. Egy növekményes betöltési végrehajtásához kell azonosítani tudja, mely adatok módosultak. A leggyakrabban használt módszer az, hogy használjon egy *magas vízjelbe beleszámított* érték, ami azt jelenti, hogy nyomon követése a legújabb értékeket néhány oszlop a forrástábla, vagy egy dátum-idő oszlop, vagy egy egyedi egész számokat tartalmazó oszlopot. 

Az SQL Server 2016 kezdve használhatja [időbeli verziózású táblák](/sql/relational-databases/tables/temporal-tables). Ezek a rendszerverzióval ellátott táblákon, hogy az adatok teljes előzményeit. Az adatbázismotor automatikusan rögzíti a külön előzménytábla minden módosítási előzményeit. Az előzményadatok lekérdezheti, ha egy FOR SYSTEM_TIME záradékot ad hozzá egy lekérdezést. Belsőleg az adatbázismotor lekérdezi az előzménytáblában, de ez átlátható az alkalmazás. 

> [!NOTE]
> Az SQL Server korábbi verzióit használhatja [adatváltozás-rögzítési](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC). Ez a megközelítés akkor időbeli verziózású táblák,-nál kevesebb kényelmes, mert rendelkezik egy különálló módosítási táblából, és kötetblokkok változásait a napló sorszáma, nem pedig egy időbélyeg. 

Historikus táblák dimenzió az adatokat, amelyek idővel hasznosak. A ténytáblák általában egy nem módosítható tranzakció, például egy rendelést a táblagépükről, ebben az esetben a rendszer verziójának előzményei tartja értelmetlen képviseli. Ehelyett a tranzakciók általában rendelkeznek egy oszlopot, amely a tranzakció dátuma, amelyek használhatók a küszöbértékek jelöli. Ha például a Wide World Importers OLTP-adatbázisban, a Sales.Invoices és Sales.InvoiceLines táblát kell egy `LastEditedWhen` mező, amely alapértelmezés szerint a `sysdatetime()`. 

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

A kihívás abban áll, hogy minden egyes az városa felosztása egy eltérő mennyiségű sor, a földrajzi adatok méretétől függően. A PIVOT operátorban működjön minden az városa ugyanannyi sort kell rendelkeznie. Működnek, a T-SQL-lekérdezés (amelyet meg lehet tekinteni [Itt][MergeLocation]) does néhány trükköket üres értékeket tartalmazó sorok ismételt kitöltésére, hogy minden városhoz azonos számú oszlopot a pivot után. Az eredményül kapott lekérdezés elemről kiderül, hogy sokkal gyorsabb, mint a sorokat egy ismétlése egyszerre kell.

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

A telepítés, és futtassa a referenciaimplementációt, kövesse a lépéseket a [GitHub információs][github]. A következőket helyezi üzembe:

  * Windows virtuális gép egy helyszíni adatbázis-kiszolgáló szimulálásához. Ez magában foglalja az SQL Server 2017-ben és a kapcsolódó eszközök, Power BI Desktop együtt.
  * Azure storage-fiókkal, amely biztosít a Blob storage, az SQL Server-adatbázisból exportált adatok tárolásához.
  * Egy Azure SQL Data Warehouse-példányhoz.
  * Az Azure Analysis Services-példányt.
  * Az Azure Data Factory és a Data Factory-folyamatot a ELT-feladathoz.

[adf]: //azure/data-factory
[github]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database

