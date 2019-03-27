---
title: Streamek feldolgozása az Azure Databricksszel
titleSuffix: Azure Reference Architectures
description: Hozzon létre egy teljes körű stream-feldolgozási folyamat az Azure-ban az Azure Databricks használatával.
author: petertaylor9999
ms.date: 11/30/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 748b191aeee931d580dd27b1ad54c4f4bd63e369
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298889"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="f03f7-103">Az Azure Databricks egy adatfolyam-feldolgozási folyamat létrehozása</span><span class="sxs-lookup"><span data-stu-id="f03f7-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="f03f7-104">Ez a referenciaarchitektúra bemutatja egy teljes körű [adatfolyam-feldolgozás](/azure/architecture/data-guide/big-data/real-time-processing) folyamat.</span><span class="sxs-lookup"><span data-stu-id="f03f7-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="f03f7-105">Ez a típusú folyamat négy fázisból áll: a betöltési, folyamat, tároló, és elemzés és jelentéskészítés.</span><span class="sxs-lookup"><span data-stu-id="f03f7-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="f03f7-106">A referenciaarchitektúra a folyamat adatokat két forrásból fogadnak, illesztést hajt végre az egyes adatfolyamokkal kapcsolódó bejegyzések, bővíti az eredményt és kiszámítja az átlagos valós időben.</span><span class="sxs-lookup"><span data-stu-id="f03f7-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="f03f7-107">Az eredmények tárolása további elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="f03f7-107">The results are stored for further analysis.</span></span> <span data-ttu-id="f03f7-108">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="f03f7-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Adatfolyam-feldolgozó az Azure Databricks a referencia-architektúra](./images/stream-processing-databricks.png)

<span data-ttu-id="f03f7-110">**A forgatókönyv**: -I taxik vállalati minden taxi út adatokat gyűjt.</span><span class="sxs-lookup"><span data-stu-id="f03f7-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="f03f7-111">Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre.</span><span class="sxs-lookup"><span data-stu-id="f03f7-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="f03f7-112">A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság.</span><span class="sxs-lookup"><span data-stu-id="f03f7-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="f03f7-113">Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi.</span><span class="sxs-lookup"><span data-stu-id="f03f7-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="f03f7-114">A taxi vállalati részlege azt szeretné, kiszámítja az átlagos tipp mérföld alapuló, valós idejű, az egyes helyek száma, a helyszíni ridership trendeket.</span><span class="sxs-lookup"><span data-stu-id="f03f7-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="f03f7-115">Architektúra</span><span class="sxs-lookup"><span data-stu-id="f03f7-115">Architecture</span></span>

