---
title: Az Azure Databricks feldolgozó Stream
description: Egy teljes körű stream-feldolgozási folyamat létrehozása az Azure Databricks használatával
author: petertaylor9999
ms.date: 11/01/2018
ms.openlocfilehash: a7e9df57572c9b3a3b0e4f418f148449aa40b04c
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295761"
---
# <a name="stream-processing-with-azure-databricks"></a><span data-ttu-id="74575-103">Az Azure Databricks feldolgozó Stream</span><span class="sxs-lookup"><span data-stu-id="74575-103">Stream processing with Azure Databricks</span></span>

<span data-ttu-id="74575-104">Ez a referenciaarchitektúra bemutatja egy teljes körű [adatfolyam-feldolgozás](/azure/architecture/data-guide/big-data/real-time-processing) folyamat.</span><span class="sxs-lookup"><span data-stu-id="74575-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="74575-105">Ez a típusú folyamat négy fázisból áll: a betöltési, folyamat, tároló, és elemzés és jelentéskészítés.</span><span class="sxs-lookup"><span data-stu-id="74575-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="74575-106">A referenciaarchitektúra a folyamat adatokat két forrásból fogadnak, illesztést hajt végre az egyes adatfolyamokkal kapcsolódó bejegyzések, bővíti az eredményt és kiszámítja az átlagos valós időben.</span><span class="sxs-lookup"><span data-stu-id="74575-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="74575-107">Az eredmények tárolása további elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="74575-107">The results are stored for further analysis.</span></span> [<span data-ttu-id="74575-108">**A megoldás üzembe helyezése**.</span><span class="sxs-lookup"><span data-stu-id="74575-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

<span data-ttu-id="74575-109">**A forgatókönyv**:-i taxik vállalati minden taxi út adatokat gyűjt.</span><span class="sxs-lookup"><span data-stu-id="74575-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="74575-110">Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre.</span><span class="sxs-lookup"><span data-stu-id="74575-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="74575-111">A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság.</span><span class="sxs-lookup"><span data-stu-id="74575-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="74575-112">Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi.</span><span class="sxs-lookup"><span data-stu-id="74575-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="74575-113">A taxi vállalati részlege azt szeretné, kiszámítja az átlagos tipp mérföld alapuló, valós idejű, az egyes helyek száma, a helyszíni ridership trendeket.</span><span class="sxs-lookup"><span data-stu-id="74575-113">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="74575-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="74575-114">Architecture</span></span>

<span data-ttu-id="74575-115">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="74575-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="74575-116">**Adatforrások**.</span><span class="sxs-lookup"><span data-stu-id="74575-116">**Data sources**.</span></span> <span data-ttu-id="74575-117">Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást.</span><span class="sxs-lookup"><span data-stu-id="74575-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="74575-118">Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="74575-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="74575-119">A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="74575-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="74575-120">Az adatforrások egy valós alkalmazásban lenne telepítve a taxi kabinetfájlok eszközök.</span><span class="sxs-lookup"><span data-stu-id="74575-120">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="74575-121">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="74575-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="74575-122">[Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="74575-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="74575-123">Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ.</span><span class="sxs-lookup"><span data-stu-id="74575-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="74575-124">Minden adatforráshoz társított eseményközpont adatfolyamot küld.</span><span class="sxs-lookup"><span data-stu-id="74575-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="74575-125">**Az Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="74575-125">**Azure Databricks**.</span></span> <span data-ttu-id="74575-126">[Databricks](/azure/azure-databricks/) egy a Microsoft Azure cloud services platformra optimalizált Apache Spark-alapú elemzési platform.</span><span class="sxs-lookup"><span data-stu-id="74575-126">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="74575-127">Databricks az taxi indításáról, összekapcsolását és adatok díjszabás és a helyek adatokat tárolja a Databricks fájlrendszerhez korrelatív adatokat feldúsítani használatos.</span><span class="sxs-lookup"><span data-stu-id="74575-127">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="74575-128">**A cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="74575-128">**Cosmos DB**.</span></span> <span data-ttu-id="74575-129">Az Azure Databricks-feladat kimenete rekordoknak, amelyek írt sorozatát [Cosmos DB](/azure/cosmos-db/) a Cassandra API-val.</span><span class="sxs-lookup"><span data-stu-id="74575-129">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="74575-130">A Cassandra API támogatja a modellezés idősorozat-adatok használata.</span><span class="sxs-lookup"><span data-stu-id="74575-130">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="74575-131">**Az Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="74575-131">**Azure Log Analytics**.</span></span> <span data-ttu-id="74575-132">Alkalmazás adatainak naplózása által gyűjtött [Azure Monitor](/azure/monitoring-and-diagnostics/) tárolja egy [Log Analytics-munkaterület](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="74575-132">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="74575-133">Log Analytics-lekérdezések segítségével elemezheti és metrikák megjelenítése elérhessék és vizsgálhassák azonosíthatja a problémákat az alkalmazásban lévő üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="74575-133">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="74575-134">Adatfeldolgozás</span><span class="sxs-lookup"><span data-stu-id="74575-134">Data ingestion</span></span>

<span data-ttu-id="74575-135">Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="74575-135">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="74575-136">Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz, négyéves időszakban (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="74575-136">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="74575-137">Kétféle típusú rekordot tartalmaz: indításáról és diszkont adatait.</span><span class="sxs-lookup"><span data-stu-id="74575-137">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="74575-138">Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="74575-138">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="74575-139">Diszkont szerepel diszkont, adózási és tipp összegeket.</span><span class="sxs-lookup"><span data-stu-id="74575-139">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="74575-140">Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját.</span><span class="sxs-lookup"><span data-stu-id="74575-140">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="74575-141">Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot.</span><span class="sxs-lookup"><span data-stu-id="74575-141">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="74575-142">Az adatok CSV formátumban tárolódik.</span><span class="sxs-lookup"><span data-stu-id="74575-142">The data is stored in CSV format.</span></span> 

<span data-ttu-id="74575-143">Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="74575-143">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="74575-144">A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el.</span><span class="sxs-lookup"><span data-stu-id="74575-144">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="74575-145">Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat.</span><span class="sxs-lookup"><span data-stu-id="74575-145">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="74575-146">A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása.</span><span class="sxs-lookup"><span data-stu-id="74575-146">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="74575-147">Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat.</span><span class="sxs-lookup"><span data-stu-id="74575-147">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="74575-148">Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="74575-148">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="74575-149">Ebben a forgatókönyvben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl.</span><span class="sxs-lookup"><span data-stu-id="74575-149">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="74575-150">Ez lehetővé teszi a Databricks párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal.</span><span class="sxs-lookup"><span data-stu-id="74575-150">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="74575-151">Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.</span><span class="sxs-lookup"><span data-stu-id="74575-151">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="74575-152">Az adatgenerátor a common data model mindkét bejegyzéstípusokat rendelkezik egy `PartitionKey` összefűzésén tulajdonságot `Medallion`, `HackLicense`, és `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="74575-152">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="74575-153">Ez a tulajdonság adja meg egy explicit partíciókulcsot az Event hubs szolgáltatásba küldésekor használható:</span><span class="sxs-lookup"><span data-stu-id="74575-153">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="74575-154">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="74575-154">Event Hubs</span></span>

<span data-ttu-id="74575-155">Az Event Hubs átviteli kapacitásának mérése [átviteli egységek](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="74575-155">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="74575-156">Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről](/azure/event-hubs/event-hubs-auto-inflate), amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.</span><span class="sxs-lookup"><span data-stu-id="74575-156">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

## <a name="stream-processing"></a><span data-ttu-id="74575-157">Stream-feldolgozás</span><span class="sxs-lookup"><span data-stu-id="74575-157">Stream processing</span></span>

<span data-ttu-id="74575-158">Az Azure Databricksben adatfeldolgozási feladatok hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="74575-158">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="74575-159">A feladat hozzá van rendelve, és a egy fürtön.</span><span class="sxs-lookup"><span data-stu-id="74575-159">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="74575-160">A feladat lehet a Java és a egy Spark írt egyéni kód [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span><span class="sxs-lookup"><span data-stu-id="74575-160">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="74575-161">Ez a referenciaarchitektúra a feladat egy Java archív a Java- és Scala-ben írt osztályok.</span><span class="sxs-lookup"><span data-stu-id="74575-161">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="74575-162">A Java-archívumot egy Databricks-feladat megadásakor az osztály a Databricks-fürt által végrehajtási van megadva.</span><span class="sxs-lookup"><span data-stu-id="74575-162">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="74575-163">Itt a **fő** módszere a **com.microsoft.pnp.TaxiCabReader** osztály tartalmazza az adatok feldolgozási logikáját.</span><span class="sxs-lookup"><span data-stu-id="74575-163">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span> 

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="74575-164">A két event hub-példányok érkező adatfolyam olvasása</span><span class="sxs-lookup"><span data-stu-id="74575-164">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="74575-165">Az adatok feldolgozási logikáját használja [Spark strukturált streamelés](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) olvasni a két Azure event hub-példány:</span><span class="sxs-lookup"><span data-stu-id="74575-165">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

```scala
val rideEventHubOptions = EventHubsConf(rideEventHubConnectionString)
      .setConsumerGroup(conf.taxiRideConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val rideEvents = spark.readStream
      .format("eventhubs")
      .options(rideEventHubOptions.toMap)
      .load

    val fareEventHubOptions = EventHubsConf(fareEventHubConnectionString)
      .setConsumerGroup(conf.taxiFareConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val fareEvents = spark.readStream
      .format("eventhubs")
      .options(fareEventHubOptions.toMap)
      .load
```

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="74575-166">Az adatokat a helyek adatokat a bővítése</span><span class="sxs-lookup"><span data-stu-id="74575-166">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="74575-167">A indításáról adatok mentése a kivételezést szélességi és hosszúsági koordinátáit tartalmazza, és ki a helyek legördülő.</span><span class="sxs-lookup"><span data-stu-id="74575-167">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="74575-168">Ezeket a koordinátákat hasznosak, amíg azok könnyen fel nem elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="74575-168">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="74575-169">Ezért ezeket az adatokat a helyek adatokat, olvasási renderelésre egy [alakzatfájl](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="74575-169">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span> 

<span data-ttu-id="74575-170">Alakzatfájl formátuma bináris és nem egyszerűen elemzett, de a [GeoTools](http://geotools.org/) kódtár biztosít eszközöket a alakzatfájl formátumot használó térinformatikai adatok.</span><span class="sxs-lookup"><span data-stu-id="74575-170">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="74575-171">Ebben a könyvtárban van használatban a **com.microsoft.pnp.GeoFinder** határozza meg az hálózatok nevét, a kivételezést alapján be és ki koordináták drop osztály.</span><span class="sxs-lookup"><span data-stu-id="74575-171">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span> 

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="74575-172">A indításáról és diszkont adatok csatlakoztatása</span><span class="sxs-lookup"><span data-stu-id="74575-172">Joining the ride and fare data</span></span>

<span data-ttu-id="74575-173">Először a indításáról és diszkont adatokat alakította át:</span><span class="sxs-lookup"><span data-stu-id="74575-173">First the ride and fare data is transformed:</span></span>

```scala
    val rides = transformedRides
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedRides.add(1)
          false
        }
      })
      .select(
        $"ride.*",
        to_neighborhood($"ride.pickupLon", $"ride.pickupLat")
          .as("pickupNeighborhood"),
        to_neighborhood($"ride.dropoffLon", $"ride.dropoffLat")
          .as("dropoffNeighborhood")
      )
      .withWatermark("pickupTime", conf.taxiRideWatermarkInterval())

    val fares = transformedFares
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedFares.add(1)
          false
        }
      })
      .select(
        $"fare.*",
        $"pickupTime"
      )
      .withWatermark("pickupTime", conf.taxiFareWatermarkInterval())
