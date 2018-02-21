---
title: "Az adatraktározás terén és adatpiacainak"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: eec883c68cf94637c3061814d0841c73b58d7e52
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="data-warehousing-and-data-marts"></a><span data-ttu-id="419d4-102">Az adatraktározás terén és adatpiacainak</span><span class="sxs-lookup"><span data-stu-id="419d4-102">Data warehousing and data marts</span></span>

<span data-ttu-id="419d4-103">Adatraktár tárháza központi, szervezeti, relációs egy vagy több különböző forrásokból származó integrált adatok sok vagy az összes területek között.</span><span class="sxs-lookup"><span data-stu-id="419d4-103">A data warehouse is a central, organizational, relational repository of integrated data from one or more disparate sources, across many or all subject areas.</span></span> <span data-ttu-id="419d4-104">Az adatraktárak aktuális és korábbi adatokat tárolhatnak, és különböző módon használható a reporting és az adatok elemzése.</span><span class="sxs-lookup"><span data-stu-id="419d4-104">Data warehouses store current and historical data and are used for reporting and analysis of the data in different ways.</span></span>

![Az Azure-ban adatraktározási](./images/data-warehousing.png)

<span data-ttu-id="419d4-106">Áthelyezni az adatokat az adatraktárban, akkor ki kell olvasni a különböző forrásokból, amely tartalmazza a fontos üzleti adatok rendszeres időközönként.</span><span class="sxs-lookup"><span data-stu-id="419d4-106">To move data into a data warehouse, it is extracted on a periodic basis from various sources that contain important business information.</span></span> <span data-ttu-id="419d4-107">Az adatok áthelyezése, képes kell formázni, tisztítani, érvényesítve, foglalja össze, és az átszervezés.</span><span class="sxs-lookup"><span data-stu-id="419d4-107">As the data is moved, it can be formatted, cleaned, validated, summarized, and reorganized.</span></span> <span data-ttu-id="419d4-108">Alternatív megoldásként az adatokat tárolhatja a legalacsonyabb szintű részletesség, a jelentés az adatraktár előírt összesített nézetek.</span><span class="sxs-lookup"><span data-stu-id="419d4-108">Alternately, the data can be stored in the lowest level of detail, with aggregated views provided in the warehouse for reporting.</span></span> <span data-ttu-id="419d4-109">Mindkét esetben az adatraktárban használt jelentések, elemzés és fontos üzleti döntéseket üzleti intelligenciával eszközökkel képező adatok állandó tárhely válik.</span><span class="sxs-lookup"><span data-stu-id="419d4-109">In either case, the data warehouse becomes a permanent storage space for data used for reporting, analysis, and forming important business decisions using business intelligence (BI) tools.</span></span>

## <a name="data-marts-and-operational-data-stores"></a><span data-ttu-id="419d4-110">Adatpiacainak és működési adatokat tároló</span><span class="sxs-lookup"><span data-stu-id="419d4-110">Data marts and operational data stores</span></span>

<span data-ttu-id="419d4-111">Léptékű adatok kezelése bonyolult, és szeretné, hogy minden adatot a teljes vállalat jelölő egyetlen adatraktár ritkább válik.</span><span class="sxs-lookup"><span data-stu-id="419d4-111">Managing data at scale is complex, and it is becoming less common to have a single data warehouse that represents all data across the entire enterprise.</span></span> <span data-ttu-id="419d4-112">Ehelyett hozzon létre a szervezetek kisebb, célzottabb adatraktárban nevű *adatpiacainak*, amely teszi közzé a kívánt adathoz elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="419d4-112">Instead, organizations create smaller, more focused data warehouses, called *data marts*, that expose the desired data for analytics purposes.</span></span> <span data-ttu-id="419d4-113">Az orchestration folyamat feltölti az adatpiac jelenti az operatív adatok tárolóban tárolt adatok.</span><span class="sxs-lookup"><span data-stu-id="419d4-113">An orchestration process populates the data marts from data maintained in an operational data store.</span></span> <span data-ttu-id="419d4-114">A működésiadat-tároló egy közvetítőként a forrásrendszerben tranzakciós és az Adatközpont között.</span><span class="sxs-lookup"><span data-stu-id="419d4-114">The operational data store acts as an intermediary between the source transactional system and the data mart.</span></span> <span data-ttu-id="419d4-115">Az operatív adatokat tároló által kezelt tranzakciós forrásrendszerben levő adatok megtisztított verziója, és általában a data warehouse-bA vagy adatpiac által kezelt a korábbi adatok egy részét.</span><span class="sxs-lookup"><span data-stu-id="419d4-115">Data managed by the operational data store is a cleaned version of the data present in the source transactional system, and is typically a subset of the historical data that is maintained by the data warehouse or data mart.</span></span> 

