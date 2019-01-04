---
title: Foglalt előtér kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: A nagy mennyiségű háttérbeli szálon végzett aszinkron feladatok elvonhatják az erőforrásokat más, előtérben futó feladatok elől.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: f52cedde5a17f098fb9218c48479fae981a2c7df
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011497"
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="d1cc1-103">Foglalt előtér kizárási minta</span><span class="sxs-lookup"><span data-stu-id="d1cc1-103">Busy Front End antipattern</span></span>

<span data-ttu-id="d1cc1-104">A nagy mennyiségű háttérbeli szálon végzett aszinkron feladatok elvonhatják az erőforrásokat más egyidejű előtérbeli feladatok elől, ami elfogadhatatlan mértékben megnöveli a válaszidőket.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="d1cc1-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="d1cc1-105">Problem description</span></span>

<span data-ttu-id="d1cc1-106">A nagy erőforrásigényű feladatok növelhetik a felhasználói kérések válaszidejét, és magas késést eredményezhetnek.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="d1cc1-107">A válaszidők javításának egyik módja az, ha a nagy erőforrásigényű feladatokat kiszervezzük egy önálló szálra.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="d1cc1-108">Ezzel a megoldással az alkalmazás továbbra is válaszkész maradhat, miközben a feldolgozás a háttérben zajlik.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="d1cc1-109">Azonban a háttérbeli szálon futó feladatok továbbra is használnak erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="d1cc1-110">Ha túl sok az ilyen feladat, azok elvonhatják az erőforrásokat azon szálak elől, amelyek a kéréseket kezelik.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="d1cc1-111">Az *erőforrás* kifejezés sok mindent foglalhat magában, köztük a processzor- és memóriakihasználtságot, illetve a hálózati vagy lemez I/O-t.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="d1cc1-112">Ez a probléma általában akkor fordul elő, ha egy alkalmazás egyetlen nagy kódtömbként jött létre, ahol az összes üzleti logika egy rétegben található, és osztozik a megjelenítési réteggel.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="d1cc1-113">A következő ASP.NET-alapú példa bemutatja a problémát.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="d1cc1-114">A teljes kódmintát [itt][code-sample] találja.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="d1cc1-115">A `WorkInFrontEnd` vezérlőben lévő `Post` metódus egy HTTP POST műveletet implementál.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="d1cc1-116">Ez a művelet egy hosszan futó, nagy processzorigényű feladatot szimulál.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="d1cc1-117">A feladat egy önálló szálon fut, hogy a POST művelet gyorsan befejeződhessen.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="d1cc1-118">A `UserProfile` vezérlőben lévő `Get` metódus egy HTTP GET műveletet valósít meg.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="d1cc1-119">Ennek a metódusnak sokkal kisebb a processzorigénye.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="d1cc1-120">Az elsődleges szempont a `Post` metódus erőforrásigénye.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="d1cc1-121">Bár a metódus egy háttérbeli szálra helyezi a feladatot, a feladat így is jelentős mértékű processzor-erőforrásokat használhat.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="d1cc1-122">Ezeket az erőforrásokat egyidejűleg más műveletek is használják, amelyeket más felhasználók végeznek.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="d1cc1-123">Ha közepes számú felhasználó egyszerre küldi el ezt a kérést, az valószínűleg rontja az összteljesítményt, és lelassítja az összes műveletet.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="d1cc1-124">A `Get` metódus használatakor a felhasználók többek között jelentős késleltetéseket tapasztalhatnak.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="d1cc1-125">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="d1cc1-125">How to fix the problem</span></span>

<span data-ttu-id="d1cc1-126">Helyezze át a jelentős erőforrás-használatú folyamatokat egy külön háttérre.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-126">Move processes that consume significant resources to a separate back end.</span></span>

<span data-ttu-id="d1cc1-127">Így az előtér az erőforrás-igényes feladatokat egy üzenetsorba állítja.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="d1cc1-128">A háttér felveszi a feladatokat aszinkron feldolgozásra.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="d1cc1-129">Az üzenetsor terheléselosztóként is működik, amely puffereli a kéréseket a háttér számára.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="d1cc1-130">Ha az üzenetsor túl hosszúra nő, az automatikus skálázás konfigurálható a háttér horizontális felskálázásra.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="d1cc1-131">Az alábbiakban az előző kód átdolgozott verziója látható.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="d1cc1-132">Ebben a verzióban a `Post` metódus a Service Bus-üzenetsorban helyez el egy üzenetet.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span>

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="d1cc1-133">A háttér lekéri az üzeneteket a Service Bus-üzenetsorból, és elvégzi a feldolgozást.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="d1cc1-134">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="d1cc1-134">Considerations</span></span>