```

<span data-ttu-id="74575-174">És a indításáról adatok diszkont adatokkal csatlakozik majd:</span><span class="sxs-lookup"><span data-stu-id="74575-174">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="74575-175">Az adatok feldolgozását és a Cosmos DB beszúrása</span><span class="sxs-lookup"><span data-stu-id="74575-175">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="74575-176">Az egyes helyek átlagos diszkont összeg kiszámítása egy adott időtartam alatt:</span><span class="sxs-lookup"><span data-stu-id="74575-176">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

```scala
val maxAvgFarePerNeighborhood = mergedTaxiTrip.selectExpr("medallion", "hackLicense", "vendorId", "pickupTime", "rateCode", "storeAndForwardFlag", "dropoffTime", "passengerCount", "tripTimeInSeconds", "tripDistanceInMiles", "pickupLon", "pickupLat", "dropoffLon", "dropoffLat", "paymentType", "fareAmount", "surcharge", "mtaTax", "tipAmount", "tollsAmount", "totalAmount", "pickupNeighborhood", "dropoffNeighborhood")
      .groupBy(window($"pickupTime", conf.windowInterval()), $"pickupNeighborhood")
      .agg(
        count("*").as("rideCount"),
        sum($"fareAmount").as("totalFareAmount"),
        sum($"tipAmount").as("totalTipAmount")
      )
      .select($"window.start", $"window.end", $"pickupNeighborhood", $"rideCount", $"totalFareAmount", $"totalTipAmount")
