---
title: Streamek feldolgozása az Azure Stream Analyticsszel
titleSuffix: Azure Reference Architectures
description: Hozzon létre egy teljes körű adatfolyam-feldolgozási folyamat az Azure-ban.
author: MikeWasson
ms.date: 11/06/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: bea9b55938080685ece2f4f5b09f2a8735391947
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483241"
---
# <a name="create-a-stream-processing-pipeline-with-azure-stream-analytics"></a><span data-ttu-id="ddf5f-103">Az Azure Stream Analytics egy adatfolyam-feldolgozási folyamat létrehozása</span><span class="sxs-lookup"><span data-stu-id="ddf5f-103">Create a stream processing pipeline with Azure Stream Analytics</span></span>

<span data-ttu-id="ddf5f-104">Ez a referenciaarchitektúra bemutatja egy teljes körű [adatfolyam-feldolgozás](/azure/architecture/data-guide/big-data/real-time-processing) folyamat.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="ddf5f-105">A folyamat adatokat két forrásból fogadnak, utal. a két adatfolyam rekordokat, és kiszámítja a gördülő átlagát között egy olyan időkeretet.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="ddf5f-106">Az eredmények tárolása további elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-106">The results are stored for further analysis.</span></span>

<span data-ttu-id="ddf5f-107">Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="ddf5f-107">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![Az Azure Stream Analytics egy adatfolyam-feldolgozási folyamat létrehozásához a referencia-architektúra](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="ddf5f-109">**A forgatókönyv**: -I taxik vállalati minden taxi út adatokat gyűjt.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="ddf5f-110">Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="ddf5f-111">A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="ddf5f-112">Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="ddf5f-113">A taxi vállalat úgy kívánja kiszámítja az átlagos tipp kiszolgálónként mérföld alapuló, valós idejű, sorrendben felhasználva azonosíthatja a trendeket.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-113">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="ddf5f-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="ddf5f-114">Architecture</span></span>

<span data-ttu-id="ddf5f-115">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="ddf5f-116">**Adatforrások**.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-116">**Data sources**.</span></span> <span data-ttu-id="ddf5f-117">Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="ddf5f-118">Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="ddf5f-119">A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="ddf5f-120">Egy valós alkalmazás esetében az adatforrások lenne telepítve a taxi kabinetfájlok eszközök.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-120">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="ddf5f-121">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="ddf5f-122">[Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="ddf5f-123">Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="ddf5f-124">Minden adatforráshoz társított eseményközpont adatfolyamot küld.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="ddf5f-125">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-125">**Azure Stream Analytics**.</span></span> <span data-ttu-id="ddf5f-126">[Stream Analytics](/azure/stream-analytics/) egy eseményfeldolgozó motor.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-126">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="ddf5f-127">Stream Analytics-feladat beolvassa az adatokat a két event hubs szolgáltatástól érkező adatfolyamok és adatfolyam-feldolgozás hajt végre.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-127">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="ddf5f-128">**A cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-128">**Cosmos DB**.</span></span> <span data-ttu-id="ddf5f-129">A Stream Analytics-feladat kimenete a rekordoknak, amelyek egy Cosmos DB dokumentum-adatbázis a JSON-dokumentumok formájában írt sorozata.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-129">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="ddf5f-130">**A Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-130">**Microsoft Power BI**.</span></span> <span data-ttu-id="ddf5f-131">A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-131">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="ddf5f-132">Ebben az architektúrában az adatokat a Cosmos DB betölti.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-132">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="ddf5f-133">Ez lehetővé teszi a felhasználók teljes készletének összegyűjtött előzményadatok elemzésére.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-133">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="ddf5f-134">Az eredményeket közvetlenül a Stream Analytics a Power BI az adatok valós idejű nézet sikerült is folyamatos átvitelére.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-134">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="ddf5f-135">További információkért lásd: [valós idejű streamelés a Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-135">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="ddf5f-136">**Az Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-136">**Azure Monitor**.</span></span> <span data-ttu-id="ddf5f-137">[Az Azure Monitor](/azure/monitoring-and-diagnostics/) teljesítmény-mérőszámokat a megoldásban üzembe helyezett Azure-szolgáltatásokkal gyűjti.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-137">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="ddf5f-138">Ezek az irányítópult vizualizációjával betekintést nyerhet a megoldás állapotát.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-138">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="ddf5f-139">Adatfeldolgozás</span><span class="sxs-lookup"><span data-stu-id="ddf5f-139">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="ddf5f-140">Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-140">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="ddf5f-141">Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz egy 4 éven keresztül (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-141">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="ddf5f-142">Kétféle típusú rekordot tartalmaz: Adatok ledolgozni, és adatokat díjszabás.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-142">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="ddf5f-143">Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-143">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="ddf5f-144">Diszkont szerepel diszkont, adózási és tipp összegeket.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-144">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="ddf5f-145">Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-145">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="ddf5f-146">Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-146">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="ddf5f-147">Az adatok CSV formátumban tárolódik.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-147">The data is stored in CSV format.</span></span>

<span data-ttu-id="ddf5f-148">[1] <span id="note1">Donovan, Brian; Munkahelyi, Dan (2016): New York City Taxi Útadatok (2010, 2013).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-148">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="ddf5f-149">Egyetemi Illinois, Urbana-Champaignben.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-149">University of Illinois at Urbana-Champaign.</span></span> <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="ddf5f-150">Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-150">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="ddf5f-151">A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-151">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="ddf5f-152">Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-152">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="ddf5f-153">A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-153">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="ddf5f-154">Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-154">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="ddf5f-155">Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-155">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="ddf5f-156">Ez az adott esetben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-156">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="ddf5f-157">Ez lehetővé teszi a Stream Analytics párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-157">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="ddf5f-158">Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-158">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Az Azure Stream Analytics és az Event Hubs streamfeldolgozási ábrája](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="ddf5f-160">Az adatgenerátor a common data model mindkét bejegyzéstípusokat rendelkezik egy `PartitionKey` tulajdonság, amely az összefűzés, `Medallion`, `HackLicense`, és `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-160">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="ddf5f-161">Ez a tulajdonság adja meg egy explicit partíciókulcsot az Event hubs szolgáltatásba küldésekor használható:</span><span class="sxs-lookup"><span data-stu-id="ddf5f-161">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="ddf5f-162">Stream-feldolgozás</span><span class="sxs-lookup"><span data-stu-id="ddf5f-162">Stream processing</span></span>

<span data-ttu-id="ddf5f-163">A stream-feldolgozási feladat van definiálva egy SQL-lekérdezés használata több különálló lépésre.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-163">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="ddf5f-164">Az első két lépés egyszerűen válasszon rögzíti a két bemeneti streamekhez.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-164">The first two steps simply select records from the two input streams.</span></span>

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

<span data-ttu-id="ddf5f-165">A következő lépésben válassza ki a megfelelő rekordokat az egyes adatfolyamokkal két bemeneti streamekhez csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-165">The next step joins the two input streams to select matching records from each stream.</span></span>

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

<span data-ttu-id="ddf5f-166">Ez a lekérdezés illeszt rekordokat konfigurálásában a megfelelő rekordokat (Medallion, HackLicense, VendorId és PickupTime) egyedi azonosítására alkalmas mezőket.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-166">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="ddf5f-167">A `JOIN` utasítást is tartalmaz a partícióazonosító.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-167">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="ddf5f-168">Ahogy említettük, ez kihasználja a tény, hogy megfelelő rekordokat mindig rendelkezzenek a partíció ugyanazzal az Azonosítóval ebben a forgatókönyvben.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-168">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="ddf5f-169">A Stream Analytics, illesztések vannak *historikus*, azaz a rekordok tartományhoz csatlakoztatott idő adott időtartamon belül.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-169">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="ddf5f-170">Ellenkező esetben a feladat lehet, hogy várnia kell, határozatlan ideig az egyezést.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-170">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="ddf5f-171">A [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) függvény meghatározza, hogy milyen két egyező rekordot is választhatók el egymástól, időben egyezést.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-171">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span>

<span data-ttu-id="ddf5f-172">A feladat utolsó lépésében kiszámítja az átlagos tipp egy lépést, 5 perces ugróablak szerint csoportosítva.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-172">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="ddf5f-173">Stream Analytics biztosít több [ablakkezelési függvényeket](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-173">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="ddf5f-174">Egy ugróablak helyezi előre a idő által meghatározott időre ebben az esetben ugrásonkénti 1 perc.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-174">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="ddf5f-175">Ez a mozgó átlag kiszámításához az elmúlt 5 percben.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-175">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="ddf5f-176">Az itt látható architektúrában csak a Stream Analytics-feladat eredményeinek menti, a Cosmos DB-hez.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-176">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="ddf5f-177">A big Data típusú adatok esetben fontolja meg is [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) menteni a nyers eseményadatok Azure Blob storage-bA.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-177">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="ddf5f-178">A nyers adatok megőrzése mellett lehetővé teszi, hogy a batch lekérdezéseket futtathat az előzményadatok későbbi időpontban, annak érdekében, hogy új hozzájut a adatokból.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-178">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="ddf5f-179">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="ddf5f-179">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="ddf5f-180">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="ddf5f-180">Event Hubs</span></span>

<span data-ttu-id="ddf5f-181">Az Event Hubs átviteli kapacitásának mérése [átviteli egységek](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-181">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="ddf5f-182">Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről](/azure/event-hubs/event-hubs-auto-inflate), amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-182">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

### <a name="stream-analytics"></a><span data-ttu-id="ddf5f-183">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="ddf5f-183">Stream Analytics</span></span>

<span data-ttu-id="ddf5f-184">A Stream Analytics a számítási erőforrások hozzárendelt feladathoz a folyamatos átviteli egységek mérése történik.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-184">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="ddf5f-185">Stream Analytics-feladatok méretezése legjobb, ha a feladat is méretezésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-185">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="ddf5f-186">Ezzel a módszerrel Stream Analytics juttathatja el a feladatot több számítási csomóponton.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-186">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="ddf5f-187">Az Event Hubs-bemenet, használja a `PARTITION BY` kulcsszó a Stream Analytics-feladat particionálásához.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-187">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="ddf5f-188">Az adatok eloszlása az Event Hubs-partíciók alapján részekre.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-188">The data will be divided into subsets based on the Event Hubs partitions.</span></span>

<span data-ttu-id="ddf5f-189">Ablakkezelési függvényekről és társításokról, historikus szükséges további SU.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-189">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="ddf5f-190">Amikor lehetséges, használjon `PARTITION BY` úgy, hogy az egyes partíciók külön dolgozza fel.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-190">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="ddf5f-191">További információkért lásd: [ismertetése és módosítása a folyamatos átviteli egységek](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-191">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="ddf5f-192">Ha nem lehetséges a teljes Stream Analytics-feladat párhuzamosíthatja, próbálja meg a feladat felosztása több lépést, egy vagy több párhuzamos lépéseket tartalmazó indítása.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-192">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="ddf5f-193">Ezzel a módszerrel az első lépéseket is futtatható egyszerre.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-193">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="ddf5f-194">Ez a referenciaarchitektúra: például a</span><span class="sxs-lookup"><span data-stu-id="ddf5f-194">For example, in this reference architecture:</span></span>

- <span data-ttu-id="ddf5f-195">1. és 2 egyszerűek `SELECT` utasításokat, amelyek ugyanazon a partíción belül időtartományban.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-195">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span>
- <span data-ttu-id="ddf5f-196">3. lépés a particionált csatlakozzon két bemeneti streamekhez keresztül hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-196">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="ddf5f-197">Ebben a lépésben kihasználja a tény, hogy a megfelelő rekordokat megosztani ugyanazzal a partíciókulccsal, és így garantáltan minden egyes bemeneti Stream partíció ugyanazzal az Azonosítóval rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-197">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="ddf5f-198">4. lépés összesítések összes partíciót.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-198">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="ddf5f-199">Ez a lépés nem méretezésnek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-199">This step cannot be parallelized.</span></span>

<span data-ttu-id="ddf5f-200">A Stream Analytics használata [feladatábra](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) hány partíciók rendelt minden lépése a feladat megtekintéséhez.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-200">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="ddf5f-201">Az alábbi ábrán ez a referenciaarchitektúra a feladat diagramja:</span><span class="sxs-lookup"><span data-stu-id="ddf5f-201">The following diagram shows the job diagram for this reference architecture:</span></span>

![Feladatábra](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="ddf5f-203">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="ddf5f-203">Cosmos DB</span></span>

<span data-ttu-id="ddf5f-204">A Cosmos DB átviteli kapacitás mérése [kérelemegység](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-204">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="ddf5f-205">Annak érdekében, hogy az elmúlt 10 000-et egy Cosmos DB-tárolók skálázása RU, meg kell adnia egy [partíciókulcs](/azure/cosmos-db/partition-data) amikor létrehozza a tárolót, és a partíciókulcs tartalmazza minden egyes dokumentum.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-205">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span>

<span data-ttu-id="ddf5f-206">Ez a referenciaarchitektúra az új dokumentumok létrehozásának csak egyszer perc (a szórási ablak időköz), így az átviteli sebességet megkövetelő meglehetősen kicsi.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-206">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="ddf5f-207">Éppen ezért hiba esetén nem kell hozzárendelni az ebben a forgatókönyvben egy partíciókulcsot.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-207">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="ddf5f-208">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="ddf5f-208">Monitoring considerations</span></span>

<span data-ttu-id="ddf5f-209">Az adatfolyam-feldolgozó megoldást fontos a teljesítmény és a rendszer állapotának figyelésére.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-209">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="ddf5f-210">[Az Azure Monitor](/azure/monitoring-and-diagnostics/) az architektúrában használt Azure-szolgáltatások a metrikák és diagnosztikai naplókat gyűjti össze.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-210">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="ddf5f-211">Az Azure Monitor az Azure platform be van építve, és nincs szükség semmilyen további programkódokat kellene megtervezni az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-211">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="ddf5f-212">A következő figyelmeztető jeleket bármelyikét jelzi, hogy horizontális felskálázást a megfelelő Azure-erőforrás:</span><span class="sxs-lookup"><span data-stu-id="ddf5f-212">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="ddf5f-213">Az Event Hubs szabályozza a kéréseket, vagy a napi kvóta közelében van.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-213">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="ddf5f-214">A Stream Analytics-feladat konzisztens módon több mint 80 %-át a lefoglalt folyamatos átviteli egységek (su-t) használ.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-214">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="ddf5f-215">A cosmos DB megkezdi a kérések szabályozása.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-215">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="ddf5f-216">A referenciaarchitektúra egy egyéni irányítópult, amelyen telepítve van az Azure Portalon is tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-216">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="ddf5f-217">Után az architektúra üzembe helyezéséhez is megtekintheti az irányítópulton nyissa meg a [az Azure Portal](https://portal.azure.com) és kiválasztásával `TaxiRidesDashboard` az irányítópultok listájából.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-217">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="ddf5f-218">Az Azure Portalon egyéni irányítópultok létrehozása és telepítése kapcsolatos további információkért lásd: [programozott módon létrehozhat irányítópultokat Azure](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="ddf5f-218">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="ddf5f-219">Az alábbi képen az irányítópult látható, miután a Stream Analytics-feladat futásának időtartama: körülbelül egy óra alatt.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-219">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![A Taxi Rides irányítópult képernyőképe](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="ddf5f-221">A panel bal alsó mutatja, hogy a SU-használat a Stream Analytics-feladat az első 15 perc során megnő, és ezután itt megáll.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-221">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="ddf5f-222">Ez a lehetőség egy tipikus mintája a feladat eléri egy stabil állapotot.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-222">This is a typical pattern as the job reaches a steady state.</span></span>

<span data-ttu-id="ddf5f-223">Figyelje meg, hogy az Event Hubs szabályozza a kéréseket, a felső jobb oldali panelen látható.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-223">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="ddf5f-224">Alkalmanként szabályozott kérelmet nem áll egy probléma, mivel az Event hubs szolgáltatás ügyfél-SDK automatikusan újrapróbálkozik, ha sávszélesség-szabályozási hibaüzenet kap.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-224">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="ddf5f-225">Azonban ha egységes szabályozási hibákat lát, az azt jelenti az event hubs átviteli egységeket kell.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-225">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="ddf5f-226">A következő diagram is mutatja egy tesztfuttatás, az Event Hubs használatával az automatikus feltöltési funkció, amely automatikusan méretezhető az átviteli egységek igény szerint.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-226">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span>

![Képernyőkép az Event Hubs automatikus skálázás](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="ddf5f-228">Az automatikus feltöltési engedélyezve lett 06:35 kapcsolatos megjelölni.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-228">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="ddf5f-229">A p, dobja el a szabályozott kérelmeinek száma, az Event Hubs automatikusan átméreteződik legfeljebb 3 átviteli egységek láthatja.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-229">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="ddf5f-230">Interestingly ennek hatására oldal, ahol egyre növekszik a Stream Analytics-feladat a SU kihasználtságát.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-230">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="ddf5f-231">Szabályozás, az Event Hubs mesterségesen lett csökkentve a feldolgozási sebességét a Stream Analytics-feladat.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-231">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="ddf5f-232">Valójában szokás, hogy egy teljesítménybeli szűk keresztmetszetet feloldása tárja fel egy másik.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-232">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="ddf5f-233">Ebben az esetben további SU lefoglalása a Stream Analytics-feladat megoldotta a problémát.</span><span class="sxs-lookup"><span data-stu-id="ddf5f-233">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="ddf5f-234">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="ddf5f-234">Deploy the solution</span></span>

<span data-ttu-id="ddf5f-235">A telepítés, és futtassa a referenciaimplementációt, kövesse a lépéseket a [GitHub információs][github].</span><span class="sxs-lookup"><span data-stu-id="ddf5f-235">To the deploy and run the reference implementation, follow the steps in the [GitHub readme][github].</span></span>

## <a name="related-resources"></a><span data-ttu-id="ddf5f-236">Kapcsolódó erőforrások</span><span class="sxs-lookup"><span data-stu-id="ddf5f-236">Related resources</span></span>

<span data-ttu-id="ddf5f-237">Tekintse át az alábbiakat érdemes [Azure példaforgatókönyvek](/azure/architecture/example-scenario) , amelyek bemutatják, hogy egyes technológiákat használó adott megoldások:</span><span class="sxs-lookup"><span data-stu-id="ddf5f-237">You may wish to review the following [Azure example scenarios](/azure/architecture/example-scenario) that demonstrate specific solutions using some of the same technologies:</span></span>

- [<span data-ttu-id="ddf5f-238">IoT és data analytics konstrukció iparág</span><span class="sxs-lookup"><span data-stu-id="ddf5f-238">IoT and data analytics in the construction industry</span></span>](/azure/architecture/example-scenario/data/big-data-with-iot)
- [<span data-ttu-id="ddf5f-239">Az adathamisítások azonnali felismerése</span><span class="sxs-lookup"><span data-stu-id="ddf5f-239">Real-time fraud detection</span></span>](/azure/architecture/example-scenario/data/fraud-detection)

<!-- links -->

[github]: https://github.com/mspnp/azure-stream-analytics-data-pipeline