- <span data-ttu-id="d1cc1-135">Ez a módszer összetettebbé teszi az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="d1cc1-136">Gondoskodni kell a biztonságos sorba állításról és sorból való eltávolításról, hogy ne vesszenek el a kérések hiba esetén.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="d1cc1-137">Az alkalmazás függőséget vesz fel egy további szolgáltatásra az üzenetsorhoz.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="d1cc1-138">A feldolgozási környezetnek megfelelően skálázhatónak kell lennie, hogy képes legyen kezelni a várt számításifeladat-mennyiséget, és teljesíteni tudja az átviteli sebességgel kapcsolatos követelményeket.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="d1cc1-139">Ennek a megoldásnak összességében növelnie kellene a válaszkészséget, de előfordulhat, hogy a háttérbe áthelyezett feladatok elvégzése hosszabb időt vesz igénybe.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="d1cc1-140">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="d1cc1-140">How to detect the problem</span></span>

<span data-ttu-id="d1cc1-141">Az elfoglalt előtér tünetei közé tartozik a magas válaszidő a nagy erőforrásigényű feladatok végrehajtásakor.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="d1cc1-142">A végfelhasználók várhatóan magas válaszidőket vagy a szolgáltatások időtúllépéséből eredő hibákat fognak jelezni. Ezek a hibák HTTP 500-as (belső kiszolgáló) vagy HTTP 503-as (a szolgáltatás nem érhető el) hibaüzeneteket is eredményezhetnek.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="d1cc1-143">Vizsgálja át a webkiszolgáló eseménynaplóit, amelyek valószínűleg részletesebb információkat tartalmaznak a hibák okairól és körülményeiről.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="d1cc1-144">A következő lépések végrehajtásával azonosíthatja a problémát:</span><span class="sxs-lookup"><span data-stu-id="d1cc1-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="d1cc1-145">Az éles rendszer folyamatmonitorozásával azonosíthatja azokat a pontokat, ahol a válaszidők lelassulnak.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="d1cc1-146">Az ezeken a pontokon gyűjtött telemetriaadatok vizsgálatával megállapíthatja, hogy mely műveletek mennek végbe és mely erőforrások vannak használatban.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span>
3. <span data-ttu-id="d1cc1-147">Megtalálhatja az összefüggéseket a gyenge válaszidők és az adott időpontokban futó műveletek mennyisége és kombinációi között.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="d1cc1-148">Végezzen terhelési teszteket a gyanús műveletekkel, így megállapíthatja, hogy mely műveletek használják az erőforrásokat és veszik el azokat más műveletek elől.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span>
5. <span data-ttu-id="d1cc1-149">Tekintse át az adott műveletek forráskódját, amiből kiderülhet, hogy a műveletek miért járnak túlzott erőforráshasználattal.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="d1cc1-150">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="d1cc1-150">Example diagnosis</span></span>

<span data-ttu-id="d1cc1-151">Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="d1cc1-152">A lassulási pontok azonosítása</span><span class="sxs-lookup"><span data-stu-id="d1cc1-152">Identify points of slowdown</span></span>

