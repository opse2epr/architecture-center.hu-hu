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
# <a name="stream-processing-with-azure-databricks"></a>Az Azure Databricks feldolgozó Stream

Ez a referenciaarchitektúra bemutatja egy teljes körű [adatfolyam-feldolgozás](/azure/architecture/data-guide/big-data/real-time-processing) folyamat. Ez a típusú folyamat négy fázisból áll: a betöltési, folyamat, tároló, és elemzés és jelentéskészítés. A referenciaarchitektúra a folyamat adatokat két forrásból fogadnak, illesztést hajt végre az egyes adatfolyamokkal kapcsolódó bejegyzések, bővíti az eredményt és kiszámítja az átlagos valós időben. Az eredmények tárolása további elemzés céljából. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

**A forgatókönyv**:-i taxik vállalati minden taxi út adatokat gyűjt. Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre. A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság. Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi. A taxi vállalati részlege azt szeretné, kiszámítja az átlagos tipp mérföld alapuló, valós idejű, az egyes helyek száma, a helyszíni ridership trendeket.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

**Adatforrások**. Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást. Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz. A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza. Az adatforrások egy valós alkalmazásban lenne telepítve a taxi kabinetfájlok eszközök.

**Azure Event Hubs**. [Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás. Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ. Minden adatforráshoz társított eseményközpont adatfolyamot küld.

**Az Azure Databricks**. [Databricks](/azure/azure-databricks/) egy a Microsoft Azure cloud services platformra optimalizált Apache Spark-alapú elemzési platform. Databricks az taxi indításáról, összekapcsolását és adatok díjszabás és a helyek adatokat tárolja a Databricks fájlrendszerhez korrelatív adatokat feldúsítani használatos.

**A cosmos DB**. Az Azure Databricks-feladat kimenete rekordoknak, amelyek írt sorozatát [Cosmos DB](/azure/cosmos-db/) a Cassandra API-val. A Cassandra API támogatja a modellezés idősorozat-adatok használata.

**Az Azure Log Analytics**. Alkalmazás adatainak naplózása által gyűjtött [Azure Monitor](/azure/monitoring-and-diagnostics/) tárolja egy [Log Analytics-munkaterület](/azure/log-analytics). Log Analytics-lekérdezések segítségével elemezheti és metrikák megjelenítése elérhessék és vizsgálhassák azonosíthatja a problémákat az alkalmazásban lévő üzeneteket.

## <a name="data-ingestion"></a>Adatfeldolgozás

Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>. Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz, négyéves időszakban (2010 &ndash; 2013). Kétféle típusú rekordot tartalmaz: indításáról és diszkont adatait. Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz. Diszkont szerepel diszkont, adózási és tipp összegeket. Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját. Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot. Az adatok CSV formátumban tárolódik. 

Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást. A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el. 

Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat. A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása. Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat. Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve. 

Ebben a forgatókönyvben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl. Ez lehetővé teszi a Databricks párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal. Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.

![](./images/stream-processing-databricks-eh.png)

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

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d  
| render timechart
``` 
### <a name="exceptions-logged-during-stream-query-execution"></a>Stream-lekérdezés végrehajtása során naplózott kivételek

```
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error" 
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Helytelen formátumú diszkont és indításáról adatok felhalmozódása

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

### <a name="job-execution-to-trace-resiliency"></a>Nyomkövetési rugalmasság feladat futtatása
```
SparkMetric_CL 
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart 
| where name_s contains "driver.DAGScheduler.job.allJobs" 
```

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra egy üzemelő példánya érhető el az [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data). 

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy töltse le a zip-fájlját a [referenciaarchitektúrákat](https://github.com/mspnp/reference-architectures) GitHub-adattárban.

2. Telepítés [Docker](https://www.docker.com/) futtatásához az adatgenerálást.

3. Telepítés [az Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

4. Telepítés [a Databricks parancssori felület](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).

5. Parancsot a parancssorba bash-parancssorból vagy PowerShell-parancssorból, jelentkezzen be Azure-fiókjába a következő:
    ```
    az login
    ```
6. Telepítse a Java ide Környezethez, a következő források:
    - JDK 1.8-AS VERZIÓJA
    - SDK-t Scala 2.11-et
    - Maven 3.5.4

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a>A New York City taxi és hálózatok adatfájlok le

1. Hozzon létre egy könyvtárat nevű `DataFile` alatt a `data/streaming_azuredatabricks` könyvtárat a helyi fájlrendszerben.

2. Nyisson meg egy webböngészőt, és navigáljon a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.

3. Kattintson a **letöltése** gombra ezen az oldalon a megfelelő évnek a taxi-adatok egy zip-fájl letöltéséhez.

4. Bontsa ki a zip-fájlt a `DataFile` könyvtár.

    > [!NOTE]
    > Ez a tömörített fájl más zip-fájlokat tartalmazza. A gyermek zip-fájlokat nem csomagolja ki.

    A directory-struktúra az alábbiakhoz hasonlóan kell kinéznie:

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

5. Nyisson meg egy webböngészőt, és navigáljon a https://www.zillow.com/howto/api/neighborhood-boundaries.htm. 

6. Kattintson a **New York-i helyek határait** letölteni a fájlt.

7. Másolás a **ZillowNeighborhoods-NY.zip** fájlt a böngészőben a **letölti** mappában a `DataFile` könyvtár.

### <a name="deploy-the-azure-resources"></a>Az Azure-erőforrások üzembe helyezése

1. A rendszerhéj vagy a Windows-parancssort futtassa a következő parancsot, és kövesse a bejelentkezési kérések jelenhetnek:

    ```bash
    az login
    ```

2. Lépjen abba a mappába `data/streaming_azuredatabricks` a GitHub-adattárban

    ```bash
    cd data/streaming_azuredatabricks
    ```

3. Futtassa az alábbi parancsokat az Azure-erőforrások üzembe helyezéséhez:

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

4. A kimenet az üzembe helyezés befejezése után a konzol íródik. A kimenetben keresse meg a következő JSON-ra:

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
Ezeket az értékeket a titkos kulcsok, amely belekerül a későbbi szakaszokban a Databricks titkos kódokhoz való. Megőrizheti azok biztonságát mindaddig, amíg szakaszt hozzáadhatja őket.

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a>A Cosmos DB-fiók egy Cassandra-tábla hozzáadása

1. Az Azure Portalon lépjen a létrehozott erőforráscsoportot a **üzembe helyezése az Azure-erőforrások** című fenti szakaszban. Kattintson a **az Azure Cosmos DB-fiók**. Hozzon létre egy táblát a Cassandra API-val.

2. Az a **áttekintése** panelen kattintson a **tábla hozzáadása**.

3. Ha a **tábla hozzáadása** panel nyílik meg, adja meg `newyorktaxi` a a **Kulcstér neve** szövegmezőben. 

4. Az a **adja meg a tábla létrehozásához CQL parancsot** területén adja meg `neighborhoodstats` mellett a szövegmezőben `newyorktaxi`.

5. Az alábbi szövegmezőbe írja be a következőket:
```
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. Az a **átviteli sebesség (1000 – 1 000 000 RU/s)** szövegmezőbe írja be az értéket `4000`.

7. Kattintson az **OK** gombra.

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a>A Databricks parancssori felület használatával a Databricks-titkos kulcsok hozzáadása

Először adja meg a titkos kulcsok EventHub:

1. Használatával a **Azure Databricks parancssori felület** telepítette a 2. lépésben előfeltétel, az Azure Databricks titkos hatókör létrehozása:
    ```
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. Adja hozzá a titkos kulcsot a taxi indításáról EventHub:
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    Miután hajtja végre, a vi-szerkesztő megnyitása. Adja meg a **taxi-indításáról-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban. Mentse és vi kilép.

3. Adja hozzá a titkos kulcsot a taxi diszkont EventHub:
    ```
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    Miután hajtja végre, a vi-szerkesztő megnyitása. Adja meg a **taxi-diszkont-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban. Mentse és vi kilép.

Írja be a titkos kulcsok a Cosmos DB:

1. Nyissa meg az Azure Portalon, és keresse meg a 3. lépésében megadott erőforráscsoportot a **üzembe helyezése az Azure-erőforrások** szakaszban. Kattintson az Azure Cosmos DB-fiók.

2. Használatával a **Azure Databricks parancssori felület**, adja hozzá a titkos kulcsot, a Cosmos DB-felhasználónév:
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
Miután hajtja végre, a vi-szerkesztő megnyitása. Adja meg a **felhasználónév** értéket a **CosmosDb** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban. Mentse és vi kilép.

3. Ezután adja hozzá a titkos kulcsot, a Cosmos DB-jelszó:
    ```
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

Miután hajtja végre, a vi-szerkesztő megnyitása. Adja meg a **titkos** értéket a **CosmosDb** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban. Mentse és vi kilép.

> [!NOTE]
> Ha használ egy [titkos hatókör Azure Key Vault-alapú](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), névvel kell rendelkeznie a hatókör **azure databricks-feladat** és a titkos kulcsok rendelkeznie kell a fent említett pontos megegyező nevekkel.

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a>Adja hozzá a Zillow környékeken adatokat a Databricks fájlrendszerhez

1. Hozzon létre egy könyvtárat a Databricks fájlrendszerhez:
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. Adatok/streaming_azuredatabricks/adatfájlját keresse meg, és adja meg a következőket:
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a>Az Azure Log Analytics-munkaterület Azonosítójára és az elsődleges kulcs hozzáadása a konfigurációs fájlok

Ebben a szakaszban van szüksége a Log Analytics-munkaterület Azonosítójára és az elsődleges kulcsot. A munkaterület-azonosító a **munkaterület azonosítója** értéket a **logAnalytics** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban. Az elsődleges kulcs a **titkos** a kimeneti szakaszban. 

1. Nyissa meg a data\streaming_azuredatabricks\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties log4j naplózás beállításához. A következő két érték szerkesztése:
    ```
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. Egyéni naplózás beállításához nyissa meg a data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties. A következő két érték szerkesztése:
    ``` 
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a>Hozhat létre a .jar fájlokat a Databricks-feladat és a Databricks-figyelés

1. A Java IDE használatával importálhatja a nevű Maven-projektfájlból **pom.xml** gyökerében található a **data/streaming_azuredatabricks** könyvtár. 

2. Hajtsa végre a tiszta buildjének kiépítéséhez. A build kimenete nevű fájlt a **azure-databricks-feladat – 1.0-SNAPSHOT.jar** és **azure-databricks-figyelés – 0.9.jar**. 

### <a name="configure-custom-logging-for-the-databricks-job"></a>A Databricks-feladat az egyéni naplózás konfigurálása

1. Másolás a **azure-databricks-figyelés – 0.9.jar** fájlt a következő parancs beírásával a Databricks fájlrendszerhez a **a Databricks parancssori felület**:
    ```
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. Másolja az egyéni naplózás tulajdonságai a data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\metrics.properties a Databricks fájlrendszerhez a következő parancs beírásával:
    ```
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. Bár még a Databricks-fürt nevét, még nem úgy döntött, válasszon ki egyet. A fürt neve alatt fogja meg a Databricks fájlrendszerbeli elérési út. Másolja a inicializálása a data\streaming_azuredatabricks\azure\azure-databricks-monitoring\scripts\spark.metrics a Databricks fájlrendszerhez a következő parancs beírásával:
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a>Databricks-fürt létrehozása

1. A Databricks munkaterületen kattintson a "Fürt", majd kattintson a "létrehozása a fürt". Adja meg a 3. lépésében létrehozott fürt neve a **a Databricks-feladat az egyéni naplózás konfigurálása** című fenti szakaszban. 

2. Válassza ki a **standard** fürt üzemmód.

3. Állítsa be **Databricks futtatókörnyezet-verziója** való **4.3-as (tartalmazza az Apache Spark 2.3.1, Scala 2.11-et)**

4. Állítsa be **Python-verzió** való **2**.

5. Állítsa be **illesztőprogram típusa** való **ugyanaz, mint a feldolgozó**

6. Állítsa be **feldolgozó típusa** való **Standard_DS3_v2**.

7. Állítsa be **min. feldolgozók** való **2**.

8. Kapcsolja ki **automatikus skálázás engedélyezéséhez**. 

9. Alább a **automatikus Bezárás** párbeszédpanelen kattintson a **Init parancsfájlok**. 

10. Adja meg **dbfs: / databricks/init/< fürtnév > / spark-metrics.sh**, és cserélje le a fürt neve a < fürtnév > 1. lépésben létrehozott.

11. Kattintson a **Hozzáadás** gombra.

12. Kattintson a **-fürt létrehozása** gombra.

### <a name="create-a-databricks-job"></a>Hozzon létre egy Databricks-feladat

1. A Databricks munkaterületen kattintson a "Jobs", "a feladat létrehozása".

2. Adja meg a feladat nevét.

3. Kattintson a "set jar", megnyílik a "Feltöltés JAR futtassa a" párbeszédpanel.

4. Húzza a **azure-databricks-feladat – 1.0-SNAPSHOT.jar** létrehozott fájlt a **hozhat létre a .JAR fájlt a Databricks-feladat** részt a **itt dobja el JAR feltöltéséhez** mezőbe.

5. Adja meg **com.microsoft.pnp.TaxiCabReader** a a **Main osztály** mező.

6. Az argumentumok mezőben adja meg a következőket:
    ```
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. A függő kódtárak telepítéséhez az alábbi lépéseket:
    
    1. A Databricks felhasználói felületén kattintson a a **otthoni** gombra.
    
    2. Az a **felhasználók** legördülő, a felhasználói fiók nevére kattintva nyissa meg a fiók munkaterület beállításait.
    
    3. A legördülő nyílra, a fiók neve melletti jelölőnégyzetet, kattintson a **létrehozása**, majd kattintson a **könyvtár** megnyitásához a **új könyvtár** párbeszédpanel.
    
    4. Az a **forrás** legördülő vezérlőt, jelölje be **Maven-koordináta**.
    
    5. Alatt a **telepítése Maven-összetevőkben** fejléc adja meg `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` a a **koordinálja** szövegmezőben. 
    
    6. Kattintson a **könyvtár létrehozása** megnyitásához a **összetevők** ablak.
    
    7. A **állapotának a futó fürtök** ellenőrizze a **automatikusan csatolja az összes fürt** jelölőnégyzetet.
    
    8. Ismételje meg az 1-7 a lépéseket a `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven-koordináta.
    
    9. Ismételje meg az 1 – 6. lépéseket a `org.geotools:gt-shapefile:19.2` Maven-koordináta.
    
    10. Kattintson a **speciális beállítások**.
    
    11. Adja meg `http://download.osgeo.org/webdav/geotools/` a a **tárház** szövegmezőben. 
    
    12. Kattintson a **könyvtár létrehozása** megnyitásához a **összetevők** ablak. 
    
    13. A **állapotának a futó fürtök** ellenőrizze a **automatikusan csatolja az összes fürt** jelölőnégyzetet.

8. A 7. lépésben létrehozott 6. lépés végén található a feladathoz hozzáadja a függő kódtárak hozzáadása:
    1. Az Azure Databricks-munkaterületet, kattintson a **feladatok**.

    2. A feladat nevére, a 2. lépésben létrehozott a **hozzon létre egy Databricks-feladat** szakaszban. 
    
    3. Mellett a **függő kódtárak** területén kattintson a **Hozzáadás** megnyitásához a **függő könyvtár hozzáadása** párbeszédpanel. 
    
    4. A **könyvtár a** kiválasztása **munkaterület**.
    
    5. Kattintson a **felhasználók**, majd a felhasználónevét, majd kattintson a `azure-eventhubs-spark_2.11:2.3.5`. 
    
    6. Kattintson az **OK** gombra.
    
    7. Ismételje meg az 1 – 6. lépéseket `spark-cassandra-connector_2.11:2.3.1` és `gt-shapefile:19.2`.

9. Mellett **fürt:**, kattintson a **szerkesztése**. Ekkor megnyílik a **fürt konfigurálása** párbeszédpanel. Az a **fürttípus** legördülő menüben válassza **meglévő fürt**. Az a **válassza ki a fürt** legördülő menüben válassza ki a létrehozott fürtöt a **hozzon létre egy Databricks-fürt** szakaszban. Kattintson a **megerősítése**.

10. Kattintson a **Futtatás most**.

### <a name="run-the-data-generator"></a>Futtassa az adatgenerátor

1. Lépjen abba a könyvtárba `data/streaming_azuredatabricks/onprem` a GitHub-adattárában.

2. Frissítse az értékeket a fájlban **main.env** módon:

    ```
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    A kapcsolati karakterlánc a taxi-indításáról eseményközpont a **taxi-indításáról-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban. A kapcsolati karakterláncot a taxi diszkont eseményközpont a **taxi-diszkont-eh** értéket a **eventHubs** kimeneti szakasz 4. lépésében a *üzembe helyezése az Azure-erőforrások* szakaszban.

3. Futtassa a következő parancsot a Docker-rendszerkép létrehozásához.

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. Lépjen vissza a szülőkönyvtárban `data/stream_azuredatabricks`.

    ```bash
    cd ..
    ```

5. Futtassa a következő parancsot beírva futtassa a Docker-rendszerképet.

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

A kimenet a következőhöz hasonlóan kell kinéznie:

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

Ellenőrizze, hogy a Databricks-feladat megfelelően fut-e, nyissa meg az Azure Portalt, és keresse meg a Cosmos DB-adatbázist. Nyissa meg a **adatkezelő** panelen, és vizsgálja meg az adatok a **rekordok taxiköltség** tábla. 

[1] <span id="note1">Donovan, Brian; Munka, Dan (2016): New York City Taxi Útadatok (2010, 2013). Egyetemi Illinois, Urbana-Champaignben. https://doi.org/10.13012/J8PN93H8
