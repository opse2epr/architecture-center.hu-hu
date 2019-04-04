---
title: Szinkron I/O kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: A hívó szál blokkolása az I/O végrehajtása során teljesítménycsökkenést okozhat, és hatással lehet a vertikális méretezhetőségre.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 1b53806b2939a7c44a8b48c9146d5e86c84d9e2e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343560"
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="5dfd5-103">Szinkron I/O kizárási minta</span><span class="sxs-lookup"><span data-stu-id="5dfd5-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="5dfd5-104">A hívó szál blokkolása az I/O végrehajtása során teljesítménycsökkenést okozhat, és hatással lehet a vertikális méretezhetőségre.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="5dfd5-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="5dfd5-105">Problem description</span></span>

<span data-ttu-id="5dfd5-106">Egy szinkron I/O művelet blokkolja a hívó szálat az I/O végrehajtása során.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="5dfd5-107">A hívó szál várakozó állapotba vált, és ez idő alatt nem tud hasznos munkát végezni, ami pazarolja a feldolgozási erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="5dfd5-108">Néhány gyakori példa I/O műveletekre:</span><span class="sxs-lookup"><span data-stu-id="5dfd5-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="5dfd5-109">Adatok lekérése és tárolása egy adatbázisban vagy bármilyen egyéb állandó tárolóban.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="5dfd5-110">Kérések küldése webszolgáltatásoknak.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="5dfd5-111">Üzenetek küldése vagy lekérése az üzenetsorokból.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="5dfd5-112">Helyi fájlok írása és olvasása.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="5dfd5-113">A kizárási minta jellemzően az alábbi okokból következhet be:</span><span class="sxs-lookup"><span data-stu-id="5dfd5-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="5dfd5-114">Ez tűnik egy adott művelet leginkább intuitív megoldási módjának.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-114">It appears to be the most intuitive way to perform an operation.</span></span>
- <span data-ttu-id="5dfd5-115">Az alkalmazásnak egy válaszra van szüksége egy adott kérdésre.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="5dfd5-116">Az alkalmazás egy olyan kódtárat használ, amely csak szinkron módszereket biztosít az I/O műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-116">The application uses a library that only provides synchronous methods for I/O.</span></span>
- <span data-ttu-id="5dfd5-117">Egy külső kódtár belső szinkron I/O műveleteket hajt végre.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="5dfd5-118">Egyetlen szinkron I/O hívás képes egy teljes hívásláncot blokkolni.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="5dfd5-119">Az alábbi kód egy fájlt tölt fel az Azure Blob Storage-ba.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="5dfd5-120">A kódblokkok két helyen várnak szinkron I/O műveleteket: a `CreateIfNotExists` és az `UploadFromStream` metódusban.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="5dfd5-121">Íme egy példa, amely választ vár egy külső szolgáltatástól.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="5dfd5-122">A `GetUserProfile` metódus egy távoli szolgáltatást hív meg, amely egy `UserProfile` profilt ad vissza.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="5dfd5-123">Mindkét példa teljes kódját megtalálja [itt][sample-app].</span><span class="sxs-lookup"><span data-stu-id="5dfd5-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="5dfd5-124">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="5dfd5-124">How to fix the problem</span></span>

<span data-ttu-id="5dfd5-125">Cserélje le a szinkron I/O műveleteket aszinkron műveletekre.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="5dfd5-126">Ez szabaddá teszi az aktuális szálat, amely így hasznos munkát tud végezni a blokkolás helyett, és ez segít javítani a számítási erőforrások kihasználtságát.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="5dfd5-127">Az I/O műveletek aszinkron végrehajtása különösen hatékony megoldás az ügyfélalkalmazásokról érkező kérések hirtelen megszaporodásának lekezelésére.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span>

