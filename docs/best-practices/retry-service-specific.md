---
title: "Szolgáltatásspecifikus útmutatás az újrapróbálkozási mechanizmushoz"
description: "Szolgáltatás beállítása az újrapróbálkozási mechanizmussal vonatkozó konkrét útmutatást."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: da1145e2f2f91befd69505ae9ef2734d6110c1d0
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/30/2018
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="af373-103">Újrapróbálkozási útmutatás adott szolgáltatásoknál</span><span class="sxs-lookup"><span data-stu-id="af373-103">Retry guidance for specific services</span></span>

<span data-ttu-id="af373-104">Az Azure-szolgáltatások és az ügyfél SDK-k tartalmaznak egy újrapróbálkozási mechanizmus.</span><span class="sxs-lookup"><span data-stu-id="af373-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="af373-105">Azonban ezek eltérnek, mert minden egyes szolgáltatás eltérő jellemzőkkel és a követelmények, és így minden újrapróbálkozási mechanizmussal van beállítva, hogy egy adott szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="af373-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="af373-106">Ez az útmutató az Azure-szolgáltatások többsége a újrapróbálkozási mechanizmus szolgáltatásokat foglalja, és segítséget nyújtanak a használata, igazítja, vagy az újrapróbálkozási mechanizmussal, hogy a szolgáltatás kiterjesztése tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="af373-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="af373-107">Átmeneti kezel, és újra próbálkozik a kapcsolatokat és a szolgáltatások és erőforrások műveleteket általános útmutatást lásd: [útmutatást újra](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="af373-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="af373-108">A következő táblázat összefoglalja az ebben az útmutatóban leírt Azure-szolgáltatásokhoz tartozó újrapróbálkozási szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="af373-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="af373-109">**Szolgáltatás**</span><span class="sxs-lookup"><span data-stu-id="af373-109">**Service**</span></span> | <span data-ttu-id="af373-110">**Ismételje meg a képességek**</span><span class="sxs-lookup"><span data-stu-id="af373-110">**Retry capabilities**</span></span> | <span data-ttu-id="af373-111">**Házirend-konfiguráció**</span><span class="sxs-lookup"><span data-stu-id="af373-111">**Policy configuration**</span></span> | <span data-ttu-id="af373-112">**Hatókör**</span><span class="sxs-lookup"><span data-stu-id="af373-112">**Scope**</span></span> | <span data-ttu-id="af373-113">**Telemetria szolgáltatások**</span><span class="sxs-lookup"><span data-stu-id="af373-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="af373-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="af373-115">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-115">Native in client</span></span> |<span data-ttu-id="af373-116">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-116">Programmatic</span></span> |<span data-ttu-id="af373-117">Ügyfél és az egyes műveletek</span><span class="sxs-lookup"><span data-stu-id="af373-117">Client and individual operations</span></span> |<span data-ttu-id="af373-118">TraceSource</span><span class="sxs-lookup"><span data-stu-id="af373-118">TraceSource</span></span> |
| <span data-ttu-id="af373-119">**[Az Entity Framework SQL-adatbázis](#sql-database-using-entity-framework-6-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span></span> |<span data-ttu-id="af373-120">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-120">Native in client</span></span> |<span data-ttu-id="af373-121">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-121">Programmatic</span></span> |<span data-ttu-id="af373-122">Globális egy AppDomain tartományban</span><span class="sxs-lookup"><span data-stu-id="af373-122">Global per AppDomain</span></span> |<span data-ttu-id="af373-123">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-123">None</span></span> |
| <span data-ttu-id="af373-124">**[Az Entity Framework Core SQL-adatbázis](#sql-database-using-entity-framework-core-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span></span> |<span data-ttu-id="af373-125">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-125">Native in client</span></span> |<span data-ttu-id="af373-126">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-126">Programmatic</span></span> |<span data-ttu-id="af373-127">Globális egy AppDomain tartományban</span><span class="sxs-lookup"><span data-stu-id="af373-127">Global per AppDomain</span></span> |<span data-ttu-id="af373-128">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-128">None</span></span> |
| <span data-ttu-id="af373-129">**[Az ADO.NET SQL-adatbázis](#sql-database-using-adonet-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span></span> |[<span data-ttu-id="af373-130">Polly</span><span class="sxs-lookup"><span data-stu-id="af373-130">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="af373-131">Programozott és deklaratív</span><span class="sxs-lookup"><span data-stu-id="af373-131">Declarative and programmatic</span></span> |<span data-ttu-id="af373-132">Egyetlen utasítások vagy kódblokkokat</span><span class="sxs-lookup"><span data-stu-id="af373-132">Single statements or blocks of code</span></span> |<span data-ttu-id="af373-133">Egyéni</span><span class="sxs-lookup"><span data-stu-id="af373-133">Custom</span></span> |
| <span data-ttu-id="af373-134">**[Service Bus](#service-bus-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-134">**[Service Bus](#service-bus-retry-guidelines)**</span></span> |<span data-ttu-id="af373-135">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-135">Native in client</span></span> |<span data-ttu-id="af373-136">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-136">Programmatic</span></span> |<span data-ttu-id="af373-137">Namespace Manager, az üzenetkezelési gyárból és az ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-137">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="af373-138">ETW</span><span class="sxs-lookup"><span data-stu-id="af373-138">ETW</span></span> |
| <span data-ttu-id="af373-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span></span> |<span data-ttu-id="af373-140">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-140">Native in client</span></span> |<span data-ttu-id="af373-141">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-141">Programmatic</span></span> |<span data-ttu-id="af373-142">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-142">Client</span></span> |<span data-ttu-id="af373-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="af373-143">TextWriter</span></span> |
| <span data-ttu-id="af373-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span></span> |<span data-ttu-id="af373-145">Natív szolgáltatásban</span><span class="sxs-lookup"><span data-stu-id="af373-145">Native in service</span></span> |<span data-ttu-id="af373-146">Non-configurable</span><span class="sxs-lookup"><span data-stu-id="af373-146">Non-configurable</span></span> |<span data-ttu-id="af373-147">Globális</span><span class="sxs-lookup"><span data-stu-id="af373-147">Global</span></span> |<span data-ttu-id="af373-148">TraceSource</span><span class="sxs-lookup"><span data-stu-id="af373-148">TraceSource</span></span> |
| <span data-ttu-id="af373-149">**[Azure Search](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-149">**[Azure Search](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="af373-150">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-150">Native in client</span></span> |<span data-ttu-id="af373-151">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-151">Programmatic</span></span> |<span data-ttu-id="af373-152">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-152">Client</span></span> |<span data-ttu-id="af373-153">ETW vagy egyéni</span><span class="sxs-lookup"><span data-stu-id="af373-153">ETW or Custom</span></span> |
| <span data-ttu-id="af373-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span></span> |<span data-ttu-id="af373-155">Natív az ADAL-könyvtár</span><span class="sxs-lookup"><span data-stu-id="af373-155">Native in ADAL library</span></span> |<span data-ttu-id="af373-156">Az ADAL-könyvtár beágyazása</span><span class="sxs-lookup"><span data-stu-id="af373-156">Embeded into ADAL library</span></span> |<span data-ttu-id="af373-157">Belső</span><span class="sxs-lookup"><span data-stu-id="af373-157">Internal</span></span> |<span data-ttu-id="af373-158">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-158">None</span></span> |
| <span data-ttu-id="af373-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span></span> |<span data-ttu-id="af373-160">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-160">Native in client</span></span> |<span data-ttu-id="af373-161">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-161">Programmatic</span></span> |<span data-ttu-id="af373-162">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-162">Client</span></span> |<span data-ttu-id="af373-163">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-163">None</span></span> | 
| <span data-ttu-id="af373-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="af373-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span></span> |<span data-ttu-id="af373-165">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-165">Native in client</span></span> |<span data-ttu-id="af373-166">Programozott</span><span class="sxs-lookup"><span data-stu-id="af373-166">Programmatic</span></span> |<span data-ttu-id="af373-167">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="af373-167">Client</span></span> |<span data-ttu-id="af373-168">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-168">None</span></span> |

> [!NOTE]
> <span data-ttu-id="af373-169">Az Azure beépített része próbálja meg újból a mechanizmusok, jelenleg nem tudja alkalmazni a különböző típusú hiba különböző újrapróbálkozási házirendje vagy a kivétel a funkciók túl az újrapróbálkozási házirendet.</span><span class="sxs-lookup"><span data-stu-id="af373-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception beyond the functionality include in the retry policy.</span></span> <span data-ttu-id="af373-170">Ezért ajánlott útmutatások érhetők el írásának időpontjában, az optimális átlagos teljesítményt és rendelkezésre állást biztosító házirendet konfigurálhat.</span><span class="sxs-lookup"><span data-stu-id="af373-170">Therefore, the best guidance available at the time of writing is to configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="af373-171">Egy a házirend finomhangolását módja elemzése a naplófájlokat, és határozza meg az átmeneti előforduló hibák típusát.</span><span class="sxs-lookup"><span data-stu-id="af373-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> <span data-ttu-id="af373-172">Például ha hálózati problémák kapcsolódó hibák többségének, akkor előfordulhat, hogy egy azonnali újrapróbálkozási kísérlet ahelyett Várjon, amíg az első újrapróbálkozásnál hosszú ideig.</span><span class="sxs-lookup"><span data-stu-id="af373-172">For example, if the majority of errors are related to network connectivity issues, you might attempt an immediate retry rather than wait a long time for the first retry.</span></span>
>
>

## <a name="azure-storage-retry-guidelines"></a><span data-ttu-id="af373-173">Azure Storage újrapróbálkozási irányelveinek</span><span class="sxs-lookup"><span data-stu-id="af373-173">Azure Storage retry guidelines</span></span>
<span data-ttu-id="af373-174">Az Azure storage szolgáltatások közé tartoznak a tábla és a blob storage, fájlok és tárüzenetsort.</span><span class="sxs-lookup"><span data-stu-id="af373-174">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-175">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-175">Retry mechanism</span></span>
<span data-ttu-id="af373-176">Az újrapróbálkozások egyedi REST művelet szinten történik, és az ügyfél API-implementáció szerves részét képezik.</span><span class="sxs-lookup"><span data-stu-id="af373-176">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="af373-177">Az SDK az ügyfél tárolási osztályokat, hogy implementálja a [IExtendedRetryPolicy felület](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-177">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="af373-178">Nincsenek a felület másik implementációja.</span><span class="sxs-lookup"><span data-stu-id="af373-178">There are different implementations of the interface.</span></span> <span data-ttu-id="af373-179">Tárolási ügyfelek kifejezetten táblák, blobok és várólisták házirendek közül választhatnak.</span><span class="sxs-lookup"><span data-stu-id="af373-179">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="af373-180">Minden egyes használ egy másik újrapróbálkozási stratégiát, amely lényegében meghatározza az újrapróbálkozási időköz és az egyéb részleteket.</span><span class="sxs-lookup"><span data-stu-id="af373-180">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="af373-181">A beépített osztályok lineáris (állandó késedelem) és a véletlenszerű újrapróbálkozási időközökkel exponenciális támogatásához.</span><span class="sxs-lookup"><span data-stu-id="af373-181">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="af373-182">Nincs is egy újrapróbálkozási házirend használható, ha egy másik folyamat kezeli a magasabb szintű újrapróbálkozások.</span><span class="sxs-lookup"><span data-stu-id="af373-182">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="af373-183">Azonban a saját újrapróbálkozási osztályokat is létrehozható, ha nincs megadva a beépített osztályok által meghatározott követelmények.</span><span class="sxs-lookup"><span data-stu-id="af373-183">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="af373-184">Alternatív újrapróbálkozások váltás elsődleges és másodlagos tárolótömbök szolgáltatáskeresésre, ha írásvédett georedundáns tárolás (RA-GRS) használ, és a kérés eredménye egy újrapróbálkozást lehetővé tevő hiba.</span><span class="sxs-lookup"><span data-stu-id="af373-184">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="af373-185">Lásd: [Azure Storage redundancia beállítások](http://msdn.microsoft.com/library/azure/dn727290.aspx) további információt.</span><span class="sxs-lookup"><span data-stu-id="af373-185">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="af373-186">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="af373-186">Policy configuration</span></span>
<span data-ttu-id="af373-187">Újrapróbálkozási házirendek programozott módon.</span><span class="sxs-lookup"><span data-stu-id="af373-187">Retry policies are configured programmatically.</span></span> <span data-ttu-id="af373-188">Egy tipikus eljárás akkor hozhat létre és tölthet egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, vagy **QueueRequestOptions**  példány.</span><span class="sxs-lookup"><span data-stu-id="af373-188">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="af373-189">A kérelem beállítások példány majd beállítható az ügyfélen, és az ügyfél összes művelet a megadott beállításokat fogja használni.</span><span class="sxs-lookup"><span data-stu-id="af373-189">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="af373-190">Az ügyfél beállításokat lehet felülbírálni úgy, hogy a kérelem beállítások osztály feltöltött példány paraméterként művelet módszerek.</span><span class="sxs-lookup"><span data-stu-id="af373-190">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="af373-191">Használhat egy **OperationContext** példány, adja meg a kódot a végrehajtási újbóli kísérlet esetén, és ha egy művelet befejeződött.</span><span class="sxs-lookup"><span data-stu-id="af373-191">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="af373-192">Ez a kód össze tudják gyűjteni a naplókat és telemetriai működésével kapcsolatos adatokat.</span><span class="sxs-lookup"><span data-stu-id="af373-192">This code can collect information about the operation for use in logs and telemetry.</span></span>

    // Set up notifications for an operation
    var context = new OperationContext();
    context.ClientRequestID = "some request id";
    context.Retrying += (sender, args) =>
    {
      /* Collect retry information */
    };
    context.RequestCompleted += (sender, args) =>
    {
      /* Collect operation completion information */
    };
    var stats = await client.GetServiceStatsAsync(null, context);

<span data-ttu-id="af373-193">Mellett jelző hiba megfelelő-e az újra gombra, a kiterjesztett újrapróbálkozási házirendek vissza egy **RetryContext** objektum, amely jelzi, hogy a legutóbbi kérelem eredményeit, az újrapróbálkozások száma a legközelebbi újrapróbálkozás fordul elő az elsődleges vagy másodlagos helyre (lásd az alábbi táblázatban részletei).</span><span class="sxs-lookup"><span data-stu-id="af373-193">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="af373-194">A tulajdonságait a **RetryContext** objektum segítségével döntse el, ha, és ha egy újabb kísérletet.</span><span class="sxs-lookup"><span data-stu-id="af373-194">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="af373-195">További részletekért lásd: [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-195">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="af373-196">Az alábbi táblázatok bemutatják az alapértelmezett beállításokat az a beépített házirendek próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="af373-196">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="af373-197">**Lehetőségek**</span><span class="sxs-lookup"><span data-stu-id="af373-197">**Request options**</span></span>

| <span data-ttu-id="af373-198">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="af373-198">**Setting**</span></span> | <span data-ttu-id="af373-199">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="af373-199">**Default value**</span></span> | <span data-ttu-id="af373-200">**Jelentése**</span><span class="sxs-lookup"><span data-stu-id="af373-200">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="af373-201">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="af373-201">MaximumExecutionTime</span></span> | <span data-ttu-id="af373-202">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-202">120 seconds</span></span> | <span data-ttu-id="af373-203">A kérelem, beleértve az összes lehetséges újrapróbálkozási kísérletek maximális végrehajtási ideje.</span><span class="sxs-lookup"><span data-stu-id="af373-203">Maximum execution time for the request, including all potential retry attempts.</span></span> |
| <span data-ttu-id="af373-204">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="af373-204">ServerTimeout</span></span> | <span data-ttu-id="af373-205">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-205">None</span></span> | <span data-ttu-id="af373-206">A kérés a kiszolgáló időkorlátja (kerekített másodperc).</span><span class="sxs-lookup"><span data-stu-id="af373-206">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="af373-207">Ha nincs megadva, az alapértelmezett érték a kiszolgáló összes kérelem fog használni.</span><span class="sxs-lookup"><span data-stu-id="af373-207">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="af373-208">Általában a legjobb lehetőség egy hagyja ki ezt a beállítást, hogy a kiszolgáló alapértelmezett szolgál.</span><span class="sxs-lookup"><span data-stu-id="af373-208">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="af373-209">LocationMode</span><span class="sxs-lookup"><span data-stu-id="af373-209">LocationMode</span></span> | <span data-ttu-id="af373-210">Nincs</span><span class="sxs-lookup"><span data-stu-id="af373-210">None</span></span> | <span data-ttu-id="af373-211">A tárfiók létrejön az írásvédett georedundáns tárolás (RA-GRS) replikációs beállítás, ha a hely mód segítségével jelzi, hogy melyik helyen kell kapnia a kérelmet.</span><span class="sxs-lookup"><span data-stu-id="af373-211">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="af373-212">Például ha **PrimaryThenSecondary** van megadva, kérelmek mindig küldi el az elsődleges helyre először.</span><span class="sxs-lookup"><span data-stu-id="af373-212">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="af373-213">A kérés nem teljesíthető, ha a másodlagos helyre való továbbítás.</span><span class="sxs-lookup"><span data-stu-id="af373-213">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="af373-214">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="af373-214">RetryPolicy</span></span> | <span data-ttu-id="af373-215">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="af373-215">ExponentialPolicy</span></span> | <span data-ttu-id="af373-216">További információ alább olvasható az egyes lehetőségek.</span><span class="sxs-lookup"><span data-stu-id="af373-216">See below for details of each option.</span></span> |

<span data-ttu-id="af373-217">**Az exponenciális házirend**</span><span class="sxs-lookup"><span data-stu-id="af373-217">**Exponential policy**</span></span> 

| <span data-ttu-id="af373-218">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="af373-218">**Setting**</span></span> | <span data-ttu-id="af373-219">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="af373-219">**Default value**</span></span> | <span data-ttu-id="af373-220">**Jelentése**</span><span class="sxs-lookup"><span data-stu-id="af373-220">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="af373-221">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="af373-221">maxAttempt</span></span> | <span data-ttu-id="af373-222">3</span><span class="sxs-lookup"><span data-stu-id="af373-222">3</span></span> | <span data-ttu-id="af373-223">Újrapróbálkozások száma.</span><span class="sxs-lookup"><span data-stu-id="af373-223">Number of retry attempts.</span></span> |
| <span data-ttu-id="af373-224">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-224">deltaBackoff</span></span> | <span data-ttu-id="af373-225">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-225">4 seconds</span></span> | <span data-ttu-id="af373-226">Vissza az indító időköz újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="af373-226">Back-off interval between retries.</span></span> <span data-ttu-id="af373-227">A TimeSpan érték, egy véletlenszerű elem, beleértve többszörösei későbbi újrapróbálkozás használható.</span><span class="sxs-lookup"><span data-stu-id="af373-227">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="af373-228">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-228">MinBackoff</span></span> | <span data-ttu-id="af373-229">3 másodpercenként</span><span class="sxs-lookup"><span data-stu-id="af373-229">3 seconds</span></span> | <span data-ttu-id="af373-230">Hozzáadandó deltaBackoff alapján kiszámított intervallumokban próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="af373-230">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="af373-231">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="af373-231">This value cannot be changed.</span></span>
| <span data-ttu-id="af373-232">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-232">MaxBackoff</span></span> | <span data-ttu-id="af373-233">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-233">120 seconds</span></span> | <span data-ttu-id="af373-234">MaxBackoff akkor használatos, ha az számított újrapróbálkozási időköz érték nagyobb, mint MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="af373-234">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="af373-235">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="af373-235">This value cannot be changed.</span></span> |

<span data-ttu-id="af373-236">**Lineáris házirend**</span><span class="sxs-lookup"><span data-stu-id="af373-236">**Linear policy**</span></span>

| <span data-ttu-id="af373-237">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="af373-237">**Setting**</span></span> | <span data-ttu-id="af373-238">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="af373-238">**Default value**</span></span> | <span data-ttu-id="af373-239">**Jelentése**</span><span class="sxs-lookup"><span data-stu-id="af373-239">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="af373-240">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="af373-240">maxAttempt</span></span> | <span data-ttu-id="af373-241">3</span><span class="sxs-lookup"><span data-stu-id="af373-241">3</span></span> | <span data-ttu-id="af373-242">Újrapróbálkozások száma.</span><span class="sxs-lookup"><span data-stu-id="af373-242">Number of retry attempts.</span></span> |
| <span data-ttu-id="af373-243">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-243">deltaBackoff</span></span> | <span data-ttu-id="af373-244">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-244">30 seconds</span></span> | <span data-ttu-id="af373-245">Vissza az indító időköz újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="af373-245">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="af373-246">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="af373-246">Retry usage guidance</span></span>
<span data-ttu-id="af373-247">A tárolási ügyfél API-ja használatával az Azure storage szolgáltatások elérésekor, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="af373-247">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="af373-248">Házirendekkel a automatikusan újrapróbálkozik a Microsoft.WindowsAzure.Storage.RetryPolicies névtér hol a követelményeknek megfelelő.</span><span class="sxs-lookup"><span data-stu-id="af373-248">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="af373-249">A legtöbb esetben ezek a házirendek elegendő lesz.</span><span class="sxs-lookup"><span data-stu-id="af373-249">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="af373-250">Használja a **ExponentialRetry** kötegműveletek, háttérfeladatok vagy nem interaktív forgatókönyvek házirend.</span><span class="sxs-lookup"><span data-stu-id="af373-250">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="af373-251">Ezekben az esetekben általában lehetővé teszi a több időt a szolgáltatás helyreállítása – következésképpen megnövekedett arra, a művelet, végül sikeresen lezajlottak.</span><span class="sxs-lookup"><span data-stu-id="af373-251">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="af373-252">Érdemes megadni a **MaximumExecutionTime** tulajdonsága a **RequestOptions** korlátozza a teljes végrehajtási idő, de vegye figyelembe a típusát és méretét, a művelet kiválasztásakor paraméter egy időtúllépési értéket.</span><span class="sxs-lookup"><span data-stu-id="af373-252">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="af373-253">Ha egy egyéni újra megvalósításához szüksége, ne hozzon létre olyan burkolók a tárolási ügyfél osztályok körül.</span><span class="sxs-lookup"><span data-stu-id="af373-253">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="af373-254">Használja a képességek kiterjesztése a meglévő házirendek segítségével a **IExtendedRetryPolicy** felületet.</span><span class="sxs-lookup"><span data-stu-id="af373-254">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="af373-255">Ha írásvédett georedundáns tárolás (RA-GRS) használ a **LocationMode** adhatja meg, hogy újból megkísérli érik el a másodlagos csak olvasható másolatát az áruházból kell az elsődleges hozzáférési sikertelen.</span><span class="sxs-lookup"><span data-stu-id="af373-255">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="af373-256">Azonban ez a beállítás használata esetén győződjön meg, hogy az alkalmazás képes sikeresen adatokkal dolgozni, amelyek esetleg elavult, ha az elsődleges áruházból replikációs még nem fejeződött be.</span><span class="sxs-lookup"><span data-stu-id="af373-256">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="af373-257">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="af373-257">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="af373-258">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="af373-258">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="af373-259">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="af373-259">**Context**</span></span> | <span data-ttu-id="af373-260">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="af373-260">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="af373-261">**Ismételje meg a házirend**</span><span class="sxs-lookup"><span data-stu-id="af373-261">**Retry policy**</span></span> | <span data-ttu-id="af373-262">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="af373-262">**Settings**</span></span> | <span data-ttu-id="af373-263">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="af373-263">**Values**</span></span> | <span data-ttu-id="af373-264">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="af373-264">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="af373-265">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="af373-265">Interactive, UI,</span></span><br /><span data-ttu-id="af373-266">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="af373-266">or foreground</span></span> |<span data-ttu-id="af373-267">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-267">2 seconds</span></span> |<span data-ttu-id="af373-268">Lineáris</span><span class="sxs-lookup"><span data-stu-id="af373-268">Linear</span></span> |<span data-ttu-id="af373-269">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="af373-269">maxAttempt</span></span><br /><span data-ttu-id="af373-270">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-270">deltaBackoff</span></span> |<span data-ttu-id="af373-271">3</span><span class="sxs-lookup"><span data-stu-id="af373-271">3</span></span><br /><span data-ttu-id="af373-272">500 ms</span><span class="sxs-lookup"><span data-stu-id="af373-272">500 ms</span></span> |<span data-ttu-id="af373-273">Kísérlet történt az 1 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-273">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="af373-274">Kísérlet történt a 2 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-274">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="af373-275">Próbálja meg a 3 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-275">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="af373-276">Háttér</span><span class="sxs-lookup"><span data-stu-id="af373-276">Background</span></span><br /><span data-ttu-id="af373-277">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="af373-277">or batch</span></span> |<span data-ttu-id="af373-278">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-278">30 seconds</span></span> |<span data-ttu-id="af373-279">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-279">Exponential</span></span> |<span data-ttu-id="af373-280">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="af373-280">maxAttempt</span></span><br /><span data-ttu-id="af373-281">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-281">deltaBackoff</span></span> |<span data-ttu-id="af373-282">5</span><span class="sxs-lookup"><span data-stu-id="af373-282">5</span></span><br /><span data-ttu-id="af373-283">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-283">4 seconds</span></span> |<span data-ttu-id="af373-284">Próbálja meg az 1 – ~ 3 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-284">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="af373-285">Kísérlet történt a 2 – ~ 7 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-285">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="af373-286">Próbálja meg a 3 - késleltetés ~ 15 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-286">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="af373-287">Telemetria</span><span class="sxs-lookup"><span data-stu-id="af373-287">Telemetry</span></span>
<span data-ttu-id="af373-288">Újrapróbálkozás a rendszer naplózza a **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="af373-288">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="af373-289">Konfigurálnia kell egy **TraceListener** az események rögzítése és a megfelelő cél a naplófájlba írja őket.</span><span class="sxs-lookup"><span data-stu-id="af373-289">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="af373-290">Használhatja a **értékének a TextWriterTraceListener figyelőre** vagy **XmlWriterTraceListener** írni egy naplófájlba, a **EventLogTraceListener** írni a Windows eseménynaplóba, vagy a **EventProviderTraceListener** nyomkövetési adatokat írni az ETW alrendszer.</span><span class="sxs-lookup"><span data-stu-id="af373-290">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="af373-291">Automatikus kiürítése a puffer, és az eseményeket, amelyek kerül a naplóba (például hiba, figyelmeztetés, tájékoztatás, és részletes) részletességi is konfigurálhatja.</span><span class="sxs-lookup"><span data-stu-id="af373-291">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="af373-292">További információkért lásd: [ügyféloldali naplózás a .NET a Storage ügyféloldali kódtára](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-292">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="af373-293">Műveletek fogadhat egy **OperationContext** példány, mely tesz elérhetővé a **újrapróbálkozás** esemény használható egyéni telemetria logika csatlakoztatni.</span><span class="sxs-lookup"><span data-stu-id="af373-293">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="af373-294">További információkért lásd: [OperationContext.Retrying esemény](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-294">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="af373-295">Példák</span><span class="sxs-lookup"><span data-stu-id="af373-295">Examples</span></span>
<span data-ttu-id="af373-296">Az alábbi példakód bemutatja, hogyan hozható létre kettő **TableRequestOptions** különböző újrapróbálkozásos beállításokkal példányok; interaktív kérelmek egyet, majd egy a háttér-kérelmekre.</span><span class="sxs-lookup"><span data-stu-id="af373-296">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="af373-297">A példa majd a készlet e két próbálja meg újra az ügyfél-házirendeket, hogy minden olyan kérelem esetében alkalmazni, és az interaktív stratégia is állít be egy adott kérelem, hogy felülírja az ügyfél alapértelmezett beállításait.</span><span class="sxs-lookup"><span data-stu-id="af373-297">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="af373-298">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-298">More information</span></span>
* [<span data-ttu-id="af373-299">Az Azure Storage ügyfél könyvtár újrapróbálkozási házirend javaslatok</span><span class="sxs-lookup"><span data-stu-id="af373-299">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="af373-300">A Storage ügyféloldali kódtára 2.0 – újrapróbálkozási házirendek megvalósítása</span><span class="sxs-lookup"><span data-stu-id="af373-300">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a><span data-ttu-id="af373-301">SQL-adatbázis Entity Framework 6 újrapróbálkozási szerint</span><span class="sxs-lookup"><span data-stu-id="af373-301">SQL Database using Entity Framework 6 retry guidelines</span></span>
<span data-ttu-id="af373-302">SQL-adatbázis egy olyan üzemeltetett SQL-adatbázis széles méretű és (megosztott) standard és premium (nem megosztott) szolgáltatás is elérhető.</span><span class="sxs-lookup"><span data-stu-id="af373-302">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="af373-303">Entitás-keretrendszer egy objektum relációs leképező, amely lehetővé teszi a .NET-fejlesztők relációs adatok tartományspecifikus-objektumok segítségével.</span><span class="sxs-lookup"><span data-stu-id="af373-303">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="af373-304">Szükségtelenné teszi, a legtöbb a fejlesztők általában kell írnia az adat-hozzáférési kód.</span><span class="sxs-lookup"><span data-stu-id="af373-304">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-305">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-305">Retry mechanism</span></span>
<span data-ttu-id="af373-306">Újrapróbálkozási támogatás áll rendelkezésre, használja az Entity Framework 6.0 SQL-adatbázis elérésekor és magasabb mechanizmuson keresztül nevű [kapcsolati rugalmasságot / újra logika](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-306">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="af373-307">Az újrapróbálkozási mechanizmus a fő jellemzői a következők:</span><span class="sxs-lookup"><span data-stu-id="af373-307">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="af373-308">Az elsődleges absztrakciós van a **IDbExecutionStrategy** felületet.</span><span class="sxs-lookup"><span data-stu-id="af373-308">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="af373-309">Ez az interfész:</span><span class="sxs-lookup"><span data-stu-id="af373-309">This interface:</span></span>
  * <span data-ttu-id="af373-310">Meghatározza a szinkron és aszinkron **Execute**\* módszerek.</span><span class="sxs-lookup"><span data-stu-id="af373-310">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="af373-311">Meghatározza az osztályokat is használható közvetlenül vagy egy adatbázis-környezet, mint egy alapértelmezett stratégia konfigurált, leképezve a szolgáltató neve, vagy leképezve a szolgáltató nevét és a kiszolgáló nevét.</span><span class="sxs-lookup"><span data-stu-id="af373-311">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="af373-312">Ha a környezet konfigurálva, próbálkozások egyéni adatbázis-művelet, amelynek lehet több szinten egy adott környezetben a művelethez.</span><span class="sxs-lookup"><span data-stu-id="af373-312">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="af373-313">Határozza meg, majd ismételje meg a sikertelen kapcsolódás, mikor és hogyan.</span><span class="sxs-lookup"><span data-stu-id="af373-313">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="af373-314">Ez magában foglalja a számos beépített implementációja a **IDbExecutionStrategy** felület:</span><span class="sxs-lookup"><span data-stu-id="af373-314">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="af373-315">Alapértelmezett – nincs újrapróbálkozás.</span><span class="sxs-lookup"><span data-stu-id="af373-315">Default - no retrying.</span></span>
  * <span data-ttu-id="af373-316">Alapértelmezett SQL-adatbázis (automatikus) – nincs újrapróbálkozás, de kivételek megvizsgálja, és azokat a javaslatát, hogy az SQL-adatbázis stratégia burkolja.</span><span class="sxs-lookup"><span data-stu-id="af373-316">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="af373-317">SQL-adatbázis észlelési logika és az alapértelmezett SQL-adatbázis - exponenciális (alaposztály öröklődik).</span><span class="sxs-lookup"><span data-stu-id="af373-317">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="af373-318">Az exponenciális vissza ki stratégiát véletlenszerűsítésének valósítja meg.</span><span class="sxs-lookup"><span data-stu-id="af373-318">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="af373-319">A beépített újrapróbálkozási osztályok állapot-nyilvántartó és nem többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="af373-319">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="af373-320">Azonban ismét felhasználhatók a jelenlegi művelet befejeződése után.</span><span class="sxs-lookup"><span data-stu-id="af373-320">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="af373-321">Ha a megadott újrapróbálkozások maximális számát, az eredményeket egy új kivétel van csomagolni.</span><span class="sxs-lookup"><span data-stu-id="af373-321">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="af373-322">Nem a jelenlegi kivétel mentése buborék.</span><span class="sxs-lookup"><span data-stu-id="af373-322">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="af373-323">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="af373-323">Policy configuration</span></span>
<span data-ttu-id="af373-324">Újrapróbálkozási nyújtott támogatás a használó Entity Framework 6.0 vagy újabb SQL-adatbázis elérésekor.</span><span class="sxs-lookup"><span data-stu-id="af373-324">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="af373-325">Újrapróbálkozási házirendek programozott módon.</span><span class="sxs-lookup"><span data-stu-id="af373-325">Retry policies are configured programmatically.</span></span> <span data-ttu-id="af373-326">A konfiguráció nem módosítható művelet alapon.</span><span class="sxs-lookup"><span data-stu-id="af373-326">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="af373-327">A környezet alapértelmezett stratégia konfigurálásakor meg kell adnia egy függvényt, amely létrehoz egy új stratégia igény szerint.</span><span class="sxs-lookup"><span data-stu-id="af373-327">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="af373-328">A következő kód bemutatja, hogyan hozhat létre egy újrapróbálkozási configuration osztály, amely kiterjeszti a **DbConfiguration** alaposztály.</span><span class="sxs-lookup"><span data-stu-id="af373-328">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="af373-329">Ezután megadhatja ezt az alapértelmezett újrapróbálkozási stratégia segítségével minden műveletnél a **SetConfiguration** metódusában a **DbConfiguration** példány az alkalmazás indításakor.</span><span class="sxs-lookup"><span data-stu-id="af373-329">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="af373-330">Alapértelmezés szerint EF automatikusan felderíti és a configuration osztály használata.</span><span class="sxs-lookup"><span data-stu-id="af373-330">By default, EF will automatically discover and use the configuration class.</span></span>

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

<span data-ttu-id="af373-331">Az újrapróbálkozási configuration osztály kontextus megadhatja a környezeti osztályt ellátása megjegyzésekkel egy **DbConfigurationType** attribútum.</span><span class="sxs-lookup"><span data-stu-id="af373-331">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="af373-332">Azonban ha csak egy konfigurációs osztály, EF használ, nincs szükség a környezet megjegyzésekkel.</span><span class="sxs-lookup"><span data-stu-id="af373-332">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

<span data-ttu-id="af373-333">Ha különböző újrapróbálkozásos stratégiák műveleteket, vagy tiltsa le a konkrét műveletek az újrapróbálkozások van szüksége, a configuration osztály, amely lehetővé teszi, hogy felfüggeszti vagy stratégiák felcserélése jelölő beállításával hozhat létre a **CallContext** .</span><span class="sxs-lookup"><span data-stu-id="af373-333">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="af373-334">A configuration osztály stratégiák váltáshoz használja ezt a jelzőt, vagy tiltsa le a stratégiát, adja meg, és egy alapértelmezett stratégia használatára.</span><span class="sxs-lookup"><span data-stu-id="af373-334">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="af373-335">További információkért lásd: [felfüggesztése végrehajtási stratégia](http://msdn.microsoft.com/dn307226#transactions_workarounds) korlátozásai a végrehajtási stratégiák újrapróbálkozás (EF6 és újabb verziók esetében) lapján.</span><span class="sxs-lookup"><span data-stu-id="af373-335">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="af373-336">A megadott újrapróbálkozási stratégiák használata egyéni műveletek egy másik módszer akkor létrehozni a szükséges stratégia osztály egy példányát, és adja meg a kívánt beállításokat paramétereknek.</span><span class="sxs-lookup"><span data-stu-id="af373-336">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="af373-337">Majd meghívása a **ExecuteAsync** metódust.</span><span class="sxs-lookup"><span data-stu-id="af373-337">You then invoke its **ExecuteAsync** method.</span></span>

    var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
    var blogs = await executionStrategy.ExecuteAsync(
        async () =>
        {
            using (var db = new BloggingContext("Blogs"))
            {
                // Acquire some values asynchronously and return them
            }
        },
        new CancellationToken()
    );

<span data-ttu-id="af373-338">A legegyszerűbben úgy használja egy **DbConfiguration** ugyanabban a szerelvényben, keresse meg az osztály a **DbContext** osztály.</span><span class="sxs-lookup"><span data-stu-id="af373-338">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="af373-339">Ez azonban nem megfelelő ugyanabban a környezetben van szükség esetén különböző helyzetekben, például a különböző interaktív és a háttér újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="af373-339">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="af373-340">A különböző környezetek külön alkalmazástartományok hajtható végre, ha a beépített támogatást használja osztályok a konfigurációs fájlban, vagy állítsa be explicit módon a kód használatával.</span><span class="sxs-lookup"><span data-stu-id="af373-340">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="af373-341">A különböző környezetekben az AppDomain végre kell hajtani, ha egy egyéni megoldás lesz szükség.</span><span class="sxs-lookup"><span data-stu-id="af373-341">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="af373-342">További információkért lásd: [kód-alapú konfigurációban (EF6 és újabb verziók esetében)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-342">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="af373-343">A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet EF6 használatakor.</span><span class="sxs-lookup"><span data-stu-id="af373-343">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="af373-344">Beállítás</span><span class="sxs-lookup"><span data-stu-id="af373-344">Setting</span></span> | <span data-ttu-id="af373-345">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="af373-345">Default value</span></span> | <span data-ttu-id="af373-346">Jelentése</span><span class="sxs-lookup"><span data-stu-id="af373-346">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="af373-347">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="af373-347">Policy</span></span> | <span data-ttu-id="af373-348">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-348">Exponential</span></span> | <span data-ttu-id="af373-349">Az exponenciális vissza-ki.</span><span class="sxs-lookup"><span data-stu-id="af373-349">Exponential back-off.</span></span> |
| <span data-ttu-id="af373-350">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="af373-350">MaxRetryCount</span></span> | <span data-ttu-id="af373-351">5</span><span class="sxs-lookup"><span data-stu-id="af373-351">5</span></span> | <span data-ttu-id="af373-352">Az újrapróbálkozások maximális számát.</span><span class="sxs-lookup"><span data-stu-id="af373-352">The maximum number of retries.</span></span> |
| <span data-ttu-id="af373-353">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="af373-353">MaxDelay</span></span> | <span data-ttu-id="af373-354">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-354">30 seconds</span></span> | <span data-ttu-id="af373-355">A maximális késleltetés, újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="af373-355">The maximum delay between retries.</span></span> <span data-ttu-id="af373-356">Ez az érték nem befolyásolja, hogyan késések sorozatát arra az esetre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="af373-356">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="af373-357">Csak egy felső határa határozza meg.</span><span class="sxs-lookup"><span data-stu-id="af373-357">It only defines an upper bound.</span></span> |
| <span data-ttu-id="af373-358">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="af373-358">DefaultCoefficient</span></span> | <span data-ttu-id="af373-359">1 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-359">1 second</span></span> | <span data-ttu-id="af373-360">Az exponenciális vissza az indító számítási együttható.</span><span class="sxs-lookup"><span data-stu-id="af373-360">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="af373-361">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="af373-361">This value cannot be changed.</span></span> |
| <span data-ttu-id="af373-362">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="af373-362">DefaultRandomFactor</span></span> | <span data-ttu-id="af373-363">1.1</span><span class="sxs-lookup"><span data-stu-id="af373-363">1.1</span></span> | <span data-ttu-id="af373-364">A többszöröző segítségével vegye fel azokat a véletlenszerű késleltetés minden bejegyzéshez.</span><span class="sxs-lookup"><span data-stu-id="af373-364">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="af373-365">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="af373-365">This value cannot be changed.</span></span> |
| <span data-ttu-id="af373-366">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="af373-366">DefaultExponentialBase</span></span> | <span data-ttu-id="af373-367">2</span><span class="sxs-lookup"><span data-stu-id="af373-367">2</span></span> | <span data-ttu-id="af373-368">A többszöröző, a következő késleltetési kiszámításához használt.</span><span class="sxs-lookup"><span data-stu-id="af373-368">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="af373-369">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="af373-369">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="af373-370">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="af373-370">Retry usage guidance</span></span>
<span data-ttu-id="af373-371">EF6 használatával SQL-adatbázis elérésekor, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="af373-371">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="af373-372">Válassza ki a megfelelő szolgáltatásra beállítás (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="af373-372">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="af373-373">Egy megosztott példánnyal hosszabb, mint a szokásos csatlakozási késedelem és szabályozási más bérlők megosztott kiszolgáló kihasználtsága miatt romolhat.</span><span class="sxs-lookup"><span data-stu-id="af373-373">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="af373-374">Ha kiszámítható teljesítményt és a megbízható kis késleltetésű műveletek szükség, vegye figyelembe, premium-lehetőség kiválasztásában.</span><span class="sxs-lookup"><span data-stu-id="af373-374">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="af373-375">A rögzített időközönként stratégia használható az Azure SQL Database nem ajánlott.</span><span class="sxs-lookup"><span data-stu-id="af373-375">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="af373-376">Ehelyett használja a exponenciális vissza az indító stratégia, mert előfordulhat, hogy túlterheltté vált a szolgáltatást, és hosszabb engedélyezése ahhoz, hogy a helyreállítás több időt.</span><span class="sxs-lookup"><span data-stu-id="af373-376">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="af373-377">Válassza ki a kapcsolat és a parancs időtúllépések megfelelő értéket, kapcsolatok meghatározásakor.</span><span class="sxs-lookup"><span data-stu-id="af373-377">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="af373-378">Az időtúllépés kiinduló, az üzleti logika tervezésével és a teszteléshez.</span><span class="sxs-lookup"><span data-stu-id="af373-378">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="af373-379">Ez az érték módosításához idővel a mennyiségű adatot szeretne, vagy módosítsa az üzleti folyamatokat.</span><span class="sxs-lookup"><span data-stu-id="af373-379">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="af373-380">Túl rövid időtúllépés kapcsolatok korai hibák eredményezhet, amikor az adatbázis foglalt.</span><span class="sxs-lookup"><span data-stu-id="af373-380">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="af373-381">Túl hosszú időtúllépés előfordulhat, hogy megfelelően működik-e túl hosszú a sikertelen kapcsolódás észlelése előtti várakozási által újrapróbálkozási logika.</span><span class="sxs-lookup"><span data-stu-id="af373-381">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="af373-382">Az időtúllépés értéke a végpontok közötti késés összetevője bár nem lehet egyszerűen megállapítani hány parancsok hajtja végre, a környezet mentése során.</span><span class="sxs-lookup"><span data-stu-id="af373-382">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="af373-383">Az alapértelmezett időtúllépési módosíthatja úgy, hogy a **CommandTimeout** tulajdonsága a **DbContext** példány.</span><span class="sxs-lookup"><span data-stu-id="af373-383">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="af373-384">Entitás-keretrendszer támogatja a konfigurációs fájlok meghatározott konfigurációk próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="af373-384">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="af373-385">Azonban a maximális rugalmasság Azure-érdemes létrehozni programozott módon az alkalmazáson belül a konfigurációs.</span><span class="sxs-lookup"><span data-stu-id="af373-385">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="af373-386">Az adott paraméterek újrapróbálkozási-házirendek, például az újrapróbálkozások és az újrapróbálkozási intervallumok száma a szolgáltatás konfigurációs fájljában tárolt, és futási időben a megfelelő házirendek létrehozására szolgál.</span><span class="sxs-lookup"><span data-stu-id="af373-386">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="af373-387">Ez lehetővé teszi, hogy a beállításokat, anélkül, hogy újra kell indítani az alkalmazást módosítani.</span><span class="sxs-lookup"><span data-stu-id="af373-387">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="af373-388">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="af373-388">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="af373-389">A késleltetés, újrapróbálkozások (le, és létrehozott egy exponenciális sorozatot) között nem adható meg.</span><span class="sxs-lookup"><span data-stu-id="af373-389">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="af373-390">Csak a maximális értékeket adhat meg, ahogy Itt; Ha létrehoz egy egyéni újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="af373-390">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="af373-391">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="af373-391">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="af373-392">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="af373-392">**Context**</span></span> | <span data-ttu-id="af373-393">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="af373-393">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="af373-394">**Ismételje meg a házirend**</span><span class="sxs-lookup"><span data-stu-id="af373-394">**Retry policy**</span></span> | <span data-ttu-id="af373-395">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="af373-395">**Settings**</span></span> | <span data-ttu-id="af373-396">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="af373-396">**Values**</span></span> | <span data-ttu-id="af373-397">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="af373-397">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="af373-398">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="af373-398">Interactive, UI,</span></span><br /><span data-ttu-id="af373-399">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="af373-399">or foreground</span></span> |<span data-ttu-id="af373-400">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-400">2 seconds</span></span> |<span data-ttu-id="af373-401">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-401">Exponential</span></span> |<span data-ttu-id="af373-402">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="af373-402">MaxRetryCount</span></span><br /><span data-ttu-id="af373-403">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="af373-403">MaxDelay</span></span> |<span data-ttu-id="af373-404">3</span><span class="sxs-lookup"><span data-stu-id="af373-404">3</span></span><br /><span data-ttu-id="af373-405">750 ms</span><span class="sxs-lookup"><span data-stu-id="af373-405">750 ms</span></span> |<span data-ttu-id="af373-406">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-406">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="af373-407">Kísérlet történt a 2 – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-407">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="af373-408">Próbálja meg a 3 – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-408">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="af373-409">Háttér</span><span class="sxs-lookup"><span data-stu-id="af373-409">Background</span></span><br /> <span data-ttu-id="af373-410">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="af373-410">or batch</span></span> |<span data-ttu-id="af373-411">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-411">30 seconds</span></span> |<span data-ttu-id="af373-412">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-412">Exponential</span></span> |<span data-ttu-id="af373-413">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="af373-413">MaxRetryCount</span></span><br /><span data-ttu-id="af373-414">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="af373-414">MaxDelay</span></span> |<span data-ttu-id="af373-415">5</span><span class="sxs-lookup"><span data-stu-id="af373-415">5</span></span><br /><span data-ttu-id="af373-416">12 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-416">12 seconds</span></span> |<span data-ttu-id="af373-417">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-417">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="af373-418">Kísérlet történt a 2 - ~ 1 másodperc késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-418">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="af373-419">Próbálja meg a 3 - ~ 3 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-419">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="af373-420">Kísérlet történt a 4 – ~ 7 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-420">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="af373-421">Kísérlet történt az 5 – 12 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-421">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="af373-422">A végpontok közötti késés célokat azt feltételezik, hogy a szolgáltatáshoz való csatlakozásokat az alapértelmezett időkorlátját.</span><span class="sxs-lookup"><span data-stu-id="af373-422">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="af373-423">Ha hosszabb ideig kapcsolat időtúllépése ad meg, a végpontok közötti késés minden újrapróbálkozási kísérlethez további időpontig bővíthető.</span><span class="sxs-lookup"><span data-stu-id="af373-423">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="af373-424">Példák</span><span class="sxs-lookup"><span data-stu-id="af373-424">Examples</span></span>
<span data-ttu-id="af373-425">Az alábbi példakód egy egyszerű adat-hozzáférési megoldás, amely használja az Entity Framework határozza meg.</span><span class="sxs-lookup"><span data-stu-id="af373-425">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="af373-426">Egy adott újrapróbálkozási stratégiát állítja a nevű osztály egy példányát meghatározásával **BlogConfiguration** , amely kiterjeszti **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="af373-426">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="af373-427">Az Entity Framework újrapróbálkozási mechanizmussal további példái megtalálhatók [kapcsolati rugalmasságot / újra logika](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-427">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="af373-428">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-428">More information</span></span>
* [<span data-ttu-id="af373-429">Az Azure SQL adatbázis-teljesítményt és a rugalmasság útmutatója</span><span class="sxs-lookup"><span data-stu-id="af373-429">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a><span data-ttu-id="af373-430">SQL-adatbázis Entity Framework Core újrapróbálkozási szerint</span><span class="sxs-lookup"><span data-stu-id="af373-430">SQL Database using Entity Framework Core retry guidelines</span></span>
<span data-ttu-id="af373-431">[Entity Framework Core](/ef/core/) van egy objektum relációs leképező, amely lehetővé teszi a .NET Core fejlesztők objektumok, tartomány-specifikus adatait.</span><span class="sxs-lookup"><span data-stu-id="af373-431">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="af373-432">Szükségtelenné teszi, a legtöbb a fejlesztők általában kell írnia az adat-hozzáférési kód.</span><span class="sxs-lookup"><span data-stu-id="af373-432">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="af373-433">Az Entity Framework jelen verziójában alapoktól készült, és nem automatikusan örökli a szolgáltatások EF6.x.</span><span class="sxs-lookup"><span data-stu-id="af373-433">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-434">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-434">Retry mechanism</span></span>
<span data-ttu-id="af373-435">Újrapróbálkozási támogatást biztosított használatával Entity Framework Core mechanizmus segítségével SQL-adatbázis nevű [kapcsolati rugalmasságot](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="af373-435">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="af373-436">Kapcsolat rugalmassága az EF Core 1.1.0-ás lett bevezetve.</span><span class="sxs-lookup"><span data-stu-id="af373-436">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="af373-437">Az elsődleges absztrakciós van a `IExecutionStrategy` felületet.</span><span class="sxs-lookup"><span data-stu-id="af373-437">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="af373-438">A végrehajtási stratégiát az SQL Server, beleértve az SQL Azure, vegye figyelembe a kivételt típusú követően újra megkísérelhető és ésszerű alapértelmezett értéket az újrapróbálkozások maximális számát, vonatkozó felszólítások közötti idő újrapróbálkozások, és így tovább.</span><span class="sxs-lookup"><span data-stu-id="af373-438">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="af373-439">Példák</span><span class="sxs-lookup"><span data-stu-id="af373-439">Examples</span></span>

<span data-ttu-id="af373-440">Az alábbi kód lehetővé teszi, hogy automatikus újrapróbálkozások a DbContext objektum, amely képviseli egy munkamenetet az adatbázis konfigurálásakor.</span><span class="sxs-lookup"><span data-stu-id="af373-440">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="af373-441">A következő kód bemutatja, hogyan hajtható végre, az automatikus újrapróbálkozásokat tranzakciót egy végrehajtási stratégia használatával.</span><span class="sxs-lookup"><span data-stu-id="af373-441">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="af373-442">A tranzakció egy delegált van meghatározva.</span><span class="sxs-lookup"><span data-stu-id="af373-442">The transaction is defined in a delegate.</span></span> <span data-ttu-id="af373-443">Akkor következik be, hogy átmeneti hiba, ha a végrehajtás stratégia újra a delegált lép működésbe.</span><span class="sxs-lookup"><span data-stu-id="af373-443">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

### <a name="more-information"></a><span data-ttu-id="af373-444">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-444">More information</span></span>
* [<span data-ttu-id="af373-445">Kapcsolat rugalmassága</span><span class="sxs-lookup"><span data-stu-id="af373-445">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="af373-446">Adatpont - EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="af373-446">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/en-us/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a><span data-ttu-id="af373-447">Az ADO.NET újrapróbálkozási szerint SQL-adatbázis</span><span class="sxs-lookup"><span data-stu-id="af373-447">SQL Database using ADO.NET retry guidelines</span></span>
<span data-ttu-id="af373-448">SQL-adatbázis egy olyan üzemeltetett SQL-adatbázis széles méretű és (megosztott) standard és premium (nem megosztott) szolgáltatás is elérhető.</span><span class="sxs-lookup"><span data-stu-id="af373-448">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-449">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-449">Retry mechanism</span></span>
<span data-ttu-id="af373-450">SQL-adatbázis nincs újrapróbálkozások ADO.NET használatával elérésekor beépített támogatása rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="af373-450">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="af373-451">Azonban a visszatérési kódok a kérelmek segítségével határozza meg, egy kérelem sikertelenségének okát.</span><span class="sxs-lookup"><span data-stu-id="af373-451">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="af373-452">SQL-adatbázis forgalomszabályozásáról további információkért lásd: [erőforrás-korlátozások az Azure SQL Database](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="af373-452">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="af373-453">A megfelelő hibakódok listájáért lásd: [SQL Database-ügyfélalkalmazások SQL hibakódok](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="af373-453">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="af373-454">A Polly könyvtár segítségével valósítja meg az újrapróbálkozások SQL-adatbázis.</span><span class="sxs-lookup"><span data-stu-id="af373-454">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="af373-455">Lásd: [Polly kezelését átmeneti hiba](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="af373-455">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="af373-456">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="af373-456">Retry usage guidance</span></span>
<span data-ttu-id="af373-457">Az ADO.NET használatával SQL-adatbázis elérésekor, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="af373-457">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="af373-458">Válassza ki a megfelelő szolgáltatásra beállítás (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="af373-458">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="af373-459">Egy megosztott példánnyal hosszabb, mint a szokásos csatlakozási késedelem és szabályozási más bérlők megosztott kiszolgáló kihasználtsága miatt romolhat.</span><span class="sxs-lookup"><span data-stu-id="af373-459">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="af373-460">Ha több kiszámítható teljesítményt és a megbízható kis késleltetésű műveletek szükség, vegye figyelembe, premium-lehetőség kiválasztásában.</span><span class="sxs-lookup"><span data-stu-id="af373-460">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="af373-461">Győződjön meg arról, hogy elvégezhető-e a megfelelő szint vagy a hatókör inkonzisztenciát mutat, amely az adatok az idempotent műveletek elkerülése érdekében az újrapróbálkozások.</span><span class="sxs-lookup"><span data-stu-id="af373-461">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="af373-462">Ideális esetben minden műveletet kell idempotent, így anélkül, hogy ez inkonzisztenciát megismételhető.</span><span class="sxs-lookup"><span data-stu-id="af373-462">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="af373-463">Ha nem ez a helyzet, az újra gombra vagy a hatókör, amely lehetővé teszi, hogy az összes kapcsolódó is vonható vissza, ha egy művelet nem sikerül; módosításai hajtható végre például a tranzakciós hatókörén belül.</span><span class="sxs-lookup"><span data-stu-id="af373-463">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="af373-464">További információkért lásd: [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="af373-464">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="af373-465">Egy rögzített időközönként stratégia nem ajánlott a az Azure SQL Database interaktív forgatókönyvek kivéve ahol csak néhány újrapróbálkozások nagyon rövid időközönként.</span><span class="sxs-lookup"><span data-stu-id="af373-465">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="af373-466">Ehelyett érdemes exponenciális vissza az indító stratégia az forgatókönyv a legtöbb.</span><span class="sxs-lookup"><span data-stu-id="af373-466">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="af373-467">Válassza ki a kapcsolat és a parancs időtúllépések megfelelő értéket, kapcsolatok meghatározásakor.</span><span class="sxs-lookup"><span data-stu-id="af373-467">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="af373-468">Túl rövid időtúllépés kapcsolatok korai hibák eredményezhet, amikor az adatbázis foglalt.</span><span class="sxs-lookup"><span data-stu-id="af373-468">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="af373-469">Túl hosszú időtúllépés előfordulhat, hogy megfelelően működik-e túl hosszú a sikertelen kapcsolódás észlelése előtti várakozási által újrapróbálkozási logika.</span><span class="sxs-lookup"><span data-stu-id="af373-469">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="af373-470">Az időtúllépés értéke a végpontok közötti késés; összetevője az újrapróbálkozási késleltetést az újrapróbálkozási házirendet minden újrapróbálkozási kísérlethez megadott hatékonyan kerül.</span><span class="sxs-lookup"><span data-stu-id="af373-470">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="af373-471">Zárja be a kapcsolat egy bizonyos újrapróbálkozások száma, még akkor is, ha vissza ki egy exponenciális használatával újrapróbálkozási logika, és próbálja megismételni a műveletet az új kapcsolat után.</span><span class="sxs-lookup"><span data-stu-id="af373-471">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="af373-472">Többször meg ugyanazt a kapcsolatot a azonos művelet lehet egy tényező való kapcsolódási problémák léptek fel.</span><span class="sxs-lookup"><span data-stu-id="af373-472">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="af373-473">Ez a módszer példáért lásd: [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-473">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="af373-474">Ha kapcsolatkészletezést használatban (alapértelmezett), amely ugyanazt a kapcsolatot választ ki a készletből bezárása és a kapcsolat újbóli megnyitása után is esély van.</span><span class="sxs-lookup"><span data-stu-id="af373-474">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="af373-475">Ha ez a helyzet, megoldási technika hívására-e a **ClearPool** metódusában a **SqlConnection** osztály megjelölni a kapcsolat nem használható fel újra.</span><span class="sxs-lookup"><span data-stu-id="af373-475">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="af373-476">Azonban akkor tegye ezt csak azt követően több kapcsolat sikertelenül, és csak akkor, ha az adott osztály hibás kapcsolatok kapcsolódó átmeneti hibák például az SQL-időtúllépések (hibakód: -2) észlelése.</span><span class="sxs-lookup"><span data-stu-id="af373-476">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="af373-477">Ha az adatok hozzáférési használ a tranzakciók által kezdeményezett **TransactionScope** példányok, az újrapróbálkozási logika nyissa meg újra a kapcsolat, és meg kell kezdeményezni egy új tranzakció hatókörében.</span><span class="sxs-lookup"><span data-stu-id="af373-477">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="af373-478">Emiatt a Újrapróbálkozást lehetővé tevő kódblokk kell foglalnia a tranzakció teljes körét.</span><span class="sxs-lookup"><span data-stu-id="af373-478">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="af373-479">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="af373-479">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="af373-480">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="af373-480">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="af373-481">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="af373-481">**Context**</span></span> | <span data-ttu-id="af373-482">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="af373-482">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="af373-483">**Ismételje meg a stratégia**</span><span class="sxs-lookup"><span data-stu-id="af373-483">**Retry strategy**</span></span> | <span data-ttu-id="af373-484">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="af373-484">**Settings**</span></span> | <span data-ttu-id="af373-485">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="af373-485">**Values**</span></span> | <span data-ttu-id="af373-486">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="af373-486">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="af373-487">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="af373-487">Interactive, UI,</span></span><br /><span data-ttu-id="af373-488">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="af373-488">or foreground</span></span> |<span data-ttu-id="af373-489">2 mp</span><span class="sxs-lookup"><span data-stu-id="af373-489">2 sec</span></span> |<span data-ttu-id="af373-490">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="af373-490">FixedInterval</span></span> |<span data-ttu-id="af373-491">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="af373-491">Retry count</span></span><br /><span data-ttu-id="af373-492">Újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="af373-492">Retry interval</span></span><br /><span data-ttu-id="af373-493">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="af373-493">First fast retry</span></span> |<span data-ttu-id="af373-494">3</span><span class="sxs-lookup"><span data-stu-id="af373-494">3</span></span><br /><span data-ttu-id="af373-495">500 ms</span><span class="sxs-lookup"><span data-stu-id="af373-495">500 ms</span></span><br /><span data-ttu-id="af373-496">igaz</span><span class="sxs-lookup"><span data-stu-id="af373-496">true</span></span> |<span data-ttu-id="af373-497">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-497">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="af373-498">Kísérlet történt a 2 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-498">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="af373-499">Próbálja meg a 3 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-499">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="af373-500">Háttér</span><span class="sxs-lookup"><span data-stu-id="af373-500">Background</span></span><br /><span data-ttu-id="af373-501">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="af373-501">or batch</span></span> |<span data-ttu-id="af373-502">30 sec</span><span class="sxs-lookup"><span data-stu-id="af373-502">30 sec</span></span> |<span data-ttu-id="af373-503">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-503">ExponentialBackoff</span></span> |<span data-ttu-id="af373-504">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="af373-504">Retry count</span></span><br /><span data-ttu-id="af373-505">Minimális biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="af373-505">Min back-off</span></span><br /><span data-ttu-id="af373-506">Maximális vissza kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="af373-506">Max back-off</span></span><br /><span data-ttu-id="af373-507">Különbözeti biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="af373-507">Delta back-off</span></span><br /><span data-ttu-id="af373-508">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="af373-508">First fast retry</span></span> |<span data-ttu-id="af373-509">5</span><span class="sxs-lookup"><span data-stu-id="af373-509">5</span></span><br /><span data-ttu-id="af373-510">0 (mp)</span><span class="sxs-lookup"><span data-stu-id="af373-510">0 sec</span></span><br /><span data-ttu-id="af373-511">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-511">60 sec</span></span><br /><span data-ttu-id="af373-512">2 mp</span><span class="sxs-lookup"><span data-stu-id="af373-512">2 sec</span></span><br /><span data-ttu-id="af373-513">hamis</span><span class="sxs-lookup"><span data-stu-id="af373-513">false</span></span> |<span data-ttu-id="af373-514">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-514">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="af373-515">Kísérlet történt a 2 - ~ 2 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-515">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="af373-516">Próbálja meg a 3 - ~ 6 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-516">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="af373-517">Kísérlet történt a 4 – ~ 14 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-517">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="af373-518">Kísérlet történt az 5 – kb. 30 másodperc késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-518">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="af373-519">A végpontok közötti késés célokat azt feltételezik, hogy a szolgáltatáshoz való csatlakozásokat az alapértelmezett időkorlátját.</span><span class="sxs-lookup"><span data-stu-id="af373-519">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="af373-520">Ha hosszabb ideig kapcsolat időtúllépése ad meg, a végpontok közötti késés minden újrapróbálkozási kísérlethez további időpontig bővíthető.</span><span class="sxs-lookup"><span data-stu-id="af373-520">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="af373-521">Példák</span><span class="sxs-lookup"><span data-stu-id="af373-521">Examples</span></span>
<span data-ttu-id="af373-522">Ez a szakasz azt mutatja, hogyan használják a Polly Azure SQL Database szolgáltatáshoz konfigurált újrapróbálkozási házirendjei eléréséhez a `Policy` osztály.</span><span class="sxs-lookup"><span data-stu-id="af373-522">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="af373-523">A következő kód egy kiterjesztésmetódus jeleníti meg a `SqlCommand` osztály, amely meghívja a `ExecuteAsync` az exponenciális leállítási.</span><span class="sxs-lookup"><span data-stu-id="af373-523">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}

```

<span data-ttu-id="af373-524">Az aszinkron kiterjesztésmetódus a következőképpen használható.</span><span class="sxs-lookup"><span data-stu-id="af373-524">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="af373-525">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-525">More information</span></span>
* [<span data-ttu-id="af373-526">Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése</span><span class="sxs-lookup"><span data-stu-id="af373-526">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="af373-527">A legtöbb lekérése SQL-adatbázis általános útmutatást lásd: [Azure SQL adatbázis-teljesítményt és a rugalmasság útmutató](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-527">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>


## <a name="service-bus-retry-guidelines"></a><span data-ttu-id="af373-528">A Service Bus újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-528">Service Bus retry guidelines</span></span>
<span data-ttu-id="af373-529">A Service Bus akkor felhő üzenetkezelési platform, amely nagyobb méretezés és rugalmasság lazán összekapcsolt üzenet exchange biztosít az alkalmazások összetevői a felhőben, vagy a helyszínen üzemeltetett is.</span><span class="sxs-lookup"><span data-stu-id="af373-529">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-530">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-530">Retry mechanism</span></span>
<span data-ttu-id="af373-531">A Service Bus használatával implementációja újrapróbálkozások valósítja meg a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály.</span><span class="sxs-lookup"><span data-stu-id="af373-531">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="af373-532">Összes Service Bus ügyfél teszi közzé a **RetryPolicy** tulajdonság, amely a implementációja egyikére állítható be a **RetryPolicy** alaposztály.</span><span class="sxs-lookup"><span data-stu-id="af373-532">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="af373-533">A beépített hitelesítés megvalósításához a következők:</span><span class="sxs-lookup"><span data-stu-id="af373-533">The built-in implementations are:</span></span>

* <span data-ttu-id="af373-534">A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-534">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="af373-535">Ez a biztonsági ki időköz, az újrapróbálkozások maximális számát, szabályozó tulajdonságok közzététele és a **TerminationTimeBuffer** tulajdonság, amely a teljes idő, a művelet elvégzéséhez korlátozására szolgál.</span><span class="sxs-lookup"><span data-stu-id="af373-535">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="af373-536">A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-536">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="af373-537">Ez használatos, amikor a Service Bus API-szintű ismételt próbálkozás esetén nincs szükség, például ha az újrapróbálkozások által felügyelt egy másik folyamat köteg vagy több lépés művelet részeként.</span><span class="sxs-lookup"><span data-stu-id="af373-537">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="af373-538">A Service Bus műveletek visszatérhet a kivételek, számos, a [függelék: üzenetküldési kivételek](http://msdn.microsoft.com/library/hh418082.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-538">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="af373-539">A listában a további információk arról, hogy mely Ha ezek határozzák meg, majd próbálja megismételni a műveletet nem megfelelő.</span><span class="sxs-lookup"><span data-stu-id="af373-539">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="af373-540">Például egy [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) azt jelzi, hogy az ügyfél kell Várjon egy ideig, majd próbálja megismételni a műveletet.</span><span class="sxs-lookup"><span data-stu-id="af373-540">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="af373-541">A előfordulása egy **ServerBusyException** is hatására a Service Bus egy másik mód, amelyben egy extra 10 másodperces késleltetési hozzáadódik a számított újrapróbálkozási késést.</span><span class="sxs-lookup"><span data-stu-id="af373-541">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="af373-542">Ebben a módban a rövid ideig alaphelyzetbe áll.</span><span class="sxs-lookup"><span data-stu-id="af373-542">This mode is reset after a short period.</span></span>

<span data-ttu-id="af373-543">A kivételek teszi közzé a Service Bus által visszaadott a **IsTransient** tulajdonságot, amely jelzi, ha az ügyfél érdemes újra megpróbálnia a művelet.</span><span class="sxs-lookup"><span data-stu-id="af373-543">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="af373-544">A beépített **RetryExponential** házirend támaszkodik a **IsTransient** tulajdonságot a **MessagingException** osztály, amely az összes Service Bus-kivételek alaposztálya.</span><span class="sxs-lookup"><span data-stu-id="af373-544">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="af373-545">Egyéni implementációja létrehozásakor a **RetryPolicy** használhatja a kivétel típusát kombinációja alaposztály és az **IsTransient** tulajdonság adja meg a részletesebb vezérlés újrapróbálkozási keresztül műveletek.</span><span class="sxs-lookup"><span data-stu-id="af373-545">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="af373-546">Például találta a **QuotaExceededException** és a szükséges műveletek a várólista kiürítése előtt üzenetet küld.</span><span class="sxs-lookup"><span data-stu-id="af373-546">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="af373-547">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="af373-547">Policy configuration</span></span>
<span data-ttu-id="af373-548">Újrapróbálkozási házirendek programozott módon van beállítva, és az alapértelmezett házirend állítható be egy **NamespaceManager** és egy **MessagingFactory**, vagy külön-külön az egyes üzenetküldési ügyfelek.</span><span class="sxs-lookup"><span data-stu-id="af373-548">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="af373-549">Az alapértelmezett újrapróbálkozási házirend beállítását egy üzenetkezelési munkamenet beállíthatja a **RetryPolicy** , a **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="af373-549">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

<span data-ttu-id="af373-550">Az üzenetkezelési gyár alapján létrehozott összes ügyfél alapértelmezett újrapróbálkozási házirend beállításához állítsa be a **RetryPolicy** , a **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="af373-550">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

<span data-ttu-id="af373-551">Az üzenetküldési ügyfél újrapróbálkozási házirend beállítását, vagy az alapértelmezett házirend felülbírálása, beállíthatja a **RetryPolicy** tulajdonságon keresztül a kívánt házirend osztály egy példányát:</span><span class="sxs-lookup"><span data-stu-id="af373-551">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="af373-552">Az újrapróbálkozási házirendje nem állítható be, az egyedi művelet szintjén.</span><span class="sxs-lookup"><span data-stu-id="af373-552">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="af373-553">Az üzenetküldési ügyfél minden műveletet vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="af373-553">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="af373-554">A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet.</span><span class="sxs-lookup"><span data-stu-id="af373-554">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="af373-555">Beállítás</span><span class="sxs-lookup"><span data-stu-id="af373-555">Setting</span></span> | <span data-ttu-id="af373-556">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="af373-556">Default value</span></span> | <span data-ttu-id="af373-557">Jelentése</span><span class="sxs-lookup"><span data-stu-id="af373-557">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="af373-558">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="af373-558">Policy</span></span> | <span data-ttu-id="af373-559">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-559">Exponential</span></span> | <span data-ttu-id="af373-560">Az exponenciális vissza-ki.</span><span class="sxs-lookup"><span data-stu-id="af373-560">Exponential back-off.</span></span> |
| <span data-ttu-id="af373-561">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-561">MinimalBackoff</span></span> | <span data-ttu-id="af373-562">0</span><span class="sxs-lookup"><span data-stu-id="af373-562">0</span></span> | <span data-ttu-id="af373-563">Minimális biztonsági ki időköz.</span><span class="sxs-lookup"><span data-stu-id="af373-563">Minimum back-off interval.</span></span> <span data-ttu-id="af373-564">Ez az újrapróbálkozási időköz deltaBackoff alapján kiszámított kerül.</span><span class="sxs-lookup"><span data-stu-id="af373-564">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="af373-565">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-565">MaximumBackoff</span></span> | <span data-ttu-id="af373-566">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-566">30 seconds</span></span> | <span data-ttu-id="af373-567">Maximális vissza az indító időköz.</span><span class="sxs-lookup"><span data-stu-id="af373-567">Maximum back-off interval.</span></span> <span data-ttu-id="af373-568">MaximumBackoff akkor használatos, ha az számított újrapróbálkozási időköz érték nagyobb, mint MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="af373-568">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="af373-569">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-569">DeltaBackoff</span></span> | <span data-ttu-id="af373-570">3 másodpercenként</span><span class="sxs-lookup"><span data-stu-id="af373-570">3 seconds</span></span> | <span data-ttu-id="af373-571">Vissza az indító időköz újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="af373-571">Back-off interval between retries.</span></span> <span data-ttu-id="af373-572">A timespan többszörösei későbbi újrapróbálkozás használható.</span><span class="sxs-lookup"><span data-stu-id="af373-572">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="af373-573">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="af373-573">TimeBuffer</span></span> | <span data-ttu-id="af373-574">5 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-574">5 seconds</span></span> | <span data-ttu-id="af373-575">A megszakítási idő puffer társított az újra gombra.</span><span class="sxs-lookup"><span data-stu-id="af373-575">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="af373-576">Ha a hátralévő idő TimeBuffer kisebb újrapróbálkozások elhagyásra kerül.</span><span class="sxs-lookup"><span data-stu-id="af373-576">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="af373-577">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="af373-577">MaxRetryCount</span></span> | <span data-ttu-id="af373-578">10</span><span class="sxs-lookup"><span data-stu-id="af373-578">10</span></span> | <span data-ttu-id="af373-579">Az újrapróbálkozások maximális számát.</span><span class="sxs-lookup"><span data-stu-id="af373-579">The maximum number of retries.</span></span> |
| <span data-ttu-id="af373-580">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="af373-580">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="af373-581">10 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-581">10 seconds</span></span> | <span data-ttu-id="af373-582">Ha az utolsó kivétel történt a **ServerBusyException**, ez az érték nem kerülnek be a számított újrapróbálkozási időközt.</span><span class="sxs-lookup"><span data-stu-id="af373-582">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="af373-583">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="af373-583">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="af373-584">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="af373-584">Retry usage guidance</span></span>
<span data-ttu-id="af373-585">Vegye figyelembe az alábbi iránymutatásokat, amikor a Service Bus használatával:</span><span class="sxs-lookup"><span data-stu-id="af373-585">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="af373-586">A beépített használatakor **RetryExponential** végrehajtására, tegye implementálja egy tartalék műveletet, mert a házirend reagál a kiszolgáló elfoglalt kivételeket, és automatikusan egy megfelelő újrapróbálkozási módra vált.</span><span class="sxs-lookup"><span data-stu-id="af373-586">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="af373-587">A Service Bus névterek megfeleltetni, amely a biztonsági mentési üzenetsorba automatikus feladatátvétel különálló névtérben levő elsődleges névtér várólista meghibásodásakor nevezett szolgáltatással támogatja.</span><span class="sxs-lookup"><span data-stu-id="af373-587">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="af373-588">A másodlagos várólistában lévő üzenetek küldhető vissza az elsődleges queue amikor azt állítja helyre.</span><span class="sxs-lookup"><span data-stu-id="af373-588">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="af373-589">Ez a szolgáltatás segítségével történő címet átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="af373-589">This feature helps to address transient failures.</span></span> <span data-ttu-id="af373-590">További információkért lásd: [aszinkron üzenetkezelési mintákat és magas rendelkezésre állású](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="af373-590">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="af373-591">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="af373-591">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="af373-592">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="af373-592">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="af373-593">Környezet</span><span class="sxs-lookup"><span data-stu-id="af373-593">Context</span></span> | <span data-ttu-id="af373-594">Példa maximális késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-594">Example maximum latency</span></span> | <span data-ttu-id="af373-595">Újrapróbálkozási házirend</span><span class="sxs-lookup"><span data-stu-id="af373-595">Retry policy</span></span> | <span data-ttu-id="af373-596">Beállítások</span><span class="sxs-lookup"><span data-stu-id="af373-596">Settings</span></span> | <span data-ttu-id="af373-597">Működés</span><span class="sxs-lookup"><span data-stu-id="af373-597">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="af373-598">Interaktív, a felhasználói felület vagy a előtér</span><span class="sxs-lookup"><span data-stu-id="af373-598">Interactive, UI, or foreground</span></span> | <span data-ttu-id="af373-599">2 másodperc \*</span><span class="sxs-lookup"><span data-stu-id="af373-599">2 seconds\*</span></span>  | <span data-ttu-id="af373-600">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-600">Exponential</span></span> | <span data-ttu-id="af373-601">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="af373-601">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="af373-602">MaximumBackoff = 30 másodperc.</span><span class="sxs-lookup"><span data-stu-id="af373-602">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="af373-603">DeltaBackoff = 300 msec.</span><span class="sxs-lookup"><span data-stu-id="af373-603">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="af373-604">TimeBuffer = 300 ms.</span><span class="sxs-lookup"><span data-stu-id="af373-604">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="af373-605">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="af373-605">MaxRetryCount = 2</span></span> | <span data-ttu-id="af373-606">1. kísérlet: Késleltetés 0 másodperc.</span><span class="sxs-lookup"><span data-stu-id="af373-606">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="af373-607">2. kísérlet: Késleltetés ~ 300 ms.</span><span class="sxs-lookup"><span data-stu-id="af373-607">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="af373-608">Attempt 3: Delay ~900 msec.</span><span class="sxs-lookup"><span data-stu-id="af373-608">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="af373-609">Háttér vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="af373-609">Background or batch</span></span> | <span data-ttu-id="af373-610">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-610">30 seconds</span></span> | <span data-ttu-id="af373-611">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="af373-611">Exponential</span></span> | <span data-ttu-id="af373-612">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="af373-612">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="af373-613">MaximumBackoff = 30 másodperc.</span><span class="sxs-lookup"><span data-stu-id="af373-613">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="af373-614">DeltaBackoff 1,75 mp =.</span><span class="sxs-lookup"><span data-stu-id="af373-614">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="af373-615">TimeBuffer = 5 másodperc.</span><span class="sxs-lookup"><span data-stu-id="af373-615">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="af373-616">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="af373-616">MaxRetryCount = 3</span></span> | <span data-ttu-id="af373-617">1. kísérlet: Késleltetés ~ 1 másodperc.</span><span class="sxs-lookup"><span data-stu-id="af373-617">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="af373-618">2. kísérlet: ~ 3 mp késleltetés.</span><span class="sxs-lookup"><span data-stu-id="af373-618">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="af373-619">3. kísérlet: Késleltetés ~ 6 MS.</span><span class="sxs-lookup"><span data-stu-id="af373-619">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="af373-620">Attempt 4: Delay ~13 msec.</span><span class="sxs-lookup"><span data-stu-id="af373-620">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="af373-621">\*További késleltetés, ha a kiszolgáló elfoglalt választ hozzáadott nem beleértve.</span><span class="sxs-lookup"><span data-stu-id="af373-621">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="af373-622">Telemetria</span><span class="sxs-lookup"><span data-stu-id="af373-622">Telemetry</span></span>
<span data-ttu-id="af373-623">A Service Bus újrapróbálkozások naplózza az ETW-eseményként is használ egy **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="af373-623">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="af373-624">Hozzá kell rendelni egy **eseményfigyelő Visszahívásán** az események rögzítése és megtekinthetők a teljesítmény-megjelenítőben, vagy azokat a megfelelő cél naplóba bejegyezni esemény forrását.</span><span class="sxs-lookup"><span data-stu-id="af373-624">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="af373-625">Használhatja a [szemantikai naplózás alkalmazás blokk](http://msdn.microsoft.com/library/dn775006.aspx) ehhez.</span><span class="sxs-lookup"><span data-stu-id="af373-625">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="af373-626">A kísérletek naplózása a következő formátumot követi a következők:</span><span class="sxs-lookup"><span data-stu-id="af373-626">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="af373-627">Példák</span><span class="sxs-lookup"><span data-stu-id="af373-627">Examples</span></span>
<span data-ttu-id="af373-628">Az alábbi példakód bemutatja, hogyan állítsa be a műveletek újrapróbálkozási házirendje esetében:</span><span class="sxs-lookup"><span data-stu-id="af373-628">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="af373-629">Egy névtér-kezelőt.</span><span class="sxs-lookup"><span data-stu-id="af373-629">A namespace manager.</span></span> <span data-ttu-id="af373-630">A házirend vonatkozik, hogy a kezelő az összes műveletet, és nem bírálható felül az egyes műveletek.</span><span class="sxs-lookup"><span data-stu-id="af373-630">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="af373-631">Egy üzenetkezelési gyárból.</span><span class="sxs-lookup"><span data-stu-id="af373-631">A messaging factory.</span></span> <span data-ttu-id="af373-632">A házirend alapján, hogy a gyári létrehozott összes ügyfelére érvényes, és nem bírálható felül az egyes ügyfelek létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="af373-632">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="af373-633">Az egyéni üzenetküldési ügyfél.</span><span class="sxs-lookup"><span data-stu-id="af373-633">An individual messaging client.</span></span> <span data-ttu-id="af373-634">Ügyfél létrehozása után az újrapróbálkozási házirendet beállíthatja, hogy az ügyfélszámítógépek számára.</span><span class="sxs-lookup"><span data-stu-id="af373-634">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="af373-635">A házirend vonatkozik, hogy az ügyfél összes művelete.</span><span class="sxs-lookup"><span data-stu-id="af373-635">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }


            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }


            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="af373-636">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-636">More information</span></span>
* [<span data-ttu-id="af373-637">Aszinkron üzenetkezelési mintát és magas rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="af373-637">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a><span data-ttu-id="af373-638">Azure Redis Cache újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-638">Azure Redis Cache retry guidelines</span></span>
<span data-ttu-id="af373-639">Azure Redis Cache egy gyors adatok elérése és alacsony késési igényű cache szolgáltatás a népszerű nyílt forráskódú Redis Cache alapján.</span><span class="sxs-lookup"><span data-stu-id="af373-639">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="af373-640">Biztonságos, Microsoft által felügyelt és az Azure-ban minden alkalmazás elérhető-e.</span><span class="sxs-lookup"><span data-stu-id="af373-640">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="af373-641">Ez a szakasz az útmutató alapján a StackExchange.Redis-ügyfelet használó gyorsítótár-hozzáféréshez.</span><span class="sxs-lookup"><span data-stu-id="af373-641">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="af373-642">Megfelelő ügyfelek listáját megtalálja az a [Redis webhely](http://redis.io/clients), és ezek különböző újrapróbálkozásos mechanizmusok.</span><span class="sxs-lookup"><span data-stu-id="af373-642">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="af373-643">Vegye figyelembe, hogy a StackExchange.Redis ügyfél használ-e az multiplexáló egyetlen kapcsolaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="af373-643">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="af373-644">Az ajánlott használati, hogy az ügyfél példányt létrehozni az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár minden műveletnél.</span><span class="sxs-lookup"><span data-stu-id="af373-644">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="af373-645">Emiatt a gyorsítótárba kapcsolat csak egyszer, és így az összes ebben a szakaszban található útmutatót az újrapróbálkozási házirendet a kezdeti kapcsolat kapcsolatos – és nem az egyes műveletek, aki hozzáfér a gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="af373-645">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-646">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-646">Retry mechanism</span></span>
<span data-ttu-id="af373-647">A StackExchange.Redis ügyfél használja egy kapcsolat manager osztály, amely olyan beállítási lehetőségekkel, incuding konfigurálva:</span><span class="sxs-lookup"><span data-stu-id="af373-647">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="af373-648">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="af373-648">**ConnectRetry**.</span></span> <span data-ttu-id="af373-649">A száma a gyorsítótárban nem sikerült kapcsolódni a rendszer megpróbálja újból.</span><span class="sxs-lookup"><span data-stu-id="af373-649">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="af373-650">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="af373-650">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="af373-651">Az újrapróbálkozási stratégiát kíván használni.</span><span class="sxs-lookup"><span data-stu-id="af373-651">The retry strategy to use.</span></span>
- <span data-ttu-id="af373-652">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="af373-652">**ConnectTimeout**.</span></span> <span data-ttu-id="af373-653">A maximális várakozási idő milliszekundumban.</span><span class="sxs-lookup"><span data-stu-id="af373-653">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="af373-654">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="af373-654">Policy configuration</span></span>
<span data-ttu-id="af373-655">Újrapróbálkozási házirendek programozott módon a beállításainak az ügyfél a gyorsítótár való csatlakozás előtt.</span><span class="sxs-lookup"><span data-stu-id="af373-655">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="af373-656">Hozzon létre egy példányát erre a **ConfigurationOptions** osztály, feltöltése a tulajdonságait, és úgy, hogy átadja a **Connect** metódust.</span><span class="sxs-lookup"><span data-stu-id="af373-656">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="af373-657">A beépített osztályok lineáris (állandó) késleltetés és exponenciális leállítási újrapróbálkozási véletlenszerű időközökkel támogatja.</span><span class="sxs-lookup"><span data-stu-id="af373-657">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="af373-658">Egyéni újrapróbálkozási házirendje implementálásával is létrehozhat a **IReconnectRetryPolicy** felületet.</span><span class="sxs-lookup"><span data-stu-id="af373-658">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="af373-659">Az alábbi példa használatával exponenciális leállítási újrapróbálkozási stratégiát konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="af373-659">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="af373-660">Másik lehetőségként adja meg a beállításokat egy karakterlánc, és adja át a a **Connect** metódust.</span><span class="sxs-lookup"><span data-stu-id="af373-660">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="af373-661">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezzel a módszerrel csak a kódot.</span><span class="sxs-lookup"><span data-stu-id="af373-661">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="af373-662">Közvetlenül a gyorsítótárba csatlakoztatásakor is megadható beállítások.</span><span class="sxs-lookup"><span data-stu-id="af373-662">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="af373-663">További információkért lásd: [verem Exchange Redis konfigurációja](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="af373-663">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="af373-664">A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet.</span><span class="sxs-lookup"><span data-stu-id="af373-664">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="af373-665">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="af373-665">**Context**</span></span> | <span data-ttu-id="af373-666">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="af373-666">**Setting**</span></span> | <span data-ttu-id="af373-667">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="af373-667">**Default value**</span></span><br /><span data-ttu-id="af373-668">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="af373-668">(v 1.2.2)</span></span> | <span data-ttu-id="af373-669">**Jelentése**</span><span class="sxs-lookup"><span data-stu-id="af373-669">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="af373-670">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="af373-670">ConfigurationOptions</span></span> |<span data-ttu-id="af373-671">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="af373-671">ConnectRetry</span></span><br /><br /><span data-ttu-id="af373-672">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="af373-672">ConnectTimeout</span></span><br /><br /><span data-ttu-id="af373-673">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="af373-673">SyncTimeout</span></span><br /><br /><span data-ttu-id="af373-674">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="af373-674">ReconnectRetryPolicy</span></span> |<span data-ttu-id="af373-675">3</span><span class="sxs-lookup"><span data-stu-id="af373-675">3</span></span><br /><br /><span data-ttu-id="af373-676">Legfeljebb 5000 ms plus SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="af373-676">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="af373-677">1000</span><span class="sxs-lookup"><span data-stu-id="af373-677">1000</span></span><br /><br /><span data-ttu-id="af373-678">5000 ms LinearRetry</span><span class="sxs-lookup"><span data-stu-id="af373-678">LinearRetry 5000 ms</span></span> |<span data-ttu-id="af373-679">Ismétlődő hányszor kísérletek csatlakozzon a kezdeti kapcsolat művelet során.</span><span class="sxs-lookup"><span data-stu-id="af373-679">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="af373-680">Időkorlát (ms) a műveletek csatlakozzon.</span><span class="sxs-lookup"><span data-stu-id="af373-680">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="af373-681">Nem a késleltetés újrapróbálkozási kísérletek között.</span><span class="sxs-lookup"><span data-stu-id="af373-681">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="af373-682">Idő (ms) szinkron műveletek engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="af373-682">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="af373-683">Ismételje meg minden 5000 ms.</span><span class="sxs-lookup"><span data-stu-id="af373-683">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="af373-684">A szinkron műveletek `SyncTimeout` adhat hozzá a végpontok közötti késés, de ha az értéket túl alacsonyra jelentős időtúllépéseket okozhat.</span><span class="sxs-lookup"><span data-stu-id="af373-684">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="af373-685">Lásd: [hibaelhárítása az Azure Redis Cache][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="af373-685">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="af373-686">Általánosságban elmondható kerülje a szinkron műveletek, és használja helyette az aszinkron műveletek.</span><span class="sxs-lookup"><span data-stu-id="af373-686">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="af373-687">További információ: [folyamatok és Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="af373-687">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="af373-688">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="af373-688">Retry usage guidance</span></span>
<span data-ttu-id="af373-689">Azure Redis Cache használata esetén, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="af373-689">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="af373-690">A StackExchange Redis ügyfél felügyeli a saját újrapróbálkozások, de csak ha a kapcsolat létrehozása a gyorsítótárba az alkalmazás első indításakor.</span><span class="sxs-lookup"><span data-stu-id="af373-690">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="af373-691">Konfigurálhatja a kapcsolat időkorlátja, újrapróbálkozási kísérleteinek száma és a kapcsolat létrehozása az újrapróbálkozások között eltelt idő, de az újrapróbálkozási házirendje nem vonatkozik a gyorsítótár műveleteket.</span><span class="sxs-lookup"><span data-stu-id="af373-691">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="af373-692">Számos újrapróbálkozás helyett fontolja meg, de visszatér az eredeti adatforrás helyette elérésével.</span><span class="sxs-lookup"><span data-stu-id="af373-692">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="af373-693">Telemetria</span><span class="sxs-lookup"><span data-stu-id="af373-693">Telemetry</span></span>
<span data-ttu-id="af373-694">Akkor is gyűjthet információt használatával kapcsolatok (de nem más műveletek) egy **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="af373-694">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="af373-695">Ezt követően a kimeneti példát alább láthatók.</span><span class="sxs-lookup"><span data-stu-id="af373-695">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="af373-696">Példák</span><span class="sxs-lookup"><span data-stu-id="af373-696">Examples</span></span>
<span data-ttu-id="af373-697">Az alábbi példakód újrapróbálkozások, amikor a StackExchange.Redis-ügyfél inicializálása állandó (lineáris) késleltetés konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="af373-697">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="af373-698">Ez a példa bemutatja, hogyan telepíthet a használt konfiguráció egy **ConfigurationOptions** példány.</span><span class="sxs-lookup"><span data-stu-id="af373-698">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries
                    
                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="af373-699">A következő példában a konfigurációs beállítások megadásával karakterláncként állítja be.</span><span class="sxs-lookup"><span data-stu-id="af373-699">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="af373-700">A kapcsolat időtúllépési érték a várakozási idő a kapcsolatot a gyorsítótárba, nem újrapróbálkozási kísérletek között eltelt maximális időtartam.</span><span class="sxs-lookup"><span data-stu-id="af373-700">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="af373-701">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság csak kód által állítható be.</span><span class="sxs-lookup"><span data-stu-id="af373-701">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="af373-702">További példákért lásd [konfigurációs](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) a projekt webhelyen.</span><span class="sxs-lookup"><span data-stu-id="af373-702">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="af373-703">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-703">More information</span></span>
* [<span data-ttu-id="af373-704">Webhely redis</span><span class="sxs-lookup"><span data-stu-id="af373-704">Redis website</span></span>](http://redis.io/)

## <a name="cosmos-db-retry-guidelines"></a><span data-ttu-id="af373-705">Cosmos DB újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-705">Cosmos DB retry guidelines</span></span>

<span data-ttu-id="af373-706">Cosmos DB egy olyan teljes körűen felügyelt több modellre adatbázis, amely támogatja a séma nélküli JSON-adatokat.</span><span class="sxs-lookup"><span data-stu-id="af373-706">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="af373-707">Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="af373-707">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-708">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-708">Retry mechanism</span></span>
<span data-ttu-id="af373-709">A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletek.</span><span class="sxs-lookup"><span data-stu-id="af373-709">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="af373-710">Az újrapróbálkozások száma és a maximális várakozási idő beállításához konfigurálása [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="af373-710">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="af373-711">Kivételek, amely kiváltja az ügyfél újrapróbálkozási házirend túl vagy nem átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="af373-711">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="af373-712">Ha Cosmos DB azelőtt gyorsítja fel, az ügyfél, HTTP 429 hibát ad vissza.</span><span class="sxs-lookup"><span data-stu-id="af373-712">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="af373-713">Az állapotkód: Ellenőrizze a `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="af373-713">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="af373-714">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="af373-714">Policy configuration</span></span>
<span data-ttu-id="af373-715">A következő táblázat alapértelmezett beállításait a `RetryOptions` osztály.</span><span class="sxs-lookup"><span data-stu-id="af373-715">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="af373-716">Beállítás</span><span class="sxs-lookup"><span data-stu-id="af373-716">Setting</span></span> | <span data-ttu-id="af373-717">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="af373-717">Default value</span></span> | <span data-ttu-id="af373-718">Leírás</span><span class="sxs-lookup"><span data-stu-id="af373-718">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="af373-719">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="af373-719">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="af373-720">9</span><span class="sxs-lookup"><span data-stu-id="af373-720">9</span></span> |<span data-ttu-id="af373-721">Ha a kérés nem teljesíthető, mivel a Cosmos DB sebessége korlátozza az ügyfél az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="af373-721">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="af373-722">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="af373-722">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="af373-723">30</span><span class="sxs-lookup"><span data-stu-id="af373-723">30</span></span> |<span data-ttu-id="af373-724">A maximális idő másodpercben. Próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="af373-724">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="af373-725">Példa</span><span class="sxs-lookup"><span data-stu-id="af373-725">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="af373-726">Telemetria</span><span class="sxs-lookup"><span data-stu-id="af373-726">Telemetry</span></span>
<span data-ttu-id="af373-727">Újrapróbálkozások bejelentkezett egy .NET keresztül strukturálatlan nyomkövetési üzenetekként **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="af373-727">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="af373-728">Konfigurálnia kell egy **TraceListener** az események rögzítése és a megfelelő cél a naplófájlba írja őket.</span><span class="sxs-lookup"><span data-stu-id="af373-728">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="af373-729">Például ha az App.config fájlban adja hozzá a következő, nyomkövetési adatokat a rendszer létrehoz és a végrehajtható fájl ugyanazon a helyen a fájlt:</span><span class="sxs-lookup"><span data-stu-id="af373-729">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a><span data-ttu-id="af373-730">Az Azure Search újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-730">Azure Search retry guidelines</span></span>
<span data-ttu-id="af373-731">Az Azure Search segítségével hatékony és kifinomult keresési-képességeket adhat a webhelyhez vagy alkalmazáshoz, gyorsan és egyszerűen finomhangolhatják a keresési eredmények között, és gazdag és finomhangolható rangsorolási modellek létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="af373-731">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-732">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-732">Retry mechanism</span></span>
<span data-ttu-id="af373-733">Az Azure Search SDK-ban való újrapróbálkozásra vezérli a `SetRetryPolicy` metódust a [SearchServiceClient] és [SearchIndexClient] osztályok.</span><span class="sxs-lookup"><span data-stu-id="af373-733">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="af373-734">Az alapértelmezett házirend ismétlések exponenciális leállítási, ha Azure Search egy 5xx vagy 408 (kérése időtúllépés) választ ad vissza.</span><span class="sxs-lookup"><span data-stu-id="af373-734">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="af373-735">Telemetria</span><span class="sxs-lookup"><span data-stu-id="af373-735">Telemetry</span></span>
<span data-ttu-id="af373-736">Nyomkövetési ETW vagy egy egyéni nyomkövetési szolgáltató regisztrálja.</span><span class="sxs-lookup"><span data-stu-id="af373-736">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="af373-737">További információkért lásd: a [AutoRest dokumentáció][autorest].</span><span class="sxs-lookup"><span data-stu-id="af373-737">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="azure-active-directory-retry-guidelines"></a><span data-ttu-id="af373-738">Az Azure Active Directory újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-738">Azure Active Directory retry guidelines</span></span>
<span data-ttu-id="af373-739">Azure Active Directory (Azure AD) egy olyan átfogó identitás- és hozzáférés felügyeleti felhőalapú megoldás, hogy a címtárszolgáltatások mag, speciális identitás irányítás, biztonsági és alkalmazáshozzáférés-kezeléshez.</span><span class="sxs-lookup"><span data-stu-id="af373-739">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="af373-740">Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi házirendeken és szabályokon alapuló hozzáférés-szabályozással bővíthessék az alkalmazásaikat.</span><span class="sxs-lookup"><span data-stu-id="af373-740">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-741">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-741">Retry mechanism</span></span>
<span data-ttu-id="af373-742">Az Azure Active Directory a az Active Directory Authentication Library (ADAL) beépített újrapróbálkozási mechanizmussal van.</span><span class="sxs-lookup"><span data-stu-id="af373-742">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="af373-743">Váratlan zárolásokat elkerülése érdekében azt javasoljuk, hogy külső gyártótól származó kódtárak és alkalmazáskód hajthatja végre **nem** próbálja meg újra a hibás kapcsolatok, de lehetővé teszi az ADAL újrapróbálkozások kezelésére.</span><span class="sxs-lookup"><span data-stu-id="af373-743">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="af373-744">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="af373-744">Retry usage guidance</span></span>
<span data-ttu-id="af373-745">Amikor az Azure Active Directoryval, vegye figyelembe az alábbi irányelveket:</span><span class="sxs-lookup"><span data-stu-id="af373-745">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="af373-746">Ha lehetséges, használja az ADAL-könyvtár és a beépített támogatása az ismételt próbálkozás.</span><span class="sxs-lookup"><span data-stu-id="af373-746">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="af373-747">Ha az Azure Active Directory a REST API-t használ, érdemes újra megpróbálnia a művelet csak akkor, ha a 5xx tartomány (például 500 belső kiszolgálóhiba 502 Hibás átjáró, 503-as szolgáltatás nem érhető el vagy 504-es számú átjáró időtúllépése) hiba eredménye.</span><span class="sxs-lookup"><span data-stu-id="af373-747">If you are using the REST API for Azure Active Directory, you should retry the operation only if the result is an error in the 5xx range (such as 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, and 504 Gateway Timeout).</span></span> <span data-ttu-id="af373-748">Nem próbálja meg újra más hibákat.</span><span class="sxs-lookup"><span data-stu-id="af373-748">Do not retry for any other errors.</span></span>
* <span data-ttu-id="af373-749">Az exponenciális vissza az indító házirend ajánlott kötegelt olyan esetekben, az Azure Active Directoryban.</span><span class="sxs-lookup"><span data-stu-id="af373-749">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="af373-750">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="af373-750">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="af373-751">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="af373-751">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="af373-752">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="af373-752">**Context**</span></span> | <span data-ttu-id="af373-753">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="af373-753">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="af373-754">**Ismételje meg a stratégia**</span><span class="sxs-lookup"><span data-stu-id="af373-754">**Retry strategy**</span></span> | <span data-ttu-id="af373-755">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="af373-755">**Settings**</span></span> | <span data-ttu-id="af373-756">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="af373-756">**Values**</span></span> | <span data-ttu-id="af373-757">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="af373-757">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="af373-758">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="af373-758">Interactive, UI,</span></span><br /><span data-ttu-id="af373-759">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="af373-759">or foreground</span></span> |<span data-ttu-id="af373-760">2 mp</span><span class="sxs-lookup"><span data-stu-id="af373-760">2 sec</span></span> |<span data-ttu-id="af373-761">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="af373-761">FixedInterval</span></span> |<span data-ttu-id="af373-762">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="af373-762">Retry count</span></span><br /><span data-ttu-id="af373-763">Újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="af373-763">Retry interval</span></span><br /><span data-ttu-id="af373-764">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="af373-764">First fast retry</span></span> |<span data-ttu-id="af373-765">3</span><span class="sxs-lookup"><span data-stu-id="af373-765">3</span></span><br /><span data-ttu-id="af373-766">500 ms</span><span class="sxs-lookup"><span data-stu-id="af373-766">500 ms</span></span><br /><span data-ttu-id="af373-767">igaz</span><span class="sxs-lookup"><span data-stu-id="af373-767">true</span></span> |<span data-ttu-id="af373-768">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-768">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="af373-769">Kísérlet történt a 2 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-769">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="af373-770">Próbálja meg a 3 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-770">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="af373-771">Háttér vagy</span><span class="sxs-lookup"><span data-stu-id="af373-771">Background or</span></span><br /><span data-ttu-id="af373-772">batch</span><span class="sxs-lookup"><span data-stu-id="af373-772">batch</span></span> |<span data-ttu-id="af373-773">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-773">60 sec</span></span> |<span data-ttu-id="af373-774">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="af373-774">ExponentialBackoff</span></span> |<span data-ttu-id="af373-775">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="af373-775">Retry count</span></span><br /><span data-ttu-id="af373-776">Minimális biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="af373-776">Min back-off</span></span><br /><span data-ttu-id="af373-777">Maximális vissza kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="af373-777">Max back-off</span></span><br /><span data-ttu-id="af373-778">Különbözeti biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="af373-778">Delta back-off</span></span><br /><span data-ttu-id="af373-779">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="af373-779">First fast retry</span></span> |<span data-ttu-id="af373-780">5</span><span class="sxs-lookup"><span data-stu-id="af373-780">5</span></span><br /><span data-ttu-id="af373-781">0 (mp)</span><span class="sxs-lookup"><span data-stu-id="af373-781">0 sec</span></span><br /><span data-ttu-id="af373-782">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="af373-782">60 sec</span></span><br /><span data-ttu-id="af373-783">2 mp</span><span class="sxs-lookup"><span data-stu-id="af373-783">2 sec</span></span><br /><span data-ttu-id="af373-784">hamis</span><span class="sxs-lookup"><span data-stu-id="af373-784">false</span></span> |<span data-ttu-id="af373-785">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-785">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="af373-786">Kísérlet történt a 2 - ~ 2 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-786">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="af373-787">Próbálja meg a 3 - ~ 6 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-787">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="af373-788">Kísérlet történt a 4 – ~ 14 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-788">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="af373-789">Kísérlet történt az 5 – kb. 30 másodperc késleltetés</span><span class="sxs-lookup"><span data-stu-id="af373-789">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="af373-790">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-790">More information</span></span>
* <span data-ttu-id="af373-791">[Az Azure Active Directory hitelesítési Kódtárai][adal]</span><span class="sxs-lookup"><span data-stu-id="af373-791">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="service-fabric-retry-guidelines"></a><span data-ttu-id="af373-792">A Service Fabric újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-792">Service Fabric retry guidelines</span></span>

<span data-ttu-id="af373-793">A Service Fabric megbízható szolgáltatások terjesztése fürt védi a cikkben szereplő lehetséges átmeneti hibák többségét ellen.</span><span class="sxs-lookup"><span data-stu-id="af373-793">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="af373-794">Átmeneti hibákat azonban továbbra is lehetséges, amelyek.</span><span class="sxs-lookup"><span data-stu-id="af373-794">Some transient faults are still possible, however.</span></span> <span data-ttu-id="af373-795">Például a naming service közepén útválasztási módosítása onnan kapta, hogy a kérelmet, ha okozza, hogy kivételt jelez.</span><span class="sxs-lookup"><span data-stu-id="af373-795">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="af373-796">Ha a kérésben 100 milliomod másodperc később, valószínűleg sikeres lesz.</span><span class="sxs-lookup"><span data-stu-id="af373-796">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="af373-797">A Service Fabric belsőleg, az ilyen átmeneti hiba kezeli.</span><span class="sxs-lookup"><span data-stu-id="af373-797">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="af373-798">Egyes beállítások segítségével konfigurálhatja a `OperationRetrySettings` osztály a szolgáltatások beállítása közben.</span><span class="sxs-lookup"><span data-stu-id="af373-798">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="af373-799">A következő kód egy példát mutat be.</span><span class="sxs-lookup"><span data-stu-id="af373-799">The following code shows an example.</span></span> <span data-ttu-id="af373-800">A legtöbb esetben ne legyen szükséges, és az alapértelmezett beállításokat sikeres lesz.</span><span class="sxs-lookup"><span data-stu-id="af373-800">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        OperationTimeout = TimeSpan.FromSeconds(30)
    };

    var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

    var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

    var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

    var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
        new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
        new ServicePartitionKey(0));
```

## <a name="more-information"></a><span data-ttu-id="af373-801">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-801">More information</span></span>

* [<span data-ttu-id="af373-802">Távoli kivételkezelést</span><span class="sxs-lookup"><span data-stu-id="af373-802">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a><span data-ttu-id="af373-803">Az Azure Event Hubs újra irányelvek</span><span class="sxs-lookup"><span data-stu-id="af373-803">Azure Event Hubs retry guidelines</span></span>

<span data-ttu-id="af373-804">Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.</span><span class="sxs-lookup"><span data-stu-id="af373-804">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="af373-805">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="af373-805">Retry mechanism</span></span>
<span data-ttu-id="af373-806">Az Azure Event Hubs ügyféloldali kódtár a újrapróbálkozásra vezérli a `RetryPolicy` tulajdonságát a `EventHubClient` osztály.</span><span class="sxs-lookup"><span data-stu-id="af373-806">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="af373-807">Az alapértelmezett exponenciális leállítási, amikor az Azure Event Hubs adja vissza egy átmeneti házirend ismétlések `EventHubsException` vagy egy `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="af373-807">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="af373-808">Példa</span><span class="sxs-lookup"><span data-stu-id="af373-808">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="af373-809">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-809">More information</span></span>
[<span data-ttu-id="af373-810">Az Azure Event Hubs szabványos .NET ügyféloldali kódtár</span><span class="sxs-lookup"><span data-stu-id="af373-810"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="af373-811">Általános REST, és próbálkozzon újra irányelveket</span><span class="sxs-lookup"><span data-stu-id="af373-811">General REST and retry guidelines</span></span>
<span data-ttu-id="af373-812">Azure vagy harmadik féltől származó szolgáltatások eléréséhez vegye figyelembe a következőket:</span><span class="sxs-lookup"><span data-stu-id="af373-812">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="af373-813">Egy rendszeres megközelítés, az újrapróbálkozásokat lehet, hogy kódú újrafelhasználható, akkor használja, így a egységes módszer alkalmazhatók minden ügyfél és az összes megoldás.</span><span class="sxs-lookup"><span data-stu-id="af373-813">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="af373-814">Fontolja meg az átmeneti hiba kezelési alkalmazás-blokk újrapróbálkozási keretrendszer használatát újrapróbálkozások kezelése, ha a célként megadott szolgáltatás vagy az ügyfél nem beépített újrapróbálkozási mechanizmussal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="af373-814">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="af373-815">Ez segítséget nyújt egy egységes újrapróbálkozási viselkedés, és azt a célként megadott szolgáltatás egy megfelelő alapértelmezett újrapróbálkozási stratégiát rendelkezhetnek.</span><span class="sxs-lookup"><span data-stu-id="af373-815">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="af373-816">Azonban szükség lehet, hogy létrehozzon egyéni újrapróbálkozási kódot azon szolgáltatások, amelyek nem szabványos viselkedését, ne használja a kivételeket úgy, hogy átmeneti hiba azt jelzi, vagy ha szeretné használni egy **újrapróbálkozási-válasz** válasz újrapróbálkozásra kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="af373-816">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="af373-817">Az átmeneti észlelési logika az ügyfél tényleges API meghívása a REST-hívások segítségével függ.</span><span class="sxs-lookup"><span data-stu-id="af373-817">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="af373-818">Egyes ügyfelek, például a újabb **HttpClient** osztály, nem egy nem sikeres HTTP-állapotkód: teljesített kérelmek kivételeinek kivételhibát.</span><span class="sxs-lookup"><span data-stu-id="af373-818">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="af373-819">Ez javítja a teljesítményt, de megakadályozza, hogy az átmeneti hiba kezelési alkalmazás-blokk használata.</span><span class="sxs-lookup"><span data-stu-id="af373-819">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="af373-820">Ebben az esetben burkolása sikerült kóddal, amely nem sikeres HTTP-állapotkódok, amely a blokk majd tudja feldolgozni a kivételeket a REST API-hívás.</span><span class="sxs-lookup"><span data-stu-id="af373-820">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="af373-821">Egy másik mechanizmus segítségével azt is megteheti, az újbóli próbálkozások meghajtó.</span><span class="sxs-lookup"><span data-stu-id="af373-821">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="af373-822">A szolgáltatás által visszaadott HTTP-állapotkód: jelzi, hogy a hiba átmeneti segítséget.</span><span class="sxs-lookup"><span data-stu-id="af373-822">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="af373-823">Előfordulhat, hogy egy ügyfél vagy a újrapróbálkozási keretrendszer állapotkód eléréséhez, vagy a megfelelő kivétel típusának meghatározására által létrehozott kivételek vizsgálata szükséges.</span><span class="sxs-lookup"><span data-stu-id="af373-823">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="af373-824">A következő HTTP-kódok általában azt jelzi, hogy egy újabb megfelelő:</span><span class="sxs-lookup"><span data-stu-id="af373-824">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="af373-825">408 kérelmi időkorlátot.</span><span class="sxs-lookup"><span data-stu-id="af373-825">408 Request Timeout</span></span>
  * <span data-ttu-id="af373-826">500 Internal Server Error</span><span class="sxs-lookup"><span data-stu-id="af373-826">500 Internal Server Error</span></span>
  * <span data-ttu-id="af373-827">502 Hibás átjáró</span><span class="sxs-lookup"><span data-stu-id="af373-827">502 Bad Gateway</span></span>
  * <span data-ttu-id="af373-828">503 Service Unavailable</span><span class="sxs-lookup"><span data-stu-id="af373-828">503 Service Unavailable</span></span>
  * <span data-ttu-id="af373-829">504-es számú átjáró időtúllépése</span><span class="sxs-lookup"><span data-stu-id="af373-829">504 Gateway Timeout</span></span>
* <span data-ttu-id="af373-830">Ha a kivételek az újrapróbálkozási logika, a következő általában jelzi, hogy egy átmeneti hiba, ha nem sikerült kapcsolatot:</span><span class="sxs-lookup"><span data-stu-id="af373-830">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="af373-831">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="af373-831">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="af373-832">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="af373-832">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="af373-833">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="af373-833">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="af373-834">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="af373-834">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="af373-835">A szolgáltatás nem érhető el állapota esetén a szolgáltatás a megfelelő késleltetés jelezhetik az újrapróbálkozás előtt a **újrapróbálkozási után** válaszfejléc vagy egy másik egyéni fejlécet.</span><span class="sxs-lookup"><span data-stu-id="af373-835">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="af373-836">Szolgáltatások további információkat is előfordulhat, hogy egyéni fejlécek küldeni, vagy a válasz tartalmának ágyazva.</span><span class="sxs-lookup"><span data-stu-id="af373-836">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="af373-837">Az átmeneti hiba kezelési alkalmazás-blokk nem használhatja a szabványos vagy egyéni "újrapróbálkozási-után" fejléc.</span><span class="sxs-lookup"><span data-stu-id="af373-837">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="af373-838">Nem próbálja meg újra az ügyfél-hibák (a 4xx tartomány hibák) jelölő állapotkódok 408 kérelem időtúllépés kivételével.</span><span class="sxs-lookup"><span data-stu-id="af373-838">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="af373-839">Alaposan tesztelje a újra azokat a stratégiákat és a feltételek, például a különböző hálózati állapotok és a különböző rendszer terhelések számos szerinti.</span><span class="sxs-lookup"><span data-stu-id="af373-839">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="af373-840">Ismételje meg a stratégiák</span><span class="sxs-lookup"><span data-stu-id="af373-840">Retry strategies</span></span>
<span data-ttu-id="af373-841">A tipikus objektumtípusokra újrapróbálkozási stratégia intervallumok a következők:</span><span class="sxs-lookup"><span data-stu-id="af373-841">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="af373-842">**Az exponenciális**: a megadott számú ki megközelítés véletlenszerű exponenciális biztonsági használatával az intervallum újrapróbálkozások között, az újrapróbálkozásokat végző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="af373-842">**Exponential**: A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="af373-843">Példa:</span><span class="sxs-lookup"><span data-stu-id="af373-843">For example:</span></span>

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* <span data-ttu-id="af373-844">**Növekményes**: a megadott számú újrapróbálkozások és a próbálkozások növekményes időközét újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="af373-844">**Incremental**: A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="af373-845">Példa:</span><span class="sxs-lookup"><span data-stu-id="af373-845">For example:</span></span>

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* <span data-ttu-id="af373-846">**LinearRetry**: használja a megadott időközönként újrapróbálkozások között, az újrapróbálkozásokat megadott számú végző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="af373-846">**LinearRetry**: A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="af373-847">Példa:</span><span class="sxs-lookup"><span data-stu-id="af373-847">For example:</span></span>

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a><span data-ttu-id="af373-848">További információ</span><span class="sxs-lookup"><span data-stu-id="af373-848">More information</span></span>
* [<span data-ttu-id="af373-849">Áramköri megszakító stratégiák</span><span class="sxs-lookup"><span data-stu-id="af373-849">Circuit breaker strategies</span></span>](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="af373-850">Átmeneti hiba Polly kezelését</span><span class="sxs-lookup"><span data-stu-id="af373-850">Transient fault handling with Polly</span></span>
<span data-ttu-id="af373-851">[Polly](http://www.thepollyproject.org) egy függvénytár programozottan kezelni az újrapróbálkozások és [áramköri megszakító] [ circuit-breaker] stratégiák.</span><span class="sxs-lookup"><span data-stu-id="af373-851">[Polly](http://www.thepollyproject.org) is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="af373-852">A Polly projekt tagja a [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="af373-852">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="af373-853">Ha az ügyfél nem natív módon támogatja az újrapróbálkozások szolgáltatások esetén Polly érvényes alternatíva, és kiküszöböli, egyéni újrapróbálkozási kód, amely implementálja megfelelően nehéz lehet írni.</span><span class="sxs-lookup"><span data-stu-id="af373-853">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="af373-854">Polly is lehetőséget biztosít nyomkövetési hibák azok bekövetkezésekor, így újrapróbálkozások bejelentkezhet.</span><span class="sxs-lookup"><span data-stu-id="af373-854">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
