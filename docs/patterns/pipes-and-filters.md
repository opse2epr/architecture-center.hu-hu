---
title: "Adatcsatornák és a szűrők"
description: "Egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat lebontva."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: b41f3e46ad5982a3a4ec6635918481cb440c5e02
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="pipes-and-filters-pattern"></a>Adatcsatornák és a szűrők minta

[!INCLUDE [header](../_includes/header.md)]

Felbontani egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat. Ez javíthatja a teljesítmény, méretezhetőség és újrahasznosításának telepítve, és egymástól függetlenül méretezi feldolgozását végző feladat elemek tételével.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egy alkalmazás, amely feldolgozza az adatokat a különböző, eltérően összetett különböző feladatok elvégzéséhez szükséges. Egy egyszerű, de rugalmatlan megközelítése egy alkalmazást a feldolgozási egységes modulként végrehajtásához. Azonban ez a megközelítés valószínű, hogy csökkentse a lehetőségek a kód újrabontása, optimalizálása, vagy ha azonos feldolgozása részei szükség az alkalmazáson belül máshol újbóli felhasználása.

Az ábra azt mutatja be, milyen probléma lépett fel a egységes megközelítéssel adatok feldolgozása. Az alkalmazás fogad, és két forrásból származó adatokat dolgozza fel. Minden egyes forrásból származó adatokat dolgozza fel ezeket az adatokat, az eredmény az üzleti logika, az alkalmazás való továbbítása előtt átalakítására feladatok sorozatát végző külön modul.

![1. ábra – egységes modulok készletével megvalósított megoldás](./_images/pipes-and-filters-modules.png)

Az egységes modulok végre feladatokat funkcionálisan nagyon hasonló, de a modulok elkülönítetten készített. A kódot, amely a feladatok szorosan alapján kialakulhat egy modulban, és az újbóli vagy a méretezhetőség kevéssé vagy egyáltalán ne gondolat identitáskezelési.

A modulokhoz, vagy a telepítési követelményeket az egyes tevékenységek által végrehajtott feladatokat sikerült azonban módosítható, mert a üzleti követelményeinek megfelelően frissülnek. Egyes feladatok intenzív számítási előfordulhat, hogy és hatékony hardver, futó, míg más előfordulhat, hogy nincs ilyen költséges erőforrások hasznosak. Is a jövőben további feldolgozás lehet szükség, vagy módosíthatja a, amelyben a feladatokat végzi el a feldolgozási sorrendben. A megoldás szükség, hogy ezeket a problémákat, és növeli a kód újbóli lehetőségeit.

## <a name="solution"></a>Megoldás

A feldolgozás le szükséges az egyes adatfolyamokkal külön összetevők (vagy a szűrők), be egy feladat végrehajtása. Minden egyes összetevő fogad és küld adatok formátuma szabványosításával ezek a szűrők is egyesíthet bele a folyamatba. A kód duplikálását elkerülje használatával, és megkönnyíti, hogy távolítsa el, cseréje vagy integrálható a további összetevők, ha az ügyféloldali bővítmények feldolgozási követelményeivel módosítása. Az alábbi ábrán egy pipe-ok és a szűrők használatával megvalósított megoldást mutat.

![2. ábra - pipe-ok és a szűrők használatával megvalósított megoldás](./_images/pipes-and-filters-solution.png)


Egyetlen kérelem feldolgozásához szükséges idő a leglassabb szűrő a feldolgozási sebességétől függ. Egy vagy több szűrőben lehet szűk keresztmetszet, különösen akkor, ha nagyszámú kérelmek egy adott adatforrásból származó adatfolyam jelennek meg. A kulcs az adatcsatorna struktúra előnye teret hagynak a futó párhuzamos példányait lassú szűrők engedélyezésével a rendszer egyenletes terhelését, valamint javítja a teljesítményt biztosít.

