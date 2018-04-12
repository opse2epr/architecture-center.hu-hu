---
title: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmushoz
description: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmus beállításához.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 332f96e73def360926b6a934bbb1361b2254ec41
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="17915-103">Újrapróbálkozási útmutatás adott szolgáltatásoknál</span><span class="sxs-lookup"><span data-stu-id="17915-103">Retry guidance for specific services</span></span>

<span data-ttu-id="17915-104">A legtöbb Azure-szolgáltatás és ügyfél-SDK tartalmaz egy újrapróbálkozási mechanizmust.</span><span class="sxs-lookup"><span data-stu-id="17915-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="17915-105">Ezek azonban eltérőek lehetnek, mert minden szolgáltatásnak eltérő jellemzői és követelményei vannak, így mindegyik újrapróbálkozási mechanizmus az adott szolgáltatásra van szabva.</span><span class="sxs-lookup"><span data-stu-id="17915-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="17915-106">Ez az útmutató összefoglalást ad az Azure-szolgáltatások többségére jellemző újrapróbálkozási funkciókról, továbbá segítséget nyújt a szolgáltatás újrapróbálkozási mechanizmusának használatához, módosításához vagy kibővítéséhez.</span><span class="sxs-lookup"><span data-stu-id="17915-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="17915-107">Általános útmutatást az átmeneti hibák kezeléséhez, valamint a szolgáltatásokat és erőforrásokat célzó kapcsolatok és műveletek újrapróbálásához az [újrapróbálkozási útmutatásban](./transient-faults.md) talál.</span><span class="sxs-lookup"><span data-stu-id="17915-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="17915-108">A következő táblázat az útmutatóban érintett Azure-szolgáltatások újrapróbálkozási funkcióiról nyújt áttekintést.</span><span class="sxs-lookup"><span data-stu-id="17915-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="17915-109">**Szolgáltatás**</span><span class="sxs-lookup"><span data-stu-id="17915-109">**Service**</span></span> | <span data-ttu-id="17915-110">**Újrapróbálkozási képességek**</span><span class="sxs-lookup"><span data-stu-id="17915-110">**Retry capabilities**</span></span> | <span data-ttu-id="17915-111">**Szabályzatkonfiguráció**</span><span class="sxs-lookup"><span data-stu-id="17915-111">**Policy configuration**</span></span> | <span data-ttu-id="17915-112">**Hatókör**</span><span class="sxs-lookup"><span data-stu-id="17915-112">**Scope**</span></span> | <span data-ttu-id="17915-113">**Telemetriafunkciók**</span><span class="sxs-lookup"><span data-stu-id="17915-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="17915-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="17915-115">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-115">Native in client</span></span> |<span data-ttu-id="17915-116">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-116">Programmatic</span></span> |<span data-ttu-id="17915-117">Ügyfél- és különálló műveletek</span><span class="sxs-lookup"><span data-stu-id="17915-117">Client and individual operations</span></span> |<span data-ttu-id="17915-118">TraceSource</span><span class="sxs-lookup"><span data-stu-id="17915-118">TraceSource</span></span> |
| <span data-ttu-id="17915-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span></span> |<span data-ttu-id="17915-120">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-120">Native in client</span></span> |<span data-ttu-id="17915-121">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-121">Programmatic</span></span> |<span data-ttu-id="17915-122">Alkalmazástartományonként globális</span><span class="sxs-lookup"><span data-stu-id="17915-122">Global per AppDomain</span></span> |<span data-ttu-id="17915-123">None</span><span class="sxs-lookup"><span data-stu-id="17915-123">None</span></span> |
| <span data-ttu-id="17915-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span></span> |<span data-ttu-id="17915-125">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-125">Native in client</span></span> |<span data-ttu-id="17915-126">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-126">Programmatic</span></span> |<span data-ttu-id="17915-127">Alkalmazástartományonként globális</span><span class="sxs-lookup"><span data-stu-id="17915-127">Global per AppDomain</span></span> |<span data-ttu-id="17915-128">None</span><span class="sxs-lookup"><span data-stu-id="17915-128">None</span></span> |
| <span data-ttu-id="17915-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span></span> |[<span data-ttu-id="17915-130">Polly</span><span class="sxs-lookup"><span data-stu-id="17915-130">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="17915-131">Deklaratív és szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-131">Declarative and programmatic</span></span> |<span data-ttu-id="17915-132">Önálló utasítások vagy kódblokkok</span><span class="sxs-lookup"><span data-stu-id="17915-132">Single statements or blocks of code</span></span> |<span data-ttu-id="17915-133">Egyéni</span><span class="sxs-lookup"><span data-stu-id="17915-133">Custom</span></span> |
| <span data-ttu-id="17915-134">**[Service Bus](#service-bus-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-134">**[Service Bus](#service-bus-retry-guidelines)**</span></span> |<span data-ttu-id="17915-135">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-135">Native in client</span></span> |<span data-ttu-id="17915-136">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-136">Programmatic</span></span> |<span data-ttu-id="17915-137">Névtérkezelő, üzenetkezelési előállító vagy ügyfél</span><span class="sxs-lookup"><span data-stu-id="17915-137">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="17915-138">ETW</span><span class="sxs-lookup"><span data-stu-id="17915-138">ETW</span></span> |
| <span data-ttu-id="17915-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span></span> |<span data-ttu-id="17915-140">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-140">Native in client</span></span> |<span data-ttu-id="17915-141">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-141">Programmatic</span></span> |<span data-ttu-id="17915-142">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="17915-142">Client</span></span> |<span data-ttu-id="17915-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="17915-143">TextWriter</span></span> |
| <span data-ttu-id="17915-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span></span> |<span data-ttu-id="17915-145">Natív, a szolgáltatás része</span><span class="sxs-lookup"><span data-stu-id="17915-145">Native in service</span></span> |<span data-ttu-id="17915-146">Nem konfigurálható</span><span class="sxs-lookup"><span data-stu-id="17915-146">Non-configurable</span></span> |<span data-ttu-id="17915-147">Globális</span><span class="sxs-lookup"><span data-stu-id="17915-147">Global</span></span> |<span data-ttu-id="17915-148">TraceSource</span><span class="sxs-lookup"><span data-stu-id="17915-148">TraceSource</span></span> |
| <span data-ttu-id="17915-149">**[Azure Search](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-149">**[Azure Search](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="17915-150">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-150">Native in client</span></span> |<span data-ttu-id="17915-151">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-151">Programmatic</span></span> |<span data-ttu-id="17915-152">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="17915-152">Client</span></span> |<span data-ttu-id="17915-153">ETW vagy egyéni</span><span class="sxs-lookup"><span data-stu-id="17915-153">ETW or Custom</span></span> |
| <span data-ttu-id="17915-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span></span> |<span data-ttu-id="17915-155">Natív, az ADAL-kódtár része</span><span class="sxs-lookup"><span data-stu-id="17915-155">Native in ADAL library</span></span> |<span data-ttu-id="17915-156">Beágyazva az ADAL-kódtárba</span><span class="sxs-lookup"><span data-stu-id="17915-156">Embeded into ADAL library</span></span> |<span data-ttu-id="17915-157">Belső</span><span class="sxs-lookup"><span data-stu-id="17915-157">Internal</span></span> |<span data-ttu-id="17915-158">Nincs</span><span class="sxs-lookup"><span data-stu-id="17915-158">None</span></span> |
| <span data-ttu-id="17915-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span></span> |<span data-ttu-id="17915-160">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-160">Native in client</span></span> |<span data-ttu-id="17915-161">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-161">Programmatic</span></span> |<span data-ttu-id="17915-162">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="17915-162">Client</span></span> |<span data-ttu-id="17915-163">None</span><span class="sxs-lookup"><span data-stu-id="17915-163">None</span></span> | 
| <span data-ttu-id="17915-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="17915-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span></span> |<span data-ttu-id="17915-165">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="17915-165">Native in client</span></span> |<span data-ttu-id="17915-166">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="17915-166">Programmatic</span></span> |<span data-ttu-id="17915-167">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="17915-167">Client</span></span> |<span data-ttu-id="17915-168">Nincs</span><span class="sxs-lookup"><span data-stu-id="17915-168">None</span></span> |

> [!NOTE]
> <span data-ttu-id="17915-169">Az Azure beépített újrapróbálkozási mechanizmusainak többsége jelenleg – az újrapróbálkozási szabályzatban foglalt funkciókon túl – nem teszi lehetővé eltérő újrapróbálkozási szabályzatok hozzárendelését a különböző típusú hibákhoz vagy kivételekhez.</span><span class="sxs-lookup"><span data-stu-id="17915-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception beyond the functionality include in the retry policy.</span></span> <span data-ttu-id="17915-170">Ezért e sorok írásakor az a legcélszerűbb eljárás, ha a lehető legjobb általános teljesítményt és rendelkezésre állást biztosító szabályzatot konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="17915-170">Therefore, the best guidance available at the time of writing is to configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="17915-171">A szabályzat finomhangolásához elemezze a naplófájlokat, hogy megállapítsa, milyen típusú átmeneti hibák szoktak történni.</span><span class="sxs-lookup"><span data-stu-id="17915-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> <span data-ttu-id="17915-172">Amennyiben például a hibák nagy része kapcsolódik hálózati kapcsolati hibákhoz, érdemes azonnali újrapróbálkozást beállítani, és nem kell hosszú ideig várakozni az első újrapróbálkozásig.</span><span class="sxs-lookup"><span data-stu-id="17915-172">For example, if the majority of errors are related to network connectivity issues, you might attempt an immediate retry rather than wait a long time for the first retry.</span></span>
>
>

## <a name="azure-storage-retry-guidelines"></a><span data-ttu-id="17915-173">Az Azure Storage újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-173">Azure Storage retry guidelines</span></span>
<span data-ttu-id="17915-174">Az Azure Storage szolgáltatásai közé tartoznak a tábla- és blobtárolók, a fájl- és tárolási üzenetsorok.</span><span class="sxs-lookup"><span data-stu-id="17915-174">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-175">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-175">Retry mechanism</span></span>
<span data-ttu-id="17915-176">Újrapróbálkozásokra az egyes REST-műveletek szintjén kerül sor, és az ügyfél-API implementálásának szerves részeit képezik.</span><span class="sxs-lookup"><span data-stu-id="17915-176">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="17915-177">Az ügyfél Storage SDK-ja az [IExtendedRetryPolicy felületet](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx) implementáló osztályokat alkalmaz.</span><span class="sxs-lookup"><span data-stu-id="17915-177">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="17915-178">A felület több módon implementálható.</span><span class="sxs-lookup"><span data-stu-id="17915-178">There are different implementations of the interface.</span></span> <span data-ttu-id="17915-179">A Storage-ügyfelek kifejezetten táblákra, blobokra és üzenetsorokra szabott szabályzatok közül választhatnak.</span><span class="sxs-lookup"><span data-stu-id="17915-179">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="17915-180">Minden implementáció különböző újrapróbálkozási stratégiát használ, amely alapvetően az újrapróbálkozási időközöket és egyéb részleteket határoz meg.</span><span class="sxs-lookup"><span data-stu-id="17915-180">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="17915-181">A beépített osztályok támogatják a lineáris (állandó késleltetés) és az exponenciális (véletlenszerű újrapróbálkozási időközök) konfigurációkat.</span><span class="sxs-lookup"><span data-stu-id="17915-181">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="17915-182">Nincs szükség újrapróbálkozási szabályzat beállítására, ha egy másik folyamat magasabb szinten kezeli az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="17915-182">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="17915-183">Ugyanakkor implementálhat saját újrapróbálkozási osztályokat, ha a beépített osztályok nem felelnek meg valamilyen konkrét követelménynek.</span><span class="sxs-lookup"><span data-stu-id="17915-183">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="17915-184">A váltakozó újrapróbálkozás a tárolási szolgáltatás elsődleges és másodlagos helye között vált, ha írásvédett georedundáns tárolót (RA-GRS) használ, és a kérés eredménye egy újrapróbálható hiba.</span><span class="sxs-lookup"><span data-stu-id="17915-184">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="17915-185">További információt [az Azure Storage redundanciabeállításainál](http://msdn.microsoft.com/library/azure/dn727290.aspx) találhat.</span><span class="sxs-lookup"><span data-stu-id="17915-185">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="17915-186">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="17915-186">Policy configuration</span></span>
<span data-ttu-id="17915-187">Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik.</span><span class="sxs-lookup"><span data-stu-id="17915-187">Retry policies are configured programmatically.</span></span> <span data-ttu-id="17915-188">Megszokott eljárás egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** vagy **QueueRequestOptions** példány létrehozása és adatokkal történő feltöltése.</span><span class="sxs-lookup"><span data-stu-id="17915-188">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

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

<span data-ttu-id="17915-189">Beállíthat egy kérésbeállítási példányt az ügyfélen, és ezt követően az ügyfelet érintő műveletek az így megadott kérésbeállításokat fogják használni.</span><span class="sxs-lookup"><span data-stu-id="17915-189">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="17915-190">Az ügyfél kérésbeállításainak felülbírálásához a kérésbeállítás-osztály egy adatokkal feltöltött példányát kell megadni paraméterként a műveleti metódusoknál.</span><span class="sxs-lookup"><span data-stu-id="17915-190">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="17915-191">Az **OperationContext** példányban határozhatja meg, hogy milyen kódot kell végrehajtani újrapróbálkozás esetén, illetve ha befejeződik a művelet.</span><span class="sxs-lookup"><span data-stu-id="17915-191">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="17915-192">Ez a kód gyűjti össze a művelettel kapcsolatos adatokat, amelyek bekerülnek a naplókba és a telemetriába.</span><span class="sxs-lookup"><span data-stu-id="17915-192">This code can collect information about the operation for use in logs and telemetry.</span></span>

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

<span data-ttu-id="17915-193">A kiterjesztett újrapróbálkozási szabályzatok jelzik, hogy egy adott hiba után szükség van-e újrapróbálkozásra, ezenkívül egy **RetryContext** objektumot adnak vissza, amely megmutatja az újrapróbálkozások számát, a legutóbbi kérés eredményét, illetve hogy a következő újrapróbálkozás az elsődleges vagy a másodlagos helyen fog történni (a részleteket lásd az alábbi táblázatban).</span><span class="sxs-lookup"><span data-stu-id="17915-193">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="17915-194">A **RetryContext** objektummal lehet meghatározni, hogy történjen-e újrapróbálkozás, és ha igen, mikor.</span><span class="sxs-lookup"><span data-stu-id="17915-194">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="17915-195">További információért lásd az [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx) ismertetését.</span><span class="sxs-lookup"><span data-stu-id="17915-195">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="17915-196">A következő táblázatban a beépített újrapróbálkozási szabályzatok alapértelmezett beállításai tekinthetők meg.</span><span class="sxs-lookup"><span data-stu-id="17915-196">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="17915-197">**Kérésbeállítások**</span><span class="sxs-lookup"><span data-stu-id="17915-197">**Request options**</span></span>

| <span data-ttu-id="17915-198">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="17915-198">**Setting**</span></span> | <span data-ttu-id="17915-199">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="17915-199">**Default value**</span></span> | <span data-ttu-id="17915-200">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="17915-200">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="17915-201">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="17915-201">MaximumExecutionTime</span></span> | <span data-ttu-id="17915-202">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-202">120 seconds</span></span> | <span data-ttu-id="17915-203">A kérés maximális végrehajtási ideje az esetleges újrapróbálkozási kísérletekkel együtt.</span><span class="sxs-lookup"><span data-stu-id="17915-203">Maximum execution time for the request, including all potential retry attempts.</span></span> |
| <span data-ttu-id="17915-204">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="17915-204">ServerTimeout</span></span> | <span data-ttu-id="17915-205">None</span><span class="sxs-lookup"><span data-stu-id="17915-205">None</span></span> | <span data-ttu-id="17915-206">Kérés időtúllépési kerete a kiszolgálón (egész másodpercre kerekített érték).</span><span class="sxs-lookup"><span data-stu-id="17915-206">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="17915-207">Ha nincs megadva, a kiszolgálóhoz érkező összes kérés esetében az alapértelmezett értéket fogja használni.</span><span class="sxs-lookup"><span data-stu-id="17915-207">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="17915-208">Általában érdemes üresen hagyni ezt az értéket, hogy a kiszolgáló az alapértelmezett beállítást használja.</span><span class="sxs-lookup"><span data-stu-id="17915-208">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="17915-209">LocationMode</span><span class="sxs-lookup"><span data-stu-id="17915-209">LocationMode</span></span> | <span data-ttu-id="17915-210">None</span><span class="sxs-lookup"><span data-stu-id="17915-210">None</span></span> | <span data-ttu-id="17915-211">Ha a tárfiókot az írásvédett georedundáns tároló (RA-GRS) replikációs beállítással hozza létre, a hely módja határozza meg, hogy melyik hely kapja meg a kérést.</span><span class="sxs-lookup"><span data-stu-id="17915-211">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="17915-212">A **PrimaryThenSecondary** érték esetében például a kérések először mindig az elsődleges helyre érkeznek.</span><span class="sxs-lookup"><span data-stu-id="17915-212">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="17915-213">Ha a kérés sikertelen, ezután a másodlagos hely kapja meg azt.</span><span class="sxs-lookup"><span data-stu-id="17915-213">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="17915-214">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="17915-214">RetryPolicy</span></span> | <span data-ttu-id="17915-215">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="17915-215">ExponentialPolicy</span></span> | <span data-ttu-id="17915-216">Az egyes beállítások részleteit az alábbiakban tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="17915-216">See below for details of each option.</span></span> |

<span data-ttu-id="17915-217">**Exponenciális szabályzat**</span><span class="sxs-lookup"><span data-stu-id="17915-217">**Exponential policy**</span></span> 

| <span data-ttu-id="17915-218">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="17915-218">**Setting**</span></span> | <span data-ttu-id="17915-219">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="17915-219">**Default value**</span></span> | <span data-ttu-id="17915-220">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="17915-220">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="17915-221">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="17915-221">maxAttempt</span></span> | <span data-ttu-id="17915-222">3</span><span class="sxs-lookup"><span data-stu-id="17915-222">3</span></span> | <span data-ttu-id="17915-223">Az újrapróbálkozási kísérletek száma.</span><span class="sxs-lookup"><span data-stu-id="17915-223">Number of retry attempts.</span></span> |
| <span data-ttu-id="17915-224">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-224">deltaBackoff</span></span> | <span data-ttu-id="17915-225">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-225">4 seconds</span></span> | <span data-ttu-id="17915-226">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="17915-226">Back-off interval between retries.</span></span> <span data-ttu-id="17915-227">Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét (egy véletlenszerű elemet belevéve) fogja használni a rendszer.</span><span class="sxs-lookup"><span data-stu-id="17915-227">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="17915-228">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-228">MinBackoff</span></span> | <span data-ttu-id="17915-229">3 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-229">3 seconds</span></span> | <span data-ttu-id="17915-230">Ez az érték hozzáadódik a deltaBackoff alapján kiszámított minden újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="17915-230">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="17915-231">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="17915-231">This value cannot be changed.</span></span>
| <span data-ttu-id="17915-232">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-232">MaxBackoff</span></span> | <span data-ttu-id="17915-233">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-233">120 seconds</span></span> | <span data-ttu-id="17915-234">A MaxBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke.</span><span class="sxs-lookup"><span data-stu-id="17915-234">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="17915-235">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="17915-235">This value cannot be changed.</span></span> |

<span data-ttu-id="17915-236">**Lineáris szabályzat**</span><span class="sxs-lookup"><span data-stu-id="17915-236">**Linear policy**</span></span>

| <span data-ttu-id="17915-237">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="17915-237">**Setting**</span></span> | <span data-ttu-id="17915-238">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="17915-238">**Default value**</span></span> | <span data-ttu-id="17915-239">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="17915-239">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="17915-240">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="17915-240">maxAttempt</span></span> | <span data-ttu-id="17915-241">3</span><span class="sxs-lookup"><span data-stu-id="17915-241">3</span></span> | <span data-ttu-id="17915-242">Az újrapróbálkozási kísérletek száma.</span><span class="sxs-lookup"><span data-stu-id="17915-242">Number of retry attempts.</span></span> |
| <span data-ttu-id="17915-243">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-243">deltaBackoff</span></span> | <span data-ttu-id="17915-244">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-244">30 seconds</span></span> | <span data-ttu-id="17915-245">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="17915-245">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="17915-246">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="17915-246">Retry usage guidance</span></span>
<span data-ttu-id="17915-247">Ügyeljen a következőkre, amikor a Storage-ügyfél API-ján keresztül próbálja elérni az Azure Storage-szolgáltatásokat:</span><span class="sxs-lookup"><span data-stu-id="17915-247">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="17915-248">Használja a Microsoft.WindowsAzure.Storage.RetryPolicies névtérben található beépített újrapróbálkozási szabályzatokat, amennyiben azok megfelelnek az elvárásainak.</span><span class="sxs-lookup"><span data-stu-id="17915-248">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="17915-249">A legtöbb esetben ezek a szabályzatok is elégségesek.</span><span class="sxs-lookup"><span data-stu-id="17915-249">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="17915-250">Az **ExponentialRetry** szabályzatot kötegelt műveletek, háttérfeladatok vagy nem interaktív forgatókönyvek esetében használja.</span><span class="sxs-lookup"><span data-stu-id="17915-250">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="17915-251">Ezekben az esetekben általában több idő áll rendelkezésre megvárni, amíg a szolgáltatás helyreáll, ami növeli a művelet sikeres teljesítésének esélyét.</span><span class="sxs-lookup"><span data-stu-id="17915-251">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="17915-252">Érdemes megadni a **RequestOptions** paraméter **MaximumExecutionTime** tulajdonságát a teljes végrehajtási idő korlátozásához, de az időtúllépési érték megadásakor vegye figyelembe a művelet típusát és méretét.</span><span class="sxs-lookup"><span data-stu-id="17915-252">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="17915-253">Ha egyéni újrapróbálkozást implementál, ne hozzon létre burkolókat a Storage-ügyfél osztályai körül.</span><span class="sxs-lookup"><span data-stu-id="17915-253">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="17915-254">Inkább meglévő szabályzatokat bővítsen ki az **IExtendedRetryPolicy** felületen keresztül.</span><span class="sxs-lookup"><span data-stu-id="17915-254">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="17915-255">Amennyiben írásvédett georedundáns tárolót (RA-GRS) használ, a **LocationMode** segítségével beállíthatja, hogy az elsődleges hozzáférés sikertelensége esetén az újrapróbálkozási kísérletek a tároló másodlagos írásvédett példányához férjenek hozzá.</span><span class="sxs-lookup"><span data-stu-id="17915-255">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="17915-256">Ebben az esetben azonban meg kell győződnie arról, hogy az alkalmazás esetlegesen elavult adatokkal is képes sikeresen működni (ha az elsődleges tároló replikálása még nem fejeződött be).</span><span class="sxs-lookup"><span data-stu-id="17915-256">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="17915-257">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="17915-257">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="17915-258">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="17915-258">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="17915-259">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="17915-259">**Context**</span></span> | <span data-ttu-id="17915-260">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="17915-260">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="17915-261">**Újrapróbálkozási szabályzat**</span><span class="sxs-lookup"><span data-stu-id="17915-261">**Retry policy**</span></span> | <span data-ttu-id="17915-262">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="17915-262">**Settings**</span></span> | <span data-ttu-id="17915-263">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="17915-263">**Values**</span></span> | <span data-ttu-id="17915-264">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="17915-264">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="17915-265">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="17915-265">Interactive, UI,</span></span><br /><span data-ttu-id="17915-266">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="17915-266">or foreground</span></span> |<span data-ttu-id="17915-267">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-267">2 seconds</span></span> |<span data-ttu-id="17915-268">Lineáris</span><span class="sxs-lookup"><span data-stu-id="17915-268">Linear</span></span> |<span data-ttu-id="17915-269">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="17915-269">maxAttempt</span></span><br /><span data-ttu-id="17915-270">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-270">deltaBackoff</span></span> |<span data-ttu-id="17915-271">3</span><span class="sxs-lookup"><span data-stu-id="17915-271">3</span></span><br /><span data-ttu-id="17915-272">500 ms</span><span class="sxs-lookup"><span data-stu-id="17915-272">500 ms</span></span> |<span data-ttu-id="17915-273">1. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-273">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="17915-274">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-274">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="17915-275">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-275">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="17915-276">Háttér</span><span class="sxs-lookup"><span data-stu-id="17915-276">Background</span></span><br /><span data-ttu-id="17915-277">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="17915-277">or batch</span></span> |<span data-ttu-id="17915-278">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-278">30 seconds</span></span> |<span data-ttu-id="17915-279">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-279">Exponential</span></span> |<span data-ttu-id="17915-280">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="17915-280">maxAttempt</span></span><br /><span data-ttu-id="17915-281">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-281">deltaBackoff</span></span> |<span data-ttu-id="17915-282">5</span><span class="sxs-lookup"><span data-stu-id="17915-282">5</span></span><br /><span data-ttu-id="17915-283">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-283">4 seconds</span></span> |<span data-ttu-id="17915-284">1. kísérlet – kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-284">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="17915-285">2. kísérlet – kb. 7 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-285">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="17915-286">3. kísérlet – kb. 15 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-286">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="17915-287">Telemetria</span><span class="sxs-lookup"><span data-stu-id="17915-287">Telemetry</span></span>
<span data-ttu-id="17915-288">Az újrapróbálkozási kísérletek a **TraceSource** osztályban lesznek naplózva.</span><span class="sxs-lookup"><span data-stu-id="17915-288">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="17915-289">Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="17915-289">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="17915-290">Használja a **TextWriterTraceListener** vagy az **XmlWriterTraceListener** osztályt az adatok a naplófájlba írásához. Az **EventLogTraceListener** osztállyal a Windows eseménynaplóba írhat, míg az **EventProviderTraceListener** osztállyal a nyomkövetési adatokat rögzítheti az ETW alrendszerben.</span><span class="sxs-lookup"><span data-stu-id="17915-290">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="17915-291">Beállíthatja, hogy a puffer ürítése automatikusan megtörténjen, valamint a naplózott események részletességét is (például: Hiba, Figyelmeztetés, Tájékoztató és Részletes).</span><span class="sxs-lookup"><span data-stu-id="17915-291">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="17915-292">További információt a [.NET-es Storage-ügyfélkódtárral történő ügyféloldali naplózást](http://msdn.microsoft.com/library/azure/dn782839.aspx) ismertető szakaszban találhat.</span><span class="sxs-lookup"><span data-stu-id="17915-292">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="17915-293">A műveletek kaphatnak egy **OperationContext**-példányt, amely közzétesz egy **Retrying** eseményt, amelyhez egyéni telemetrialogika csatolható.</span><span class="sxs-lookup"><span data-stu-id="17915-293">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="17915-294">További információ: [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="17915-294">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="17915-295">Példák</span><span class="sxs-lookup"><span data-stu-id="17915-295">Examples</span></span>
<span data-ttu-id="17915-296">A következő mintakód azt mutatja be, hogyan lehet két **TableRequestOptions**-példányt létrehozni eltérő újrapróbálkozási beállításokkal. Az egyik az interaktív, a másik pedig a háttérkérések kezelésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="17915-296">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="17915-297">A példa ezután az ügyfélre alkalmazza ezt a két újrapróbálkozási szabályzatot, így azok minden kérésre vonatkozni fognak, ezenkívül az interaktív stratégiát egy adott kéréshez köti, így az felülbírálja az ügyfél alapértelmezett beállításait.</span><span class="sxs-lookup"><span data-stu-id="17915-297">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="17915-298">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-298">More information</span></span>
* [<span data-ttu-id="17915-299">Az Azure Storage ügyféloldali kódtárhoz javasolt újrapróbálkozási szabályzatok</span><span class="sxs-lookup"><span data-stu-id="17915-299">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="17915-300">Storage ügyféloldali kódtár 2.0 – újrapróbálkozási szabályzatok implementálása</span><span class="sxs-lookup"><span data-stu-id="17915-300">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a><span data-ttu-id="17915-301">Az Entity Framework 6-ot használó SQL Database újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-301">SQL Database using Entity Framework 6 retry guidelines</span></span>
<span data-ttu-id="17915-302">Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.</span><span class="sxs-lookup"><span data-stu-id="17915-302">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="17915-303">Az Entity Framework egy objektumrelációs leképező .NET-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet relációs adatokkal dolgozni.</span><span class="sxs-lookup"><span data-stu-id="17915-303">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="17915-304">Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.</span><span class="sxs-lookup"><span data-stu-id="17915-304">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-305">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-305">Retry mechanism</span></span>
<span data-ttu-id="17915-306">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el, a [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmus segítségével.</span><span class="sxs-lookup"><span data-stu-id="17915-306">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="17915-307">Az újrapróbálkozási mechanizmus fő funkciói a következők:</span><span class="sxs-lookup"><span data-stu-id="17915-307">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="17915-308">Az elsődleges absztrakció az **IDbExecutionStrategy** felület.</span><span class="sxs-lookup"><span data-stu-id="17915-308">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="17915-309">Ez a felület:</span><span class="sxs-lookup"><span data-stu-id="17915-309">This interface:</span></span>
  * <span data-ttu-id="17915-310">Definiálja a szinkron és aszinkron **feldolgozási**\* metódusokat.</span><span class="sxs-lookup"><span data-stu-id="17915-310">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="17915-311">Definiálja azokat az osztályokat, amelyek felhasználhatók közvetlenül, illetve konfigurálhatók alapértelmezett stratégiaként egy adatbázis-környezetben, leképezhetők egy szolgáltatónévre, vagy pedig leképezhetők egy szolgáltatónévre vagy kiszolgálónévre.</span><span class="sxs-lookup"><span data-stu-id="17915-311">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="17915-312">Ha egy környezethez konfigurálják, az újrapróbálkozások az egyes adatbázis-műveletek szintjén történnek, amelyekből több is lehet egy adott környezeti művelet esetében.</span><span class="sxs-lookup"><span data-stu-id="17915-312">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="17915-313">Definiálja, hogy a sikertelen csatlakozást mikor kövesse újrapróbálkozás, és hogyan.</span><span class="sxs-lookup"><span data-stu-id="17915-313">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="17915-314">Az **IDbExecutionStrategy** felület számos beépített implementálását tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="17915-314">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="17915-315">Alapértelmezett – nincs újrapróbálkozás.</span><span class="sxs-lookup"><span data-stu-id="17915-315">Default - no retrying.</span></span>
  * <span data-ttu-id="17915-316">Az SQL Database alapértelmezett beállítása (automatikus) – nincs újrapróbálkozás, de megvizsgálja a kivételeket, és beburkolja azokat, az SQL Database stratégia használatát javasolva.</span><span class="sxs-lookup"><span data-stu-id="17915-316">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="17915-317">Az SQL Database alapértelmezett beállítása – exponenciális (az alaposztálytól örökölt), plusz az SQL Database észlelési logikája.</span><span class="sxs-lookup"><span data-stu-id="17915-317">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="17915-318">Véletlenszerűsítést tartalmazó exponenciális visszatartási stratégiát implementál.</span><span class="sxs-lookup"><span data-stu-id="17915-318">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="17915-319">A beépített újrapróbálkozási osztályok állapotalapúak, és nem alkalmasak a többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="17915-319">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="17915-320">Ugyanakkor az aktuális művelet befejezése után újra felhasználhatók.</span><span class="sxs-lookup"><span data-stu-id="17915-320">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="17915-321">Amennyiben az újrapróbálkozások száma meghaladja a megadott értéket, a szolgáltatás új kivételbe burkolja az eredményeket.</span><span class="sxs-lookup"><span data-stu-id="17915-321">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="17915-322">Nem rendezi buborékba az aktuális kivételt.</span><span class="sxs-lookup"><span data-stu-id="17915-322">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="17915-323">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="17915-323">Policy configuration</span></span>
<span data-ttu-id="17915-324">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el.</span><span class="sxs-lookup"><span data-stu-id="17915-324">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="17915-325">Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik.</span><span class="sxs-lookup"><span data-stu-id="17915-325">Retry policies are configured programmatically.</span></span> <span data-ttu-id="17915-326">A konfiguráció nem módosítható az egyes műveletek szintjén.</span><span class="sxs-lookup"><span data-stu-id="17915-326">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="17915-327">Ha egy környezetfüggő stratégiát tesz alapértelmezetté, megad egy függvényt, amely igény szerint hoz létre egy új stratégiát.</span><span class="sxs-lookup"><span data-stu-id="17915-327">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="17915-328">A következő kód azt mutatja be, hogyan hozható létre egy újrapróbálkozási konfigurációs osztályt, amely kibővíti a **DbConfiguration** alaposztályt.</span><span class="sxs-lookup"><span data-stu-id="17915-328">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="17915-329">Az alkalmazás indításakor, a **DbConfiguration**-példány **SetConfiguration** metódusával teheti az alapértelmezett újrapróbálkozási stratégiává minden művelet számára.</span><span class="sxs-lookup"><span data-stu-id="17915-329">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="17915-330">Alapértelmezés szerint az EF automatikusan észleli és használatba veszi a konfigurációs osztályt.</span><span class="sxs-lookup"><span data-stu-id="17915-330">By default, EF will automatically discover and use the configuration class.</span></span>

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

<span data-ttu-id="17915-331">Ha meg kíván adni egy újrapróbálkozási konfigurációs osztályt a környezethez, lássa el a környezeti osztályt egy **DbConfigurationType** attribútummal.</span><span class="sxs-lookup"><span data-stu-id="17915-331">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="17915-332">Amennyiben csak egy konfigurációs osztálya van, az EF azt fogja használni, nem szükséges külön jeleznie ezt.</span><span class="sxs-lookup"><span data-stu-id="17915-332">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

<span data-ttu-id="17915-333">Ha eltérő újrapróbálkozási stratégiára van szüksége adott műveletek esetében, vagy le kívánja tiltani az újrapróbálkozásokat egyes műveleteknél, létrehozhat egy konfigurációs osztályt, amely lehetővé teszi stratégiák felfüggesztését vagy cseréjét egy jelző a **CallContext** osztályban történő elhelyezésével.</span><span class="sxs-lookup"><span data-stu-id="17915-333">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="17915-334">A konfigurációs osztály ezt a jelzőt használva vált stratégiát vagy tiltja le a felhasználó által megadott stratégiát, majd az alapértelmezett stratégiát használja.</span><span class="sxs-lookup"><span data-stu-id="17915-334">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="17915-335">További információt a végrehajtási stratégiák újrapróbálkozásainak korlátozásait ismertető oldal [végrehajtási stratégia felfüggesztését](http://msdn.microsoft.com/dn307226#transactions_workarounds) ismertető szakaszában talál (EF6-os vagy újabb verzió).</span><span class="sxs-lookup"><span data-stu-id="17915-335">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="17915-336">Bizonyos műveletek esetében egy másik lehetőség adott újrapróbálkozási stratégia használatára, ha létrehozza a kívánt stratégiaosztály egy példányát, és paramétereket használva ellátja a kívánt beállításokkal.</span><span class="sxs-lookup"><span data-stu-id="17915-336">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="17915-337">Ezután meghívja annak **ExecuteAsync** metódusát.</span><span class="sxs-lookup"><span data-stu-id="17915-337">You then invoke its **ExecuteAsync** method.</span></span>

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

<span data-ttu-id="17915-338">A **DbConfiguration** osztály használatának legegyszerűbb módja, ha megkeresi ugyanazt a szerelvényt, amely a **DbContext** osztályban szerepel.</span><span class="sxs-lookup"><span data-stu-id="17915-338">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="17915-339">Ez azonban nem alkalmazható, ha különböző forgatókönyvekben van szükség ugyanarra a környezetre, például a különböző interaktív és háttérbeli újrapróbálkozási stratégiák esetében.</span><span class="sxs-lookup"><span data-stu-id="17915-339">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="17915-340">Ha a különböző környezetek különálló alkalmazástartományokban futnak, igénybe veheti a konfigurációs osztályok a konfigurációs fájlban történő megadásának beépített lehetőségét, illetve beállíthatja azt közvetlenül is a kódban.</span><span class="sxs-lookup"><span data-stu-id="17915-340">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="17915-341">Ha a különböző környezeteknek ugyanabban az alkalmazástartományban kell futniuk, egyéni megoldásra lesz szükség.</span><span class="sxs-lookup"><span data-stu-id="17915-341">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="17915-342">További információt a [kódalapú konfigurálást (EF6-os vagy újabb verzió)](http://msdn.microsoft.com/data/jj680699.aspx) ismertető szakaszban találhat.</span><span class="sxs-lookup"><span data-stu-id="17915-342">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="17915-343">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók az EF6 használatakor.</span><span class="sxs-lookup"><span data-stu-id="17915-343">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="17915-344">Beállítás</span><span class="sxs-lookup"><span data-stu-id="17915-344">Setting</span></span> | <span data-ttu-id="17915-345">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="17915-345">Default value</span></span> | <span data-ttu-id="17915-346">Jelentés</span><span class="sxs-lookup"><span data-stu-id="17915-346">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="17915-347">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="17915-347">Policy</span></span> | <span data-ttu-id="17915-348">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-348">Exponential</span></span> | <span data-ttu-id="17915-349">Exponenciális visszatartás.</span><span class="sxs-lookup"><span data-stu-id="17915-349">Exponential back-off.</span></span> |
| <span data-ttu-id="17915-350">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="17915-350">MaxRetryCount</span></span> | <span data-ttu-id="17915-351">5</span><span class="sxs-lookup"><span data-stu-id="17915-351">5</span></span> | <span data-ttu-id="17915-352">Az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="17915-352">The maximum number of retries.</span></span> |
| <span data-ttu-id="17915-353">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="17915-353">MaxDelay</span></span> | <span data-ttu-id="17915-354">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-354">30 seconds</span></span> | <span data-ttu-id="17915-355">Az újrapróbálkozások közötti maximális késleltetés.</span><span class="sxs-lookup"><span data-stu-id="17915-355">The maximum delay between retries.</span></span> <span data-ttu-id="17915-356">Az érték nem befolyásolja az egymást követő késleltetések kiszámítását.</span><span class="sxs-lookup"><span data-stu-id="17915-356">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="17915-357">Csak a felső határt adja meg.</span><span class="sxs-lookup"><span data-stu-id="17915-357">It only defines an upper bound.</span></span> |
| <span data-ttu-id="17915-358">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="17915-358">DefaultCoefficient</span></span> | <span data-ttu-id="17915-359">1 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-359">1 second</span></span> | <span data-ttu-id="17915-360">Az exponenciális visszatartás számításának együtthatója.</span><span class="sxs-lookup"><span data-stu-id="17915-360">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="17915-361">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="17915-361">This value cannot be changed.</span></span> |
| <span data-ttu-id="17915-362">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="17915-362">DefaultRandomFactor</span></span> | <span data-ttu-id="17915-363">1.1</span><span class="sxs-lookup"><span data-stu-id="17915-363">1.1</span></span> | <span data-ttu-id="17915-364">A véletlenszerű késleltetés bejegyzésekhez való hozzáadására szolgáló szorzó.</span><span class="sxs-lookup"><span data-stu-id="17915-364">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="17915-365">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="17915-365">This value cannot be changed.</span></span> |
| <span data-ttu-id="17915-366">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="17915-366">DefaultExponentialBase</span></span> | <span data-ttu-id="17915-367">2</span><span class="sxs-lookup"><span data-stu-id="17915-367">2</span></span> | <span data-ttu-id="17915-368">A következő késleltetés kiszámítására szolgáló szorzó.</span><span class="sxs-lookup"><span data-stu-id="17915-368">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="17915-369">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="17915-369">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="17915-370">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="17915-370">Retry usage guidance</span></span>
<span data-ttu-id="17915-371">Ügyeljen a következőkre, amikor az EF6 használatával éri el az SQL Database-t:</span><span class="sxs-lookup"><span data-stu-id="17915-371">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="17915-372">Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="17915-372">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="17915-373">A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják.</span><span class="sxs-lookup"><span data-stu-id="17915-373">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="17915-374">Ha kiszámítható teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.</span><span class="sxs-lookup"><span data-stu-id="17915-374">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="17915-375">Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását.</span><span class="sxs-lookup"><span data-stu-id="17915-375">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="17915-376">Használjon inkább egy exponenciális visszatartási stratégiát, mivel a szolgáltatás túlterhelődhet, és a hosszabb késleltetés több időt ad a helyreállításra.</span><span class="sxs-lookup"><span data-stu-id="17915-376">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="17915-377">A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez.</span><span class="sxs-lookup"><span data-stu-id="17915-377">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="17915-378">Az időtúllépés ideális értékét saját üzleti logikájának felépítése és tesztek alapján állapíthatja meg.</span><span class="sxs-lookup"><span data-stu-id="17915-378">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="17915-379">A későbbiekben szükség lehet az érték módosítására, ahogy változik az adatmennyiség, vagy változnak az üzleti folyamatok.</span><span class="sxs-lookup"><span data-stu-id="17915-379">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="17915-380">A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt.</span><span class="sxs-lookup"><span data-stu-id="17915-380">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="17915-381">A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást.</span><span class="sxs-lookup"><span data-stu-id="17915-381">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="17915-382">Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Bár nehéz előre megállapítani, hogy hány parancsot kell végrehajtani a környezet mentésekor.</span><span class="sxs-lookup"><span data-stu-id="17915-382">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="17915-383">Az alapértelmezett időtúllépést a **DbContext** példány **CommandTimeout** tulajdonságában adhatja meg.</span><span class="sxs-lookup"><span data-stu-id="17915-383">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="17915-384">Az Entity Framework támogatja a konfigurációs fájlokban definiált újrapróbálkozási konfigurációk használatát.</span><span class="sxs-lookup"><span data-stu-id="17915-384">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="17915-385">Ugyanakkor a minél nagyobb rugalmasság érdekében azt javasoljuk, hogy a konfigurációt szoftveresen hozza létre az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="17915-385">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="17915-386">Az újrapróbálkozási szabályzatok konkrét paraméterei – például az újrapróbálkozások száma és az újrapróbálkozási időközök – a szolgáltatás konfigurációs fájljában tárolhatók, és a futtatás során felhasználhatók a megfelelő szabályzatok létrehozására.</span><span class="sxs-lookup"><span data-stu-id="17915-386">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="17915-387">Így a beállítások az alkalmazás újraindítása nélkül módosíthatók.</span><span class="sxs-lookup"><span data-stu-id="17915-387">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="17915-388">A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="17915-388">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="17915-389">Az újrapróbálkozási kísérletek közötti késleltetés nem adható meg (rögzített, és egy exponenciális sorozat részeként jön létre).</span><span class="sxs-lookup"><span data-stu-id="17915-389">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="17915-390">Ha nem hoz létre egyéni újrapróbálkozási stratégiát, csak a maximális értékeket adhatja meg, az itt látható módon.</span><span class="sxs-lookup"><span data-stu-id="17915-390">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="17915-391">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="17915-391">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="17915-392">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="17915-392">**Context**</span></span> | <span data-ttu-id="17915-393">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="17915-393">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="17915-394">**Újrapróbálkozási szabályzat**</span><span class="sxs-lookup"><span data-stu-id="17915-394">**Retry policy**</span></span> | <span data-ttu-id="17915-395">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="17915-395">**Settings**</span></span> | <span data-ttu-id="17915-396">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="17915-396">**Values**</span></span> | <span data-ttu-id="17915-397">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="17915-397">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="17915-398">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="17915-398">Interactive, UI,</span></span><br /><span data-ttu-id="17915-399">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="17915-399">or foreground</span></span> |<span data-ttu-id="17915-400">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-400">2 seconds</span></span> |<span data-ttu-id="17915-401">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-401">Exponential</span></span> |<span data-ttu-id="17915-402">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="17915-402">MaxRetryCount</span></span><br /><span data-ttu-id="17915-403">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="17915-403">MaxDelay</span></span> |<span data-ttu-id="17915-404">3</span><span class="sxs-lookup"><span data-stu-id="17915-404">3</span></span><br /><span data-ttu-id="17915-405">750 ms</span><span class="sxs-lookup"><span data-stu-id="17915-405">750 ms</span></span> |<span data-ttu-id="17915-406">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-406">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="17915-407">2. kísérlet – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-407">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="17915-408">3. kísérlet – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-408">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="17915-409">Háttér</span><span class="sxs-lookup"><span data-stu-id="17915-409">Background</span></span><br /> <span data-ttu-id="17915-410">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="17915-410">or batch</span></span> |<span data-ttu-id="17915-411">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-411">30 seconds</span></span> |<span data-ttu-id="17915-412">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-412">Exponential</span></span> |<span data-ttu-id="17915-413">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="17915-413">MaxRetryCount</span></span><br /><span data-ttu-id="17915-414">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="17915-414">MaxDelay</span></span> |<span data-ttu-id="17915-415">5</span><span class="sxs-lookup"><span data-stu-id="17915-415">5</span></span><br /><span data-ttu-id="17915-416">12 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-416">12 seconds</span></span> |<span data-ttu-id="17915-417">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-417">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="17915-418">2. kísérlet – kb. 1 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-418">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="17915-419">3. kísérlet – kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-419">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="17915-420">4. kísérlet – kb. 7 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-420">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="17915-421">5. kísérlet – 12 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-421">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="17915-422">A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz.</span><span class="sxs-lookup"><span data-stu-id="17915-422">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="17915-423">Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.</span><span class="sxs-lookup"><span data-stu-id="17915-423">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="17915-424">Példák</span><span class="sxs-lookup"><span data-stu-id="17915-424">Examples</span></span>
<span data-ttu-id="17915-425">A következő mintakód egy egyszerű adatelérési megoldást definiál, amely az Entity Frameworköt használja.</span><span class="sxs-lookup"><span data-stu-id="17915-425">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="17915-426">Egy adott újrapróbálkozási stratégiát állít be a **BlogConfiguration** osztály egy példányának definiálásával, amely a **DbConfiguration** osztályt bővíti ki.</span><span class="sxs-lookup"><span data-stu-id="17915-426">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="17915-427">A [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmussal foglalkozó témakörben további példákat talál az Entity Framework újrapróbálkozási mechanizmusának használatára.</span><span class="sxs-lookup"><span data-stu-id="17915-427">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="17915-428">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-428">More information</span></span>
* [<span data-ttu-id="17915-429">Az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatója</span><span class="sxs-lookup"><span data-stu-id="17915-429">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a><span data-ttu-id="17915-430">Az Entity Framework Core-t használó SQL Database újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-430">SQL Database using Entity Framework Core retry guidelines</span></span>
<span data-ttu-id="17915-431">Az [Entity Framework Core](/ef/core/) egy objektumrelációs leképező .NET Core-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet adatokkal dolgozni.</span><span class="sxs-lookup"><span data-stu-id="17915-431">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="17915-432">Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.</span><span class="sxs-lookup"><span data-stu-id="17915-432">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="17915-433">Az Entity Framework e verzióját az alapoktól építettük újra, ezért nem örökli meg automatikusan az EF6.x összes funkcióját.</span><span class="sxs-lookup"><span data-stu-id="17915-433">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-434">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-434">Retry mechanism</span></span>
<span data-ttu-id="17915-435">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework Core-ral érik el, a [kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency) mechanizmus segítségével.</span><span class="sxs-lookup"><span data-stu-id="17915-435">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="17915-436">A kapcsolat rugalmassága először az EF Core 1.1.0-ban vált elérhetővé.</span><span class="sxs-lookup"><span data-stu-id="17915-436">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="17915-437">Az elsődleges absztrakció az `IExecutionStrategy` felület.</span><span class="sxs-lookup"><span data-stu-id="17915-437">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="17915-438">Az SQL Server, és így az SQL Azure végrehajtási stratégiája felismeri a kivételtípusokat, amelyeknél elérhető az újrapróbálkozás, és észszerű alapértelmezett értékek vannak beállítva többek között az újrapróbálkozások maximális számánál és az újrapróbálkozások közötti késleltetésnél.</span><span class="sxs-lookup"><span data-stu-id="17915-438">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="17915-439">Példák</span><span class="sxs-lookup"><span data-stu-id="17915-439">Examples</span></span>

<span data-ttu-id="17915-440">A következő kód lehetővé teszi az automatikus újrapróbálkozást a DbContext objektum konfigurálásakor, ami egy munkamenetet jelöl az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="17915-440">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="17915-441">A következő kód bemutatja, egy végrehajtási stratégia alkalmazásával hogyan hajtható végre egy tranzakció automatikus újrapróbálkozással.</span><span class="sxs-lookup"><span data-stu-id="17915-441">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="17915-442">A tranzakciót egy delegált definiálja.</span><span class="sxs-lookup"><span data-stu-id="17915-442">The transaction is defined in a delegate.</span></span> <span data-ttu-id="17915-443">Átmeneti hiba előfordulásakor a végrehajtási stratégia ismét meghívja a delegáltat.</span><span class="sxs-lookup"><span data-stu-id="17915-443">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="17915-444">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-444">More information</span></span>
* [<span data-ttu-id="17915-445">Kapcsolat rugalmassága</span><span class="sxs-lookup"><span data-stu-id="17915-445">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="17915-446">Adatpontok – EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="17915-446">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a><span data-ttu-id="17915-447">Az ADO.NET-et használó SQL Database újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-447">SQL Database using ADO.NET retry guidelines</span></span>
<span data-ttu-id="17915-448">Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.</span><span class="sxs-lookup"><span data-stu-id="17915-448">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-449">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-449">Retry mechanism</span></span>
<span data-ttu-id="17915-450">Az SQL Database nem tartalmaz beépített támogatást az újrapróbálkozásokhoz, ha az ADO.NET használatával érik el.</span><span class="sxs-lookup"><span data-stu-id="17915-450">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="17915-451">Ugyanakkor a kérések válaszkódjából megállapítható, hogy a kérés miért hiúsult meg.</span><span class="sxs-lookup"><span data-stu-id="17915-451">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="17915-452">További információt az SQL Database szabályozásáról [az Azure SQL Database erőforrás-korlátait](/azure/sql-database/sql-database-resource-limits) ismertető szakaszban talál.</span><span class="sxs-lookup"><span data-stu-id="17915-452">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="17915-453">A kapcsolódó hibakódok listáját [az SQL Database ügyfélalkalmazásaiban felmerülő SQL-hibakódokat](/azure/sql-database/sql-database-develop-error-messages) ismertető szakaszban találja.</span><span class="sxs-lookup"><span data-stu-id="17915-453">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="17915-454">A Polly kódtárt alkalmazva implementálhatja az újrapróbálkozást az SQL Database-ben.</span><span class="sxs-lookup"><span data-stu-id="17915-454">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="17915-455">További információt az [átmeneti hibák a Polly használatával történő kezelését](#transient-fault-handling-with-polly) ismertető szakaszban talál.</span><span class="sxs-lookup"><span data-stu-id="17915-455">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="17915-456">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="17915-456">Retry usage guidance</span></span>
<span data-ttu-id="17915-457">Ügyeljen a következőkre, amikor az ADO.NET használatával éri el az SQL Database-t:</span><span class="sxs-lookup"><span data-stu-id="17915-457">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="17915-458">Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="17915-458">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="17915-459">A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják.</span><span class="sxs-lookup"><span data-stu-id="17915-459">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="17915-460">Ha kiszámíthatóbb teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.</span><span class="sxs-lookup"><span data-stu-id="17915-460">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="17915-461">Gondoskodjon arról, hogy az újrapróbálkozás a megfelelő szinten vagy hatókörrel legyen végrehajtva, amivel elkerülheti, hogy a nem idempotens műveletek miatt inkonzisztencia keletkezzen az adatokban.</span><span class="sxs-lookup"><span data-stu-id="17915-461">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="17915-462">Ideális esetben minden műveletnek idempotensnek kellene lennie, hogy inkonzisztencia veszélye nélkül lehessen ismételni azokat.</span><span class="sxs-lookup"><span data-stu-id="17915-462">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="17915-463">Ellenkező esetben az újrapróbálkozást olyan szinten vagy hatókörrel kell végrehajtani, hogy az összes kapcsolódó változtatást vissza lehessen vonni egy művelet meghiúsulásakor – például egy tranzakciós hatókörben.</span><span class="sxs-lookup"><span data-stu-id="17915-463">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="17915-464">További információt a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee) foglalkozó részében talál.</span><span class="sxs-lookup"><span data-stu-id="17915-464">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="17915-465">Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását. Az interaktív forgatókönyvek kivételt képeznek, mivel csak néhány újrapróbálkozás történik nagyon rövid időközökkel.</span><span class="sxs-lookup"><span data-stu-id="17915-465">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="17915-466">Ehelyett a legtöbb esetben exponenciális visszatartási stratégia használata javasolt.</span><span class="sxs-lookup"><span data-stu-id="17915-466">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="17915-467">A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez.</span><span class="sxs-lookup"><span data-stu-id="17915-467">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="17915-468">A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt.</span><span class="sxs-lookup"><span data-stu-id="17915-468">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="17915-469">A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást.</span><span class="sxs-lookup"><span data-stu-id="17915-469">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="17915-470">Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Gyakorlatilag hozzáadódik az újrapróbálkozási szabályzatban megadott újrapróbálkozási késleltetéshez minden egyes újrapróbálkozási kísérlet esetén.</span><span class="sxs-lookup"><span data-stu-id="17915-470">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="17915-471">Megszakítja a kapcsolatot adott számú újrapróbálkozás után, akár egy exponenciális visszatartási újrapróbálkozási logika használata esetén is, és egy új kapcsolatot létesítve próbálja meg újra végrehajtani a műveletet.</span><span class="sxs-lookup"><span data-stu-id="17915-471">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="17915-472">Ha ugyanazzal a művelettel többször próbálkozik újra ugyanazon a kapcsolaton keresztül, az önmagában is csatlakozási problémákat okozhat.</span><span class="sxs-lookup"><span data-stu-id="17915-472">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="17915-473">Erre a technikára egy példát a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx) foglalkozó részében találhat.</span><span class="sxs-lookup"><span data-stu-id="17915-473">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="17915-474">Ha kapcsolatkészletezést alkalmaz (ez az alapértelmezett beállítás), akkor előfordulhat, hogy a készletből ismét ugyanazt a kapcsolatot választja a rendszer, akár a kapcsolat lezárása és ismételt megnyitása után is.</span><span class="sxs-lookup"><span data-stu-id="17915-474">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="17915-475">Ebben az esetben az **SqlConnection** osztályból kell meghívni a **ClearPool** metódust, és jelezni, hogy a kapcsolat nem újrahasználható.</span><span class="sxs-lookup"><span data-stu-id="17915-475">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="17915-476">Ezt azonban csak abban az esetben javasoljuk, ha számos csatlakozási kísérlet meghiúsult, és csak akkor, ha az átmeneti hibák, például SQL-időtúllépések (-2-es hibakód) adott osztálya hibás kapcsolatokhoz kötődik.</span><span class="sxs-lookup"><span data-stu-id="17915-476">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="17915-477">Amennyiben az adatelérési kód **TransactionScope**-példányként kezdeményezett tranzakciókat alkalmaz, az újrapróbálkozási logikának újra meg kell nyitnia a kapcsolatot, és új tranzakció-hatókört kell kezdeményeznie.</span><span class="sxs-lookup"><span data-stu-id="17915-477">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="17915-478">Ezért az újrapróbálható kódblokknak a tranzakció teljes hatókörét le kell fedne.</span><span class="sxs-lookup"><span data-stu-id="17915-478">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="17915-479">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="17915-479">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="17915-480">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="17915-480">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="17915-481">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="17915-481">**Context**</span></span> | <span data-ttu-id="17915-482">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="17915-482">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="17915-483">**Újrapróbálkozási stratégia**</span><span class="sxs-lookup"><span data-stu-id="17915-483">**Retry strategy**</span></span> | <span data-ttu-id="17915-484">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="17915-484">**Settings**</span></span> | <span data-ttu-id="17915-485">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="17915-485">**Values**</span></span> | <span data-ttu-id="17915-486">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="17915-486">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="17915-487">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="17915-487">Interactive, UI,</span></span><br /><span data-ttu-id="17915-488">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="17915-488">or foreground</span></span> |<span data-ttu-id="17915-489">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-489">2 sec</span></span> |<span data-ttu-id="17915-490">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="17915-490">FixedInterval</span></span> |<span data-ttu-id="17915-491">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="17915-491">Retry count</span></span><br /><span data-ttu-id="17915-492">Újrapróbálkozási időköz</span><span class="sxs-lookup"><span data-stu-id="17915-492">Retry interval</span></span><br /><span data-ttu-id="17915-493">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="17915-493">First fast retry</span></span> |<span data-ttu-id="17915-494">3</span><span class="sxs-lookup"><span data-stu-id="17915-494">3</span></span><br /><span data-ttu-id="17915-495">500 ms</span><span class="sxs-lookup"><span data-stu-id="17915-495">500 ms</span></span><br /><span data-ttu-id="17915-496">true</span><span class="sxs-lookup"><span data-stu-id="17915-496">true</span></span> |<span data-ttu-id="17915-497">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-497">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="17915-498">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-498">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="17915-499">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-499">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="17915-500">Háttér</span><span class="sxs-lookup"><span data-stu-id="17915-500">Background</span></span><br /><span data-ttu-id="17915-501">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="17915-501">or batch</span></span> |<span data-ttu-id="17915-502">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-502">30 sec</span></span> |<span data-ttu-id="17915-503">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-503">ExponentialBackoff</span></span> |<span data-ttu-id="17915-504">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="17915-504">Retry count</span></span><br /><span data-ttu-id="17915-505">Visszatartás (min.)</span><span class="sxs-lookup"><span data-stu-id="17915-505">Min back-off</span></span><br /><span data-ttu-id="17915-506">Visszatartás (max.)</span><span class="sxs-lookup"><span data-stu-id="17915-506">Max back-off</span></span><br /><span data-ttu-id="17915-507">Visszatartás (változás)</span><span class="sxs-lookup"><span data-stu-id="17915-507">Delta back-off</span></span><br /><span data-ttu-id="17915-508">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="17915-508">First fast retry</span></span> |<span data-ttu-id="17915-509">5</span><span class="sxs-lookup"><span data-stu-id="17915-509">5</span></span><br /><span data-ttu-id="17915-510">0 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-510">0 sec</span></span><br /><span data-ttu-id="17915-511">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-511">60 sec</span></span><br /><span data-ttu-id="17915-512">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-512">2 sec</span></span><br /><span data-ttu-id="17915-513">false</span><span class="sxs-lookup"><span data-stu-id="17915-513">false</span></span> |<span data-ttu-id="17915-514">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-514">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="17915-515">2. kísérlet – kb. 2 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-515">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="17915-516">3. kísérlet – kb. 6 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-516">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="17915-517">4. kísérlet – kb. 14 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-517">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="17915-518">5. kísérlet – kb. 30 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-518">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="17915-519">A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz.</span><span class="sxs-lookup"><span data-stu-id="17915-519">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="17915-520">Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.</span><span class="sxs-lookup"><span data-stu-id="17915-520">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="17915-521">Példák</span><span class="sxs-lookup"><span data-stu-id="17915-521">Examples</span></span>
<span data-ttu-id="17915-522">Ebben a szakaszban azt mutatjuk be, hogy miként használhatja a Pollyt az Azure SQL Database elérésére a `Policy` osztályban konfigurált újrapróbálkozási szabályzatok segítségével.</span><span class="sxs-lookup"><span data-stu-id="17915-522">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="17915-523">A következő programkód bemutatja, hogyan bővítheti az `SqlCommand` osztályt, ha az exponenciális visszatartással hívja meg az `ExecuteAsync` metódust.</span><span class="sxs-lookup"><span data-stu-id="17915-523">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="17915-524">Ezt az aszinkron bővítési metódust a következőképpen lehet használni.</span><span class="sxs-lookup"><span data-stu-id="17915-524">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="17915-525">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-525">More information</span></span>
* [<span data-ttu-id="17915-526">Cloud Service Fundamentals adatelérési réteg – átmeneti hibák kezelése</span><span class="sxs-lookup"><span data-stu-id="17915-526">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="17915-527">Ha további útmutatásra van szüksége az SQL Database minél jobb kihasználásához, olvassa el [az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatóját](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="17915-527">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>


## <a name="service-bus-retry-guidelines"></a><span data-ttu-id="17915-528">A Service Bus újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-528">Service Bus retry guidelines</span></span>
<span data-ttu-id="17915-529">A Service Bus egy felhőalapú üzenetkezelési platform, amely skálázható és rugalmas módon biztosít lazán kapcsolódó üzenetváltásokat az alkalmazások összetevői számára, legyen szó felhőalapú vagy helyszíni megoldásról.</span><span class="sxs-lookup"><span data-stu-id="17915-529">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-530">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-530">Retry mechanism</span></span>
<span data-ttu-id="17915-531">A Service Bus a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály implementációi alapján implementálja az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="17915-531">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="17915-532">Az összes Service Bus-ügyfél elérhetővé tesz egy **RetryPolicy** tulajdonságot, amely beállítható a **RetryPolicy** alaposztály egyik implementációjaként.</span><span class="sxs-lookup"><span data-stu-id="17915-532">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="17915-533">A beépített implementációk a következők:</span><span class="sxs-lookup"><span data-stu-id="17915-533">The built-in implementations are:</span></span>

* <span data-ttu-id="17915-534">A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="17915-534">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="17915-535">Ez elérhetővé teszi azokat a tulajdonságokat, amelyek a visszatartási időközöket, az újrapróbálkozások számát és a **TerminationTimeBuffer** tulajdonságot szabályozzák, amely korlátozza, hogy a művelet hányszor hajtható végre.</span><span class="sxs-lookup"><span data-stu-id="17915-535">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="17915-536">A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="17915-536">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="17915-537">Ezt akkor szokták használni, ha nincs szükség újrapróbálkozásra a Service Bus API-szintjén, például ha az újrapróbálkozásokat egy másik folyamat kezeli egy kötegelt vagy többlépéses művelet részeként.</span><span class="sxs-lookup"><span data-stu-id="17915-537">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="17915-538">A Service Bus-műveletek számos kivételt adhatnak vissza. Ezek listáját [az üzenetkezelési kivételek függelékében](http://msdn.microsoft.com/library/hh418082.aspx) találja.</span><span class="sxs-lookup"><span data-stu-id="17915-538">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="17915-539">A lista azt is ismerteti, hogy ezek közül melyik utal arra, hogy a művelet újrapróbálható.</span><span class="sxs-lookup"><span data-stu-id="17915-539">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="17915-540">Például a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) azt jelzi, hogy az ügyfélnek várnia kell egy ideig, majd ismét megpróbálkozni a művelettel.</span><span class="sxs-lookup"><span data-stu-id="17915-540">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="17915-541">A **ServerBusyException** jelentkezésekor a Service Bus eltérő módba vált, amelyben további 10 másodperc adódik a kiszámított újrapróbálkozási késleltetéshez.</span><span class="sxs-lookup"><span data-stu-id="17915-541">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="17915-542">Ebből a módból rövid időn belül automatikusan kilép.</span><span class="sxs-lookup"><span data-stu-id="17915-542">This mode is reset after a short period.</span></span>

<span data-ttu-id="17915-543">A Service Bus által visszaadott kivételek elérhetővé teszik az **IsTransient** tulajdonságot, amely jelzi, hogy az ügyfélnek érdemes-e újrapróbálkoznia a művelettel.</span><span class="sxs-lookup"><span data-stu-id="17915-543">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="17915-544">A beépített **RetryExponential** szabályzat a **MessagingException** osztály (az összes Service Bus-kivétel alaposztálya) **IsTransient** tulajdonságára hagyatkozik.</span><span class="sxs-lookup"><span data-stu-id="17915-544">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="17915-545">A **RetryPolicy** alaposztály egyéni implementációi esetében a kivételtípus és az **IsTransient** tulajdonság közös használatával pontosabban szabályozhatja az újrapróbálkozási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="17915-545">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="17915-546">Például észlelhet egy **QuotaExceededException**-kivételt, és utasítást adhat, hogy csak az üzenetsor kiürítése után próbálkozzon újra az üzenetküldéssel.</span><span class="sxs-lookup"><span data-stu-id="17915-546">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="17915-547">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="17915-547">Policy configuration</span></span>
<span data-ttu-id="17915-548">Az újrapróbálkozási szabályzatok beállítása szoftveresen történik, és beállítható alapértelmezett szabályzatként a **NamespaceManager** és a **MessagingFactory** számára, vagy egyenként, az egyes üzenetkezelési ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="17915-548">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="17915-549">Az üzenetkezelési munkamenet alapértelmezett újrapróbálkozási szabályzatát a **NamespaceManager** **RetryPolicy** tulajdonságában állíthatja be.</span><span class="sxs-lookup"><span data-stu-id="17915-549">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

<span data-ttu-id="17915-550">Az üzenetkezelési előállítóból létrehozott ügyelek alapértelmezett újrapróbálkozási szabályzatát a **MessagingFactory** **RetryPolicy** tulajdonságában állíthatja be.</span><span class="sxs-lookup"><span data-stu-id="17915-550">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

<span data-ttu-id="17915-551">Az üzenetkezelési ügyfél alapértelmezett újrapróbálkozási szabályzatának beállításához, illetve az alapértelmezett szabályzat felülbírálásához a szükséges szabályzatosztály példányának **RetryPolicy** tulajdonságát kell megadnia:</span><span class="sxs-lookup"><span data-stu-id="17915-551">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="17915-552">Az újrapróbálkozási szabályzat nem állítható be az egyes műveletek szintjén.</span><span class="sxs-lookup"><span data-stu-id="17915-552">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="17915-553">Az az üzenetkezelési ügyfél összes műveletére érvényes lesz.</span><span class="sxs-lookup"><span data-stu-id="17915-553">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="17915-554">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.</span><span class="sxs-lookup"><span data-stu-id="17915-554">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="17915-555">Beállítás</span><span class="sxs-lookup"><span data-stu-id="17915-555">Setting</span></span> | <span data-ttu-id="17915-556">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="17915-556">Default value</span></span> | <span data-ttu-id="17915-557">Jelentés</span><span class="sxs-lookup"><span data-stu-id="17915-557">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="17915-558">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="17915-558">Policy</span></span> | <span data-ttu-id="17915-559">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-559">Exponential</span></span> | <span data-ttu-id="17915-560">Exponenciális visszatartás.</span><span class="sxs-lookup"><span data-stu-id="17915-560">Exponential back-off.</span></span> |
| <span data-ttu-id="17915-561">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-561">MinimalBackoff</span></span> | <span data-ttu-id="17915-562">0</span><span class="sxs-lookup"><span data-stu-id="17915-562">0</span></span> | <span data-ttu-id="17915-563">Minimális visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="17915-563">Minimum back-off interval.</span></span> <span data-ttu-id="17915-564">Ez hozzáadódik a deltaBackoff alapján kiszámított újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="17915-564">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="17915-565">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-565">MaximumBackoff</span></span> | <span data-ttu-id="17915-566">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-566">30 seconds</span></span> | <span data-ttu-id="17915-567">Maximális visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="17915-567">Maximum back-off interval.</span></span> <span data-ttu-id="17915-568">A MaximumBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke.</span><span class="sxs-lookup"><span data-stu-id="17915-568">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="17915-569">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-569">DeltaBackoff</span></span> | <span data-ttu-id="17915-570">3 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-570">3 seconds</span></span> | <span data-ttu-id="17915-571">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="17915-571">Back-off interval between retries.</span></span> <span data-ttu-id="17915-572">Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét fogja használni a rendszer.</span><span class="sxs-lookup"><span data-stu-id="17915-572">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="17915-573">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="17915-573">TimeBuffer</span></span> | <span data-ttu-id="17915-574">5 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-574">5 seconds</span></span> | <span data-ttu-id="17915-575">Az újrapróbálkozáshoz tartozó leállítási időpuffer.</span><span class="sxs-lookup"><span data-stu-id="17915-575">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="17915-576">A rendszer felhagy az újrapróbálkozással, ha a TimeBuffer értékben megadottnál kevesebb idő van hátra.</span><span class="sxs-lookup"><span data-stu-id="17915-576">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="17915-577">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="17915-577">MaxRetryCount</span></span> | <span data-ttu-id="17915-578">10</span><span class="sxs-lookup"><span data-stu-id="17915-578">10</span></span> | <span data-ttu-id="17915-579">Az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="17915-579">The maximum number of retries.</span></span> |
| <span data-ttu-id="17915-580">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="17915-580">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="17915-581">10 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-581">10 seconds</span></span> | <span data-ttu-id="17915-582">Ha az utolsó kivétel a **ServerBusyException** volt, ez az érték hozzáadódik a kiszámított újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="17915-582">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="17915-583">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="17915-583">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="17915-584">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="17915-584">Retry usage guidance</span></span>
<span data-ttu-id="17915-585">Ügyeljen a következőkre a Service Bus használata során:</span><span class="sxs-lookup"><span data-stu-id="17915-585">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="17915-586">A beépített **RetryExponential** implementáció használatakor nincs szükség tartalékműveletek megvalósítására, mivel a szabályzat a „foglalt kiszolgáló” kivételekre reagálva automatikusan átvált a megfelelő újrapróbálkozási módra.</span><span class="sxs-lookup"><span data-stu-id="17915-586">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="17915-587">A Service Bus támogatja a Párosított névterek nevű funkciót, amely automatikus feladatátvételt implementál, és az elsődleges névtér üzenetsorának hibájakor egy másik névtér tartalék üzenetsorára vált.</span><span class="sxs-lookup"><span data-stu-id="17915-587">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="17915-588">A másodlagos üzenetsor üzenetei továbbküldhetők az elsődleges üzenetsornak, miután az helyreállt.</span><span class="sxs-lookup"><span data-stu-id="17915-588">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="17915-589">Ez a funkció az átmeneti hibák kezelésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="17915-589">This feature helps to address transient failures.</span></span> <span data-ttu-id="17915-590">További információért lásd [az aszinkron üzenetkezelési minták és a magas rendelkezésre állás](http://msdn.microsoft.com/library/azure/dn292562.aspx) ismertetését.</span><span class="sxs-lookup"><span data-stu-id="17915-590">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="17915-591">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="17915-591">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="17915-592">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="17915-592">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="17915-593">Környezet</span><span class="sxs-lookup"><span data-stu-id="17915-593">Context</span></span> | <span data-ttu-id="17915-594">Példa a maximális késésre</span><span class="sxs-lookup"><span data-stu-id="17915-594">Example maximum latency</span></span> | <span data-ttu-id="17915-595">Újrapróbálkozási szabályzat</span><span class="sxs-lookup"><span data-stu-id="17915-595">Retry policy</span></span> | <span data-ttu-id="17915-596">Beállítások</span><span class="sxs-lookup"><span data-stu-id="17915-596">Settings</span></span> | <span data-ttu-id="17915-597">Működés</span><span class="sxs-lookup"><span data-stu-id="17915-597">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="17915-598">Interaktív, felhasználói felület vagy előtér</span><span class="sxs-lookup"><span data-stu-id="17915-598">Interactive, UI, or foreground</span></span> | <span data-ttu-id="17915-599">2 másodperc\*</span><span class="sxs-lookup"><span data-stu-id="17915-599">2 seconds\*</span></span>  | <span data-ttu-id="17915-600">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-600">Exponential</span></span> | <span data-ttu-id="17915-601">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="17915-601">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="17915-602">MaximumBackoff = 30 mp.</span><span class="sxs-lookup"><span data-stu-id="17915-602">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="17915-603">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="17915-603">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="17915-604">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="17915-604">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="17915-605">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="17915-605">MaxRetryCount = 2</span></span> | <span data-ttu-id="17915-606">1. kísérlet: 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-606">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="17915-607">2. kísérlet: kb. 300 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-607">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="17915-608">3. kísérlet: kb. 900 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-608">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="17915-609">Háttér vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="17915-609">Background or batch</span></span> | <span data-ttu-id="17915-610">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-610">30 seconds</span></span> | <span data-ttu-id="17915-611">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="17915-611">Exponential</span></span> | <span data-ttu-id="17915-612">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="17915-612">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="17915-613">MaximumBackoff = 30 mp.</span><span class="sxs-lookup"><span data-stu-id="17915-613">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="17915-614">DeltaBackoff = 1,75 mp.</span><span class="sxs-lookup"><span data-stu-id="17915-614">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="17915-615">TimeBuffer = 5 mp.</span><span class="sxs-lookup"><span data-stu-id="17915-615">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="17915-616">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="17915-616">MaxRetryCount = 3</span></span> | <span data-ttu-id="17915-617">1. kísérlet: kb. 1 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-617">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="17915-618">2. kísérlet: kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-618">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="17915-619">3. kísérlet: kb. 6 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-619">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="17915-620">4. kísérlet: kb. 13 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-620">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="17915-621">\* Nem tartalmazza a további késleltetést, amely a „foglalt kiszolgáló” válasz esetén adódik az értékhez.</span><span class="sxs-lookup"><span data-stu-id="17915-621">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="17915-622">Telemetria</span><span class="sxs-lookup"><span data-stu-id="17915-622">Telemetry</span></span>
<span data-ttu-id="17915-623">A Service Bus ETW-eseményként naplózza az újrapróbálkozásokat egy **EventSource** használatával.</span><span class="sxs-lookup"><span data-stu-id="17915-623">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="17915-624">Egy **EventListener** az eseményforráshoz csatolása szükséges, ha rögzíteni kívánja az eseményeket, és meg kívánja tekinteni azokat a Teljesítménynaplóban, vagy ha egy megfelelő célnaplóba írná azokat.</span><span class="sxs-lookup"><span data-stu-id="17915-624">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="17915-625">Ehhez a [szemantikus naplózási alkalmazásblokk](http://msdn.microsoft.com/library/dn775006.aspx) használatát javasoljuk.</span><span class="sxs-lookup"><span data-stu-id="17915-625">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="17915-626">Az újrapróbálkozási események formátuma a következő:</span><span class="sxs-lookup"><span data-stu-id="17915-626">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="17915-627">Példák</span><span class="sxs-lookup"><span data-stu-id="17915-627">Examples</span></span>
<span data-ttu-id="17915-628">A következő mintakód bemutatja, hogyan állíthatja be az újrapróbálkozási szabályzatot a következőkhöz:</span><span class="sxs-lookup"><span data-stu-id="17915-628">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="17915-629">Egy névtérkezelő.</span><span class="sxs-lookup"><span data-stu-id="17915-629">A namespace manager.</span></span> <span data-ttu-id="17915-630">A szabályzat a kezelő összes műveletére vonatkozik, és nem bírálható felül az egyes műveletek esetében.</span><span class="sxs-lookup"><span data-stu-id="17915-630">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="17915-631">Egy üzenetkezelési előállító.</span><span class="sxs-lookup"><span data-stu-id="17915-631">A messaging factory.</span></span> <span data-ttu-id="17915-632">A szabályzat az előállítóból létrehozott összes ügyfélre vonatkozik, és nem bírálható felül az egyes ügyfelek létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="17915-632">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="17915-633">Egy különálló üzenetkezelési ügyfél.</span><span class="sxs-lookup"><span data-stu-id="17915-633">An individual messaging client.</span></span> <span data-ttu-id="17915-634">Az ügyfél létrehozása után beállíthatja annak újrapróbálkozási szabályzatát.</span><span class="sxs-lookup"><span data-stu-id="17915-634">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="17915-635">A szabályzat az ügyfél összes műveletére vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="17915-635">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="17915-636">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-636">More information</span></span>
* [<span data-ttu-id="17915-637">Aszinkron üzenetkezelési minták és magas rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="17915-637">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a><span data-ttu-id="17915-638">Az Azure Redis Cache újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-638">Azure Redis Cache retry guidelines</span></span>
<span data-ttu-id="17915-639">Az Azure Redis Cache gyors adathozzáférést és alacsony késést kínáló gyorsítótár-szolgáltatás, amely a népszerű, nyílt forráskódú Redis Cache-re épül.</span><span class="sxs-lookup"><span data-stu-id="17915-639">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="17915-640">Biztonságos, a Microsoft felügyeli, és az Azure bármelyik alkalmazásából elérhető.</span><span class="sxs-lookup"><span data-stu-id="17915-640">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="17915-641">Ebben az útmutatóban azt feltételezzük, hogy a StackExchange.Redis ügyfelet használja a gyorsítótár eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="17915-641">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="17915-642">A további alkalmas ügyfelek listája a [Redis webhelyén](http://redis.io/clients) tekinthető meg, ám ezeknek eltérő újrapróbálkozási mechanizmusai lehetnek.</span><span class="sxs-lookup"><span data-stu-id="17915-642">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="17915-643">Vegye figyelembe, hogy a StackExchange.Redis ügyfél egyetlen kapcsolaton keresztül végez multiplexálást.</span><span class="sxs-lookup"><span data-stu-id="17915-643">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="17915-644">A javasolt felhasználás az, ha létrehozza az ügyfél egy példányát az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár elérését célzó összes művelethez.</span><span class="sxs-lookup"><span data-stu-id="17915-644">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="17915-645">Így a gyorsítótárral való kapcsolat csak egyszer jön létre, ezért az ebben a szakaszban leírt összes útmutatás ezen első kapcsolat újrapróbálkozási szabályzatára vonatkozik, nem pedig a gyorsítótárhoz hozzáférő egyes műveletekre.</span><span class="sxs-lookup"><span data-stu-id="17915-645">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-646">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-646">Retry mechanism</span></span>
<span data-ttu-id="17915-647">A StackExchange.Redis ügyfél egy számos beállítással konfigurált kapcsolatkezelő osztályt használ. Néhány példa ezekre a beállításokra:</span><span class="sxs-lookup"><span data-stu-id="17915-647">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="17915-648">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="17915-648">**ConnectRetry**.</span></span> <span data-ttu-id="17915-649">Ennyi alkalommal próbálkozik újra a gyorsítótárhoz való sikertelen kapcsolódás esetén.</span><span class="sxs-lookup"><span data-stu-id="17915-649">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="17915-650">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="17915-650">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="17915-651">A használt újrapróbálkozási stratégia.</span><span class="sxs-lookup"><span data-stu-id="17915-651">The retry strategy to use.</span></span>
- <span data-ttu-id="17915-652">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="17915-652">**ConnectTimeout**.</span></span> <span data-ttu-id="17915-653">A maximális várakozási idő milliszekundumban.</span><span class="sxs-lookup"><span data-stu-id="17915-653">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="17915-654">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="17915-654">Policy configuration</span></span>
<span data-ttu-id="17915-655">Az újrapróbálkozási szabályzat konfigurálása szoftveresen történik. Az ügyfél beállításait a gyorsítótárhoz való kapcsolódás előtt kell megadni.</span><span class="sxs-lookup"><span data-stu-id="17915-655">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="17915-656">Ehhez létre kell hozni a **ConfigurationOptions** osztály egy példányát, feltölteni adatokkal a tulajdonságait, majd továbbítani azt a **Connect** metódusnak.</span><span class="sxs-lookup"><span data-stu-id="17915-656">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="17915-657">A beépített osztályok támogatják a lineáris (állandó) késleltetést, illetve az exponenciális visszatartást véletlenszerű újrapróbálkozási időközökkel.</span><span class="sxs-lookup"><span data-stu-id="17915-657">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="17915-658">Ezenkívül létrehozhat egyéni újrapróbálkozási szabályzatot az **IReconnectRetryPolicy** felület implementálásával.</span><span class="sxs-lookup"><span data-stu-id="17915-658">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="17915-659">A következő példa exponenciális visszatartással konfigurálja az újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="17915-659">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="17915-660">Egy másik lehetőség, hogy a beállításokat karakterláncként adja meg és továbbítja a **Connect** metódusnak.</span><span class="sxs-lookup"><span data-stu-id="17915-660">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="17915-661">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezen a módon, csak a programkódon keresztül.</span><span class="sxs-lookup"><span data-stu-id="17915-661">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="17915-662">Amikor a gyorsítótárhoz csatlakozik, közvetlenül is megadhat beállításokat.</span><span class="sxs-lookup"><span data-stu-id="17915-662">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="17915-663">További információért tekintse meg [a Stack Exchange Redis konfigurálását](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="17915-663">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="17915-664">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.</span><span class="sxs-lookup"><span data-stu-id="17915-664">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="17915-665">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="17915-665">**Context**</span></span> | <span data-ttu-id="17915-666">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="17915-666">**Setting**</span></span> | <span data-ttu-id="17915-667">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="17915-667">**Default value**</span></span><br /><span data-ttu-id="17915-668">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="17915-668">(v 1.2.2)</span></span> | <span data-ttu-id="17915-669">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="17915-669">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="17915-670">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="17915-670">ConfigurationOptions</span></span> |<span data-ttu-id="17915-671">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="17915-671">ConnectRetry</span></span><br /><br /><span data-ttu-id="17915-672">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="17915-672">ConnectTimeout</span></span><br /><br /><span data-ttu-id="17915-673">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="17915-673">SyncTimeout</span></span><br /><br /><span data-ttu-id="17915-674">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="17915-674">ReconnectRetryPolicy</span></span> |<span data-ttu-id="17915-675">3</span><span class="sxs-lookup"><span data-stu-id="17915-675">3</span></span><br /><br /><span data-ttu-id="17915-676">Legfeljebb 5000 ms, plusz SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="17915-676">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="17915-677">1000</span><span class="sxs-lookup"><span data-stu-id="17915-677">1000</span></span><br /><br /><span data-ttu-id="17915-678">LinearRetry 5000 ms</span><span class="sxs-lookup"><span data-stu-id="17915-678">LinearRetry 5000 ms</span></span> |<span data-ttu-id="17915-679">Ennyi alkalommal kell megismételni a csatlakozási kísérletet az első csatlakozási művelet során.</span><span class="sxs-lookup"><span data-stu-id="17915-679">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="17915-680">A csatlakozási műveletek időtúllépési értéke (ms).</span><span class="sxs-lookup"><span data-stu-id="17915-680">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="17915-681">Nem az újrapróbálkozás kísérletek közötti késleltetést jelzi.</span><span class="sxs-lookup"><span data-stu-id="17915-681">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="17915-682">A szinkron műveletek számára biztosított idő (ms).</span><span class="sxs-lookup"><span data-stu-id="17915-682">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="17915-683">Újrapróbálkozás 5000 ms időközönként.</span><span class="sxs-lookup"><span data-stu-id="17915-683">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="17915-684">A szinkron műveletek esetében a `SyncTimeout` hozzájárulhat a végpontok közötti késéshez, de a túl alacsony érték gyakori időtúllépéseket eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="17915-684">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="17915-685">További információért lásd [az Azure Redis Cache hibaelhárítását][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="17915-685">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="17915-686">Általában kerülje a szinkron műveletek használatát, és alkalmazzon inkább aszinkron műveleteket.</span><span class="sxs-lookup"><span data-stu-id="17915-686">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="17915-687">További információért lásd a [folyamatokat és a multiplexereket](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="17915-687">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="17915-688">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="17915-688">Retry usage guidance</span></span>
<span data-ttu-id="17915-689">Ügyeljen a következőkre az Azure Redis Cache használata során:</span><span class="sxs-lookup"><span data-stu-id="17915-689">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="17915-690">A StackExchange Redis ügyfél kezeli a saját újrapróbálkozásait, de csak amikor az alkalmazás első indításakor próbál kapcsolódni a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="17915-690">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="17915-691">Meghatározhatja a kapcsolati időtúllépés értékét, az újrapróbálkozási kísérletek számát, valamint a kapcsolat létrehozására tett ismételt próbálkozások között eltelt időt, de az újrapróbálkozási szabályzat nem vonatkozik a gyorsítótárra irányuló műveletekre.</span><span class="sxs-lookup"><span data-stu-id="17915-691">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="17915-692">Nagy számú újrapróbálkozási kísérlet helyett érdemes lehet inkább az eredeti adatforráshoz csatlakozni.</span><span class="sxs-lookup"><span data-stu-id="17915-692">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="17915-693">Telemetria</span><span class="sxs-lookup"><span data-stu-id="17915-693">Telemetry</span></span>
<span data-ttu-id="17915-694">**TextWriter** használatával adatokat gyűjthet a kapcsolatokról (más műveletekről azonban nem).</span><span class="sxs-lookup"><span data-stu-id="17915-694">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="17915-695">Az alábbi példa azt mutatja be, hogy ez milyen kimenetet eredményez.</span><span class="sxs-lookup"><span data-stu-id="17915-695">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="17915-696">Példák</span><span class="sxs-lookup"><span data-stu-id="17915-696">Examples</span></span>
<span data-ttu-id="17915-697">A következő mintakód állandó (lineáris) késleltetést állít be az újrapróbálkozások között a StackExchange.Redis ügyfél inicializálásakor.</span><span class="sxs-lookup"><span data-stu-id="17915-697">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="17915-698">Ez a példa azt mutatja be, hogyan állítható be a konfiguráció egy **ConfigurationOptions**-példánnyal.</span><span class="sxs-lookup"><span data-stu-id="17915-698">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="17915-699">A következő példa karakterláncként adja meg a beállításokat, és így határozza meg a konfigurációt.</span><span class="sxs-lookup"><span data-stu-id="17915-699">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="17915-700">A kapcsolat időtúllépése a leghosszabb idő, amennyit a kapcsolat a gyorsítótárra várhat, nem pedig az újrapróbálkozási kísérletek közötti időköz.</span><span class="sxs-lookup"><span data-stu-id="17915-700">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="17915-701">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonságot csak a programkódon keresztül lehet beállítani.</span><span class="sxs-lookup"><span data-stu-id="17915-701">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="17915-702">További példákért tekintse meg a projekt webhelyének a [konfigurációval](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) foglalkozó szakaszát.</span><span class="sxs-lookup"><span data-stu-id="17915-702">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="17915-703">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-703">More information</span></span>
* [<span data-ttu-id="17915-704">Redis-webhely</span><span class="sxs-lookup"><span data-stu-id="17915-704">Redis website</span></span>](http://redis.io/)

## <a name="cosmos-db-retry-guidelines"></a><span data-ttu-id="17915-705">A Cosmos DB újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-705">Cosmos DB retry guidelines</span></span>

<span data-ttu-id="17915-706">A Cosmos DB egy teljes körűen felügyelt, többmodelles adatbázis-szolgáltatás, amely támogatja a séma nélküli JSON-adatok használatát.</span><span class="sxs-lookup"><span data-stu-id="17915-706">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="17915-707">Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="17915-707">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-708">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-708">Retry mechanism</span></span>
<span data-ttu-id="17915-709">A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletekkel.</span><span class="sxs-lookup"><span data-stu-id="17915-709">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="17915-710">Az újrapróbálkozások számának és a maximális várakozási idő beállításához a [ConnectionPolicy.RetryOptions] konfigurálása szükséges.</span><span class="sxs-lookup"><span data-stu-id="17915-710">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="17915-711">Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="17915-711">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="17915-712">Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza.</span><span class="sxs-lookup"><span data-stu-id="17915-712">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="17915-713">Ellenőrizze a `DocumentClientException` állapotkódját.</span><span class="sxs-lookup"><span data-stu-id="17915-713">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="17915-714">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="17915-714">Policy configuration</span></span>
<span data-ttu-id="17915-715">A következő táblázatban a `RetryOptions` osztály alapértelmezett beállításait tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="17915-715">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="17915-716">Beállítás</span><span class="sxs-lookup"><span data-stu-id="17915-716">Setting</span></span> | <span data-ttu-id="17915-717">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="17915-717">Default value</span></span> | <span data-ttu-id="17915-718">Leírás</span><span class="sxs-lookup"><span data-stu-id="17915-718">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="17915-719">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="17915-719">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="17915-720">9</span><span class="sxs-lookup"><span data-stu-id="17915-720">9</span></span> |<span data-ttu-id="17915-721">Az újrapróbálkozások maximális száma abban az esetben, ha a Cosmos DB az ügyfélre alkalmazott korlátozása miatt sikertelen a kísérlet.</span><span class="sxs-lookup"><span data-stu-id="17915-721">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="17915-722">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="17915-722">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="17915-723">30</span><span class="sxs-lookup"><span data-stu-id="17915-723">30</span></span> |<span data-ttu-id="17915-724">A maximális újrapróbálkozási idő másodpercben.</span><span class="sxs-lookup"><span data-stu-id="17915-724">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="17915-725">Példa</span><span class="sxs-lookup"><span data-stu-id="17915-725">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="17915-726">Telemetria</span><span class="sxs-lookup"><span data-stu-id="17915-726">Telemetry</span></span>
<span data-ttu-id="17915-727">Az újrapróbálkozási kísérletek strukturálatlan nyomkövetési üzenetekként lesznek naplózva a .NET **TraceSource** használatával.</span><span class="sxs-lookup"><span data-stu-id="17915-727">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="17915-728">Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="17915-728">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="17915-729">Amennyiben például a következőt adja hozzá az App.config fájlhoz, a szövegfájlban a végrehajtható fájllal megegyező helyen jönnek létre a nyomkövetési adatok:</span><span class="sxs-lookup"><span data-stu-id="17915-729">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

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


## <a name="azure-search-retry-guidelines"></a><span data-ttu-id="17915-730">Az Azure Search újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-730">Azure Search retry guidelines</span></span>
<span data-ttu-id="17915-731">Az Azure Search hatékony és kifinomult keresési lehetőségekkel egészíthet ki egy webhelyet vagy alkalmazást, gyorsan és könnyen pontosítja a keresési eredményeket, továbbá részletes és finomhangolt rangsorolási modelleket képes létrehozni.</span><span class="sxs-lookup"><span data-stu-id="17915-731">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-732">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-732">Retry mechanism</span></span>
<span data-ttu-id="17915-733">Az Azure Search SDK újrapróbálkozási viselkedését a [SearchServiceClient] és a [SearchIndexClient] osztály `SetRetryPolicy` metódusa vezérli.</span><span class="sxs-lookup"><span data-stu-id="17915-733">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="17915-734">Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Search 5xx-es vagy 408-as (Kérés időtúllépése) választ ad vissza.</span><span class="sxs-lookup"><span data-stu-id="17915-734">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="17915-735">Telemetria</span><span class="sxs-lookup"><span data-stu-id="17915-735">Telemetry</span></span>
<span data-ttu-id="17915-736">Nyomkövetés ETW-vel vagy egyéni nyomkövetési szolgáltató regisztrálásával.</span><span class="sxs-lookup"><span data-stu-id="17915-736">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="17915-737">További információt az [AutoRest dokumentációjában][autorest] találhat.</span><span class="sxs-lookup"><span data-stu-id="17915-737">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="azure-active-directory-retry-guidelines"></a><span data-ttu-id="17915-738">Az Azure Active Directory újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-738">Azure Active Directory retry guidelines</span></span>
<span data-ttu-id="17915-739">Az Azure Active Directory egy átfogó, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést, a biztonsági szolgáltatásokat és az alkalmazáshozzáférés-felügyeletet.</span><span class="sxs-lookup"><span data-stu-id="17915-739">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="17915-740">Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi szabályzatokon és szabályokon alapuló hozzáférés-vezérléssel bővíthessék az alkalmazásaikat.</span><span class="sxs-lookup"><span data-stu-id="17915-740">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-741">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-741">Retry mechanism</span></span>
<span data-ttu-id="17915-742">Az Azure Active Directory beépített újrapróbálkozási mechanizmussal rendelkezik az Active Directory Authentication Library (ADAL) részeként.</span><span class="sxs-lookup"><span data-stu-id="17915-742">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="17915-743">A váratlan zárolások elkerülése érdekében azt javasoljuk, hogy külső kódtárak és alkalmazáskódok **ne** próbálkozhassanak újra sikertelen kapcsolódás esetén, és ezek újrapróbálkozásait az ADAL kezelje.</span><span class="sxs-lookup"><span data-stu-id="17915-743">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="17915-744">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="17915-744">Retry usage guidance</span></span>
<span data-ttu-id="17915-745">Ügyeljen a következőkre az Azure Active Directory használata során:</span><span class="sxs-lookup"><span data-stu-id="17915-745">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="17915-746">Amikor csak lehetséges, az ADAL-kódtárat és az újrapróbálkozások beépített támogatását használja.</span><span class="sxs-lookup"><span data-stu-id="17915-746">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="17915-747">Amennyiben a REST API-t használja az Azure Active Directory mellett, csak akkor érdemes engedélyezni az újrapróbálkozást a műveletek számára, ha az eredmény az 5xx-es tartományba esik (például: 500 Belső kiszolgálóhiba; 502 Hibás átjáró; 503 A szolgáltatás nem érhető el; és 504 Időtúllépés az átjárón).</span><span class="sxs-lookup"><span data-stu-id="17915-747">If you are using the REST API for Azure Active Directory, you should retry the operation only if the result is an error in the 5xx range (such as 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, and 504 Gateway Timeout).</span></span> <span data-ttu-id="17915-748">Más hibák esetében ne engedélyezze az újrapróbálkozást.</span><span class="sxs-lookup"><span data-stu-id="17915-748">Do not retry for any other errors.</span></span>
* <span data-ttu-id="17915-749">Az exponenciális visszatartási szabályzat használatát az Azure Active Directory Batch-forgatókönyvei esetében javasoljuk.</span><span class="sxs-lookup"><span data-stu-id="17915-749">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="17915-750">A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="17915-750">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="17915-751">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="17915-751">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="17915-752">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="17915-752">**Context**</span></span> | <span data-ttu-id="17915-753">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="17915-753">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="17915-754">**Újrapróbálkozási stratégia**</span><span class="sxs-lookup"><span data-stu-id="17915-754">**Retry strategy**</span></span> | <span data-ttu-id="17915-755">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="17915-755">**Settings**</span></span> | <span data-ttu-id="17915-756">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="17915-756">**Values**</span></span> | <span data-ttu-id="17915-757">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="17915-757">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="17915-758">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="17915-758">Interactive, UI,</span></span><br /><span data-ttu-id="17915-759">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="17915-759">or foreground</span></span> |<span data-ttu-id="17915-760">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-760">2 sec</span></span> |<span data-ttu-id="17915-761">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="17915-761">FixedInterval</span></span> |<span data-ttu-id="17915-762">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="17915-762">Retry count</span></span><br /><span data-ttu-id="17915-763">Újrapróbálkozási időköz</span><span class="sxs-lookup"><span data-stu-id="17915-763">Retry interval</span></span><br /><span data-ttu-id="17915-764">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="17915-764">First fast retry</span></span> |<span data-ttu-id="17915-765">3</span><span class="sxs-lookup"><span data-stu-id="17915-765">3</span></span><br /><span data-ttu-id="17915-766">500 ms</span><span class="sxs-lookup"><span data-stu-id="17915-766">500 ms</span></span><br /><span data-ttu-id="17915-767">true</span><span class="sxs-lookup"><span data-stu-id="17915-767">true</span></span> |<span data-ttu-id="17915-768">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-768">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="17915-769">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-769">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="17915-770">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-770">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="17915-771">Háttér vagy</span><span class="sxs-lookup"><span data-stu-id="17915-771">Background or</span></span><br /><span data-ttu-id="17915-772">kötegelt</span><span class="sxs-lookup"><span data-stu-id="17915-772">batch</span></span> |<span data-ttu-id="17915-773">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-773">60 sec</span></span> |<span data-ttu-id="17915-774">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="17915-774">ExponentialBackoff</span></span> |<span data-ttu-id="17915-775">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="17915-775">Retry count</span></span><br /><span data-ttu-id="17915-776">Visszatartás (min.)</span><span class="sxs-lookup"><span data-stu-id="17915-776">Min back-off</span></span><br /><span data-ttu-id="17915-777">Visszatartás (max.)</span><span class="sxs-lookup"><span data-stu-id="17915-777">Max back-off</span></span><br /><span data-ttu-id="17915-778">Visszatartás (változás)</span><span class="sxs-lookup"><span data-stu-id="17915-778">Delta back-off</span></span><br /><span data-ttu-id="17915-779">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="17915-779">First fast retry</span></span> |<span data-ttu-id="17915-780">5</span><span class="sxs-lookup"><span data-stu-id="17915-780">5</span></span><br /><span data-ttu-id="17915-781">0 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-781">0 sec</span></span><br /><span data-ttu-id="17915-782">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-782">60 sec</span></span><br /><span data-ttu-id="17915-783">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="17915-783">2 sec</span></span><br /><span data-ttu-id="17915-784">false</span><span class="sxs-lookup"><span data-stu-id="17915-784">false</span></span> |<span data-ttu-id="17915-785">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-785">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="17915-786">2. kísérlet – kb. 2 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-786">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="17915-787">3. kísérlet – kb. 6 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-787">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="17915-788">4. kísérlet – kb. 14 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-788">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="17915-789">5. kísérlet – kb. 30 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="17915-789">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="17915-790">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-790">More information</span></span>
* <span data-ttu-id="17915-791">[Az Azure Active Directory hitelesítési kódtárai][adal]</span><span class="sxs-lookup"><span data-stu-id="17915-791">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="service-fabric-retry-guidelines"></a><span data-ttu-id="17915-792">A Service Fabric újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-792">Service Fabric retry guidelines</span></span>

<span data-ttu-id="17915-793">A megbízható szolgáltatások Service Fabric-fürtön belüli elosztásával a cikkben tárgyalt legtöbb átmeneti hiba elkerülhető.</span><span class="sxs-lookup"><span data-stu-id="17915-793">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="17915-794">Azonban így is előfordulhatnak átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="17915-794">Some transient faults are still possible, however.</span></span> <span data-ttu-id="17915-795">Előfordulhat például, hogy a kérés érkezésekor az elnevezési szolgáltatás egy útválasztási változtatást végez, ami kivételt eredményez.</span><span class="sxs-lookup"><span data-stu-id="17915-795">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="17915-796">Ugyanez a kérés 100 milliszekundummal később talán sikeres lett volna.</span><span class="sxs-lookup"><span data-stu-id="17915-796">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="17915-797">A Service Fabric az ehhez hasonló átmeneti hibák belső kezelését elvégzi.</span><span class="sxs-lookup"><span data-stu-id="17915-797">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="17915-798">A szolgáltatás beállítása közben konfigurálhat egyes beállításokat az `OperationRetrySettings` osztállyal.</span><span class="sxs-lookup"><span data-stu-id="17915-798">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="17915-799">Az alábbi kód erre mutat egy példát.</span><span class="sxs-lookup"><span data-stu-id="17915-799">The following code shows an example.</span></span> <span data-ttu-id="17915-800">A legtöbb esetben ez nem szükséges, és az alapértelmezett beállítások tökéletesen megfelelnek.</span><span class="sxs-lookup"><span data-stu-id="17915-800">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

## <a name="more-information"></a><span data-ttu-id="17915-801">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-801">More information</span></span>

* [<span data-ttu-id="17915-802">Távoli kivételkezelés</span><span class="sxs-lookup"><span data-stu-id="17915-802">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a><span data-ttu-id="17915-803">Az Azure Event Hubs újrapróbálkozási irányelvei</span><span class="sxs-lookup"><span data-stu-id="17915-803">Azure Event Hubs retry guidelines</span></span>

<span data-ttu-id="17915-804">Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.</span><span class="sxs-lookup"><span data-stu-id="17915-804">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="17915-805">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="17915-805">Retry mechanism</span></span>
<span data-ttu-id="17915-806">Az Azure Event Hubs Client Library újrapróbálkozási viselkedését az `EventHubClient` osztály `RetryPolicy` tulajdonsága vezérli.</span><span class="sxs-lookup"><span data-stu-id="17915-806">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="17915-807">Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Event Hub egy átmeneti `EventHubsException` vagy egy `OperationCanceledException` választ ad.</span><span class="sxs-lookup"><span data-stu-id="17915-807">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="17915-808">Példa</span><span class="sxs-lookup"><span data-stu-id="17915-808">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="17915-809">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-809">More information</span></span>
[<span data-ttu-id="17915-810">.NET standard ügyféloldali kódtár az Azure Event Hubshoz</span><span class="sxs-lookup"><span data-stu-id="17915-810"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="17915-811">Általános REST- és újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="17915-811">General REST and retry guidelines</span></span>
<span data-ttu-id="17915-812">Vegye figyelembe a következőket, amikor az Azure-t vagy külső szolgáltatást próbál elérni:</span><span class="sxs-lookup"><span data-stu-id="17915-812">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="17915-813">Alkalmazzon rendszerszintű megközelítést az újrapróbálkozások kezelésére, például egy újrafelhasználható kód formájában, hogy minden ügyfél és megoldás esetében ugyanaz a módszertan érvényesüljön.</span><span class="sxs-lookup"><span data-stu-id="17915-813">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="17915-814">Érdemes lehet egy újrapróbálkozási keretrendszert használni (például az átmenetihiba-kezelési alkalmazásblokkot) az újrapróbálkozások kezelésére, ha a célszolgáltatás vagy -ügyfél nem rendelkezik beépített újrapróbálkozási mechanizmussal.</span><span class="sxs-lookup"><span data-stu-id="17915-814">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="17915-815">Így konzisztens újrapróbálkozási viselkedés valósítható meg, és megfelelő alapértelmezett újrapróbálkozási stratégiát tehet elérhetővé a célszolgáltatás számára.</span><span class="sxs-lookup"><span data-stu-id="17915-815">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="17915-816">Ugyanakkor szükség lehet egyéni újrapróbálkozási kód kialakítására a nem szabványos viselkedésű szolgáltatások esetében, amelyek nem kivételek alapján állapítják meg az átmeneti hibákat, illetve ha **Retry-Response** válasszal kívánja kezelni az újrapróbálkozási viselkedést.</span><span class="sxs-lookup"><span data-stu-id="17915-816">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="17915-817">Az átmeneti hibák észlelési logikája a REST-hívások végrehajtására ténylegesen használt ügyfél-API-tól függ.</span><span class="sxs-lookup"><span data-stu-id="17915-817">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="17915-818">Egyes ügyfelek, például az újabb **HttpClient** osztály, nem váltanak ki kivételeket, ha a kérés befejeződik, de nem sikert jelző HTTP-állapotkódot ad vissza.</span><span class="sxs-lookup"><span data-stu-id="17915-818">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="17915-819">Ez növeli a teljesítményt, de megakadályozza az átmenetihiba-kezelési alkalmazásblokk használatát.</span><span class="sxs-lookup"><span data-stu-id="17915-819">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="17915-820">Ebben az esetben a hívást olyan kóddal kell beburkolni a REST API számára, amely kivételeket vált ki a nem sikert jelző HTTP-állapotkódok esetén, amelyet aztán feldolgozhat a blokk.</span><span class="sxs-lookup"><span data-stu-id="17915-820">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="17915-821">Ezenkívül használhat egy másik mechanizmust is az újrapróbálkozások kezelésére.</span><span class="sxs-lookup"><span data-stu-id="17915-821">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="17915-822">A szolgáltatás által visszaadott HTTP-állapotkód segíthet megállapítani, hogy a hiba átmeneti jellegű-e.</span><span class="sxs-lookup"><span data-stu-id="17915-822">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="17915-823">Lehet, hogy az állapotkód vagy az azzal egyenértékű kivételtípus megismeréséhez meg kell vizsgálnia az ügyfél vagy az újrapróbálkozási keretrendszer által előállított kivételeket.</span><span class="sxs-lookup"><span data-stu-id="17915-823">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="17915-824">A következő HTTP-kódok általában azt jelzik, hogy érdemes újrapróbálkozni:</span><span class="sxs-lookup"><span data-stu-id="17915-824">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="17915-825">408 Kérés időtúllépése</span><span class="sxs-lookup"><span data-stu-id="17915-825">408 Request Timeout</span></span>
  * <span data-ttu-id="17915-826">500 Belső kiszolgálóhiba</span><span class="sxs-lookup"><span data-stu-id="17915-826">500 Internal Server Error</span></span>
  * <span data-ttu-id="17915-827">502 Hibás átjáró</span><span class="sxs-lookup"><span data-stu-id="17915-827">502 Bad Gateway</span></span>
  * <span data-ttu-id="17915-828">503 A szolgáltatás nem érhető el</span><span class="sxs-lookup"><span data-stu-id="17915-828">503 Service Unavailable</span></span>
  * <span data-ttu-id="17915-829">504 Időtúllépés az átjárón</span><span class="sxs-lookup"><span data-stu-id="17915-829">504 Gateway Timeout</span></span>
* <span data-ttu-id="17915-830">Amennyiben kivételekre épül az újrapróbálkozási logikája, a következők általában a csatlakozást meghiúsító átmeneti hibát jeleznek:</span><span class="sxs-lookup"><span data-stu-id="17915-830">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="17915-831">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="17915-831">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="17915-832">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="17915-832">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="17915-833">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="17915-833">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="17915-834">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="17915-834">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="17915-835">Amennyiben egy szolgáltatás állapota „Nem érhető el”, előfordulhat, hogy a szolgáltatás a megfelelő késleltetést jelzi, mielőtt újrapróbálkozna a **Retry-After** válaszfejlécben vagy egy másik egyéni fejlécben.</span><span class="sxs-lookup"><span data-stu-id="17915-835">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="17915-836">A szolgáltatások további információkat küldhetnek egyéni fejlécként, vagy a válasz tartalmába beágyazva.</span><span class="sxs-lookup"><span data-stu-id="17915-836">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="17915-837">Az átmenetihiba-kezelési alkalmazásblokk nem képes használni sem a szabványos, sem pedig az egyéni „retry-after”-fejléceket.</span><span class="sxs-lookup"><span data-stu-id="17915-837">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="17915-838">Ne próbálkozzon újra, ha az állapotkód ügyfélhibát jelez (4xx-es tartomány). E szabály alól csak a 408-as Kérés időtúllépése képez kivételt.</span><span class="sxs-lookup"><span data-stu-id="17915-838">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="17915-839">Tesztelje le alaposan az újrapróbálkozási stratégiáit és mechanizmusait különböző feltételek mellett, például különböző hálózati állapotok és rendszerterhelések esetén.</span><span class="sxs-lookup"><span data-stu-id="17915-839">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="17915-840">Újrapróbálkozási stratégiák</span><span class="sxs-lookup"><span data-stu-id="17915-840">Retry strategies</span></span>
<span data-ttu-id="17915-841">A következők az újrapróbálkozási stratégiák időközeinek jellemző típusai:</span><span class="sxs-lookup"><span data-stu-id="17915-841">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="17915-842">**Exponenciális**: Ez az újrapróbálkozási szabályzat véletlenszerű exponenciális visszatartási megközelítéssel hajt végre adott számú újrapróbálkozást, hogy megállapítsa az újrapróbálkozások között eltelt időt.</span><span class="sxs-lookup"><span data-stu-id="17915-842">**Exponential**: A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="17915-843">Példa:</span><span class="sxs-lookup"><span data-stu-id="17915-843">For example:</span></span>

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* <span data-ttu-id="17915-844">**Növekményes**: Ez az újrapróbálkozási stratégia adott számú újrapróbálkozási kísérletet engedélyez, és fokozatosan növeli a két újrapróbálkozás között eltelt időt.</span><span class="sxs-lookup"><span data-stu-id="17915-844">**Incremental**: A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="17915-845">Példa:</span><span class="sxs-lookup"><span data-stu-id="17915-845">For example:</span></span>

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* <span data-ttu-id="17915-846">**Lineáris**: Ez az újrapróbálkozási szabályzat adott számú újrapróbálkozást hajt végre, és az egyes újrapróbálkozások között eltelt idő előre rögzítve van.</span><span class="sxs-lookup"><span data-stu-id="17915-846">**LinearRetry**: A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="17915-847">Példa:</span><span class="sxs-lookup"><span data-stu-id="17915-847">For example:</span></span>

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a><span data-ttu-id="17915-848">További információ</span><span class="sxs-lookup"><span data-stu-id="17915-848">More information</span></span>
* [<span data-ttu-id="17915-849">Áramköri-megszakító stratégiák</span><span class="sxs-lookup"><span data-stu-id="17915-849">Circuit breaker strategies</span></span>](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="17915-850">Átmeneti hibák kezelése a Polly használatával</span><span class="sxs-lookup"><span data-stu-id="17915-850">Transient fault handling with Polly</span></span>
<span data-ttu-id="17915-851">A [Polly](http://www.thepollyproject.org) kódtár lehetővé teszi az újrapróbálkozások és az [áramköri-megszakító][circuit-breaker] stratégiák szoftveres kezelését.</span><span class="sxs-lookup"><span data-stu-id="17915-851">[Polly](http://www.thepollyproject.org) is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="17915-852">A Polly projekt a [.NET Foundation][dotnet-foundation] keretében érhető el.</span><span class="sxs-lookup"><span data-stu-id="17915-852">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="17915-853">A Polly olyan szolgáltatások számára kínál alternatívát, amelyekben az ügyfél nem támogatja natív módon az újrapróbálkozásokat. Megszünteti az olyan egyéni újrapróbálkozási kódok írásának szükségét, amelyeket egyébként nehéz lenne megfelelően implementálni.</span><span class="sxs-lookup"><span data-stu-id="17915-853">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="17915-854">A Polly ezenkívül módot ad a hibák nyomon követésére, így naplózhatóvá teszi az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="17915-854">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

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
