---
title: Foglalt adatbázissal kapcsolatos kizárási minták
titleSuffix: Performance antipatterns for cloud apps
description: A feldolgozás adatbázis-kiszolgálóra való kiszervezése teljesítménybeli és skálázhatósági problémákat okozhat.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="busy-database-antipattern"></a><span data-ttu-id="76099-103">Foglalt adatbázissal kapcsolatos kizárási minták</span><span class="sxs-lookup"><span data-stu-id="76099-103">Busy Database antipattern</span></span>

<span data-ttu-id="76099-104">A feldolgozás adatbázis-kiszolgálóra való kiszervezése azt okozhatja, hogy jelentős mennyiségű időt kell kód futtatására fordítani az adatok tárolására és lekérésére vonatkozó kérések megválaszolása helyett.</span><span class="sxs-lookup"><span data-stu-id="76099-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="76099-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="76099-105">Problem description</span></span>

<span data-ttu-id="76099-106">Számos adatbázisrendszer képes kódok futtatására.</span><span class="sxs-lookup"><span data-stu-id="76099-106">Many database systems can run code.</span></span> <span data-ttu-id="76099-107">Ilyenek például a tárolt eljárások és az eseményindítók.</span><span class="sxs-lookup"><span data-stu-id="76099-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="76099-108">Gyakran hatékonyabb ezt a feldolgozást az adatokhoz közel elvégezni ahelyett, hogy az adatokat egy ügyfélalkalmazásba továbbítanánk feldolgozás céljából.</span><span class="sxs-lookup"><span data-stu-id="76099-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="76099-109">Ezen funkciók túlzott mértékű használata azonban csökkentheti a teljesítményt több okból is:</span><span class="sxs-lookup"><span data-stu-id="76099-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="76099-110">Előfordulhat, hogy az adatbázis túl sok időt tölt a feldolgozással az új ügyfélkérések elfogadása és az adatok lekérése helyett.</span><span class="sxs-lookup"><span data-stu-id="76099-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="76099-111">Az adatbázisok általában megosztott erőforrások, így magas kihasználtság esetén szűk keresztmetszet jöhet létre.</span><span class="sxs-lookup"><span data-stu-id="76099-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="76099-112">Forgalmi díjas adattárak esetén a futtatókörnyezet költségei jelentősen megnövekedhetnek.</span><span class="sxs-lookup"><span data-stu-id="76099-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="76099-113">Ez különösen igaz a felügyelt adatbázisokra.</span><span class="sxs-lookup"><span data-stu-id="76099-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="76099-114">Például az Azure SQL Database [Database Transaction Unit][dtu] (DTU) egységek alapján számláz.</span><span class="sxs-lookup"><span data-stu-id="76099-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="76099-115">Az adatbázisok vertikális felskálázási kapacitása véges, a horizontális felskálázás pedig nem egyértelmű.</span><span class="sxs-lookup"><span data-stu-id="76099-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="76099-116">Emiatt érdemes lehet a feldolgozást áthelyezni egy olyan számítási erőforrásra, amelyet könnyű horizontálisan felskálázni, például egy virtuális gépre vagy egy App Service-alkalmazásra.</span><span class="sxs-lookup"><span data-stu-id="76099-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="76099-117">A kizárási minta jellemzően az alábbi okokból következhet be:</span><span class="sxs-lookup"><span data-stu-id="76099-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="76099-118">Az adatbázis szolgáltatásnak minősül, nem adattárnak.</span><span class="sxs-lookup"><span data-stu-id="76099-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="76099-119">Előfordulhat, hogy egy alkalmazás az adatbázist használja adatok formázásához (például XML formátumúra konvertálásához), sztringadatok módosításához vagy összetett számítások végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="76099-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="76099-120">A fejlesztők olyan lekérdezéseket próbálnak írni, amelyek eredményei közvetlenül a felhasználók számára jelennek meg.</span><span class="sxs-lookup"><span data-stu-id="76099-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="76099-121">Például lehet, hogy egy lekérdezés mezőket kombinál, vagy területi beállítás szerint formáz dátumokat, időpontokat és pénznemet.</span><span class="sxs-lookup"><span data-stu-id="76099-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="76099-122">A fejlesztők megpróbálják kijavítani a [Felesleges beolvasások][ExtraneousFetching] kizárási mintát számítások az adatbázis számára való elküldésével.</span><span class="sxs-lookup"><span data-stu-id="76099-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="76099-123">A tárolt eljárásokat tárolják az üzleti logikát, talán azért, mert könnyebb őket karbantartani és frissíteni.</span><span class="sxs-lookup"><span data-stu-id="76099-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="76099-124">A következő példa lekérdezi a 20 legértékesebb rendelést egy megadott értékesítési területre vonatkozóan, és az eredményeket XML-fájlként formázza.</span><span class="sxs-lookup"><span data-stu-id="76099-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="76099-125">A példa Transact-SQL függvényeket használ az adatok elemzéséhez és az eredmények XML formátumúra konvertálásához.</span><span class="sxs-lookup"><span data-stu-id="76099-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="76099-126">A teljes kódmintát [itt][sample-app] találja.</span><span class="sxs-lookup"><span data-stu-id="76099-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="76099-127">Természetesen ez egy összetett lekérdezés.</span><span class="sxs-lookup"><span data-stu-id="76099-127">Clearly, this is complex query.</span></span> <span data-ttu-id="76099-128">Ahogyan azt a későbbiekben látni fogjuk, a lekérdezés jelentős mértékű feldolgozási erőforrást használ az adatbázis-kiszolgálón.</span><span class="sxs-lookup"><span data-stu-id="76099-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="76099-129">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="76099-129">How to fix the problem</span></span>

