---
title: Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban
description: Az Azure-ban egy ELT munkafolyamat automatizálása az Azure Data Factory használatával
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: f004c02da93335e74b07b9720236832ad7f744db
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876902"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a><span data-ttu-id="db5c3-103">Az SQL Data Warehouse és az Azure Data Factory automatizált vállalati bi-ban</span><span class="sxs-lookup"><span data-stu-id="db5c3-103">Automated enterprise BI with SQL Data Warehouse and Azure Data Factory</span></span>

<span data-ttu-id="db5c3-104">Ez a referenciaarchitektúra bemutatja, hogyan hajthat végre a növekményes betöltése egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kinyerési, betöltési, átalakítási) folyamatot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-104">This reference architecture shows how to perform incremental loading in an [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) pipeline.</span></span> <span data-ttu-id="db5c3-105">Azure Data Factory használatával az ELT folyamatok automatizálásához.</span><span class="sxs-lookup"><span data-stu-id="db5c3-105">It uses Azure Data Factory to automate the ELT pipeline.</span></span> <span data-ttu-id="db5c3-106">A folyamat növekményes helyezi át a legújabb OLTP adatokat a helyszíni SQL Server-adatbázisból az SQL Data Warehouse-bA.</span><span class="sxs-lookup"><span data-stu-id="db5c3-106">The pipeline incrementally moves the latest OLTP data from an on-premises SQL Server database into SQL Data Warehouse.</span></span> <span data-ttu-id="db5c3-107">Tranzakciós adatok átalakításának egy táblázatos modell elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="db5c3-107">Transactional data is transformed into a tabular model for analysis.</span></span> [<span data-ttu-id="db5c3-108">**A megoldás üzembe helyezése**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

<span data-ttu-id="db5c3-109">Ez az architektúra épít látható [Enterprise BI és az SQL Data Warehouse](./enterprise-bi-sqldw.md), de bizonyos funkciók, fontos vállalati adatraktározási forgatókönyvekben.</span><span class="sxs-lookup"><span data-stu-id="db5c3-109">This architecture builds on the one shown in [Enterprise BI with SQL Data Warehouse](./enterprise-bi-sqldw.md), but adds some features that are important for enterprise data warehousing scenarios.</span></span>

-   <span data-ttu-id="db5c3-110">Automation használatával a Data Factory-folyamatot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-110">Automation of the pipeline using Data Factory.</span></span>
-   <span data-ttu-id="db5c3-111">Növekményes betöltés.</span><span class="sxs-lookup"><span data-stu-id="db5c3-111">Incremental loading.</span></span>
-   <span data-ttu-id="db5c3-112">Több adatforrás integrálásával.</span><span class="sxs-lookup"><span data-stu-id="db5c3-112">Integrating multiple data sources.</span></span>
-   <span data-ttu-id="db5c3-113">Bináris adatot, például térinformatikai adatok és lemezképek betöltése.</span><span class="sxs-lookup"><span data-stu-id="db5c3-113">Loading binary data such as geospatial data and images.</span></span>

## <a name="architecture"></a><span data-ttu-id="db5c3-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="db5c3-114">Architecture</span></span>

<span data-ttu-id="db5c3-115">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="db5c3-115">The architecture consists of the following components.</span></span>

### <a name="data-sources"></a><span data-ttu-id="db5c3-116">Adatforrások</span><span class="sxs-lookup"><span data-stu-id="db5c3-116">Data sources</span></span>

<span data-ttu-id="db5c3-117">**A helyszíni SQL Server**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-117">**On-premises SQL Server**.</span></span> <span data-ttu-id="db5c3-118">Az adatok helyszíni SQL Server-adatbázis található.</span><span class="sxs-lookup"><span data-stu-id="db5c3-118">The source data is located in a SQL Server database on premises.</span></span> <span data-ttu-id="db5c3-119">A helyszíni környezetben, az architektúra üzembe helyezése az Azure-ban telepített SQL Server virtuális gép üzembe helyezési szkriptjei szimulálásához.</span><span class="sxs-lookup"><span data-stu-id="db5c3-119">To simulate the on-premises environment, the deployment scripts for this architecture provision a virtual machine in Azure with SQL Server installed.</span></span> <span data-ttu-id="db5c3-120">[Wide World Importers OLTP-mintaadatbázist] [wwi] a forrásadatbázis szolgál.</span><span class="sxs-lookup"><span data-stu-id="db5c3-120">The [Wide World Importers OLTP sample database][wwi] is used as the source database.</span></span>

<span data-ttu-id="db5c3-121">**Külső adatok**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-121">**External data**.</span></span> <span data-ttu-id="db5c3-122">Adattárházak gyakran előfordul, hogy több adatforrás integrálhatja.</span><span class="sxs-lookup"><span data-stu-id="db5c3-122">A common scenario for data warehouses is to integrate multiple data sources.</span></span> <span data-ttu-id="db5c3-123">Ez a referenciaarchitektúra egy külső adatkészlet, amely tartalmazza a város feltöltések évek szerint, és integrálja az OLTP adatbázis adataival tölti be.</span><span class="sxs-lookup"><span data-stu-id="db5c3-123">This reference architecture loads an external data set that contains city populations by year, and integrates it with the data from the OLTP database.</span></span> <span data-ttu-id="db5c3-124">Használhatja ezeket az adatokat például az elemzésekhez: "Nem minden régióban eladások növekedési nagyobb vagy népességnövekedés?"</span><span class="sxs-lookup"><span data-stu-id="db5c3-124">You can use this data for insights such as: "Does sales growth in each region match or exceed population growth?"</span></span>

### <a name="ingestion-and-data-storage"></a><span data-ttu-id="db5c3-125">Adatfeldolgozási és az adatok tárolása</span><span class="sxs-lookup"><span data-stu-id="db5c3-125">Ingestion and data storage</span></span>

<span data-ttu-id="db5c3-126">**A BLOB Storage-**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-126">**Blob Storage**.</span></span> <span data-ttu-id="db5c3-127">A BLOB storage lesz egy átmeneti területre a forrásadatok mielőtt betöltené azokat az SQL Data Warehouse-bA.</span><span class="sxs-lookup"><span data-stu-id="db5c3-127">Blob storage is used as a staging area for the source data before loading it into SQL Data Warehouse.</span></span>

<span data-ttu-id="db5c3-128">**Azure SQL Data Warehouse**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-128">**Azure SQL Data Warehouse**.</span></span> <span data-ttu-id="db5c3-129">[Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer analytics végre, a nagy mennyiségű adat.</span><span class="sxs-lookup"><span data-stu-id="db5c3-129">[SQL Data Warehouse](/azure/sql-data-warehouse/) is a distributed system designed to perform analytics on large data.</span></span> <span data-ttu-id="db5c3-130">Támogatja a nagy olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas nagy teljesítményű elemzési futtatásához.</span><span class="sxs-lookup"><span data-stu-id="db5c3-130">It supports massive parallel processing (MPP), which makes it suitable for running high-performance analytics.</span></span> 

<span data-ttu-id="db5c3-131">**Az Azure Data Factory**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-131">**Azure Data Factory**.</span></span> <span data-ttu-id="db5c3-132">[A data Factory] [adf] egy felügyelt szolgáltatás, amellyel előkészíthető és automatizálható az adatok áthelyezését és átalakítását.</span><span class="sxs-lookup"><span data-stu-id="db5c3-132">[Data Factory][adf] is a managed service that orchestrates and automates data movement and data transformation.</span></span> <span data-ttu-id="db5c3-133">Ebben az architektúrában koordinálja a ELT folyamat különböző szakaszaiban.</span><span class="sxs-lookup"><span data-stu-id="db5c3-133">In this architecture, it coordinates the various stages of the ELT process.</span></span>

### <a name="analysis-and-reporting"></a><span data-ttu-id="db5c3-134">Elemzés és jelentéskészítés</span><span class="sxs-lookup"><span data-stu-id="db5c3-134">Analysis and reporting</span></span>

