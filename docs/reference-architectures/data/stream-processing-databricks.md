---
title: Streamek feldolgozása az Azure Databricksszel
titleSuffix: Azure Reference Architectures
description: Hozzon létre egy teljes körű stream-feldolgozási folyamat az Azure-ban az Azure Databricks használatával.
author: petertaylor9999
ms.date: 11/30/2018
ms.custom: seodec18
ms.openlocfilehash: f7364334f889388ad432efadd46362a9fa82fe8b
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644121"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a>Az Azure Databricks egy adatfolyam-feldolgozási folyamat létrehozása

Ez a referenciaarchitektúra bemutatja egy teljes körű [adatfolyam-feldolgozás](/azure/architecture/data-guide/big-data/real-time-processing) folyamat. Ez a típusú folyamat négy fázisból áll: a betöltési, folyamat, tároló, és elemzés és jelentéskészítés. A referenciaarchitektúra a folyamat adatokat két forrásból fogadnak, illesztést hajt végre az egyes adatfolyamokkal kapcsolódó bejegyzések, bővíti az eredményt és kiszámítja az átlagos valós időben. Az eredmények tárolása további elemzés céljából. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Adatfolyam-feldolgozó az Azure Databricks a referencia-architektúra](./images/stream-processing-databricks.png)

**A forgatókönyv**: -I taxik vállalati minden taxi út adatokat gyűjt. Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre. A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság. Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi. A taxi vállalati részlege azt szeretné, kiszámítja az átlagos tipp mérföld alapuló, valós idejű, az egyes helyek száma, a helyszíni ridership trendeket.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

**Adatforrások**. Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást. Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz. A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza. Az adatforrások egy valós alkalmazásban lenne telepítve a taxi kabinetfájlok eszközök.