<span data-ttu-id="d1cc1-153">Tagolja az egyes metódusokat, hogy nyomon követhesse az egyes kérések futási idejét és erőforrás-használatát.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="d1cc1-154">Ezután monitorozza az alkalmazást éles környezetben.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-154">Then monitor the application in production.</span></span> <span data-ttu-id="d1cc1-155">Ezzel átfogó képet kaphat arról, hogy a kérések hogyan versengenek egymással.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="d1cc1-156">A nagy nyomással járó időszakokban a lassan futó, nagy erőforrásigényű kérések valószínűleg hatással lesznek a többi műveletre. Ezt a viselkedést úgy figyelheti meg, ha a rendszer monitorozásakor észreveszi a teljesítménycsökkenést.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="d1cc1-157">Az alábbi képen egy monitorozási irányítópult látható.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="d1cc1-158">(A tesztekhez az [AppDynamics] segítségével végeztük.) Kezdetben a rendszer terhelése alacsony.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="d1cc1-159">Ezután a felhasználók elkezdik lekérni a `UserProfile` GET metódust.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="d1cc1-160">A teljesítmény viszonylag jó marad egészen addig, amíg más felhasználók el nem kezdenek kéréseket küldeni a `WorkInFrontEnd` POST metódus számára.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="d1cc1-161">Ekkor a válaszidők jelentős mértékben megnövekednek (első nyíl).</span><span class="sxs-lookup"><span data-stu-id="d1cc1-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="d1cc1-162">A válaszidők csak akkor kezdenek el csökkenni, amikor a `WorkInFrontEnd` vezérlő számára küldött kérések mennyisége lecsökken (második nyíl).</span><span class="sxs-lookup"><span data-stu-id="d1cc1-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![Az AppDynamics Business Transactions (Üzleti tranzakciók) paneljén látható, milyen hatással van az összes kérés válaszidejére a WorkInFrontEnd vezérlő használata][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="d1cc1-164">A telemetriaadatok vizsgálata és az összefüggések felderítése</span><span class="sxs-lookup"><span data-stu-id="d1cc1-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="d1cc1-165">A következő képen néhány olyan mérőszám látható, amelyek ugyanezen időszak alatt az erőforráshasználat monitorozása céljából lettek összegyűjtve.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="d1cc1-166">Először kevés felhasználó fér hozzá a rendszerhez.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="d1cc1-167">Ahogy további felhasználók csatlakoznak, a processzorkihasználtság rendkívül magasra emelkedik (100%).</span><span class="sxs-lookup"><span data-stu-id="d1cc1-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="d1cc1-168">A processzor kihasználtságával együtt kezdetben a hálózati I/O is megnövekszik.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="d1cc1-169">A processzorhasználat tetőzésével azonban a hálózati I/O visszaesik.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="d1cc1-170">Ennek az az oka, hogy a rendszer csak viszonylag kevés kérést tud egyszerre kezelni, miután a processzor elérte a maximális kapacitását.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="d1cc1-171">Ahogy a felhasználók bontják a kapcsolatot, a processzor terhelése fokozatosan csökken.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-171">As users disconnect, the CPU load tails off.</span></span>

