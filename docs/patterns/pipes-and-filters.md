---
title: Csövek és szűrők mintája
titleSuffix: Cloud Design Patterns
description: Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 7084b538159f7104d2322e35f94f43e905f700bf
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011684"
---
# <a name="pipes-and-filters-pattern"></a>Csövek és szűrők mintája

[!INCLUDE [header](../_includes/header.md)]

Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává. Ezzel javítható a teljesítmény, a skálázhatóság és az újrahasznosíthatóság, mivel lehetővé teszi a feldolgozást végső feladatelemek független üzembe helyezését és skálázását.

## <a name="context-and-problem"></a>Kontextus és probléma

Az alkalmazásnak több, változó összetettségű feladatot kell végrehajtania a feldolgozandó adatokon. Egy egyszerű, de rugalmatlan megközelítés az alkalmazások megvalósításához, ha ezt a feldolgozást egy monolitikus modulként hajtatjuk végre. Ez a megközelítés azonban valószínűleg csökkenti a kód átdolgozásának, optimalizálásának vagy ismételt felhasználásának lehetőségét, amennyiben ugyanilyen feldolgozás máshol is szükséges lenne az alkalmazásban.

Az ábra az adatfeldolgozással kapcsolatos problémákat szemlélteti a monolitikus megközelítés alkalmazásakor. Az alkalmazások két forrásból fogadnak és dolgoznak fel adatokat. Az egyes forrásokból származó adatok feldolgozása egy-egy különálló modul feladata, amely elvégzi az adatok átalakításához szükséges feladatokat, majd az eredményt az alkalmazás üzleti logikája számára továbbítja.

![1. ábra: Monolitikus modulok használatával implementált megoldás](./_images/pipes-and-filters-modules.png)

A monolitikus modulok által elvégzett feladatok egy része funkcionálisan nagyon hasonló, de az egyes modulok megtervezése külön-külön történt. A feladatokat implementáló kód szorosan össze van kapcsolva a modulokban, és annak kifejlesztése során kevésbé vagy egyáltalán nem vették figyelembe az újrafelhasználhatóságot vagy a skálázhatóságot.

Az egyes modulok által végrehajtott feldolgozási feladatok, illetve az egyes feladatok üzembe helyezési követelményei azonban megváltozhatnak az üzleti követelmények változásával. Egyes feladatok nagy számításigényűek lehetnek, így nagy teljesítményű hardveren futtathatók hatékonyan, míg mások nem igényelnek ennyire költséges erőforrásokat. A jövőben azonban további feldolgozásra is szükség lehet, vagy módosulhat a feladatok feldolgozási sorrendje. Olyan megoldásra van szükség, amellyel megoldhatók ezek a problémák, és nő a kód újrafelhasználhatóságának lehetősége.

## <a name="solution"></a>Megoldás

Ossza fel az egyes streamekhez tartozó feldolgozási folyamatot olyan különálló összetevőkre (vagy szűrőkre), amelyek mindegyike egyetlen feladatot végez. Az egyes összetevők által küldött és fogadott adatok formátumának szabványosításával ezek a szűrők egy folyamattá kombinálhatóak. Ezzel elkerülhető a kód ismétlődése, illetve megkönnyíti további összetevők eltávolítását, cseréjét vagy integrálását, ha a feldolgozási követelmények változnak. A következő ábra egy csövek és szűrők használatával implementált megoldást mutat be.

![2. ábra – Csövek és szűrők használatával implementált megoldás](./_images/pipes-and-filters-solution.png)

Az egy kérelem feldolgozásához szükséges idő a folyamat leglassabb szűrőjének sebességétől függ. Egy vagy több szűrő szűk keresztmetszetté válhat, különösen akkor, ha nagy számú kérelem található egy adott adatforrástól érkező streamben. A folyamatstruktúra legfontosabb előnye, hogy lehetőséget biztosít a lassú szűrők példányainak egyidejű futtatására, lehetővé téve ezzel a rendszer számára a terhelés elosztását és a teljesítmény növelését.

A folyamatot alkotó szűrők különböző gépeken futhatnak, így egymástól függetlenül skálázhatóak, és kihasználhatják a számos felhőkörnyezet által nyújtott rugalmasságot. A nagy számítási igényű szűrők nagy teljesítményű hardveren futhatnak, míg más, kevésbé erőforrás-igényes szűrők üzemeltethetők kevésbé költséges, hagyományos hardveren is. A szűrőknek még csak nem is kell azonos adatközpontban vagy földrajzi helyen lenniük, ami lehetővé teszi, hogy az egyes folyamatelemek olyan környezetben futhassanak, amely közel van a számukra szükséges erőforrásokhoz.  A következő ábrán látható példa az 1. forrásból származó adatokhoz tartozó folyamatra van alkalmazva.

