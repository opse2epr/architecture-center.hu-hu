# <a name="event-processing-using-azure-functions"></a>Események feldolgozása az Azure Functions használatával

Ez a referenciaarchitektúra bemutatja, egy kiszolgáló nélküli, eseményvezérelt architektúra, amely egy adatfolyam betölti, feldolgozza az adatokat és az eredményeket egy háttér-adatbázisba írja. Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![](./_images/serverless-event-processing.png)

## <a name="architecture"></a>Architektúra

**Az Event Hubs** betölti az adatfolyamot. [Az Event Hubs] [ eh] nagy átviteli sebességű adatstreamelési forgatókönyvekhez készült.

> [!NOTE]
> IoT-forgatókönyvek esetén javasoljuk, hogy az IoT Hub. Az IoT Hub rendelkezik egy beépített végpont, amely kompatibilis az Azure Event Hubs API, így mindkét szolgáltatás ebben az architektúrában a háttérbeli feldolgozás jelentős módosítása nélkül is használhatja. További információkért lásd: [IoT-eszközök csatlakoztatása az Azure-bA: az IoT Hub és az Event Hubs][iot].

**Függvényalkalmazásnak**. [Az Azure Functions] [ functions] kiszolgáló nélküli számítási megoldás. Az eseményvezérelt modellt használ, ahol a kódrészleteket (a "function") egy eseményindító hív. Ebben az architektúrában esemény érkezik, az Event Hubs, amikor azok aktiválása egy függvényt, amely dolgozza fel a, és írja az eredmények tárolása.

Az Event Hubsból az egyes rekordok feldolgozása Függvényalkalmazások alkalmasak. Összetettebb adatfolyam-feldolgozási forgatókönyveihez fontolja meg az Apache Spark az Azure Databricks, vagy az Azure Stream Analytics használatával.

**A cosmos DB**. [A cosmos DB] [ cosmosdb] többmodelles adatbázis-szolgáltatás. Ebben a forgatókönyvben a eseményfeldolgozó függvény JSON-rekordokat, a Cosmos DB használatával tárolja [SQL API][cosmosdb-sql].

**Queue storage**. [Queue storage] [ queue] feldolgozatlan üzenetek szolgál. Egy esemény feldolgozása során hiba történik, ha a függvény az eseményadatokat egy feldolgozatlan üzenetek sorhoz későbbi feldolgozáshoz tárolja. További információkért lásd: [rugalmassággal kapcsolatos szempontok](#resiliency-considerations).

**Az Azure Monitor**. [A figyelő] [ monitor] teljesítmény-mérőszámokat a megoldásban üzembe helyezett Azure-szolgáltatásokkal gyűjti. Ezek az irányítópult vizualizációjával képet is kaphat a megoldás állapotát.

**Az Azure folyamatok**. [A folyamatok] [ pipelines] van egy folyamatos integrációs (CI) és a folyamatos továbbítás (CD) a buildek esetében teszteket, szolgáltatást, és központilag telepíti az alkalmazást.

## <a name="scalability-considerations"></a>Méretezési szempontok

### <a name="event-hubs"></a>Event Hubs

Az Event Hubs átviteli kapacitásának mérése [átviteli egységek][eh-throughput]. Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről][eh-autoscale], amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.

A [Eseményközpont-eseményindító] [ eh-trigger] a függvényben található alkalmazás méretezi a partíciók száma szerint az eseményközpont. Mindegyik partíció hozzá van rendelve egy függvény példány egyszerre. Átviteli sebesség maximalizálása kapja a kötegben, egyenként helyett.

### <a name="cosmos-db"></a>Cosmos DB

A Cosmos DB átviteli kapacitás mérése [kérelemegység] [ ru] (RU). Annak érdekében, hogy az elmúlt 10 000-et egy Cosmos DB-tárolók skálázása RU, meg kell adnia egy [partíciókulcs] [ partition-key] amikor létrehozza a tárolót, és a partíciókulcs tartalmazza minden egyes dokumentum létrehozott.

Íme néhány remek partíciókulcs jellemzői:

- A kulcs értéke lemezterület mérete nagy. 
- Az egyenletes eloszlás a per kulcs értékét, elkerülve a billentyűparancsok olvasási/írási művelet lesz.
- Nem minden egyetlen kulcs értékét a tárolt adatok maximális túl fogja lépni a fizikai partíciók maximális mérete (10 GB). 
- A partíciókulcs a dokumentum nem fognak változni. A meglévő dokumentumot a partíciós kulcs nem frissíthető. 

Ez a referenciaarchitektúra esetben a függvény, amely adatokat küld eszközönként pontosan egy dokumentum tárolja. A függvény folyamatosan frissíti a dokumentumok legújabb Eszközállapot upsert művelet használatával. Eszközazonosító azért ebben a forgatókönyvben egy remek partíciókulcs lesz írási műveletek egyenletesen vannak elosztva a kulcsokat, és az egyes partíciók méretét szigorúan csak korlátozott lesz, mert nincs külön dokumentumot minden egyes kulcs értékét. Partíciókulcsok kapcsolatos további információkért lásd: [particionálási és horizontális az Azure Cosmos DB][cosmosdb-scale].

## <a name="resiliency-considerations"></a>Rugalmassággal kapcsolatos szempontok

Az Event Hubs-eseményindító a Functions használatakor a feldolgozási ciklus belül kivételeket. Ha egy nem kezelt kivétel lép fel, a Functions futtatókörnyezete nem próbálkozik újra az üzeneteket. Nem lehet feldolgozni egy üzenetet, ha az üzenet helyezzen egy feldolgozatlan üzenetek sorhoz. Vizsgálja meg az üzeneteket, és meghatározhatja a javítási művelet sávon kívüli folyamat alkalmazásával. 