<span data-ttu-id="76099-130">Helyezze át a feldolgozást az adatbázis-kiszolgálóról más alkalmazásrétegekbe.</span><span class="sxs-lookup"><span data-stu-id="76099-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="76099-131">Ideális esetben érdemes az adatbázist arra korlátozni, hogy adatelérési műveleteket végezzen azon képességekkel, amelyekre optimalizálva lett, ilyen például az RDBMS-be való aggregáció.</span><span class="sxs-lookup"><span data-stu-id="76099-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="76099-132">Például az előző Transact-SQL kód lecserélhető egy utasításra, amely egyszerűen lekéri a feldolgozandó adatokat.</span><span class="sxs-lookup"><span data-stu-id="76099-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="76099-133">Az alkalmazás ezután a .NET-keretrendszerbeli `System.Xml.Linq` API-k használatával formázza az eredményeket XML-ként.</span><span class="sxs-lookup"><span data-stu-id="76099-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="76099-134">Ez a kód viszonylag összetett.</span><span class="sxs-lookup"><span data-stu-id="76099-134">This code is somewhat complex.</span></span> <span data-ttu-id="76099-135">Új alkalmazáshoz érdemes lehet egy szerializációs kódtárat használni.</span><span class="sxs-lookup"><span data-stu-id="76099-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="76099-136">Azonban itt azt a feltételezzük, hogy a fejlesztőcsapat egy meglévő alkalmazást tervez újra, így a metódusnak az eredeti kóddal megegyező formátumban kell visszaadnia az eredményeket.</span><span class="sxs-lookup"><span data-stu-id="76099-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="76099-137">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="76099-137">Considerations</span></span>

- <span data-ttu-id="76099-138">Számos adatbázisrendszer nagy mértékben optimalizálva van bizonyos típusú adatfeldolgozások elvégzésére, például nagy méretű adatkészletekből aggregált értékek kiszámítására.</span><span class="sxs-lookup"><span data-stu-id="76099-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="76099-139">Ezeket a feldolgozástípusokat ne helyezze át máshová az adatbázisból.</span><span class="sxs-lookup"><span data-stu-id="76099-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="76099-140">Ne helyezze át a feldolgozási folyamatokat, ha emiatt az adatbázis hálózati adatátvitele nagy mértékben megnövekszik.</span><span class="sxs-lookup"><span data-stu-id="76099-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="76099-141">Tekintse meg a [Felesleges beolvasások kizárási mintát][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="76099-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="76099-142">Ha a feldolgozást egy alkalmazásrétegbe helyezi át, akkor lehet, hogy azt a réteget horizontálisan fel kell skálázni, hogy kezelni tudja a további munkamennyiséget.</span><span class="sxs-lookup"><span data-stu-id="76099-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="76099-143">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="76099-143">How to detect the problem</span></span>

<span data-ttu-id="76099-144">Foglalt adatbázist jelez többek között az adatbázist elérő műveletek átviteli sebességének és válaszidejének aránytalan romlása.</span><span class="sxs-lookup"><span data-stu-id="76099-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span>

<span data-ttu-id="76099-145">A következő lépések végrehajtásával azonosíthatja a problémát:</span><span class="sxs-lookup"><span data-stu-id="76099-145">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="76099-146">A teljesítménymonitorozás révén határozza meg, hogy az éles rendszer mennyi időt fordít az adatbázis-tevékenységek elvégzésére.</span><span class="sxs-lookup"><span data-stu-id="76099-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="76099-147">Vizsgálja meg az adatbázis által ezen idő alatt elvégzett feladatokat.</span><span class="sxs-lookup"><span data-stu-id="76099-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="76099-148">Ha arra gyanakszik, hogy bizonyos műveletek túl sok adatbázis-tevékenységet okoznak, végezzen terhelési teszteket ellenőrzött környezetben.</span><span class="sxs-lookup"><span data-stu-id="76099-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="76099-149">Minden tesztben futtasson gyanús műveleteket változó felhasználói terheléssel.</span><span class="sxs-lookup"><span data-stu-id="76099-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="76099-150">A terhelési tesztekből származó telemetria megvizsgálásával elemezze az adatbázis használatát.</span><span class="sxs-lookup"><span data-stu-id="76099-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="76099-151">Ha az adatbázis-tevékenység jelentős mennyiségű adatfeldolgozást, de kevés adatforgalmat jelez, tekintse át a forráskódot, és döntse el, hogy nem lenne-e jobb a feldolgozást máshol végezni.</span><span class="sxs-lookup"><span data-stu-id="76099-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="76099-152">Ha az adatbázis-tevékenység mennyisége alacsony, vagy a válaszidők viszonylag rövidek, akkor valószínűleg nincs foglalt adatbázis típusú teljesítményprobléma.</span><span class="sxs-lookup"><span data-stu-id="76099-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="76099-153">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="76099-153">Example diagnosis</span></span>