![A 3. ábrán látható példa az 1. forrásból származó adatokhoz tartozó folyamatra van alkalmazva](./_images/pipes-and-filters-load-balancing.png)

Ha a szűrők be- és kimenete streamként van felépítve, akkor lehetséges az egyes szűrők általi egyidejű feldolgozás is. A folyamat első szűrője elkezd működni, és létrehozza a kimenetét, amelyet a rendszer még az előtt a soron következő szűrőnek továbbít, hogy az első szűrő végrehajtaná a feladatát.

További előnyt jelent a modell által biztosított rugalmasság. Ha egy szűrő meghibásodik vagy ha az azt futtató gép már nem érhető el, akkor a folyamat képes átütemezni a szűrő által végrehajtott feladatot és az összetevő másik példányára átirányítani. Egyetlen szűrő meghibásodása nem feltétlenül eredményezi a teljes folyamat meghibásodását.

A Csövek és szűrők minta és a [Kompenzáló tranzakció minta](./compensating-transaction.md) együttes alkalmazása is egy lehetséges alternatív módszer az elosztott tranzakciók implementálására. Az elosztott tranzakciók különálló, kompenzálható feladatokra bonthatók le, amelyek egy olyan szűrővel implementálhatók, amely a Kompenzáló tranzakció mintát is implementálja. A folyamatban lévő szűrők olyan önállóan üzemeltetett feladatokként is implementálhatók, amelyeket a rendszer az általuk karbantartott adatok közelében futtat.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- **Összetettség**. A minta által biztosított megnövelt rugalmasság összetettséget is eredményezhet, különösen akkor, ha a folyamat szűrői különböző kiszolgálókon vannak elosztva.

- **Megbízhatóság**. Olyan infrastruktúrát használjon, amely biztosítja, hogy ne legyen adatvesztés a folyamat szűrői közötti adatátvitel során.

- **Idempotencia**. Ha egy üzenet fogadását követően a folyamatban lévő szűrő meghibásodik, és a rendszer a feladatot a szűrő egy másik példányára ütemezi át, akkor lehetséges, hogy a feladat egy része már végre lett hajtva. Ha ez a feladat frissíti a globális állapot egyes aspektusait (például az adatbázisban tárolt információkat), akkor meg lehet ismételni ugyanezt a frissítést. Hasonló probléma fordulhat elő akkor, ha egy szűrő azután hibásodik meg, miután közzétette az eredményeit a folyamat következő szűrőjének, de még nem jelezte, hogy a feladatot sikeresen elvégezte. Ezekben az esetekben lehetséges, hogy ugyanazt a feladatot a szűrő egy másik példánya is elvégzi, így ugyanazok az eredmények kétszer lesznek közzétéve. Ez azt eredményezheti, hogy a folyamat soron következő szűrői ugyanazokat az adatokat kétszer fogják feldolgozni. Ezért a folyamatok szűrőinek idempotensnek kell lenniük. További információ: [Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) (Jonathan Oliver blogjában).

- **Ismétlődő üzenetek**. Ha egy folyamat szűrője azután hibásodik meg, hogy közzétett egy üzenetet a folyamat következő szakaszában, akkor a rendszer futtathatja a szűrő másik példányát, és az ugyanennek az üzenetnek a másolatát fogja közzétenni a folyamatban. Ez azt eredményezheti, hogy ugyanazon üzenet két példánya lesz elküldve a következő szűrő számára. Ennek elkerülése érdekében a folyamatnak észlelnie kell és el kell távolítania az ismétlődő üzeneteket.

    >  Ha az adott folyamatot üzenetsorok (például Microsoft Azure Service Bus-üzenetsorok) használatával implementálja, az üzenetsor-kezelési infrastruktúra az ismétlődő üzenetek automatikus felismerését és eltávolítását biztosíthatja.

- **Kontextus és állapot**. A folyamatokban a szűrők gyakorlatilag elkülönítve futnak, és nem tudják feltételezni azok aktiválásuk módját. Ez azt jelenti, hogy mindegyik szűrő számára a feladatának elvégzéséhez megfelelő kontextust kell biztosítani. A kontextus nagy mennyiségű állapotinformációt is tartalmazhat.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Az alkalmazások számára szükséges feldolgozási folyamat egyszerűen lebontható független lépések sorozatára.

