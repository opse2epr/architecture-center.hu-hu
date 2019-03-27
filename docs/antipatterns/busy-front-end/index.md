---
title: Foglalt előtér kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: 'A nagy mennyiségű háttérbeli szálon végzett aszinkron feladatok elvonhatják az erőforrásokat más, előtérben futó feladatok elől.'
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="busy-front-end-antipattern"></a>Foglalt előtér kizárási minta

A nagy mennyiségű háttérbeli szálon végzett aszinkron feladatok elvonhatják az erőforrásokat más egyidejű előtérbeli feladatok elől, ami elfogadhatatlan mértékben megnöveli a válaszidőket.

## <a name="problem-description"></a>A probléma leírása

A nagy erőforrásigényű feladatok növelhetik a felhasználói kérések válaszidejét, és magas késést eredményezhetnek. A válaszidők javításának egyik módja az, ha a nagy erőforrásigényű feladatokat kiszervezzük egy önálló szálra. Ezzel a megoldással az alkalmazás továbbra is válaszkész maradhat, miközben a feldolgozás a háttérben zajlik. Azonban a háttérbeli szálon futó feladatok továbbra is használnak erőforrásokat. Ha túl sok az ilyen feladat, azok elvonhatják az erőforrásokat azon szálak elől, amelyek a kéréseket kezelik.

> [!NOTE]
> Az *erőforrás* kifejezés sok mindent foglalhat magában, köztük a processzor- és memóriakihasználtságot, illetve a hálózati vagy lemez I/O-t.

Ez a probléma általában akkor fordul elő, ha egy alkalmazás egyetlen nagy kódtömbként jött létre, ahol az összes üzleti logika egy rétegben található, és osztozik a megjelenítési réteggel.

A következő ASP.NET-alapú példa bemutatja a problémát. A teljes kódmintát [itt][code-sample] találja.

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

- A `WorkInFrontEnd` vezérlőben lévő `Post` metódus egy HTTP POST műveletet implementál. Ez a művelet egy hosszan futó, nagy processzorigényű feladatot szimulál. A feladat egy önálló szálon fut, hogy a POST művelet gyorsan befejeződhessen.

- A `UserProfile` vezérlőben lévő `Get` metódus egy HTTP GET műveletet valósít meg. Ennek a metódusnak sokkal kisebb a processzorigénye.

Az elsődleges szempont a `Post` metódus erőforrásigénye. Bár a metódus egy háttérbeli szálra helyezi a feladatot, a feladat így is jelentős mértékű processzor-erőforrásokat használhat. Ezeket az erőforrásokat egyidejűleg más műveletek is használják, amelyeket más felhasználók végeznek. Ha közepes számú felhasználó egyszerre küldi el ezt a kérést, az valószínűleg rontja az összteljesítményt, és lelassítja az összes műveletet. A `Get` metódus használatakor a felhasználók többek között jelentős késleltetéseket tapasztalhatnak.

## <a name="how-to-fix-the-problem"></a>A probléma megoldása

Helyezze át a jelentős erőforrás-használatú folyamatokat egy külön háttérre.

Így az előtér az erőforrás-igényes feladatokat egy üzenetsorba állítja. A háttér felveszi a feladatokat aszinkron feldolgozásra. Az üzenetsor terheléselosztóként is működik, amely puffereli a kéréseket a háttér számára. Ha az üzenetsor túl hosszúra nő, az automatikus skálázás konfigurálható a háttér horizontális felskálázásra.

Az alábbiakban az előző kód átdolgozott verziója látható. Ebben a verzióban a `Post` metódus a Service Bus-üzenetsorban helyez el egy üzenetet.

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

A háttér lekéri az üzeneteket a Service Bus-üzenetsorból, és elvégzi a feldolgozást.

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

## <a name="considerations"></a>Megfontolandó szempontok

- Ez a módszer összetettebbé teszi az alkalmazást. Gondoskodni kell a biztonságos sorba állításról és sorból való eltávolításról, hogy ne vesszenek el a kérések hiba esetén.
- Az alkalmazás függőséget vesz fel egy további szolgáltatásra az üzenetsorhoz.
- A feldolgozási környezetnek megfelelően skálázhatónak kell lennie, hogy képes legyen kezelni a várt számításifeladat-mennyiséget, és teljesíteni tudja az átviteli sebességgel kapcsolatos követelményeket.
- Ennek a megoldásnak összességében növelnie kellene a válaszkészséget, de előfordulhat, hogy a háttérbe áthelyezett feladatok elvégzése hosszabb időt vesz igénybe.

## <a name="how-to-detect-the-problem"></a>A probléma észlelése

