# <a name="event-processing-using-azure-functions"></a><span data-ttu-id="c7a46-101">Események feldolgozása az Azure Functions használatával</span><span class="sxs-lookup"><span data-stu-id="c7a46-101">Event processing using Azure Functions</span></span>

<span data-ttu-id="c7a46-102">Ez a referenciaarchitektúra bemutatja, egy kiszolgáló nélküli, eseményvezérelt architektúra, amely egy adatfolyam betölti, feldolgozza az adatokat és az eredményeket egy háttér-adatbázisba írja.</span><span class="sxs-lookup"><span data-stu-id="c7a46-102">This reference architecture shows a serverless, event-driven architecture that ingests a stream of data, processes the data, and writes the results to a back-end database.</span></span> <span data-ttu-id="c7a46-103">Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="c7a46-103">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![](./_images/serverless-event-processing.png)

## <a name="architecture"></a><span data-ttu-id="c7a46-104">Architektúra</span><span class="sxs-lookup"><span data-stu-id="c7a46-104">Architecture</span></span>

<span data-ttu-id="c7a46-105">**Az Event Hubs** betölti az adatfolyamot.</span><span class="sxs-lookup"><span data-stu-id="c7a46-105">**Event Hubs** ingests the data stream.</span></span> <span data-ttu-id="c7a46-106">[Az Event Hubs] [ eh] nagy átviteli sebességű adatstreamelési forgatókönyvekhez készült.</span><span class="sxs-lookup"><span data-stu-id="c7a46-106">[Event Hubs][eh] is designed for high-throughput data streaming scenarios.</span></span>

> [!NOTE]
> <span data-ttu-id="c7a46-107">IoT-forgatókönyvek esetén javasoljuk, hogy az IoT Hub.</span><span class="sxs-lookup"><span data-stu-id="c7a46-107">For IoT scenarios, we recommend IoT Hub.</span></span> <span data-ttu-id="c7a46-108">Az IoT Hub rendelkezik egy beépített végpont, amely kompatibilis az Azure Event Hubs API, így mindkét szolgáltatás ebben az architektúrában a háttérbeli feldolgozás jelentős módosítása nélkül is használhatja.</span><span class="sxs-lookup"><span data-stu-id="c7a46-108">IoT Hub has a built-in endpoint that’s compatible with the Azure Event Hubs API, so you can use either service in this architecture with no major changes in the backend processing.</span></span> <span data-ttu-id="c7a46-109">További információkért lásd: [IoT-eszközök csatlakoztatása az Azure-bA: az IoT Hub és az Event Hubs][iot].</span><span class="sxs-lookup"><span data-stu-id="c7a46-109">For more information, see [Connecting IoT Devices to Azure: IoT Hub and Event Hubs][iot].</span></span>

<span data-ttu-id="c7a46-110">**Függvényalkalmazásnak**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-110">**Function App**.</span></span> <span data-ttu-id="c7a46-111">[Az Azure Functions] [ functions] kiszolgáló nélküli számítási megoldás.</span><span class="sxs-lookup"><span data-stu-id="c7a46-111">[Azure Functions][functions] is a serverless compute option.</span></span> <span data-ttu-id="c7a46-112">Az eseményvezérelt modellt használ, ahol a kódrészleteket (a "function") egy eseményindító hív.</span><span class="sxs-lookup"><span data-stu-id="c7a46-112">It uses an event-driven model, where a piece of code (a “function”) is invoked by a trigger.</span></span> <span data-ttu-id="c7a46-113">Ebben az architektúrában esemény érkezik, az Event Hubs, amikor azok aktiválása egy függvényt, amely dolgozza fel a, és írja az eredmények tárolása.</span><span class="sxs-lookup"><span data-stu-id="c7a46-113">In this architecture, when events arrive at Event Hubs, they trigger a function that processes the events and writes the results to storage.</span></span>