<span data-ttu-id="76099-154">Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.</span><span class="sxs-lookup"><span data-stu-id="76099-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="76099-155">Az adatbázis-tevékenység mértékének monitorozása</span><span class="sxs-lookup"><span data-stu-id="76099-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="76099-156">A következő diagram bemutatja egy mintaalkalmazás terhelési tesztjének eredményeit. A teszt lépéses terhelést használt legfeljebb 50 egyidejű felhasználóval.</span><span class="sxs-lookup"><span data-stu-id="76099-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="76099-157">A kérések mennyisége hamar eléri a korlátot, és azon a szinten marad, miközben az átlagos válaszidő folyamatosan nő.</span><span class="sxs-lookup"><span data-stu-id="76099-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="76099-158">A két mérőszám logaritmikus skálán jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="76099-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![Az adatbázisban végzett feldolgozásra vonatkozó terhelési tesztek eredményei][ProcessingInDatabaseLoadTest]

<span data-ttu-id="76099-160">A következő grafikon a processzorkihasználtságot és a DTU-k számát mutatja a szolgáltatási kvóta százalékában.</span><span class="sxs-lookup"><span data-stu-id="76099-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="76099-161">A DTU-kkal mérhető, hogy az adatbázis mennyi feldolgozást végez.</span><span class="sxs-lookup"><span data-stu-id="76099-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="76099-162">A grafikonon látható, hogy a processzor és a DTU-k kihasználtsága egyaránt gyorsan elérte a 100%-ot.</span><span class="sxs-lookup"><span data-stu-id="76099-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Azure SQL Database-figyelő, amely feldolgozás közben az adatbázis teljesítményét mutatja][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="76099-164">Az adatbázis által elvégzett feladatok vizsgálata</span><span class="sxs-lookup"><span data-stu-id="76099-164">Examine the work performed by the database</span></span>

<span data-ttu-id="76099-165">Előfordulhat, hogy az adatbázis valódi adathozzáférési műveleteket végez, nem feldolgozást, ezért fontos megismerni az adatbázis foglalt állapota mellett futó SQL-utasításokat.</span><span class="sxs-lookup"><span data-stu-id="76099-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="76099-166">A rendszer monitorozásával rögzítse az SQL forgalmi adatokat, és vesse össze az SQL-műveleteket az alkalmazáskérésekkel.</span><span class="sxs-lookup"><span data-stu-id="76099-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="76099-167">Ha az adatbázis műveletei tisztán adathozzáférési műveletek nagy mennyiségű feldolgozás nélkül, akkor a probléma okai [felesleges beolvasások][ExtraneousFetching] lehetnek.</span><span class="sxs-lookup"><span data-stu-id="76099-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="76099-168">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="76099-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="76099-169">A következő grafikonon egy, a frissített kóddal végzett terhelési teszt látható.</span><span class="sxs-lookup"><span data-stu-id="76099-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="76099-170">Az átviteli sebesség jóval magasabb, másodpercenként több mint 400 kérés a korábbi 12-vel szemben.</span><span class="sxs-lookup"><span data-stu-id="76099-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="76099-171">Az átlagos válaszidő szintén sokkal alacsonyabb, alig több mint 0,1 másodperc a korábbi 4 másodperchez képest.</span><span class="sxs-lookup"><span data-stu-id="76099-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![Az adatbázisban végzett feldolgozásra vonatkozó terhelési tesztek eredményei][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="76099-173">A processzor- és DTU-kihasználtság azt mutatja, hogy a rendszer hosszabb idő alatt érte el a telítettséget a megnövelt adatátviteli sebesség ellenére is.</span><span class="sxs-lookup"><span data-stu-id="76099-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![Azure SQL Database-figyelő, amely az adatbázis teljesítményét mutatja, miközben feldolgozást végez az ügyfélalkalmazásban][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="76099-175">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="76099-175">Related resources</span></span>

- <span data-ttu-id="76099-176">[Felesleges beolvasások– kizárási minta][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="76099-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>

[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