<span data-ttu-id="5dfd5-128">Számos kódtár szinkron és aszinkron verziókat egyaránt kínál a különféle metódusokhoz.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="5dfd5-129">Amikor csak lehetséges, használja az aszinkron verziókat.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="5dfd5-130">Íme az előbbi példa aszinkron verziója a fájl feltöltéséhez az Azure Blob Storage-ba.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="5dfd5-131">Az `await` operátor visszaadja az irányítást a hívó környezetnek az aszinkron művelet végrehajtása közben.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="5dfd5-132">Az ezt az utasítást követő kód olyan folytatólagos műveleteket tartalmaz, amelyeknek a végrehajtására az aszinkron művelet befejeztével kerül sor.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="5dfd5-133">A jól megtervezett szolgáltatásoknak aszinkron működést is biztosítaniuk kell.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="5dfd5-134">Íme a felhasználói profilokat visszaadó webszolgáltatás aszinkron verziója.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="5dfd5-135">A `GetUserProfileAsync` metódus a Felhasználói profil szolgáltatás aszinkron verzióját használja.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="5dfd5-136">Az olyan kódtárak esetében, amelyek nem biztosítják a műveletek aszinkron verzióit, esetleg létre lehet hozni aszinkron burkolókat egyes szinkron metódusok köré.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="5dfd5-137">Ezt a megközelítést csak óvatosan alkalmazza.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-137">Follow this approach with caution.</span></span> <span data-ttu-id="5dfd5-138">Növelheti ugyan vele az aszinkron burkolót meghívó szál válaszképességét, azonban több erőforrást használ fel.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="5dfd5-139">Előfordulhat, hogy létre kell hozni egy extra szálat, és a szál által végzett munka szinkronizálása többletmunkát jelent.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="5dfd5-140">A megoldás előnyeiről és hátrányairól a következő blogbejegyzés értekezik, amely azzal foglalkozik, hogy [érdemes-e aszinkron burkolókba csomagolni a szinkron metódusokat.][async-wrappers]</span><span class="sxs-lookup"><span data-stu-id="5dfd5-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="5dfd5-141">Íme egy példa egy aszinkron burkolóra egy szinkron metódus körül.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="5dfd5-142">Így a hívó kód most a burkolóra várakozhat:</span><span class="sxs-lookup"><span data-stu-id="5dfd5-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="5dfd5-143">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="5dfd5-143">Considerations</span></span>

- <span data-ttu-id="5dfd5-144">A várhatóan nagyon rövid élettartamú és nem versengő I/O műveletek jobb teljesítményt biztosíthatnak szinkron műveletekként.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="5dfd5-145">Ilyen lehet például egy SSD meghajtón található kisméretű fájlok olvasása.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="5dfd5-146">A feladatok másik szálra küldéséhez, majd a feladat befejeztével a szál szinkronizálásához szükséges többletráfordítás esetleg meghaladja az aszinkron I/O működés előnyeit.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="5dfd5-147">Az ilyen esetek azonban meglehetősen ritkák, és a legtöbb I/O műveletet érdemes aszinkron módon végrehajtani.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="5dfd5-148">Az I/O teljesítmény fokozásával előfordulhat, hogy a rendszer egyéb részei válnak szűk keresztmetszetekké.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="5dfd5-149">Például a szálak blokkolásának megszüntetésével nagyobb egyidejű kérésmennyiség érkezhet a megosztott erőforrásokra, ami erőforráshiányhoz vagy leszabályozáshoz vezethet.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="5dfd5-150">Ha ez problematikus méreteket ölt, előfordulhat, hogy horizontálisan fel kell skáláznia a webkiszolgálók számát, vagy particionálnia kell az adattárakat a versenyhelyzet feloldása érdekében.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="5dfd5-151">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="5dfd5-151">How to detect the problem</span></span>

<span data-ttu-id="5dfd5-152">A felhasználók számára úgy tűnhet, hogy az alkalmazás nem válaszol vagy rendszeresen lefagy.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="5dfd5-153">Az alkalmazás időtúllépési kivételeket produkálhat.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="5dfd5-154">Ezek a hibák HTTP 500 (belső kiszolgáló) hibaüzeneteket eredményezhetnek.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="5dfd5-155">A kiszolgálón a beérkező ügyfélkérések esetleg blokkolva lesznek, amíg egy szál fel nem szabadul, emiatt pedig túl hosszúra nőhetnek a kérésvárólisták, ami HTTP 503-as (a szolgáltatás nem érhető el) hibákat okozhat.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="5dfd5-156">A következő lépések végrehajtásával azonosíthatja a problémát:</span><span class="sxs-lookup"><span data-stu-id="5dfd5-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="5dfd5-157">Az éles rendszer monitorozásával megállapíthatja, hogy a blokkolt feldolgozószálak korlátozzák-e a teljesítményt.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="5dfd5-158">Ha a kérések blokkolásának a szálak hiánya az oka, ellenőrizze az alkalmazást, és állapítsa meg, hogy mely műveletekre jellemző a szinkron I/O működés.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="5dfd5-159">Végezze el az egyes szinkron I/O működésű műveletek szabályozott terheléstesztjét annak megállapítása érdekében, hogy az adott műveletek hatással vannak-e a rendszer teljesítményére.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="5dfd5-160">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="5dfd5-160">Example diagnosis</span></span>

<span data-ttu-id="5dfd5-161">Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="5dfd5-162">A webkiszolgáló teljesítményének monitorozása</span><span class="sxs-lookup"><span data-stu-id="5dfd5-162">Monitor web server performance</span></span>