- Az alkalmazások által végrehajtott feldolgozási lépések különböző skálázhatósági követelményekkel rendelkeznek.

    >  Lehetőség van csoportosítani olyan szűrőket, amelyeknek azonos skálázásúaknak kell lenniük egy adott folyamatban. További információkért lásd a [számításierőforrás-konszolidálási mintát](./compute-resource-consolidation.md).

- Rugalmas kialakítás szükséges ahhoz, hogy lehetővé váljon az alkalmazás által végrehajtott feldolgozási lépések sorrendjének módosítása, illetve lépések hozzáadása és eltávolítása.

- A rendszer számára előnyös lehet, ha az egyes lépésekhez tartozó feldolgozási folyamatok különböző kiszolgálókra vannak elosztva.

- Olyan megbízható megoldásra van szükség, amely minimálisra csökkenti egy adott lépés meghibásodásának hatását az adatok feldolgozása során.

Nem érdemes ezt a mintát használni, ha:

- Az alkalmazások által végrehajtott feldolgozási lépések nem függetlenek egymástól, illetve azokat egy adott tranzakció részeként, együtt kell végrehajtani.

- Az egyes lépések végrehajtásához szükséges kontextus- vagy állapotinformációk mennyisége csökkenti ennek a módszernek a hatékonyságát. Az állapotinformációkat továbbra is lehet inkább adatbázisban megőrizni, de ne használja ezt a stratégiát, ha az adatbázis további terhelése túlzott mértékű versenyt okozna.

## <a name="example"></a>Példa

A folyamat implementálásához szükséges infrastruktúra biztosításához üzenetsorok sorozatát is használhatja. A kezdő üzenetsor feldolgozatlan üzeneteket kap. A szűrési feladatként megvalósított összetevő figyeli az adott üzenetsor üzeneteit, elvégzi a feladatát, majd közzéteszi az átalakított üzenetet a sorban következő üzenetsornak. Egy másik szűrési feladat figyelheti ennek az üzenetsornak az üzeneteit, feldolgozhatja azokat, majd elküldheti az eredményeket egy másik üzenetsornak, és így tovább, amíg a teljes körűen átalakított adatok meg nem jelennek az üzenetsor végső üzenetében. A következő ábrán egy üzenetsorok használatával megvalósított folyamat látható.

![4. ábra – Üzenetsorok használatával megvalósított folyamat](./_images/pipes-and-filters-message-queues.png)

Ha a megoldást az Azure-ban hozza létre, a Service Bus-üzenetsorok megbízható és skálázható üzenetsor-kezelési mechanizmust biztosítanak. Az alábbi, C# nyelven írt `ServiceBusPipeFilter` osztály példáján egy olyan szűrő implementálását mutatjuk be, amely a bemeneti üzeneteket egy üzenetsorból kapja, feldolgozza azokat, majd az eredményeket egy másik üzenetsorban teszi közzé.

> A `ServiceBusPipeFilter` osztály a PipesAndFilters.Shared projektben van meghatározva, amely a [GitHubról](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters) érhető el.

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

A `ServiceBusPipeFilter` osztály `Start` metódusa be- és kimeneti üzenetsorpárhoz csatlakozik, a `Close` metódus pedig lecsatlakozik a bemeneti üzenetsorról. Az `OnPipeFilterMessageAsync` metódus végzi el az üzenetek tényleges feldolgozását, a metódus `asyncFilterTask` paramétere pedig meghatározza a végrehajtandó feldolgozást. Az `OnPipeFilterMessageAsync` metódus a bemeneti üzenetsor bejövő üzeneteire vár, azok beérkezési sorrendjében futtatja az `asyncFilterTask` paraméter által megadott kódot az üzeneteken, végül a kimeneti üzenetsoron teszi közzé az eredményeket. Magukat az üzenetsorokat a konstruktor adja meg.

A mintául szolgáló megoldás szűrőket implementál feldolgozói szerepkörökben. Az egyes feldolgozói szerepkörök egymástól függetlenül skálázhatók az általuk végzett üzleti feldolgozás összetettségétől vagy a feldolgozáshoz szükséges erőforrásoktól függően. Ezenkívül az egyes feldolgozói szerepkörök több példánya párhuzamosan is futtatható a teljesítmény növelése érdekében.