## <a name="when-to-use-this-solution"></a><span data-ttu-id="419d4-116">Ez a megoldás használatával</span><span class="sxs-lookup"><span data-stu-id="419d4-116">When to use this solution</span></span>

<span data-ttu-id="419d4-117">Adatraktár akkor válassza, ha be kell kapcsolni a nagy mennyiségű adatot az operációs rendszerek olyan formátumra, amely könnyen értelmezhető, aktuális és pontos.</span><span class="sxs-lookup"><span data-stu-id="419d4-117">Choose a data warehouse when you need to turn massive amounts of data from operational systems into a format that is easy to understand, current, and accurate.</span></span> <span data-ttu-id="419d4-118">Az adatraktárak nem kell azonos tömör adatok szerkezetének az lehet, hogy használja az a működési/OLTP-adatbázisok.</span><span class="sxs-lookup"><span data-stu-id="419d4-118">Data warehouses do not need to follow the same terse data structure you may be using in your operational/OLTP databases.</span></span> <span data-ttu-id="419d4-119">Célszerű üzleti felhasználók és elemzők, a séma bővítésével leegyszerűsítheti a adatok kapcsolatok átalakításának és több táblázatot konszolidálhatják egy oszlopnevek is használhatja.</span><span class="sxs-lookup"><span data-stu-id="419d4-119">You can use column names that make sense to business users and analysts, restructure the schema to simplify data relationships, and consolidate several tables into one.</span></span> <span data-ttu-id="419d4-120">Az alábbi lépések segítségével útmutató felhasználók, akik ad hoc jelentések létrehozása és a BI rendszereken az egy adatbázis-rendszergazda (DBA) vagy a fejlesztővel adatok nélkül az adatok elemzése és jelentések létrehozása.</span><span class="sxs-lookup"><span data-stu-id="419d4-120">These steps help guide users who need to create ad hoc reports, or create reports and analyze the data in BI systems, without the help of a database administrator (DBA) or data developer.</span></span>

<span data-ttu-id="419d4-121">Adatraktár esetekben érdemes szüksége előzményadatokat különálló, a teljesítményre vonatkozó megfontolásból tranzakció forrásrendszerektől.</span><span class="sxs-lookup"><span data-stu-id="419d4-121">Consider using a data warehouse when you need to keep historical data separate from the source transaction systems for performance reasons.</span></span> <span data-ttu-id="419d4-122">Az adatraktárak könnyen előzményadatokat férhetnek több helyről, ha egy központi helyen közös formátumok, közös kulcsok, közös adatmodellekben és közös hozzáférési módszer használatával.</span><span class="sxs-lookup"><span data-stu-id="419d4-122">Data warehouses make it easy to access historical data from multiple locations, by providing a centralized location using common formats, common keys, common data models, and common access methods.</span></span>

<span data-ttu-id="419d4-123">Az adatraktárak vannak optimalizálva, olvasási hozzáférést, a jelentések futtatása ellen a forrásrendszerben tranzakció képest gyorsabb jelentéskészítésre eredményez.</span><span class="sxs-lookup"><span data-stu-id="419d4-123">Data warehouses are optimized for read access, resulting in faster report generation compared to running reports against the source transaction system.</span></span> <span data-ttu-id="419d4-124">Az adatraktárakra emellett a következő előnyöket biztosítják:</span><span class="sxs-lookup"><span data-stu-id="419d4-124">In addition, data warehouses provide the following benefits:</span></span>