```

<span data-ttu-id="74575-177">Amely majd szúr be a Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="74575-177">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="74575-178">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="74575-178">Security considerations</span></span>

<span data-ttu-id="74575-179">Az Azure Database munkaterület elérését szabályzatokkal vezérelhető a [felügyeleti konzol](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="74575-179">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="74575-180">A felügyeleti konzolon adja hozzá a felhasználók, felhasználói engedélyek kezelése és egyszeri bejelentkezés beállítása funkcióit tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="74575-180">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="74575-181">Hozzáférés-vezérlés a munkaterületek, fürtök, feladatok és táblák is beállítható úgy, hogy a felügyeleti konzolon keresztül.</span><span class="sxs-lookup"><span data-stu-id="74575-181">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="74575-182">Titkos kódok kezelése</span><span class="sxs-lookup"><span data-stu-id="74575-182">Managing secrets</span></span>

<span data-ttu-id="74575-183">Az Azure Databricks tartalmaz egy [titkoskód-tárolót](https://docs.azuredatabricks.net/user-guide/secrets/index.html) , amely titkos adatait, beleértve a kapcsolati karakterláncok, hozzáférési kulcsokat, felhasználónevek és jelszavak tárolására szolgál.</span><span class="sxs-lookup"><span data-stu-id="74575-183">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="74575-184">Titkos kulcsok az Azure Databricks titkoskód-tárolót szerint vannak particionálva **hatókörök**:</span><span class="sxs-lookup"><span data-stu-id="74575-184">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="74575-185">Titkos kódok lettek hozzáadva a hatókör szintjén:</span><span class="sxs-lookup"><span data-stu-id="74575-185">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="74575-186">Egy Azure Key Vault biztonsági hatókört a natív Azure Databricks-hatókör helyett használható.</span><span class="sxs-lookup"><span data-stu-id="74575-186">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="74575-187">További tudnivalókért lásd: [Azure Key Vault biztonsági hatókörök](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="74575-187">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="74575-188">A kódban, titkos kulcsok az Azure Databricks-n keresztül elért [titkok segédprogramok](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span><span class="sxs-lookup"><span data-stu-id="74575-188">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>


## <a name="monitoring-considerations"></a><span data-ttu-id="74575-189">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="74575-189">Monitoring considerations</span></span>

<span data-ttu-id="74575-190">Az Azure Databricks az Apache Spark alapul, és mindkettő használja [log4j](https://logging.apache.org/log4j/2.x/) , a naplózáshoz standardní knihovna.</span><span class="sxs-lookup"><span data-stu-id="74575-190">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="74575-191">Az Apache Spark által biztosított alapértelmezett naplózása mellett ez a referenciaarchitektúra küld naplókat és mérőszámokat [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="74575-191">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="74575-192">A **com.microsoft.pnp.TaxiCabReader** osztály az Apache Spark naplózási rendszer a naplók elküldése az Azure Log Analyticshez szereplő értékek használatával konfigurálja a **log4j.properties** fájlt.</span><span class="sxs-lookup"><span data-stu-id="74575-192">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="74575-193">Az Apache Spark-naplózó üzenetek karakterláncok, amelyek az Azure Log Analytics naplóüzenetek JSON-ként formázott van szükség.</span><span class="sxs-lookup"><span data-stu-id="74575-193">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="74575-194">A **com.microsoft.pnp.log4j.LogAnalyticsAppender** osztály alakítja át, JSON ezeket az üzeneteket:</span><span class="sxs-lookup"><span data-stu-id="74575-194">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

```scala

    @Override
    protected void append(LoggingEvent loggingEvent) {
        if (this.layout == null) {
            this.setLayout(new JSONLayout());
        }

        String json = this.getLayout().format(loggingEvent);
        try {
            this.client.send(json, this.logType);
        } catch(IOException ioe) {
            LogLog.warn("Error sending LoggingEvent to Log Analytics", ioe);
        }
    }

```

<span data-ttu-id="74575-195">Mint a **com.microsoft.pnp.TaxiCabReader** osztály indításáról és diszkont üzeneteket dolgoz fel. lehetséges, vagy lehetnek helytelen formátumú, és ezért nem érvényes.</span><span class="sxs-lookup"><span data-stu-id="74575-195">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="74575-196">Éles környezetben fontos elemzése alapján azonosítja a probléma adódott az adatforrásokat, ezért gyorsan kell rögzíteni adatvesztés elkerülése érdekében ezen helytelenül formázott üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="74575-196">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="74575-197">A **com.microsoft.pnp.TaxiCabReader** osztály regisztrál egy Apache Spark akkumulátor, amely nyomon követi a helytelen formátumú diszkont és indításáról rekordok száma:</span><span class="sxs-lookup"><span data-stu-id="74575-197">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="74575-198">Az Apache Spark mérőszámok küldése Dropwizard kódtárat használja, és a natív Dropwizard metrikák mezők némelyike nem kompatibilis az Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="74575-198">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="74575-199">Ezért ez a referenciaarchitektúra egy egyéni Dropwizard fogadó és riporter tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="74575-199">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="74575-200">A metrikák az Azure Log Analytics által elvárt formátumú formázza azt.</span><span class="sxs-lookup"><span data-stu-id="74575-200">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="74575-201">Ha az Apache Spark-metrikák jelentések az egyéni metrikákat a helytelen formátumú indításáról és diszkont adatok is küld.</span><span class="sxs-lookup"><span data-stu-id="74575-201">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="74575-202">Az utolsó metrikát, be kell jelentkeznie az Azure Log Analytics-munkaterülethez összesített végrehajtási állapotát a Spark strukturált Stream feladat előrehaladását.</span><span class="sxs-lookup"><span data-stu-id="74575-202">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="74575-203">Ebben az esetben egy egyéni StreamingQuery használatával figyelő végrehajtani a **com.microsoft.pnp.StreamingMetricsListener** osztály.</span><span class="sxs-lookup"><span data-stu-id="74575-203">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="74575-204">Ez az osztály regisztrálva van az Apache Spark-munkamenetet a feladat futtatásakor:</span><span class="sxs-lookup"><span data-stu-id="74575-204">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="74575-205">A StreamingMetricsListener módszerei által meghívott az Apache Spark-futtatókörnyezet, amikor egy strukturált streamelési esemény bekövetkezésekor a küldő jelentkezzen üzenetek és mérőszámok az Azure Log Analytics-munkaterületet.</span><span class="sxs-lookup"><span data-stu-id="74575-205">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="74575-206">A következő lekérdezéseket a munkaterület segítségével az alkalmazás figyelése:</span><span class="sxs-lookup"><span data-stu-id="74575-206">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="74575-207">Lekérdezések folyamatos átviteli teljesítmény és a késés</span><span class="sxs-lookup"><span data-stu-id="74575-207">Latency and throughput for streaming queries</span></span> 

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d  
| render timechart
``` 
### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="74575-208">Stream-lekérdezés végrehajtása során naplózott kivételek</span><span class="sxs-lookup"><span data-stu-id="74575-208">Exceptions logged during stream query execution</span></span>

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error" 
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="74575-209">Helytelen formátumú diszkont és indításáról adatok felhalmozódása</span><span class="sxs-lookup"><span data-stu-id="74575-209">Accumulation of malformed fare and ride data</span></span>