A következő kód bemutatja, hogyan az adatfeldolgozási függvény kivételek elkapja, és a kézbesítetlen levelek várólistája alakzatot állapotba állítja a feldolgozatlan üzenetek.

```csharp
[FunctionName("RawTelemetryFunction")]
[StorageAccount("DeadLetterStorage")]
public static async Task RunAsync(
    [EventHubTrigger("%EventHubName%", Connection = "EventHubConnection", ConsumerGroup ="%EventHubConsumerGroup%")]EventData[] messages,
    [Queue("deadletterqueue")] IAsyncCollector<DeadLetterMessage> deadLetterMessages,
    ILogger logger)
{
    foreach (var message in messages)
    {
        DeviceState deviceState = null;

        try
        {
            deviceState = telemetryProcessor.Deserialize(message.Body.Array, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error deserializing message", message.SystemProperties.PartitionKey, message.SystemProperties.SequenceNumber);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message });
        }

        try
        {
            await stateChangeProcessor.UpdateState(deviceState, logger);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error updating status document", deviceState);
            await deadLetterMessages.AddAsync(new DeadLetterMessage { Issue = ex.Message, EventData = message, DeviceState = deviceState });
        }
    }
}
```

Figyelje meg, hogy a függvény használja a [a Queue storage kimeneti kötésének] [ queue-binding] elemek elhelyezése az üzenetsorban.

A fent látható kód kivételeket is naplózza az Application Insightsba. A kulcs és a sorrend partíciószám segítségével feldolgozatlan üzenetek összekapcsolását a kivételekkel a naplókban. 

A kézbesíthetetlen levelek sorában lévő üzenetek elegendő információt kell rendelkeznie, így hiba összefüggésében értelmezni lehet. Ebben a példában a `DeadLetterMessage` osztály tartalmazza a kivétel üzenetét, az eredeti eseményadatokat és a deszerializált eseményüzenet (ha elérhető). 

```csharp
public class DeadLetterMessage
{
    public string Issue { get; set; }
    public EventData EventData { get; set; }
    public DeviceState DeviceState { get; set; }
}
```

Használat [Azure Monitor] [ monitor] az event hubs figyeléséhez. Ha nincs a bemeneti, de nincs kimenet látható, az azt jelenti, hogy üzeneteket nem feldolgozott. Ebben az esetben lépnek [Log Analytics] [ log-analytics] és egyéb hibák és kivételek.

## <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok

Az üzembe helyezés itt látható egy adott Azure-régióban található. A vész-helyreállítási rugalmasabb módszer, a földrajzi elosztás funkcióinak kihasználása a különböző szolgáltatások:

- **Az Event Hubs**. Hozzon létre két Event Hubs-névterek, az elsődleges (aktív) névtér és a egy másodlagos (passzív) névtér. Üzenetek a rendszer automatikusan irányítja az aktív névteret, kivéve, ha átadja a feladatokat a másodlagos névtér. További információkért lásd: [Azure Event Hubs Geo-disaster recovery][eh-dr].

- **Függvényalkalmazásnak**. Olvasni a másodlagos Event Hubs-névtér vár második függvényalkalmazás üzembe helyezése. Ez a függvény ír egy másodlagos tárfiók kézbesítetlen levelek várólistájára vonatkozik.

- **A cosmos DB**. A cosmos DB támogatja a [több fő régióban][cosmosdb-geo], amely lehetővé teszi, hogy minden olyan régióban, adja hozzá a Cosmos DB-fiókot az írási műveletek. Ha nem engedélyezi több főkiszolgálós, továbbra is adhatók át az elsődleges írási régió. A Cosmos DB ügyféloldali SDK-kkal és az Azure-függvény kötések automatikusan kezeli a feladatátvételt, így nem kell minden olyan alkalmazás konfigurációs beállításainak frissítése.

- **Azure Storage**. Használat [RA-GRS] [ ra-grs] tárhelyet a kézbesítetlen levelek várólistájára vonatkozik. Ez létrehoz egy csak olvasható replikát egy másik régióban. Ha az elsődleges régió elérhetetlenné válik, a jelenleg a várólistában lévő elemek érheti el. Emellett üzembe helyezhető egy másik tárfiókot, amely a függvény írhat a feladatátvétel után a másodlagos régióban.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra üzembe helyezéséhez keresse meg a [GitHub információs][readme]. 

<!-- links -->

[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[cosmosdb-sql]: /azure/cosmos-db/sql-api-introduction
[eh]: /azure/event-hubs/
[eh-autoscale]: /azure/event-hubs/event-hubs-auto-inflate
[eh-dr]: /azure/event-hubs/event-hubs-geo-dr
[eh-throughput]: /azure/event-hubs/event-hubs-features#throughput-units
[eh-trigger]: /azure/azure-functions/functions-bindings-event-hubs
[functions]: /azure/azure-functions/functions-overview
[iot]: /azure/iot-hub/iot-hub-compare-event-hubs
[log-analytics]: /azure/log-analytics/log-analytics-queries
[monitor]: /azure/azure-monitor/overview
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[queue]: /azure/storage/queues/storage-queues-introduction
[queue-binding]: /azure/azure-functions/functions-bindings-storage-queue#output
[ra-grs]: /azure/storage/common/storage-redundancy-grs
[ru]: /azure/cosmos-db/request-units

[github]: https://github.com/mspnp/serverless-reference-implementation
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md