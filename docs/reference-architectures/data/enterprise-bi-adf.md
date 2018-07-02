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
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a><span data-ttu-id="8215e-103">Az SQL-adatraktár és az Azure Data Factory automatizált vállalati BI</span><span class="sxs-lookup"><span data-stu-id="8215e-103">Automated enterprise BI with SQL Data Warehouse and Azure Data Factory</span></span>

<span data-ttu-id="8215e-104">A referencia-architektúrában bemutatja, hogyan hajthat végre a növekményes betöltése egy [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (kivonat-betöltési-átalakítási) folyamat.</span><span class="sxs-lookup"><span data-stu-id="8215e-104">This reference architecture shows how to perform incremental loading in an [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) pipeline.</span></span> <span data-ttu-id="8215e-105">Az Azure Data Factory segítségével automatizálhatja a ELT folyamat.</span><span class="sxs-lookup"><span data-stu-id="8215e-105">It uses Azure Data Factory to automate the ELT pipeline.</span></span> <span data-ttu-id="8215e-106">A feldolgozási sor Növekményesen helyez a legújabb OLTP adatokat a helyszíni SQL Server-adatbázist az SQL Data Warehouse.</span><span class="sxs-lookup"><span data-stu-id="8215e-106">The pipeline incrementally moves the latest OLTP data from an on-premises SQL Server database into SQL Data Warehouse.</span></span> <span data-ttu-id="8215e-107">Tranzakciós adatok alakul egy táblázatos modell elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="8215e-107">Transactional data is transformed into a tabular model for analysis.</span></span> [<span data-ttu-id="8215e-108">**A megoldás üzembe helyezése**.</span><span class="sxs-lookup"><span data-stu-id="8215e-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

<span data-ttu-id="8215e-109">Ez az architektúra látható egy épít [az SQL Data Warehouse szolgáltatással vállalati BI](./enterprise-bi-sqldw.md), néhány fontos vállalati adatraktározási forgatókönyvekben szolgáltatásokkal bővíti, de.</span><span class="sxs-lookup"><span data-stu-id="8215e-109">This architecture builds on the one shown in [Enterprise BI with SQL Data Warehouse](./enterprise-bi-sqldw.md), but adds some features that are important for enterprise data warehousing scenarios.</span></span>

-   <span data-ttu-id="8215e-110">A Data Factory használatával folyamatának Automation.</span><span class="sxs-lookup"><span data-stu-id="8215e-110">Automation of the pipeline using Data Factory.</span></span>
-   <span data-ttu-id="8215e-111">Növekményes betöltése.</span><span class="sxs-lookup"><span data-stu-id="8215e-111">Incremental loading.</span></span>
-   <span data-ttu-id="8215e-112">Több adatforrást integrálásával.</span><span class="sxs-lookup"><span data-stu-id="8215e-112">Integrating multiple data sources.</span></span>
-   <span data-ttu-id="8215e-113">Bináris adatok, például a földrajzi adatok és a képek betöltését.</span><span class="sxs-lookup"><span data-stu-id="8215e-113">Loading binary data such as geospatial data and images.</span></span>

## <a name="architecture"></a><span data-ttu-id="8215e-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="8215e-114">Architecture</span></span>

<span data-ttu-id="8215e-115">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="8215e-115">The architecture consists of the following components.</span></span>

### <a name="data-sources"></a><span data-ttu-id="8215e-116">Adatforrások</span><span class="sxs-lookup"><span data-stu-id="8215e-116">Data sources</span></span>

<span data-ttu-id="8215e-117">**Helyszíni SQL Server**.</span><span class="sxs-lookup"><span data-stu-id="8215e-117">**On-premises SQL Server**.</span></span> <span data-ttu-id="8215e-118">Az adatok a helyszíni SQL Server-adatbázisban található.</span><span class="sxs-lookup"><span data-stu-id="8215e-118">The source data is located in a SQL Server database on premises.</span></span> <span data-ttu-id="8215e-119">A helyszíni környezetben, az az architektúra biztosítása a telepített SQL Server Azure virtuális gép üzembe helyezési parancsfájlok szimulálásához.</span><span class="sxs-lookup"><span data-stu-id="8215e-119">To simulate the on-premises environment, the deployment scripts for this architecture provision a virtual machine in Azure with SQL Server installed.</span></span> <span data-ttu-id="8215e-120">[Wide World Importers OLTP-mintaadatbázist] [wwi] szolgál a forrás-adatbázisként.</span><span class="sxs-lookup"><span data-stu-id="8215e-120">The [Wide World Importers OLTP sample database][wwi] is used as the source database.</span></span>

<span data-ttu-id="8215e-121">**Külső adatok**.</span><span class="sxs-lookup"><span data-stu-id="8215e-121">**External data**.</span></span> <span data-ttu-id="8215e-122">Az adatraktárak egy általános forgatókönyv integrálásához több adatforrás.</span><span class="sxs-lookup"><span data-stu-id="8215e-122">A common scenario for data warehouses is to integrate multiple data sources.</span></span> <span data-ttu-id="8215e-123">A referencia-architektúrában betölt egy külső adathalmaz évente város feltöltések tartalmazza, amely integrálható a az OLTP adatbázis adatait.</span><span class="sxs-lookup"><span data-stu-id="8215e-123">This reference architecture loads an external data set that contains city populations by year, and integrates it with the data from the OLTP database.</span></span> <span data-ttu-id="8215e-124">Használhatja ezeket az adatokat, mint az elemzések: "Nem minden régióban értékesítési növekedés nagyobb vagy populáció növekedésének?"</span><span class="sxs-lookup"><span data-stu-id="8215e-124">You can use this data for insights such as: "Does sales growth in each region match or exceed population growth?"</span></span>

### <a name="ingestion-and-data-storage"></a><span data-ttu-id="8215e-125">Adatfeldolgozást és adattárolási</span><span class="sxs-lookup"><span data-stu-id="8215e-125">Ingestion and data storage</span></span>

<span data-ttu-id="8215e-126">**BLOB Storage**.</span><span class="sxs-lookup"><span data-stu-id="8215e-126">**Blob Storage**.</span></span> <span data-ttu-id="8215e-127">A BLOB storage szolgál egy átmeneti területre a forrásadatok azt az SQL Data Warehouse betöltése előtt.</span><span class="sxs-lookup"><span data-stu-id="8215e-127">Blob storage is used as a staging area for the source data before loading it into SQL Data Warehouse.</span></span>

<span data-ttu-id="8215e-128">**Azure SQL Data Warehouse**.</span><span class="sxs-lookup"><span data-stu-id="8215e-128">**Azure SQL Data Warehouse**.</span></span> <span data-ttu-id="8215e-129">[Az SQL Data Warehouse](/azure/sql-data-warehouse/) egy elosztott rendszer készült elemzés végrehajtásához, nagy.</span><span class="sxs-lookup"><span data-stu-id="8215e-129">[SQL Data Warehouse](/azure/sql-data-warehouse/) is a distributed system designed to perform analytics on large data.</span></span> <span data-ttu-id="8215e-130">Támogatja a jelentős olyan párhuzamos feldolgozási (MPP), épp ezért kiválóan alkalmas a nagy teljesítményű analytics futtatásához.</span><span class="sxs-lookup"><span data-stu-id="8215e-130">It supports massive parallel processing (MPP), which makes it suitable for running high-performance analytics.</span></span> 

<span data-ttu-id="8215e-131">**Az Azure Data Factory**.</span><span class="sxs-lookup"><span data-stu-id="8215e-131">**Azure Data Factory**.</span></span> <span data-ttu-id="8215e-132">[Adat-előállító] [adf] koordinálja és automatizálja az adatmozgás és adatok átalakítása felügyelt szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="8215e-132">[Data Factory][adf] is a managed service that orchestrates and automates data movement and data transformation.</span></span> <span data-ttu-id="8215e-133">Ebben az architektúrában koordinálja a ELT folyamat különböző szakaszaiban.</span><span class="sxs-lookup"><span data-stu-id="8215e-133">In this architecture, it coordinates the various stages of the ELT process.</span></span>

### <a name="analysis-and-reporting"></a><span data-ttu-id="8215e-134">Elemzési és jelentéskészítési</span><span class="sxs-lookup"><span data-stu-id="8215e-134">Analysis and reporting</span></span>

<span data-ttu-id="8215e-135">**Az Azure Analysis Services**.</span><span class="sxs-lookup"><span data-stu-id="8215e-135">**Azure Analysis Services**.</span></span> <span data-ttu-id="8215e-136">[Analysis Services](/azure/analysis-services/) egy teljes körűen felügyelt szolgáltatás, amely a modellezési képességekkel adatokat biztosít.</span><span class="sxs-lookup"><span data-stu-id="8215e-136">[Analysis Services](/azure/analysis-services/) is a fully managed service that provides data modeling capabilities.</span></span> <span data-ttu-id="8215e-137">A szemantikai modell Analysis Services-bA betöltött.</span><span class="sxs-lookup"><span data-stu-id="8215e-137">The semantic model is loaded into Analysis Services.</span></span>

<span data-ttu-id="8215e-138">**Power BI**.</span><span class="sxs-lookup"><span data-stu-id="8215e-138">**Power BI**.</span></span> <span data-ttu-id="8215e-139">A Power BI eszközcsomagot jelent üzleti analytics üzleti elemzések készítése adatok elemzésére.</span><span class="sxs-lookup"><span data-stu-id="8215e-139">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="8215e-140">Ebben az architektúrában lekéri az Analysis Servicesben tárolt a szemantikai modellben.</span><span class="sxs-lookup"><span data-stu-id="8215e-140">In this architecture, it queries the semantic model stored in Analysis Services.</span></span>

### <a name="authentication"></a><span data-ttu-id="8215e-141">Hitelesítés</span><span class="sxs-lookup"><span data-stu-id="8215e-141">Authentication</span></span>

<span data-ttu-id="8215e-142">**Az Azure Active Directory** (az Azure AD) hitelesíti a Power BI Analysis Services-kiszolgálóhoz csatlakozó felhasználók számára.</span><span class="sxs-lookup"><span data-stu-id="8215e-142">**Azure Active Directory** (Azure AD) authenticates users who connect to the Analysis Services server through Power BI.</span></span>

<span data-ttu-id="8215e-143">Adat-előállító segítségével is használja az Azure AD az SQL Data Warehouse egy egyszerű vagy egy felügyelt szolgáltatás identitásának (MSI) használatával hitelesíteni.</span><span class="sxs-lookup"><span data-stu-id="8215e-143">Data Factory can use also use Azure AD to authenticate to SQL Data Warehouse, by using a service principal or Managed Service Identity (MSI).</span></span> <span data-ttu-id="8215e-144">Az egyszerűség kedvéért központi telepítésre példát a SQL Server-hitelesítést használ.</span><span class="sxs-lookup"><span data-stu-id="8215e-144">For simplicity, the example deployment uses SQL Server authentication.</span></span>

## <a name="data-pipeline"></a><span data-ttu-id="8215e-145">Adatfolyamat</span><span class="sxs-lookup"><span data-stu-id="8215e-145">Data pipeline</span></span>

<span data-ttu-id="8215e-146">[Azure Data Factory] [adf], a folyamat feladat koordinálja tevékenységek logikai csoportosítása &mdash; ebben az esetben be- és az SQL Data Warehouse-adatok átalakítása.</span><span class="sxs-lookup"><span data-stu-id="8215e-146">In [Azure Data Factory][adf], a pipeline is a logical grouping of activities used to coordinate a task &mdash; in this case, loading and transforming data into SQL Data Warehouse.</span></span> 

<span data-ttu-id="8215e-147">A referencia-architektúrában definiál egy fő folyamat, amely gyermek folyamatok sorozatát futtatja.</span><span class="sxs-lookup"><span data-stu-id="8215e-147">This reference architecture defines a master pipeline that runs a sequence of child pipelines.</span></span> <span data-ttu-id="8215e-148">Minden gyermek csővezeték adatokat tölt be egy vagy több data warehouse tábláiba.</span><span class="sxs-lookup"><span data-stu-id="8215e-148">Each child pipeline loads data into one or more data warehouse tables.</span></span>

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a><span data-ttu-id="8215e-149">Növekményes betöltése</span><span class="sxs-lookup"><span data-stu-id="8215e-149">Incremental loading</span></span>

<span data-ttu-id="8215e-150">Egy automatikus ETL vagy ELT folyamat futtatásakor csak a megváltozott, mióta az előző futtatása adatok betöltésére.</span><span class="sxs-lookup"><span data-stu-id="8215e-150">When you run an automated ETL or ELT process, it's most efficient to load only the data that changed since the previous run.</span></span> <span data-ttu-id="8215e-151">Ez a lehetőség egy *növekményes terhelés*, és nem a betölti az összes adat egy teljes terhelése.</span><span class="sxs-lookup"><span data-stu-id="8215e-151">This is called an *incremental load*, as opposed to a full load that loads all of the data.</span></span> <span data-ttu-id="8215e-152">Hajtsa végre egy növekményes terhelés, kell azonosítani tudja, melyik adatok változásairól.</span><span class="sxs-lookup"><span data-stu-id="8215e-152">To perform an incremental load, you need a way to identify which data has changed.</span></span> <span data-ttu-id="8215e-153">A leggyakrabban használt módszer az, hogy használja a *magas vízjel alapján* érték, amely azt jelenti, hogy a legutóbbi értékét a forrástáblában, dátum és idő oszlop vagy egy egyedi egészszám-oszloppal néhány oszlop nyomon követése.</span><span class="sxs-lookup"><span data-stu-id="8215e-153">The most common approach is to use a *high water mark* value, which means tracking the latest value of some column in the source table, either a datetime column or a unique integer column.</span></span> 

<span data-ttu-id="8215e-154">SQL Server 2016-os verziótól kezdődően használhatja [ideiglenes táblák](/sql/relational-databases/tables/temporal-tables).</span><span class="sxs-lookup"><span data-stu-id="8215e-154">Starting with SQL Server 2016, you can use [temporal tables](/sql/relational-databases/tables/temporal-tables).</span></span> <span data-ttu-id="8215e-155">Ezek a rendszerverzióval ellátott táblákon, amely az adatmódosítások teljes megőrzése.</span><span class="sxs-lookup"><span data-stu-id="8215e-155">These are system-versioned tables that keep a full history of data changes.</span></span> <span data-ttu-id="8215e-156">Az adatbázis-kezelő automatikusan egy külön tábla minden változás előzményadatait rögzíti.</span><span class="sxs-lookup"><span data-stu-id="8215e-156">The database engine automatically records the history of every change in a separate history table.</span></span> <span data-ttu-id="8215e-157">A korábbi adatok lekérheti a lekérdezés egy FOR SYSTEM_TIME záradék hozzáadásával.</span><span class="sxs-lookup"><span data-stu-id="8215e-157">You can query the historical data by adding a FOR SYSTEM_TIME clause to a query.</span></span> <span data-ttu-id="8215e-158">Belsőleg az adatbázismotor lekérdezi a előzménytábla, ez azonban átlátható az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="8215e-158">Internally, the database engine queries the history table, but this is transparent to the application.</span></span> 

> [!NOTE]
> <span data-ttu-id="8215e-159">Az SQL Server korábbi verzióit használhatja [adatváltozás-rögzítés](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span><span class="sxs-lookup"><span data-stu-id="8215e-159">For earlier versions of SQL Server, you can use [Change Data Capture](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span></span> <span data-ttu-id="8215e-160">Ezt a módszert nem ideiglenes táblák, mint kisebb kényelmes, mert van egy különálló módosítási táblából, és kötetblokkok változásait időbélyeg helyett olyan naplósorszámot.</span><span class="sxs-lookup"><span data-stu-id="8215e-160">This approach is less convenient than temporal tables, because you have to query a separate change table, and changes are tracked by a log sequence number, rather than a timestamp.</span></span> 

<span data-ttu-id="8215e-161">A historikus táblák hasznosak dimenzió adatokat, amelyek idővel megváltozhat.</span><span class="sxs-lookup"><span data-stu-id="8215e-161">Temporal tables are useful for dimension data, which can change over time.</span></span> <span data-ttu-id="8215e-162">A ténytáblák általában egy módosítható tranzakció, például egy pénztári, ebben az esetben megőrzi a rendszer verziójának előzményei értelmetlen jelölik.</span><span class="sxs-lookup"><span data-stu-id="8215e-162">Fact tables usually represent an immutable transaction such as a sale, in which case keeping the system version history doesn't make sense.</span></span> <span data-ttu-id="8215e-163">Ehelyett tranzakciók általában kell egy olyan oszlopot, amely a tranzakció dátuma, amely a vízjel értékként használható jelöli.</span><span class="sxs-lookup"><span data-stu-id="8215e-163">Instead, transactions usually have a column that represents the transaction date, which can be used as the watermark value.</span></span> <span data-ttu-id="8215e-164">Például a Wide World Importers OLTP adatbázismodell a Sales.Invoices és Sales.InvoiceLines kell egy `LastEditedWhen` mező, amely alapértelmezés szerint az `sysdatetime()`.</span><span class="sxs-lookup"><span data-stu-id="8215e-164">For example, in the Wide World Importers OLTP databse, the Sales.Invoices and Sales.InvoiceLines tables have a `LastEditedWhen` field that defaults to `sysdatetime()`.</span></span> 

<span data-ttu-id="8215e-165">Az általános folyamat a ELT adatcsatorna itt található:</span><span class="sxs-lookup"><span data-stu-id="8215e-165">Here is the general flow for the ELT pipeline:</span></span>

1. <span data-ttu-id="8215e-166">Minden táblához adatbázisában, amikor a legutóbbi ELT feladat futtatta levágási idejének nyomon követésére.</span><span class="sxs-lookup"><span data-stu-id="8215e-166">For each table in the source database, track the cutoff time when the last ELT job ran.</span></span> <span data-ttu-id="8215e-167">Az adatraktárban tárolja az adatait.</span><span class="sxs-lookup"><span data-stu-id="8215e-167">Store this information in the data warehouse.</span></span> <span data-ttu-id="8215e-168">(A kezdeti beállítás mindig lesz állítva "1-1-1900".)</span><span class="sxs-lookup"><span data-stu-id="8215e-168">(On initial setup, all times are set to '1-1-1900'.)</span></span>

2. <span data-ttu-id="8215e-169">Során az adatok exportálása a lépést, a megadott idő objektumnak átadott paraméterként adatbázisában tárolt eljárások.</span><span class="sxs-lookup"><span data-stu-id="8215e-169">During the data export step, the cutoff time is passed as a parameter to a set of stored procedures in the source database.</span></span> <span data-ttu-id="8215e-170">A tárolt eljárások lekérdezés megváltozott vagy a megadott idő után létrehozott bejegyzésekhez.</span><span class="sxs-lookup"><span data-stu-id="8215e-170">These stored procedures query for any records that were changed or created after the cutoff time.</span></span> <span data-ttu-id="8215e-171">Az értékesítési ténytábla a `LastEditedWhen` oszlopot használja.</span><span class="sxs-lookup"><span data-stu-id="8215e-171">For the Sales fact table, the `LastEditedWhen` column is used.</span></span> <span data-ttu-id="8215e-172">A dimenzió adatok rendszerverzióval ellátott historikus táblákon szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="8215e-172">For the dimension data, system-versioned temporal tables are used.</span></span>

3. <span data-ttu-id="8215e-173">Az adatáttelepítés végeztével frissítse a tábla tárolja a megadott időpontban.</span><span class="sxs-lookup"><span data-stu-id="8215e-173">When the data migration is complete, update the table that stores the cutoff times.</span></span>

<span data-ttu-id="8215e-174">Is célszerű jegyezze fel a *Leszármaztatás* az egyes ELT futtatásához.</span><span class="sxs-lookup"><span data-stu-id="8215e-174">It's also useful to record a *lineage* for each ELT run.</span></span> <span data-ttu-id="8215e-175">Egy adott rekord a Leszármaztatás társítja a rekord, futtassa a ELT előállított adatokat.</span><span class="sxs-lookup"><span data-stu-id="8215e-175">For a given record, the lineage associates that record with the ELT run that produced the data.</span></span> <span data-ttu-id="8215e-176">Minden táblának megjelenítő kezdő és záró betöltési idők minden egyes ETL futtatásához egy új Leszármaztatás rekord jön létre.</span><span class="sxs-lookup"><span data-stu-id="8215e-176">For each ETL run, a new lineage record is created for every table, showing the starting and ending load times.</span></span> <span data-ttu-id="8215e-177">A rekordokban Leszármaztatás kulcsokat a dimenzió és a ténytáblákat táblákban tárolódnak.</span><span class="sxs-lookup"><span data-stu-id="8215e-177">The lineage keys for each record are stored in the dimension and fact tables.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="8215e-178">Az adatraktár egy új köteg adatok tölti be, miután frissítése az Analysis Services táblázatos modell.</span><span class="sxs-lookup"><span data-stu-id="8215e-178">After a new batch of data is loaded into the warehouse, refresh the Analysis Services tabular model.</span></span> <span data-ttu-id="8215e-179">Lásd: [aszinkron frissítse a REST API-val](/azure/analysis-services/analysis-services-async-refresh).</span><span class="sxs-lookup"><span data-stu-id="8215e-179">See [Asynchronous refresh with the REST API](/azure/analysis-services/analysis-services-async-refresh).</span></span>

## <a name="data-cleansing"></a><span data-ttu-id="8215e-180">Adatok tisztítása</span><span class="sxs-lookup"><span data-stu-id="8215e-180">Data cleansing</span></span>

<span data-ttu-id="8215e-181">Adatok tisztítására a ELT folyamat részének kell lennie.</span><span class="sxs-lookup"><span data-stu-id="8215e-181">Data cleansing should be part of the ELT process.</span></span> <span data-ttu-id="8215e-182">A referencia-architektúrában egy hibás adat forrása a város feltöltési táblában, ahol egyes városokat rendelkeznek nulla feltöltési esetleg mert adatot nem volt elérhető.</span><span class="sxs-lookup"><span data-stu-id="8215e-182">In this reference architecture, one source of bad data is the city population table, where some cities have zero population, perhaps because no data was available.</span></span> <span data-ttu-id="8215e-183">A feldolgozás során az a ELT-feldolgozási folyamat ezen városokat eltávolítja a város feltöltési táblából.</span><span class="sxs-lookup"><span data-stu-id="8215e-183">During processing, the ELT pipeline removes those cities from the city population table.</span></span> <span data-ttu-id="8215e-184">Az előkészítési táblák, nem pedig külső táblák tisztításokat végezhet.</span><span class="sxs-lookup"><span data-stu-id="8215e-184">Perform data cleansing on staging tables, rather than external tables.</span></span>

<span data-ttu-id="8215e-185">Ez a tárolt eljárás, amely nulla feltöltési rendelkező városokat eltávolítja a város feltöltési táblából.</span><span class="sxs-lookup"><span data-stu-id="8215e-185">Here is the stored procedure that removes the cities with zero population from the City Population table.</span></span> <span data-ttu-id="8215e-186">(A forrásfájl található [Itt](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span><span class="sxs-lookup"><span data-stu-id="8215e-186">(You can find the source file [here](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span></span> 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a><span data-ttu-id="8215e-187">Külső adatforrások</span><span class="sxs-lookup"><span data-stu-id="8215e-187">External data sources</span></span>

<span data-ttu-id="8215e-188">Az adatraktárak gyakran több forrásból származó adatok összesítése.</span><span class="sxs-lookup"><span data-stu-id="8215e-188">Data warehouses often consolidate data from multiple sources.</span></span> <span data-ttu-id="8215e-189">A referencia-architektúrában betölti demográfiai adatait tartalmazó külső adatforráshoz.</span><span class="sxs-lookup"><span data-stu-id="8215e-189">This reference architecture loads an external data source that contains demographics data.</span></span> <span data-ttu-id="8215e-190">Ez az adatkészlet érhető el az Azure blob storage részeként a [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) minta.</span><span class="sxs-lookup"><span data-stu-id="8215e-190">This dataset is available in Azure blob storage as part of the [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) sample.</span></span>

<span data-ttu-id="8215e-191">Az Azure Data Factory közvetlenül átmásolhatja blob storage használata a [blob-tároló összekötő](/azure/data-factory/connector-azure-blob-storage).</span><span class="sxs-lookup"><span data-stu-id="8215e-191">Azure Data Factory can copy directly from blob storage, using the [blob storage connector](/azure/data-factory/connector-azure-blob-storage).</span></span> <span data-ttu-id="8215e-192">Azonban az összekötő számára szükséges kapcsolati karakterláncot vagy a közös hozzáférésű jogosultságkód, ezért nem használható nyilvános olvasási hozzáférés a blob másolása.</span><span class="sxs-lookup"><span data-stu-id="8215e-192">However, the connector requires a connection string or a shared access signature, so it can't be used to copy a blob with public read access.</span></span> <span data-ttu-id="8215e-193">Megoldás a PolyBase használatával létrehoz egy külső táblát a Blob storage keresztül, és másolja a külső táblákhoz az SQL Data Warehouse.</span><span class="sxs-lookup"><span data-stu-id="8215e-193">As a workaround, you can use PolyBase to create an external table over Blob storage and then copy the external tables into SQL Data Warehouse.</span></span> 

## <a name="handling-large-binary-data"></a><span data-ttu-id="8215e-194">Nagy bináris adatok kezelése</span><span class="sxs-lookup"><span data-stu-id="8215e-194">Handling large binary data</span></span> 

<span data-ttu-id="8215e-195">A forrásadatbázis városokat táblához birtokló hely oszlop egy [geográfiai](/sql/t-sql/spatial-geography/spatial-types-geography) térbeli adatok típusa.</span><span class="sxs-lookup"><span data-stu-id="8215e-195">In the source database, the Cities table has a Location column that holds a [geography](/sql/t-sql/spatial-geography/spatial-types-geography) spatial data type.</span></span> <span data-ttu-id="8215e-196">Az SQL Data Warehouse nem támogatja a **geográfiai** írja be a natív módon, így ez a mező alakítja át a **varbinary** típus betöltése során.</span><span class="sxs-lookup"><span data-stu-id="8215e-196">SQL Data Warehouse doesn't support the **geography** type natively, so this field is converted to a **varbinary** type during loading.</span></span> <span data-ttu-id="8215e-197">(Lásd: [nem támogatott adattípusú megoldásai](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span><span class="sxs-lookup"><span data-stu-id="8215e-197">(See [Workarounds for unsupported data types](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span></span>

<span data-ttu-id="8215e-198">Ugyanakkor támogatja a PolyBase az oszlop maximális méretet `varbinary(8000)`, ami azt jelenti, hogy néhány adat csonkolódik.</span><span class="sxs-lookup"><span data-stu-id="8215e-198">However, PolyBase supports a maximum column size of `varbinary(8000)`, which means some data could be truncated.</span></span> <span data-ttu-id="8215e-199">A megoldás a probléma, hogy feloszthatja az adatok adattömbökbe exportálás során, és majd állítják vissza a adattömböket, az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="8215e-199">A workaround for this problem is to break the data up into chunks during export, and then reassemble the chunks, as follows:</span></span>

1. <span data-ttu-id="8215e-200">A hely oszlopban egy ideiglenes előkészítési tábla létrehozása.</span><span class="sxs-lookup"><span data-stu-id="8215e-200">Create a temporary staging table for the Location column.</span></span>

2. <span data-ttu-id="8215e-201">Mindegyik városhoz ossza fel a helyadatok 8000 bájtos adattömböket, ami azt eredményezi, 1 &ndash; mindegyik városhoz N sorát.</span><span class="sxs-lookup"><span data-stu-id="8215e-201">For each city, split the location data into 8000-byte chunks, resulting in 1 &ndash; N rows for each city.</span></span>

3. <span data-ttu-id="8215e-202">Az adattömbök elhelyezkedését, használja a T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) sorok átalakítás oszlopokba, és ezután az egyes város oszlopértékeit összefűzésére operátor.</span><span class="sxs-lookup"><span data-stu-id="8215e-202">To reassemble the chunks, use the T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operator to convert rows into columns and then concatenate the column values for each city.</span></span>

<span data-ttu-id="8215e-203">A probléma az, hogy minden egyes város felosztása sorok, a földrajzi adatok méretétől függően különböző számú.</span><span class="sxs-lookup"><span data-stu-id="8215e-203">The challenge is that each city will be split into a different number of rows, depending on the size of geography data.</span></span> <span data-ttu-id="8215e-204">A PIVOT operátorban működjön minden város azonos számú sort kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="8215e-204">For the PIVOT operator to work, every city must have the same number of rows.</span></span> <span data-ttu-id="8215e-205">-Re, a T-SQL-lekérdezések (amelyet meg lehet tekinteni [here][MergeLocation]) létezik néhány ütés üres értékeket tartalmazó sorok ismételt kitöltésére úgy, hogy minden város azonos számú oszlopot a PowerPivot után.</span><span class="sxs-lookup"><span data-stu-id="8215e-205">To make this work, the T-SQL query (which you can view [here][MergeLocation]) does some tricks to pad out the rows with blank values, so that every city has the same number of columns after the pivot.</span></span> <span data-ttu-id="8215e-206">Az eredményül kapott lekérdezés elemről kiderül, hogy sokkal gyorsabb, mint a sorokat egy ismétlése egyszerre kell.</span><span class="sxs-lookup"><span data-stu-id="8215e-206">The resulting query turns out to be much faster than looping through the rows one at a time.</span></span>

<span data-ttu-id="8215e-207">Ugyanezt a megközelítést képadatok szolgál.</span><span class="sxs-lookup"><span data-stu-id="8215e-207">The same approach is used for image data.</span></span>

## <a name="slowly-changing-dimensions"></a><span data-ttu-id="8215e-208">Lassú a dimenziók módosítása</span><span class="sxs-lookup"><span data-stu-id="8215e-208">Slowly changing dimensions</span></span>

<span data-ttu-id="8215e-209">Dimenzió adatok viszonylag statikus, de ez módosítható.</span><span class="sxs-lookup"><span data-stu-id="8215e-209">Dimension data is relatively static, but it can change.</span></span> <span data-ttu-id="8215e-210">A termék például egy másik termékkel kategóriához beolvasása rendelik.</span><span class="sxs-lookup"><span data-stu-id="8215e-210">For example, a product might get reassigned to a different product category.</span></span> <span data-ttu-id="8215e-211">Nincsenek lassan módosítása a dimenziók kezelésével több megközelítés közül.</span><span class="sxs-lookup"><span data-stu-id="8215e-211">There are several approaches to handling slowly changing dimensions.</span></span> <span data-ttu-id="8215e-212">Egy közös technika, úgynevezett [típus 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), adjon hozzá egy új rekordot, amikor egy dimenzió módosítások-hoz.</span><span class="sxs-lookup"><span data-stu-id="8215e-212">A common technique, called [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), is to add a new record whenever a dimension changes.</span></span> 

<span data-ttu-id="8215e-213">Ahhoz, hogy a típus 2 megközelítés implementálása a dimenziótáblák kell további oszlopokat, amelyek egy adott rekord érvényes dátumtartományt.</span><span class="sxs-lookup"><span data-stu-id="8215e-213">In order to implement the Type 2 approach, dimension tables need additional columns that specify the effective date range for a given record.</span></span> <span data-ttu-id="8215e-214">Is a forrásadatbázisból elsődleges kulcsok fog lehet duplikálni, így a dimenziótáblában egy mesterséges elsődleges kulccsal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="8215e-214">Also, primary keys from the source database will be duplicated, so the dimension table must have an artificial primary key.</span></span>

<span data-ttu-id="8215e-215">A következő kép bemutatja a Dimension.City tábla.</span><span class="sxs-lookup"><span data-stu-id="8215e-215">The following image shows the Dimension.City table.</span></span> <span data-ttu-id="8215e-216">A `WWI City ID` oszlop az elsődleges kulcsot a forrás-adatbázisból.</span><span class="sxs-lookup"><span data-stu-id="8215e-216">The `WWI City ID` column is the primary key from the source database.</span></span> <span data-ttu-id="8215e-217">A `City Key` oszlop, mint egy mesterséges kulcs az ETL-folyamat során.</span><span class="sxs-lookup"><span data-stu-id="8215e-217">The `City Key` column is an artificial key generated during the ETL pipeline.</span></span> <span data-ttu-id="8215e-218">Is figyelje meg, hogy a tábla tartalmaz `Valid From` és `Valid To` oszlop, adja meg a tartomány minden egyes sorára volt érvényes.</span><span class="sxs-lookup"><span data-stu-id="8215e-218">Also notice that the table has `Valid From` and `Valid To` columns, which define the range when each row was valid.</span></span> <span data-ttu-id="8215e-219">Aktuális értékei egy `Valid To` egyenlő "9999-12-31'.</span><span class="sxs-lookup"><span data-stu-id="8215e-219">Current values have a `Valid To` equal to '9999-12-31'.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="8215e-220">Ez a megközelítés előnye megőrzi az előzményadatok, amely elemzés igen hasznos lehet.</span><span class="sxs-lookup"><span data-stu-id="8215e-220">The advantage of this approach is that it preserves historical data, which can be valuable for analysis.</span></span> <span data-ttu-id="8215e-221">Azonban azt is jelenti a ugyanaz az entitás több sort lesz.</span><span class="sxs-lookup"><span data-stu-id="8215e-221">However, it also means there will be multiple rows for the same entity.</span></span> <span data-ttu-id="8215e-222">Például az alábbiakban a megfelelő rekordok `WWI City ID` = 28561:</span><span class="sxs-lookup"><span data-stu-id="8215e-222">For example, here are the records that match `WWI City ID` = 28561:</span></span>

![](./images/city-dimension-table-2.png)

<span data-ttu-id="8215e-223">Minden egyes forgalmi tény szeretné, hogy a tény társítandó város dimenzió tábla, egy sorban a számla dátumnak megfelelő.</span><span class="sxs-lookup"><span data-stu-id="8215e-223">For each Sales fact, you want to associate that fact with a single row in City dimension table, corresponding to the invoice date.</span></span> <span data-ttu-id="8215e-224">Az ETL-folyamat részeként egy további oszlop létrehozása, amely</span><span class="sxs-lookup"><span data-stu-id="8215e-224">As part of the ETL process, create an additional column that</span></span> 

<span data-ttu-id="8215e-225">A következő T-SQL-lekérdezés ideiglenes táblát hoz létre minden egyes számla társítja a megfelelő város kulcsot a város dimenzió táblából.</span><span class="sxs-lookup"><span data-stu-id="8215e-225">The following T-SQL query creates a temporary table that associates each invoice with the correct City Key from the City dimension table.</span></span>

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

<span data-ttu-id="8215e-226">Ez a táblázat az értékesítési ténytábla oszlopát feltölti a következőkre használható:</span><span class="sxs-lookup"><span data-stu-id="8215e-226">This table is used to populate a column in the Sales fact table:</span></span>

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

<span data-ttu-id="8215e-227">Ez az oszlop lehetővé teszi, hogy a Power BI-lekérdezést a helyes város bejegyzés egy adott értékesítési számla.</span><span class="sxs-lookup"><span data-stu-id="8215e-227">This column enables a Power BI query to find the correct City record for a given sales invoice.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="8215e-228">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="8215e-228">Security considerations</span></span>

<span data-ttu-id="8215e-229">Használhatja a fokozott biztonság érdekében [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) csak a virtuális hálózat az Azure-szolgáltatások erőforrások biztonságossá tételére.</span><span class="sxs-lookup"><span data-stu-id="8215e-229">For additional security, you can use [Virtual Network service endpoints](/azure/virtual-network/virtual-network-service-endpoints-overview) to secure Azure service resources to only your virtual network.</span></span> <span data-ttu-id="8215e-230">Teljes távolítja el ezeket az erőforrásokat, átengedi a forgalmat a virtuális hálózat csak a nyilvános Internet-hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="8215e-230">This fully removes public Internet access to those resources, allowing traffic only from your virtual network.</span></span>

<span data-ttu-id="8215e-231">Ezt a módszert használja hozzon létre egy Vnetet az Azure-ban, és létrehozhat saját Szolgáltatásvégpontok az Azure-szolgáltatásokhoz.</span><span class="sxs-lookup"><span data-stu-id="8215e-231">With this approach, you create a VNet in Azure and then create private service endpoints for Azure services.</span></span> <span data-ttu-id="8215e-232">Ezek a szolgáltatások majd az adott virtuális hálózati forgalom korlátozódik.</span><span class="sxs-lookup"><span data-stu-id="8215e-232">Those services are then restricted to traffic from that virtual network.</span></span> <span data-ttu-id="8215e-233">Is elérhető azokat a helyi hálózatról átjárón keresztül.</span><span class="sxs-lookup"><span data-stu-id="8215e-233">You can also reach them from your on-premises network through a gateway.</span></span>

<span data-ttu-id="8215e-234">Vegye figyelembe a következő korlátozások vonatkoznak:</span><span class="sxs-lookup"><span data-stu-id="8215e-234">Be aware of the following limitations:</span></span>

- <span data-ttu-id="8215e-235">Időben a referencia-architektúrában jött létre, Szolgáltatásvégpontok támogatott Azure tárolási és az Azure SQL Data Warehouse, de nem Azure Analysis Service VNet.</span><span class="sxs-lookup"><span data-stu-id="8215e-235">At the time this reference architecture was created, VNet service endpoints are supported for Azure Storage and Azure SQL Data Warehouse, but not for Azure Analysis Service.</span></span> <span data-ttu-id="8215e-236">A legfrissebb állapotának [Itt](https://azure.microsoft.com/updates/?product=virtual-network).</span><span class="sxs-lookup"><span data-stu-id="8215e-236">Check the latest status [here](https://azure.microsoft.com/updates/?product=virtual-network).</span></span> 

- <span data-ttu-id="8215e-237">Ha Szolgáltatásvégpontok engedélyezve vannak az Azure Storage, a PolyBase az SQL Data Warehouse nem lehet másolni adatok a tároló.</span><span class="sxs-lookup"><span data-stu-id="8215e-237">If service endpoints are enabled for Azure Storage, PolyBase cannot copy data from Storage into SQL Data Warehouse.</span></span> <span data-ttu-id="8215e-238">A megoldás a probléma van.</span><span class="sxs-lookup"><span data-stu-id="8215e-238">There is a mitigation for this issue.</span></span> <span data-ttu-id="8215e-239">További információkért lásd: [VNet Szolgáltatásvégpontok használata az Azure storage hatását](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span><span class="sxs-lookup"><span data-stu-id="8215e-239">For more information, see [Impact of using VNet Service Endpoints with Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span></span> 

- <span data-ttu-id="8215e-240">Az Azure Storage helyez át adatokat a helyszíni, akkor nyilvános IP-címek engedélyezési listája a helyszíni vagy ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="8215e-240">To move data from on-premises into Azure Storage, you will need to whitelist public IP addresses from your on-premises or ExpressRoute.</span></span> <span data-ttu-id="8215e-241">További információkért lásd: [virtuális hálózatok védelmét biztosító Azure-szolgáltatások](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span><span class="sxs-lookup"><span data-stu-id="8215e-241">For details, see [Securing Azure services to virtual networks](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span></span>

- <span data-ttu-id="8215e-242">Ahhoz, hogy az Analysis Services adatokat olvasni az SQL Data Warehouse, telepítse a Windows virtuális Gépet a virtuális hálózat, amely tartalmazza az SQL Data Warehouse szolgáltatásvégpontot.</span><span class="sxs-lookup"><span data-stu-id="8215e-242">To enable Analysis Services to read data from SQL Data Warehouse, deploy a Windows VM to the virtual network that contains the SQL Data Warehouse service endpoint.</span></span> <span data-ttu-id="8215e-243">Telepítés [Azure a helyszíni adatátjáró](/azure/analysis-services/analysis-services-gateway) a virtuális Gépet.</span><span class="sxs-lookup"><span data-stu-id="8215e-243">Install [Azure On-premises Data Gateway](/azure/analysis-services/analysis-services-gateway) on this VM.</span></span> <span data-ttu-id="8215e-244">Az Azure Analysis service majd csatlakozni a data gateway.</span><span class="sxs-lookup"><span data-stu-id="8215e-244">Then connect your Azure Analysis service to the data gateway.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="8215e-245">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="8215e-245">Deploy the solution</span></span>

<span data-ttu-id="8215e-246">A központi telepítés a referenciaarchitektúra [GitHub] [ref-architektúrája-tárház-mappa] érhető el.</span><span class="sxs-lookup"><span data-stu-id="8215e-246">A deployment for this reference architecture is available on [GitHub][ref-arch-repo-folder].</span></span> <span data-ttu-id="8215e-247">A következőket helyezi üzembe:</span><span class="sxs-lookup"><span data-stu-id="8215e-247">It deploys the following:</span></span>

  * <span data-ttu-id="8215e-248">Windows virtuális gép egy helyi adatbázis-kiszolgáló szimulálásához.</span><span class="sxs-lookup"><span data-stu-id="8215e-248">A Windows VM to simulate an on-premises database server.</span></span> <span data-ttu-id="8215e-249">SQL Server 2017 és a kapcsolódó eszközök, valamint Power BI Desktop tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="8215e-249">It includes SQL Server 2017 and related tools, along with Power BI Desktop.</span></span>
  * <span data-ttu-id="8215e-250">Egy Azure storage-fiók, amely Blob-tároló az SQL Server adatbázisból exportált adatok tárolásához.</span><span class="sxs-lookup"><span data-stu-id="8215e-250">An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.</span></span>
  * <span data-ttu-id="8215e-251">Egy Azure SQL Data Warehouse-példányhoz.</span><span class="sxs-lookup"><span data-stu-id="8215e-251">An Azure SQL Data Warehouse instance.</span></span>
  * <span data-ttu-id="8215e-252">Egy Azure Analysis Services-példányon.</span><span class="sxs-lookup"><span data-stu-id="8215e-252">An Azure Analysis Services instance.</span></span>
  * <span data-ttu-id="8215e-253">Az Azure Data Factory és a Data Factory-folyamathoz, ELT feladat.</span><span class="sxs-lookup"><span data-stu-id="8215e-253">Azure Data Factory and the Data Factory pipeline for the ELT job.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="8215e-254">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="8215e-254">Prerequisites</span></span>

1. <span data-ttu-id="8215e-255">Klónozás, elágazás vagy az [Azure hivatkozás architektúrák] [ref-architektúrája-tárház] GitHub tárház zip-fájl letöltése.</span><span class="sxs-lookup"><span data-stu-id="8215e-255">Clone, fork, or download the zip file for the [Azure reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="8215e-256">Telepítse az [Azure építőelemeket] [azbb-wiki] (azbb).</span><span class="sxs-lookup"><span data-stu-id="8215e-256">Install the [Azure Building Blocks][azbb-wiki] (azbb).</span></span>

3. <span data-ttu-id="8215e-257">A Power BI Desktop kapcsolatos további információkért lásd: első lépések a Power BI Desktop.</span><span class="sxs-lookup"><span data-stu-id="8215e-257">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account as follows:</span></span>

    ```bash
    az login  
    ```

### <a name="variables"></a><span data-ttu-id="8215e-258">Változók</span><span class="sxs-lookup"><span data-stu-id="8215e-258">Variables</span></span>

<span data-ttu-id="8215e-259">A következő lépések bizonyos felhasználói változók tartalmazzák.</span><span class="sxs-lookup"><span data-stu-id="8215e-259">The steps that follow include some user-defined variables.</span></span> <span data-ttu-id="8215e-260">Szüksége lesz az Ön által meghatározott értékeket cseréli le.</span><span class="sxs-lookup"><span data-stu-id="8215e-260">You will need to replace these with values that you define.</span></span>

- <span data-ttu-id="8215e-261">`<data_factory_name>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-261">`<data_factory_name>`.</span></span> <span data-ttu-id="8215e-262">Adat-előállító neve.</span><span class="sxs-lookup"><span data-stu-id="8215e-262">Data Factory name.</span></span>
- <span data-ttu-id="8215e-263">`<analysis_server_name>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-263">`<analysis_server_name>`.</span></span> <span data-ttu-id="8215e-264">Analysis Services-kiszolgáló neve.</span><span class="sxs-lookup"><span data-stu-id="8215e-264">Analysis Services server name.</span></span>
- <span data-ttu-id="8215e-265">`<active_directory_upn>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-265">`<active_directory_upn>`.</span></span> <span data-ttu-id="8215e-266">Az Azure Active Directory egyszerű felhasználónév (UPN).</span><span class="sxs-lookup"><span data-stu-id="8215e-266">Your Azure Active Directory user principal name (UPN).</span></span> <span data-ttu-id="8215e-267">Például: `user@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="8215e-267">For example, `user@contoso.com`.</span></span>
- <span data-ttu-id="8215e-268">`<data_warehouse_server_name>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-268">`<data_warehouse_server_name>`.</span></span> <span data-ttu-id="8215e-269">Az SQL Data Warehouse-kiszolgáló neve.</span><span class="sxs-lookup"><span data-stu-id="8215e-269">SQL Data Warehouse server name.</span></span>
- <span data-ttu-id="8215e-270">`<data_warehouse_password>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-270">`<data_warehouse_password>`.</span></span> <span data-ttu-id="8215e-271">Az SQL Data Warehouse rendszergazdai jelszót.</span><span class="sxs-lookup"><span data-stu-id="8215e-271">SQL Data Warehouse administrator password.</span></span>
- <span data-ttu-id="8215e-272">`<resource_group_name>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-272">`<resource_group_name>`.</span></span> <span data-ttu-id="8215e-273">Az erőforráscsoport neve.</span><span class="sxs-lookup"><span data-stu-id="8215e-273">The name of the resource group.</span></span>
- <span data-ttu-id="8215e-274">`<region>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-274">`<region>`.</span></span> <span data-ttu-id="8215e-275">Az Azure-régió, hova szeretné telepíteni az erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="8215e-275">The Azure region where the resources will be deployed.</span></span>
- <span data-ttu-id="8215e-276">`<storage_account_name>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-276">`<storage_account_name>`.</span></span> <span data-ttu-id="8215e-277">Tárfiók neve.</span><span class="sxs-lookup"><span data-stu-id="8215e-277">Storage account name.</span></span> <span data-ttu-id="8215e-278">Hajtsa végre a [elnevezési szabályait](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) Storage-fiókok.</span><span class="sxs-lookup"><span data-stu-id="8215e-278">Must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span>
- <span data-ttu-id="8215e-279">`<sql-db-password>`.</span><span class="sxs-lookup"><span data-stu-id="8215e-279">`<sql-db-password>`.</span></span> <span data-ttu-id="8215e-280">SQL Server bejelentkezési jelszót.</span><span class="sxs-lookup"><span data-stu-id="8215e-280">SQL Server login password.</span></span>

### <a name="deploy-azure-data-factory"></a><span data-ttu-id="8215e-281">Az Azure Data Factory telepítése</span><span class="sxs-lookup"><span data-stu-id="8215e-281">Deploy Azure Data Factory</span></span>

1. <span data-ttu-id="8215e-282">Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-tárház] [ref-architektúrája-tárház].</span><span class="sxs-lookup"><span data-stu-id="8215e-282">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="8215e-283">A következő parancsot az Azure parancssori felület futtatásával hozzon létre egy erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="8215e-283">Run the following Azure CLI command to create a resource group.</span></span>  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    <span data-ttu-id="8215e-284">Adja meg, amely támogatja az SQL Data Warehouse, az Azure Analysis Services és a Data Factory v2 egy régiót.</span><span class="sxs-lookup"><span data-stu-id="8215e-284">Specify a region that supports SQL Data Warehouse, Azure Analysis Services, and Data Factory v2.</span></span> <span data-ttu-id="8215e-285">Lásd: [régiónként Azure termékek](https://azure.microsoft.com/global-infrastructure/services/)</span><span class="sxs-lookup"><span data-stu-id="8215e-285">See [Azure Products by Region](https://azure.microsoft.com/global-infrastructure/services/)</span></span>

3. <span data-ttu-id="8215e-286">A következő parancsot</span><span class="sxs-lookup"><span data-stu-id="8215e-286">Run the following command</span></span>

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

<span data-ttu-id="8215e-287">Ezt követően az Azure portál használatával a hitelesítési kulcs lekérése az Azure Data Factory [integrációs futásidejű](/azure/data-factory/concepts-integration-runtime), az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="8215e-287">Next, use the Azure Portal to get the authentication key for the Azure Data Factory [integration runtime](/azure/data-factory/concepts-integration-runtime), as follows:</span></span>

1. <span data-ttu-id="8215e-288">Az a [Azure Portal](https://portal.azure.com/), keresse meg a Data Factory-példány.</span><span class="sxs-lookup"><span data-stu-id="8215e-288">In the [Azure Portal](https://portal.azure.com/), navigate to the Data Factory instance.</span></span>

2. <span data-ttu-id="8215e-289">A Data Factory paneljén kattintson **Szerző & figyelő**.</span><span class="sxs-lookup"><span data-stu-id="8215e-289">In the Data Factory blade, click **Author & Monitor**.</span></span> <span data-ttu-id="8215e-290">Az Azure Data Factory portálon ekkor megnyílik egy másik böngészőablakban.</span><span class="sxs-lookup"><span data-stu-id="8215e-290">This opens the Azure Data Factory portal in another browser window.</span></span>

    ![](./images/adf-blade.png)

3. <span data-ttu-id="8215e-291">Az Azure Data Factory-portálon válassza ki a ceruza ikonra ("Szerző").</span><span class="sxs-lookup"><span data-stu-id="8215e-291">In the Azure Data Factory portal, select the pencil icon ("Author").</span></span> 

4. <span data-ttu-id="8215e-292">Kattintson a **kapcsolatok**, majd válassza ki **integrációs futtatókörnyezetek**.</span><span class="sxs-lookup"><span data-stu-id="8215e-292">Click **Connections**, and then select **Integration Runtimes**.</span></span>

5. <span data-ttu-id="8215e-293">A **sourceIntegrationRuntime**, kattintson a ceruza ikonra ("Edit").</span><span class="sxs-lookup"><span data-stu-id="8215e-293">Under **sourceIntegrationRuntime**, click the pencil icon ("Edit").</span></span>

    > [!NOTE]
    > <span data-ttu-id="8215e-294">A portálon jelennek meg a "nem érhető el" állapotának.</span><span class="sxs-lookup"><span data-stu-id="8215e-294">The portal will show the status as "unavailable".</span></span> <span data-ttu-id="8215e-295">Ez egy várható üzenet, amíg a helyszíni kiszolgáló központi telepítése.</span><span class="sxs-lookup"><span data-stu-id="8215e-295">This is expected until you deploy the on-premises server.</span></span>

6. <span data-ttu-id="8215e-296">Található **Key1** , és másolja a hitelesítési kulcs értékét.</span><span class="sxs-lookup"><span data-stu-id="8215e-296">Find **Key1** and copy the value of the authentication key.</span></span>

<span data-ttu-id="8215e-297">Szüksége lesz a hitelesítési kulcs a következő lépéssel.</span><span class="sxs-lookup"><span data-stu-id="8215e-297">You will need the authentication key for the next step.</span></span>

### <a name="deploy-the-simulated-on-premises-server"></a><span data-ttu-id="8215e-298">A referencia-architektúrában kapcsolatos további információkért látogasson el a GitHub-tárházban.</span><span class="sxs-lookup"><span data-stu-id="8215e-298">Deploy the simulated on-premises server</span></span>

<span data-ttu-id="8215e-299">Ebben a lépésben egy virtuális Gépet, amely tartalmazza az SQL Server 2017 és a kapcsolódó eszközök szimulált helyszíni kiszolgálóként központilag telepíti.</span><span class="sxs-lookup"><span data-stu-id="8215e-299">This step deploys a VM as a simulated on-premises server, which includes SQL Server 2017 and related tools.</span></span> <span data-ttu-id="8215e-300">Azt is betölti a [Wide World Importers OLTP adatbázis] [wwi] az SQL-kiszolgálóra.</span><span class="sxs-lookup"><span data-stu-id="8215e-300">It also loads the [Wide World Importers OLTP database][wwi] into SQL Server.</span></span>

1. <span data-ttu-id="8215e-301">Keresse meg a `data\enterprise_bi_sqldw_advanced\onprem\templates` a tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="8215e-301">Navigate to the `data\enterprise_bi_sqldw_advanced\onprem\templates` folder of the repository.</span></span>

2. <span data-ttu-id="8215e-302">Az a `onprem.parameters.json` fájlt, keresse meg `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="8215e-302">In the `onprem.parameters.json` file, search for `adminPassword`.</span></span> <span data-ttu-id="8215e-303">Ez az a jelszót az SQL Server Virtuálisgép jelentkezni.</span><span class="sxs-lookup"><span data-stu-id="8215e-303">This is the password to log into the SQL Server VM.</span></span> <span data-ttu-id="8215e-304">Cserélje le az érték egy másik jelszót.</span><span class="sxs-lookup"><span data-stu-id="8215e-304">Replace the value with another password.</span></span>

3. <span data-ttu-id="8215e-305">A ugyanazt a fájlt, keresse meg a `SqlUserCredentials`.</span><span class="sxs-lookup"><span data-stu-id="8215e-305">In the same file, search for `SqlUserCredentials`.</span></span> <span data-ttu-id="8215e-306">Ez a tulajdonság határozza meg az SQL Server fiók hitelesítő adatait.</span><span class="sxs-lookup"><span data-stu-id="8215e-306">This property specifies the SQL Server account credentials.</span></span> <span data-ttu-id="8215e-307">A jelszó cseréje egy másik értéket.</span><span class="sxs-lookup"><span data-stu-id="8215e-307">Replace the password with a different value.</span></span>

4. <span data-ttu-id="8215e-308">Ugyanebben a fájlban, illessze be azokat az integrációs futásidejű hitelesítési kulcs a `IntegrationRuntimeGatewayKey` paraméter, a lent látható módon:</span><span class="sxs-lookup"><span data-stu-id="8215e-308">In the same file, paste the Integration Runtime authentication key into the `IntegrationRuntimeGatewayKey` parameter, as shown below:</span></span>

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

5. <span data-ttu-id="8215e-309">A következő parancsot.</span><span class="sxs-lookup"><span data-stu-id="8215e-309">Run the following command.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

<span data-ttu-id="8215e-310">Ez a lépés a 20-30 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="8215e-310">This step may take 20 to 30 minutes to complete.</span></span> <span data-ttu-id="8215e-311">Ez magában foglalja a fut egy [DSC](/powershell/dsc/overview) parancsprogramot, az eszközök telepítése, majd állítsa vissza az adatbázist.</span><span class="sxs-lookup"><span data-stu-id="8215e-311">It includes running a [DSC](/powershell/dsc/overview) script to install the tools and restore the database.</span></span> 

### <a name="deploy-azure-resources"></a><span data-ttu-id="8215e-312">Azure-erőforrások telepítése</span><span class="sxs-lookup"><span data-stu-id="8215e-312">Deploy Azure resources</span></span>

<span data-ttu-id="8215e-313">Ez a lépés látja el az SQL Data Warehouse, az Azure Analysis Services és a Data Factory.</span><span class="sxs-lookup"><span data-stu-id="8215e-313">This step provisions SQL Data Warehouse, Azure Analysis Services, and Data Factory.</span></span>

1. <span data-ttu-id="8215e-314">Keresse meg a `data\enterprise_bi_sqldw_advanced\azure\templates` mappa [GitHub-tárház] [ref-architektúrája-tárház].</span><span class="sxs-lookup"><span data-stu-id="8215e-314">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="8215e-315">A következő Azure CLI parancsot.</span><span class="sxs-lookup"><span data-stu-id="8215e-315">Run the following Azure CLI command.</span></span> <span data-ttu-id="8215e-316">Cserélje le a paraméterértékeket csúcsos zárójelek látható.</span><span class="sxs-lookup"><span data-stu-id="8215e-316">Replace the parameter values shown in angle brackets.</span></span>

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - <span data-ttu-id="8215e-317">A `storageAccountName` paraméter kell követnie a [elnevezési szabályait](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) Storage-fiókok.</span><span class="sxs-lookup"><span data-stu-id="8215e-317">The `storageAccountName` parameter must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span> 
    - <span data-ttu-id="8215e-318">Az a `analysisServerAdmin` paraméter, használja az Azure Active Directory egyszerű felhasználónév (UPN).</span><span class="sxs-lookup"><span data-stu-id="8215e-318">For the `analysisServerAdmin` parameter, use your Azure Active Directory user principal name (UPN).</span></span>

3. <span data-ttu-id="8215e-319">A következő parancsot az Azure parancssori felület a hozzáférési kulcsot a tárfiók eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="8215e-319">Run the following Azure CLI command to get the access key for the storage account.</span></span> <span data-ttu-id="8215e-320">Ezt a kulcsot a következő lépésben fogja használni.</span><span class="sxs-lookup"><span data-stu-id="8215e-320">You will use this key in the next step.</span></span>

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. <span data-ttu-id="8215e-321">A következő Azure CLI parancsot.</span><span class="sxs-lookup"><span data-stu-id="8215e-321">Run the following Azure CLI command.</span></span> <span data-ttu-id="8215e-322">Cserélje le a paraméterértékeket csúcsos zárójelek látható.</span><span class="sxs-lookup"><span data-stu-id="8215e-322">Replace the parameter values shown in angle brackets.</span></span> 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    <span data-ttu-id="8215e-323">A kapcsolati karakterláncokkal rendelkezik karakterláncrészletek szög zárójelbe kell cserélni.</span><span class="sxs-lookup"><span data-stu-id="8215e-323">The connection strings have substrings shown in angle brackets that must be replaced.</span></span> <span data-ttu-id="8215e-324">A `<storage_account_key>`, a kulcs, amely az előző lépésben kapott.</span><span class="sxs-lookup"><span data-stu-id="8215e-324">For `<storage_account_key>`, use the key that you got in the previous step.</span></span> <span data-ttu-id="8215e-325">A `<sql-db-password>`, a megadott SQL Server-fiókja jelszavát használja a `onprem.parameters.json` korábban fájlt.</span><span class="sxs-lookup"><span data-stu-id="8215e-325">For `<sql-db-password>`, use the SQL Server account password that you specified in the `onprem.parameters.json` file previously.</span></span>

### <a name="run-the-data-warehouse-scripts"></a><span data-ttu-id="8215e-326">A data warehouse parancsfájlok futtatása</span><span class="sxs-lookup"><span data-stu-id="8215e-326">Run the data warehouse scripts</span></span>

1. <span data-ttu-id="8215e-327">Az a [Azure Portal](https://portal.azure.com/), a helyszíni virtuális Gépet, amelynek neve található `sql-vm1`.</span><span class="sxs-lookup"><span data-stu-id="8215e-327">In the [Azure Portal](https://portal.azure.com/), find the on-premises VM, which is named `sql-vm1`.</span></span> <span data-ttu-id="8215e-328">A felhasználónév és jelszó a virtuális gép számára meg van adva a `onprem.parameters.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="8215e-328">The user name and password for the VM are specified in the `onprem.parameters.json` file.</span></span>

2. <span data-ttu-id="8215e-329">Kattintson a **Connect** és a távoli asztal használatával csatlakoztassa a virtuális Gépet.</span><span class="sxs-lookup"><span data-stu-id="8215e-329">Click **Connect** and use Remote Desktop to connect to the VM.</span></span>

3. <span data-ttu-id="8215e-330">A távoli asztali munkamenetből nyisson meg egy parancssort, és keresse meg a virtuális Gépen a következő mappát:</span><span class="sxs-lookup"><span data-stu-id="8215e-330">From your Remote Desktop session, open a command prompt and navigate to the following folder on the VM:</span></span>

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. <span data-ttu-id="8215e-331">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8215e-331">Run the following command:</span></span>

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    <span data-ttu-id="8215e-332">A `<data_warehouse_server_name>` és `<data_warehouse_password>`, az adatraktár-kiszolgáló nevét és a korábbi jelszó használata.</span><span class="sxs-lookup"><span data-stu-id="8215e-332">For `<data_warehouse_server_name>` and `<data_warehouse_password>`, use the data warehouse server name and password from earlier.</span></span>

<span data-ttu-id="8215e-333">Ebben a lépésben ellenőrzéséhez használhatja az SQL Server Management Studio (SSMS) az SQL Data Warehouse-adatbázishoz való kapcsolódáshoz.</span><span class="sxs-lookup"><span data-stu-id="8215e-333">To verify this step, you can use SQL Server Management Studio (SSMS) to connect to the SQL Data Warehouse database.</span></span> <span data-ttu-id="8215e-334">Az adatbázis táblasémákat kell megjelennie.</span><span class="sxs-lookup"><span data-stu-id="8215e-334">You should see the database table schemas.</span></span>

### <a name="run-the-data-factory-pipeline"></a><span data-ttu-id="8215e-335">Futtassa a Data Factory-folyamat</span><span class="sxs-lookup"><span data-stu-id="8215e-335">Run the Data Factory pipeline</span></span>

1. <span data-ttu-id="8215e-336">A azonos a távoli asztal munkamenetet nyissa meg egy PowerShell-ablakot.</span><span class="sxs-lookup"><span data-stu-id="8215e-336">From the same Remote Desktop session, open a PowerShell window.</span></span>

2. <span data-ttu-id="8215e-337">Futtassa az alábbi PowerShell-parancsot.</span><span class="sxs-lookup"><span data-stu-id="8215e-337">Run the following PowerShell command.</span></span> <span data-ttu-id="8215e-338">Válasszon **Igen** megjelenésekor.</span><span class="sxs-lookup"><span data-stu-id="8215e-338">Choose **Yes** when prompted.</span></span>

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. <span data-ttu-id="8215e-339">Futtassa az alábbi PowerShell-parancsot.</span><span class="sxs-lookup"><span data-stu-id="8215e-339">Run the following PowerShell command.</span></span> <span data-ttu-id="8215e-340">Adja meg, amikor a rendszer kéri az Azure hitelesítő adatait.</span><span class="sxs-lookup"><span data-stu-id="8215e-340">Enter your Azure credentials when prompted.</span></span>

    ```powershell
    Connect-AzureRmAccount 
    ```

4. <span data-ttu-id="8215e-341">Futtassa a következő PowerShell-parancsokat.</span><span class="sxs-lookup"><span data-stu-id="8215e-341">Run the following PowerShell commands.</span></span> <span data-ttu-id="8215e-342">Cserélje le a csúcsos zárójelek értékeket.</span><span class="sxs-lookup"><span data-stu-id="8215e-342">Replace the values in angle brackets.</span></span>

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
    <span data-ttu-id="8215e-343">Összes értékesítés: = SUM ('Tény értékesítés' [összes, beleértve a adó])</span><span class="sxs-lookup"><span data-stu-id="8215e-343">Total Sales:=SUM('Fact Sale'[Total Including Tax])</span></span>
    ```

4. Repeat these steps to create the following measures:

    ```
    <span data-ttu-id="8215e-344">Évek száma: (MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber])) + 1 =</span><span class="sxs-lookup"><span data-stu-id="8215e-344">Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1</span></span>
    
    <span data-ttu-id="8215e-345">Feltöltési megkezdése: = CALCULATE (SUM ("Tény CityPopulation" [feltöltési]), szűrő ("Tény CityPopulation", "Tény CityPopulation" [YearNumber] = MIN('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="8215e-345">Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="8215e-346">Befejezési feltöltési: = CALCULATE (SUM ("Tény CityPopulation" [feltöltési]), szűrő ("Tény CityPopulation", "Tény CityPopulation" [YearNumber] = MAX('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="8215e-346">Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="8215e-347">CAGR: = IFERROR ((([a feltöltési befejező] / [kezdő feltöltési]) ^ (1 / [évek száma]))-1,0)</span><span class="sxs-lookup"><span data-stu-id="8215e-347">CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)</span></span>
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