<span data-ttu-id="db5c3-135">**Az Azure Analysis Services**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-135">**Azure Analysis Services**.</span></span> <span data-ttu-id="db5c3-136">[Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely adatmodellezési képességekkel.</span><span class="sxs-lookup"><span data-stu-id="db5c3-136">[Analysis Services](/azure/analysis-services/) is a fully managed service that provides data modeling capabilities.</span></span> <span data-ttu-id="db5c3-137">A szemantikai modell betöltött Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="db5c3-137">The semantic model is loaded into Analysis Services.</span></span>

<span data-ttu-id="db5c3-138">**Power BI**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-138">**Power BI**.</span></span> <span data-ttu-id="db5c3-139">A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese.</span><span class="sxs-lookup"><span data-stu-id="db5c3-139">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="db5c3-140">Ebben az architektúrában a szemantikai modell az Analysis Servicesben tárolt kérdezi le.</span><span class="sxs-lookup"><span data-stu-id="db5c3-140">In this architecture, it queries the semantic model stored in Analysis Services.</span></span>

### <a name="authentication"></a><span data-ttu-id="db5c3-141">Hitelesítés</span><span class="sxs-lookup"><span data-stu-id="db5c3-141">Authentication</span></span>

<span data-ttu-id="db5c3-142">**Az Azure Active Directory** (Azure AD) hitelesíti a felhasználók, akik a Power bi-ban az Analysis Services-kiszolgálóhoz csatlakozhat.</span><span class="sxs-lookup"><span data-stu-id="db5c3-142">**Azure Active Directory** (Azure AD) authenticates users who connect to the Analysis Services server through Power BI.</span></span>

<span data-ttu-id="db5c3-143">A Data Factory használatával is az Azure AD hitelesítése az SQL Data Warehouse, egy szolgáltatásnév vagy a Felügyeltszolgáltatás-identitás (MSI).</span><span class="sxs-lookup"><span data-stu-id="db5c3-143">Data Factory can use also use Azure AD to authenticate to SQL Data Warehouse, by using a service principal or Managed Service Identity (MSI).</span></span> <span data-ttu-id="db5c3-144">Az egyszerűség kedvéért a központi telepítésre példát az SQL Server-hitelesítést használ.</span><span class="sxs-lookup"><span data-stu-id="db5c3-144">For simplicity, the example deployment uses SQL Server authentication.</span></span>

## <a name="data-pipeline"></a><span data-ttu-id="db5c3-145">Adatfolyamat</span><span class="sxs-lookup"><span data-stu-id="db5c3-145">Data pipeline</span></span>

<span data-ttu-id="db5c3-146">Az [Azure Data Factoryban] [adf], egy folyamatot a feladat koordinálja a tevékenységek logikus csoportosításai egy &mdash; ebben az esetben be- és az adatok átalakítása az SQL Data Warehouse-bA.</span><span class="sxs-lookup"><span data-stu-id="db5c3-146">In [Azure Data Factory][adf], a pipeline is a logical grouping of activities used to coordinate a task &mdash; in this case, loading and transforming data into SQL Data Warehouse.</span></span> 

<span data-ttu-id="db5c3-147">Ez a referenciaarchitektúra egy fő folyamatot, amely gyermek folyamatok sorozatát futtatja határozza meg.</span><span class="sxs-lookup"><span data-stu-id="db5c3-147">This reference architecture defines a master pipeline that runs a sequence of child pipelines.</span></span> <span data-ttu-id="db5c3-148">Minden egyes gyermek folyamat adatokat tölt be egy vagy több adattárház táblái.</span><span class="sxs-lookup"><span data-stu-id="db5c3-148">Each child pipeline loads data into one or more data warehouse tables.</span></span>

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a><span data-ttu-id="db5c3-149">Növekményes betöltés</span><span class="sxs-lookup"><span data-stu-id="db5c3-149">Incremental loading</span></span>

<span data-ttu-id="db5c3-150">Automatikus ETL vagy ELT folyamat futtatásakor a leghatékonyabb csak, mivel az előző futtatása az adatok betöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="db5c3-150">When you run an automated ETL or ELT process, it's most efficient to load only the data that changed since the previous run.</span></span> <span data-ttu-id="db5c3-151">Ezt nevezzük egy *növekményes betöltés*, és ne pedig a teljes terhelés, az adatok betöltésekor.</span><span class="sxs-lookup"><span data-stu-id="db5c3-151">This is called an *incremental load*, as opposed to a full load that loads all of the data.</span></span> <span data-ttu-id="db5c3-152">Egy növekményes betöltési végrehajtásához kell azonosítani tudja, mely adatok módosultak.</span><span class="sxs-lookup"><span data-stu-id="db5c3-152">To perform an incremental load, you need a way to identify which data has changed.</span></span> <span data-ttu-id="db5c3-153">A leggyakrabban használt módszer az, hogy használjon egy *magas vízjelbe beleszámított* érték, ami azt jelenti, hogy nyomon követése a legújabb értékeket néhány oszlop a forrástábla, vagy egy dátum-idő oszlop, vagy egy egyedi egész számokat tartalmazó oszlopot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-153">The most common approach is to use a *high water mark* value, which means tracking the latest value of some column in the source table, either a datetime column or a unique integer column.</span></span> 

<span data-ttu-id="db5c3-154">Az SQL Server 2016 kezdve használhatja [időbeli verziózású táblák](/sql/relational-databases/tables/temporal-tables).</span><span class="sxs-lookup"><span data-stu-id="db5c3-154">Starting with SQL Server 2016, you can use [temporal tables](/sql/relational-databases/tables/temporal-tables).</span></span> <span data-ttu-id="db5c3-155">Ezek a rendszerverzióval ellátott táblákon, hogy az adatok teljes előzményeit.</span><span class="sxs-lookup"><span data-stu-id="db5c3-155">These are system-versioned tables that keep a full history of data changes.</span></span> <span data-ttu-id="db5c3-156">Az adatbázismotor automatikusan rögzíti a külön előzménytábla minden módosítási előzményeit.</span><span class="sxs-lookup"><span data-stu-id="db5c3-156">The database engine automatically records the history of every change in a separate history table.</span></span> <span data-ttu-id="db5c3-157">Az előzményadatok lekérdezheti, ha egy FOR SYSTEM_TIME záradékot ad hozzá egy lekérdezést.</span><span class="sxs-lookup"><span data-stu-id="db5c3-157">You can query the historical data by adding a FOR SYSTEM_TIME clause to a query.</span></span> <span data-ttu-id="db5c3-158">Belsőleg az adatbázismotor lekérdezi az előzménytáblában, de ez átlátható az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="db5c3-158">Internally, the database engine queries the history table, but this is transparent to the application.</span></span> 