<span data-ttu-id="c7a46-114">Az Event Hubsból az egyes rekordok feldolgozása Függvényalkalmazások alkalmasak.</span><span class="sxs-lookup"><span data-stu-id="c7a46-114">Function Apps are suitable for processing individual records from Event Hubs.</span></span> <span data-ttu-id="c7a46-115">Összetettebb adatfolyam-feldolgozási forgatókönyveihez fontolja meg az Apache Spark az Azure Databricks, vagy az Azure Stream Analytics használatával.</span><span class="sxs-lookup"><span data-stu-id="c7a46-115">For more complex stream processing scenarios, consider Apache Spark using Azure Databricks, or Azure Stream Analytics.</span></span>

<span data-ttu-id="c7a46-116">**A cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-116">**Cosmos DB**.</span></span> <span data-ttu-id="c7a46-117">[A cosmos DB] [ cosmosdb] többmodelles adatbázis-szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="c7a46-117">[Cosmos DB][cosmosdb] is a multi-model database service.</span></span> <span data-ttu-id="c7a46-118">Ebben a forgatókönyvben a eseményfeldolgozó függvény JSON-rekordokat, a Cosmos DB használatával tárolja [SQL API][cosmosdb-sql].</span><span class="sxs-lookup"><span data-stu-id="c7a46-118">For this scenario, the event-processing function stores JSON records, using the Cosmos DB [SQL API][cosmosdb-sql].</span></span>

<span data-ttu-id="c7a46-119">**Queue storage**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-119">**Queue storage**.</span></span> <span data-ttu-id="c7a46-120">[Queue storage] [ queue] feldolgozatlan üzenetek szolgál.</span><span class="sxs-lookup"><span data-stu-id="c7a46-120">[Queue storage][queue] is used for dead letter messages.</span></span> <span data-ttu-id="c7a46-121">Egy esemény feldolgozása során hiba történik, ha a függvény az eseményadatokat egy feldolgozatlan üzenetek sorhoz későbbi feldolgozáshoz tárolja.</span><span class="sxs-lookup"><span data-stu-id="c7a46-121">If an error occurs while processing an event, the function stores the event data in a dead letter queue for later processing.</span></span> <span data-ttu-id="c7a46-122">További információkért lásd: [rugalmassággal kapcsolatos szempontok](#resiliency-considerations).</span><span class="sxs-lookup"><span data-stu-id="c7a46-122">For more information, see [Resiliency Considerations](#resiliency-considerations).</span></span>

<span data-ttu-id="c7a46-123">**Az Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-123">**Azure Monitor**.</span></span> <span data-ttu-id="c7a46-124">[A figyelő] [ monitor] teljesítmény-mérőszámokat a megoldásban üzembe helyezett Azure-szolgáltatásokkal gyűjti.</span><span class="sxs-lookup"><span data-stu-id="c7a46-124">[Monitor][monitor] collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="c7a46-125">Ezek az irányítópult vizualizációjával képet is kaphat a megoldás állapotát.</span><span class="sxs-lookup"><span data-stu-id="c7a46-125">By visualizing these in a dashboard, you can get visibility  into the health of the solution.</span></span>

<span data-ttu-id="c7a46-126">**Az Azure folyamatok**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-126">**Azure Pipelines**.</span></span> <span data-ttu-id="c7a46-127">[A folyamatok] [ pipelines] van egy folyamatos integrációs (CI) és a folyamatos továbbítás (CD) a buildek esetében teszteket, szolgáltatást, és központilag telepíti az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="c7a46-127">[Pipelines][pipelines] is a continuous integration (CI) and continuous delivery (CD) service that builds, tests, and deploys the application.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="c7a46-128">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="c7a46-128">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="c7a46-129">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="c7a46-129">Event Hubs</span></span>