* <span data-ttu-id="419d4-125">A különböző forrásokból származó összes előzményadatokat tárolja, és elérhető az adatraktár igazság egyetlen forrásaként.</span><span class="sxs-lookup"><span data-stu-id="419d4-125">All historical data from multiple sources can be stored and accessed from a data warehouse as the single source of truth.</span></span>
* <span data-ttu-id="419d4-126">Az adatminőségi javíthatja törli az adatokat a program az importáláskor az adatraktárba, pontosabb adatokat szolgáltató való egységes kódok és leírásokat.</span><span class="sxs-lookup"><span data-stu-id="419d4-126">You can improve data quality by cleaning up data as it is imported into the data warehouse, providing more accurate data as well as providing consistent codes and descriptions.</span></span>
* <span data-ttu-id="419d4-127">Jelentéskészítési eszközök ne befolyásolják a tranzakciós forrásrendszerek feldolgozási ciklus lekérdezéshez.</span><span class="sxs-lookup"><span data-stu-id="419d4-127">Reporting tools do not compete with the transactional source systems for query processing cycles.</span></span> <span data-ttu-id="419d4-128">Egy data warehouse lehetővé teszi a tranzakciós rendszer írási műveletekről, kezelése során az adatraktár megfelel a legtöbb az olvasási kérések túlnyomórészt összpontosítanak.</span><span class="sxs-lookup"><span data-stu-id="419d4-128">A data warehouse allows the transactional system to focus predominantly on handling writes, while the data warehouse satisfies the majority of read requests.</span></span>
* <span data-ttu-id="419d4-129">Adatraktár összevonni az adatokat a különböző szoftver segítségével.</span><span class="sxs-lookup"><span data-stu-id="419d4-129">A data warehouse can help consolidate data from different software.</span></span>
* <span data-ttu-id="419d4-130">Adatok adatbányászati eszközök segítséget rejtett minták használatával automatikus módszereit, amelyek a raktárban tárolt adatok alapján.</span><span class="sxs-lookup"><span data-stu-id="419d4-130">Data mining tools can help you find hidden patterns using automatic methodologies against data stored in your warehouse.</span></span>
* <span data-ttu-id="419d4-131">Az adatraktárak megkönnyítik a biztonságos hozzáférés biztosítása jogosult felhasználókra, míg mások való hozzáférés korlátozása.</span><span class="sxs-lookup"><span data-stu-id="419d4-131">Data warehouses make it easier to provide secure access to authorized users, while restricting access to others.</span></span> <span data-ttu-id="419d4-132">Nincs szükség az üzleti felhasználónak hozzáférést biztosít a forrásadatok, ezzel kiküszöbölve egy potenciális támadási felület éles tranzakció egy vagy több rendszer ellen.</span><span class="sxs-lookup"><span data-stu-id="419d4-132">There is no need to grant business users access to the source data, thereby removing a potential attack vector against one or more production transaction systems.</span></span>
* <span data-ttu-id="419d4-133">Az adatraktárak könnyebben üzleti intelligenciát biztosító megoldások felett az adatokat, például létrehozásához [OLAP-kockák](online-analytical-processing.md).</span><span class="sxs-lookup"><span data-stu-id="419d4-133">Data warehouses make it easier to create business intelligence solutions on top of the data, such as [OLAP cubes](online-analytical-processing.md).</span></span>

## <a name="challenges"></a><span data-ttu-id="419d4-134">Kihívásai</span><span class="sxs-lookup"><span data-stu-id="419d4-134">Challenges</span></span>

<span data-ttu-id="419d4-135">Egy adatraktár az az üzleti igényeinek megfelelően beállítása átvihetők a következő problémákkal egy részénél:</span><span class="sxs-lookup"><span data-stu-id="419d4-135">Properly configuring a data warehouse to fit the needs of your business can bring some of the following challenges:</span></span>

* <span data-ttu-id="419d4-136">Véglegesítő megfelelően modellezheti az üzleti fogalmak szükséges időt.</span><span class="sxs-lookup"><span data-stu-id="419d4-136">Committing the time required to properly model your business concepts.</span></span> <span data-ttu-id="419d4-137">Fontos lépés, ez, vagy az adatraktárak sem áll, ahol koncepció leképezési meghajtók a projekt a további információkat.</span><span class="sxs-lookup"><span data-stu-id="419d4-137">This is an important step, as data warehouses are information driven, where concept mapping drives the rest of the project.</span></span> <span data-ttu-id="419d4-138">Ez magában foglalja a szabványosítása üzleti-mal kapcsolatos kifejezések és a közös formázza az adathordozót (például a pénznem és dátum), és át lesznek strukturálva a séma oly módon, hogy az üzleti felhasználók számára értelmes állomásnevet, de továbbra is biztosítja az adatok összesítések és kapcsolatok pontosságát.</span><span class="sxs-lookup"><span data-stu-id="419d4-138">This involves standardizing business-related terms and common formats (such as currency and dates), and restructuring the schema in a way that makes sense to business users but still ensures accuracy of data aggregates and relationships.</span></span>
* <span data-ttu-id="419d4-139">Tervezési, valamint beállítja a adatok előkészítése.</span><span class="sxs-lookup"><span data-stu-id="419d4-139">Planning and setting up your data orchestration.</span></span> <span data-ttu-id="419d4-140">Szempont például a forrásrendszerből tranzakciós adatok másolása az adatraktár és az előzményadatok áthelyezése kívül a működési adatokat tárolja, és az adatraktárba.</span><span class="sxs-lookup"><span data-stu-id="419d4-140">Consideration include how to copy data from the source transactional system to the data warehouse, and when to move historical data out of your operational data stores and into the warehouse.</span></span>
* <span data-ttu-id="419d4-141">Fenntartása, illetve az adatminőségi javítása tisztítási, mert az adatok importálása az adatraktárba.</span><span class="sxs-lookup"><span data-stu-id="419d4-141">Maintaining or improving data quality by cleaning the data as it is imported into the warehouse.</span></span>

## <a name="data-warehousing-in-azure"></a><span data-ttu-id="419d4-142">Az Azure-ban adatraktározási</span><span class="sxs-lookup"><span data-stu-id="419d4-142">Data warehousing in Azure</span></span>