Az elfoglalt előtér tünetei közé tartozik a magas válaszidő a nagy erőforrásigényű feladatok végrehajtásakor. A végfelhasználók várhatóan magas válaszidőket vagy a szolgáltatások időtúllépéséből eredő hibákat fognak jelezni. Ezek a hibák HTTP 500-as (belső kiszolgáló) vagy HTTP 503-as (a szolgáltatás nem érhető el) hibaüzeneteket is eredményezhetnek. Vizsgálja át a webkiszolgáló eseménynaplóit, amelyek valószínűleg részletesebb információkat tartalmaznak a hibák okairól és körülményeiről.

A következő lépések végrehajtásával azonosíthatja a problémát:

1. Az éles rendszer folyamatmonitorozásával azonosíthatja azokat a pontokat, ahol a válaszidők lelassulnak.
2. Az ezeken a pontokon gyűjtött telemetriaadatok vizsgálatával megállapíthatja, hogy mely műveletek mennek végbe és mely erőforrások vannak használatban.
3. Megtalálhatja az összefüggéseket a gyenge válaszidők és az adott időpontokban futó műveletek mennyisége és kombinációi között.
4. Végezzen terhelési teszteket a gyanús műveletekkel, így megállapíthatja, hogy mely műveletek használják az erőforrásokat és veszik el azokat más műveletek elől.
5. Tekintse át az adott műveletek forráskódját, amiből kiderülhet, hogy a műveletek miért járnak túlzott erőforráshasználattal.

## <a name="example-diagnosis"></a>Diagnosztikai példa

Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.

### <a name="identify-points-of-slowdown"></a>A lassulási pontok azonosítása

Tagolja az egyes metódusokat, hogy nyomon követhesse az egyes kérések futási idejét és erőforrás-használatát. Ezután monitorozza az alkalmazást éles környezetben. Ezzel átfogó képet kaphat arról, hogy a kérések hogyan versengenek egymással. A nagy nyomással járó időszakokban a lassan futó, nagy erőforrásigényű kérések valószínűleg hatással lesznek a többi műveletre. Ezt a viselkedést úgy figyelheti meg, ha a rendszer monitorozásakor észreveszi a teljesítménycsökkenést.

Az alábbi képen egy monitorozási irányítópult látható. (A tesztekhez az [AppDynamics] segítségével végeztük.) Kezdetben a rendszer terhelése alacsony. Ezután a felhasználók elkezdik lekérni a `UserProfile` GET metódust. A teljesítmény viszonylag jó marad egészen addig, amíg más felhasználók el nem kezdenek kéréseket küldeni a `WorkInFrontEnd` POST metódus számára. Ekkor a válaszidők jelentős mértékben megnövekednek (első nyíl). A válaszidők csak akkor kezdenek el csökkenni, amikor a `WorkInFrontEnd` vezérlő számára küldött kérések mennyisége lecsökken (második nyíl).