```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedrides"

SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "metrics.malformedfares" 
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="74575-210">Nyomkövetési rugalmasság feladat futtatása</span><span class="sxs-lookup"><span data-stu-id="74575-210">Job execution to trace resiliency</span></span>
```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "driver.DAGScheduler.job.allJobs" 
```

## <a name="deploy-the-solution"></a><span data-ttu-id="74575-211">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="74575-211">Deploy the solution</span></span>

<span data-ttu-id="74575-212">Ez a referenciaarchitektúra egy üzemelő példánya érhető el az [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span><span class="sxs-lookup"><span data-stu-id="74575-212">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="74575-213">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="74575-213">Prerequisites</span></span>

1. <span data-ttu-id="74575-214">Klónozza, ágaztassa vagy töltse le a zip-fájlját a [referenciaarchitektúrákat](https://github.com/mspnp/reference-architectures) GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="74575-214">Clone, fork, or download the zip file for the [reference architectures](https://github.com/mspnp/reference-architectures) GitHub repository.</span></span>

2. <span data-ttu-id="74575-215">Telepítés [Docker](https://www.docker.com/) futtatásához az adatgenerálást.</span><span class="sxs-lookup"><span data-stu-id="74575-215">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="74575-216">Telepítés [az Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="74575-216">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="74575-217">Telepítés [a Databricks parancssori felület](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span><span class="sxs-lookup"><span data-stu-id="74575-217">Install [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span></span>

5. <span data-ttu-id="74575-218">Parancsot a parancssorba bash-parancssorból vagy PowerShell-parancssorból, jelentkezzen be Azure-fiókjába a következő:</span><span class="sxs-lookup"><span data-stu-id="74575-218">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>
    ```
    az login
    ```
6. <span data-ttu-id="74575-219">Telepítse a Java ide Környezethez, a következő források:</span><span class="sxs-lookup"><span data-stu-id="74575-219">Install a Java IDE, with the following resources:</span></span>
    - <span data-ttu-id="74575-220">JDK 1.8-AS VERZIÓJA</span><span class="sxs-lookup"><span data-stu-id="74575-220">JDK 1.8</span></span>
    - <span data-ttu-id="74575-221">SDK-t Scala 2.11-et</span><span class="sxs-lookup"><span data-stu-id="74575-221">Scala SDK 2.11</span></span>
    - <span data-ttu-id="74575-222">Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="74575-222">Maven 3.5.4</span></span>

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a><span data-ttu-id="74575-223">A New York City taxi és hálózatok adatfájlok le</span><span class="sxs-lookup"><span data-stu-id="74575-223">Download the New York City taxi and neighborhood data files</span></span>

1. <span data-ttu-id="74575-224">Hozzon létre egy könyvtárat nevű `DataFile` alatt a `data/streaming_azuredatabricks` könyvtárat a helyi fájlrendszerben.</span><span class="sxs-lookup"><span data-stu-id="74575-224">Create a directory named `DataFile` under the `data/streaming_azuredatabricks` directory in your local file system.</span></span>

2. <span data-ttu-id="74575-225">Nyisson meg egy webböngészőt, és navigáljon a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="74575-225">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="74575-226">Kattintson a **letöltése** gombra ezen az oldalon a megfelelő évnek a taxi-adatok egy zip-fájl letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="74575-226">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="74575-227">Bontsa ki a zip-fájlt a `DataFile` könyvtár.</span><span class="sxs-lookup"><span data-stu-id="74575-227">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="74575-228">Ez a tömörített fájl más zip-fájlokat tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="74575-228">This zip file contains other zip files.</span></span> <span data-ttu-id="74575-229">A gyermek zip-fájlokat nem csomagolja ki.</span><span class="sxs-lookup"><span data-stu-id="74575-229">Don't extract the child zip files.</span></span>

    <span data-ttu-id="74575-230">A directory-struktúra az alábbiakhoz hasonlóan kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="74575-230">The directory structure should look like the following:</span></span>

    ```
    /data
        /streaming_azuredatabricks
            /DataFile
                /FOIL2013
                    trip_data_1.zip
                    trip_data_2.zip
                    trip_data_3.zip
                    ...
    ```

5. <span data-ttu-id="74575-231">Nyisson meg egy webböngészőt, és navigáljon a https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span><span class="sxs-lookup"><span data-stu-id="74575-231">Open a web browser and navigate to https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span></span> 

6. <span data-ttu-id="74575-232">Kattintson a **New York-i helyek határait** letölteni a fájlt.</span><span class="sxs-lookup"><span data-stu-id="74575-232">Click on **New York Neighborhood Boundaries** to download the file.</span></span>

7. <span data-ttu-id="74575-233">Másolás a **ZillowNeighborhoods-NY.zip** fájlt a böngészőben a **letölti** mappában a `DataFile` könyvtár.</span><span class="sxs-lookup"><span data-stu-id="74575-233">Copy the **ZillowNeighborhoods-NY.zip** file from your browser's **downloads** directory to the `DataFile` directory.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="74575-234">Az Azure-erőforrások üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="74575-234">Deploy the Azure resources</span></span>

1. <span data-ttu-id="74575-235">A rendszerhéj vagy a Windows-parancssort futtassa a következő parancsot, és kövesse a bejelentkezési kérések jelenhetnek:</span><span class="sxs-lookup"><span data-stu-id="74575-235">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="74575-236">Lépjen abba a mappába `data/streaming_azuredatabricks` a GitHub-adattárban</span><span class="sxs-lookup"><span data-stu-id="74575-236">Navigate to the folder `data/streaming_azuredatabricks` in the GitHub repository</span></span>

    ```bash
    cd data/streaming_azuredatabricks
    ```

3. <span data-ttu-id="74575-237">Futtassa az alábbi parancsokat az Azure-erőforrások üzembe helyezéséhez:</span><span class="sxs-lookup"><span data-stu-id="74575-237">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Region]'
    export eventHubNamespace='[Event Hubs namespace name]'
    export databricksWorkspaceName='[Azure Databricks workspace name]'
    export cosmosDatabaseAccount='[Cosmos DB database name]'
    export logAnalyticsWorkspaceName='[Log Analytics workspace name]'
    export logAnalyticsWorkspaceRegion='[Log Analytics region]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
        --template-file ./azure/deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. <span data-ttu-id="74575-238">A kimenet az üzembe helyezés befejezése után a konzol íródik.</span><span class="sxs-lookup"><span data-stu-id="74575-238">The output of the deployment is written to the console once complete.</span></span> <span data-ttu-id="74575-239">A kimenetben keresse meg a következő JSON-ra:</span><span class="sxs-lookup"><span data-stu-id="74575-239">Search the output for the following JSON:</span></span>

```JSON
"outputs": {
        "cosmosDb": {
          "type": "Object",
          "value": {
            "hostName": <value>,
            "secret": <value>,
            "username": <value>
          }
        },
        "eventHubs": {
          "type": "Object",
          "value": {
            "taxi-fare-eh": <value>,
            "taxi-ride-eh": <value>
          }
        },
        "logAnalytics": {
          "type": "Object",
          "value": {
            "secret": <value>,
            "workspaceId": <value>
          }
        }
},
```
<span data-ttu-id="74575-240">Ezeket az értékeket a titkos kulcsok, amely belekerül a későbbi szakaszokban a Databricks titkos kódokhoz való.</span><span class="sxs-lookup"><span data-stu-id="74575-240">These values are the secrets that will be added to Databricks secrets in upcoming sections.</span></span> <span data-ttu-id="74575-241">Megőrizheti azok biztonságát mindaddig, amíg szakaszt hozzáadhatja őket.</span><span class="sxs-lookup"><span data-stu-id="74575-241">Keep them secure until you add them in those sections.</span></span>

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a><span data-ttu-id="74575-242">A Cosmos DB-fiók egy Cassandra-tábla hozzáadása</span><span class="sxs-lookup"><span data-stu-id="74575-242">Add a Cassandra table to the Cosmos DB Account</span></span>

1. <span data-ttu-id="74575-243">Az Azure Portalon lépjen a létrehozott erőforráscsoportot a **üzembe helyezése az Azure-erőforrások** című fenti szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-243">In the Azure portal, navigate to the resource group created in the **deploy the Azure resources** section above.</span></span> <span data-ttu-id="74575-244">Kattintson a **az Azure Cosmos DB-fiók**.</span><span class="sxs-lookup"><span data-stu-id="74575-244">Click on **Azure Cosmos DB Account**.</span></span> <span data-ttu-id="74575-245">Hozzon létre egy táblát a Cassandra API-val.</span><span class="sxs-lookup"><span data-stu-id="74575-245">Create a table with the Cassandra API.</span></span>

2. <span data-ttu-id="74575-246">Az a **áttekintése** panelen kattintson a **tábla hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="74575-246">In the **overview** blade, click **add table**.</span></span>

3. <span data-ttu-id="74575-247">Ha a **tábla hozzáadása** panel nyílik meg, adja meg `newyorktaxi` a a **Kulcstér neve** szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="74575-247">When the **add table** blade opens, enter `newyorktaxi` in the **Keyspace name** text box.</span></span> 

4. <span data-ttu-id="74575-248">Az a **adja meg a tábla létrehozásához CQL parancsot** területén adja meg `neighborhoodstats` mellett a szövegmezőben `newyorktaxi`.</span><span class="sxs-lookup"><span data-stu-id="74575-248">In the **enter CQL command to create the table** section, enter `neighborhoodstats` in the text box beside `newyorktaxi`.</span></span>

5. <span data-ttu-id="74575-249">Az alábbi szövegmezőbe írja be a következőket:</span><span class="sxs-lookup"><span data-stu-id="74575-249">In the text box below, enter the following::</span></span>
```
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. <span data-ttu-id="74575-250">Az a **átviteli sebesség (1000 – 1 000 000 RU/s)** szövegmezőbe írja be az értéket `4000`.</span><span class="sxs-lookup"><span data-stu-id="74575-250">In the **Throughput (1,000 - 1,000,000 RU/s)** text box enter the value `4000`.</span></span>