<span data-ttu-id="419d4-143">Az Azure előfordulhat, hogy egy vagy több adatforráshoz, hogy felhasználói tranzakciókat, vagy a különböző részlegek által használt különböző üzleti alkalmazások.</span><span class="sxs-lookup"><span data-stu-id="419d4-143">In Azure, you may have one or more sources of data, whether from customer transactions, or from various business applications used by various departments.</span></span> <span data-ttu-id="419d4-144">Ez hagyományosan adatai egy vagy több [OLTP](online-transaction-processing.md) adatbázisok.</span><span class="sxs-lookup"><span data-stu-id="419d4-144">This data is traditionally stored in one or more [OLTP](online-transaction-processing.md) databases.</span></span> <span data-ttu-id="419d4-145">Az adatok a más adathordozók, például a hálózati megosztások, Azure Storage Blobsba vagy data lake állandósítható.</span><span class="sxs-lookup"><span data-stu-id="419d4-145">The data could be persisted in other storage mediums such as network shares, Azure Storage Blobs, or a data lake.</span></span> <span data-ttu-id="419d4-146">Az adatok is tárolhatók, az adatraktár saját magát vagy egy relációs adatbázisban, például az Azure SQL Database.</span><span class="sxs-lookup"><span data-stu-id="419d4-146">The data could also be stored by the data warehouse itself or in a relational database such as Azure SQL Database.</span></span> <span data-ttu-id="419d4-147">Az analitikai adatok tárolási réteg célja lekérdezések elemzés és a data warehouse-bA vagy adatpiac jelentéskészítési eszközök által kibocsátott kielégítéséhez.</span><span class="sxs-lookup"><span data-stu-id="419d4-147">The purpose of the analytical data store layer is to satisfy queries issued by analytics and reporting tools against the data warehouse or data mart.</span></span> <span data-ttu-id="419d4-148">Az Azure ezen analitikai tároló képesség érheti el az Azure SQL Data Warehouse szolgáltatással, vagy az Azure HDInsight Hive vagy az interaktív lekérdezés segítségével.</span><span class="sxs-lookup"><span data-stu-id="419d4-148">In Azure, this analytical store capability can be met with Azure SQL Data Warehouse, or with Azure HDInsight using Hive or Interactive Query.</span></span> <span data-ttu-id="419d4-149">Emellett bizonyos fokú rendszeresen áthelyezése vagy adatok másolása az adattárolás az adatraktáron, amely teheti az orchestration kell a Azure HDInsight Azure Data Factory és az Oozie használatával.</span><span class="sxs-lookup"><span data-stu-id="419d4-149">In addition, you will need some level of orchestration to periodically move or copy data from data storage to the data warehouse, which can be done using Azure Data Factory or Oozie on Azure HDInsight.</span></span>

<span data-ttu-id="419d4-150">Kapcsolódó szolgáltatások:</span><span class="sxs-lookup"><span data-stu-id="419d4-150">Related services:</span></span>

* [<span data-ttu-id="419d4-151">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="419d4-151">Azure SQL Database</span></span>](/azure/sql-database/)
* [<span data-ttu-id="419d4-152">SQL Server virtuális gépen</span><span class="sxs-lookup"><span data-stu-id="419d4-152">SQL Server in a VM</span></span>](/sql/sql-server/sql-server-technical-documentation)
* [<span data-ttu-id="419d4-153">Az Azure Data warehouse-bA</span><span class="sxs-lookup"><span data-stu-id="419d4-153">Azure Data Warehouse</span></span>](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
* [<span data-ttu-id="419d4-154">Apache Hive hdinsight</span><span class="sxs-lookup"><span data-stu-id="419d4-154">Apache Hive on HDInsight</span></span>](/azure/hdinsight/hadoop/hdinsight-use-hive)
* [<span data-ttu-id="419d4-155">A HDInsight (Hive LLAP) interaktív lekérdezés</span><span class="sxs-lookup"><span data-stu-id="419d4-155">Interactive Query (Hive LLAP) on HDInsight</span></span>](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)


## <a name="technology-choices"></a><span data-ttu-id="419d4-156">Technológiai lehetőségek</span><span class="sxs-lookup"><span data-stu-id="419d4-156">Technology choices</span></span>

- [<span data-ttu-id="419d4-157">Az adatraktárak</span><span class="sxs-lookup"><span data-stu-id="419d4-157">Data warehouses</span></span>](../technology-choices/data-warehouses.md)
- [<span data-ttu-id="419d4-158">Vezénylési folyamat</span><span class="sxs-lookup"><span data-stu-id="419d4-158">Pipeline orchestration</span></span>](../technology-choices/pipeline-orchestration-data-movement.md)
