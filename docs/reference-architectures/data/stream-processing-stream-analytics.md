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
# <a name="stream-processing-with-azure-stream-analytics"></a>Az Azure Stream Analytics feldolgozási Stream

Ez a referenciaarchitektúra bemutatja egy teljes körű stream-feldolgozási folyamat. A folyamat adatokat két forrásból fogadnak, utal. a két adatfolyam rekordokat, és kiszámítja a gördülő átlagát között egy olyan időkeretet. Az eredmények tárolása további elemzés céljából. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![](./images/stream-processing-asa/stream-processing-asa.png)

**A forgatókönyv**:-i taxik vállalati minden taxi út adatokat gyűjt. Ebben a forgatókönyvben feltételezzük adatküldés két külön eszközökre. A taxi rendelkezik, amely minden egyes indításáról információt küld a mérő &mdash; begyűjtést és dropoff helyeket, időtartama és távolság. Egy különálló eszköz fogad az ügyfelektől származó kifizetések és vitel kapcsolatos adatokat küldi. A taxi vállalat úgy kívánja kiszámítja az átlagos tipp kiszolgálónként mérföld alapuló, valós idejű, sorrendben felhasználva azonosíthatja a trendeket.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

**Adatforrások**. Ebben az architektúrában nincsenek valós idejű adatstreamek generáló két adatforrást. Az első stream indításáról információkat tartalmaz, és a második diszkont információkat tartalmaz. A referenciaarchitektúra egy szimulált adatok generátort, amely a statikus fájlokat olvas, és leküldi az adatokat az Event Hubs tartalmazza. Egy valós alkalmazás esetében az adatforrások lenne telepítve a taxi kabinetfájlok eszközök.

**Azure Event Hubs**. [Az Event Hubs](/azure/event-hubs/) egy esemény-feldolgozó szolgáltatás. Ez az architektúra két event hub-példányok, egyet az egyes adatforrások használ. Minden adatforráshoz társított eseményközpont adatfolyamot küld.

**Azure Stream Analytics**. [Stream Analytics](/azure/stream-analytics/) egy eseményfeldolgozó motor. Stream Analytics-feladat beolvassa az adatokat a két event hubs szolgáltatástól érkező adatfolyamok és adatfolyam-feldolgozás hajt végre.

**A cosmos DB**. A Stream Analytics-feladat kimenete a rekordoknak, amelyek egy Cosmos DB dokumentum-adatbázis a JSON-dokumentumok formájában írt sorozata.

**A Microsoft Power BI**. A Power BI egy üzleti elemzési eszközök az üzleti elemzések készítése adatelemzéshez együttese. Ebben az architektúrában az adatokat a Cosmos DB betölti. Ez lehetővé teszi a felhasználók teljes készletének összegyűjtött előzményadatok elemzésére. Az eredményeket közvetlenül a Stream Analytics a Power BI az adatok valós idejű nézet sikerült is folyamatos átvitelére. További információkért lásd: [valós idejű streamelés a Power BI](/power-bi/service-real-time-streaming).

**Az Azure Monitor**. [Az Azure Monitor](/azure/monitoring-and-diagnostics/) teljesítmény-mérőszámokat a megoldásban üzembe helyezett Azure-szolgáltatásokkal gyűjti. Ezek az irányítópult vizualizációjával betekintést nyerhet a megoldás állapotát. 

## <a name="data-ingestion"></a>Adatfeldolgozás

