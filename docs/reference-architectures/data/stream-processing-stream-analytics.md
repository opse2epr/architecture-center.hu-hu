---
title: Az Azure Stream Analytics feldolgozási Stream
description: Hozzon létre egy teljes körű adatfolyam-feldolgozási folyamat az Azure-ban
author: MikeWasson
ms.date: 08/09/2018
ms.openlocfilehash: 82887bdd45f811ac733ead18c1f256098e575253
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/10/2018
ms.locfileid: "40025552"
---
# <a name="stream-processing-with-azure-stream-analytics"></a><span data-ttu-id="34e23-103">Az Azure Stream Analytics feldolgozási Stream</span><span class="sxs-lookup"><span data-stu-id="34e23-103">Stream processing with Azure Stream Analytics</span></span>

<span data-ttu-id="34e23-104">Ez a referenciaarchitektúra bemutatja egy teljes körű stream-feldolgozási folyamat.</span><span class="sxs-lookup"><span data-stu-id="34e23-104">This reference architecture shows an end-to-end stream processing pipeline.</span></span> <span data-ttu-id="34e23-105">A folyamat adatokat két forrásból fogadnak, utal. a két adatfolyam rekordokat, és kiszámítja a gördülő átlagát között egy olyan időkeretet.</span><span class="sxs-lookup"><span data-stu-id="34e23-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="34e23-106">Az eredmények tárolása további elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="34e23-106">The results are stored for further analysis.</span></span> [<span data-ttu-id="34e23-107">**A megoldás üzembe helyezése**.</span><span class="sxs-lookup"><span data-stu-id="34e23-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="34e23-108">**A forgatókönyv**:-i taxik vállalati minden taxi út adatokat gyűjt.</span><span class="sxs-lookup"><span data-stu-id="34e23-108">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="34e23-109">Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre.</span><span class="sxs-lookup"><span data-stu-id="34e23-109">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="34e23-110">A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság.</span><span class="sxs-lookup"><span data-stu-id="34e23-110">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="34e23-111">Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi.</span><span class="sxs-lookup"><span data-stu-id="34e23-111">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="34e23-112">A taxi vállalat úgy kívánja kiszámítja az átlagos tipp kiszolgálónként mérföld alapuló, valós idejű, sorrendben felhasználva azonosíthatja a trendeket.</span><span class="sxs-lookup"><span data-stu-id="34e23-112">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="34e23-113">Architektúra</span><span class="sxs-lookup"><span data-stu-id="34e23-113">Architecture</span></span>

<span data-ttu-id="34e23-114">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="34e23-114">The architecture consists of the following components.</span></span>

