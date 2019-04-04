---
title: Nem megfelelő példányosítás kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: Elkerülheti, hogy folyamatosan új példányok jöjjenek létre olyan objektumokból, amelyeket elvileg csak egyszer kéne létrehozni, hogy utána meg lehessen őket osztani.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a2e42e35ae1b56b61c8f9f9ecb21ee104cd3222e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346348"
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="74dfc-103">Nem megfelelő példányosítás kizárási minta</span><span class="sxs-lookup"><span data-stu-id="74dfc-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="74dfc-104">Ronthatja a teljesítményt, ha folyamatosan új példányokat hoz létre olyan objektumokból, amelyeket elvileg csak egyszer kéne létrehozni, hogy utána meg lehessen őket osztani.</span><span class="sxs-lookup"><span data-stu-id="74dfc-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span>

## <a name="problem-description"></a><span data-ttu-id="74dfc-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="74dfc-105">Problem description</span></span>

<span data-ttu-id="74dfc-106">Számos kódtár biztosít absztrakt entitásokat a külső erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="74dfc-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="74dfc-107">Ezek az osztályok általában belsőleg kezelik az adott erőforrással kialakított saját kapcsolataikat, és olyan közvetítőként funkcionálnak, amelyeken keresztül az ügyfelek elérhetik az erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="74dfc-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="74dfc-108">Néhány példa az Azure-alkalmazásoknál használható közvetítőosztályokra:</span><span class="sxs-lookup"><span data-stu-id="74dfc-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="74dfc-109">`System.Net.Http.HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="74dfc-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="74dfc-110">HTTP használatával kommunikál egy webszolgáltatással.</span><span class="sxs-lookup"><span data-stu-id="74dfc-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="74dfc-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span><span class="sxs-lookup"><span data-stu-id="74dfc-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="74dfc-112">Üzeneteket küld és fogad egy Service Bus-üzenetsorból.</span><span class="sxs-lookup"><span data-stu-id="74dfc-112">Posts and receives messages to a Service Bus queue.</span></span>
- <span data-ttu-id="74dfc-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span><span class="sxs-lookup"><span data-stu-id="74dfc-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="74dfc-114">Kapcsolódik egy Cosmos DB-példányhoz.</span><span class="sxs-lookup"><span data-stu-id="74dfc-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="74dfc-115">`StackExchange.Redis.ConnectionMultiplexer`.</span><span class="sxs-lookup"><span data-stu-id="74dfc-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="74dfc-116">Kapcsolódik a Redishez, beleértve az Azure Redis Cache-t is.</span><span class="sxs-lookup"><span data-stu-id="74dfc-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="74dfc-117">Ezeket az osztályokat csak egyszer kell példányosítani, majd újra felhasználhatók az alkalmazás teljes élettartama során.</span><span class="sxs-lookup"><span data-stu-id="74dfc-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="74dfc-118">Azonban gyakori téveszme, hogy ezeket az osztályokat csak akkor kell létrehozni, amikor épp szükség van rájuk, és gyorsan ki kell őket adni.</span><span class="sxs-lookup"><span data-stu-id="74dfc-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="74dfc-119">(Az itt felsorolt osztályok .NET-kódtárak, azonban ez a jelenség nem csak a .NET-re jellemző.) A következő ASP.NET egy `HttpClient` példányt hoz létre egy távoli szolgáltatással folytatott kommunikációhoz.</span><span class="sxs-lookup"><span data-stu-id="74dfc-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="74dfc-120">A teljes kódmintát [itt][sample-app] találja.</span><span class="sxs-lookup"><span data-stu-id="74dfc-120">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