A szűrők adatcsatorna alkotó futtathatja eltérő gépeken, engedélyezése, hogy egymástól függetlenül lehet méretezni, és sok felhőkörnyezetekben adja meg, hogy a rugalmasság előnyeinek kihasználása. Egy szűrő, amely számításilag intenzív nagy teljesítményű hardverre, futtathatja, miközben más kevesebb, mint a szűrők igényelnek az alábbiakon tárolható olcsóbb a hagyományos hardvereken. A szűrők nem is kell ugyanabban az adatközpontban vagy földrajzi hely, amely lehetővé teszi, hogy az egyes elemei a folyamat futtatásához olyan környezetben, amelynek mérete megközelítőleg az erőforrásokat igényel.  Az alábbi ábrán látható forrás 1 az adatokat a folyamatot alkalmazza.

![3. ábrán látható egy példa a feldolgozási sor az adatok alkalmazott forrás 1](./_images/pipes-and-filters-load-balancing.png)

Ha a bemeneti és kimeneti egy szűrő felépítése adatfolyamként, akkor lehet minden szűrőhöz párhuzamos feldolgozása végrehajtásához. Az adatcsatorna első szűrő indítsa el a tevékenységeket és a kimeneti az eredményeket, amely átadott közvetlenül be a következő szűrő sorrendben az első szűrő teendőit befejezése előtt.

További előny, a rugalmasságot biztosít. ezt a modellt. Szűrő meghibásodik, vagy a futtató gép már nem érhető el, ha a feldolgozási sor ütemezze újra a szűrő teljesített munkáját, és közvetlen a munka az összetevő egy másik példánya. Sikertelen volt-e egyetlen szűrőt, nem feltétlenül ezért nem a teljes folyamat.

A pipe-ok és a szűrők minta használatával együtt a [Compensating tranzakció mintát](compensating-transaction.md) van egy másik módjáról az elosztott tranzakciók végrehajtására. Elosztott tranzakció bonthatók be különálló, compensable feladatokat, amelyek a tranzakció Compensating mintát megvalósító szűrő használatával valósítható. A szűrők egy folyamaton belül, az adatokat, amelyek megőriznek hamarosan futó külön futtatott feladatok valósítható meg.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ez a kialakítás megvalósítása meghatározásakor a következő szempontokat kell figyelembe vennie:
- **Összetettsége**. A nagyobb rugalmasság, amely ebben a mintában is vezethet összetettségét, különösen akkor, ha egy folyamaton belül szűrők elosztott különböző kiszolgálókon.

- **Megbízhatóság**. Használjon olyan infrastruktúra, amely biztosítja, hogy egy folyamat szűrő áramló adatokat nem lesz elveszett.