<span data-ttu-id="5dfd5-163">Az Azure webalkalmazások és webes szerepkörök esetében érdemes monitorozni az IIS webkiszolgáló teljesítményét.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="5dfd5-164">Különösen a kérésvárólisták hosszára érdemes figyelni annak kiderítése érdekében, hogy a nagyon aktív időszakokban az elérhető szálakra való várakozás blokkolja-e a kéréseket.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="5dfd5-165">Ezekre az adatokra az Azure Diagnostics használatának engedélyezésével tehet szert.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="5dfd5-166">További információkért lásd:</span><span class="sxs-lookup"><span data-stu-id="5dfd5-166">For more information, see:</span></span>

- <span data-ttu-id="5dfd5-167">[Alkalmazások monitorozása az Azure App Service-ben][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="5dfd5-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="5dfd5-168">[Teljesítményszámlálók létrehozása és használata Azure-alkalmazásokban][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="5dfd5-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="5dfd5-169">Alakítsa ki úgy az alkalmazást, hogy látható legyen, hogyan kezeli a fogadott kéréseket.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="5dfd5-170">A kérésfolyam nyomon követése segít azonosítani, hogy lassú lefutású hívásokat hajt-e végre és blokkolja-e az aktuális szálat.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="5dfd5-171">A szálakra vonatkozó adatok gyűjtése is rámutathat azokra a kérésekre, amelyek blokkolva vannak.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="5dfd5-172">Az alkalmazás terheléstesztje</span><span class="sxs-lookup"><span data-stu-id="5dfd5-172">Load test the application</span></span>

<span data-ttu-id="5dfd5-173">A következő diagramon a korábban bemutatott szinkron `GetUserProfile` metódus teljesítménye látható változó terhelések mellett, 4000 egyidejű felhasználóig.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="5dfd5-174">Az alkalmazás egy Azure Cloud Services webes szerepkörben futó ASP.NET-alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![A szinkron I/O műveleteket végző mintaalkalmazás teljesítménydiagramja][sync-performance]

<span data-ttu-id="5dfd5-176">A szinkron működés kötelezően 2 másodpercig alszik a szinkron I/O szimulálása végett, így a minimális válaszidő valamivel több, mint 2 másodperc.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="5dfd5-177">Amikor a terhelés eléri a hozzávetőleg 2500 egyidejű felhasználót, az átlagos válaszidő elér egy plafont, bár a kérések másodpercenkénti mennyisége tovább nő.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="5dfd5-178">A két mutató logaritmikus skálán van megjelenítve.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="5dfd5-179">A kérések másodpercenkénti száma ettől a ponttól a teszt végéig a kétszeresére nő.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="5dfd5-180">Ha így önmagában nézzük, ebből a tesztből nem derül ki egyértelműen, hogy a szinkron I/O működés problémát jelent-e.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="5dfd5-181">Nagyobb terhelés esetén az alkalmazás elérhet egy olyan fordulópontot, ahol a webkiszolgáló már nem képes időben feldolgozni a kéréseket, így az ügyfélalkalmazásoknál időtúllépési kivételek jelennek meg.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="5dfd5-182">A beérkező kéréseket az IIS webkiszolgáló várólistára sorolja, és egy, az ASP.NET-szálkészletben futó szálnak osztja ki.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="5dfd5-183">Mivel mindegyik művelet szinkron végzi az I/O tevékenységeket, a szál blokkolva van, amíg a művelet véget nem ér.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="5dfd5-184">A munkaterhelés növekedésével végül az ASP.NET-szálkészletben található minden egyes szál le lesz foglalva és blokkolva lesz.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="5dfd5-185">Ezen a ponton minden további beérkező kérés a várólistában várakozik, amíg egy szál elérhetővé nem válik.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="5dfd5-186">Ahogy nő a várólista hossza, a kérések túllépik az időkorlátot.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="5dfd5-187">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="5dfd5-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="5dfd5-188">A következő diagram a kód aszinkron verzióján végzett terhelésteszt eredményeit mutatja be.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![Az aszinkron I/O műveleteket végző mintaalkalmazás teljesítménydiagramja][async-performance]

<span data-ttu-id="5dfd5-190">A teljesítmény sokkal magasabb.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-190">Throughput is far higher.</span></span> <span data-ttu-id="5dfd5-191">Az előző teszttel egyező időtartam alatt a rendszer mintegy tízszer akkora teljesítményt mutatott a kérések másodpercenkénti számát tekintve.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="5dfd5-192">Emellett az átlagos válaszidő is viszonylag állandó, és hozzávetőleg 25-ször kisebb az előző tesztben mért értékeknél.</span><span class="sxs-lookup"><span data-stu-id="5dfd5-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