<span data-ttu-id="74dfc-121">Egy webalkalmazáson belül ez a technika nem méretezhető.</span><span class="sxs-lookup"><span data-stu-id="74dfc-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="74dfc-122">A rendszer egy új `HttpClient` objektumot hoz létre minden egyes felhasználói kéréshez.</span><span class="sxs-lookup"><span data-stu-id="74dfc-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="74dfc-123">Nagy terhelés mellett a webkiszolgáló kimerítheti az elérhető szoftvercsatornák számát, ami `SocketException` hibákat eredményez.</span><span class="sxs-lookup"><span data-stu-id="74dfc-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="74dfc-124">A probléma nem korlátozódik a `HttpClient` osztályra.</span><span class="sxs-lookup"><span data-stu-id="74dfc-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="74dfc-125">Az erőforrásokat burkoló, valamint azok az osztályok is hasonló hibákat okozhatnak, amelyeknek költséges a létrehozása.</span><span class="sxs-lookup"><span data-stu-id="74dfc-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="74dfc-126">A következő példa egy `ExpensiveToCreateService` osztály példányait hozza létre.</span><span class="sxs-lookup"><span data-stu-id="74dfc-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="74dfc-127">Itt a probléma nem feltétlenül a szoftvercsatornák kimerülése, hanem egyszerűen az egyes példányok létrehozásához szükséges idő hossza.</span><span class="sxs-lookup"><span data-stu-id="74dfc-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="74dfc-128">Az osztály példányainak folyamatos létrehozása és megsemmisítése negatívan befolyásolhatja a rendszer méretezhetőségét.</span><span class="sxs-lookup"><span data-stu-id="74dfc-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="74dfc-129">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="74dfc-129">How to fix the problem</span></span>

<span data-ttu-id="74dfc-130">Ha a külső erőforrást burkoló osztály megosztható és szálbiztos, hozza létre annak egy megosztott egyszeri példányát vagy újrahasználható példányainak készletét.</span><span class="sxs-lookup"><span data-stu-id="74dfc-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="74dfc-131">A következő példában egy statikus `HttpClient` példány szerepel, amely minden kérésben megosztja a kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="74dfc-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="74dfc-132">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="74dfc-132">Considerations</span></span>

- <span data-ttu-id="74dfc-133">Az ilyen kizárási minta kulcseleme egy *megosztható* objektum példányainak ismétlődő létrehozása és megsemmisítése.</span><span class="sxs-lookup"><span data-stu-id="74dfc-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="74dfc-134">Ha az adott osztály nem megosztható (nem szálbiztos), a kizárási minta nem alkalmazható.</span><span class="sxs-lookup"><span data-stu-id="74dfc-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="74dfc-135">A megosztott erőforrás típusától függhet, hogy egyszeri példányt kell használni, vagy egy készletet kell létrehozni.</span><span class="sxs-lookup"><span data-stu-id="74dfc-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="74dfc-136">A `HttpClient` osztályt megosztva és nem készletben kell használni.</span><span class="sxs-lookup"><span data-stu-id="74dfc-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="74dfc-137">Más objektumok támogathatják a készletben való használatot, ami lehetővé teszi, hogy a rendszer több példányon ossza szét a számítási feladatokat.</span><span class="sxs-lookup"><span data-stu-id="74dfc-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="74dfc-138">A több kérés között megosztott objektumoknak szálbiztosnak *kell* lenniük.</span><span class="sxs-lookup"><span data-stu-id="74dfc-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="74dfc-139">A `HttpClient` osztályt így kell használni, más osztályok azonban nem feltétlenül támogatják az egyidejű kéréseket, ezért mindenképpen olvassa át a rendelkezésre álló dokumentációt.</span><span class="sxs-lookup"><span data-stu-id="74dfc-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="74dfc-140">Körültekintően adja meg a megosztott objektumok tulajdonságait, mert versenyhelyzetekhez vezethet.</span><span class="sxs-lookup"><span data-stu-id="74dfc-140">Be careful about setting properties on shared objects, as this can lead to race conditions.</span></span> <span data-ttu-id="74dfc-141">Például a(z) `DefaultRequestHeaders` beállítása a(z) `HttpClient` osztályon minden egyes kérés előtt versenyhelyzetet eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="74dfc-141">For example, setting `DefaultRequestHeaders` on the `HttpClient` class before each request can create a race condition.</span></span> <span data-ttu-id="74dfc-142">Ezeket a tulajdonságokat egy alkalommal állítsa be (például indításkor), és hozzon létre különálló példányokat, ha különböző beállításokat kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="74dfc-142">Set such properties once (for example, during startup), and create separate instances if you need to configure different settings.</span></span>