**Azure Event Hubs**. [Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás. Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ. Minden adatforráshoz társított eseményközpont adatfolyamot küld.

**Az Azure Databricks**. [Databricks](/azure/azure-databricks/) egy a Microsoft Azure cloud services platformra optimalizált Apache Spark-alapú elemzési platform. Databricks az taxi indításáról, összekapcsolását és adatok díjszabás és a helyek adatokat tárolja a Databricks fájlrendszerhez korrelatív adatokat feldúsítani használatos.

**A cosmos DB**. Az Azure Databricks-feladat kimenete rekordoknak, amelyek írt sorozatát [Cosmos DB](/azure/cosmos-db/) a Cassandra API-val. A Cassandra API támogatja a modellezés idősorozat-adatok használata.

**Az Azure Log Analytics**. Alkalmazás adatainak naplózása által gyűjtött [Azure Monitor](/azure/monitoring-and-diagnostics/) tárolja egy [Log Analytics-munkaterület](/azure/log-analytics). Log Analytics-lekérdezések segítségével elemezheti és metrikák megjelenítése elérhessék és vizsgálhassák azonosíthatja a problémákat az alkalmazásban lévő üzeneteket.

## <a name="data-ingestion"></a>Adatfeldolgozás

<!-- markdownlint-disable MD033 -->

Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>. Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz, négyéves időszakban (2010 &ndash; 2013). Kétféle típusú rekordot tartalmaz: Adatok ledolgozni, és adatokat díjszabás. Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz. Diszkont szerepel diszkont, adózási és tipp összegeket. Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját. Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot. Az adatok CSV formátumban tárolódik.

> [1] <span id="note1">Donovan, Brian; Munkahelyi, Dan (2016): New York City Taxi Útadatok (2010, 2013). Egyetemi Illinois, Urbana-Champaignben. <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást. A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el.

Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat. A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása. Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat. Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve.

Ebben a forgatókönyvben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl. Ez lehetővé teszi a Databricks párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal. Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.

![Az Azure Databricks és Event Hubs streamfeldolgozási ábrája](./images/stream-processing-databricks-eh.png)

Az adatgenerátor a common data model mindkét bejegyzéstípusokat rendelkezik egy `PartitionKey` összefűzésén tulajdonságot `Medallion`, `HackLicense`, és `VendorId`.

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

Ez a tulajdonság adja meg egy explicit partíciókulcsot az Event hubs szolgáltatásba küldésekor használható:

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a>Event Hubs

Az Event Hubs átviteli kapacitásának mérése [átviteli egységek](/azure/event-hubs/event-hubs-features#throughput-units). Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről](/azure/event-hubs/event-hubs-auto-inflate), amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.

## <a name="stream-processing"></a>Stream-feldolgozás

Az Azure Databricksben adatfeldolgozási feladatok hajtja végre. A feladat hozzá van rendelve, és a egy fürtön. A feladat lehet a Java és a egy Spark írt egyéni kód [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).

Ez a referenciaarchitektúra a feladat egy Java archív a Java- és Scala-ben írt osztályok. A Java-archívumot egy Databricks-feladat megadásakor az osztály a Databricks-fürt által végrehajtási van megadva. Itt a **fő** módszere a **com.microsoft.pnp.TaxiCabReader** osztály tartalmazza az adatok feldolgozási logikáját.

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a>A két event hub-példányok érkező adatfolyam olvasása

Az adatok feldolgozási logikáját használja [Spark strukturált streamelés](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) olvasni a két Azure event hub-példány:

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a>Az adatokat a helyek adatokat a bővítése

A indításáról adatok mentése a kivételezést szélességi és hosszúsági koordinátáit tartalmazza, és ki a helyek legördülő. Ezeket a koordinátákat hasznosak, amíg azok könnyen fel nem elemzés céljából. Ezért ezeket az adatokat a helyek adatokat, olvasási renderelésre egy [alakzatfájl](https://en.wikipedia.org/wiki/Shapefile).

Alakzatfájl formátuma bináris és nem egyszerűen elemzett, de a [GeoTools](http://geotools.org/) kódtár biztosít eszközöket a alakzatfájl formátumot használó térinformatikai adatok. Ebben a könyvtárban van használatban a **com.microsoft.pnp.GeoFinder** határozza meg az hálózatok nevét, a kivételezést alapján be és ki koordináták drop osztály.

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a>A indításáról és diszkont adatok csatlakoztatása

Először a indításáról és diszkont adatokat alakította át:

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

És a indításáról adatok diszkont adatokkal csatlakozik majd:

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a>Az adatok feldolgozását és a Cosmos DB beszúrása

Az egyes helyek átlagos diszkont összeg kiszámítása egy adott időtartam alatt:

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

Amely majd szúr be a Cosmos DB:

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a>Biztonsági szempontok

Az Azure Database munkaterület elérését szabályzatokkal vezérelhető a [felügyeleti konzol](https://docs.databricks.com/administration-guide/admin-settings/index.html). A felügyeleti konzolon adja hozzá a felhasználók, felhasználói engedélyek kezelése és egyszeri bejelentkezés beállítása funkcióit tartalmazza. Hozzáférés-vezérlés a munkaterületek, fürtök, feladatok és táblák is beállítható úgy, hogy a felügyeleti konzolon keresztül.

### <a name="managing-secrets"></a>Titkos kódok kezelése

Az Azure Databricks tartalmaz egy [titkoskód-tárolót](https://docs.azuredatabricks.net/user-guide/secrets/index.html) , amely titkos adatait, beleértve a kapcsolati karakterláncok, hozzáférési kulcsokat, felhasználónevek és jelszavak tárolására szolgál. Titkos kulcsok az Azure Databricks titkoskód-tárolót szerint vannak particionálva **hatókörök**:

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

Titkos kódok lettek hozzáadva a hatókör szintjén:

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> Egy Azure Key Vault biztonsági hatókört a natív Azure Databricks-hatókör helyett használható. További tudnivalókért lásd: [Azure Key Vault biztonsági hatókörök](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).

A kódban, titkos kulcsok az Azure Databricks-n keresztül elért [titkok segédprogramok](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).

## <a name="monitoring-considerations"></a>Felügyeleti szempontok

Az Azure Databricks az Apache Spark alapul, és mindkettő használja [log4j](https://logging.apache.org/log4j/2.x/) , a naplózáshoz standardní knihovna. Az Apache Spark által biztosított alapértelmezett naplózása mellett ez a referenciaarchitektúra küld naplókat és mérőszámokat [Azure Log Analytics](/azure/log-analytics/).

A **com.microsoft.pnp.TaxiCabReader** osztály az Apache Spark naplózási rendszer a naplók elküldése az Azure Log Analyticshez szereplő értékek használatával konfigurálja a **log4j.properties** fájlt. Az Apache Spark-naplózó üzenetek karakterláncok, amelyek az Azure Log Analytics naplóüzenetek JSON-ként formázott van szükség. A **com.microsoft.pnp.log4j.LogAnalyticsAppender** osztály alakítja át, JSON ezeket az üzeneteket:

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

Mint a **com.microsoft.pnp.TaxiCabReader** osztály indításáról és diszkont üzeneteket dolgoz fel. lehetséges, vagy lehetnek helytelen formátumú, és ezért nem érvényes. Éles környezetben fontos elemzése alapján azonosítja a probléma adódott az adatforrásokat, ezért gyorsan kell rögzíteni adatvesztés elkerülése érdekében ezen helytelenül formázott üzeneteket. A **com.microsoft.pnp.TaxiCabReader** osztály regisztrál egy Apache Spark akkumulátor, amely nyomon követi a helytelen formátumú diszkont és indításáról rekordok száma:

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

Az Apache Spark mérőszámok küldése Dropwizard kódtárat használja, és a natív Dropwizard metrikák mezők némelyike nem kompatibilis az Azure Log Analytics. Ezért ez a referenciaarchitektúra egy egyéni Dropwizard fogadó és riporter tartalmazza. A metrikák az Azure Log Analytics által elvárt formátumú formázza azt. Ha az Apache Spark-metrikák jelentések az egyéni metrikákat a helytelen formátumú indításáról és diszkont adatok is küld.

Az utolsó metrikát, be kell jelentkeznie az Azure Log Analytics-munkaterülethez összesített végrehajtási állapotát a Spark strukturált Stream feladat előrehaladását. Ebben az esetben egy egyéni StreamingQuery használatával figyelő végrehajtani a **com.microsoft.pnp.StreamingMetricsListener** osztály. Ez az osztály regisztrálva van az Apache Spark-munkamenetet a feladat futtatásakor:

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

A StreamingMetricsListener módszerei által meghívott az Apache Spark-futtatókörnyezet, amikor egy strukturált streamelési esemény bekövetkezésekor a küldő jelentkezzen üzenetek és mérőszámok az Azure Log Analytics-munkaterületet. A következő lekérdezéseket a munkaterület segítségével az alkalmazás figyelése:

### <a name="latency-and-throughput-for-streaming-queries"></a>Lekérdezések folyamatos átviteli teljesítmény és a késés

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a>Stream-lekérdezés végrehajtása során naplózott kivételek

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Helytelen formátumú diszkont és indításáról adatok felhalmozódása

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

### <a name="job-execution-to-trace-resiliency"></a>Nyomkövetési rugalmasság feladat futtatása

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A telepítés, és futtassa a referenciaimplementációt, kövesse a lépéseket a [GitHub információs](https://github.com/mspnp/azure-databricks-streaming-analytics).