<span data-ttu-id="c7a46-130">Az Event Hubs átviteli kapacitásának mérése [átviteli egységek][eh-throughput].</span><span class="sxs-lookup"><span data-stu-id="c7a46-130">The throughput capacity of Event Hubs is measured in [throughput units][eh-throughput].</span></span> <span data-ttu-id="c7a46-131">Automatikus skálázási egy eseményközpontba engedélyezésével is [automatikus feltöltésről][eh-autoscale], amely automatikusan méretezi az átviteli egységek alapján a forgalom, akár a beállított maximális értéket.</span><span class="sxs-lookup"><span data-stu-id="c7a46-131">You can autoscale an event hub by enabling [auto-inflate][eh-autoscale], which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

<span data-ttu-id="c7a46-132">A [Eseményközpont-eseményindító] [ eh-trigger] a függvényben található alkalmazás méretezi a partíciók száma szerint az eseményközpont.</span><span class="sxs-lookup"><span data-stu-id="c7a46-132">The [Event Hub trigger][eh-trigger] in the function app scales according to the number of partitions in the event hub.</span></span> <span data-ttu-id="c7a46-133">Mindegyik partíció hozzá van rendelve egy függvény példány egyszerre.</span><span class="sxs-lookup"><span data-stu-id="c7a46-133">Each partition is assigned one function instance at a time.</span></span> <span data-ttu-id="c7a46-134">Átviteli sebesség maximalizálása kapja a kötegben, egyenként helyett.</span><span class="sxs-lookup"><span data-stu-id="c7a46-134">To maximize throughput, receive the events in a batch, instead of one at a time.</span></span>

### <a name="cosmos-db"></a><span data-ttu-id="c7a46-135">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="c7a46-135">Cosmos DB</span></span>

<span data-ttu-id="c7a46-136">A Cosmos DB átviteli kapacitás mérése [kérelemegység] [ ru] (RU).</span><span class="sxs-lookup"><span data-stu-id="c7a46-136">Throughput capacity for Cosmos DB is measured in [Request Units][ru] (RU).</span></span> <span data-ttu-id="c7a46-137">Annak érdekében, hogy az elmúlt 10 000-et egy Cosmos DB-tárolók skálázása RU, meg kell adnia egy [partíciókulcs] [ partition-key] amikor létrehozza a tárolót, és a partíciókulcs tartalmazza minden egyes dokumentum létrehozott.</span><span class="sxs-lookup"><span data-stu-id="c7a46-137">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key][partition-key] when you create the container, and include the partition key in every document that you create.</span></span>

<span data-ttu-id="c7a46-138">Íme néhány remek partíciókulcs jellemzői:</span><span class="sxs-lookup"><span data-stu-id="c7a46-138">Here are some characteristics of a good partition key:</span></span>

- <span data-ttu-id="c7a46-139">A kulcs értéke lemezterület mérete nagy.</span><span class="sxs-lookup"><span data-stu-id="c7a46-139">The key value space is large.</span></span> 
- <span data-ttu-id="c7a46-140">Az egyenletes eloszlás a per kulcs értékét, elkerülve a billentyűparancsok olvasási/írási művelet lesz.</span><span class="sxs-lookup"><span data-stu-id="c7a46-140">There will be an even distribution of reads/writes per key value, avoiding hot keys.</span></span>
- <span data-ttu-id="c7a46-141">Nem minden egyetlen kulcs értékét a tárolt adatok maximális túl fogja lépni a fizikai partíciók maximális mérete (10 GB).</span><span class="sxs-lookup"><span data-stu-id="c7a46-141">The maximum data stored for any single key value will not exceed the maximum physical partition size (10 GB).</span></span> 
- <span data-ttu-id="c7a46-142">A partíciókulcs a dokumentum nem fognak változni.</span><span class="sxs-lookup"><span data-stu-id="c7a46-142">The partition key for a document won't change.</span></span> <span data-ttu-id="c7a46-143">A meglévő dokumentumot a partíciós kulcs nem frissíthető.</span><span class="sxs-lookup"><span data-stu-id="c7a46-143">You can't update the partition key on an existing document.</span></span> 