- <span data-ttu-id="74dfc-143">Egyes erőforrástípusok ritkák, és nem szabad lefoglalni őket.</span><span class="sxs-lookup"><span data-stu-id="74dfc-143">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="74dfc-144">Ilyenek például az adatbázis-kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="74dfc-144">Database connections are an example.</span></span> <span data-ttu-id="74dfc-145">Egy megnyitott adatbázis-kapcsolat szükségtelen lefoglalása azt eredményezheti, hogy más egyidejű felhasználók nem tudják majd elérni az adatbázist.</span><span class="sxs-lookup"><span data-stu-id="74dfc-145">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="74dfc-146">A .NET-keretrendszerben számos olyan objektumot, amelyek külső erőforrásokhoz építenek ki kapcsolatot, az ilyen kapcsolatokat kezelő egyéb osztályok statikus példányosító metódusai hoznak létre.</span><span class="sxs-lookup"><span data-stu-id="74dfc-146">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="74dfc-147">Az ilyen objektumokat menteni kell és újra felhasználni, nem pedig elvetni, majd újra létrehozni.</span><span class="sxs-lookup"><span data-stu-id="74dfc-147">These objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="74dfc-148">Például az Azure Service Busban a `QueueClient` objektum egy `MessagingFactory` objektumon keresztül jön létre.</span><span class="sxs-lookup"><span data-stu-id="74dfc-148">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="74dfc-149">Belül a `MessagingFactory` kezeli a kapcsolatokat.</span><span class="sxs-lookup"><span data-stu-id="74dfc-149">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="74dfc-150">További információkért lásd: [Ajánlott eljárások a teljesítmény javításához a Service Bus-üzenetkezelés használatával][service-bus-messaging].</span><span class="sxs-lookup"><span data-stu-id="74dfc-150">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="74dfc-151">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="74dfc-151">How to detect the problem</span></span>

<span data-ttu-id="74dfc-152">A probléma tünete lehet a teljesítmény hirtelen csökkenése vagy a hibák arányának megnövekedése, amelyet a következők valamelyike kísér:</span><span class="sxs-lookup"><span data-stu-id="74dfc-152">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span>

- <span data-ttu-id="74dfc-153">A kivételek számának növekedése, amely arra utal, hogy az erőforrások, például szoftvercsatornák, adatbázis-kapcsolatok, fájlmutatók stb. kimerülőben vannak.</span><span class="sxs-lookup"><span data-stu-id="74dfc-153">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span>
- <span data-ttu-id="74dfc-154">Megnövekedett mértékű memóriahasználat és szemétgyűjtés.</span><span class="sxs-lookup"><span data-stu-id="74dfc-154">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="74dfc-155">A hálózati, meghajtó- és adatbázis-tevékenységek növekedése.</span><span class="sxs-lookup"><span data-stu-id="74dfc-155">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="74dfc-156">A következő lépések végrehajtásával azonosíthatja a problémát:</span><span class="sxs-lookup"><span data-stu-id="74dfc-156">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="74dfc-157">Az éles rendszer folyamatmonitorozásával azonosíthatja azokat a pontokat, ahol a válaszidők lelassulnak vagy a rendszer működése az erőforrások hiányai miatt meghiúsul.</span><span class="sxs-lookup"><span data-stu-id="74dfc-157">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="74dfc-158">Az ezeken a pontokon gyűjtött telemetriaadatok vizsgálatával megállapíthatja, hogy mely műveletek hoznak létre és semmisítenek meg erőforrás-igényes objektumokat.</span><span class="sxs-lookup"><span data-stu-id="74dfc-158">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="74dfc-159">Az egyes gyanús műveletek terheléstesztjét egy kontrollált tesztkörnyezetben végezze el, ne az éles rendszeren.</span><span class="sxs-lookup"><span data-stu-id="74dfc-159">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="74dfc-160">Tekintse át a forráskódot, és vizsgálja meg, hogyan kezeli a rendszer a közvetítő kódokat.</span><span class="sxs-lookup"><span data-stu-id="74dfc-160">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="74dfc-161">Tekintse át a rendszer nagy terheltsége esetén lassan futó vagy kivételeket eredményező műveletek hívásláncait.</span><span class="sxs-lookup"><span data-stu-id="74dfc-161">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="74dfc-162">Ezek az információk segíthetnek megérteni, hogyan használják ezek a műveletek az erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="74dfc-162">This information can help to identify how these operations are using resources.</span></span> <span data-ttu-id="74dfc-163">A kivételek segíthetnek megállapítani, hogy a hibákat a megosztott erőforrások kimerülése okozza-e.</span><span class="sxs-lookup"><span data-stu-id="74dfc-163">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="74dfc-164">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="74dfc-164">Example diagnosis</span></span>