<span data-ttu-id="f03f7-116">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="f03f7-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="f03f7-117">**Adatforrások**.</span><span class="sxs-lookup"><span data-stu-id="f03f7-117">**Data sources**.</span></span> <span data-ttu-id="f03f7-118">Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást.</span><span class="sxs-lookup"><span data-stu-id="f03f7-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="f03f7-119">Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="f03f7-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="f03f7-120">A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="f03f7-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="f03f7-121">Az adatforrások egy valós alkalmazásban lenne telepítve a taxi kabinetfájlok eszközök.</span><span class="sxs-lookup"><span data-stu-id="f03f7-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="f03f7-122">**Azure Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="f03f7-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="f03f7-123">[Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="f03f7-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="f03f7-124">Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ.</span><span class="sxs-lookup"><span data-stu-id="f03f7-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="f03f7-125">Minden adatforráshoz társított eseményközpont adatfolyamot küld.</span><span class="sxs-lookup"><span data-stu-id="f03f7-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="f03f7-126">**Az Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="f03f7-126">**Azure Databricks**.</span></span> <span data-ttu-id="f03f7-127">[Databricks](/azure/azure-databricks/) egy a Microsoft Azure cloud services platformra optimalizált Apache Spark-alapú elemzési platform.</span><span class="sxs-lookup"><span data-stu-id="f03f7-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="f03f7-128">Databricks az taxi indításáról, összekapcsolását és adatok díjszabás és a helyek adatokat tárolja a Databricks fájlrendszerhez korrelatív adatokat feldúsítani használatos.</span><span class="sxs-lookup"><span data-stu-id="f03f7-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="f03f7-129">**A cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="f03f7-129">**Cosmos DB**.</span></span> <span data-ttu-id="f03f7-130">Az Azure Databricks-feladat kimenete rekordoknak, amelyek írt sorozatát [Cosmos DB](/azure/cosmos-db/) a Cassandra API-val.</span><span class="sxs-lookup"><span data-stu-id="f03f7-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="f03f7-131">A Cassandra API támogatja a modellezés idősorozat-adatok használata.</span><span class="sxs-lookup"><span data-stu-id="f03f7-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="f03f7-132">**Az Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="f03f7-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="f03f7-133">Alkalmazás adatainak naplózása által gyűjtött [Azure Monitor](/azure/monitoring-and-diagnostics/) tárolja egy [Log Analytics-munkaterület](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="f03f7-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="f03f7-134">Log Analytics-lekérdezések segítségével elemezheti és metrikák megjelenítése elérhessék és vizsgálhassák azonosíthatja a problémákat az alkalmazásban lévő üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="f03f7-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="f03f7-135">Adatfeldolgozás</span><span class="sxs-lookup"><span data-stu-id="f03f7-135">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="f03f7-136">Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>.</span><span class="sxs-lookup"><span data-stu-id="f03f7-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="f03f7-137">Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz, négyéves időszakban (2010 &ndash; 2013).</span><span class="sxs-lookup"><span data-stu-id="f03f7-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="f03f7-138">Kétféle típusú rekordot tartalmaz: Adatok ledolgozni, és adatokat díjszabás.</span><span class="sxs-lookup"><span data-stu-id="f03f7-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="f03f7-139">Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="f03f7-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="f03f7-140">Diszkont szerepel diszkont, adózási és tipp összegeket.</span><span class="sxs-lookup"><span data-stu-id="f03f7-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="f03f7-141">Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját.</span><span class="sxs-lookup"><span data-stu-id="f03f7-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="f03f7-142">Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot.</span><span class="sxs-lookup"><span data-stu-id="f03f7-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="f03f7-143">Az adatok CSV formátumban tárolódik.</span><span class="sxs-lookup"><span data-stu-id="f03f7-143">The data is stored in CSV format.</span></span>

> <span data-ttu-id="f03f7-144">[1] <span id="note1">Donovan, Brian; Munkahelyi, Dan (2016): New York City Taxi Útadatok (2010, 2013).</span><span class="sxs-lookup"><span data-stu-id="f03f7-144">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="f03f7-145">Egyetemi Illinois, Urbana-Champaignben.</span><span class="sxs-lookup"><span data-stu-id="f03f7-145">University of Illinois at Urbana-Champaign.</span></span> <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="f03f7-146">Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="f03f7-146">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="f03f7-147">A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el.</span><span class="sxs-lookup"><span data-stu-id="f03f7-147">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="f03f7-148">Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat.</span><span class="sxs-lookup"><span data-stu-id="f03f7-148">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="f03f7-149">A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása.</span><span class="sxs-lookup"><span data-stu-id="f03f7-149">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="f03f7-150">Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat.</span><span class="sxs-lookup"><span data-stu-id="f03f7-150">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="f03f7-151">Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="f03f7-151">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="f03f7-152">Ebben a forgatókönyvben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl.</span><span class="sxs-lookup"><span data-stu-id="f03f7-152">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="f03f7-153">Ez lehetővé teszi a Databricks párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal.</span><span class="sxs-lookup"><span data-stu-id="f03f7-153">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="f03f7-154">Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.</span><span class="sxs-lookup"><span data-stu-id="f03f7-154">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Az Azure Databricks és Event Hubs streamfeldolgozási ábrája](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="f03f7-156">Az adatgenerátor a common data model mindkét bejegyzéstípusokat rendelkezik egy `PartitionKey` összefűzésén tulajdonságot `Medallion`, `HackLicense`, és `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="f03f7-156">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="f03f7-157">Ez a tulajdonság adja meg egy explicit partíciókulcsot az Event hubs szolgáltatásba küldésekor használható:</span><span class="sxs-lookup"><span data-stu-id="f03f7-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="f03f7-158">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="f03f7-158">Event Hubs</span></span>

<span data-ttu-id="f03f7-159">Az Event Hubs átviteli kapacitásának mérése [átviteli egységek](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="f03f7-159">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="f03f7-160">Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről](/azure/event-hubs/event-hubs-auto-inflate), amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.</span><span class="sxs-lookup"><span data-stu-id="f03f7-160">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="f03f7-161">Stream-feldolgozás</span><span class="sxs-lookup"><span data-stu-id="f03f7-161">Stream processing</span></span>

<span data-ttu-id="f03f7-162">Az Azure Databricksben adatfeldolgozási feladatok hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="f03f7-162">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="f03f7-163">A feladat hozzá van rendelve, és a egy fürtön.</span><span class="sxs-lookup"><span data-stu-id="f03f7-163">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="f03f7-164">A feladat lehet a Java és a egy Spark írt egyéni kód [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span><span class="sxs-lookup"><span data-stu-id="f03f7-164">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="f03f7-165">Ez a referenciaarchitektúra a feladat egy Java archív a Java- és Scala-ben írt osztályok.</span><span class="sxs-lookup"><span data-stu-id="f03f7-165">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="f03f7-166">A Java-archívumot egy Databricks-feladat megadásakor az osztály a Databricks-fürt által végrehajtási van megadva.</span><span class="sxs-lookup"><span data-stu-id="f03f7-166">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="f03f7-167">Itt a **fő** módszere a **com.microsoft.pnp.TaxiCabReader** osztály tartalmazza az adatok feldolgozási logikáját.</span><span class="sxs-lookup"><span data-stu-id="f03f7-167">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="f03f7-168">A két event hub-példányok érkező adatfolyam olvasása</span><span class="sxs-lookup"><span data-stu-id="f03f7-168">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="f03f7-169">Az adatok feldolgozási logikáját használja [Spark strukturált streamelés](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) olvasni a két Azure event hub-példány:</span><span class="sxs-lookup"><span data-stu-id="f03f7-169">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="f03f7-170">Az adatokat a helyek adatokat a bővítése</span><span class="sxs-lookup"><span data-stu-id="f03f7-170">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="f03f7-171">A indításáról adatok mentése a kivételezést szélességi és hosszúsági koordinátáit tartalmazza, és ki a helyek legördülő.</span><span class="sxs-lookup"><span data-stu-id="f03f7-171">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="f03f7-172">Ezeket a koordinátákat hasznosak, amíg azok könnyen fel nem elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="f03f7-172">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="f03f7-173">Ezért ezeket az adatokat a helyek adatokat, olvasási renderelésre egy [alakzatfájl](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="f03f7-173">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="f03f7-174">Alakzatfájl formátuma bináris és nem egyszerűen elemzett, de a [GeoTools](http://geotools.org/) kódtár biztosít eszközöket a alakzatfájl formátumot használó térinformatikai adatok.</span><span class="sxs-lookup"><span data-stu-id="f03f7-174">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="f03f7-175">Ebben a könyvtárban van használatban a **com.microsoft.pnp.GeoFinder** határozza meg az hálózatok nevét, a kivételezést alapján be és ki koordináták drop osztály.</span><span class="sxs-lookup"><span data-stu-id="f03f7-175">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="f03f7-176">A indításáról és diszkont adatok csatlakoztatása</span><span class="sxs-lookup"><span data-stu-id="f03f7-176">Joining the ride and fare data</span></span>

<span data-ttu-id="f03f7-177">Először a indításáról és diszkont adatokat alakította át:</span><span class="sxs-lookup"><span data-stu-id="f03f7-177">First the ride and fare data is transformed:</span></span>

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

<span data-ttu-id="f03f7-178">És a indításáról adatok diszkont adatokkal csatlakozik majd:</span><span class="sxs-lookup"><span data-stu-id="f03f7-178">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="f03f7-179">Az adatok feldolgozását és a Cosmos DB beszúrása</span><span class="sxs-lookup"><span data-stu-id="f03f7-179">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="f03f7-180">Az egyes helyek átlagos diszkont összeg kiszámítása egy adott időtartam alatt:</span><span class="sxs-lookup"><span data-stu-id="f03f7-180">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

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

<span data-ttu-id="f03f7-181">Amely majd szúr be a Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="f03f7-181">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="f03f7-182">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="f03f7-182">Security considerations</span></span>

<span data-ttu-id="f03f7-183">Az Azure Database munkaterület elérését szabályzatokkal vezérelhető a [felügyeleti konzol](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="f03f7-183">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="f03f7-184">A felügyeleti konzolon adja hozzá a felhasználók, felhasználói engedélyek kezelése és egyszeri bejelentkezés beállítása funkcióit tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="f03f7-184">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="f03f7-185">Hozzáférés-vezérlés a munkaterületek, fürtök, feladatok és táblák is beállítható úgy, hogy a felügyeleti konzolon keresztül.</span><span class="sxs-lookup"><span data-stu-id="f03f7-185">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="f03f7-186">Titkos kódok kezelése</span><span class="sxs-lookup"><span data-stu-id="f03f7-186">Managing secrets</span></span>

<span data-ttu-id="f03f7-187">Az Azure Databricks tartalmaz egy [titkoskód-tárolót](https://docs.azuredatabricks.net/user-guide/secrets/index.html) , amely titkos adatait, beleértve a kapcsolati karakterláncok, hozzáférési kulcsokat, felhasználónevek és jelszavak tárolására szolgál.</span><span class="sxs-lookup"><span data-stu-id="f03f7-187">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="f03f7-188">Titkos kulcsok az Azure Databricks titkoskód-tárolót szerint vannak particionálva **hatókörök**:</span><span class="sxs-lookup"><span data-stu-id="f03f7-188">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="f03f7-189">Titkos kódok lettek hozzáadva a hatókör szintjén:</span><span class="sxs-lookup"><span data-stu-id="f03f7-189">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="f03f7-190">Egy Azure Key Vault biztonsági hatókört a natív Azure Databricks-hatókör helyett használható.</span><span class="sxs-lookup"><span data-stu-id="f03f7-190">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="f03f7-191">További tudnivalókért lásd: [Azure Key Vault biztonsági hatókörök](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="f03f7-191">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="f03f7-192">A kódban, titkos kulcsok az Azure Databricks-n keresztül elért [titkok segédprogramok](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span><span class="sxs-lookup"><span data-stu-id="f03f7-192">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="f03f7-193">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="f03f7-193">Monitoring considerations</span></span>

<span data-ttu-id="f03f7-194">Az Azure Databricks az Apache Spark alapul, és mindkettő használja [log4j](https://logging.apache.org/log4j/2.x/) , a naplózáshoz standardní knihovna.</span><span class="sxs-lookup"><span data-stu-id="f03f7-194">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="f03f7-195">Az Apache Spark által biztosított alapértelmezett naplózása mellett ez a referenciaarchitektúra küld naplókat és mérőszámokat [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="f03f7-195">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="f03f7-196">A **com.microsoft.pnp.TaxiCabReader** osztály az Apache Spark naplózási rendszer a naplók elküldése az Azure Log Analyticshez szereplő értékek használatával konfigurálja a **log4j.properties** fájlt.</span><span class="sxs-lookup"><span data-stu-id="f03f7-196">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="f03f7-197">Az Apache Spark-naplózó üzenetek karakterláncok, amelyek az Azure Log Analytics naplóüzenetek JSON-ként formázott van szükség.</span><span class="sxs-lookup"><span data-stu-id="f03f7-197">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="f03f7-198">A **com.microsoft.pnp.log4j.LogAnalyticsAppender** osztály alakítja át, JSON ezeket az üzeneteket:</span><span class="sxs-lookup"><span data-stu-id="f03f7-198">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

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

<span data-ttu-id="f03f7-199">Mint a **com.microsoft.pnp.TaxiCabReader** osztály indításáról és diszkont üzeneteket dolgoz fel. lehetséges, vagy lehetnek helytelen formátumú, és ezért nem érvényes.</span><span class="sxs-lookup"><span data-stu-id="f03f7-199">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="f03f7-200">Éles környezetben fontos elemzése alapján azonosítja a probléma adódott az adatforrásokat, ezért gyorsan kell rögzíteni adatvesztés elkerülése érdekében ezen helytelenül formázott üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="f03f7-200">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="f03f7-201">A **com.microsoft.pnp.TaxiCabReader** osztály regisztrál egy Apache Spark akkumulátor, amely nyomon követi a helytelen formátumú diszkont és indításáról rekordok száma:</span><span class="sxs-lookup"><span data-stu-id="f03f7-201">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="f03f7-202">Az Apache Spark mérőszámok küldése Dropwizard kódtárat használja, és a natív Dropwizard metrikák mezők némelyike nem kompatibilis az Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="f03f7-202">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="f03f7-203">Ezért ez a referenciaarchitektúra egy egyéni Dropwizard fogadó és riporter tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="f03f7-203">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="f03f7-204">A metrikák az Azure Log Analytics által elvárt formátumú formázza azt.</span><span class="sxs-lookup"><span data-stu-id="f03f7-204">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="f03f7-205">Ha az Apache Spark-metrikák jelentések az egyéni metrikákat a helytelen formátumú indításáról és diszkont adatok is küld.</span><span class="sxs-lookup"><span data-stu-id="f03f7-205">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="f03f7-206">Az utolsó metrikát, be kell jelentkeznie az Azure Log Analytics-munkaterülethez összesített végrehajtási állapotát a Spark strukturált Stream feladat előrehaladását.</span><span class="sxs-lookup"><span data-stu-id="f03f7-206">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="f03f7-207">Ebben az esetben egy egyéni StreamingQuery használatával figyelő végrehajtani a **com.microsoft.pnp.StreamingMetricsListener** osztály.</span><span class="sxs-lookup"><span data-stu-id="f03f7-207">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="f03f7-208">Ez az osztály regisztrálva van az Apache Spark-munkamenetet a feladat futtatásakor:</span><span class="sxs-lookup"><span data-stu-id="f03f7-208">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="f03f7-209">A StreamingMetricsListener módszerei által meghívott az Apache Spark-futtatókörnyezet, amikor egy strukturált streamelési esemény bekövetkezésekor a küldő jelentkezzen üzenetek és mérőszámok az Azure Log Analytics-munkaterületet.</span><span class="sxs-lookup"><span data-stu-id="f03f7-209">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="f03f7-210">A következő lekérdezéseket a munkaterület segítségével az alkalmazás figyelése:</span><span class="sxs-lookup"><span data-stu-id="f03f7-210">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="f03f7-211">Lekérdezések folyamatos átviteli teljesítmény és a késés</span><span class="sxs-lookup"><span data-stu-id="f03f7-211">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="f03f7-212">Stream-lekérdezés végrehajtása során naplózott kivételek</span><span class="sxs-lookup"><span data-stu-id="f03f7-212">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="f03f7-213">Helytelen formátumú diszkont és indításáról adatok felhalmozódása</span><span class="sxs-lookup"><span data-stu-id="f03f7-213">Accumulation of malformed fare and ride data</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedrides"

SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedfares"
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="f03f7-214">Nyomkövetési rugalmasság feladat futtatása</span><span class="sxs-lookup"><span data-stu-id="f03f7-214">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a><span data-ttu-id="f03f7-215">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="f03f7-215">Deploy the solution</span></span>

<span data-ttu-id="f03f7-216">A telepítés, és futtassa a referenciaimplementációt, kövesse a lépéseket a [GitHub információs](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="f03f7-216">To the deploy and run the reference implementation, follow the steps in the [GitHub readme](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>