<span data-ttu-id="c7a46-144">Ez a referenciaarchitektúra esetben a függvény, amely adatokat küld eszközönként pontosan egy dokumentum tárolja.</span><span class="sxs-lookup"><span data-stu-id="c7a46-144">In the scenario for this reference architecture, the function stores exactly one document per device that is sending data.</span></span> <span data-ttu-id="c7a46-145">A függvény folyamatosan frissíti a dokumentumok legújabb Eszközállapot upsert művelet használatával.</span><span class="sxs-lookup"><span data-stu-id="c7a46-145">The function continually updates the documents with latest device status, using an upsert operation.</span></span> <span data-ttu-id="c7a46-146">Eszközazonosító azért ebben a forgatókönyvben egy remek partíciókulcs lesz írási műveletek egyenletesen vannak elosztva a kulcsokat, és az egyes partíciók méretét szigorúan csak korlátozott lesz, mert nincs külön dokumentumot minden egyes kulcs értékét.</span><span class="sxs-lookup"><span data-stu-id="c7a46-146">Device ID is a good partition key for this scenario, because writes will be evenly distributed across the keys, and the size of each partition will be strictly bounded, because there is a single document for each key value.</span></span> <span data-ttu-id="c7a46-147">Partíciókulcsok kapcsolatos további információkért lásd: [particionálási és horizontális az Azure Cosmos DB][cosmosdb-scale].</span><span class="sxs-lookup"><span data-stu-id="c7a46-147">For more information about partition keys, see [Partition and scale in Azure Cosmos DB][cosmosdb-scale].</span></span>

## <a name="resiliency-considerations"></a><span data-ttu-id="c7a46-148">Rugalmassággal kapcsolatos szempontok</span><span class="sxs-lookup"><span data-stu-id="c7a46-148">Resiliency considerations</span></span>

<span data-ttu-id="c7a46-149">Az Event Hubs-eseményindító a Functions használatakor a feldolgozási ciklus belül kivételeket.</span><span class="sxs-lookup"><span data-stu-id="c7a46-149">When using the Event Hubs trigger with Functions, catch exceptions within your processing loop.</span></span> <span data-ttu-id="c7a46-150">Ha egy nem kezelt kivétel lép fel, a Functions futtatókörnyezete nem próbálkozik újra az üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="c7a46-150">If an unhandled exception occurs, the Functions runtime does not retry the messages.</span></span> <span data-ttu-id="c7a46-151">Nem lehet feldolgozni egy üzenetet, ha az üzenet helyezzen egy feldolgozatlan üzenetek sorhoz.</span><span class="sxs-lookup"><span data-stu-id="c7a46-151">If a message cannot be processed, put the message into a dead letter queue.</span></span> <span data-ttu-id="c7a46-152">Vizsgálja meg az üzeneteket, és meghatározhatja a javítási művelet sávon kívüli folyamat alkalmazásával.</span><span class="sxs-lookup"><span data-stu-id="c7a46-152">Use an out-of-band process to examine the messages and determine corrective action.</span></span> 

<span data-ttu-id="c7a46-153">A következő kód bemutatja, hogyan az adatfeldolgozási függvény kivételek elkapja, és a kézbesítetlen levelek várólistája alakzatot állapotba állítja a feldolgozatlan üzenetek.</span><span class="sxs-lookup"><span data-stu-id="c7a46-153">The following code shows how the ingestion function catches exceptions and puts unprocessed messages onto a dead letter queue.</span></span>

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

<span data-ttu-id="c7a46-154">Figyelje meg, hogy a függvény használja a [a Queue storage kimeneti kötésének] [ queue-binding] elemek elhelyezése az üzenetsorban.</span><span class="sxs-lookup"><span data-stu-id="c7a46-154">Notice that the function uses the [Queue storage output binding][queue-binding] to put items in the queue.</span></span>