7. <span data-ttu-id="74575-251">Kattintson az **OK** gombra.</span><span class="sxs-lookup"><span data-stu-id="74575-251">Click **OK**.</span></span>

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a><span data-ttu-id="74575-252">A Databricks parancssori felület használatával a Databricks-titkos kulcsok hozzáadása</span><span class="sxs-lookup"><span data-stu-id="74575-252">Add the Databricks secrets using the Databricks CLI</span></span>

<span data-ttu-id="74575-253">Először adja meg a titkos kulcsok EventHub:</span><span class="sxs-lookup"><span data-stu-id="74575-253">First, enter the secrets for EventHub:</span></span>

1. <span data-ttu-id="74575-254">Használatával a **Azure Databricks parancssori felület** telepítette a 2. lépésben előfeltétel, az Azure Databricks titkos hatókör létrehozása:</span><span class="sxs-lookup"><span data-stu-id="74575-254">Using the **Azure Databricks CLI** installed in step 2 of the prerequisites, create the Azure Databricks secret scope:</span></span>
    ```
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. <span data-ttu-id="74575-255">Adja hozzá a titkos kulcsot a taxi indításáról EventHub:</span><span class="sxs-lookup"><span data-stu-id="74575-255">Add the secret for the taxi ride EventHub:</span></span>
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    <span data-ttu-id="74575-256">Miután hajtja végre, a vi-szerkesztő megnyitása.</span><span class="sxs-lookup"><span data-stu-id="74575-256">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="74575-257">Adja meg a **taxi-indításáról-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-257">Enter the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="74575-258">Mentse és vi kilép.</span><span class="sxs-lookup"><span data-stu-id="74575-258">Save and exit vi.</span></span>

3. <span data-ttu-id="74575-259">Adja hozzá a titkos kulcsot a taxi diszkont EventHub:</span><span class="sxs-lookup"><span data-stu-id="74575-259">Add the secret for the taxi fare EventHub:</span></span>
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    <span data-ttu-id="74575-260">Miután hajtja végre, a vi-szerkesztő megnyitása.</span><span class="sxs-lookup"><span data-stu-id="74575-260">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="74575-261">Adja meg a **taxi-diszkont-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-261">Enter the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="74575-262">Mentse és vi kilép.</span><span class="sxs-lookup"><span data-stu-id="74575-262">Save and exit vi.</span></span>

<span data-ttu-id="74575-263">Írja be a titkos kulcsok a Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="74575-263">Next, enter the secrets for Cosmos DB:</span></span>

1. <span data-ttu-id="74575-264">Nyissa meg az Azure Portalon, és keresse meg a 3. lépésében megadott erőforráscsoportot a **üzembe helyezése az Azure-erőforrások** szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-264">Open the Azure portal, and navigate to the resource group specified in step 3 of the **deploy the Azure resources** section.</span></span> <span data-ttu-id="74575-265">Kattintson az Azure Cosmos DB-fiók.</span><span class="sxs-lookup"><span data-stu-id="74575-265">Click on the Azure Cosmos DB Account.</span></span>

2. <span data-ttu-id="74575-266">Használatával a **Azure Databricks parancssori felület**, adja hozzá a titkos kulcsot, a Cosmos DB-felhasználónév:</span><span class="sxs-lookup"><span data-stu-id="74575-266">Using the **Azure Databricks CLI**, add the secret for the Cosmos DB user name:</span></span>
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
<span data-ttu-id="74575-267">Miután hajtja végre, a vi-szerkesztő megnyitása.</span><span class="sxs-lookup"><span data-stu-id="74575-267">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="74575-268">Adja meg a **felhasználónév** értéket a **CosmosDb** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-268">Enter the **username** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="74575-269">Mentse és vi kilép.</span><span class="sxs-lookup"><span data-stu-id="74575-269">Save and exit vi.</span></span>

3. <span data-ttu-id="74575-270">Ezután adja hozzá a titkos kulcsot, a Cosmos DB-jelszó:</span><span class="sxs-lookup"><span data-stu-id="74575-270">Next, add the secret for the Cosmos DB password:</span></span>
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

<span data-ttu-id="74575-271">Miután hajtja végre, a vi-szerkesztő megnyitása.</span><span class="sxs-lookup"><span data-stu-id="74575-271">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="74575-272">Adja meg a **titkos** értéket a **CosmosDb** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-272">Enter the **secret** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="74575-273">Mentse és vi kilép.</span><span class="sxs-lookup"><span data-stu-id="74575-273">Save and exit vi.</span></span>

> [!NOTE]
> <span data-ttu-id="74575-274">Ha használ egy [titkos hatókör Azure Key Vault-alapú](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), névvel kell rendelkeznie a hatókör **azure databricks-feladat** és a titkos kulcsok rendelkeznie kell a fent említett pontos megegyező nevekkel.</span><span class="sxs-lookup"><span data-stu-id="74575-274">If using an [Azure Key Vault-backed secret scope](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), the scope must be named **azure-databricks-job** and the secrets must have the exact same names as those above.</span></span>

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a><span data-ttu-id="74575-275">Adja hozzá a Zillow környékeken adatokat a Databricks fájlrendszerhez</span><span class="sxs-lookup"><span data-stu-id="74575-275">Add the Zillow Neighborhoods data file to the Databricks file system</span></span>

1. <span data-ttu-id="74575-276">Hozzon létre egy könyvtárat a Databricks fájlrendszerhez:</span><span class="sxs-lookup"><span data-stu-id="74575-276">Create a directory in the Databricks file system:</span></span>
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. <span data-ttu-id="74575-277">Adatok/streaming_azuredatabricks/adatfájlját keresse meg, és adja meg a következőket:</span><span class="sxs-lookup"><span data-stu-id="74575-277">Navigate to data/streaming_azuredatabricks/DataFile and enter the following:</span></span>
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a><span data-ttu-id="74575-278">Az Azure Log Analytics-munkaterület Azonosítójára és az elsődleges kulcs hozzáadása a konfigurációs fájlok</span><span class="sxs-lookup"><span data-stu-id="74575-278">Add the Azure Log Analytics workspace ID and primary key to configuration files</span></span>

<span data-ttu-id="74575-279">Ebben a szakaszban van szüksége a Log Analytics-munkaterület Azonosítójára és az elsődleges kulcsot.</span><span class="sxs-lookup"><span data-stu-id="74575-279">For this section, you require the Log Analytics workspace ID and primary key.</span></span> <span data-ttu-id="74575-280">A munkaterület-azonosító a **munkaterület azonosítója** értéket a **logAnalytics** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-280">The workspace ID is the **workspaceId** value from the **logAnalytics** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="74575-281">Az elsődleges kulcs a **titkos** a kimeneti szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-281">The primary key is the **secret** from the output section.</span></span> 

1. <span data-ttu-id="74575-282">Nyissa meg a data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties log4j naplózás beállításához.</span><span class="sxs-lookup"><span data-stu-id="74575-282">To configure log4j logging, open data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties.</span></span> <span data-ttu-id="74575-283">A következő két érték szerkesztése:</span><span class="sxs-lookup"><span data-stu-id="74575-283">Edit the following two values:</span></span>
    ```
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. <span data-ttu-id="74575-284">Egyéni naplózás beállításához nyissa meg a data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties.</span><span class="sxs-lookup"><span data-stu-id="74575-284">To configure custom logging, open data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties.</span></span> <span data-ttu-id="74575-285">A következő két érték szerkesztése:</span><span class="sxs-lookup"><span data-stu-id="74575-285">Edit the following two values:</span></span>
    ``` 
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a><span data-ttu-id="74575-286">Hozhat létre a .jar fájlokat a Databricks-feladat és a Databricks-figyelés</span><span class="sxs-lookup"><span data-stu-id="74575-286">Build the .jar files for the Databricks job and Databricks monitoring</span></span>

1. <span data-ttu-id="74575-287">A Java IDE használatával importálhatja a nevű Maven-projektfájlból **pom.xml** gyökerében található a **data/streaming_azuredatabricks** könyvtár.</span><span class="sxs-lookup"><span data-stu-id="74575-287">Use your Java IDE to import the Maven project file named **pom.xml** located in the root of the **data/streaming_azuredatabricks** directory.</span></span> 

2. <span data-ttu-id="74575-288">Hajtsa végre a tiszta buildjének kiépítéséhez.</span><span class="sxs-lookup"><span data-stu-id="74575-288">Perform a clean build.</span></span> <span data-ttu-id="74575-289">A build kimenete nevű fájlt a **azure-databricks-feladat – 1.0-SNAPSHOT.jar** és **azure-databricks-figyelés – 0.9.jar**.</span><span class="sxs-lookup"><span data-stu-id="74575-289">The output of this build is files named **azure-databricks-job-1.0-SNAPSHOT.jar** and **azure-databricks-monitoring-0.9.jar**.</span></span> 

### <a name="configure-custom-logging-for-the-databricks-job"></a><span data-ttu-id="74575-290">A Databricks-feladat az egyéni naplózás konfigurálása</span><span class="sxs-lookup"><span data-stu-id="74575-290">Configure custom logging for the Databricks job</span></span>

1. <span data-ttu-id="74575-291">Másolás a **azure-databricks-figyelés – 0.9.jar** fájlt a következő parancs beírásával a Databricks fájlrendszerhez a **a Databricks parancssori felület**:</span><span class="sxs-lookup"><span data-stu-id="74575-291">Copy the **azure-databricks-monitoring-0.9.jar** file to the Databricks file system by entering the following command in the **Databricks CLI**:</span></span>
    ```
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. <span data-ttu-id="74575-292">Másolja az egyéni naplózás tulajdonságai a data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties a Databricks fájlrendszerhez a következő parancs beírásával:</span><span class="sxs-lookup"><span data-stu-id="74575-292">Copy the custom logging properties from data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. <span data-ttu-id="74575-293">Bár még a Databricks-fürt nevét, még nem úgy döntött, válasszon ki egyet.</span><span class="sxs-lookup"><span data-stu-id="74575-293">While you haven't yet decided on a name for your Databricks cluster, select one now.</span></span> <span data-ttu-id="74575-294">A fürt neve alatt fogja meg a Databricks fájlrendszerbeli elérési út.</span><span class="sxs-lookup"><span data-stu-id="74575-294">You'll enter the name below in the Databricks file system path for your cluster.</span></span> <span data-ttu-id="74575-295">Másolja a inicializálása a data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics a Databricks fájlrendszerhez a következő parancs beírásával:</span><span class="sxs-lookup"><span data-stu-id="74575-295">Copy the initialization script from data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics to the Databricks file system by entering the following command:</span></span>
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a><span data-ttu-id="74575-296">Databricks-fürt létrehozása</span><span class="sxs-lookup"><span data-stu-id="74575-296">Create a Databricks cluster</span></span>