> [!NOTE]
> <span data-ttu-id="db5c3-159">Az SQL Server korábbi verzióit használhatja [adatváltozás-rögzítési](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span><span class="sxs-lookup"><span data-stu-id="db5c3-159">For earlier versions of SQL Server, you can use [Change Data Capture](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span></span> <span data-ttu-id="db5c3-160">Ez a megközelítés akkor időbeli verziózású táblák,-nál kevesebb kényelmes, mert rendelkezik egy különálló módosítási táblából, és kötetblokkok változásait a napló sorszáma, nem pedig egy időbélyeg.</span><span class="sxs-lookup"><span data-stu-id="db5c3-160">This approach is less convenient than temporal tables, because you have to query a separate change table, and changes are tracked by a log sequence number, rather than a timestamp.</span></span> 

<span data-ttu-id="db5c3-161">Historikus táblák dimenzió az adatokat, amelyek idővel hasznosak.</span><span class="sxs-lookup"><span data-stu-id="db5c3-161">Temporal tables are useful for dimension data, which can change over time.</span></span> <span data-ttu-id="db5c3-162">A ténytáblák általában egy nem módosítható tranzakció, például egy rendelést a táblagépükről, ebben az esetben a rendszer verziójának előzményei tartja értelmetlen képviseli.</span><span class="sxs-lookup"><span data-stu-id="db5c3-162">Fact tables usually represent an immutable transaction such as a sale, in which case keeping the system version history doesn't make sense.</span></span> <span data-ttu-id="db5c3-163">Ehelyett a tranzakciók általában rendelkeznek egy oszlopot, amely a tranzakció dátuma, amelyek használhatók a küszöbértékek jelöli.</span><span class="sxs-lookup"><span data-stu-id="db5c3-163">Instead, transactions usually have a column that represents the transaction date, which can be used as the watermark value.</span></span> <span data-ttu-id="db5c3-164">Ha például a Wide World Importers OLTP-adatbázisban, a Sales.Invoices és Sales.InvoiceLines táblát kell egy `LastEditedWhen` mező, amely alapértelmezés szerint a `sysdatetime()`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-164">For example, in the Wide World Importers OLTP database, the Sales.Invoices and Sales.InvoiceLines tables have a `LastEditedWhen` field that defaults to `sysdatetime()`.</span></span> 

<span data-ttu-id="db5c3-165">Itt látható az általános folyamat az ELT folyamatok:</span><span class="sxs-lookup"><span data-stu-id="db5c3-165">Here is the general flow for the ELT pipeline:</span></span>

1. <span data-ttu-id="db5c3-166">A forrásadatbázis minden táblához nyomon követheti a megszakítási idő, amikor az utolsó ELT feladat futott.</span><span class="sxs-lookup"><span data-stu-id="db5c3-166">For each table in the source database, track the cutoff time when the last ELT job ran.</span></span> <span data-ttu-id="db5c3-167">Ez az információ Store az adatraktárban.</span><span class="sxs-lookup"><span data-stu-id="db5c3-167">Store this information in the data warehouse.</span></span> <span data-ttu-id="db5c3-168">(A kezdeti beállítás, minden esetben vannak beállítva: 1-1-1900-hoz ".)</span><span class="sxs-lookup"><span data-stu-id="db5c3-168">(On initial setup, all times are set to '1-1-1900'.)</span></span>

2. <span data-ttu-id="db5c3-169">Során az adatok exportálása a lépést, a megszakítási idő tárolt eljárások a forrásadatbázis átadott paraméterként.</span><span class="sxs-lookup"><span data-stu-id="db5c3-169">During the data export step, the cutoff time is passed as a parameter to a set of stored procedures in the source database.</span></span> <span data-ttu-id="db5c3-170">Ezek tárolt eljárások lekérdezést, amelyet a megszakítási idő után létrehozott vagy megváltozott rekordokat.</span><span class="sxs-lookup"><span data-stu-id="db5c3-170">These stored procedures query for any records that were changed or created after the cutoff time.</span></span> <span data-ttu-id="db5c3-171">A Sales (tény) tábla a `LastEditedWhen` oszlopot használja.</span><span class="sxs-lookup"><span data-stu-id="db5c3-171">For the Sales fact table, the `LastEditedWhen` column is used.</span></span> <span data-ttu-id="db5c3-172">A dimenzió adatok rendszerverzióval ellátott historikus táblákon szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="db5c3-172">For the dimension data, system-versioned temporal tables are used.</span></span>

3. <span data-ttu-id="db5c3-173">Ha az adatok migrálása befejeződött, frissítse a tábla tárolja a megszakítási idő.</span><span class="sxs-lookup"><span data-stu-id="db5c3-173">When the data migration is complete, update the table that stores the cutoff times.</span></span>

<span data-ttu-id="db5c3-174">Emellett akkor is hasznos, jegyezze fel a *leszármaztatási* esetében minden egyes Futtatás ELT.</span><span class="sxs-lookup"><span data-stu-id="db5c3-174">It's also useful to record a *lineage* for each ELT run.</span></span> <span data-ttu-id="db5c3-175">Egy adott rekord a leszármaztatási társítja, amely az adatok előállított rekordot a ELT, futtassa a.</span><span class="sxs-lookup"><span data-stu-id="db5c3-175">For a given record, the lineage associates that record with the ELT run that produced the data.</span></span> <span data-ttu-id="db5c3-176">Az egyes ETL futtatások mindegyik táblához, megjeleníti a kezdési és befejezési lapbetöltési idők leszármaztatási új rekord jön létre.</span><span class="sxs-lookup"><span data-stu-id="db5c3-176">For each ETL run, a new lineage record is created for every table, showing the starting and ending load times.</span></span> <span data-ttu-id="db5c3-177">A leszármaztatási kulcsok minden egyes rekord a dimenzió és a ténytáblákat táblákban tárolódnak.</span><span class="sxs-lookup"><span data-stu-id="db5c3-177">The lineage keys for each record are stored in the dimension and fact tables.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="db5c3-178">Után az adatok egy új batch tölti be az adatraktár, frissítse az Analysis Services táblázatos modellt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-178">After a new batch of data is loaded into the warehouse, refresh the Analysis Services tabular model.</span></span> <span data-ttu-id="db5c3-179">Lásd: [aszinkron frissítése a REST API-val](/azure/analysis-services/analysis-services-async-refresh).</span><span class="sxs-lookup"><span data-stu-id="db5c3-179">See [Asynchronous refresh with the REST API](/azure/analysis-services/analysis-services-async-refresh).</span></span>

## <a name="data-cleansing"></a><span data-ttu-id="db5c3-180">Adattisztító</span><span class="sxs-lookup"><span data-stu-id="db5c3-180">Data cleansing</span></span>

<span data-ttu-id="db5c3-181">Adattisztítás ELT folyamatának részeként kell lennie.</span><span class="sxs-lookup"><span data-stu-id="db5c3-181">Data cleansing should be part of the ELT process.</span></span> <span data-ttu-id="db5c3-182">Ez a referenciaarchitektúra egy hibás adatforrás a város population táblában, ahol egyes városoknak rendelkezik nulla population például mert nincs adat nem volt elérhető.</span><span class="sxs-lookup"><span data-stu-id="db5c3-182">In this reference architecture, one source of bad data is the city population table, where some cities have zero population, perhaps because no data was available.</span></span> <span data-ttu-id="db5c3-183">A feldolgozás során az ELT folyamatok eltávolítja a kiválasztott városok az városa population tábla.</span><span class="sxs-lookup"><span data-stu-id="db5c3-183">During processing, the ELT pipeline removes those cities from the city population table.</span></span> <span data-ttu-id="db5c3-184">Hajtsa végre az előkészítési táblák ahelyett, hogy a külső táblák adattisztító.</span><span class="sxs-lookup"><span data-stu-id="db5c3-184">Perform data cleansing on staging tables, rather than external tables.</span></span>

<span data-ttu-id="db5c3-185">Íme a tárolt eljárást, amely a várost, a nulla population eltávolítja az városa Population táblából.</span><span class="sxs-lookup"><span data-stu-id="db5c3-185">Here is the stored procedure that removes the cities with zero population from the City Population table.</span></span> <span data-ttu-id="db5c3-186">(A forrásfájl annak [Itt](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span><span class="sxs-lookup"><span data-stu-id="db5c3-186">(You can find the source file [here](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span></span> 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a><span data-ttu-id="db5c3-187">Külső adatforrások</span><span class="sxs-lookup"><span data-stu-id="db5c3-187">External data sources</span></span>

<span data-ttu-id="db5c3-188">Adattárházak gyakran több forrásból származó adatok egyesíthetők.</span><span class="sxs-lookup"><span data-stu-id="db5c3-188">Data warehouses often consolidate data from multiple sources.</span></span> <span data-ttu-id="db5c3-189">Ez a referenciaarchitektúra egy külső adatforrás, amely tartalmazza a demográfiai adatokat tölt be.</span><span class="sxs-lookup"><span data-stu-id="db5c3-189">This reference architecture loads an external data source that contains demographics data.</span></span> <span data-ttu-id="db5c3-190">Ez az adatkészlet érhető el az Azure blob storage-ban részeként a [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) minta.</span><span class="sxs-lookup"><span data-stu-id="db5c3-190">This dataset is available in Azure blob storage as part of the [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) sample.</span></span>

<span data-ttu-id="db5c3-191">Az Azure Data Factory másolhatja, közvetlenül a blob storage használatával a [blob storage-összekötő](/azure/data-factory/connector-azure-blob-storage).</span><span class="sxs-lookup"><span data-stu-id="db5c3-191">Azure Data Factory can copy directly from blob storage, using the [blob storage connector](/azure/data-factory/connector-azure-blob-storage).</span></span> <span data-ttu-id="db5c3-192">Azonban az összekötő számára szükséges kapcsolati karakterlánc vagy közös hozzáférésű jogosultságkódot, így nem használható nyilvános olvasási hozzáférés a blob másolásához.</span><span class="sxs-lookup"><span data-stu-id="db5c3-192">However, the connector requires a connection string or a shared access signature, so it can't be used to copy a blob with public read access.</span></span> <span data-ttu-id="db5c3-193">Áthidaló megoldásként használhatja a PolyBase külső tábla létrehozása tárt a Blob storage, és másolja a külső táblák az SQL Data Warehouse-bA.</span><span class="sxs-lookup"><span data-stu-id="db5c3-193">As a workaround, you can use PolyBase to create an external table over Blob storage and then copy the external tables into SQL Data Warehouse.</span></span> 

## <a name="handling-large-binary-data"></a><span data-ttu-id="db5c3-194">Nagy méretű bináris adatok kezelése</span><span class="sxs-lookup"><span data-stu-id="db5c3-194">Handling large binary data</span></span> 

<span data-ttu-id="db5c3-195">A forrás-adatbázisban, a városok táblának van egy hely oszlopot tartalmazó egy [földrajzi](/sql/t-sql/spatial-geography/spatial-types-geography) térbeli adatok típusa.</span><span class="sxs-lookup"><span data-stu-id="db5c3-195">In the source database, the Cities table has a Location column that holds a [geography](/sql/t-sql/spatial-geography/spatial-types-geography) spatial data type.</span></span> <span data-ttu-id="db5c3-196">Az SQL Data Warehouse nem támogatja a **földrajzi** írja be a natív módon, így ez a mező alakítja át egy **varbinary** típus betöltése során.</span><span class="sxs-lookup"><span data-stu-id="db5c3-196">SQL Data Warehouse doesn't support the **geography** type natively, so this field is converted to a **varbinary** type during loading.</span></span> <span data-ttu-id="db5c3-197">(Lásd: [nem támogatott adattípusok megoldásai](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span><span class="sxs-lookup"><span data-stu-id="db5c3-197">(See [Workarounds for unsupported data types](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span></span>

<span data-ttu-id="db5c3-198">Azonban a PolyBase támogatja egy oszlop maximális mérete `varbinary(8000)`, ami azt jelenti, hogy néhány adat csonkolódik.</span><span class="sxs-lookup"><span data-stu-id="db5c3-198">However, PolyBase supports a maximum column size of `varbinary(8000)`, which means some data could be truncated.</span></span> <span data-ttu-id="db5c3-199">A probléma áthidaló felosztása a adatokat adattömbökre exportálás során, és ezután szétbontani módon az adattömböket:</span><span class="sxs-lookup"><span data-stu-id="db5c3-199">A workaround for this problem is to break the data up into chunks during export, and then reassemble the chunks, as follows:</span></span>

1. <span data-ttu-id="db5c3-200">Hozzon létre egy átmeneti előkészítési táblába, az a hely oszlopban.</span><span class="sxs-lookup"><span data-stu-id="db5c3-200">Create a temporary staging table for the Location column.</span></span>

2. <span data-ttu-id="db5c3-201">Mindegyik városhoz ossza fel a helyadatok 8000 bájtos adattömböket, ami 1 &ndash; mindegyik városhoz N sorát.</span><span class="sxs-lookup"><span data-stu-id="db5c3-201">For each city, split the location data into 8000-byte chunks, resulting in 1 &ndash; N rows for each city.</span></span>

3. <span data-ttu-id="db5c3-202">Az adattömbök szétbontani, használja a T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operátor sorok átalakítása oszlopokat, és majd összefűzi a minden Város oszlop értékeit.</span><span class="sxs-lookup"><span data-stu-id="db5c3-202">To reassemble the chunks, use the T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operator to convert rows into columns and then concatenate the column values for each city.</span></span>

<span data-ttu-id="db5c3-203">A kihívás abban áll, hogy minden egyes az városa felosztása egy eltérő mennyiségű sor, a földrajzi adatok méretétől függően.</span><span class="sxs-lookup"><span data-stu-id="db5c3-203">The challenge is that each city will be split into a different number of rows, depending on the size of geography data.</span></span> <span data-ttu-id="db5c3-204">A PIVOT operátorban működjön minden az városa ugyanannyi sort kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="db5c3-204">For the PIVOT operator to work, every city must have the same number of rows.</span></span> <span data-ttu-id="db5c3-205">Működnek, a T-SQL-lekérdezés (amelyet meg lehet tekinteni [here][MergeLocation]) hajtja végre bizonyos trükköket üres értékeket tartalmazó sorok ismételt kitöltésére, hogy minden városhoz azonos számú oszlopot a pivot után.</span><span class="sxs-lookup"><span data-stu-id="db5c3-205">To make this work, the T-SQL query (which you can view [here][MergeLocation]) does some tricks to pad out the rows with blank values, so that every city has the same number of columns after the pivot.</span></span> <span data-ttu-id="db5c3-206">Az eredményül kapott lekérdezés elemről kiderül, hogy sokkal gyorsabb, mint a sorokat egy ismétlése egyszerre kell.</span><span class="sxs-lookup"><span data-stu-id="db5c3-206">The resulting query turns out to be much faster than looping through the rows one at a time.</span></span>

<span data-ttu-id="db5c3-207">Ugyanezzel a módszerrel képadatok szolgál.</span><span class="sxs-lookup"><span data-stu-id="db5c3-207">The same approach is used for image data.</span></span>

## <a name="slowly-changing-dimensions"></a><span data-ttu-id="db5c3-208">Lassan változó dimenzió</span><span class="sxs-lookup"><span data-stu-id="db5c3-208">Slowly changing dimensions</span></span>

<span data-ttu-id="db5c3-209">Dimenzió adatok viszonylag statikusak, de módosíthatja azt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-209">Dimension data is relatively static, but it can change.</span></span> <span data-ttu-id="db5c3-210">Termék például egy másik termékkel kategóriához első hozzárendelni.</span><span class="sxs-lookup"><span data-stu-id="db5c3-210">For example, a product might get reassigned to a different product category.</span></span> <span data-ttu-id="db5c3-211">Nincsenek a lassan változó dimenzió kezelése több megközelítés közül.</span><span class="sxs-lookup"><span data-stu-id="db5c3-211">There are several approaches to handling slowly changing dimensions.</span></span> <span data-ttu-id="db5c3-212">Egy közös leképezésnek hívott technika [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), adjon hozzá egy új rekordot, amikor egy dimenzió módosítások.</span><span class="sxs-lookup"><span data-stu-id="db5c3-212">A common technique, called [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), is to add a new record whenever a dimension changes.</span></span> 

<span data-ttu-id="db5c3-213">A Type 2 megközelítés megvalósításához dimenziótáblák kell további oszlopokat, amelyek egy adott rekord érvényes dátumtartományt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-213">In order to implement the Type 2 approach, dimension tables need additional columns that specify the effective date range for a given record.</span></span> <span data-ttu-id="db5c3-214">Is a forrásadatbázisból elsődleges kulcsok többszörözni, így a táblát egy mesterséges elsődleges kulccsal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="db5c3-214">Also, primary keys from the source database will be duplicated, so the dimension table must have an artificial primary key.</span></span>

<span data-ttu-id="db5c3-215">Az alábbi képen látható a Dimension.City tábla.</span><span class="sxs-lookup"><span data-stu-id="db5c3-215">The following image shows the Dimension.City table.</span></span> <span data-ttu-id="db5c3-216">A `WWI City ID` oszlop az elsődleges kulcsot a forrásadatbázisból.</span><span class="sxs-lookup"><span data-stu-id="db5c3-216">The `WWI City ID` column is the primary key from the source database.</span></span> <span data-ttu-id="db5c3-217">A `City Key` oszlop egy az ETL-folyamat során létrehozott mesterséges kulcsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-217">The `City Key` column is an artificial key generated during the ETL pipeline.</span></span> <span data-ttu-id="db5c3-218">Figyelje meg, hogy a tábla rendelkezik `Valid From` és `Valid To` oszlopot, így tartományának megadása, ha minden sor volt érvényes.</span><span class="sxs-lookup"><span data-stu-id="db5c3-218">Also notice that the table has `Valid From` and `Valid To` columns, which define the range when each row was valid.</span></span> <span data-ttu-id="db5c3-219">Aktuális értékek rendelkezik egy `Valid To` egyenlő "9999-12-31'.</span><span class="sxs-lookup"><span data-stu-id="db5c3-219">Current values have a `Valid To` equal to '9999-12-31'.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="db5c3-220">Ez a megközelítés az az előnye, megőrzi az előzményadatok, amely elemzéshez hasznos lehet.</span><span class="sxs-lookup"><span data-stu-id="db5c3-220">The advantage of this approach is that it preserves historical data, which can be valuable for analysis.</span></span> <span data-ttu-id="db5c3-221">Azonban azt is jelenti az ugyanazon entitás több sor lesz.</span><span class="sxs-lookup"><span data-stu-id="db5c3-221">However, it also means there will be multiple rows for the same entity.</span></span> <span data-ttu-id="db5c3-222">Például az alábbiakban a megfelelő rekordok `WWI City ID` = 28561:</span><span class="sxs-lookup"><span data-stu-id="db5c3-222">For example, here are the records that match `WWI City ID` = 28561:</span></span>

![](./images/city-dimension-table-2.png)

<span data-ttu-id="db5c3-223">Az egyes értékesítési egyedkapcsolat szeretné a tény társítása az városa dimenziótábla, egyetlen sor invoice Date megfelelő.</span><span class="sxs-lookup"><span data-stu-id="db5c3-223">For each Sales fact, you want to associate that fact with a single row in City dimension table, corresponding to the invoice date.</span></span> <span data-ttu-id="db5c3-224">Az ETL-folyamat részeként egy további oszlop létrehozása, amely</span><span class="sxs-lookup"><span data-stu-id="db5c3-224">As part of the ETL process, create an additional column that</span></span> 

<span data-ttu-id="db5c3-225">A következő T-SQL-lekérdezést hoz létre egy ideiglenes táblát, amely összekapcsolja minden számlán a megfelelő várost kulcsot az városa dimenzió táblából.</span><span class="sxs-lookup"><span data-stu-id="db5c3-225">The following T-SQL query creates a temporary table that associates each invoice with the correct City Key from the City dimension table.</span></span>

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

<span data-ttu-id="db5c3-226">Ez a táblázat egy oszlopot a értékesítés ténytáblában feltöltésére szolgál:</span><span class="sxs-lookup"><span data-stu-id="db5c3-226">This table is used to populate a column in the Sales fact table:</span></span>

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

<span data-ttu-id="db5c3-227">Ez az oszlop lehetővé teszi, hogy a Power BI lekérdezés keresse meg a megfelelő várost rekordot egy megadott értékesítési számla.</span><span class="sxs-lookup"><span data-stu-id="db5c3-227">This column enables a Power BI query to find the correct City record for a given sales invoice.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="db5c3-228">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="db5c3-228">Security considerations</span></span>

<span data-ttu-id="db5c3-229">A fokozott biztonság érdekében használhat [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) biztonságossá tétele Azure-szolgáltatási erőforrások a virtuális hálózaton.</span><span class="sxs-lookup"><span data-stu-id="db5c3-229">For additional security, you can use [Virtual Network service endpoints](/azure/virtual-network/virtual-network-service-endpoints-overview) to secure Azure service resources to only your virtual network.</span></span> <span data-ttu-id="db5c3-230">Ezzel eltávolítja ezeket az erőforrásokat, így csak a virtuális hálózatból érkező forgalom nyilvános internetkapcsolaton keresztüli hozzáférés teljes mértékben.</span><span class="sxs-lookup"><span data-stu-id="db5c3-230">This fully removes public Internet access to those resources, allowing traffic only from your virtual network.</span></span>

<span data-ttu-id="db5c3-231">Ezzel a módszerrel hozhat létre egy Vnetet az Azure-ban, és hozzon létre saját Szolgáltatásvégpontok az Azure-szolgáltatásokhoz.</span><span class="sxs-lookup"><span data-stu-id="db5c3-231">With this approach, you create a VNet in Azure and then create private service endpoints for Azure services.</span></span> <span data-ttu-id="db5c3-232">Ezeket a szolgáltatásokat majd korlátozva vannak a forgalom a virtuális hálózatról.</span><span class="sxs-lookup"><span data-stu-id="db5c3-232">Those services are then restricted to traffic from that virtual network.</span></span> <span data-ttu-id="db5c3-233">Is elérheti azokat a helyszíni hálózatból egy átjárón keresztül.</span><span class="sxs-lookup"><span data-stu-id="db5c3-233">You can also reach them from your on-premises network through a gateway.</span></span>

<span data-ttu-id="db5c3-234">Vegye figyelembe a következő korlátozások vonatkoznak:</span><span class="sxs-lookup"><span data-stu-id="db5c3-234">Be aware of the following limitations:</span></span>

- <span data-ttu-id="db5c3-235">A időben Ez a referenciaarchitektúra lett létrehozva, a virtuális hálózati Szolgáltatásvégpontok Azure Storage és Azure SQL Data warehouse-ba, de az Azure Analysis Service a nem támogatott.</span><span class="sxs-lookup"><span data-stu-id="db5c3-235">At the time this reference architecture was created, VNet service endpoints are supported for Azure Storage and Azure SQL Data Warehouse, but not for Azure Analysis Service.</span></span> <span data-ttu-id="db5c3-236">A legfrissebb állapotának [Itt](https://azure.microsoft.com/updates/?product=virtual-network).</span><span class="sxs-lookup"><span data-stu-id="db5c3-236">Check the latest status [here](https://azure.microsoft.com/updates/?product=virtual-network).</span></span> 

- <span data-ttu-id="db5c3-237">Ha a Szolgáltatásvégpontok engedélyezve vannak az Azure Storage, a PolyBase nem adatmásolás Storage-ból az SQL Data Warehouse-bA.</span><span class="sxs-lookup"><span data-stu-id="db5c3-237">If service endpoints are enabled for Azure Storage, PolyBase cannot copy data from Storage into SQL Data Warehouse.</span></span> <span data-ttu-id="db5c3-238">Nincs egy megoldás erre a problémára.</span><span class="sxs-lookup"><span data-stu-id="db5c3-238">There is a mitigation for this issue.</span></span> <span data-ttu-id="db5c3-239">További információkért lásd: [hatását a virtuális hálózati Szolgáltatásvégpontok használatával és az Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span><span class="sxs-lookup"><span data-stu-id="db5c3-239">For more information, see [Impact of using VNet Service Endpoints with Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span></span> 

- <span data-ttu-id="db5c3-240">Adatok áthelyezése a helyszínről az Azure Storage-ba, szüksége lesz a nyilvános IP-címeket a helyszíni vagy ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="db5c3-240">To move data from on-premises into Azure Storage, you will need to whitelist public IP addresses from your on-premises or ExpressRoute.</span></span> <span data-ttu-id="db5c3-241">További információkért lásd: [virtuális hálózatok biztonságossá tétele Azure-szolgáltatások](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span><span class="sxs-lookup"><span data-stu-id="db5c3-241">For details, see [Securing Azure services to virtual networks](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span></span>

- <span data-ttu-id="db5c3-242">Ahhoz, hogy az Analysis Services adatokat olvasni az SQL Data Warehouse, Windows virtuális gép üzembe helyezése a virtuális hálózathoz, amely tartalmazza az SQL Data Warehouse szolgáltatásvégpontot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-242">To enable Analysis Services to read data from SQL Data Warehouse, deploy a Windows VM to the virtual network that contains the SQL Data Warehouse service endpoint.</span></span> <span data-ttu-id="db5c3-243">Telepítés [Azure a helyszíni adatátjáró](/azure/analysis-services/analysis-services-gateway) a virtuális gépen.</span><span class="sxs-lookup"><span data-stu-id="db5c3-243">Install [Azure On-premises Data Gateway](/azure/analysis-services/analysis-services-gateway) on this VM.</span></span> <span data-ttu-id="db5c3-244">Kapcsolódjon az Azure elemzési szolgáltatás az átjárót.</span><span class="sxs-lookup"><span data-stu-id="db5c3-244">Then connect your Azure Analysis service to the data gateway.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="db5c3-245">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="db5c3-245">Deploy the solution</span></span>

<span data-ttu-id="db5c3-246">Ez a referenciaarchitektúra egy üzemelő példánya [GitHub] [ref-arch-adattár-mappa] érhető el.</span><span class="sxs-lookup"><span data-stu-id="db5c3-246">A deployment for this reference architecture is available on [GitHub][ref-arch-repo-folder].</span></span> <span data-ttu-id="db5c3-247">A következőket helyezi üzembe:</span><span class="sxs-lookup"><span data-stu-id="db5c3-247">It deploys the following:</span></span>

  * <span data-ttu-id="db5c3-248">Windows virtuális gép egy helyszíni adatbázis-kiszolgáló szimulálásához.</span><span class="sxs-lookup"><span data-stu-id="db5c3-248">A Windows VM to simulate an on-premises database server.</span></span> <span data-ttu-id="db5c3-249">Ez magában foglalja az SQL Server 2017-ben és a kapcsolódó eszközök, Power BI Desktop együtt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-249">It includes SQL Server 2017 and related tools, along with Power BI Desktop.</span></span>
  * <span data-ttu-id="db5c3-250">Azure storage-fiókkal, amely biztosít a Blob storage, az SQL Server-adatbázisból exportált adatok tárolásához.</span><span class="sxs-lookup"><span data-stu-id="db5c3-250">An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.</span></span>
  * <span data-ttu-id="db5c3-251">Egy Azure SQL Data Warehouse-példányhoz.</span><span class="sxs-lookup"><span data-stu-id="db5c3-251">An Azure SQL Data Warehouse instance.</span></span>
  * <span data-ttu-id="db5c3-252">Az Azure Analysis Services-példányt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-252">An Azure Analysis Services instance.</span></span>
  * <span data-ttu-id="db5c3-253">Az Azure Data Factory és a Data Factory-folyamatot a ELT-feladathoz.</span><span class="sxs-lookup"><span data-stu-id="db5c3-253">Azure Data Factory and the Data Factory pipeline for the ELT job.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="db5c3-254">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="db5c3-254">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a><span data-ttu-id="db5c3-255">Változók</span><span class="sxs-lookup"><span data-stu-id="db5c3-255">Variables</span></span>

<span data-ttu-id="db5c3-256">A következő lépések tartalmazzák a felhasználó által definiált változókat.</span><span class="sxs-lookup"><span data-stu-id="db5c3-256">The steps that follow include some user-defined variables.</span></span> <span data-ttu-id="db5c3-257">Cserélje le ezeket az Ön által meghatározott értékeket kell.</span><span class="sxs-lookup"><span data-stu-id="db5c3-257">You will need to replace these with values that you define.</span></span>

- <span data-ttu-id="db5c3-258">`<data_factory_name>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-258">`<data_factory_name>`.</span></span> <span data-ttu-id="db5c3-259">Adat-előállító nevét.</span><span class="sxs-lookup"><span data-stu-id="db5c3-259">Data Factory name.</span></span>
- <span data-ttu-id="db5c3-260">`<analysis_server_name>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-260">`<analysis_server_name>`.</span></span> <span data-ttu-id="db5c3-261">Analysis Services-kiszolgáló neve.</span><span class="sxs-lookup"><span data-stu-id="db5c3-261">Analysis Services server name.</span></span>
- <span data-ttu-id="db5c3-262">`<active_directory_upn>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-262">`<active_directory_upn>`.</span></span> <span data-ttu-id="db5c3-263">Az Azure Active Directory egyszerű felhasználónév (UPN).</span><span class="sxs-lookup"><span data-stu-id="db5c3-263">Your Azure Active Directory user principal name (UPN).</span></span> <span data-ttu-id="db5c3-264">Például: `user@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-264">For example, `user@contoso.com`.</span></span>
- <span data-ttu-id="db5c3-265">`<data_warehouse_server_name>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-265">`<data_warehouse_server_name>`.</span></span> <span data-ttu-id="db5c3-266">Az SQL Data Warehouse-kiszolgáló neve.</span><span class="sxs-lookup"><span data-stu-id="db5c3-266">SQL Data Warehouse server name.</span></span>
- <span data-ttu-id="db5c3-267">`<data_warehouse_password>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-267">`<data_warehouse_password>`.</span></span> <span data-ttu-id="db5c3-268">Az SQL Data Warehouse-rendszergazdai jelszót.</span><span class="sxs-lookup"><span data-stu-id="db5c3-268">SQL Data Warehouse administrator password.</span></span>
- <span data-ttu-id="db5c3-269">`<resource_group_name>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-269">`<resource_group_name>`.</span></span> <span data-ttu-id="db5c3-270">Az erőforráscsoport neve.</span><span class="sxs-lookup"><span data-stu-id="db5c3-270">The name of the resource group.</span></span>
- <span data-ttu-id="db5c3-271">`<region>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-271">`<region>`.</span></span> <span data-ttu-id="db5c3-272">A Azure régióban, ahol az erőforrások üzembe helyezve.</span><span class="sxs-lookup"><span data-stu-id="db5c3-272">The Azure region where the resources will be deployed.</span></span>
- <span data-ttu-id="db5c3-273">`<storage_account_name>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-273">`<storage_account_name>`.</span></span> <span data-ttu-id="db5c3-274">Tárfiók neve.</span><span class="sxs-lookup"><span data-stu-id="db5c3-274">Storage account name.</span></span> <span data-ttu-id="db5c3-275">Hajtsa végre a [elnevezési szabályok](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) tárfiókok esetében.</span><span class="sxs-lookup"><span data-stu-id="db5c3-275">Must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span>
- <span data-ttu-id="db5c3-276">`<sql-db-password>`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-276">`<sql-db-password>`.</span></span> <span data-ttu-id="db5c3-277">Az SQL Server bejelentkezési jelszót.</span><span class="sxs-lookup"><span data-stu-id="db5c3-277">SQL Server login password.</span></span>

### <a name="deploy-azure-data-factory"></a><span data-ttu-id="db5c3-278">Az Azure Data Factory üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="db5c3-278">Deploy Azure Data Factory</span></span>

1. <span data-ttu-id="db5c3-279">Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-adattár] [ref-arch-tárház].</span><span class="sxs-lookup"><span data-stu-id="db5c3-279">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="db5c3-280">Futtassa a következő Azure CLI-paranccsal hozzon létre egy erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-280">Run the following Azure CLI command to create a resource group.</span></span>  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    <span data-ttu-id="db5c3-281">Adja meg, amely támogatja az SQL Data Warehouse, az Azure Analysis Services és a Data Factory v2 egy régiót.</span><span class="sxs-lookup"><span data-stu-id="db5c3-281">Specify a region that supports SQL Data Warehouse, Azure Analysis Services, and Data Factory v2.</span></span> <span data-ttu-id="db5c3-282">Lásd: [az Azure-termékek régiók szerint](https://azure.microsoft.com/global-infrastructure/services/)</span><span class="sxs-lookup"><span data-stu-id="db5c3-282">See [Azure Products by Region](https://azure.microsoft.com/global-infrastructure/services/)</span></span>

3. <span data-ttu-id="db5c3-283">A következő parancs futtatásával</span><span class="sxs-lookup"><span data-stu-id="db5c3-283">Run the following command</span></span>

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

<span data-ttu-id="db5c3-284">Ezután az Azure Portal használatával a hitelesítési kulcs lekérése az Azure Data Factory [integrációs modul](/azure/data-factory/concepts-integration-runtime), az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="db5c3-284">Next, use the Azure Portal to get the authentication key for the Azure Data Factory [integration runtime](/azure/data-factory/concepts-integration-runtime), as follows:</span></span>

1. <span data-ttu-id="db5c3-285">Az a [az Azure Portal](https://portal.azure.com/), keresse meg a Data Factory-példányt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-285">In the [Azure Portal](https://portal.azure.com/), navigate to the Data Factory instance.</span></span>

2. <span data-ttu-id="db5c3-286">A Data Factory panelen kattintson a **létrehozás és Monitorozás**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-286">In the Data Factory blade, click **Author & Monitor**.</span></span> <span data-ttu-id="db5c3-287">Ekkor megnyílik az Azure Data Factory portálon egy másik böngészőablakban.</span><span class="sxs-lookup"><span data-stu-id="db5c3-287">This opens the Azure Data Factory portal in another browser window.</span></span>

    ![](./images/adf-blade.png)

3. <span data-ttu-id="db5c3-288">Az Azure Data Factory-portálon válassza a ceruza ikonra ("Szerző").</span><span class="sxs-lookup"><span data-stu-id="db5c3-288">In the Azure Data Factory portal, select the pencil icon ("Author").</span></span> 

4. <span data-ttu-id="db5c3-289">Kattintson a **kapcsolatok**, majd válassza ki **integrációs modulok**.</span><span class="sxs-lookup"><span data-stu-id="db5c3-289">Click **Connections**, and then select **Integration Runtimes**.</span></span>

5. <span data-ttu-id="db5c3-290">A **sourceIntegrationRuntime**, kattintson a ceruza ikonra ("Szerkesztés").</span><span class="sxs-lookup"><span data-stu-id="db5c3-290">Under **sourceIntegrationRuntime**, click the pencil icon ("Edit").</span></span>

    > [!NOTE]
    > <span data-ttu-id="db5c3-291">A portálon az jelenik meg a "nem" állapot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-291">The portal will show the status as "unavailable".</span></span> <span data-ttu-id="db5c3-292">Ez várható, amíg a helyszíni kiszolgálón telepít.</span><span class="sxs-lookup"><span data-stu-id="db5c3-292">This is expected until you deploy the on-premises server.</span></span>

6. <span data-ttu-id="db5c3-293">Keresés **Key1** , és másolja a hitelesítési kulcs értékét.</span><span class="sxs-lookup"><span data-stu-id="db5c3-293">Find **Key1** and copy the value of the authentication key.</span></span>

<span data-ttu-id="db5c3-294">Szüksége lesz a hitelesítési kulcs a következő lépéssel.</span><span class="sxs-lookup"><span data-stu-id="db5c3-294">You will need the authentication key for the next step.</span></span>

### <a name="deploy-the-simulated-on-premises-server"></a><span data-ttu-id="db5c3-295">A szimulált helyszíni kiszolgáló üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="db5c3-295">Deploy the simulated on-premises server</span></span>

<span data-ttu-id="db5c3-296">Ebben a lépésben egy szimulált helyszíni kiszolgálóként, amely tartalmazza az SQL Server 2017-ben és a kapcsolódó eszközök üzembe helyez egy virtuális Gépet.</span><span class="sxs-lookup"><span data-stu-id="db5c3-296">This step deploys a VM as a simulated on-premises server, which includes SQL Server 2017 and related tools.</span></span> <span data-ttu-id="db5c3-297">Emellett betölti a [Wide World Importers OLTP adatbázis] [wwi] SQL-kiszolgálóra.</span><span class="sxs-lookup"><span data-stu-id="db5c3-297">It also loads the [Wide World Importers OLTP database][wwi] into SQL Server.</span></span>

1. <span data-ttu-id="db5c3-298">Keresse meg a `data\enterprise_bi_sqldw_advanced\onprem\templates` mappában található az adattárban.</span><span class="sxs-lookup"><span data-stu-id="db5c3-298">Navigate to the `data\enterprise_bi_sqldw_advanced\onprem\templates` folder of the repository.</span></span>

2. <span data-ttu-id="db5c3-299">Az a `onprem.parameters.json` fájlt, keressen rá a `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-299">In the `onprem.parameters.json` file, search for `adminPassword`.</span></span> <span data-ttu-id="db5c3-300">Ez az a jelszó az SQL Server VM-ba való bejelentkezéshez.</span><span class="sxs-lookup"><span data-stu-id="db5c3-300">This is the password to log into the SQL Server VM.</span></span> <span data-ttu-id="db5c3-301">Cserélje le az értéket egy másik jelszót.</span><span class="sxs-lookup"><span data-stu-id="db5c3-301">Replace the value with another password.</span></span>

3. <span data-ttu-id="db5c3-302">Ugyanebben a fájlban lévő keresése `SqlUserCredentials`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-302">In the same file, search for `SqlUserCredentials`.</span></span> <span data-ttu-id="db5c3-303">Ez a tulajdonság határozza meg az SQL Server fiók hitelesítő adatait.</span><span class="sxs-lookup"><span data-stu-id="db5c3-303">This property specifies the SQL Server account credentials.</span></span> <span data-ttu-id="db5c3-304">Cserélje le a jelszót egy másik értéket.</span><span class="sxs-lookup"><span data-stu-id="db5c3-304">Replace the password with a different value.</span></span>

4. <span data-ttu-id="db5c3-305">Ugyanebben a fájlban illessze be az Integration Runtime hitelesítési kulcsot a `IntegrationRuntimeGatewayKey` paramétert, a lent látható módon:</span><span class="sxs-lookup"><span data-stu-id="db5c3-305">In the same file, paste the Integration Runtime authentication key into the `IntegrationRuntimeGatewayKey` parameter, as shown below:</span></span>

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

5. <span data-ttu-id="db5c3-306">Futtassa a következő parancsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-306">Run the following command.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

<span data-ttu-id="db5c3-307">Ebben a lépésben 20 – 30 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="db5c3-307">This step may take 20 to 30 minutes to complete.</span></span> <span data-ttu-id="db5c3-308">Ez magában foglalja a futó egy [DSC](/powershell/dsc/overview) szkriptet az eszközök telepítésére, és állítsa vissza az adatbázist.</span><span class="sxs-lookup"><span data-stu-id="db5c3-308">It includes running a [DSC](/powershell/dsc/overview) script to install the tools and restore the database.</span></span> 

### <a name="deploy-azure-resources"></a><span data-ttu-id="db5c3-309">Az Azure-erőforrások üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="db5c3-309">Deploy Azure resources</span></span>

<span data-ttu-id="db5c3-310">Ebben a lépésben az SQL Data warehouse-ba, az Azure Analysis Services és a Data Factory látja el.</span><span class="sxs-lookup"><span data-stu-id="db5c3-310">This step provisions SQL Data Warehouse, Azure Analysis Services, and Data Factory.</span></span>

1. <span data-ttu-id="db5c3-311">Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-adattár] [ref-arch-tárház].</span><span class="sxs-lookup"><span data-stu-id="db5c3-311">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="db5c3-312">Futtassa a következő Azure CLI-parancsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-312">Run the following Azure CLI command.</span></span> <span data-ttu-id="db5c3-313">Cserélje le a csúcsos zárójelben látható paraméter értékét.</span><span class="sxs-lookup"><span data-stu-id="db5c3-313">Replace the parameter values shown in angle brackets.</span></span>

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - <span data-ttu-id="db5c3-314">A `storageAccountName` paraméter kell követnie a [elnevezési szabályok](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) tárfiókok esetében.</span><span class="sxs-lookup"><span data-stu-id="db5c3-314">The `storageAccountName` parameter must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span> 
    - <span data-ttu-id="db5c3-315">Az a `analysisServerAdmin` paramétert, használja az Azure Active Directory egyszerű felhasználónév (UPN).</span><span class="sxs-lookup"><span data-stu-id="db5c3-315">For the `analysisServerAdmin` parameter, use your Azure Active Directory user principal name (UPN).</span></span>

3. <span data-ttu-id="db5c3-316">Futtassa a következő Azure CLI-parancsot a tárfiók a tárelérési kulcs lekérésével.</span><span class="sxs-lookup"><span data-stu-id="db5c3-316">Run the following Azure CLI command to get the access key for the storage account.</span></span> <span data-ttu-id="db5c3-317">Ezt a kulcsot a következő lépésben fogja használni.</span><span class="sxs-lookup"><span data-stu-id="db5c3-317">You will use this key in the next step.</span></span>

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. <span data-ttu-id="db5c3-318">Futtassa a következő Azure CLI-parancsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-318">Run the following Azure CLI command.</span></span> <span data-ttu-id="db5c3-319">Cserélje le a csúcsos zárójelben látható paraméter értékét.</span><span class="sxs-lookup"><span data-stu-id="db5c3-319">Replace the parameter values shown in angle brackets.</span></span> 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    <span data-ttu-id="db5c3-320">A kapcsolati karakterláncok rendelkezik oszt fel kell helyettesíteni csúcsos zárójelben látható.</span><span class="sxs-lookup"><span data-stu-id="db5c3-320">The connection strings have substrings shown in angle brackets that must be replaced.</span></span> <span data-ttu-id="db5c3-321">A `<storage_account_key>`, használja az előző lépésben kapott kulcsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-321">For `<storage_account_key>`, use the key that you got in the previous step.</span></span> <span data-ttu-id="db5c3-322">A `<sql-db-password>`, a megadott SQL Server-fiókja jelszavát használja a `onprem.parameters.json` korábban már fájlba.</span><span class="sxs-lookup"><span data-stu-id="db5c3-322">For `<sql-db-password>`, use the SQL Server account password that you specified in the `onprem.parameters.json` file previously.</span></span>

### <a name="run-the-data-warehouse-scripts"></a><span data-ttu-id="db5c3-323">A data warehouse parancsfájlok futtatása</span><span class="sxs-lookup"><span data-stu-id="db5c3-323">Run the data warehouse scripts</span></span>

1. <span data-ttu-id="db5c3-324">Az a [az Azure Portal](https://portal.azure.com/), keresse meg a helyszíni virtuális gép, amelynek neve `sql-vm1`.</span><span class="sxs-lookup"><span data-stu-id="db5c3-324">In the [Azure Portal](https://portal.azure.com/), find the on-premises VM, which is named `sql-vm1`.</span></span> <span data-ttu-id="db5c3-325">A felhasználónév és jelszó a virtuális gép meg van adva a `onprem.parameters.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="db5c3-325">The user name and password for the VM are specified in the `onprem.parameters.json` file.</span></span>

2. <span data-ttu-id="db5c3-326">Kattintson a **Connect** és a távoli asztal használata a virtuális Géphez való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="db5c3-326">Click **Connect** and use Remote Desktop to connect to the VM.</span></span>

3. <span data-ttu-id="db5c3-327">A távoli asztali munkamenetből nyisson meg egy parancssort, és keresse meg a virtuális gépen a következő mappában:</span><span class="sxs-lookup"><span data-stu-id="db5c3-327">From your Remote Desktop session, open a command prompt and navigate to the following folder on the VM:</span></span>

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. <span data-ttu-id="db5c3-328">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="db5c3-328">Run the following command:</span></span>

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    <span data-ttu-id="db5c3-329">A `<data_warehouse_server_name>` és `<data_warehouse_password>`, adatraktár-kiszolgáló nevét és a korábbi jelszót használja.</span><span class="sxs-lookup"><span data-stu-id="db5c3-329">For `<data_warehouse_server_name>` and `<data_warehouse_password>`, use the data warehouse server name and password from earlier.</span></span>

<span data-ttu-id="db5c3-330">Ebben a lépésben ellenőrzéséhez használhatja az SQL Server Management Studio (SSMS) az SQL Data Warehouse-adatbázishoz való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="db5c3-330">To verify this step, you can use SQL Server Management Studio (SSMS) to connect to the SQL Data Warehouse database.</span></span> <span data-ttu-id="db5c3-331">Az adatbázis táblasémákat kell megjelennie.</span><span class="sxs-lookup"><span data-stu-id="db5c3-331">You should see the database table schemas.</span></span>

### <a name="run-the-data-factory-pipeline"></a><span data-ttu-id="db5c3-332">A Data Factory-folyamat futtatása</span><span class="sxs-lookup"><span data-stu-id="db5c3-332">Run the Data Factory pipeline</span></span>

1. <span data-ttu-id="db5c3-333">Az azonos távoli asztali munkamenetet nyissa meg egy PowerShell-ablakot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-333">From the same Remote Desktop session, open a PowerShell window.</span></span>

2. <span data-ttu-id="db5c3-334">Futtassa az alábbi PowerShell-parancsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-334">Run the following PowerShell command.</span></span> <span data-ttu-id="db5c3-335">Válasszon **Igen** amikor a rendszer kéri.</span><span class="sxs-lookup"><span data-stu-id="db5c3-335">Choose **Yes** when prompted.</span></span>

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. <span data-ttu-id="db5c3-336">Futtassa az alábbi PowerShell-parancsot.</span><span class="sxs-lookup"><span data-stu-id="db5c3-336">Run the following PowerShell command.</span></span> <span data-ttu-id="db5c3-337">Adja meg, amikor a rendszer kéri az Azure hitelesítő adatait.</span><span class="sxs-lookup"><span data-stu-id="db5c3-337">Enter your Azure credentials when prompted.</span></span>

    ```powershell
    Connect-AzureRmAccount 
    ```

4. <span data-ttu-id="db5c3-338">Futtassa a következő PowerShell-parancsokat.</span><span class="sxs-lookup"><span data-stu-id="db5c3-338">Run the following PowerShell commands.</span></span> <span data-ttu-id="db5c3-339">Cserélje le a csúcsos zárójeleket értékeit.</span><span class="sxs-lookup"><span data-stu-id="db5c3-339">Replace the values in angle brackets.</span></span>

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
    <span data-ttu-id="db5c3-340">Értékesítési összeg: összeg = ("Tény eladás" [adóval együtt összesen])</span><span class="sxs-lookup"><span data-stu-id="db5c3-340">Total Sales:=SUM('Fact Sale'[Total Including Tax])</span></span>
    ```

4. Repeat these steps to create the following measures:

    ```
    <span data-ttu-id="db5c3-341">Évek száma: (MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber])) + 1 =</span><span class="sxs-lookup"><span data-stu-id="db5c3-341">Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1</span></span>
    
    <span data-ttu-id="db5c3-342">Nyitó feltöltése: = CALCULATE (SUM ("Tény CityPopulation" [Population]), szűrő ('Tény CityPopulation', 'Tény CityPopulation' [YearNumber] = MIN('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="db5c3-342">Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="db5c3-343">Befejezési népesség: = CALCULATE (SUM ("Tény CityPopulation" [Population]), szűrő ('Tény CityPopulation', "Tény CityPopulation" [YearNumber] = MAX('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="db5c3-343">Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="db5c3-344">CAGR: = IFERROR ((([befejező Population] / [kezdő Population]) ^ (1 / [évek száma])) – 1,0)</span><span class="sxs-lookup"><span data-stu-id="db5c3-344">CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)</span></span>
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