<span data-ttu-id="c7a46-155">A fent látható kód kivételeket is naplózza az Application Insightsba.</span><span class="sxs-lookup"><span data-stu-id="c7a46-155">The code shown above also logs exceptions to Application Insights.</span></span> <span data-ttu-id="c7a46-156">A kulcs és a sorrend partíciószám segítségével feldolgozatlan üzenetek összekapcsolását a kivételekkel a naplókban.</span><span class="sxs-lookup"><span data-stu-id="c7a46-156">You can use the partition key and sequence number to correlate dead letter messages with the exceptions in the logs.</span></span> 

<span data-ttu-id="c7a46-157">A kézbesíthetetlen levelek sorában lévő üzenetek elegendő információt kell rendelkeznie, így hiba összefüggésében értelmezni lehet.</span><span class="sxs-lookup"><span data-stu-id="c7a46-157">Messages in the dead letter queue should have enough information so that you can understand the context of error.</span></span> <span data-ttu-id="c7a46-158">Ebben a példában a `DeadLetterMessage` osztály tartalmazza a kivétel üzenetét, az eredeti eseményadatokat és a deszerializált eseményüzenet (ha elérhető).</span><span class="sxs-lookup"><span data-stu-id="c7a46-158">In this example, the `DeadLetterMessage` class contains the exception message, the original event data, and the deserialized event message (if available).</span></span> 

```csharp
public class DeadLetterMessage
{
    public string Issue { get; set; }
    public EventData EventData { get; set; }
    public DeviceState DeviceState { get; set; }
}
```

<span data-ttu-id="c7a46-159">Használat [Azure Monitor] [ monitor] az event hubs figyeléséhez.</span><span class="sxs-lookup"><span data-stu-id="c7a46-159">Use [Azure Monitor][monitor] to monitor the event hub.</span></span> <span data-ttu-id="c7a46-160">Ha nincs a bemeneti, de nincs kimenet látható, az azt jelenti, hogy üzeneteket nem feldolgozott.</span><span class="sxs-lookup"><span data-stu-id="c7a46-160">If you see there is input but no output, it means that messages are not being processed.</span></span> <span data-ttu-id="c7a46-161">Ebben az esetben lépnek [Log Analytics] [ log-analytics] és egyéb hibák és kivételek.</span><span class="sxs-lookup"><span data-stu-id="c7a46-161">In that case, go into [Log Analytics][log-analytics] and look for exceptions or other errors.</span></span>

## <a name="disaster-recovery-considerations"></a><span data-ttu-id="c7a46-162">Vészhelyreállítási szempontok</span><span class="sxs-lookup"><span data-stu-id="c7a46-162">Disaster recovery considerations</span></span>

<span data-ttu-id="c7a46-163">Az üzembe helyezés itt látható egy adott Azure-régióban található.</span><span class="sxs-lookup"><span data-stu-id="c7a46-163">The deployment shown here resides in a single Azure region.</span></span> <span data-ttu-id="c7a46-164">A vész-helyreállítási rugalmasabb módszer, a földrajzi elosztás funkcióinak kihasználása a különböző szolgáltatások:</span><span class="sxs-lookup"><span data-stu-id="c7a46-164">For a more resilient approach to disaster-recovery, take advantage of geo-distribution features in the various services:</span></span>

- <span data-ttu-id="c7a46-165">**Az Event Hubs**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-165">**Event Hubs**.</span></span> <span data-ttu-id="c7a46-166">Hozzon létre két Event Hubs-névterek, az elsődleges (aktív) névtér és a egy másodlagos (passzív) névtér.</span><span class="sxs-lookup"><span data-stu-id="c7a46-166">Create two Event Hubs namespaces, a primary (active) namespace and a secondary (passive) namespace.</span></span> <span data-ttu-id="c7a46-167">Üzenetek a rendszer automatikusan irányítja az aktív névteret, kivéve, ha átadja a feladatokat a másodlagos névtér.</span><span class="sxs-lookup"><span data-stu-id="c7a46-167">Messages are automatically routed to the active namespace unless you fail over to the secondary namespace.</span></span> <span data-ttu-id="c7a46-168">További információkért lásd: [Azure Event Hubs Geo-disaster recovery][eh-dr].</span><span class="sxs-lookup"><span data-stu-id="c7a46-168">For more information, see [Azure Event Hubs Geo-disaster recovery][eh-dr].</span></span>