1. <span data-ttu-id="74575-297">A Databricks munkaterületen kattintson a "Fürt", majd kattintson a "létrehozása a fürt".</span><span class="sxs-lookup"><span data-stu-id="74575-297">In the Databricks workspace, click "Clusters", then click "create cluster".</span></span> <span data-ttu-id="74575-298">Adja meg a 3. lépésében létrehozott fürt neve a **a Databricks-feladat az egyéni naplózás konfigurálása** című fenti szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-298">Enter the cluster name you created in step 3 of the **configure custom logging for the Databricks job** section above.</span></span> 

2. <span data-ttu-id="74575-299">Válassza ki a **standard** fürt üzemmód.</span><span class="sxs-lookup"><span data-stu-id="74575-299">Select a **standard** cluster mode.</span></span>

3. <span data-ttu-id="74575-300">Állítsa be **Databricks futtatókörnyezet-verziója** való **4.3-as (tartalmazza az Apache Spark 2.3.1, Scala 2.11-et)**</span><span class="sxs-lookup"><span data-stu-id="74575-300">Set **Databricks runtime version** to **4.3 (includes Apache Spark 2.3.1, Scala 2.11)**</span></span>

4. <span data-ttu-id="74575-301">Állítsa be **Python-verzió** való **2**.</span><span class="sxs-lookup"><span data-stu-id="74575-301">Set **Python version** to **2**.</span></span>