Az alábbi kód egy `PipeFilterARoleEntry` nevű Azure-beli feldolgozói szerepkörre mutat példát, amelynek meghatározása a mintamegoldás PipeFilterA projektjében történt.

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

Ez a szerepkör tartalmaz egy `ServiceBusPipeFilter` objektumot. A szerepkör `OnStart` metódusa a bemeneti üzenetek fogadásához és a kimeneti üzenetek közzétételéhez csatlakozik az üzenetsorokhoz (az üzenetsorok nevei a `Constants` osztályban vannak meghatározva). A `Run` metódus meghívja az `OnPipeFilterMessagesAsync` metódust a fogadott üzenetek bizonyos mértékű feldolgozására (ebben a példában a feldolgozást rövid idejű várakozással szimuláljuk). Amikor a feldolgozás befejeződött, a rendszer egy új üzenetet állít össze, amely tartalmazza az eredményeket (ebben az esetben a bemeneti üzenet egy hozzáadott egyéni tulajdonsággal rendelkezik) és a kimeneti üzenetsorban lesz közzétéve.

A mintakód egy másik, `PipeFilterBRoleEntry` nevű feldolgozói szerepkört is tartalmaz a PipeFilterB projektben. Ez a szerepkör hasonlít a `PipeFilterARoleEntry` szerepkörhöz azzal a különbséggel, hogy eltérő feldolgozási műveletet hajt végre a `Run` metódusban. A példában szereplő megoldásban ennek a két szerepkörnek a kombinálásával hozunk létre egy folyamatot, ahol a `PipeFilterARoleEntry` szerepkör kimeneti üzenetsora lesz a `PipeFilterBRoleEntry` szerepkör bemeneti üzenetsora.

A mintamegoldás két további szerepkört is biztosít: `InitialSenderRoleEntry` (az InitialSender projektben) és `FinalReceiverRoleEntry` (a FinalReceiver projektben). A folyamat kezdő üzenetét az `InitialSenderRoleEntry` szerepkör biztosítja. Az `OnStart` metódus egyetlen üzenetsorhoz kapcsolódik, és ezen az üzenetsoron a `Run` metódus közzétesz egy metódust. Ezt az üzenetsort használja a `PipeFilterARoleEntry` szerepkör bemeneti üzenetsorként, így az itt közzétett üzenetek fogadását és feldolgozását a `PipeFilterARoleEntry` szerepkör végzi. A feldolgozott üzenet ezt követően a `PipeFilterBRoleEntry` szerepkörön halad át.

A `FinalReceiveRoleEntry` szerepkör bemeneti üzenetsora a `PipeFilterBRoleEntry` szerepkör kimeneti üzenetsora. Az üzenetet a `FinalReceiveRoleEntry` szerepkör `Run` metódusa fogadja (lásd alább), és hajtja végre rajta a végső feldolgozási műveleteket. Ezt követően a metódus a folyamat szűrői által hozzáadott egyéni tulajdonságokat a nyomkövetési kimenetbe írja.

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters) talál egy, a minta bemutatására szolgáló példát.
- [Versengő felhasználókat ismertető minta](./competing-consumers.md). A folyamatok adott szűrők több példányát is tartalmazhatják. Ezen megközelítés lehetőséget biztosít a lassú szűrők példányainak egyidejű futtatására, lehetővé téve ezzel a rendszer számára a terhelés elosztását és a teljesítmény növelését. A szűrőpéldányok versenyezni fognak egymással a bemeneti adatokért, egy adott szűrő két példánya nem dolgozhatja fel ugyanazokat az adatokat. Megmagyarázza ezt a megközelítést.
- [Számításierőforrás-konszolidálási minta](./compute-resource-consolidation.md). Lehetőség van csoportosítani olyan szűrőket, amelyeknek azonos skálázásúaknak kell lenniük egy adott folyamatban. További információkat biztosít ezen stratégia előnyeiről és hátrányairól.
- [Kompenzáló tranzakció mintája](./compensating-transaction.md). A szűrők implementálhatóak visszavonható műveletként is, vagy olyan kompenzáló műveletként, amely hiba esetén visszaállít egy korábbi verzió szerinti állapotot. Ismerteti, hogy ez a módszer hogyan implementálható a végső konzisztencia fenntartásához vagy eléréséhez.
- [Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) (Jonathan Oliver blogjában).