<span data-ttu-id="34e23-115">**Adatforrások**.</span><span class="sxs-lookup"><span data-stu-id="34e23-115">**Data sources**.</span></span> <span data-ttu-id="34e23-116">Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást.</span><span class="sxs-lookup"><span data-stu-id="34e23-116">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="34e23-117">Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="34e23-117">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="34e23-118">A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="34e23-118">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="34e23-119">Egy valós alkalmazás esetében az adatforrások lenne telepítve a taxi kabinetfájlok eszközök.</span><span class="sxs-lookup"><span data-stu-id="34e23-119">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="34e23-120">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="34e23-120">**Azure Event Hubs**.</span></span> <span data-ttu-id="34e23-121">[Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="34e23-121">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="34e23-122">Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ.</span><span class="sxs-lookup"><span data-stu-id="34e23-122">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="34e23-123">Minden adatforráshoz társított eseményközpont adatfolyamot küld.</span><span class="sxs-lookup"><span data-stu-id="34e23-123">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="34e23-124">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="34e23-124">**Azure Stream Analytics**.</span></span> <span data-ttu-id="34e23-125">[Stream Analytics](/azure/stream-analytics/) egy eseményfeldolgozó motor.</span><span class="sxs-lookup"><span data-stu-id="34e23-125">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="34e23-126">Stream Analytics-feladat beolvassa az adatokat a két event hubs szolgáltatástól érkező adatfolyamok és adatfolyam-feldolgozás hajt végre.</span><span class="sxs-lookup"><span data-stu-id="34e23-126">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="34e23-127">**A cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="34e23-127">**Cosmos DB**.</span></span> <span data-ttu-id="34e23-128">A Stream Analytics-feladat kimenete a rekordoknak, amelyek egy Cosmos DB dokumentum-adatbázis a JSON-dokumentumok formájában írt sorozata.</span><span class="sxs-lookup"><span data-stu-id="34e23-128">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="34e23-129">**A Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="34e23-129">**Microsoft Power BI**.</span></span> <span data-ttu-id="34e23-130">A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese.</span><span class="sxs-lookup"><span data-stu-id="34e23-130">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="34e23-131">Ebben az architektúrában az adatokat a Cosmos DB betölti.</span><span class="sxs-lookup"><span data-stu-id="34e23-131">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="34e23-132">Ez lehetővé teszi a felhasználók teljes készletének összegyűjtött előzményadatok elemzésére.</span><span class="sxs-lookup"><span data-stu-id="34e23-132">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="34e23-133">Az eredményeket közvetlenül a Stream Analytics a Power BI az adatok valós idejű nézet sikerült is folyamatos átvitelére.</span><span class="sxs-lookup"><span data-stu-id="34e23-133">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="34e23-134">További információkért lásd: [valós idejű streamelés a Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="34e23-134">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="34e23-135">**Az Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="34e23-135">**Azure Monitor**.</span></span> <span data-ttu-id="34e23-136">[Az Azure Monitor](/azure/monitoring-and-diagnostics/) teljesítmény-mérőszámokat a megoldásban üzembe helyezett Azure-szolgáltatásokkal gyűjti.</span><span class="sxs-lookup"><span data-stu-id="34e23-136">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="34e23-137">Ezek az irányítópult vizualizációjával betekintést nyerhet a megoldás állapotát.</span><span class="sxs-lookup"><span data-stu-id="34e23-137">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span> 

## <a name="data-ingestion"></a><span data-ttu-id="34e23-138">Adatfeldolgozás</span><span class="sxs-lookup"><span data-stu-id="34e23-138">Data ingestion</span></span>

<span data-ttu-id="34e23-139">Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="34e23-139">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="34e23-140">Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz egy 4 éven keresztül (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="34e23-140">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="34e23-141">Kétféle típusú rekordot tartalmaz: indításáról és diszkont adatait.</span><span class="sxs-lookup"><span data-stu-id="34e23-141">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="34e23-142">Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="34e23-142">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="34e23-143">Diszkont szerepel diszkont, adózási és tipp összegeket.</span><span class="sxs-lookup"><span data-stu-id="34e23-143">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="34e23-144">Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját.</span><span class="sxs-lookup"><span data-stu-id="34e23-144">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="34e23-145">Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot.</span><span class="sxs-lookup"><span data-stu-id="34e23-145">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="34e23-146">Az adatok CSV formátumban tárolódik.</span><span class="sxs-lookup"><span data-stu-id="34e23-146">The data is stored in CSV format.</span></span> 

<span data-ttu-id="34e23-147">Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="34e23-147">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="34e23-148">A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el.</span><span class="sxs-lookup"><span data-stu-id="34e23-148">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="34e23-149">Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat.</span><span class="sxs-lookup"><span data-stu-id="34e23-149">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="34e23-150">A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása.</span><span class="sxs-lookup"><span data-stu-id="34e23-150">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="34e23-151">Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat.</span><span class="sxs-lookup"><span data-stu-id="34e23-151">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="34e23-152">Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="34e23-152">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="34e23-153">Ez az adott esetben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl.</span><span class="sxs-lookup"><span data-stu-id="34e23-153">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="34e23-154">Ez lehetővé teszi a Stream Analytics párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal.</span><span class="sxs-lookup"><span data-stu-id="34e23-154">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="34e23-155">Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.</span><span class="sxs-lookup"><span data-stu-id="34e23-155">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="34e23-156">Az adatgenerátor a common data model mindkét bejegyzéstípusokat rendelkezik egy `PartitionKey` tulajdonság, amely az összefűzés, `Medallion`, `HackLicense`, és `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="34e23-156">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

```csharp
public abstract class TaxiData
{
    public TaxiData()
    {
    }

    [JsonProperty]
    public long Medallion { get; set; }

    [JsonProperty]
    public long HackLicense { get; set; }

    [JsonProperty]
    public string VendorId { get; set; }

    [JsonProperty]
    public DateTimeOffset PickupTime { get; set; }

    [JsonIgnore]
    public string PartitionKey
    {
        get => $"{Medallion}_{HackLicense}_{VendorId}";
    }
```

<span data-ttu-id="34e23-157">Ez a tulajdonság adja meg egy explicit partíciókulcsot az Event hubs szolgáltatásba küldésekor használható:</span><span class="sxs-lookup"><span data-stu-id="34e23-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="34e23-158">Stream-feldolgozás</span><span class="sxs-lookup"><span data-stu-id="34e23-158">Stream processing</span></span>

<span data-ttu-id="34e23-159">A stream-feldolgozási feladat van definiálva egy SQL-lekérdezés használata több különálló lépésre.</span><span class="sxs-lookup"><span data-stu-id="34e23-159">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="34e23-160">Az első két lépés egyszerűen válasszon rögzíti a két bemeneti streamekhez.</span><span class="sxs-lookup"><span data-stu-id="34e23-160">The first two steps simply select records from the two input streams.</span></span>

```sql
WITH
Step1 AS (
    SELECT PartitionId,
           TRY_CAST(Medallion AS nvarchar(max)) AS Medallion,
           TRY_CAST(HackLicense AS nvarchar(max)) AS HackLicense,
           VendorId,
           TRY_CAST(PickupTime AS datetime) AS PickupTime,
           TripDistanceInMiles
    FROM [TaxiRide] PARTITION BY PartitionId
),
Step2 AS (
    SELECT PartitionId,
           medallion AS Medallion,
           hack_license AS HackLicense,
           vendor_id AS VendorId,
           TRY_CAST(pickup_datetime AS datetime) AS PickupTime,
           tip_amount AS TipAmount
    FROM [TaxiFare] PARTITION BY PartitionId
),
```

<span data-ttu-id="34e23-161">A következő lépésben válassza ki a megfelelő rekordokat az egyes adatfolyamokkal két bemeneti streamekhez csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="34e23-161">The next step joins the two input streams to select matching records from each stream.</span></span>

```sql
Step3 AS (
  SELECT
         tr.Medallion,
         tr.HackLicense,
         tr.VendorId,
         tr.PickupTime,
         tr.TripDistanceInMiles,
         tf.TipAmount
    FROM [Step1] tr
    PARTITION BY PartitionId
    JOIN [Step2] tf PARTITION BY PartitionId
      ON tr.Medallion = tf.Medallion
     AND tr.HackLicense = tf.HackLicense
     AND tr.VendorId = tf.VendorId
     AND tr.PickupTime = tf.PickupTime
     AND tr.PartitionId = tf.PartitionId
     AND DATEDIFF(minute, tr, tf) BETWEEN 0 AND 15
)
```

<span data-ttu-id="34e23-162">Ez a lekérdezés illeszt rekordokat konfigurálásában a megfelelő rekordokat (Medallion, HackLicense, VendorId és PickupTime) egyedi azonosítására alkalmas mezőket.</span><span class="sxs-lookup"><span data-stu-id="34e23-162">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="34e23-163">A `JOIN` utasítást is tartalmaz a partícióazonosító.</span><span class="sxs-lookup"><span data-stu-id="34e23-163">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="34e23-164">Ahogy említettük, ez kihasználja a tény, hogy megfelelő rekordokat mindig rendelkezzenek a partíció ugyanazzal az Azonosítóval ebben a forgatókönyvben.</span><span class="sxs-lookup"><span data-stu-id="34e23-164">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="34e23-165">A Stream Analytics, illesztések vannak *historikus*, azaz a rekordok tartományhoz csatlakoztatott idő adott időtartamon belül.</span><span class="sxs-lookup"><span data-stu-id="34e23-165">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="34e23-166">Ellenkező esetben a feladat lehet, hogy várnia kell, határozatlan ideig az egyezést.</span><span class="sxs-lookup"><span data-stu-id="34e23-166">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="34e23-167">A [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) függvény meghatározza, hogy milyen két egyező rekordot is választhatók el egymástól, időben egyezést.</span><span class="sxs-lookup"><span data-stu-id="34e23-167">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span> 

<span data-ttu-id="34e23-168">A feladat utolsó lépésében kiszámítja az átlagos tipp egy lépést, 5 perces ugróablak szerint csoportosítva.</span><span class="sxs-lookup"><span data-stu-id="34e23-168">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="34e23-169">Stream Analytics biztosít több [ablakkezelési függvényeket](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="34e23-169">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="34e23-170">Egy ugróablak helyezi előre a idő által meghatározott időre ebben az esetben ugrásonkénti 1 perc.</span><span class="sxs-lookup"><span data-stu-id="34e23-170">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="34e23-171">Ez a mozgó átlag kiszámításához az elmúlt 5 percben.</span><span class="sxs-lookup"><span data-stu-id="34e23-171">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="34e23-172">Az itt látható architektúrában csak a Stream Analytics-feladat eredményeinek menti, a Cosmos DB-hez.</span><span class="sxs-lookup"><span data-stu-id="34e23-172">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="34e23-173">A big Data típusú adatok esetben fontolja meg is [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) menteni a nyers eseményadatok Azure Blob storage-bA.</span><span class="sxs-lookup"><span data-stu-id="34e23-173">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="34e23-174">A nyers adatok megőrzése mellett lehetővé teszi, hogy a batch lekérdezéseket futtathat az előzményadatok későbbi időpontban, annak érdekében, hogy új hozzájut a adatokból.</span><span class="sxs-lookup"><span data-stu-id="34e23-174">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="34e23-175">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="34e23-175">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="34e23-176">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="34e23-176">Event Hubs</span></span>

<span data-ttu-id="34e23-177">Az Event Hubs átviteli kapacitásának mérése [átviteli egységek](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="34e23-177">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="34e23-178">Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről](/azure/event-hubs/event-hubs-auto-inflate), amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.</span><span class="sxs-lookup"><span data-stu-id="34e23-178">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

### <a name="stream-analytics"></a><span data-ttu-id="34e23-179">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="34e23-179">Stream Analytics</span></span>

<span data-ttu-id="34e23-180">A Stream Analytics a számítási erőforrások hozzárendelt feladathoz a folyamatos átviteli egységek mérése történik.</span><span class="sxs-lookup"><span data-stu-id="34e23-180">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="34e23-181">Stream Analytics-feladatok méretezése legjobb, ha a feladat is méretezésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="34e23-181">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="34e23-182">Ezzel a módszerrel Stream Analytics juttathatja el a feladatot több számítási csomóponton.</span><span class="sxs-lookup"><span data-stu-id="34e23-182">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="34e23-183">Az Event Hubs-bemenet, használja a `PARTITION BY` kulcsszó a Stream Analytics-feladat particionálásához.</span><span class="sxs-lookup"><span data-stu-id="34e23-183">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="34e23-184">Az adatok eloszlása az Event Hubs-partíciók alapján részekre.</span><span class="sxs-lookup"><span data-stu-id="34e23-184">The data will be divided into subsets based on the Event Hubs partitions.</span></span> 

<span data-ttu-id="34e23-185">Ablakkezelési függvényekről és társításokról, historikus szükséges további SU.</span><span class="sxs-lookup"><span data-stu-id="34e23-185">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="34e23-186">Amikor lehetséges, használjon `PARTITION BY` úgy, hogy az egyes partíciók külön dolgozza fel.</span><span class="sxs-lookup"><span data-stu-id="34e23-186">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="34e23-187">További információkért lásd: [ismertetése és módosítása a folyamatos átviteli egységek](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="34e23-187">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="34e23-188">Ha nem lehetséges a teljes Stream Analytics-feladat párhuzamosíthatja, próbálja meg a feladat felosztása több lépést, egy vagy több párhuzamos lépéseket tartalmazó indítása.</span><span class="sxs-lookup"><span data-stu-id="34e23-188">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="34e23-189">Ezzel a módszerrel az első lépéseket is futtatható egyszerre.</span><span class="sxs-lookup"><span data-stu-id="34e23-189">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="34e23-190">Ez a referenciaarchitektúra: például a</span><span class="sxs-lookup"><span data-stu-id="34e23-190">For example, in this reference architecture:</span></span>

- <span data-ttu-id="34e23-191">1. és 2 egyszerűek `SELECT` utasításokat, amelyek ugyanazon a partíción belül időtartományban.</span><span class="sxs-lookup"><span data-stu-id="34e23-191">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span> 
- <span data-ttu-id="34e23-192">3. lépés a particionált csatlakozzon két bemeneti streamekhez keresztül hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="34e23-192">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="34e23-193">Ebben a lépésben kihasználja a tény, hogy a megfelelő rekordokat megosztani ugyanazzal a partíciókulccsal, és így garantáltan minden egyes bemeneti Stream partíció ugyanazzal az Azonosítóval rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="34e23-193">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="34e23-194">4. lépés összesítések összes partíciót.</span><span class="sxs-lookup"><span data-stu-id="34e23-194">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="34e23-195">Ez a lépés nem méretezésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="34e23-195">This step cannot be parallelized.</span></span>

<span data-ttu-id="34e23-196">A Stream Analytics használata [feladatábra](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) hány partíciók rendelt minden lépése a feladat megtekintéséhez.</span><span class="sxs-lookup"><span data-stu-id="34e23-196">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="34e23-197">Az alábbi ábrán ez a referenciaarchitektúra a feladat diagramja:</span><span class="sxs-lookup"><span data-stu-id="34e23-197">The following diagram shows the job diagram for this reference architecture:</span></span>

![](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="34e23-198">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="34e23-198">Cosmos DB</span></span>

<span data-ttu-id="34e23-199">A Cosmos DB átviteli kapacitás mérése [kérelemegység](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="34e23-199">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="34e23-200">Annak érdekében, hogy az elmúlt 10 000-et egy Cosmos DB-tárolók skálázása RU, meg kell adnia egy [partíciókulcs](/azure/cosmos-db/partition-data) amikor létrehozza a tárolót, és a partíciókulcs tartalmazza minden egyes dokumentum.</span><span class="sxs-lookup"><span data-stu-id="34e23-200">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span> 

<span data-ttu-id="34e23-201">Ez a referenciaarchitektúra az új dokumentumok létrehozásának csak egyszer perc (a szórási ablak időköz), így az átviteli sebességet megkövetelő meglehetősen kicsi.</span><span class="sxs-lookup"><span data-stu-id="34e23-201">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="34e23-202">Éppen ezért hiba esetén nem kell hozzárendelni az ebben a forgatókönyvben egy partíciókulcsot.</span><span class="sxs-lookup"><span data-stu-id="34e23-202">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="34e23-203">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="34e23-203">Monitoring considerations</span></span>

<span data-ttu-id="34e23-204">Az adatfolyam-feldolgozó megoldást fontos a teljesítmény és a rendszer állapotának figyelésére.</span><span class="sxs-lookup"><span data-stu-id="34e23-204">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="34e23-205">[Az Azure Monitor](/azure/monitoring-and-diagnostics/) az architektúrában használt Azure-szolgáltatások a metrikák és diagnosztikai naplókat gyűjti össze.</span><span class="sxs-lookup"><span data-stu-id="34e23-205">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="34e23-206">Az Azure Monitor az Azure platform be van építve, és nincs szükség semmilyen további programkódokat kellene megtervezni az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="34e23-206">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="34e23-207">A következő figyelmeztető jeleket bármelyikét jelzi, hogy horizontális felskálázást a megfelelő Azure-erőforrás:</span><span class="sxs-lookup"><span data-stu-id="34e23-207">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="34e23-208">Az Event Hubs szabályozza a kéréseket, vagy a napi kvóta közelében van.</span><span class="sxs-lookup"><span data-stu-id="34e23-208">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="34e23-209">A Stream Analytics-feladat konzisztens módon több mint 80 %-át a lefoglalt folyamatos átviteli egységek (su-t) használ.</span><span class="sxs-lookup"><span data-stu-id="34e23-209">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="34e23-210">A cosmos DB megkezdi a kérések szabályozása.</span><span class="sxs-lookup"><span data-stu-id="34e23-210">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="34e23-211">A referenciaarchitektúra egy egyéni irányítópult, amelyen telepítve van az Azure Portalon is tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="34e23-211">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="34e23-212">Után az architektúra üzembe helyezéséhez is megtekintheti az irányítópulton nyissa meg a [az Azure Portal](https://portal.azure.com) és kiválasztásával `TaxiRidesDashboard` az irányítópultok listájából.</span><span class="sxs-lookup"><span data-stu-id="34e23-212">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="34e23-213">Az Azure Portalon egyéni irányítópultok létrehozása és telepítése kapcsolatos további információkért lásd: [programozott módon létrehozhat irányítópultokat Azure](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="34e23-213">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="34e23-214">Az alábbi képen az irányítópult látható, miután a Stream Analytics-feladat futásának időtartama: körülbelül egy óra alatt.</span><span class="sxs-lookup"><span data-stu-id="34e23-214">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="34e23-215">A panel bal alsó mutatja, hogy a SU-használat a Stream Analytics-feladat az első 15 perc során megnő, és ezután itt megáll.</span><span class="sxs-lookup"><span data-stu-id="34e23-215">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="34e23-216">Ez a lehetőség egy tipikus mintája a feladat eléri egy stabil állapotot.</span><span class="sxs-lookup"><span data-stu-id="34e23-216">This is a typical pattern as the job reaches a steady state.</span></span> 

<span data-ttu-id="34e23-217">Figyelje meg, hogy az Event Hubs szabályozza a kéréseket, a felső jobb oldali panelen látható.</span><span class="sxs-lookup"><span data-stu-id="34e23-217">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="34e23-218">Alkalmanként szabályozott kérelmet nem áll egy probléma, mivel az Event hubs szolgáltatás ügyfél-SDK automatikusan újrapróbálkozik, ha sávszélesség-szabályozási hibaüzenet kap.</span><span class="sxs-lookup"><span data-stu-id="34e23-218">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="34e23-219">Azonban ha egységes szabályozási hibákat lát, az azt jelenti az event hubs átviteli egységeket kell.</span><span class="sxs-lookup"><span data-stu-id="34e23-219">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="34e23-220">A következő diagram is mutatja egy tesztfuttatás, az Event Hubs használatával az automatikus feltöltési funkció, amely automatikusan méretezhető az átviteli egységek igény szerint.</span><span class="sxs-lookup"><span data-stu-id="34e23-220">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span> 

![](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="34e23-221">Az automatikus feltöltési engedélyezve lett 06:35 kapcsolatos megjelölni.</span><span class="sxs-lookup"><span data-stu-id="34e23-221">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="34e23-222">A p, dobja el a szabályozott kérelmeinek száma, az Event Hubs automatikusan átméreteződik legfeljebb 3 átviteli egységek láthatja.</span><span class="sxs-lookup"><span data-stu-id="34e23-222">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="34e23-223">Interestingly ennek hatására oldal, ahol egyre növekszik a Stream Analytics-feladat a SU kihasználtságát.</span><span class="sxs-lookup"><span data-stu-id="34e23-223">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="34e23-224">Szabályozás, az Event Hubs mesterségesen lett csökkentve a feldolgozási sebességét a Stream Analytics-feladat.</span><span class="sxs-lookup"><span data-stu-id="34e23-224">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="34e23-225">Valójában szokás, hogy egy teljesítménybeli szűk keresztmetszetet feloldása tárja fel egy másik.</span><span class="sxs-lookup"><span data-stu-id="34e23-225">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="34e23-226">Ebben az esetben további SU lefoglalása a Stream Analytics-feladat megoldotta a problémát.</span><span class="sxs-lookup"><span data-stu-id="34e23-226">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="34e23-227">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="34e23-227">Deploy the solution</span></span>

<span data-ttu-id="34e23-228">Ez a referenciaarchitektúra egy üzemelő példánya érhető el az [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span><span class="sxs-lookup"><span data-stu-id="34e23-228">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="34e23-229">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="34e23-229">Prerequisites</span></span>

1. <span data-ttu-id="34e23-230">Klónozza, ágaztassa vagy töltse le a zip-fájlját a [referenciaarchitektúrákat](https://github.com/mspnp/reference-architectures) GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="34e23-230">Clone, fork, or download the zip file for the [reference architectures](https://github.com/mspnp/reference-architectures) GitHub repository.</span></span>

2. <span data-ttu-id="34e23-231">Telepítés [Docker](https://www.docker.com/) futtatásához az adatgenerálást.</span><span class="sxs-lookup"><span data-stu-id="34e23-231">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="34e23-232">Telepítés [az Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="34e23-232">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="34e23-233">Parancsot a parancssorba bash-parancssorból vagy PowerShell-parancssorból, jelentkezzen be Azure-fiókjába a következő:</span><span class="sxs-lookup"><span data-stu-id="34e23-233">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

    ```
    az login
    ```

### <a name="download-the-source-data-files"></a><span data-ttu-id="34e23-234">Az adatok forrásfájlok letöltéséhez</span><span class="sxs-lookup"><span data-stu-id="34e23-234">Download the source data files</span></span>

1. <span data-ttu-id="34e23-235">Hozzon létre egy könyvtárat nevű `DataFile` alatt a `data/streaming_asa` könyvtárat a GitHub-adattárat a.</span><span class="sxs-lookup"><span data-stu-id="34e23-235">Create a directory named `DataFile` under the `data/streaming_asa` directory in the GitHub repo.</span></span>

2. <span data-ttu-id="34e23-236">Nyisson meg egy webböngészőt, és navigáljon a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="34e23-236">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="34e23-237">Kattintson a **letöltése** gombra ezen az oldalon a megfelelő évnek a taxi-adatok egy zip-fájl letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="34e23-237">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="34e23-238">Bontsa ki a zip-fájlt a `DataFile` könyvtár.</span><span class="sxs-lookup"><span data-stu-id="34e23-238">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="34e23-239">Ez a tömörített fájl más zip-fájlokat tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="34e23-239">This zip file contains other zip files.</span></span> <span data-ttu-id="34e23-240">A gyermek zip-fájlokat nem csomagolja ki.</span><span class="sxs-lookup"><span data-stu-id="34e23-240">Don't extract the child zip files.</span></span>

<span data-ttu-id="34e23-241">A directory-struktúra az alábbiakhoz hasonlóan kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="34e23-241">The directory structure should look like the following:</span></span>

```
/data
    /streaming_asa
        /DataFile
            /FOIL2013
                trip_data_1.zip
                trip_data_2.zip
                trip_data_3.zip
                ...
```

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="34e23-242">Az Azure-erőforrások üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="34e23-242">Deploy the Azure resources</span></span>

1. <span data-ttu-id="34e23-243">A rendszerhéj vagy a Windows-parancssort futtassa a következő parancsot, és kövesse a bejelentkezési kérések jelenhetnek:</span><span class="sxs-lookup"><span data-stu-id="34e23-243">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="34e23-244">Lépjen abba a mappába `data/streaming_asa` a GitHub-adattárban</span><span class="sxs-lookup"><span data-stu-id="34e23-244">Navigate to the folder `data/streaming_asa` in the GitHub repository</span></span>

    ```bash
    cd data/streaming_asa
    ```

2. <span data-ttu-id="34e23-245">Futtassa az alábbi parancsokat az Azure-erőforrások üzembe helyezéséhez:</span><span class="sxs-lookup"><span data-stu-id="34e23-245">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Location]'
    export cosmosDatabaseAccount='[Cosmos DB account name]'
    export cosmosDatabase='[Cosmod DB database name]'
    export cosmosDataBaseCollection='[Cosmos DB collection name]'
    export eventHubNamespace='[Event Hubs namespace name]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
      --template-file ./azure/deployresources.json --parameters \
      eventHubNamespace=$eventHubNamespace \
      outputCosmosDatabaseAccount=$cosmosDatabaseAccount \
      outputCosmosDatabase=$cosmosDatabase \
      outputCosmosDatabaseCollection=$cosmosDataBaseCollection

    # Create a database 
    az cosmosdb database create --name $cosmosDatabaseAccount \
        --db-name $cosmosDatabase --resource-group $resourceGroup

    # Create a collection
    az cosmosdb collection create --collection-name $cosmosDataBaseCollection \
        --name $cosmosDatabaseAccount --db-name $cosmosDatabase \
        --resource-group $resourceGroup
    ```

3. <span data-ttu-id="34e23-246">Az Azure Portalon keresse meg a létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="34e23-246">In the Azure portal, navigate to the resource group that was created.</span></span>

4. <span data-ttu-id="34e23-247">A Stream Analytics-feladat panel megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="34e23-247">Open the blade for the Stream Analytics job.</span></span>

5. <span data-ttu-id="34e23-248">Kattintson a **Start** elindítani a feladatot.</span><span class="sxs-lookup"><span data-stu-id="34e23-248">Click **Start** to start the job.</span></span> <span data-ttu-id="34e23-249">Válassza ki **most** kimenetként, kezdő időpont.</span><span class="sxs-lookup"><span data-stu-id="34e23-249">Select **Now** as the output start time.</span></span> <span data-ttu-id="34e23-250">Várjon, amíg a feladat elindításához.</span><span class="sxs-lookup"><span data-stu-id="34e23-250">Wait for the job to start.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="34e23-251">Futtassa az adatgenerátor</span><span class="sxs-lookup"><span data-stu-id="34e23-251">Run the data generator</span></span>

1. <span data-ttu-id="34e23-252">Az Event Hub-kapcsolati karakterláncok beolvasása.</span><span class="sxs-lookup"><span data-stu-id="34e23-252">Get the Event Hub connection strings.</span></span> <span data-ttu-id="34e23-253">Ezek az Azure Portalról, vagy a következő CLI-parancsok futtatásával kaphat:</span><span class="sxs-lookup"><span data-stu-id="34e23-253">You can get these from the Azure portal, or by running the following CLI commands:</span></span>

    ```bash
    # RIDE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-ride \
        --name taxi-ride-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString

    # FARE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-fare \
        --name taxi-fare-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString
    ```

2. <span data-ttu-id="34e23-254">Lépjen abba a könyvtárba `data/streaming_asa/onprem` a GitHub-adattárban</span><span class="sxs-lookup"><span data-stu-id="34e23-254">Navigate to the directory `data/streaming_asa/onprem` in the GitHub repository</span></span>

3. <span data-ttu-id="34e23-255">Frissítse az értékeket a fájlban `main.env` módon:</span><span class="sxs-lookup"><span data-stu-id="34e23-255">Update the values in the file `main.env` as follows:</span></span>

    ```
    RIDE_EVENT_HUB=[Connection string for taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```

4. <span data-ttu-id="34e23-256">Futtassa a következő parancsot a Docker-rendszerkép létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="34e23-256">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

5. <span data-ttu-id="34e23-257">Lépjen vissza a szülőkönyvtárban `data/stream_asa`.</span><span class="sxs-lookup"><span data-stu-id="34e23-257">Navigate back to the parent directory, `data/stream_asa`.</span></span>

    ```bash
    cd ..
    ```

6. <span data-ttu-id="34e23-258">Futtassa a következő parancsot beírva futtassa a Docker-rendszerképet.</span><span class="sxs-lookup"><span data-stu-id="34e23-258">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="34e23-259">A kimenet a következőhöz hasonlóan kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="34e23-259">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="34e23-260">Lehetővé teszik a program futtatása legalább 5 percnek, azaz a meghatározott a Stream Analytics lekérdezési ablak.</span><span class="sxs-lookup"><span data-stu-id="34e23-260">Let the program run for at least 5 minutes, which is the window defined in the Stream Analytics query.</span></span> <span data-ttu-id="34e23-261">A Stream Analytics-feladat megfelelően fut-e, nyissa meg az Azure Portalt, és keresse meg a Cosmos DB-adatbázist.</span><span class="sxs-lookup"><span data-stu-id="34e23-261">To verify the Stream Analytics job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="34e23-262">Nyissa meg a **adatkezelő** panel és a dokumentumok megtekintése.</span><span class="sxs-lookup"><span data-stu-id="34e23-262">Open the **Data Explorer** blade and view the documents.</span></span> 

<span data-ttu-id="34e23-263">[1] <span id="note1">Donovan, Brian; Munka, Dan (2016): New York City Taxi Útadatok (2010, 2013).</span><span class="sxs-lookup"><span data-stu-id="34e23-263">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="34e23-264">Egyetemi Illinois, Urbana-Champaignben.</span><span class="sxs-lookup"><span data-stu-id="34e23-264">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="34e23-265">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="34e23-265">https://doi.org/10.13012/J8PN93H8</span></span>