5. <span data-ttu-id="74575-302">Állítsa be **illesztőprogram típusa** való **ugyanaz, mint a feldolgozó**</span><span class="sxs-lookup"><span data-stu-id="74575-302">Set **Driver Type** to **Same as worker**</span></span>

6. <span data-ttu-id="74575-303">Állítsa be **feldolgozó típusa** való **Standard_DS3_v2**.</span><span class="sxs-lookup"><span data-stu-id="74575-303">Set **Worker Type** to **Standard_DS3_v2**.</span></span>

7. <span data-ttu-id="74575-304">Állítsa be **min. feldolgozók** való **2**.</span><span class="sxs-lookup"><span data-stu-id="74575-304">Set **Min Workers** to **2**.</span></span>

8. <span data-ttu-id="74575-305">Kapcsolja ki **automatikus skálázás engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="74575-305">Deselect **Enable autoscaling**.</span></span> 

9. <span data-ttu-id="74575-306">Alább a **automatikus Bezárás** párbeszédpanelen kattintson a **Init parancsfájlok**.</span><span class="sxs-lookup"><span data-stu-id="74575-306">Below the **Auto Termination** dialog box, click on **Init Scripts**.</span></span> 

10. <span data-ttu-id="74575-307">Adja meg **dbfs: / databricks/init/< fürtnév > / spark-metrics.sh**, és cserélje le a fürt neve a < fürtnév > 1. lépésben létrehozott.</span><span class="sxs-lookup"><span data-stu-id="74575-307">Enter **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**, substituting the cluster name created in step 1 for <cluster-name>.</span></span>

11. <span data-ttu-id="74575-308">Kattintson a **Hozzáadás** gombra.</span><span class="sxs-lookup"><span data-stu-id="74575-308">Click the **Add** button.</span></span>

12. <span data-ttu-id="74575-309">Kattintson a **-fürt létrehozása** gombra.</span><span class="sxs-lookup"><span data-stu-id="74575-309">Click the **Create Cluster** button.</span></span>

### <a name="create-a-databricks-job"></a><span data-ttu-id="74575-310">Hozzon létre egy Databricks-feladat</span><span class="sxs-lookup"><span data-stu-id="74575-310">Create a Databricks job</span></span>

1. <span data-ttu-id="74575-311">A Databricks munkaterületen kattintson a "Jobs", "a feladat létrehozása".</span><span class="sxs-lookup"><span data-stu-id="74575-311">In the Databricks workspace, click "Jobs", "create job".</span></span>

2. <span data-ttu-id="74575-312">Adja meg a feladat nevét.</span><span class="sxs-lookup"><span data-stu-id="74575-312">Enter a job name.</span></span>

3. <span data-ttu-id="74575-313">Kattintson a "set jar", megnyílik a "Feltöltés JAR futtassa a" párbeszédpanel.</span><span class="sxs-lookup"><span data-stu-id="74575-313">Click "set jar", this opens the "Upload JAR to Run" dialog box.</span></span>

4. <span data-ttu-id="74575-314">Húzza a **azure-databricks-feladat – 1.0-SNAPSHOT.jar** létrehozott fájlt a **hozhat létre a .JAR fájlt a Databricks-feladat** részt a **itt dobja el JAR feltöltéséhez** mezőbe.</span><span class="sxs-lookup"><span data-stu-id="74575-314">Drag the **azure-databricks-job-1.0-SNAPSHOT.jar** file you created in the **build the .jar for the Databricks job** section to the **Drop JAR here to upload** box.</span></span>

5. <span data-ttu-id="74575-315">Adja meg **com.microsoft.pnp.TaxiCabReader** a a **Main osztály** mező.</span><span class="sxs-lookup"><span data-stu-id="74575-315">Enter **com.microsoft.pnp.TaxiCabReader** in the **Main Class** field.</span></span>

6. <span data-ttu-id="74575-316">Az argumentumok mezőben adja meg a következőket:</span><span class="sxs-lookup"><span data-stu-id="74575-316">In the arguments field, enter the following:</span></span>
    ```
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. <span data-ttu-id="74575-317">A függő kódtárak telepítéséhez az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="74575-317">Install the dependent libraries by following these steps:</span></span>
    
    1. <span data-ttu-id="74575-318">A Databricks felhasználói felületén kattintson a a **otthoni** gombra.</span><span class="sxs-lookup"><span data-stu-id="74575-318">In the Databricks user interface, click on the **home** button.</span></span>
    
    2. <span data-ttu-id="74575-319">Az a **felhasználók** legördülő, a felhasználói fiók nevére kattintva nyissa meg a fiók munkaterület beállításait.</span><span class="sxs-lookup"><span data-stu-id="74575-319">In the **Users** drop-down, click on your user account name to open your account workspace settings.</span></span>
    
    3. <span data-ttu-id="74575-320">A legördülő nyílra, a fiók neve melletti jelölőnégyzetet, kattintson a **létrehozása**, majd kattintson a **könyvtár** megnyitásához a **új könyvtár** párbeszédpanel.</span><span class="sxs-lookup"><span data-stu-id="74575-320">Click on the drop-down arrow beside your account name, click on **create**, and click on **Library** to open the **New Library** dialog.</span></span>
    
    4. <span data-ttu-id="74575-321">Az a **forrás** legördülő vezérlőt, jelölje be **Maven-koordináta**.</span><span class="sxs-lookup"><span data-stu-id="74575-321">In the **Source** drop-down control, select **Maven Coordinate**.</span></span>
    
    5. <span data-ttu-id="74575-322">Alatt a **telepítése Maven-összetevőkben** fejléc adja meg `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` a a **koordinálja** szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="74575-322">Under the **Install Maven Artifacts** heading, enter `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` in the **Coordinate** text box.</span></span> 
    
    6. <span data-ttu-id="74575-323">Kattintson a **könyvtár létrehozása** megnyitásához a **összetevők** ablak.</span><span class="sxs-lookup"><span data-stu-id="74575-323">Click on **Create Library** to open the **Artifacts** window.</span></span>
    
    7. <span data-ttu-id="74575-324">A **állapotának a futó fürtök** ellenőrizze a **automatikusan csatolja az összes fürt** jelölőnégyzetet.</span><span class="sxs-lookup"><span data-stu-id="74575-324">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>
    
    8. <span data-ttu-id="74575-325">Ismételje meg az 1-7 a lépéseket a `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven-koordináta.</span><span class="sxs-lookup"><span data-stu-id="74575-325">Repeat steps 1 - 7 for the `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven coordinate.</span></span>
    
    9. <span data-ttu-id="74575-326">Ismételje meg az 1 – 6. lépéseket a `org.geotools:gt-shapefile:19.2` Maven-koordináta.</span><span class="sxs-lookup"><span data-stu-id="74575-326">Repeat steps 1 - 6 for the `org.geotools:gt-shapefile:19.2` Maven coordinate.</span></span>
    
    10. <span data-ttu-id="74575-327">Kattintson a **speciális beállítások**.</span><span class="sxs-lookup"><span data-stu-id="74575-327">Click on **Advanced Options**.</span></span>
    
    11. <span data-ttu-id="74575-328">Adja meg `http://download.osgeo.org/webdav/geotools/` a a **tárház** szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="74575-328">Enter `http://download.osgeo.org/webdav/geotools/` in the **Repository** text box.</span></span> 
    
    12. <span data-ttu-id="74575-329">Kattintson a **könyvtár létrehozása** megnyitásához a **összetevők** ablak.</span><span class="sxs-lookup"><span data-stu-id="74575-329">Click **Create Library** to open the **Artifacts** window.</span></span> 
    
    13. <span data-ttu-id="74575-330">A **állapotának a futó fürtök** ellenőrizze a **automatikusan csatolja az összes fürt** jelölőnégyzetet.</span><span class="sxs-lookup"><span data-stu-id="74575-330">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