- <span data-ttu-id="c7a46-169">**Függvényalkalmazásnak**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-169">**Function App**.</span></span> <span data-ttu-id="c7a46-170">Olvasni a másodlagos Event Hubs-névtér vár második függvényalkalmazás üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="c7a46-170">Deploy a second function app that is waiting to read from the secondary Event Hubs namespace.</span></span> <span data-ttu-id="c7a46-171">Ez a függvény ír egy másodlagos tárfiók kézbesítetlen levelek várólistájára vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="c7a46-171">This function writes to a secondary storage account for dead letter queue.</span></span>

- <span data-ttu-id="c7a46-172">**A cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-172">**Cosmos DB**.</span></span> <span data-ttu-id="c7a46-173">A cosmos DB támogatja a [több fő régióban][cosmosdb-geo], amely lehetővé teszi, hogy minden olyan régióban, adja hozzá a Cosmos DB-fiókot az írási műveletek.</span><span class="sxs-lookup"><span data-stu-id="c7a46-173">Cosmos DB supports [multiple master regions][cosmosdb-geo], which enables writes to any region that you add to your Cosmos DB account.</span></span> <span data-ttu-id="c7a46-174">Ha nem engedélyezi több főkiszolgálós, továbbra is adhatók át az elsődleges írási régió.</span><span class="sxs-lookup"><span data-stu-id="c7a46-174">If you don’t enable multi-master, you can still fail over the primary write region.</span></span> <span data-ttu-id="c7a46-175">A Cosmos DB ügyféloldali SDK-kkal és az Azure-függvény kötések automatikusan kezeli a feladatátvételt, így nem kell minden olyan alkalmazás konfigurációs beállításainak frissítése.</span><span class="sxs-lookup"><span data-stu-id="c7a46-175">The Cosmos DB client SDKs and the Azure Function bindings automatically handle the failover, so you don’t need to update any application configuration settings.</span></span>

- <span data-ttu-id="c7a46-176">**Azure Storage**.</span><span class="sxs-lookup"><span data-stu-id="c7a46-176">**Azure Storage**.</span></span> <span data-ttu-id="c7a46-177">Használat [RA-GRS] [ ra-grs] tárhelyet a kézbesítetlen levelek várólistájára vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="c7a46-177">Use [RA-GRS][ra-grs] storage for the dead letter queue.</span></span> <span data-ttu-id="c7a46-178">Ez létrehoz egy csak olvasható replikát egy másik régióban.</span><span class="sxs-lookup"><span data-stu-id="c7a46-178">This creates a read-only replica in another region.</span></span> <span data-ttu-id="c7a46-179">Ha az elsődleges régió elérhetetlenné válik, a jelenleg a várólistában lévő elemek érheti el.</span><span class="sxs-lookup"><span data-stu-id="c7a46-179">If the primary region becomes unavailable, you can read the items currently in the queue.</span></span> <span data-ttu-id="c7a46-180">Emellett üzembe helyezhető egy másik tárfiókot, amely a függvény írhat a feladatátvétel után a másodlagos régióban.</span><span class="sxs-lookup"><span data-stu-id="c7a46-180">In addition, provision another storage account in the secondary region that the function can write to after a fail-over.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="c7a46-181">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="c7a46-181">Deploy the solution</span></span>

<span data-ttu-id="c7a46-182">Ez a referenciaarchitektúra üzembe helyezéséhez keresse meg a [GitHub információs][readme].</span><span class="sxs-lookup"><span data-stu-id="c7a46-182">To deploy this reference architecture, view the [GitHub readme][readme].</span></span> 

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