![AppDynamics-mérőszámok a processzor és a hálózat kihasználtságáról][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="d1cc1-173">Ekkor úgy tűnik, hogy a `WorkInFrontEnd` vezérlő `Post` metódusát kell közelebbről megvizsgálni.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="d1cc1-174">Az elmélet megerősítéséhez további lépések szükségesek ellenőrzött környezetben.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="d1cc1-175">Terhelési tesztelés végrehajtása</span><span class="sxs-lookup"><span data-stu-id="d1cc1-175">Perform load testing</span></span>

<span data-ttu-id="d1cc1-176">A következő lépés tesztek végrehajtása ellenőrzött környezetben.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="d1cc1-177">Például hajtson végre több olyan terhelési tesztet, amelyek először tartalmazzák, majd kihagyják az egyes kéréseket, és ez alapján mérje fel a hatásukat.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="d1cc1-178">Az alábbi grafikonon látható terhelésiteszt-eredmények egy ugyanolyan felhőszolgáltatás üzemelő példányán lettek elvégezve, mint a korábbi tesztek.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="d1cc1-179">A tesztben 500 felhasználó hajtotta végre a `Get` műveletet a `UserProfile` vezérlőben, miközben lépéses terhelés is történt, ahol a felhasználók a `Post` műveletet végezték a `WorkInFrontEnd` vezérlőben.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span>

![A WorkInFrontEnd vezérlő terhelési tesztjének kezdeti eredményei][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="d1cc1-181">Kezdetben a lépéses terhelés 0, így az egyedüli aktív felhasználók a `UserProfile` kéréseket hajtják végre.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="d1cc1-182">A rendszer körülbelül másodpercenként 500 kérésre képes válaszolni.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="d1cc1-183">60 másodperc után további 100 felhasználó kezd el POST kéréseket küldeni a `WorkInFrontEnd` vezérlőnek.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="d1cc1-184">A `UserProfile` vezérlőnek elküldött számításifeladat-mennyiség szinte azonnal másodpercenként 150 kérésre csökken.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="d1cc1-185">Ez a terhelési teszt működési mechanizmusa miatt van.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="d1cc1-186">A teszt a kérések elküldése előtt megvárja az előző kérdésre kapott választ, ezért minél hosszabb ideig tart a válasz érkezése, annál alacsonyabb lesz a kérések aránya.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="d1cc1-187">Ahogy több felhasználó küld POST kéréseket a `WorkInFrontEnd` vezérlőnek, úgy csökken tovább a `UserProfile` vezérlő válaszadási aránya.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="d1cc1-188">Azonban a `WorkInFrontEnd` vezérlő által kezelt kérések száma viszonylag egyenletes marad.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="d1cc1-189">Láthatóvá válik a rendszer túltelítődése, ahogy a két kérés összesített sebessége egy egyenletesen alacsony korlát felé tart.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>

### <a name="review-the-source-code"></a><span data-ttu-id="d1cc1-190">A forráskód áttekintése</span><span class="sxs-lookup"><span data-stu-id="d1cc1-190">Review the source code</span></span>

<span data-ttu-id="d1cc1-191">Az utolsó lépés a forráskód áttekintése.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-191">The final step is to look at the source code.</span></span> <span data-ttu-id="d1cc1-192">A fejlesztőcsapat tisztában van azzal, hogy a `Post` metódus jelentős időt vehet igénybe, ezért az eredeti implementációban egy külön szál lett erre a célra használva.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="d1cc1-193">Ezzel a közvetlen probléma megoldódott, mivel a `Post` metódus nem blokkolt le arra várva, hogy egy hosszan futó feladat befejeződjön.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="d1cc1-194">Azonban ez a metódus továbbra is használja a processzort, a memóriát és az egyéb erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="d1cc1-195">A folyamat aszinkron módon való futásának engedélyezése valójában csökkentheti a teljesítményt, mivel a felhasználók nagy mennyiségű ilyen műveletet aktiválhatnak egyszerre, felügyelet nélkül.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="d1cc1-196">A kiszolgálók csak véges számú szálat tudnak egyszerre futtatni.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="d1cc1-197">Ennek elérése után az alkalmazások valószínűleg kivételt kapnak, ha megpróbálnak elindítani egy új szálat.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="d1cc1-198">Ez nem jelenti azt, hogy az aszinkron műveleteket kerülni kellene.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="d1cc1-199">Az aszinkron várakoztatás végrehajtása a hálózati hívásoknál ajánlott eljárás.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="d1cc1-200">(Lásd: [Szinkron I/O][sync-io] kizárási minta.) A probléma itt az, hogy nagy processzorigényű feladatok lettek elindítva egy másik szálon.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="d1cc1-201">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="d1cc1-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="d1cc1-202">A következő képen a teljesítmény monitorozása látható a megoldás implementálása után.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="d1cc1-203">A terhelés hasonló, mint korábban, de a `UserProfile` vezérlő válaszideje már sokkal rövidebb.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="d1cc1-204">Az adott időtartam alatt fogadott kérések száma 2759-ről 23 565-re nőtt.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span>

![Az AppDynamics Business Transactions (Üzleti tranzakciók) paneljén látható, milyen hatással van az összes kérés válaszidejére a WorkInBackground vezérlő használata][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="d1cc1-206">Emellett a `WorkInBackground` vezérlő sokkal nagyobb mennyiségű kérést kezelt.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="d1cc1-207">Ebben az esetben azonban nem lehet közvetlen párhuzamot vonni, mivel e vezérlő feladatvégzése teljesen más, mint az eredeti kód.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="d1cc1-208">Az új verzió egyszerűen sorba állít egy kérést, ahelyett, hogy elvégezne egy időigényes számítási feladatot.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="d1cc1-209">A fő szempont az, hogy ez a metódus már nem csökkenti az egész rendszer teljesítményét nagy terhelés esetén.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="d1cc1-210">A teljesítményjavulás a processzor és a hálózat kihasználtságában is megmutatkozik.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="d1cc1-211">A processzor kihasználtsága egyszer sem érte el a 100%-ot, a kezelt hálózati kérések száma a korábbinál sokkal nagyobb volt, és nem csökkent a számítási feladat befejezéséig.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![AppDynamics-mérőszámok, amelyek a WorkInBackground vezérlő processzor- és hálózatkihasználtságát mutatják][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="d1cc1-213">A következő grafikon egy terhelési teszt eredményeit mutatja.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="d1cc1-214">A kiszolgált kérések teljes mennyisége nagy mértékben nőtt a korábbi tesztekhez képest.</span><span class="sxs-lookup"><span data-stu-id="d1cc1-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![A BackgroundImageProcessing vezérlő terhelési tesztjének eredményei][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="d1cc1-216">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="d1cc1-216">Related guidance</span></span>

- <span data-ttu-id="d1cc1-217">[Automatikus skálázással kapcsolatos ajánlott eljárások][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="d1cc1-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="d1cc1-218">[Háttérfeladatokkal kapcsolatos ajánlott eljárások][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="d1cc1-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="d1cc1-219">[Üzenetsor-alapú terheléskiegyenlítési minta][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="d1cc1-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="d1cc1-220">[Webes üzenetsor-feldolgozó architektúrastílus][web-queue-worker]</span><span class="sxs-lookup"><span data-stu-id="d1cc1-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: https://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg
