---
title: "Szolgáltatásspecifikus útmutatás az újrapróbálkozási mechanizmushoz"
description: "Szolgáltatás beállítása az újrapróbálkozási mechanizmussal vonatkozó konkrét útmutatást."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 6aba60dc3a60e96e59e2034d4a1e380e0f1c996a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="f07fc-103">Ismételje meg az adott szolgáltatások útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-103">Retry guidance for specific services</span></span>

<span data-ttu-id="f07fc-104">Az Azure-szolgáltatások és az ügyfél SDK-k tartalmaznak egy újrapróbálkozási mechanizmus.</span><span class="sxs-lookup"><span data-stu-id="f07fc-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="f07fc-105">Azonban ezek eltérnek, mert minden egyes szolgáltatás eltérő jellemzőkkel és a követelmények, és így minden újrapróbálkozási mechanizmussal van beállítva, hogy egy adott szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="f07fc-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="f07fc-106">Ez az útmutató az Azure-szolgáltatások többsége a újrapróbálkozási mechanizmus szolgáltatásokat foglalja, és segítséget nyújtanak a használata, igazítja, vagy az újrapróbálkozási mechanizmussal, hogy a szolgáltatás kiterjesztése tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="f07fc-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="f07fc-107">Átmeneti kezel, és újra próbálkozik a kapcsolatokat és a szolgáltatások és erőforrások műveleteket általános útmutatást lásd: [útmutatást újra](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="f07fc-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="f07fc-108">A következő táblázat összefoglalja az ebben az útmutatóban leírt Azure-szolgáltatásokhoz tartozó újrapróbálkozási szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="f07fc-109">**Szolgáltatás**</span><span class="sxs-lookup"><span data-stu-id="f07fc-109">**Service**</span></span> | <span data-ttu-id="f07fc-110">**Ismételje meg a képességek**</span><span class="sxs-lookup"><span data-stu-id="f07fc-110">**Retry capabilities**</span></span> | <span data-ttu-id="f07fc-111">**Házirend-konfiguráció**</span><span class="sxs-lookup"><span data-stu-id="f07fc-111">**Policy configuration**</span></span> | <span data-ttu-id="f07fc-112">**Hatókör**</span><span class="sxs-lookup"><span data-stu-id="f07fc-112">**Scope**</span></span> | <span data-ttu-id="f07fc-113">**Telemetria szolgáltatások**</span><span class="sxs-lookup"><span data-stu-id="f07fc-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="f07fc-114">**[Az Azure Storage](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-115">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-115">Native in client</span></span> |<span data-ttu-id="f07fc-116">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-116">Programmatic</span></span> |<span data-ttu-id="f07fc-117">Ügyfél és az egyes műveletek</span><span class="sxs-lookup"><span data-stu-id="f07fc-117">Client and individual operations</span></span> |<span data-ttu-id="f07fc-118">TraceSource</span><span class="sxs-lookup"><span data-stu-id="f07fc-118">TraceSource</span></span> |
| <span data-ttu-id="f07fc-119">**[Az Entity Framework SQL-adatbázis](#sql-database-using-entity-framework-6-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-120">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-120">Native in client</span></span> |<span data-ttu-id="f07fc-121">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-121">Programmatic</span></span> |<span data-ttu-id="f07fc-122">Globális egy AppDomain tartományban</span><span class="sxs-lookup"><span data-stu-id="f07fc-122">Global per AppDomain</span></span> |<span data-ttu-id="f07fc-123">None</span><span class="sxs-lookup"><span data-stu-id="f07fc-123">None</span></span> |
| <span data-ttu-id="f07fc-124">**[Az Entity Framework Core SQL-adatbázis](#sql-database-using-entity-framework-core-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-125">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-125">Native in client</span></span> |<span data-ttu-id="f07fc-126">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-126">Programmatic</span></span> |<span data-ttu-id="f07fc-127">Globális egy AppDomain tartományban</span><span class="sxs-lookup"><span data-stu-id="f07fc-127">Global per AppDomain</span></span> |<span data-ttu-id="f07fc-128">None</span><span class="sxs-lookup"><span data-stu-id="f07fc-128">None</span></span> |
| <span data-ttu-id="f07fc-129">**[Az ADO.NET SQL-adatbázis](#sql-database-using-adonet-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span></span> |[<span data-ttu-id="f07fc-130">Polly</span><span class="sxs-lookup"><span data-stu-id="f07fc-130">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="f07fc-131">Programozott és deklaratív</span><span class="sxs-lookup"><span data-stu-id="f07fc-131">Declarative and programmatic</span></span> |<span data-ttu-id="f07fc-132">Egyetlen utasítások vagy kódblokkokat</span><span class="sxs-lookup"><span data-stu-id="f07fc-132">Single statements or blocks of code</span></span> |<span data-ttu-id="f07fc-133">Egyéni</span><span class="sxs-lookup"><span data-stu-id="f07fc-133">Custom</span></span> |
| <span data-ttu-id="f07fc-134">**[A Service Bus](#service-bus-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-134">**[Service Bus](#service-bus-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-135">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-135">Native in client</span></span> |<span data-ttu-id="f07fc-136">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-136">Programmatic</span></span> |<span data-ttu-id="f07fc-137">Namespace Manager, az üzenetkezelési gyárból és az ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-137">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="f07fc-138">ETW</span><span class="sxs-lookup"><span data-stu-id="f07fc-138">ETW</span></span> |
| <span data-ttu-id="f07fc-139">**[Azure Redis gyorsítótár](#azure-redis-cache-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-140">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-140">Native in client</span></span> |<span data-ttu-id="f07fc-141">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-141">Programmatic</span></span> |<span data-ttu-id="f07fc-142">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-142">Client</span></span> |<span data-ttu-id="f07fc-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="f07fc-143">TextWriter</span></span> |
| <span data-ttu-id="f07fc-144">**[A DocumentDB API](#documentdb-api-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-144">**[DocumentDB API](#documentdb-api-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-145">Natív szolgáltatásban</span><span class="sxs-lookup"><span data-stu-id="f07fc-145">Native in service</span></span> |<span data-ttu-id="f07fc-146">Nem konfigurálható</span><span class="sxs-lookup"><span data-stu-id="f07fc-146">Non-configurable</span></span> |<span data-ttu-id="f07fc-147">Globális</span><span class="sxs-lookup"><span data-stu-id="f07fc-147">Global</span></span> |<span data-ttu-id="f07fc-148">TraceSource</span><span class="sxs-lookup"><span data-stu-id="f07fc-148">TraceSource</span></span> |
| <span data-ttu-id="f07fc-149">**[Az Azure Search](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-149">**[Azure Search](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-150">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-150">Native in client</span></span> |<span data-ttu-id="f07fc-151">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-151">Programmatic</span></span> |<span data-ttu-id="f07fc-152">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-152">Client</span></span> |<span data-ttu-id="f07fc-153">ETW vagy egyéni</span><span class="sxs-lookup"><span data-stu-id="f07fc-153">ETW or Custom</span></span> |
| <span data-ttu-id="f07fc-154">**[Az Azure Active Directory](#azure-active-directory-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-155">Natív az ADAL-könyvtár</span><span class="sxs-lookup"><span data-stu-id="f07fc-155">Native in ADAL library</span></span> |<span data-ttu-id="f07fc-156">Az ADAL-könyvtár beágyazása</span><span class="sxs-lookup"><span data-stu-id="f07fc-156">Embeded into ADAL library</span></span> |<span data-ttu-id="f07fc-157">Belső</span><span class="sxs-lookup"><span data-stu-id="f07fc-157">Internal</span></span> |<span data-ttu-id="f07fc-158">None</span><span class="sxs-lookup"><span data-stu-id="f07fc-158">None</span></span> |
| <span data-ttu-id="f07fc-159">**[A Service Fabric](#service-fabric-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-160">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-160">Native in client</span></span> |<span data-ttu-id="f07fc-161">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-161">Programmatic</span></span> |<span data-ttu-id="f07fc-162">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-162">Client</span></span> |<span data-ttu-id="f07fc-163">None</span><span class="sxs-lookup"><span data-stu-id="f07fc-163">None</span></span> | 
| <span data-ttu-id="f07fc-164">**[Az Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="f07fc-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span></span> |<span data-ttu-id="f07fc-165">Natív ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-165">Native in client</span></span> |<span data-ttu-id="f07fc-166">Programozott</span><span class="sxs-lookup"><span data-stu-id="f07fc-166">Programmatic</span></span> |<span data-ttu-id="f07fc-167">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="f07fc-167">Client</span></span> |<span data-ttu-id="f07fc-168">None</span><span class="sxs-lookup"><span data-stu-id="f07fc-168">None</span></span> |

> [!NOTE]
> <span data-ttu-id="f07fc-169">Az Azure beépített része próbálja meg újból a mechanizmusok, jelenleg nem tudja alkalmazni a különböző típusú hiba különböző újrapróbálkozási házirendje vagy a kivétel a funkciók túl az újrapróbálkozási házirendet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception beyond the functionality include in the retry policy.</span></span> <span data-ttu-id="f07fc-170">Ezért ajánlott útmutatások érhetők el írásának időpontjában, az optimális átlagos teljesítményt és rendelkezésre állást biztosító házirendet konfigurálhat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-170">Therefore, the best guidance available at the time of writing is to configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="f07fc-171">Egy a házirend finomhangolását módja elemzése a naplófájlokat, és határozza meg az átmeneti előforduló hibák típusát.</span><span class="sxs-lookup"><span data-stu-id="f07fc-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> <span data-ttu-id="f07fc-172">Például ha hálózati problémák kapcsolódó hibák többségének, akkor előfordulhat, hogy egy azonnali újrapróbálkozási kísérlet ahelyett Várjon, amíg az első újrapróbálkozásnál hosszú ideig.</span><span class="sxs-lookup"><span data-stu-id="f07fc-172">For example, if the majority of errors are related to network connectivity issues, you might attempt an immediate retry rather than wait a long time for the first retry.</span></span>
>
>

## <a name="azure-storage-retry-guidelines"></a><span data-ttu-id="f07fc-173">Az Azure Storage újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-173">Azure Storage retry guidelines</span></span>
<span data-ttu-id="f07fc-174">Az Azure storage szolgáltatások közé tartoznak a tábla és a blob storage, fájlok és tárüzenetsort.</span><span class="sxs-lookup"><span data-stu-id="f07fc-174">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-175">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-175">Retry mechanism</span></span>
<span data-ttu-id="f07fc-176">Az újrapróbálkozások egyedi REST művelet szinten történik, és az ügyfél API-implementáció szerves részét képezik.</span><span class="sxs-lookup"><span data-stu-id="f07fc-176">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="f07fc-177">Az SDK az ügyfél tárolási osztályokat, hogy implementálja a [IExtendedRetryPolicy felület](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-177">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="f07fc-178">Nincsenek a felület másik implementációja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-178">There are different implementations of the interface.</span></span> <span data-ttu-id="f07fc-179">Tárolási ügyfelek kifejezetten táblák, blobok és várólisták házirendek közül választhatnak.</span><span class="sxs-lookup"><span data-stu-id="f07fc-179">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="f07fc-180">Minden egyes használ egy másik újrapróbálkozási stratégiát, amely lényegében meghatározza az újrapróbálkozási időköz és az egyéb részleteket.</span><span class="sxs-lookup"><span data-stu-id="f07fc-180">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="f07fc-181">A beépített osztályok lineáris (állandó késedelem) és a véletlenszerű újrapróbálkozási időközökkel exponenciális támogatásához.</span><span class="sxs-lookup"><span data-stu-id="f07fc-181">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="f07fc-182">Nincs is egy újrapróbálkozási házirend használható, ha egy másik folyamat kezeli a magasabb szintű újrapróbálkozások.</span><span class="sxs-lookup"><span data-stu-id="f07fc-182">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="f07fc-183">Azonban a saját újrapróbálkozási osztályokat is létrehozható, ha nincs megadva a beépített osztályok által meghatározott követelmények.</span><span class="sxs-lookup"><span data-stu-id="f07fc-183">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="f07fc-184">Alternatív újrapróbálkozások váltás elsődleges és másodlagos tárolótömbök szolgáltatáskeresésre, ha írásvédett georedundáns tárolás (RA-GRS) használ, és a kérés eredménye egy újrapróbálkozást lehetővé tevő hiba.</span><span class="sxs-lookup"><span data-stu-id="f07fc-184">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="f07fc-185">Lásd: [Azure Storage redundancia beállítások](http://msdn.microsoft.com/library/azure/dn727290.aspx) további információt.</span><span class="sxs-lookup"><span data-stu-id="f07fc-185">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="f07fc-186">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="f07fc-186">Policy configuration</span></span>
<span data-ttu-id="f07fc-187">Újrapróbálkozási házirendek programozott módon.</span><span class="sxs-lookup"><span data-stu-id="f07fc-187">Retry policies are configured programmatically.</span></span> <span data-ttu-id="f07fc-188">Egy tipikus eljárás akkor hozhat létre és tölthet egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, vagy **QueueRequestOptions**  példány.</span><span class="sxs-lookup"><span data-stu-id="f07fc-188">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="f07fc-189">A kérelem beállítások példány majd beállítható az ügyfélen, és az ügyfél összes művelet a megadott beállításokat fogja használni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-189">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="f07fc-190">Az ügyfél beállításokat lehet felülbírálni úgy, hogy a kérelem beállítások osztály feltöltött példány paraméterként művelet módszerek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-190">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="f07fc-191">Használhat egy **OperationContext** példány, adja meg a kódot a végrehajtási újbóli kísérlet esetén, és ha egy művelet befejeződött.</span><span class="sxs-lookup"><span data-stu-id="f07fc-191">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="f07fc-192">Ez a kód össze tudják gyűjteni a naplókat és telemetriai működésével kapcsolatos adatokat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-192">This code can collect information about the operation for use in logs and telemetry.</span></span>

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

<span data-ttu-id="f07fc-193">Mellett jelző hiba megfelelő-e az újra gombra, a kiterjesztett újrapróbálkozási házirendek vissza egy **RetryContext** objektum, amely jelzi, hogy a legutóbbi kérelem eredményeit, az újrapróbálkozások száma a legközelebbi újrapróbálkozás fordul elő az elsődleges vagy másodlagos helyre (lásd az alábbi táblázatban részletei).</span><span class="sxs-lookup"><span data-stu-id="f07fc-193">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="f07fc-194">A tulajdonságait a **RetryContext** objektum segítségével döntse el, ha, és ha egy újabb kísérletet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-194">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="f07fc-195">További részletekért lásd: [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-195">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="f07fc-196">Az alábbi táblázat a beépített alapértelmezett beállításainak ismételje meg a házirendeket.</span><span class="sxs-lookup"><span data-stu-id="f07fc-196">The following table shows the default settings for the built-in retry policies.</span></span>

| <span data-ttu-id="f07fc-197">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="f07fc-197">**Context**</span></span> | <span data-ttu-id="f07fc-198">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="f07fc-198">**Setting**</span></span> | <span data-ttu-id="f07fc-199">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="f07fc-199">**Default value**</span></span> | <span data-ttu-id="f07fc-200">**Jelentése**</span><span class="sxs-lookup"><span data-stu-id="f07fc-200">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="f07fc-201">Tábla / Blob / fájl</span><span class="sxs-lookup"><span data-stu-id="f07fc-201">Table / Blob / File</span></span><br /><span data-ttu-id="f07fc-202">QueueRequestOptions</span><span class="sxs-lookup"><span data-stu-id="f07fc-202">QueueRequestOptions</span></span> |<span data-ttu-id="f07fc-203">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="f07fc-203">MaximumExecutionTime</span></span><br /><br /><span data-ttu-id="f07fc-204">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="f07fc-204">ServerTimeout</span></span><br /><br /><br /><br /><br /><span data-ttu-id="f07fc-205">LocationMode</span><span class="sxs-lookup"><span data-stu-id="f07fc-205">LocationMode</span></span><br /><br /><br /><br /><br /><br /><br /><span data-ttu-id="f07fc-206">a retryPolicy</span><span class="sxs-lookup"><span data-stu-id="f07fc-206">RetryPolicy</span></span> |<span data-ttu-id="f07fc-207">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-207">120 seconds</span></span><br /><br /><span data-ttu-id="f07fc-208">None</span><span class="sxs-lookup"><span data-stu-id="f07fc-208">None</span></span><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><span data-ttu-id="f07fc-209">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="f07fc-209">ExponentialPolicy</span></span> |<span data-ttu-id="f07fc-210">A kérelem, beleértve az összes lehetséges újrapróbálkozási kísérletek maximális végrehajtási ideje.</span><span class="sxs-lookup"><span data-stu-id="f07fc-210">Maximum execution time for the request, including all potential retry attempts.</span></span><br /><span data-ttu-id="f07fc-211">A kérés a kiszolgáló időkorlátja (kerekített másodperc).</span><span class="sxs-lookup"><span data-stu-id="f07fc-211">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="f07fc-212">Ha nincs megadva, az alapértelmezett érték a kiszolgáló összes kérelem fog használni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-212">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="f07fc-213">Általában a legjobb lehetőség egy hagyja ki ezt a beállítást, hogy a kiszolgáló alapértelmezett szolgál.</span><span class="sxs-lookup"><span data-stu-id="f07fc-213">Usually, the best option is to omit this setting so that the server default is used.</span></span><br /><span data-ttu-id="f07fc-214">A tárfiók létrejön az írásvédett georedundáns tárolás (RA-GRS) replikációs beállítás, ha a hely mód segítségével jelzi, hogy melyik helyen kell kapnia a kérelmet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-214">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="f07fc-215">Például ha **PrimaryThenSecondary** van megadva, kérelmek mindig küldi el az elsődleges helyre először.</span><span class="sxs-lookup"><span data-stu-id="f07fc-215">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="f07fc-216">A kérés nem teljesíthető, ha a másodlagos helyre való továbbítás.</span><span class="sxs-lookup"><span data-stu-id="f07fc-216">If a request fails, it is sent to the secondary location.</span></span><br /><span data-ttu-id="f07fc-217">További információ alább olvasható az egyes lehetőségek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-217">See below for details of each option.</span></span> |
| <span data-ttu-id="f07fc-218">Az exponenciális házirend</span><span class="sxs-lookup"><span data-stu-id="f07fc-218">Exponential policy</span></span> |<span data-ttu-id="f07fc-219">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="f07fc-219">maxAttempt</span></span><br /><span data-ttu-id="f07fc-220">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-220">deltaBackoff</span></span><br /><br /><br /><span data-ttu-id="f07fc-221">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-221">MinBackoff</span></span><br /><br /><span data-ttu-id="f07fc-222">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-222">MaxBackoff</span></span> |<span data-ttu-id="f07fc-223">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-223">3</span></span><br /><span data-ttu-id="f07fc-224">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-224">4 seconds</span></span><br /><br /><br /><span data-ttu-id="f07fc-225">3 másodpercenként</span><span class="sxs-lookup"><span data-stu-id="f07fc-225">3 seconds</span></span><br /><br /><span data-ttu-id="f07fc-226">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-226">120 seconds</span></span> |<span data-ttu-id="f07fc-227">Újrapróbálkozások száma.</span><span class="sxs-lookup"><span data-stu-id="f07fc-227">Number of retry attempts.</span></span><br /><span data-ttu-id="f07fc-228">Vissza az indító időköz újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="f07fc-228">Back-off interval between retries.</span></span> <span data-ttu-id="f07fc-229">A TimeSpan érték, egy véletlenszerű elem, beleértve többszörösei későbbi újrapróbálkozás használható.</span><span class="sxs-lookup"><span data-stu-id="f07fc-229">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span><br /><span data-ttu-id="f07fc-230">Hozzáadandó deltaBackoff alapján kiszámított intervallumokban próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="f07fc-230">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="f07fc-231">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="f07fc-231">This value cannot be changed.</span></span><br /><span data-ttu-id="f07fc-232">MaxBackoff akkor használatos, ha az számított újrapróbálkozási időköz érték nagyobb, mint MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="f07fc-232">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="f07fc-233">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="f07fc-233">This value cannot be changed.</span></span> |
| <span data-ttu-id="f07fc-234">Lineáris házirend</span><span class="sxs-lookup"><span data-stu-id="f07fc-234">Linear policy</span></span> |<span data-ttu-id="f07fc-235">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="f07fc-235">maxAttempt</span></span><br /><span data-ttu-id="f07fc-236">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-236">deltaBackoff</span></span> |<span data-ttu-id="f07fc-237">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-237">3</span></span><br /><span data-ttu-id="f07fc-238">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-238">30 seconds</span></span> |<span data-ttu-id="f07fc-239">Újrapróbálkozások száma.</span><span class="sxs-lookup"><span data-stu-id="f07fc-239">Number of retry attempts.</span></span><br /><span data-ttu-id="f07fc-240">Vissza az indító időköz újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="f07fc-240">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="f07fc-241">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-241">Retry usage guidance</span></span>
<span data-ttu-id="f07fc-242">A tárolási ügyfél API-ja használatával az Azure storage szolgáltatások elérésekor, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="f07fc-242">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="f07fc-243">Házirendekkel a automatikusan újrapróbálkozik a Microsoft.WindowsAzure.Storage.RetryPolicies névtér hol a követelményeknek megfelelő.</span><span class="sxs-lookup"><span data-stu-id="f07fc-243">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="f07fc-244">A legtöbb esetben ezek a házirendek elegendő lesz.</span><span class="sxs-lookup"><span data-stu-id="f07fc-244">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="f07fc-245">Használja a **ExponentialRetry** kötegműveletek, háttérfeladatok vagy nem interaktív forgatókönyvek házirend.</span><span class="sxs-lookup"><span data-stu-id="f07fc-245">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="f07fc-246">Ezekben az esetekben általában lehetővé teszi a több időt a szolgáltatás helyreállítása – következésképpen megnövekedett arra, a művelet, végül sikeresen lezajlottak.</span><span class="sxs-lookup"><span data-stu-id="f07fc-246">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="f07fc-247">Érdemes megadni a **MaximumExecutionTime** tulajdonsága a **RequestOptions** korlátozza a teljes végrehajtási idő, de vegye figyelembe a típusát és méretét, a művelet kiválasztásakor paraméter egy időtúllépési értéket.</span><span class="sxs-lookup"><span data-stu-id="f07fc-247">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="f07fc-248">Ha egy egyéni újra megvalósításához szüksége, ne hozzon létre olyan burkolók a tárolási ügyfél osztályok körül.</span><span class="sxs-lookup"><span data-stu-id="f07fc-248">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="f07fc-249">Használja a képességek kiterjesztése a meglévő házirendek segítségével a **IExtendedRetryPolicy** felületet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-249">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="f07fc-250">Ha írásvédett georedundáns tárolás (RA-GRS) használ a **LocationMode** adhatja meg, hogy újból megkísérli érik el a másodlagos csak olvasható másolatát az áruházból kell az elsődleges hozzáférési sikertelen.</span><span class="sxs-lookup"><span data-stu-id="f07fc-250">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="f07fc-251">Azonban ez a beállítás használata esetén győződjön meg, hogy az alkalmazás képes sikeresen adatokkal dolgozni, amelyek esetleg elavult, ha az elsődleges áruházból replikációs még nem fejeződött be.</span><span class="sxs-lookup"><span data-stu-id="f07fc-251">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="f07fc-252">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="f07fc-252">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="f07fc-253">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="f07fc-253">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="f07fc-254">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="f07fc-254">**Context**</span></span> | <span data-ttu-id="f07fc-255">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-255">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="f07fc-256">**Ismételje meg a házirend**</span><span class="sxs-lookup"><span data-stu-id="f07fc-256">**Retry policy**</span></span> | <span data-ttu-id="f07fc-257">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="f07fc-257">**Settings**</span></span> | <span data-ttu-id="f07fc-258">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="f07fc-258">**Values**</span></span> | <span data-ttu-id="f07fc-259">**Működés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-259">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="f07fc-260">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="f07fc-260">Interactive, UI,</span></span><br /><span data-ttu-id="f07fc-261">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="f07fc-261">or foreground</span></span> |<span data-ttu-id="f07fc-262">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-262">2 seconds</span></span> |<span data-ttu-id="f07fc-263">lineáris</span><span class="sxs-lookup"><span data-stu-id="f07fc-263">Linear</span></span> |<span data-ttu-id="f07fc-264">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="f07fc-264">maxAttempt</span></span><br /><span data-ttu-id="f07fc-265">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-265">deltaBackoff</span></span> |<span data-ttu-id="f07fc-266">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-266">3</span></span><br /><span data-ttu-id="f07fc-267">500 ms</span><span class="sxs-lookup"><span data-stu-id="f07fc-267">500 ms</span></span> |<span data-ttu-id="f07fc-268">Kísérlet történt az 1 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-268">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="f07fc-269">Kísérlet történt a 2 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-269">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="f07fc-270">Próbálja meg a 3 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-270">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="f07fc-271">Háttér</span><span class="sxs-lookup"><span data-stu-id="f07fc-271">Background</span></span><br /><span data-ttu-id="f07fc-272">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="f07fc-272">or batch</span></span> |<span data-ttu-id="f07fc-273">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-273">30 seconds</span></span> |<span data-ttu-id="f07fc-274">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="f07fc-274">Exponential</span></span> |<span data-ttu-id="f07fc-275">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="f07fc-275">maxAttempt</span></span><br /><span data-ttu-id="f07fc-276">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-276">deltaBackoff</span></span> |<span data-ttu-id="f07fc-277">5</span><span class="sxs-lookup"><span data-stu-id="f07fc-277">5</span></span><br /><span data-ttu-id="f07fc-278">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-278">4 seconds</span></span> |<span data-ttu-id="f07fc-279">Próbálja meg az 1 – ~ 3 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-279">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="f07fc-280">Kísérlet történt a 2 – ~ 7 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-280">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="f07fc-281">Próbálja meg a 3 - késleltetés ~ 15 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-281">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="f07fc-282">Telemetria</span><span class="sxs-lookup"><span data-stu-id="f07fc-282">Telemetry</span></span>
<span data-ttu-id="f07fc-283">Újrapróbálkozás a rendszer naplózza a **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-283">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="f07fc-284">Konfigurálnia kell egy **TraceListener** az események rögzítése és a megfelelő cél a naplófájlba írja őket.</span><span class="sxs-lookup"><span data-stu-id="f07fc-284">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="f07fc-285">Használhatja a **értékének a TextWriterTraceListener figyelőre** vagy **XmlWriterTraceListener** írni egy naplófájlba, a **EventLogTraceListener** írni a Windows eseménynaplóba, vagy a **EventProviderTraceListener** nyomkövetési adatokat írni az ETW alrendszer.</span><span class="sxs-lookup"><span data-stu-id="f07fc-285">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="f07fc-286">Automatikus kiürítése a puffer, és az eseményeket, amelyek kerül a naplóba (például hiba, figyelmeztetés, tájékoztatás, és részletes) részletességi is konfigurálhatja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-286">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="f07fc-287">További információkért lásd: [ügyféloldali naplózás a .NET a Storage ügyféloldali kódtára](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-287">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="f07fc-288">Műveletek fogadhat egy **OperationContext** példány, mely tesz elérhetővé a **újrapróbálkozás** esemény használható egyéni telemetria logika csatlakoztatni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-288">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="f07fc-289">További információkért lásd: [OperationContext.Retrying esemény](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-289">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="f07fc-290">Példák</span><span class="sxs-lookup"><span data-stu-id="f07fc-290">Examples</span></span>
<span data-ttu-id="f07fc-291">Az alábbi példakód bemutatja, hogyan hozható létre kettő **TableRequestOptions** különböző újrapróbálkozásos beállításokkal példányok; interaktív kérelmek egyet, majd egy a háttér-kérelmekre.</span><span class="sxs-lookup"><span data-stu-id="f07fc-291">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="f07fc-292">A példa majd a készlet e két próbálja meg újra az ügyfél-házirendeket, hogy minden olyan kérelem esetében alkalmazni, és az interaktív stratégia is állít be egy adott kérelem, hogy felülírja az ügyfél alapértelmezett beállításait.</span><span class="sxs-lookup"><span data-stu-id="f07fc-292">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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
                // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
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

### <a name="more-information"></a><span data-ttu-id="f07fc-293">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-293">More information</span></span>
* [<span data-ttu-id="f07fc-294">Az Azure Storage ügyfél könyvtár újrapróbálkozási házirend javaslatok</span><span class="sxs-lookup"><span data-stu-id="f07fc-294">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="f07fc-295">A Storage ügyféloldali kódtára 2.0 – újrapróbálkozási házirendek megvalósítása</span><span class="sxs-lookup"><span data-stu-id="f07fc-295">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a><span data-ttu-id="f07fc-296">SQL-adatbázis Entity Framework 6 újrapróbálkozási szerint</span><span class="sxs-lookup"><span data-stu-id="f07fc-296">SQL Database using Entity Framework 6 retry guidelines</span></span>
<span data-ttu-id="f07fc-297">SQL-adatbázis egy olyan üzemeltetett SQL-adatbázis széles méretű és (megosztott) standard és premium (nem megosztott) szolgáltatás is elérhető.</span><span class="sxs-lookup"><span data-stu-id="f07fc-297">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="f07fc-298">Entitás-keretrendszer egy objektum relációs leképező, amely lehetővé teszi a .NET-fejlesztők relációs adatok tartományspecifikus-objektumok segítségével.</span><span class="sxs-lookup"><span data-stu-id="f07fc-298">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="f07fc-299">Szükségtelenné teszi, a legtöbb a fejlesztők általában kell írnia az adat-hozzáférési kód.</span><span class="sxs-lookup"><span data-stu-id="f07fc-299">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-300">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-300">Retry mechanism</span></span>
<span data-ttu-id="f07fc-301">Újrapróbálkozási támogatás áll rendelkezésre, használja az Entity Framework 6.0 SQL-adatbázis elérésekor és magasabb mechanizmuson keresztül nevű [kapcsolati rugalmasságot / újra logika](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-301">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="f07fc-302">Az újrapróbálkozási mechanizmus a fő jellemzői a következők:</span><span class="sxs-lookup"><span data-stu-id="f07fc-302">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="f07fc-303">Az elsődleges absztrakciós van a **IDbExecutionStrategy** felületet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-303">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="f07fc-304">Ez az interfész:</span><span class="sxs-lookup"><span data-stu-id="f07fc-304">This interface:</span></span>
  * <span data-ttu-id="f07fc-305">Meghatározza a szinkron és aszinkron **Execute*** módszerek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-305">Defines synchronous and asynchronous **Execute*** methods.</span></span>
  * <span data-ttu-id="f07fc-306">Meghatározza az osztályokat is használható közvetlenül vagy egy adatbázis-környezet, mint egy alapértelmezett stratégia konfigurált, leképezve a szolgáltató neve, vagy leképezve a szolgáltató nevét és a kiszolgáló nevét.</span><span class="sxs-lookup"><span data-stu-id="f07fc-306">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="f07fc-307">Ha a környezet konfigurálva, próbálkozások egyéni adatbázis-művelet, amelynek lehet több szinten egy adott környezetben a művelethez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-307">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="f07fc-308">Határozza meg, majd ismételje meg a sikertelen kapcsolódás, mikor és hogyan.</span><span class="sxs-lookup"><span data-stu-id="f07fc-308">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="f07fc-309">Ez magában foglalja a számos beépített implementációja a **IDbExecutionStrategy** felület:</span><span class="sxs-lookup"><span data-stu-id="f07fc-309">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="f07fc-310">Alapértelmezett – nincs újrapróbálkozás.</span><span class="sxs-lookup"><span data-stu-id="f07fc-310">Default - no retrying.</span></span>
  * <span data-ttu-id="f07fc-311">Alapértelmezett SQL-adatbázis (automatikus) – nincs újrapróbálkozás, de kivételek megvizsgálja, és azokat a javaslatát, hogy az SQL-adatbázis stratégia burkolja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-311">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="f07fc-312">SQL-adatbázis észlelési logika és az alapértelmezett SQL-adatbázis - exponenciális (alaposztály öröklődik).</span><span class="sxs-lookup"><span data-stu-id="f07fc-312">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="f07fc-313">Az exponenciális vissza ki stratégiát véletlenszerűsítésének valósítja meg.</span><span class="sxs-lookup"><span data-stu-id="f07fc-313">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="f07fc-314">A beépített újrapróbálkozási osztályok állapot-nyilvántartó és nem többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="f07fc-314">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="f07fc-315">Azonban ismét felhasználhatók a jelenlegi művelet befejeződése után.</span><span class="sxs-lookup"><span data-stu-id="f07fc-315">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="f07fc-316">Ha a megadott újrapróbálkozások maximális számát, az eredményeket egy új kivétel van csomagolni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-316">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="f07fc-317">Nem a jelenlegi kivétel mentése buborék.</span><span class="sxs-lookup"><span data-stu-id="f07fc-317">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="f07fc-318">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="f07fc-318">Policy configuration</span></span>
<span data-ttu-id="f07fc-319">Újrapróbálkozási nyújtott támogatás a használó Entity Framework 6.0 vagy újabb SQL-adatbázis elérésekor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-319">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="f07fc-320">Újrapróbálkozási házirendek programozott módon.</span><span class="sxs-lookup"><span data-stu-id="f07fc-320">Retry policies are configured programmatically.</span></span> <span data-ttu-id="f07fc-321">A konfiguráció nem módosítható művelet alapon.</span><span class="sxs-lookup"><span data-stu-id="f07fc-321">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="f07fc-322">A környezet alapértelmezett stratégia konfigurálásakor meg kell adnia egy függvényt, amely létrehoz egy új stratégia igény szerint.</span><span class="sxs-lookup"><span data-stu-id="f07fc-322">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="f07fc-323">A következő kód bemutatja, hogyan hozhat létre egy újrapróbálkozási configuration osztály, amely kiterjeszti a **DbConfiguration** alaposztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-323">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="f07fc-324">Ezután megadhatja ezt az alapértelmezett újrapróbálkozási stratégia segítségével minden műveletnél a **SetConfiguration** metódusában a **DbConfiguration** példány az alkalmazás indításakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-324">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="f07fc-325">Alapértelmezés szerint EF automatikusan felderíti és a configuration osztály használata.</span><span class="sxs-lookup"><span data-stu-id="f07fc-325">By default, EF will automatically discover and use the configuration class.</span></span>

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

<span data-ttu-id="f07fc-326">Az újrapróbálkozási configuration osztály kontextus megadhatja a környezeti osztályt ellátása megjegyzésekkel egy **DbConfigurationType** attribútum.</span><span class="sxs-lookup"><span data-stu-id="f07fc-326">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="f07fc-327">Azonban ha csak egy konfigurációs osztály, EF használ, nincs szükség a környezet megjegyzésekkel.</span><span class="sxs-lookup"><span data-stu-id="f07fc-327">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

<span data-ttu-id="f07fc-328">Ha különböző újrapróbálkozásos stratégiák műveleteket, vagy tiltsa le a konkrét műveletek az újrapróbálkozások van szüksége, a configuration osztály, amely lehetővé teszi, hogy felfüggeszti vagy stratégiák felcserélése jelölő beállításával hozhat létre a **CallContext** .</span><span class="sxs-lookup"><span data-stu-id="f07fc-328">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="f07fc-329">A configuration osztály stratégiák váltáshoz használja ezt a jelzőt, vagy tiltsa le a stratégiát, adja meg, és egy alapértelmezett stratégia használatára.</span><span class="sxs-lookup"><span data-stu-id="f07fc-329">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="f07fc-330">További információkért lásd: [felfüggesztése végrehajtási stratégia](http://msdn.microsoft.com/dn307226#transactions_workarounds) korlátozásai a végrehajtási stratégiák újrapróbálkozás (EF6 és újabb verziók esetében) lapján.</span><span class="sxs-lookup"><span data-stu-id="f07fc-330">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="f07fc-331">A megadott újrapróbálkozási stratégiák használata egyéni műveletek egy másik módszer akkor létrehozni a szükséges stratégia osztály egy példányát, és adja meg a kívánt beállításokat paramétereknek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-331">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="f07fc-332">Majd meghívása a **ExecuteAsync** metódust.</span><span class="sxs-lookup"><span data-stu-id="f07fc-332">You then invoke its **ExecuteAsync** method.</span></span>

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

<span data-ttu-id="f07fc-333">A legegyszerűbben úgy használja egy **DbConfiguration** ugyanabban a szerelvényben, keresse meg az osztály a **DbContext** osztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-333">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="f07fc-334">Ez azonban nem megfelelő ugyanabban a környezetben van szükség esetén különböző helyzetekben, például a különböző interaktív és a háttér újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="f07fc-334">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="f07fc-335">A különböző környezetek külön alkalmazástartományok hajtható végre, ha a beépített támogatást használja osztályok a konfigurációs fájlban, vagy állítsa be explicit módon a kód használatával.</span><span class="sxs-lookup"><span data-stu-id="f07fc-335">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="f07fc-336">A különböző környezetekben az AppDomain végre kell hajtani, ha egy egyéni megoldás lesz szükség.</span><span class="sxs-lookup"><span data-stu-id="f07fc-336">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="f07fc-337">További információkért lásd: [kód-alapú konfigurációban (EF6 és újabb verziók esetében)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-337">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="f07fc-338">A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet EF6 használatakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-338">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

![Ismételje meg a útmutatást tábla](./images/retry-service-specific/RetryServiceSpecificGuidanceTable4.png)

### <a name="retry-usage-guidance"></a><span data-ttu-id="f07fc-340">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-340">Retry usage guidance</span></span>
<span data-ttu-id="f07fc-341">EF6 használatával SQL-adatbázis elérésekor, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="f07fc-341">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="f07fc-342">Válassza ki a megfelelő szolgáltatásra beállítás (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="f07fc-342">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="f07fc-343">Egy megosztott példánnyal hosszabb, mint a szokásos csatlakozási késedelem és szabályozási más bérlők megosztott kiszolgáló kihasználtsága miatt romolhat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-343">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="f07fc-344">Ha kiszámítható teljesítményt és a megbízható kis késleltetésű műveletek szükség, vegye figyelembe, premium-lehetőség kiválasztásában.</span><span class="sxs-lookup"><span data-stu-id="f07fc-344">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="f07fc-345">A rögzített időközönként stratégia használható az Azure SQL Database nem ajánlott.</span><span class="sxs-lookup"><span data-stu-id="f07fc-345">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="f07fc-346">Ehelyett használja a exponenciális vissza az indító stratégia, mert előfordulhat, hogy túlterheltté vált a szolgáltatást, és hosszabb engedélyezése ahhoz, hogy a helyreállítás több időt.</span><span class="sxs-lookup"><span data-stu-id="f07fc-346">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="f07fc-347">Válassza ki a kapcsolat és a parancs időtúllépések megfelelő értéket, kapcsolatok meghatározásakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-347">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="f07fc-348">Az időtúllépés kiinduló, az üzleti logika tervezésével és a teszteléshez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-348">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="f07fc-349">Ez az érték módosításához idővel a mennyiségű adatot szeretne, vagy módosítsa az üzleti folyamatokat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-349">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="f07fc-350">Túl rövid időtúllépés kapcsolatok korai hibák eredményezhet, amikor az adatbázis foglalt.</span><span class="sxs-lookup"><span data-stu-id="f07fc-350">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="f07fc-351">Túl hosszú időtúllépés előfordulhat, hogy megfelelően működik-e túl hosszú a sikertelen kapcsolódás észlelése előtti várakozási által újrapróbálkozási logika.</span><span class="sxs-lookup"><span data-stu-id="f07fc-351">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="f07fc-352">Az időtúllépés értéke a végpontok közötti késés összetevője bár nem lehet egyszerűen megállapítani hány parancsok hajtja végre, a környezet mentése során.</span><span class="sxs-lookup"><span data-stu-id="f07fc-352">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="f07fc-353">Az alapértelmezett időtúllépési módosíthatja úgy, hogy a **CommandTimeout** tulajdonsága a **DbContext** példány.</span><span class="sxs-lookup"><span data-stu-id="f07fc-353">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="f07fc-354">Entitás-keretrendszer támogatja a konfigurációs fájlok meghatározott konfigurációk próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="f07fc-354">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="f07fc-355">Azonban a maximális rugalmasság Azure-érdemes létrehozni programozott módon az alkalmazáson belül a konfigurációs.</span><span class="sxs-lookup"><span data-stu-id="f07fc-355">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="f07fc-356">Az adott paraméterek újrapróbálkozási-házirendek, például az újrapróbálkozások és az újrapróbálkozási intervallumok száma a szolgáltatás konfigurációs fájljában tárolt, és futási időben a megfelelő házirendek létrehozására szolgál.</span><span class="sxs-lookup"><span data-stu-id="f07fc-356">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="f07fc-357">Ez lehetővé teszi, hogy a beállításokat, anélkül, hogy újra kell indítani az alkalmazást módosítani.</span><span class="sxs-lookup"><span data-stu-id="f07fc-357">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="f07fc-358">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="f07fc-358">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="f07fc-359">A késleltetés, újrapróbálkozások (le, és létrehozott egy exponenciális sorozatot) között nem adható meg.</span><span class="sxs-lookup"><span data-stu-id="f07fc-359">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="f07fc-360">Csak a maximális értékeket adhat meg, ahogy Itt; Ha létrehoz egy egyéni újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="f07fc-360">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="f07fc-361">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="f07fc-361">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="f07fc-362">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="f07fc-362">**Context**</span></span> | <span data-ttu-id="f07fc-363">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-363">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="f07fc-364">**Ismételje meg a házirend**</span><span class="sxs-lookup"><span data-stu-id="f07fc-364">**Retry policy**</span></span> | <span data-ttu-id="f07fc-365">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="f07fc-365">**Settings**</span></span> | <span data-ttu-id="f07fc-366">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="f07fc-366">**Values**</span></span> | <span data-ttu-id="f07fc-367">**Működés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-367">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="f07fc-368">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="f07fc-368">Interactive, UI,</span></span><br /><span data-ttu-id="f07fc-369">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="f07fc-369">or foreground</span></span> |<span data-ttu-id="f07fc-370">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-370">2 seconds</span></span> |<span data-ttu-id="f07fc-371">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="f07fc-371">Exponential</span></span> |<span data-ttu-id="f07fc-372">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="f07fc-372">MaxRetryCount</span></span><br /><span data-ttu-id="f07fc-373">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="f07fc-373">MaxDelay</span></span> |<span data-ttu-id="f07fc-374">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-374">3</span></span><br /><span data-ttu-id="f07fc-375">750 ms</span><span class="sxs-lookup"><span data-stu-id="f07fc-375">750 ms</span></span> |<span data-ttu-id="f07fc-376">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-376">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="f07fc-377">Kísérlet történt a 2 – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-377">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="f07fc-378">Próbálja meg a 3 – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-378">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="f07fc-379">Háttér</span><span class="sxs-lookup"><span data-stu-id="f07fc-379">Background</span></span><br /> <span data-ttu-id="f07fc-380">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="f07fc-380">or batch</span></span> |<span data-ttu-id="f07fc-381">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-381">30 seconds</span></span> |<span data-ttu-id="f07fc-382">Az exponenciális</span><span class="sxs-lookup"><span data-stu-id="f07fc-382">Exponential</span></span> |<span data-ttu-id="f07fc-383">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="f07fc-383">MaxRetryCount</span></span><br /><span data-ttu-id="f07fc-384">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="f07fc-384">MaxDelay</span></span> |<span data-ttu-id="f07fc-385">5</span><span class="sxs-lookup"><span data-stu-id="f07fc-385">5</span></span><br /><span data-ttu-id="f07fc-386">12 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-386">12 seconds</span></span> |<span data-ttu-id="f07fc-387">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-387">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="f07fc-388">Kísérlet történt a 2 - ~ 1 másodperc késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-388">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="f07fc-389">Próbálja meg a 3 - ~ 3 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-389">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="f07fc-390">Kísérlet történt a 4 – ~ 7 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-390">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="f07fc-391">Kísérlet történt az 5 – 12 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-391">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="f07fc-392">A végpontok közötti késés célokat azt feltételezik, hogy a szolgáltatáshoz való csatlakozásokat az alapértelmezett időkorlátját.</span><span class="sxs-lookup"><span data-stu-id="f07fc-392">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="f07fc-393">Ha hosszabb ideig kapcsolat időtúllépése ad meg, a végpontok közötti késés minden újrapróbálkozási kísérlethez további időpontig bővíthető.</span><span class="sxs-lookup"><span data-stu-id="f07fc-393">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="f07fc-394">Példák</span><span class="sxs-lookup"><span data-stu-id="f07fc-394">Examples</span></span>
<span data-ttu-id="f07fc-395">Az alábbi példakód egy egyszerű adat-hozzáférési megoldás, amely használja az Entity Framework határozza meg.</span><span class="sxs-lookup"><span data-stu-id="f07fc-395">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="f07fc-396">Egy adott újrapróbálkozási stratégiát állítja a nevű osztály egy példányát meghatározásával **BlogConfiguration** , amely kiterjeszti **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-396">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="f07fc-397">Az Entity Framework újrapróbálkozási mechanizmussal további példái megtalálhatók [kapcsolati rugalmasságot / újra logika](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-397">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="f07fc-398">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-398">More information</span></span>
* [<span data-ttu-id="f07fc-399">Az Azure SQL adatbázis-teljesítményt és a rugalmasság útmutatója</span><span class="sxs-lookup"><span data-stu-id="f07fc-399">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a><span data-ttu-id="f07fc-400">SQL-adatbázis Entity Framework Core újrapróbálkozási szerint</span><span class="sxs-lookup"><span data-stu-id="f07fc-400">SQL Database using Entity Framework Core retry guidelines</span></span>
<span data-ttu-id="f07fc-401">[Entity Framework Core](/ef/core/) van egy objektum relációs leképező, amely lehetővé teszi a .NET Core fejlesztők objektumok, tartomány-specifikus adatait.</span><span class="sxs-lookup"><span data-stu-id="f07fc-401">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="f07fc-402">Szükségtelenné teszi, a legtöbb a fejlesztők általában kell írnia az adat-hozzáférési kód.</span><span class="sxs-lookup"><span data-stu-id="f07fc-402">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="f07fc-403">Az Entity Framework jelen verziójában alapoktól készült, és nem automatikusan örökli a szolgáltatások EF6.x.</span><span class="sxs-lookup"><span data-stu-id="f07fc-403">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-404">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-404">Retry mechanism</span></span>
<span data-ttu-id="f07fc-405">Újrapróbálkozási támogatást biztosított használatával Entity Framework Core mechanizmus segítségével SQL-adatbázis nevű [kapcsolati rugalmasságot](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="f07fc-405">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="f07fc-406">Kapcsolat rugalmassága az EF Core 1.1.0-ás lett bevezetve.</span><span class="sxs-lookup"><span data-stu-id="f07fc-406">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="f07fc-407">Az elsődleges absztrakciós van a `IExecutionStrategy` felületet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-407">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="f07fc-408">A végrehajtási stratégiát az SQL Server, beleértve az SQL Azure, vegye figyelembe a kivételt típusú követően újra megkísérelhető és ésszerű alapértelmezett értéket az újrapróbálkozások maximális számát, vonatkozó felszólítások közötti idő újrapróbálkozások, és így tovább.</span><span class="sxs-lookup"><span data-stu-id="f07fc-408">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="f07fc-409">Példák</span><span class="sxs-lookup"><span data-stu-id="f07fc-409">Examples</span></span>

<span data-ttu-id="f07fc-410">Az alábbi kód lehetővé teszi, hogy automatikus újrapróbálkozások a DbContext objektum, amely képviseli egy munkamenetet az adatbázis konfigurálásakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-410">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="f07fc-411">A következő kód bemutatja, hogyan hajtható végre, az automatikus újrapróbálkozásokat tranzakciót egy végrehajtási stratégia használatával.</span><span class="sxs-lookup"><span data-stu-id="f07fc-411">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="f07fc-412">A tranzakció egy delegált van meghatározva.</span><span class="sxs-lookup"><span data-stu-id="f07fc-412">The transaction is defined in a delegate.</span></span> <span data-ttu-id="f07fc-413">Akkor következik be, hogy átmeneti hiba, ha a végrehajtás stratégia újra a delegált lép működésbe.</span><span class="sxs-lookup"><span data-stu-id="f07fc-413">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="f07fc-414">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-414">More information</span></span>
* [<span data-ttu-id="f07fc-415">Kapcsolat rugalmassága</span><span class="sxs-lookup"><span data-stu-id="f07fc-415">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="f07fc-416">Adatpont - EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="f07fc-416">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/en-us/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a><span data-ttu-id="f07fc-417">Az ADO.NET újrapróbálkozási szerint SQL-adatbázis</span><span class="sxs-lookup"><span data-stu-id="f07fc-417">SQL Database using ADO.NET retry guidelines</span></span>
<span data-ttu-id="f07fc-418">SQL-adatbázis egy olyan üzemeltetett SQL-adatbázis széles méretű és (megosztott) standard és premium (nem megosztott) szolgáltatás is elérhető.</span><span class="sxs-lookup"><span data-stu-id="f07fc-418">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-419">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-419">Retry mechanism</span></span>
<span data-ttu-id="f07fc-420">SQL-adatbázis nincs újrapróbálkozások ADO.NET használatával elérésekor beépített támogatása rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="f07fc-420">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="f07fc-421">Azonban a visszatérési kódok a kérelmek segítségével határozza meg, egy kérelem sikertelenségének okát.</span><span class="sxs-lookup"><span data-stu-id="f07fc-421">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="f07fc-422">SQL-adatbázis forgalomszabályozásáról további információkért lásd: [erőforrás-korlátozások az Azure SQL Database](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="f07fc-422">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="f07fc-423">A megfelelő hibakódok listájáért lásd: [SQL Database-ügyfélalkalmazások SQL hibakódok](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="f07fc-423">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="f07fc-424">A Polly könyvtár segítségével valósítja meg az újrapróbálkozások SQL-adatbázis.</span><span class="sxs-lookup"><span data-stu-id="f07fc-424">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="f07fc-425">Lásd: [Polly kezelését átmeneti hiba](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="f07fc-425">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="f07fc-426">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-426">Retry usage guidance</span></span>
<span data-ttu-id="f07fc-427">Az ADO.NET használatával SQL-adatbázis elérésekor, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="f07fc-427">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="f07fc-428">Válassza ki a megfelelő szolgáltatásra beállítás (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="f07fc-428">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="f07fc-429">Egy megosztott példánnyal hosszabb, mint a szokásos csatlakozási késedelem és szabályozási más bérlők megosztott kiszolgáló kihasználtsága miatt romolhat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-429">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="f07fc-430">Ha több kiszámítható teljesítményt és a megbízható kis késleltetésű műveletek szükség, vegye figyelembe, premium-lehetőség kiválasztásában.</span><span class="sxs-lookup"><span data-stu-id="f07fc-430">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="f07fc-431">Győződjön meg arról, hogy elvégezhető-e a megfelelő szint vagy a hatókör inkonzisztenciát mutat, amely az adatok az idempotent műveletek elkerülése érdekében az újrapróbálkozások.</span><span class="sxs-lookup"><span data-stu-id="f07fc-431">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="f07fc-432">Ideális esetben minden műveletet kell idempotent, így anélkül, hogy ez inkonzisztenciát megismételhető.</span><span class="sxs-lookup"><span data-stu-id="f07fc-432">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="f07fc-433">Ha nem ez a helyzet, az újra gombra vagy a hatókör, amely lehetővé teszi, hogy az összes kapcsolódó is vonható vissza, ha egy művelet nem sikerül; módosításai hajtható végre például a tranzakciós hatókörén belül.</span><span class="sxs-lookup"><span data-stu-id="f07fc-433">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="f07fc-434">További információkért lásd: [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="f07fc-434">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="f07fc-435">Egy rögzített időközönként stratégia nem ajánlott a az Azure SQL Database interaktív forgatókönyvek kivéve ahol csak néhány újrapróbálkozások nagyon rövid időközönként.</span><span class="sxs-lookup"><span data-stu-id="f07fc-435">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="f07fc-436">Ehelyett érdemes exponenciális vissza az indító stratégia az forgatókönyv a legtöbb.</span><span class="sxs-lookup"><span data-stu-id="f07fc-436">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="f07fc-437">Válassza ki a kapcsolat és a parancs időtúllépések megfelelő értéket, kapcsolatok meghatározásakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-437">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="f07fc-438">Túl rövid időtúllépés kapcsolatok korai hibák eredményezhet, amikor az adatbázis foglalt.</span><span class="sxs-lookup"><span data-stu-id="f07fc-438">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="f07fc-439">Túl hosszú időtúllépés előfordulhat, hogy megfelelően működik-e túl hosszú a sikertelen kapcsolódás észlelése előtti várakozási által újrapróbálkozási logika.</span><span class="sxs-lookup"><span data-stu-id="f07fc-439">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="f07fc-440">Az időtúllépés értéke a végpontok közötti késés; összetevője az újrapróbálkozási késleltetést az újrapróbálkozási házirendet minden újrapróbálkozási kísérlethez megadott hatékonyan kerül.</span><span class="sxs-lookup"><span data-stu-id="f07fc-440">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="f07fc-441">Zárja be a kapcsolat egy bizonyos újrapróbálkozások száma, még akkor is, ha vissza ki egy exponenciális használatával újrapróbálkozási logika, és próbálja megismételni a műveletet az új kapcsolat után.</span><span class="sxs-lookup"><span data-stu-id="f07fc-441">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="f07fc-442">Többször meg ugyanazt a kapcsolatot a azonos művelet lehet egy tényező való kapcsolódási problémák léptek fel.</span><span class="sxs-lookup"><span data-stu-id="f07fc-442">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="f07fc-443">Ez a módszer példáért lásd: [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-443">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="f07fc-444">Ha kapcsolatkészletezést használatban (alapértelmezett), amely ugyanazt a kapcsolatot választ ki a készletből bezárása és a kapcsolat újbóli megnyitása után is esély van.</span><span class="sxs-lookup"><span data-stu-id="f07fc-444">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="f07fc-445">Ha ez a helyzet, megoldási technika hívására-e a **ClearPool** metódusában a **SqlConnection** osztály megjelölni a kapcsolat nem használható fel újra.</span><span class="sxs-lookup"><span data-stu-id="f07fc-445">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="f07fc-446">Azonban akkor tegye ezt csak azt követően több kapcsolat sikertelenül, és csak akkor, ha az adott osztály hibás kapcsolatok kapcsolódó átmeneti hibák például az SQL-időtúllépések (hibakód: -2) észlelése.</span><span class="sxs-lookup"><span data-stu-id="f07fc-446">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="f07fc-447">Ha az adatok hozzáférési használ a tranzakciók által kezdeményezett **TransactionScope** példányok, az újrapróbálkozási logika nyissa meg újra a kapcsolat, és meg kell kezdeményezni egy új tranzakció hatókörében.</span><span class="sxs-lookup"><span data-stu-id="f07fc-447">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="f07fc-448">Emiatt a Újrapróbálkozást lehetővé tevő kódblokk kell foglalnia a tranzakció teljes körét.</span><span class="sxs-lookup"><span data-stu-id="f07fc-448">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="f07fc-449">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="f07fc-449">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="f07fc-450">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="f07fc-450">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="f07fc-451">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="f07fc-451">**Context**</span></span> | <span data-ttu-id="f07fc-452">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-452">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="f07fc-453">**Ismételje meg a stratégia**</span><span class="sxs-lookup"><span data-stu-id="f07fc-453">**Retry strategy**</span></span> | <span data-ttu-id="f07fc-454">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="f07fc-454">**Settings**</span></span> | <span data-ttu-id="f07fc-455">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="f07fc-455">**Values**</span></span> | <span data-ttu-id="f07fc-456">**Működés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-456">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="f07fc-457">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="f07fc-457">Interactive, UI,</span></span><br /><span data-ttu-id="f07fc-458">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="f07fc-458">or foreground</span></span> |<span data-ttu-id="f07fc-459">2 mp</span><span class="sxs-lookup"><span data-stu-id="f07fc-459">2 sec</span></span> |<span data-ttu-id="f07fc-460">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="f07fc-460">FixedInterval</span></span> |<span data-ttu-id="f07fc-461">Újbóli próbálkozások száma</span><span class="sxs-lookup"><span data-stu-id="f07fc-461">Retry count</span></span><br /><span data-ttu-id="f07fc-462">Újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="f07fc-462">Retry interval</span></span><br /><span data-ttu-id="f07fc-463">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="f07fc-463">First fast retry</span></span> |<span data-ttu-id="f07fc-464">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-464">3</span></span><br /><span data-ttu-id="f07fc-465">500 ms</span><span class="sxs-lookup"><span data-stu-id="f07fc-465">500 ms</span></span><br /><span data-ttu-id="f07fc-466">igaz</span><span class="sxs-lookup"><span data-stu-id="f07fc-466">true</span></span> |<span data-ttu-id="f07fc-467">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-467">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="f07fc-468">Kísérlet történt a 2 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-468">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="f07fc-469">Próbálja meg a 3 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-469">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="f07fc-470">Háttér</span><span class="sxs-lookup"><span data-stu-id="f07fc-470">Background</span></span><br /><span data-ttu-id="f07fc-471">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="f07fc-471">or batch</span></span> |<span data-ttu-id="f07fc-472">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-472">30 sec</span></span> |<span data-ttu-id="f07fc-473">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-473">ExponentialBackoff</span></span> |<span data-ttu-id="f07fc-474">Újbóli próbálkozások száma</span><span class="sxs-lookup"><span data-stu-id="f07fc-474">Retry count</span></span><br /><span data-ttu-id="f07fc-475">Minimális biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="f07fc-475">Min back-off</span></span><br /><span data-ttu-id="f07fc-476">Maximális vissza kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="f07fc-476">Max back-off</span></span><br /><span data-ttu-id="f07fc-477">Különbözeti biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="f07fc-477">Delta back-off</span></span><br /><span data-ttu-id="f07fc-478">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="f07fc-478">First fast retry</span></span> |<span data-ttu-id="f07fc-479">5</span><span class="sxs-lookup"><span data-stu-id="f07fc-479">5</span></span><br /><span data-ttu-id="f07fc-480">0 (mp)</span><span class="sxs-lookup"><span data-stu-id="f07fc-480">0 sec</span></span><br /><span data-ttu-id="f07fc-481">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-481">60 sec</span></span><br /><span data-ttu-id="f07fc-482">2 mp</span><span class="sxs-lookup"><span data-stu-id="f07fc-482">2 sec</span></span><br /><span data-ttu-id="f07fc-483">hamis</span><span class="sxs-lookup"><span data-stu-id="f07fc-483">false</span></span> |<span data-ttu-id="f07fc-484">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-484">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="f07fc-485">Kísérlet történt a 2 - ~ 2 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-485">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="f07fc-486">Próbálja meg a 3 - ~ 6 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-486">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="f07fc-487">Kísérlet történt a 4 – ~ 14 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-487">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="f07fc-488">Kísérlet történt az 5 – kb. 30 másodperc késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-488">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="f07fc-489">A végpontok közötti késés célokat azt feltételezik, hogy a szolgáltatáshoz való csatlakozásokat az alapértelmezett időkorlátját.</span><span class="sxs-lookup"><span data-stu-id="f07fc-489">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="f07fc-490">Ha hosszabb ideig kapcsolat időtúllépése ad meg, a végpontok közötti késés minden újrapróbálkozási kísérlethez további időpontig bővíthető.</span><span class="sxs-lookup"><span data-stu-id="f07fc-490">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="f07fc-491">Példák</span><span class="sxs-lookup"><span data-stu-id="f07fc-491">Examples</span></span>
<span data-ttu-id="f07fc-492">Ez a szakasz azt mutatja, hogyan használják a Polly Azure SQL Database szolgáltatáshoz konfigurált újrapróbálkozási házirendjei eléréséhez a `Policy` osztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-492">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="f07fc-493">A következő kód egy kiterjesztésmetódus jeleníti meg a `SqlCommand` osztály, amely meghívja a `ExecuteAsync` az exponenciális leállítási.</span><span class="sxs-lookup"><span data-stu-id="f07fc-493">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="f07fc-494">Az aszinkron kiterjesztésmetódus a következőképpen használható.</span><span class="sxs-lookup"><span data-stu-id="f07fc-494">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="f07fc-495">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-495">More information</span></span>
* [<span data-ttu-id="f07fc-496">Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése</span><span class="sxs-lookup"><span data-stu-id="f07fc-496">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="f07fc-497">A legtöbb lekérése SQL-adatbázis általános útmutatást lásd: [Azure SQL adatbázis-teljesítményt és a rugalmasság útmutató](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-497">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>


## <a name="service-bus-retry-guidelines"></a><span data-ttu-id="f07fc-498">A Service Bus újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-498">Service Bus retry guidelines</span></span>
<span data-ttu-id="f07fc-499">A Service Bus akkor felhő üzenetkezelési platform, amely nagyobb méretezés és rugalmasság lazán összekapcsolt üzenet exchange biztosít az alkalmazások összetevői a felhőben, vagy a helyszínen üzemeltetett is.</span><span class="sxs-lookup"><span data-stu-id="f07fc-499">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-500">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-500">Retry mechanism</span></span>
<span data-ttu-id="f07fc-501">A Service Bus használatával implementációja újrapróbálkozások valósítja meg a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-501">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="f07fc-502">Összes Service Bus ügyfél teszi közzé a **RetryPolicy** tulajdonság, amely a implementációja egyikére állítható be a **RetryPolicy** alaposztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-502">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="f07fc-503">A beépített hitelesítés megvalósításához a következők:</span><span class="sxs-lookup"><span data-stu-id="f07fc-503">The built-in implementations are:</span></span>

* <span data-ttu-id="f07fc-504">A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-504">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="f07fc-505">Ez a biztonsági ki időköz, az újrapróbálkozások maximális számát, szabályozó tulajdonságok közzététele és a **TerminationTimeBuffer** tulajdonság, amely a teljes idő, a művelet elvégzéséhez korlátozására szolgál.</span><span class="sxs-lookup"><span data-stu-id="f07fc-505">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="f07fc-506">A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-506">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="f07fc-507">Ez használatos, amikor a Service Bus API-szintű ismételt próbálkozás esetén nincs szükség, például ha az újrapróbálkozások által felügyelt egy másik folyamat köteg vagy több lépés művelet részeként.</span><span class="sxs-lookup"><span data-stu-id="f07fc-507">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="f07fc-508">A Service Bus műveletek visszatérhet a kivételek, számos, a [függelék: üzenetküldési kivételek](http://msdn.microsoft.com/library/hh418082.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-508">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="f07fc-509">A listában a további információk arról, hogy mely Ha ezek határozzák meg, majd próbálja megismételni a műveletet nem megfelelő.</span><span class="sxs-lookup"><span data-stu-id="f07fc-509">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="f07fc-510">Például egy [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) azt jelzi, hogy az ügyfél kell Várjon egy ideig, majd próbálja megismételni a műveletet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-510">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="f07fc-511">A előfordulása egy **ServerBusyException** is hatására a Service Bus egy másik mód, amelyben egy extra 10 másodperces késleltetési hozzáadódik a számított újrapróbálkozási késést.</span><span class="sxs-lookup"><span data-stu-id="f07fc-511">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="f07fc-512">Ebben a módban a rövid ideig alaphelyzetbe áll.</span><span class="sxs-lookup"><span data-stu-id="f07fc-512">This mode is reset after a short period.</span></span>

<span data-ttu-id="f07fc-513">A kivételek teszi közzé a Service Bus által visszaadott a **IsTransient** tulajdonságot, amely jelzi, ha az ügyfél érdemes újra megpróbálnia a művelet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-513">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="f07fc-514">A beépített **RetryExponential** házirend támaszkodik a **IsTransient** tulajdonságot a **MessagingException** osztály, amely az összes Service Bus-kivételek alaposztálya.</span><span class="sxs-lookup"><span data-stu-id="f07fc-514">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="f07fc-515">Egyéni implementációja létrehozásakor a **RetryPolicy** használhatja a kivétel típusát kombinációja alaposztály és az **IsTransient** tulajdonság adja meg a részletesebb vezérlés újrapróbálkozási keresztül műveletek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-515">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="f07fc-516">Például találta a **QuotaExceededException** és a szükséges műveletek a várólista kiürítése előtt üzenetet küld.</span><span class="sxs-lookup"><span data-stu-id="f07fc-516">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="f07fc-517">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="f07fc-517">Policy configuration</span></span>
<span data-ttu-id="f07fc-518">Újrapróbálkozási házirendek programozott módon van beállítva, és az alapértelmezett házirend állítható be egy **NamespaceManager** és egy **MessagingFactory**, vagy külön-külön az egyes üzenetküldési ügyfelek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-518">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="f07fc-519">Az alapértelmezett újrapróbálkozási házirend beállítását egy üzenetkezelési munkamenet beállíthatja a **RetryPolicy** , a **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-519">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

<span data-ttu-id="f07fc-520">Az üzenetkezelési gyár alapján létrehozott összes ügyfél alapértelmezett újrapróbálkozási házirend beállításához állítsa be a **RetryPolicy** , a **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-520">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

<span data-ttu-id="f07fc-521">Az üzenetküldési ügyfél újrapróbálkozási házirend beállítását, vagy az alapértelmezett házirend felülbírálása, beállíthatja a **RetryPolicy** tulajdonságon keresztül a kívánt házirend osztály egy példányát:</span><span class="sxs-lookup"><span data-stu-id="f07fc-521">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="f07fc-522">Az újrapróbálkozási házirendje nem állítható be, az egyedi művelet szintjén.</span><span class="sxs-lookup"><span data-stu-id="f07fc-522">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="f07fc-523">Az üzenetküldési ügyfél minden műveletet vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="f07fc-523">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="f07fc-524">A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-524">The following table shows the default settings for the built-in retry policy.</span></span>

![Ismételje meg a útmutatást tábla](./images/retry-service-specific/RetryServiceSpecificGuidanceTable7.png)

### <a name="retry-usage-guidance"></a><span data-ttu-id="f07fc-526">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-526">Retry usage guidance</span></span>
<span data-ttu-id="f07fc-527">Vegye figyelembe az alábbi iránymutatásokat, amikor a Service Bus használatával:</span><span class="sxs-lookup"><span data-stu-id="f07fc-527">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="f07fc-528">A beépített használatakor **RetryExponential** végrehajtására, tegye implementálja egy tartalék műveletet, mert a házirend reagál a kiszolgáló elfoglalt kivételeket, és automatikusan egy megfelelő újrapróbálkozási módra vált.</span><span class="sxs-lookup"><span data-stu-id="f07fc-528">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="f07fc-529">A Service Bus névterek megfeleltetni, amely a biztonsági mentési üzenetsorba automatikus feladatátvétel különálló névtérben levő elsődleges névtér várólista meghibásodásakor nevezett szolgáltatással támogatja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-529">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="f07fc-530">A másodlagos várólistában lévő üzenetek küldhető vissza az elsődleges queue amikor azt állítja helyre.</span><span class="sxs-lookup"><span data-stu-id="f07fc-530">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="f07fc-531">Ez a szolgáltatás segítségével történő címet átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="f07fc-531">This feature helps to address transient failures.</span></span> <span data-ttu-id="f07fc-532">További információkért lásd: [aszinkron üzenetkezelési mintákat és magas rendelkezésre állású](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="f07fc-532">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="f07fc-533">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="f07fc-533">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="f07fc-534">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="f07fc-534">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

![Ismételje meg a útmutatást tábla](./images/retry-service-specific/RetryServiceSpecificGuidanceTable8.png)

### <a name="telemetry"></a><span data-ttu-id="f07fc-536">Telemetria</span><span class="sxs-lookup"><span data-stu-id="f07fc-536">Telemetry</span></span>
<span data-ttu-id="f07fc-537">A Service Bus újrapróbálkozások naplózza az ETW-eseményként is használ egy **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-537">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="f07fc-538">Hozzá kell rendelni egy **eseményfigyelő Visszahívásán** az események rögzítése és megtekinthetők a teljesítmény-megjelenítőben, vagy azokat a megfelelő cél naplóba bejegyezni esemény forrását.</span><span class="sxs-lookup"><span data-stu-id="f07fc-538">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="f07fc-539">Használhatja a [szemantikai naplózás alkalmazás blokk](http://msdn.microsoft.com/library/dn775006.aspx) ehhez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-539">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="f07fc-540">A kísérletek naplózása a következő formátumot követi a következők:</span><span class="sxs-lookup"><span data-stu-id="f07fc-540">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="f07fc-541">Példák</span><span class="sxs-lookup"><span data-stu-id="f07fc-541">Examples</span></span>
<span data-ttu-id="f07fc-542">Az alábbi példakód bemutatja, hogyan állítsa be a műveletek újrapróbálkozási házirendje esetében:</span><span class="sxs-lookup"><span data-stu-id="f07fc-542">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="f07fc-543">Egy névtér-kezelőt.</span><span class="sxs-lookup"><span data-stu-id="f07fc-543">A namespace manager.</span></span> <span data-ttu-id="f07fc-544">A házirend vonatkozik, hogy a kezelő az összes műveletet, és nem bírálható felül az egyes műveletek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-544">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="f07fc-545">Egy üzenetkezelési gyárból.</span><span class="sxs-lookup"><span data-stu-id="f07fc-545">A messaging factory.</span></span> <span data-ttu-id="f07fc-546">A házirend alapján, hogy a gyári létrehozott összes ügyfelére érvényes, és nem bírálható felül az egyes ügyfelek létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-546">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="f07fc-547">Az egyéni üzenetküldési ügyfél.</span><span class="sxs-lookup"><span data-stu-id="f07fc-547">An individual messaging client.</span></span> <span data-ttu-id="f07fc-548">Ügyfél létrehozása után az újrapróbálkozási házirendet beállíthatja, hogy az ügyfélszámítógépek számára.</span><span class="sxs-lookup"><span data-stu-id="f07fc-548">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="f07fc-549">A házirend vonatkozik, hogy az ügyfél összes művelete.</span><span class="sxs-lookup"><span data-stu-id="f07fc-549">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="f07fc-550">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-550">More information</span></span>
* [<span data-ttu-id="f07fc-551">Aszinkron üzenetkezelési mintát és magas rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="f07fc-551">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a><span data-ttu-id="f07fc-552">Azure Redis Cache újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-552">Azure Redis Cache retry guidelines</span></span>
<span data-ttu-id="f07fc-553">Azure Redis Cache egy gyors adatok elérése és alacsony késési igényű cache szolgáltatás a népszerű nyílt forráskódú Redis Cache alapján.</span><span class="sxs-lookup"><span data-stu-id="f07fc-553">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="f07fc-554">Biztonságos, Microsoft által felügyelt és az Azure-ban minden alkalmazás elérhető-e.</span><span class="sxs-lookup"><span data-stu-id="f07fc-554">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="f07fc-555">Ez a szakasz az útmutató alapján a StackExchange.Redis-ügyfelet használó gyorsítótár-hozzáféréshez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-555">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="f07fc-556">Megfelelő ügyfelek listáját megtalálja az a [Redis webhely](http://redis.io/clients), és ezek különböző újrapróbálkozásos mechanizmusok.</span><span class="sxs-lookup"><span data-stu-id="f07fc-556">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="f07fc-557">Vegye figyelembe, hogy a StackExchange.Redis ügyfél használ-e az multiplexáló egyetlen kapcsolaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="f07fc-557">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="f07fc-558">Az ajánlott használati, hogy az ügyfél példányt létrehozni az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár minden műveletnél.</span><span class="sxs-lookup"><span data-stu-id="f07fc-558">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="f07fc-559">Emiatt a gyorsítótárba kapcsolat csak egyszer, és így az összes ebben a szakaszban található útmutatót az újrapróbálkozási házirendet a kezdeti kapcsolat kapcsolatos – és nem az egyes műveletek, aki hozzáfér a gyorsítótárban.</span><span class="sxs-lookup"><span data-stu-id="f07fc-559">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-560">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-560">Retry mechanism</span></span>
<span data-ttu-id="f07fc-561">A StackExchange.Redis ügyfél használja egy kapcsolat manager osztály, amely olyan beállítási lehetőségekkel, incuding konfigurálva:</span><span class="sxs-lookup"><span data-stu-id="f07fc-561">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="f07fc-562">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-562">**ConnectRetry**.</span></span> <span data-ttu-id="f07fc-563">A száma a gyorsítótárban nem sikerült kapcsolódni a rendszer megpróbálja újból.</span><span class="sxs-lookup"><span data-stu-id="f07fc-563">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="f07fc-564">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-564">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="f07fc-565">Az újrapróbálkozási stratégiát kíván használni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-565">The retry strategy to use.</span></span>
- <span data-ttu-id="f07fc-566">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-566">**ConnectTimeout**.</span></span> <span data-ttu-id="f07fc-567">A maximális várakozási idő milliszekundumban.</span><span class="sxs-lookup"><span data-stu-id="f07fc-567">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="f07fc-568">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="f07fc-568">Policy configuration</span></span>
<span data-ttu-id="f07fc-569">Újrapróbálkozási házirendek programozott módon a beállításainak az ügyfél a gyorsítótár való csatlakozás előtt.</span><span class="sxs-lookup"><span data-stu-id="f07fc-569">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="f07fc-570">Hozzon létre egy példányát erre a **ConfigurationOptions** osztály, feltöltése a tulajdonságait, és úgy, hogy átadja a **Connect** metódust.</span><span class="sxs-lookup"><span data-stu-id="f07fc-570">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="f07fc-571">A beépített osztályok lineáris (állandó) késleltetés és exponenciális leállítási újrapróbálkozási véletlenszerű időközökkel támogatja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-571">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="f07fc-572">Egyéni újrapróbálkozási házirendje implementálásával is létrehozhat a **IReconnectRetryPolicy** felületet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-572">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="f07fc-573">Az alábbi példa használatával exponenciális leállítási újrapróbálkozási stratégiát konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-573">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="f07fc-574">Másik lehetőségként adja meg a beállításokat egy karakterlánc, és adja át a a **Connect** metódust.</span><span class="sxs-lookup"><span data-stu-id="f07fc-574">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="f07fc-575">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezzel a módszerrel csak a kódot.</span><span class="sxs-lookup"><span data-stu-id="f07fc-575">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="f07fc-576">Közvetlenül a gyorsítótárba csatlakoztatásakor is megadható beállítások.</span><span class="sxs-lookup"><span data-stu-id="f07fc-576">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="f07fc-577">További információkért lásd: [verem Exchange Redis konfigurációja](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="f07fc-577">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="f07fc-578">A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-578">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="f07fc-579">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="f07fc-579">**Context**</span></span> | <span data-ttu-id="f07fc-580">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="f07fc-580">**Setting**</span></span> | <span data-ttu-id="f07fc-581">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="f07fc-581">**Default value**</span></span><br /><span data-ttu-id="f07fc-582">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="f07fc-582">(v 1.2.2)</span></span> | <span data-ttu-id="f07fc-583">**Jelentése**</span><span class="sxs-lookup"><span data-stu-id="f07fc-583">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="f07fc-584">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="f07fc-584">ConfigurationOptions</span></span> |<span data-ttu-id="f07fc-585">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="f07fc-585">ConnectRetry</span></span><br /><br /><span data-ttu-id="f07fc-586">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="f07fc-586">ConnectTimeout</span></span><br /><br /><span data-ttu-id="f07fc-587">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="f07fc-587">SyncTimeout</span></span><br /><br /><span data-ttu-id="f07fc-588">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="f07fc-588">ReconnectRetryPolicy</span></span> |<span data-ttu-id="f07fc-589">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-589">3</span></span><br /><br /><span data-ttu-id="f07fc-590">Legfeljebb 5000 ms plus SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="f07fc-590">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="f07fc-591">1000</span><span class="sxs-lookup"><span data-stu-id="f07fc-591">1000</span></span><br /><br /><span data-ttu-id="f07fc-592">5000 ms LinearRetry</span><span class="sxs-lookup"><span data-stu-id="f07fc-592">LinearRetry 5000 ms</span></span> |<span data-ttu-id="f07fc-593">Ismétlődő hányszor kísérletek csatlakozzon a kezdeti kapcsolat művelet során.</span><span class="sxs-lookup"><span data-stu-id="f07fc-593">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="f07fc-594">Időkorlát (ms) a műveletek csatlakozzon.</span><span class="sxs-lookup"><span data-stu-id="f07fc-594">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="f07fc-595">Nem a késleltetés újrapróbálkozási kísérletek között.</span><span class="sxs-lookup"><span data-stu-id="f07fc-595">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="f07fc-596">Idő (ms) szinkron műveletek engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="f07fc-596">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="f07fc-597">Ismételje meg minden 5000 ms.</span><span class="sxs-lookup"><span data-stu-id="f07fc-597">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="f07fc-598">A szinkron műveletek `SyncTimeout` adhat hozzá a végpontok közötti késés, de ha az értéket túl alacsonyra jelentős időtúllépéseket okozhat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-598">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="f07fc-599">Lásd: [hibaelhárítása az Azure Redis Cache][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="f07fc-599">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="f07fc-600">Általánosságban elmondható kerülje a szinkron műveletek, és használja helyette az aszinkron műveletek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-600">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="f07fc-601">További információ: [folyamatok és Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="f07fc-601">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="f07fc-602">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-602">Retry usage guidance</span></span>
<span data-ttu-id="f07fc-603">Azure Redis Cache használata esetén, vegye figyelembe a következő irányelveket:</span><span class="sxs-lookup"><span data-stu-id="f07fc-603">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="f07fc-604">A StackExchange Redis ügyfél felügyeli a saját újrapróbálkozások, de csak ha a kapcsolat létrehozása a gyorsítótárba az alkalmazás első indításakor.</span><span class="sxs-lookup"><span data-stu-id="f07fc-604">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="f07fc-605">Konfigurálhatja a kapcsolat időkorlátja, újrapróbálkozási kísérleteinek száma és a kapcsolat létrehozása az újrapróbálkozások között eltelt idő, de az újrapróbálkozási házirendje nem vonatkozik a gyorsítótár műveleteket.</span><span class="sxs-lookup"><span data-stu-id="f07fc-605">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="f07fc-606">Számos újrapróbálkozás helyett fontolja meg, de visszatér az eredeti adatforrás helyette elérésével.</span><span class="sxs-lookup"><span data-stu-id="f07fc-606">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="f07fc-607">Telemetria</span><span class="sxs-lookup"><span data-stu-id="f07fc-607">Telemetry</span></span>
<span data-ttu-id="f07fc-608">Akkor is gyűjthet információt használatával kapcsolatok (de nem más műveletek) egy **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-608">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="f07fc-609">Ezt követően a kimeneti példát alább láthatók.</span><span class="sxs-lookup"><span data-stu-id="f07fc-609">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="f07fc-610">Példák</span><span class="sxs-lookup"><span data-stu-id="f07fc-610">Examples</span></span>
<span data-ttu-id="f07fc-611">Az alábbi példakód újrapróbálkozások, amikor a StackExchange.Redis-ügyfél inicializálása állandó (lineáris) késleltetés konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-611">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="f07fc-612">Ez a példa bemutatja, hogyan telepíthet a használt konfiguráció egy **ConfigurationOptions** példány.</span><span class="sxs-lookup"><span data-stu-id="f07fc-612">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="f07fc-613">A következő példában a konfigurációs beállítások megadásával karakterláncként állítja be.</span><span class="sxs-lookup"><span data-stu-id="f07fc-613">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="f07fc-614">A kapcsolat időtúllépési érték a várakozási idő a kapcsolatot a gyorsítótárba, nem újrapróbálkozási kísérletek között eltelt maximális időtartam.</span><span class="sxs-lookup"><span data-stu-id="f07fc-614">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="f07fc-615">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság csak kód által állítható be.</span><span class="sxs-lookup"><span data-stu-id="f07fc-615">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="f07fc-616">További példákért lásd [konfigurációs](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) a projekt webhelyen.</span><span class="sxs-lookup"><span data-stu-id="f07fc-616">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="f07fc-617">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-617">More information</span></span>
* [<span data-ttu-id="f07fc-618">Webhely redis</span><span class="sxs-lookup"><span data-stu-id="f07fc-618">Redis website</span></span>](http://redis.io/)

## <a name="documentdb-api-retry-guidelines"></a><span data-ttu-id="f07fc-619">A DocumentDB API újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-619">DocumentDB API retry guidelines</span></span>

<span data-ttu-id="f07fc-620">Cosmos DB egy teljes körűen felügyelt több modellre adatbázis, amely támogatja a séma nélküli JSON-adatokat a [DocumentDB API][documentdb-api].</span><span class="sxs-lookup"><span data-stu-id="f07fc-620">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data by using the [DocumentDB API][documentdb-api].</span></span> <span data-ttu-id="f07fc-621">Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="f07fc-621">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-622">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-622">Retry mechanism</span></span>
<span data-ttu-id="f07fc-623">A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-623">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="f07fc-624">Az újrapróbálkozások száma és a maximális várakozási idő beállításához konfigurálása [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="f07fc-624">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="f07fc-625">Kivételek, amely kiváltja az ügyfél újrapróbálkozási házirend túl vagy nem átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="f07fc-625">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="f07fc-626">Ha Cosmos DB azelőtt gyorsítja fel, az ügyfél, HTTP 429 hibát ad vissza.</span><span class="sxs-lookup"><span data-stu-id="f07fc-626">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="f07fc-627">Az állapotkód: Ellenőrizze a `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="f07fc-627">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="f07fc-628">Házirend-konfiguráció</span><span class="sxs-lookup"><span data-stu-id="f07fc-628">Policy configuration</span></span>
<span data-ttu-id="f07fc-629">A következő táblázat alapértelmezett beállításait a `RetryOptions` osztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-629">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="f07fc-630">Beállítás</span><span class="sxs-lookup"><span data-stu-id="f07fc-630">Setting</span></span> | <span data-ttu-id="f07fc-631">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="f07fc-631">Default value</span></span> | <span data-ttu-id="f07fc-632">Leírás</span><span class="sxs-lookup"><span data-stu-id="f07fc-632">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="f07fc-633">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="f07fc-633">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="f07fc-634">9</span><span class="sxs-lookup"><span data-stu-id="f07fc-634">9</span></span> |<span data-ttu-id="f07fc-635">Ha a kérés nem teljesíthető, mivel a Cosmos DB sebessége korlátozza az ügyfél az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="f07fc-635">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="f07fc-636">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="f07fc-636">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="f07fc-637">30</span><span class="sxs-lookup"><span data-stu-id="f07fc-637">30</span></span> |<span data-ttu-id="f07fc-638">A maximális idő másodpercben. Próbálkozzon újra.</span><span class="sxs-lookup"><span data-stu-id="f07fc-638">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="f07fc-639">Példa</span><span class="sxs-lookup"><span data-stu-id="f07fc-639">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="f07fc-640">Telemetria</span><span class="sxs-lookup"><span data-stu-id="f07fc-640">Telemetry</span></span>
<span data-ttu-id="f07fc-641">Újrapróbálkozások bejelentkezett egy .NET keresztül strukturálatlan nyomkövetési üzenetekként **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="f07fc-641">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="f07fc-642">Konfigurálnia kell egy **TraceListener** az események rögzítése és a megfelelő cél a naplófájlba írja őket.</span><span class="sxs-lookup"><span data-stu-id="f07fc-642">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="f07fc-643">Például ha az App.config fájlban adja hozzá a következő, nyomkövetési adatokat a rendszer létrehoz és a végrehajtható fájl ugyanazon a helyen a fájlt:</span><span class="sxs-lookup"><span data-stu-id="f07fc-643">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="DocumentDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a><span data-ttu-id="f07fc-644">Az Azure Search újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-644">Azure Search retry guidelines</span></span>
<span data-ttu-id="f07fc-645">Az Azure Search segítségével hatékony és kifinomult keresési-képességeket adhat a webhelyhez vagy alkalmazáshoz, gyorsan és egyszerűen finomhangolhatják a keresési eredmények között, és gazdag és finomhangolható rangsorolási modellek létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="f07fc-645">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-646">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-646">Retry mechanism</span></span>
<span data-ttu-id="f07fc-647">Az Azure Search SDK-ban való újrapróbálkozásra vezérli a `SetRetryPolicy` metódust a [SearchServiceClient] és [SearchIndexClient] osztályok.</span><span class="sxs-lookup"><span data-stu-id="f07fc-647">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="f07fc-648">Az alapértelmezett házirend ismétlések exponenciális leállítási, ha Azure Search egy 5xx vagy 408 (kérése időtúllépés) választ ad vissza.</span><span class="sxs-lookup"><span data-stu-id="f07fc-648">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="f07fc-649">Telemetria</span><span class="sxs-lookup"><span data-stu-id="f07fc-649">Telemetry</span></span>
<span data-ttu-id="f07fc-650">Nyomkövetési ETW vagy egy egyéni nyomkövetési szolgáltató regisztrálja.</span><span class="sxs-lookup"><span data-stu-id="f07fc-650">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="f07fc-651">További információkért lásd: a [AutoRest dokumentáció][autorest].</span><span class="sxs-lookup"><span data-stu-id="f07fc-651">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="azure-active-directory-retry-guidelines"></a><span data-ttu-id="f07fc-652">Az Azure Active Directory újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-652">Azure Active Directory retry guidelines</span></span>
<span data-ttu-id="f07fc-653">Azure Active Directory (Azure AD) egy olyan átfogó identitás- és hozzáférés felügyeleti felhőalapú megoldás, hogy a címtárszolgáltatások mag, speciális identitás irányítás, biztonsági és alkalmazáshozzáférés-kezeléshez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-653">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="f07fc-654">Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi házirendeken és szabályokon alapuló hozzáférés-szabályozással bővíthessék az alkalmazásaikat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-654">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-655">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-655">Retry mechanism</span></span>
<span data-ttu-id="f07fc-656">Az Azure Active Directory a az Active Directory Authentication Library (ADAL) beépített újrapróbálkozási mechanizmussal van.</span><span class="sxs-lookup"><span data-stu-id="f07fc-656">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="f07fc-657">Váratlan zárolásokat elkerülése érdekében azt javasoljuk, hogy külső gyártótól származó kódtárak és alkalmazáskód hajthatja végre *nem* próbálja meg újra a hibás kapcsolatok, de lehetővé teszi az ADAL újrapróbálkozások kezelésére.</span><span class="sxs-lookup"><span data-stu-id="f07fc-657">To avoid unexpected lockouts, we recommend that third party libraries and application code do *not* retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="f07fc-658">Ismételje meg a használati útmutató</span><span class="sxs-lookup"><span data-stu-id="f07fc-658">Retry usage guidance</span></span>
<span data-ttu-id="f07fc-659">Amikor az Azure Active Directoryval, vegye figyelembe az alábbi irányelveket:</span><span class="sxs-lookup"><span data-stu-id="f07fc-659">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="f07fc-660">Ha lehetséges, használja az ADAL-könyvtár és a beépített támogatása az ismételt próbálkozás.</span><span class="sxs-lookup"><span data-stu-id="f07fc-660">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="f07fc-661">Ha az Azure Active Directory a REST API-t használ, érdemes újra megpróbálnia a művelet csak akkor, ha a 5xx tartomány (például 500 belső kiszolgálóhiba 502 Hibás átjáró, 503-as szolgáltatás nem érhető el vagy 504-es számú átjáró időtúllépése) hiba eredménye.</span><span class="sxs-lookup"><span data-stu-id="f07fc-661">If you are using the REST API for Azure Active Directory, you should retry the operation only if the result is an error in the 5xx range (such as 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, and 504 Gateway Timeout).</span></span> <span data-ttu-id="f07fc-662">Nem próbálja meg újra más hibákat.</span><span class="sxs-lookup"><span data-stu-id="f07fc-662">Do not retry for any other errors.</span></span>
* <span data-ttu-id="f07fc-663">Az exponenciális vissza az indító házirend ajánlott kötegelt olyan esetekben, az Azure Active Directoryban.</span><span class="sxs-lookup"><span data-stu-id="f07fc-663">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="f07fc-664">Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési.</span><span class="sxs-lookup"><span data-stu-id="f07fc-664">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="f07fc-665">Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.</span><span class="sxs-lookup"><span data-stu-id="f07fc-665">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="f07fc-666">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="f07fc-666">**Context**</span></span> | <span data-ttu-id="f07fc-667">**A minta cél E2E<br />maximális késleltetés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-667">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="f07fc-668">**Ismételje meg a stratégia**</span><span class="sxs-lookup"><span data-stu-id="f07fc-668">**Retry strategy**</span></span> | <span data-ttu-id="f07fc-669">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="f07fc-669">**Settings**</span></span> | <span data-ttu-id="f07fc-670">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="f07fc-670">**Values**</span></span> | <span data-ttu-id="f07fc-671">**Működés**</span><span class="sxs-lookup"><span data-stu-id="f07fc-671">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="f07fc-672">Interaktív, felhasználói felületén</span><span class="sxs-lookup"><span data-stu-id="f07fc-672">Interactive, UI,</span></span><br /><span data-ttu-id="f07fc-673">előtérben vagy a</span><span class="sxs-lookup"><span data-stu-id="f07fc-673">or foreground</span></span> |<span data-ttu-id="f07fc-674">2 mp</span><span class="sxs-lookup"><span data-stu-id="f07fc-674">2 sec</span></span> |<span data-ttu-id="f07fc-675">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="f07fc-675">FixedInterval</span></span> |<span data-ttu-id="f07fc-676">Újbóli próbálkozások száma</span><span class="sxs-lookup"><span data-stu-id="f07fc-676">Retry count</span></span><br /><span data-ttu-id="f07fc-677">Újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="f07fc-677">Retry interval</span></span><br /><span data-ttu-id="f07fc-678">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="f07fc-678">First fast retry</span></span> |<span data-ttu-id="f07fc-679">3</span><span class="sxs-lookup"><span data-stu-id="f07fc-679">3</span></span><br /><span data-ttu-id="f07fc-680">500 ms</span><span class="sxs-lookup"><span data-stu-id="f07fc-680">500 ms</span></span><br /><span data-ttu-id="f07fc-681">igaz</span><span class="sxs-lookup"><span data-stu-id="f07fc-681">true</span></span> |<span data-ttu-id="f07fc-682">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-682">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="f07fc-683">Kísérlet történt a 2 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-683">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="f07fc-684">Próbálja meg a 3 - 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-684">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="f07fc-685">Háttér vagy</span><span class="sxs-lookup"><span data-stu-id="f07fc-685">Background or</span></span><br /><span data-ttu-id="f07fc-686">Kötegelt</span><span class="sxs-lookup"><span data-stu-id="f07fc-686">batch</span></span> |<span data-ttu-id="f07fc-687">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-687">60 sec</span></span> |<span data-ttu-id="f07fc-688">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="f07fc-688">ExponentialBackoff</span></span> |<span data-ttu-id="f07fc-689">Újbóli próbálkozások száma</span><span class="sxs-lookup"><span data-stu-id="f07fc-689">Retry count</span></span><br /><span data-ttu-id="f07fc-690">Minimális biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="f07fc-690">Min back-off</span></span><br /><span data-ttu-id="f07fc-691">Maximális vissza kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="f07fc-691">Max back-off</span></span><br /><span data-ttu-id="f07fc-692">Különbözeti biztonsági kikapcsolása</span><span class="sxs-lookup"><span data-stu-id="f07fc-692">Delta back-off</span></span><br /><span data-ttu-id="f07fc-693">Első gyors újrapróbálkozási</span><span class="sxs-lookup"><span data-stu-id="f07fc-693">First fast retry</span></span> |<span data-ttu-id="f07fc-694">5</span><span class="sxs-lookup"><span data-stu-id="f07fc-694">5</span></span><br /><span data-ttu-id="f07fc-695">0 (mp)</span><span class="sxs-lookup"><span data-stu-id="f07fc-695">0 sec</span></span><br /><span data-ttu-id="f07fc-696">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="f07fc-696">60 sec</span></span><br /><span data-ttu-id="f07fc-697">2 mp</span><span class="sxs-lookup"><span data-stu-id="f07fc-697">2 sec</span></span><br /><span data-ttu-id="f07fc-698">hamis</span><span class="sxs-lookup"><span data-stu-id="f07fc-698">false</span></span> |<span data-ttu-id="f07fc-699">Kísérlet történt az 1 – 0 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-699">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="f07fc-700">Kísérlet történt a 2 - ~ 2 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-700">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="f07fc-701">Próbálja meg a 3 - ~ 6 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-701">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="f07fc-702">Kísérlet történt a 4 – ~ 14 mp késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-702">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="f07fc-703">Kísérlet történt az 5 – kb. 30 másodperc késleltetés</span><span class="sxs-lookup"><span data-stu-id="f07fc-703">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="f07fc-704">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-704">More information</span></span>
* <span data-ttu-id="f07fc-705">[Az Azure Active Directory hitelesítési Kódtárai][adal]</span><span class="sxs-lookup"><span data-stu-id="f07fc-705">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="service-fabric-retry-guidelines"></a><span data-ttu-id="f07fc-706">A Service Fabric újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-706">Service Fabric retry guidelines</span></span>

<span data-ttu-id="f07fc-707">A Service Fabric megbízható szolgáltatások terjesztése fürt védi a cikkben szereplő lehetséges átmeneti hibák többségét ellen.</span><span class="sxs-lookup"><span data-stu-id="f07fc-707">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="f07fc-708">Átmeneti hibákat azonban továbbra is lehetséges, amelyek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-708">Some transient faults are still possible, however.</span></span> <span data-ttu-id="f07fc-709">Például a naming service közepén útválasztási módosítása onnan kapta, hogy a kérelmet, ha okozza, hogy kivételt jelez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-709">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="f07fc-710">Ha a kérésben 100 milliomod másodperc később, valószínűleg sikeres lesz.</span><span class="sxs-lookup"><span data-stu-id="f07fc-710">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="f07fc-711">A Service Fabric belsőleg, az ilyen átmeneti hiba kezeli.</span><span class="sxs-lookup"><span data-stu-id="f07fc-711">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="f07fc-712">Egyes beállítások segítségével konfigurálhatja a `OperationRetrySettings` osztály a szolgáltatások beállítása közben.</span><span class="sxs-lookup"><span data-stu-id="f07fc-712">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="f07fc-713">A következő kód egy példát mutat be.</span><span class="sxs-lookup"><span data-stu-id="f07fc-713">The following code shows an example.</span></span> <span data-ttu-id="f07fc-714">A legtöbb esetben ne legyen szükséges, és az alapértelmezett beállításokat sikeres lesz.</span><span class="sxs-lookup"><span data-stu-id="f07fc-714">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

## <a name="more-information"></a><span data-ttu-id="f07fc-715">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-715">More information</span></span>

* [<span data-ttu-id="f07fc-716">Távoli kivételkezelést</span><span class="sxs-lookup"><span data-stu-id="f07fc-716">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a><span data-ttu-id="f07fc-717">Az Azure Event Hubs újra irányelvek</span><span class="sxs-lookup"><span data-stu-id="f07fc-717">Azure Event Hubs retry guidelines</span></span>

<span data-ttu-id="f07fc-718">Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-718">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="f07fc-719">Ismételje meg a mechanizmus</span><span class="sxs-lookup"><span data-stu-id="f07fc-719">Retry mechanism</span></span>
<span data-ttu-id="f07fc-720">Az Azure Event Hubs ügyféloldali kódtár a újrapróbálkozásra vezérli a `RetryPolicy` tulajdonságát a `EventHubClient` osztály.</span><span class="sxs-lookup"><span data-stu-id="f07fc-720">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="f07fc-721">Az alapértelmezett exponenciális leállítási, amikor az Azure Event Hubs adja vissza egy átmeneti házirend ismétlések `EventHubsException` vagy egy `OperationCanceledException`.</span><span class="sxs-lookup"><span data-stu-id="f07fc-721">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="f07fc-722">Példa</span><span class="sxs-lookup"><span data-stu-id="f07fc-722">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="f07fc-723">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-723">More information</span></span>
[<span data-ttu-id="f07fc-724">Az Azure Event Hubs szabványos .NET ügyféloldali kódtár</span><span class="sxs-lookup"><span data-stu-id="f07fc-724"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="f07fc-725">Általános REST, és próbálkozzon újra irányelveket</span><span class="sxs-lookup"><span data-stu-id="f07fc-725">General REST and retry guidelines</span></span>
<span data-ttu-id="f07fc-726">Azure vagy harmadik féltől származó szolgáltatások eléréséhez vegye figyelembe a következőket:</span><span class="sxs-lookup"><span data-stu-id="f07fc-726">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="f07fc-727">Egy rendszeres megközelítés, az újrapróbálkozásokat lehet, hogy kódú újrafelhasználható, akkor használja, így a egységes módszer alkalmazhatók minden ügyfél és az összes megoldás.</span><span class="sxs-lookup"><span data-stu-id="f07fc-727">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="f07fc-728">Fontolja meg az átmeneti hiba kezelési alkalmazás-blokk újrapróbálkozási keretrendszer használatát újrapróbálkozások kezelése, ha a célként megadott szolgáltatás vagy az ügyfél nem beépített újrapróbálkozási mechanizmussal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="f07fc-728">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="f07fc-729">Ez segítséget nyújt egy egységes újrapróbálkozási viselkedés, és azt a célként megadott szolgáltatás egy megfelelő alapértelmezett újrapróbálkozási stratégiát rendelkezhetnek.</span><span class="sxs-lookup"><span data-stu-id="f07fc-729">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="f07fc-730">Azonban szükség lehet, hogy létrehozzon egyéni újrapróbálkozási kódot azon szolgáltatások, amelyek nem szabványos viselkedését, ne használja a kivételeket úgy, hogy átmeneti hiba azt jelzi, vagy ha szeretné használni egy **újrapróbálkozási-válasz** válasz újrapróbálkozásra kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="f07fc-730">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="f07fc-731">Az átmeneti észlelési logika az ügyfél tényleges API meghívása a REST-hívások segítségével függ.</span><span class="sxs-lookup"><span data-stu-id="f07fc-731">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="f07fc-732">Egyes ügyfelek, például a újabb **HttpClient** osztály, nem egy nem sikeres HTTP-állapotkód: teljesített kérelmek kivételeinek kivételhibát.</span><span class="sxs-lookup"><span data-stu-id="f07fc-732">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="f07fc-733">Ez javítja a teljesítményt, de megakadályozza, hogy az átmeneti hiba kezelési alkalmazás-blokk használata.</span><span class="sxs-lookup"><span data-stu-id="f07fc-733">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="f07fc-734">Ebben az esetben burkolása sikerült kóddal, amely nem sikeres HTTP-állapotkódok, amely a blokk majd tudja feldolgozni a kivételeket a REST API-hívás.</span><span class="sxs-lookup"><span data-stu-id="f07fc-734">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="f07fc-735">Egy másik mechanizmus segítségével azt is megteheti, az újbóli próbálkozások meghajtó.</span><span class="sxs-lookup"><span data-stu-id="f07fc-735">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="f07fc-736">A szolgáltatás által visszaadott HTTP-állapotkód: jelzi, hogy a hiba átmeneti segítséget.</span><span class="sxs-lookup"><span data-stu-id="f07fc-736">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="f07fc-737">Előfordulhat, hogy egy ügyfél vagy a újrapróbálkozási keretrendszer állapotkód eléréséhez, vagy a megfelelő kivétel típusának meghatározására által létrehozott kivételek vizsgálata szükséges.</span><span class="sxs-lookup"><span data-stu-id="f07fc-737">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="f07fc-738">A következő HTTP-kódok általában azt jelzi, hogy egy újabb megfelelő:</span><span class="sxs-lookup"><span data-stu-id="f07fc-738">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="f07fc-739">408 kérelmi időkorlátot.</span><span class="sxs-lookup"><span data-stu-id="f07fc-739">408 Request Timeout</span></span>
  * <span data-ttu-id="f07fc-740">500 belső kiszolgálóhiba</span><span class="sxs-lookup"><span data-stu-id="f07fc-740">500 Internal Server Error</span></span>
  * <span data-ttu-id="f07fc-741">502 Hibás átjáró</span><span class="sxs-lookup"><span data-stu-id="f07fc-741">502 Bad Gateway</span></span>
  * <span data-ttu-id="f07fc-742">503-as szolgáltatás nem érhető el</span><span class="sxs-lookup"><span data-stu-id="f07fc-742">503 Service Unavailable</span></span>
  * <span data-ttu-id="f07fc-743">504-es számú átjáró időtúllépése</span><span class="sxs-lookup"><span data-stu-id="f07fc-743">504 Gateway Timeout</span></span>
* <span data-ttu-id="f07fc-744">Ha a kivételek az újrapróbálkozási logika, a következő általában jelzi, hogy egy átmeneti hiba, ha nem sikerült kapcsolatot:</span><span class="sxs-lookup"><span data-stu-id="f07fc-744">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="f07fc-745">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="f07fc-745">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="f07fc-746">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="f07fc-746">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="f07fc-747">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="f07fc-747">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="f07fc-748">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="f07fc-748">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="f07fc-749">A szolgáltatás nem érhető el állapota esetén a szolgáltatás a megfelelő késleltetés jelezhetik az újrapróbálkozás előtt a **újrapróbálkozási után** válaszfejléc vagy egy másik egyéni fejlécet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-749">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="f07fc-750">Szolgáltatások további információkat is előfordulhat, hogy egyéni fejlécek küldeni, vagy a válasz tartalmának ágyazva.</span><span class="sxs-lookup"><span data-stu-id="f07fc-750">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="f07fc-751">Az átmeneti hiba kezelési alkalmazás-blokk nem használhatja a szabványos vagy egyéni "újrapróbálkozási-után" fejléc.</span><span class="sxs-lookup"><span data-stu-id="f07fc-751">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="f07fc-752">Nem próbálja meg újra az ügyfél-hibák (a 4xx tartomány hibák) jelölő állapotkódok 408 kérelem időtúllépés kivételével.</span><span class="sxs-lookup"><span data-stu-id="f07fc-752">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="f07fc-753">Alaposan tesztelje a újra azokat a stratégiákat és a feltételek, például a különböző hálózati állapotok és a különböző rendszer terhelések számos szerinti.</span><span class="sxs-lookup"><span data-stu-id="f07fc-753">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="f07fc-754">Ismételje meg a stratégiák</span><span class="sxs-lookup"><span data-stu-id="f07fc-754">Retry strategies</span></span>
<span data-ttu-id="f07fc-755">A tipikus objektumtípusokra újrapróbálkozási stratégia intervallumok a következők:</span><span class="sxs-lookup"><span data-stu-id="f07fc-755">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="f07fc-756">**Az exponenciális**: a megadott számú ki megközelítés véletlenszerű exponenciális biztonsági használatával az intervallum újrapróbálkozások között, az újrapróbálkozásokat végző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="f07fc-756">**Exponential**: A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="f07fc-757">Példa:</span><span class="sxs-lookup"><span data-stu-id="f07fc-757">For example:</span></span>

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* <span data-ttu-id="f07fc-758">**Növekményes**: a megadott számú újrapróbálkozások és a próbálkozások növekményes időközét újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="f07fc-758">**Incremental**: A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="f07fc-759">Példa:</span><span class="sxs-lookup"><span data-stu-id="f07fc-759">For example:</span></span>

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* <span data-ttu-id="f07fc-760">**LinearRetry**: használja a megadott időközönként újrapróbálkozások között, az újrapróbálkozásokat megadott számú végző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="f07fc-760">**LinearRetry**: A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="f07fc-761">Példa:</span><span class="sxs-lookup"><span data-stu-id="f07fc-761">For example:</span></span>

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a><span data-ttu-id="f07fc-762">További információ</span><span class="sxs-lookup"><span data-stu-id="f07fc-762">More information</span></span>
* [<span data-ttu-id="f07fc-763">Áramköri megszakító stratégiák</span><span class="sxs-lookup"><span data-stu-id="f07fc-763">Circuit breaker strategies</span></span>](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="f07fc-764">Átmeneti hiba Polly kezelését</span><span class="sxs-lookup"><span data-stu-id="f07fc-764">Transient fault handling with Polly</span></span>
<span data-ttu-id="f07fc-765">[Polly](http://www.thepollyproject.org) egy függvénytár programozottan kezelni az újrapróbálkozások és [áramköri megszakító] [ circuit-breaker] stratégiák.</span><span class="sxs-lookup"><span data-stu-id="f07fc-765">[Polly](http://www.thepollyproject.org) is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="f07fc-766">A Polly projekt tagja a [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="f07fc-766">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="f07fc-767">Ha az ügyfél nem natív módon támogatja az újrapróbálkozások szolgáltatások esetén Polly érvényes alternatíva, és kiküszöböli, egyéni újrapróbálkozási kód, amely implementálja megfelelően nehéz lehet írni.</span><span class="sxs-lookup"><span data-stu-id="f07fc-767">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="f07fc-768">Polly is lehetőséget biztosít nyomkövetési hibák azok bekövetkezésekor, így újrapróbálkozások bejelentkezhet.</span><span class="sxs-lookup"><span data-stu-id="f07fc-768">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[documentdb-api]: /azure/documentdb/documentdb-introduction
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