8. <span data-ttu-id="74575-331">A 7. lépésben létrehozott 6. lépés végén található a feladathoz hozzáadja a függő kódtárak hozzáadása:</span><span class="sxs-lookup"><span data-stu-id="74575-331">Add the dependent libraries added in step 7 to the job created at the end of step 6:</span></span>
    1. <span data-ttu-id="74575-332">Az Azure Databricks-munkaterületet, kattintson a **feladatok**.</span><span class="sxs-lookup"><span data-stu-id="74575-332">In the Azure Databricks workspace, click on **Jobs**.</span></span>

    2. <span data-ttu-id="74575-333">A feladat nevére, a 2. lépésben létrehozott a **hozzon létre egy Databricks-feladat** szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-333">Click on the job name created in step 2 of the **create a Databricks job** section.</span></span> 
    
    3. <span data-ttu-id="74575-334">Mellett a **függő kódtárak** területén kattintson a **Hozzáadás** megnyitásához a **függő könyvtár hozzáadása** párbeszédpanel.</span><span class="sxs-lookup"><span data-stu-id="74575-334">Beside the **Dependent Libraries** section, click on **Add** to open the **Add Dependent Library** dialog.</span></span> 
    
    4. <span data-ttu-id="74575-335">A **könyvtár a** kiválasztása **munkaterület**.</span><span class="sxs-lookup"><span data-stu-id="74575-335">Under **Library From** select **Workspace**.</span></span>
    
    5. <span data-ttu-id="74575-336">Kattintson a **felhasználók**, majd a felhasználónevét, majd kattintson a `azure-eventhubs-spark_2.11:2.3.5`.</span><span class="sxs-lookup"><span data-stu-id="74575-336">Click on **users**, then your username, then click on `azure-eventhubs-spark_2.11:2.3.5`.</span></span> 
    
    6. <span data-ttu-id="74575-337">Kattintson az **OK** gombra.</span><span class="sxs-lookup"><span data-stu-id="74575-337">Click **OK**.</span></span>
    
    7. <span data-ttu-id="74575-338">Ismételje meg az 1 – 6. lépéseket `spark-cassandra-connector_2.11:2.3.1` és `gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="74575-338">Repeat steps 1 - 6 for `spark-cassandra-connector_2.11:2.3.1` and `gt-shapefile:19.2`.</span></span>

9. <span data-ttu-id="74575-339">Mellett **fürt:**, kattintson a **szerkesztése**.</span><span class="sxs-lookup"><span data-stu-id="74575-339">Beside **Cluster:**, click on **Edit**.</span></span> <span data-ttu-id="74575-340">Ekkor megnyílik a **fürt konfigurálása** párbeszédpanel.</span><span class="sxs-lookup"><span data-stu-id="74575-340">This opens the **Configure Cluster** dialog.</span></span> <span data-ttu-id="74575-341">Az a **fürttípus** legördülő menüben válassza **meglévő fürt**.</span><span class="sxs-lookup"><span data-stu-id="74575-341">In the **Cluster Type** drop-down, select **Existing Cluster**.</span></span> <span data-ttu-id="74575-342">Az a **válassza ki a fürt** legördülő menüben válassza ki a létrehozott fürtöt a **hozzon létre egy Databricks-fürt** szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-342">In the **Select Cluster** drop-down, select the cluster created the **create a Databricks cluster** section.</span></span> <span data-ttu-id="74575-343">Kattintson a **megerősítése**.</span><span class="sxs-lookup"><span data-stu-id="74575-343">Click **confirm**.</span></span>

10. <span data-ttu-id="74575-344">Kattintson a **Futtatás most**.</span><span class="sxs-lookup"><span data-stu-id="74575-344">Click **run now**.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="74575-345">Futtassa az adatgenerátor</span><span class="sxs-lookup"><span data-stu-id="74575-345">Run the data generator</span></span>

1. <span data-ttu-id="74575-346">Lépjen abba a könyvtárba `data/streaming_azuredatabricks/onprem` a GitHub-adattárában.</span><span class="sxs-lookup"><span data-stu-id="74575-346">Navigate to the directory `data/streaming_azuredatabricks/onprem` in the GitHub repository.</span></span>

2. <span data-ttu-id="74575-347">Frissítse az értékeket a fájlban **main.env** módon:</span><span class="sxs-lookup"><span data-stu-id="74575-347">Update the values in the file **main.env** as follows:</span></span>

    ```
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    <span data-ttu-id="74575-348">A kapcsolati karakterlánc a taxi-indításáról eseményközpont a **taxi-indításáról-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-348">The connection string for the taxi-ride event hub is the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="74575-349">A kapcsolati karakterláncot a taxi diszkont eseményközpont a **taxi-diszkont-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.</span><span class="sxs-lookup"><span data-stu-id="74575-349">The connection string for the taxi-fare event hub the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span>

3. <span data-ttu-id="74575-350">Futtassa a következő parancsot a Docker-rendszerkép létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="74575-350">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. <span data-ttu-id="74575-351">Lépjen vissza a szülőkönyvtárban `data/stream_azuredatabricks`.</span><span class="sxs-lookup"><span data-stu-id="74575-351">Navigate back to the parent directory, `data/stream_azuredatabricks`.</span></span>

    ```bash
    cd ..
    ```

5. <span data-ttu-id="74575-352">Futtassa a következő parancsot beírva futtassa a Docker-rendszerképet.</span><span class="sxs-lookup"><span data-stu-id="74575-352">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="74575-353">A kimenet a következőhöz hasonlóan kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="74575-353">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="74575-354">Ellenőrizze, hogy a Databricks-feladat megfelelően fut-e, nyissa meg az Azure Portalt, és keresse meg a Cosmos DB-adatbázist.</span><span class="sxs-lookup"><span data-stu-id="74575-354">To verify the Databricks job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="74575-355">Nyissa meg a **adatkezelő** panelen, és vizsgálja meg az adatok a **rekordok taxiköltség** tábla.</span><span class="sxs-lookup"><span data-stu-id="74575-355">Open the **Data Explorer** blade and examine the data in the **taxi records** table.</span></span> 

<span data-ttu-id="74575-356">[1] <span id="note1">Donovan, Brian; Munka, Dan (2016): New York City Taxi Útadatok (2010, 2013).</span><span class="sxs-lookup"><span data-stu-id="74575-356">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="74575-357">Egyetemi Illinois, Urbana-Champaignben.</span><span class="sxs-lookup"><span data-stu-id="74575-357">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="74575-358">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="74575-358">https://doi.org/10.13012/J8PN93H8</span></span>