- **Idempotencia**. Ha egy szűrőt a folyamat meghiúsul, miután üzenet és a munka átütemezése a szűrő másik példányára, része a munka lehet, hogy rendelkezik már befejeződött. Ha ez a munkahelyi néhány szempontja, hogy a globális állapota (például egy adatbázisában tárolt információit) frissíti, a sikerült ismétlődő ugyanazt a frissítést. Probléma akkor fordulhat elő, szűrő meghibásodásakor könyvelési az eredményeket a következő szűrő, az adatcsatorna, még mielőtt elvégezte teendőit sikeresen jelző után. Ebben az esetben ugyanaz a munkahelyi sikerült meg kell ismételni a szűrőt, hogy a program kétszer ugyanazt az eredményt, amely egy másik példánya. Ez a feldolgozási sorban lévő ugyanazokat az adatokat kétszer a későbbi szűrőket vezethet. Ezért a folyamat a szűrőket úgy kell megtervezni, kell lennie az idempotent. További információ: [idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathan Oliver blogjában.

- **Ismétlődő üzenetek**. Ha egy folyamaton belül szűrő meghiúsul, miután egy üzenetet, amely a feldolgozási folyamat következő szakasza könyvelés, a szűrő egy másik példánya lehet, hogy futtatható, és azt fogja elküldeni ugyanazt az üzenetet, a folyamat egy példányát. Ez ugyanaz az üzenet a következő szűrő átadandó két példánya okozhat. Ennek elkerülése érdekében a feldolgozási sorban lévő kell észlelése és kiküszöbölheti a duplikált üzenetek.

    >  Üzenetsorok (például a Microsoft Azure Service Bus-üzenetsorok) használatával megvalósításához a folyamatot, ha az üzenet üzenetsor-kezelési infrastruktúra nyújthatnak automatikus duplikált üzenetek észlelésének és eltávolítása.

- **És az állapot**. Egy sorban a minden szűrő lényegében elkülönítési fut, és ne ellenőrizze a bármely feltételezéseket hogyan lett meghívva. Ez azt jelenti, hogy minden szűrő a munkájuk elvégzéséhez elegendő kontextusú kell megadni. Ebben a környezetben állapotadatokat nagy mennyiségű tartalmazhatnak.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:
- Az alkalmazás által igényelt feldolgozását is könnyen oszlanak független ismertetett lépések.

- A feldolgozási lépéseket végzi el az alkalmazás különböző méretezhetőség követelményekkel rendelkezik.

    >  Lehetőség biztonságicsoport-szűrőkkel kell méretezni együtt ugyanabban a folyamatban. További információkért lásd: a [számítási erőforrás-összevonási mintát](compute-resource-consolidation.md).

- Rugalmasság szükséges rendelje újra a feldolgozási lépéseket végzi el egy alkalmazást vagy a funkció hozzáadása és eltávolítása a lépéseket a engedélyezéséhez.

- A rendszer vonatkozhat a lépéseket a feldolgozási terjesztése a különféle kiszolgálók között.

- A megbízható megoldás szükség, amely minimálisra csökkenti egy lépésben sikertelensége által okozott hatások, amíg az adatok feldolgozása folyamatban van.

Ebben a mintában előfordulhat, hogy nem lehet hasznos:
- A feldolgozási lépéseket végzi el az alkalmazás nem független, és/vagy együtt ugyanabban a tranzakcióban részeként végrehajtását.

- A lépés szükséges környezet vagy az állapot adatmennyiség nem elég hatékony teszi ezt a módszert használja. Ehelyett megőrizni az adatait az adatbázis esetleg, de nem használja ezt a stratégiát, ha az adatbázis további terhelése túl sok versengés hatására.

## <a name="example"></a>Példa

Üzenet-várólistákból sorozatát segítségével biztosítja a folyamat végrehajtásához szükséges infrastruktúrát. Az első üzenet-várólista feldolgozatlan üzeneteket fogad. Egy szűrő tevékenységként megvalósított összetevő figyeli az ennél a várakozási üzenet, a tevékenységeket hajt végre, és majd küldi az átalakított üzenet a következő várólistára sorrendben. Egy másik szűrő feladat az ennél a várakozási üzeneteket, dolgozza fel őket, és így tovább utáni a eredményeket egy másik várólistához, amíg a teljes átalakított adatok megjelennek-e a várólista utolsó üzenetébe. Az alábbi ábrán láthatja a végrehajtási folyamat üzenetsorok használatával.

![4. ábra - üzenetsorok használatával folyamat végrehajtása](./_images/pipes-and-filters-message-queues.png)


Ha van olyan megoldást épülő Azure Service Bus-üzenetsorok használatával egy olyan mechanizmust, megbízható és méretezhető Üzenetsor-kezelés. A `ServiceBusPipeFilter` osztály a C# alább látható bemutatja, hogyan implementálható, amely megkapja a bemeneti üzenetet az üzenetsorból, ezek az üzenetek folyamatokat, és egy másik várólistához eredmények visszaküldés szűrőt.

>  A `ServiceBusPipeFilter` osztály definiálva van a PipesAndFilters.Shared projekt eléréséhez [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).

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

A `Start` metódust a `ServiceBusPipeFilter` osztály csatlakozik a két bemeneti és kimeneti várólisták, és a `Close` metódus bontja a kapcsolatot a bemeneti várólistát. A `OnPipeFilterMessageAsync` metódus hajtja végre a tényleges feldolgozását az üzeneteket, a `asyncFilterTask` ezt a módszert paraméter határozza meg a feldolgozás végezhető el. A `OnPipeFilterMessageAsync` a bemeneti várólistára, a bejövő üzenetek metódust vár a megadott kód futtatása a `asyncFilterTask` paraméter keresztül minden üzenetet, mert megérkeznek, és az eredményeket a kimeneti várólista küldi. A várólisták magukat a gyártó által meg van adva.

A minta megoldás szűrők feldolgozói szerepkörök meg valósítja meg. Egymástól függetlenül, minden egyes feldolgozói szerepkör is méretezhető, amely elvégzi az üzleti feldolgozása vagy a feldolgozáshoz szükséges erőforrások összetettségétől függően. Emellett minden egyes feldolgozói szerepkör több példánya is futtatható párhuzamos átviteli sebesség növelése érdekében.

A következő kód bemutatja egy Azure feldolgozói szerepkörnek nevű `PipeFilterARoleEntry`az PipeFilterA projekt, a megoldást a definiált.

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

Ez a szerepkör tartalmazza a `ServiceBusPipeFilter` objektum. A `OnStart` metódus a szerepkör a fogadással bemeneti és kimeneti üzenetek közzététele a várólisták csatlakozik (határozzák meg a várólista nevét a `Constants` osztály). A `Run` metódus meghívja a `OnPipeFilterMessagesAsync` egyes feldolgozási végre minden üzenetet kapott metódus (ebben a példában a feldolgozási szimulálják várakozással rövid idő alatt). Ha feldolgozása befejeződött, egy új üzenet összeállított az eredményeket tartalmazó (a bemeneti üzenet van ebben az esetben a hozzáadott egyéni tulajdonság is), és ezt az üzenetet a rendszer visszaküldi a kimeneti várólistában.

A mintakód tartalmaz egy másik feldolgozói szerepkör nevű `PipeFilterBRoleEntry` a PipeFilterB projektben. Ez a szerepkör hasonlít `PipeFilterARoleEntry` azzal a különbséggel, hogy a különböző feldolgozó segítségével végzi a `Run` metódust. A példa a megoldásban ez a két szerepkör kombinált egy folyamatot, a kimeneti várólista összeállítani a `PipeFilterARoleEntry` szerepköre a bemeneti várólista a `PipeFilterBRoleEntry` szerepkör.

A minta megoldás is biztosít két további szerepkörök nevű `InitialSenderRoleEntry` (a projektben InitialSender) és `FinalReceiverRoleEntry` (a projektben FinalReceiver). A `InitialSenderRoleEntry` szerepkör szolgáltatás a kezdeti üzenetet jelenít meg a folyamat. A `OnStart` módszer egy adott sorba csatlakozik, és a `Run` metódus által a várólistának metódus. A várólista által használt bemeneti várólista a `PipeFilterARoleEntry` szerepkör, ezért fogad és dolgoz fel üzenetet egy üzenet azt eredményezi a `PipeFilterARoleEntry` szerepkör. A feldolgozott üzenet majd haladnak keresztül az `PipeFilterBRoleEntry` szerepkör.

A bemeneti várólista a `FinalReceiveRoleEntry` szerepköre a kimeneti várólista a `PipeFilterBRoleEntry` szerepkör. A `Run` metódust a `FinalReceiveRoleEntry` szerepkör az alábbi megkapja az üzenetet, és néhány végső feldolgozásra. Ezután ír a követés eredményét a szűrők a folyamat által hozzáadott egyéni tulajdonságok értékeit.

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

##<a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:
- Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).
- [Versengő fogyasztók mintát](competing-consumers.md). Egy folyamat egy vagy több szűrő több példányát is tartalmazhat. Ez a megközelítés futó párhuzamos példányait lassú szűrők engedélyezésével a rendszer egyenletes terhelését, valamint javítja a teljesítményt. Szűrő minden példánya a rendszer "versenyeznek" az bemeneti a más osztályt, szűrő két példánya nem fogja tudni feldolgozni ugyanazokat az adatokat. Ez a megközelítés magyarázattal szolgál.
- [Számítási erőforrás-összevonási mintát](compute-resource-consolidation.md). Biztonságicsoport-szűrőkkel, amelyek ugyanabba a folyamatba együtt kell méretezni is lehet. További információt az előnyöket és a stratégia kompromisszumot nyújt.
- [Kompenzációs tranzakció mintát](compensating-transaction.md). Egy szűrő, amely visszafordítható, illetve egy kompenzációs művelet, amely visszaállítja az állapotát egy korábbi verzióját meghibásodása lett műveletként valósítható meg. Ismerteti, hogyan ez valósítható karbantartása, vagy a végleges konzisztencia elérése.
- [Idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathan Oliver blogjában.