![Az AppDynamics Business Transactions (Üzleti tranzakciók) paneljén látható, milyen hatással van az összes kérés válaszidejére a WorkInFrontEnd vezérlő használata][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a>A telemetriaadatok vizsgálata és az összefüggések felderítése

A következő képen néhány olyan mérőszám látható, amelyek ugyanezen időszak alatt az erőforráshasználat monitorozása céljából lettek összegyűjtve. Először kevés felhasználó fér hozzá a rendszerhez. Ahogy további felhasználók csatlakoznak, a processzorkihasználtság rendkívül magasra emelkedik (100%). A processzor kihasználtságával együtt kezdetben a hálózati I/O is megnövekszik. A processzorhasználat tetőzésével azonban a hálózati I/O visszaesik. Ennek az az oka, hogy a rendszer csak viszonylag kevés kérést tud egyszerre kezelni, miután a processzor elérte a maximális kapacitását. Ahogy a felhasználók bontják a kapcsolatot, a processzor terhelése fokozatosan csökken.

![AppDynamics-mérőszámok a processzor és a hálózat kihasználtságáról][AppDynamics-Metrics-Front-End-Requests]

Ekkor úgy tűnik, hogy a `WorkInFrontEnd` vezérlő `Post` metódusát kell közelebbről megvizsgálni. Az elmélet megerősítéséhez további lépések szükségesek ellenőrzött környezetben.

### <a name="perform-load-testing"></a>Terhelési tesztelés végrehajtása

A következő lépés tesztek végrehajtása ellenőrzött környezetben. Például hajtson végre több olyan terhelési tesztet, amelyek először tartalmazzák, majd kihagyják az egyes kéréseket, és ez alapján mérje fel a hatásukat.

Az alábbi grafikonon látható terhelésiteszt-eredmények egy ugyanolyan felhőszolgáltatás üzemelő példányán lettek elvégezve, mint a korábbi tesztek. A tesztben 500 felhasználó hajtotta végre a `Get` műveletet a `UserProfile` vezérlőben, miközben lépéses terhelés is történt, ahol a felhasználók a `Post` műveletet végezték a `WorkInFrontEnd` vezérlőben.

![A WorkInFrontEnd vezérlő terhelési tesztjének kezdeti eredményei][Initial-Load-Test-Results-Front-End]

Kezdetben a lépéses terhelés 0, így az egyedüli aktív felhasználók a `UserProfile` kéréseket hajtják végre. A rendszer körülbelül másodpercenként 500 kérésre képes válaszolni. 60 másodperc után további 100 felhasználó kezd el POST kéréseket küldeni a `WorkInFrontEnd` vezérlőnek. A `UserProfile` vezérlőnek elküldött számításifeladat-mennyiség szinte azonnal másodpercenként 150 kérésre csökken. Ez a terhelési teszt működési mechanizmusa miatt van. A teszt a kérések elküldése előtt megvárja az előző kérdésre kapott választ, ezért minél hosszabb ideig tart a válasz érkezése, annál alacsonyabb lesz a kérések aránya.

Ahogy több felhasználó küld POST kéréseket a `WorkInFrontEnd` vezérlőnek, úgy csökken tovább a `UserProfile` vezérlő válaszadási aránya. Azonban a `WorkInFrontEnd` vezérlő által kezelt kérések száma viszonylag egyenletes marad. Láthatóvá válik a rendszer túltelítődése, ahogy a két kérés összesített sebessége egy egyenletesen alacsony korlát felé tart.

### <a name="review-the-source-code"></a>A forráskód áttekintése

Az utolsó lépés a forráskód áttekintése. A fejlesztőcsapat tisztában van azzal, hogy a `Post` metódus jelentős időt vehet igénybe, ezért az eredeti implementációban egy külön szál lett erre a célra használva. Ezzel a közvetlen probléma megoldódott, mivel a `Post` metódus nem blokkolt le arra várva, hogy egy hosszan futó feladat befejeződjön.

Azonban ez a metódus továbbra is használja a processzort, a memóriát és az egyéb erőforrásokat. A folyamat aszinkron módon való futásának engedélyezése valójában csökkentheti a teljesítményt, mivel a felhasználók nagy mennyiségű ilyen műveletet aktiválhatnak egyszerre, felügyelet nélkül. A kiszolgálók csak véges számú szálat tudnak egyszerre futtatni. Ennek elérése után az alkalmazások valószínűleg kivételt kapnak, ha megpróbálnak elindítani egy új szálat.

> [!NOTE]
> Ez nem jelenti azt, hogy az aszinkron műveleteket kerülni kellene. Az aszinkron várakoztatás végrehajtása a hálózati hívásoknál ajánlott eljárás. (Lásd: [Szinkron I/O][sync-io] kizárási minta.) A probléma itt az, hogy nagy processzorigényű feladatok lettek elindítva egy másik szálon.

### <a name="implement-the-solution-and-verify-the-result"></a>A megoldás megvalósítása és az eredmény ellenőrzése

A következő képen a teljesítmény monitorozása látható a megoldás implementálása után. A terhelés hasonló, mint korábban, de a `UserProfile` vezérlő válaszideje már sokkal rövidebb. Az adott időtartam alatt fogadott kérések száma 2759-ről 23 565-re nőtt.

![Az AppDynamics Business Transactions (Üzleti tranzakciók) paneljén látható, milyen hatással van az összes kérés válaszidejére a WorkInBackground vezérlő használata][AppDynamics-Transactions-Background-Requests]

Emellett a `WorkInBackground` vezérlő sokkal nagyobb mennyiségű kérést kezelt. Ebben az esetben azonban nem lehet közvetlen párhuzamot vonni, mivel e vezérlő feladatvégzése teljesen más, mint az eredeti kód. Az új verzió egyszerűen sorba állít egy kérést, ahelyett, hogy elvégezne egy időigényes számítási feladatot. A fő szempont az, hogy ez a metódus már nem csökkenti az egész rendszer teljesítményét nagy terhelés esetén.

A teljesítményjavulás a processzor és a hálózat kihasználtságában is megmutatkozik. A processzor kihasználtsága egyszer sem érte el a 100%-ot, a kezelt hálózati kérések száma a korábbinál sokkal nagyobb volt, és nem csökkent a számítási feladat befejezéséig.

![AppDynamics-mérőszámok, amelyek a WorkInBackground vezérlő processzor- és hálózatkihasználtságát mutatják][AppDynamics-Metrics-Background-Requests]

A következő grafikon egy terhelési teszt eredményeit mutatja. A kiszolgált kérések teljes mennyisége nagy mértékben nőtt a korábbi tesztekhez képest.

![A BackgroundImageProcessing vezérlő terhelési tesztjének eredményei][Load-Test-Results-Background]

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Automatikus skálázással kapcsolatos ajánlott eljárások][autoscaling]
- [Háttérfeladatokkal kapcsolatos ajánlott eljárások][background-jobs]
- [Üzenetsor-alapú terheléskiegyenlítési minta][load-leveling]
- [Webes üzenetsor-feldolgozó architektúrastílus][web-queue-worker]

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