<span data-ttu-id="74dfc-165">Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.</span><span class="sxs-lookup"><span data-stu-id="74dfc-165">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="74dfc-166">A lassulási vagy meghiúsulási pontok azonosítása</span><span class="sxs-lookup"><span data-stu-id="74dfc-166">Identify points of slow down or failure</span></span>

<span data-ttu-id="74dfc-167">Az alábbi ábra a [New Relic APM][new-relic] használatával létrehozott eredményeket mutatja, ahol is a gyenge válaszidejű műveletek láthatók.</span><span class="sxs-lookup"><span data-stu-id="74dfc-167">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="74dfc-168">Ebben az esetben a `NewHttpClientInstancePerRequest` vezérlő `GetProductAsync` metódusát érdemes közelebbről is megvizsgálni.</span><span class="sxs-lookup"><span data-stu-id="74dfc-168">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="74dfc-169">Vegyük észre, hogy ezeknek a műveleteknek a futtatásakor a hibák aránya is nő.</span><span class="sxs-lookup"><span data-stu-id="74dfc-169">Notice that the error rate also increases when these operations are running.</span></span>

![A mintaalkalmazás a New Relic monitorozási irányítópultján, amint egy HttpClient-objektum új példányát hozza létre mindegyik kéréshez][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="74dfc-171">A telemetriaadatok vizsgálata és az összefüggések felderítése</span><span class="sxs-lookup"><span data-stu-id="74dfc-171">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="74dfc-172">A következő képen a szálak profilkészítésével gyűjtött adatok láthatóak ugyanarra az időszakra vonatkozóan, mint az előző képen.</span><span class="sxs-lookup"><span data-stu-id="74dfc-172">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="74dfc-173">A rendszer jelentős mennyiségű időt tölt a szoftvercsatorna-kapcsolatok megnyitásával, és még ennél is többet a bezárásukkal és a szoftvercsatorna-kivételek kezelésével.</span><span class="sxs-lookup"><span data-stu-id="74dfc-173">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![A mintaalkalmazás a New Relic szálprofilkészítőjén, amint egy HttpClient-objektum új példányát hozza létre mindegyik kéréshez][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="74dfc-175">Terhelésteszt végrehajtása</span><span class="sxs-lookup"><span data-stu-id="74dfc-175">Performing load testing</span></span>

<span data-ttu-id="74dfc-176">A terhelésteszttel szimulálhatja a felhasználók által végrehajtható jellemző műveleteket.</span><span class="sxs-lookup"><span data-stu-id="74dfc-176">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="74dfc-177">Ez segíthet azonosítani, hogy a rendszer mely részein tapasztalható erőforrásfogyás a különféle terhelések mellett.</span><span class="sxs-lookup"><span data-stu-id="74dfc-177">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="74dfc-178">A teszteket egy kontrollált környezetben hajtsa végre, ne az éles rendszeren.</span><span class="sxs-lookup"><span data-stu-id="74dfc-178">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="74dfc-179">A következő diagramon a `NewHttpClientInstancePerRequest` vezérlő által kezelt kérések átviteli sebessége látható, ahogy az egyidejű felhasználók száma fokozatosan 100-ra nő.</span><span class="sxs-lookup"><span data-stu-id="74dfc-179">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![A mintaalkalmazás átviteli sebessége, amint egy HttpClient-objektum új példányát hozza létre mindegyik kéréshez][throughput-new-HTTPClient-instance]

<span data-ttu-id="74dfc-181">Eleinte a másodpercenként kezelt kérések mennyisége is nő, ahogy a számítási feladatok száma növekszik.</span><span class="sxs-lookup"><span data-stu-id="74dfc-181">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="74dfc-182">30 felhasználó környékén azonban a sikeres kérések mennyisége elér egy határt, és a rendszerben kivételek kezdenek jelentkezni.</span><span class="sxs-lookup"><span data-stu-id="74dfc-182">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="74dfc-183">Innentől a kivételek mennyisége fokozatosan növekszik a felhasználóterhelés növekedésével együtt.</span><span class="sxs-lookup"><span data-stu-id="74dfc-183">From then on, the volume of exceptions gradually increases with the user load.</span></span>

<span data-ttu-id="74dfc-184">A terhelésteszt ezeket a hibákat HTTP 500-as (belső kiszolgáló) hibákként sorolta be.</span><span class="sxs-lookup"><span data-stu-id="74dfc-184">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="74dfc-185">A telemetria áttekintésével kiderült, hogy a hibákat az okozta, hogy a rendszer szoftvercsatorna-erőforrásai kimerültek, ahogy egyre több és több `HttpClient` objektum jött létre.</span><span class="sxs-lookup"><span data-stu-id="74dfc-185">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="74dfc-186">A következő diagramon egy hasonló teszt látható egy vezérlővel, amely az egyedi `ExpensiveToCreateService` objektumot hozza létre.</span><span class="sxs-lookup"><span data-stu-id="74dfc-186">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![A mintaalkalmazás átviteli sebessége, amint egy ExpensiveToCreateService új példányát hozza létre mindegyik kéréshez][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="74dfc-188">Ezúttal a vezérlő nem ad ki kivételeket, az átviteli sebesség azonban még mindig plafonba ütközik, miközben az átlagos válaszidő a 20-szorosára nő.</span><span class="sxs-lookup"><span data-stu-id="74dfc-188">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="74dfc-189">(A diagramon a válaszidő és az átviteli sebesség logaritmikus skálán van megjelenítve.) A telemetria kimutatta, hogy a probléma elsődleges oka az `ExpensiveToCreateService` új példányainak létrehozása volt.</span><span class="sxs-lookup"><span data-stu-id="74dfc-189">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="74dfc-190">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="74dfc-190">Implement the solution and verify the result</span></span>

<span data-ttu-id="74dfc-191">Miután beállítottuk, hogy a `GetProductAsync` metódus egyetlen `HttpClient` példányt osszon meg, a második terhelésteszt teljesítményjavulást mutatott.</span><span class="sxs-lookup"><span data-stu-id="74dfc-191">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="74dfc-192">A rendszer nem jelzett hibákat, és képes volt kezelni a másodpercenként akár 500 kérésig növő terhelést is.</span><span class="sxs-lookup"><span data-stu-id="74dfc-192">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="74dfc-193">Az átlagos válaszidő az előző teszthez képest a felére csökkent.</span><span class="sxs-lookup"><span data-stu-id="74dfc-193">The average response time was cut in half, compared with the previous test.</span></span>

![A mintaalkalmazás átviteli sebessége, amint egy HttpClient-objektum már meglévő példányát használja az egyes kérésekhez][throughput-single-HTTPClient-instance]

<span data-ttu-id="74dfc-195">Összehasonlításképpen az alábbi ábra a híváslánc-telemetriát mutatja.</span><span class="sxs-lookup"><span data-stu-id="74dfc-195">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="74dfc-196">Ezúttal a rendszer az idő nagy részében valós munkát végez, és nem csupán szoftvercsatornákat nyit meg és zár be.</span><span class="sxs-lookup"><span data-stu-id="74dfc-196">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![A mintaalkalmazás a New Relic szálprofilkészítőjén, amint egy HttpClient-objektum egyetlen példányát hozza létre az összes kéréshez][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="74dfc-198">A következő diagramon egy hasonló terhelésteszt eredményei láthatók az `ExpensiveToCreateService` objektum egy megosztott példányának használatával.</span><span class="sxs-lookup"><span data-stu-id="74dfc-198">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="74dfc-199">Újra azt látjuk, hogy a feldolgozott kérések mennyisége a felhasználóterheléssel arányosan növekszik, míg az átlagos válaszidő alacsony marad.</span><span class="sxs-lookup"><span data-stu-id="74dfc-199">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span>

![A mintaalkalmazás átviteli sebessége, amint egy HttpClient-objektum már meglévő példányát használja az egyes kérésekhez][throughput-single-ExpensiveToCreateService-instance]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