Adatforrás szimulálásához, ez a referenciaarchitektúra használja a [New York City-i taxik adatait](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) adatkészlet<sup>[[1]](#note1)</sup>. Ez az adatkészlet taxi lelassítja a New York City kapcsolatos adatokat tartalmaz egy 4 éven keresztül (2010 &ndash; 2013). Kétféle típusú rekordot tartalmaz: indításáról és diszkont adatait. Indításáról adatok út időtartama, trip távolság és begyűjtés és dropoff helye tartalmaz. Diszkont szerepel diszkont, adózási és tipp összegeket. Mindkét rekord típusa közös mező például medallion száma, a feltörés licenc és a gyártó azonosítóját. Együttesen ezek három mezőt azonosítja egy taxi és a egy illesztőprogramot. Az adatok CSV formátumban tárolódik. 

Az adatgenerátor, amelyek a rekordokat olvas be, és elküldi őket az Azure Event Hubs .NET Core-alkalmazást. A generátor indításáról adatok JSON formátumban és diszkont adatok CSV formátumban küldi el. 

Az Event Hubs használ [partíciók](/azure/event-hubs/event-hubs-features#partitions) szegmentálása az adatokat. A partíciók lehetővé teszik a fogyasztó a párhuzamos mindegyik partíció beolvasása. Ha az adatok küldése az Event Hubsba, a partíciós kulcs explicit módon is megadhat. Ellenkező esetben rekordok partíciókra Ciklikus időszeleteléses módon vannak hozzárendelve. 

Ez az adott esetben ledolgozni adatokat, és diszkont adatokat kell megtörténhet a azonos Partícióazonosító egy adott taxi cab-fájl. Ez lehetővé teszi a Stream Analytics párhuzamosság foka alkalmazza, ha azt a két adatfolyam utal. Egy rekordot a partíció *n* a indításáról, az adatok egy rekordot a partíció egyezni fog *n* diszkont adatok.

![](./images/stream-processing-asa/stream-processing-eh.png)

Az adatgenerátor a common data model mindkét bejegyzéstípusokat rendelkezik egy `PartitionKey` tulajdonság, amely az összefűzés, `Medallion`, `HackLicense`, és `VendorId`.

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

## <a name="stream-processing"></a>Stream-feldolgozás

A stream-feldolgozási feladat van definiálva egy SQL-lekérdezés használata több különálló lépésre. Az első két lépés egyszerűen válasszon rögzíti a két bemeneti streamekhez.

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

A következő lépésben válassza ki a megfelelő rekordokat az egyes adatfolyamokkal két bemeneti streamekhez csatlakozik.

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

Ez a lekérdezés illeszt rekordokat konfigurálásában a megfelelő rekordokat (Medallion, HackLicense, VendorId és PickupTime) egyedi azonosítására alkalmas mezőket. A `JOIN` utasítást is tartalmaz a partícióazonosító. Ahogy említettük, ez kihasználja a tény, hogy megfelelő rekordokat mindig rendelkezzenek a partíció ugyanazzal az Azonosítóval ebben a forgatókönyvben.

A Stream Analytics, illesztések vannak *historikus*, azaz a rekordok tartományhoz csatlakoztatott idő adott időtartamon belül. Ellenkező esetben a feladat lehet, hogy várnia kell, határozatlan ideig az egyezést. A [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) függvény meghatározza, hogy milyen két egyező rekordot is választhatók el egymástól, időben egyezést. 

A feladat utolsó lépésében kiszámítja az átlagos tipp egy lépést, 5 perces ugróablak szerint csoportosítva.

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

Stream Analytics biztosít több [ablakkezelési függvényeket](/azure/stream-analytics/stream-analytics-window-functions). Egy ugróablak helyezi előre a idő által meghatározott időre ebben az esetben ugrásonkénti 1 perc. Ez a mozgó átlag kiszámításához az elmúlt 5 percben.

Az itt látható architektúrában csak a Stream Analytics-feladat eredményeinek menti, a Cosmos DB-hez. A big Data típusú adatok esetben fontolja meg is [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) menteni a nyers eseményadatok Azure Blob storage-bA. A nyers adatok megőrzése mellett lehetővé teszi, hogy a batch lekérdezéseket futtathat az előzményadatok későbbi időpontban, annak érdekében, hogy új hozzájut a adatokból.

## <a name="scalability-considerations"></a>Méretezési szempontok

### <a name="event-hubs"></a>Event Hubs

Az Event Hubs átviteli kapacitásának mérése [átviteli egységek](/azure/event-hubs/event-hubs-features#throughput-units). Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről](/azure/event-hubs/event-hubs-auto-inflate), amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket. 

### <a name="stream-analytics"></a>Stream Analytics

A Stream Analytics a számítási erőforrások hozzárendelt feladathoz a folyamatos átviteli egységek mérése történik. Stream Analytics-feladatok méretezése legjobb, ha a feladat is méretezésnek megfelelően. Ezzel a módszerrel Stream Analytics juttathatja el a feladatot több számítási csomóponton.

Az Event Hubs-bemenet, használja a `PARTITION BY` kulcsszó a Stream Analytics-feladat particionálásához. Az adatok eloszlása az Event Hubs-partíciók alapján részekre. 

Ablakkezelési függvényekről és társításokról, historikus szükséges további SU. Amikor lehetséges, használjon `PARTITION BY` úgy, hogy az egyes partíciók külön dolgozza fel. További információkért lásd: [ismertetése és módosítása a folyamatos átviteli egységek](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).

Ha nem lehetséges a teljes Stream Analytics-feladat párhuzamosíthatja, próbálja meg a feladat felosztása több lépést, egy vagy több párhuzamos lépéseket tartalmazó indítása. Ezzel a módszerrel az első lépéseket is futtatható egyszerre. Ez a referenciaarchitektúra: például a

- 1. és 2 egyszerűek `SELECT` utasításokat, amelyek ugyanazon a partíción belül időtartományban. 
- 3. lépés a particionált csatlakozzon két bemeneti streamekhez keresztül hajtja végre. Ebben a lépésben kihasználja a tény, hogy a megfelelő rekordokat megosztani ugyanazzal a partíciókulccsal, és így garantáltan minden egyes bemeneti Stream partíció ugyanazzal az Azonosítóval rendelkezik.
- 4. lépés összesítések összes partíciót. Ez a lépés nem méretezésnek megfelelően.

A Stream Analytics használata [feladatábra](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) hány partíciók rendelt minden lépése a feladat megtekintéséhez. Az alábbi ábrán ez a referenciaarchitektúra a feladat diagramja:

![](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a>Cosmos DB

A Cosmos DB átviteli kapacitás mérése [kérelemegység](/azure/cosmos-db/request-units) (RU). Annak érdekében, hogy az elmúlt 10 000-et egy Cosmos DB-tárolók skálázása RU, meg kell adnia egy [partíciókulcs](/azure/cosmos-db/partition-data) amikor létrehozza a tárolót, és a partíciókulcs tartalmazza minden egyes dokumentum. 

Ez a referenciaarchitektúra az új dokumentumok létrehozásának csak egyszer perc (a szórási ablak időköz), így az átviteli sebességet megkövetelő meglehetősen kicsi. Éppen ezért hiba esetén nem kell hozzárendelni az ebben a forgatókönyvben egy partíciókulcsot.

## <a name="monitoring-considerations"></a>Felügyeleti szempontok

Az adatfolyam-feldolgozó megoldást fontos a teljesítmény és a rendszer állapotának figyelésére. [Az Azure Monitor](/azure/monitoring-and-diagnostics/) az architektúrában használt Azure-szolgáltatások a metrikák és diagnosztikai naplókat gyűjti össze. Az Azure Monitor az Azure platform be van építve, és nincs szükség semmilyen további programkódokat kellene megtervezni az alkalmazásban.

A következő figyelmeztető jeleket bármelyikét jelzi, hogy horizontális felskálázást a megfelelő Azure-erőforrás:

- Az Event Hubs szabályozza a kéréseket, vagy a napi kvóta közelében van.
- A Stream Analytics-feladat konzisztens módon több mint 80 %-át a lefoglalt folyamatos átviteli egységek (su-t) használ.
- A cosmos DB megkezdi a kérések szabályozása.

A referenciaarchitektúra egy egyéni irányítópult, amelyen telepítve van az Azure Portalon is tartalmaz. Után az architektúra üzembe helyezéséhez is megtekintheti az irányítópulton nyissa meg a [az Azure Portal](https://portal.azure.com) és kiválasztásával `TaxiRidesDashboard` az irányítópultok listájából. Az Azure Portalon egyéni irányítópultok létrehozása és telepítése kapcsolatos további információkért lásd: [programozott módon létrehozhat irányítópultokat Azure](/azure/azure-portal/azure-portal-dashboards-create-programmatically).

Az alábbi képen az irányítópult látható, miután a Stream Analytics-feladat futásának időtartama: körülbelül egy óra alatt.

![](./images/stream-processing-asa/asa-dashboard.png)

A panel bal alsó mutatja, hogy a SU-használat a Stream Analytics-feladat az első 15 perc során megnő, és ezután itt megáll. Ez a lehetőség egy tipikus mintája a feladat eléri egy stabil állapotot. 

Figyelje meg, hogy az Event Hubs szabályozza a kéréseket, a felső jobb oldali panelen látható. Alkalmanként szabályozott kérelmet nem áll egy probléma, mivel az Event hubs szolgáltatás ügyfél-SDK automatikusan újrapróbálkozik, ha sávszélesség-szabályozási hibaüzenet kap. Azonban ha egységes szabályozási hibákat lát, az azt jelenti az event hubs átviteli egységeket kell. A következő diagram is mutatja egy tesztfuttatás, az Event Hubs használatával az automatikus feltöltési funkció, amely automatikusan méretezhető az átviteli egységek igény szerint. 

![](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

Az automatikus feltöltési engedélyezve lett 06:35 kapcsolatos megjelölni. A p, dobja el a szabályozott kérelmeinek száma, az Event Hubs automatikusan átméreteződik legfeljebb 3 átviteli egységek láthatja.

Interestingly ennek hatására oldal, ahol egyre növekszik a Stream Analytics-feladat a SU kihasználtságát. Szabályozás, az Event Hubs mesterségesen lett csökkentve a feldolgozási sebességét a Stream Analytics-feladat. Valójában szokás, hogy egy teljesítménybeli szűk keresztmetszetet feloldása tárja fel egy másik. Ebben az esetben további SU lefoglalása a Stream Analytics-feladat megoldotta a problémát.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra egy üzemelő példánya érhető el az [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data). 

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy töltse le a zip-fájlját a [referenciaarchitektúrákat](https://github.com/mspnp/reference-architectures) GitHub-adattárban.

2. Telepítés [Docker](https://www.docker.com/) futtatásához az adatgenerálást.

3. Telepítés [az Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

4. Parancsot a parancssorba bash-parancssorból vagy PowerShell-parancssorból, jelentkezzen be Azure-fiókjába a következő:

    ```
    az login
    ```

### <a name="download-the-source-data-files"></a>Az adatok forrásfájlok letöltéséhez

1. Hozzon létre egy könyvtárat nevű `DataFile` alatt a `data/streaming_asa` könyvtárat a GitHub-adattárat a.

2. Nyisson meg egy webböngészőt, és navigáljon a https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.

3. Kattintson a **letöltése** gombra ezen az oldalon a megfelelő évnek a taxi-adatok egy zip-fájl letöltéséhez.

4. Bontsa ki a zip-fájlt a `DataFile` könyvtár.

    > [!NOTE]
    > Ez a tömörített fájl más zip-fájlokat tartalmazza. A gyermek zip-fájlokat nem csomagolja ki.

A directory-struktúra az alábbiakhoz hasonlóan kell kinéznie:

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

### <a name="deploy-the-azure-resources"></a>Az Azure-erőforrások üzembe helyezése

1. A rendszerhéj vagy a Windows-parancssort futtassa a következő parancsot, és kövesse a bejelentkezési kérések jelenhetnek:

    ```bash
    az login
    ```

2. Lépjen abba a mappába `data/streaming_asa` a GitHub-adattárban

    ```bash
    cd data/streaming_asa
    ```

2. Futtassa az alábbi parancsokat az Azure-erőforrások üzembe helyezéséhez:

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

3. Az Azure Portalon keresse meg a létrehozott erőforráscsoportot.

4. A Stream Analytics-feladat panel megnyitásához.

5. Kattintson a **Start** elindítani a feladatot. Válassza ki **most** kimenetként, kezdő időpont. Várjon, amíg a feladat elindításához.

### <a name="run-the-data-generator"></a>Futtassa az adatgenerátor

1. Az Event Hub-kapcsolati karakterláncok beolvasása. Ezek az Azure Portalról, vagy a következő CLI-parancsok futtatásával kaphat:

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

2. Lépjen abba a könyvtárba `data/streaming_asa/onprem` a GitHub-adattárban

3. Frissítse az értékeket a fájlban `main.env` módon:

    ```
    RIDE_EVENT_HUB=[Connection string for taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```

4. Futtassa a következő parancsot a Docker-rendszerkép létrehozásához.

    ```bash
    docker build --no-cache -t dataloader .
    ```

5. Lépjen vissza a szülőkönyvtárban `data/stream_asa`.

    ```bash
    cd ..
    ```

6. Futtassa a következő parancsot beírva futtassa a Docker-rendszerképet.

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

Lehetővé teszik a program futtatása legalább 5 percnek, azaz a meghatározott a Stream Analytics lekérdezési ablak. A Stream Analytics-feladat megfelelően fut-e, nyissa meg az Azure Portalt, és keresse meg a Cosmos DB-adatbázist. Nyissa meg a **adatkezelő** panel és a dokumentumok megtekintése. 

[1] <span id="note1">Donovan, Brian; Munka, Dan (2016): New York City Taxi Útadatok (2010, 2013). Egyetemi Illinois, Urbana-Champaignben. https://doi.org/10.13012/J8PN93H8
