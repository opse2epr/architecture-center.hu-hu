---
title: Versengő fogyasztó számára
description: Engedélyezze a több egyidejű fogyasztók ugyanazt a üzenetkezelési csatornát a Beérkezett üzenetek feldolgozásához.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
ms.openlocfilehash: d72a09ef7613bebe3701634e4eac0716400e471d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="competing-consumers-pattern"></a>Versengő fogyasztók minta

[!INCLUDE [header](../_includes/header.md)]

Engedélyezze a több egyidejű fogyasztók ugyanazt a üzenetkezelési csatornát a Beérkezett üzenetek feldolgozásához. Ez lehetővé teszi, hogy a rendszer több üzenetek egyidejűleg a teljesítmény, méretezhetőség és a rendelkezésre állás javítása érdekében, és a terhelés elosztása érdekében optimalizálása érdekében.

## <a name="context-and-problem"></a>A környezetben, és probléma

A felhőben futó alkalmazáshoz várhatóan sok kérelmek kezeléséhez. Helyett az egyes kérelmek szinkron módon feldolgozni, egy általános módszer van, az alkalmazás egy üzenetkezelési rendszerek őket átadása egy másik (egy fogyasztó) szolgáltatást, amely aszinkron módon kezeli őket. Ez a stratégia segít annak érdekében, hogy az üzleti logika, az alkalmazás nem blokkolja-e a kérelem feldolgozása során.

Kérelmek száma az idők során számos oka nagymértékben változhat. Váratlan növekedését riasztásviharnak felhasználói tevékenység vagy több bérlő érkező összesített kérelmek egy előre nem látható munkaterhelés okozhat. Csúcsidőszakon, a rendszer módosítania kell feldolgozni a több száz kérelmek / másodperc, amíg a többi időszakban a szám nagyon kis lehet. Ezenkívül ezek a kérelmek kezelésére végzett munka jellege magas változó lehet. A fogyasztó szolgáltatás egyetlen példányát használja okozhat azt válnak árasztott kérések a példányt, vagy a levelezőrendszerrel túlterheltek egy beérkezése az alkalmazáshoz érkező üzenetek által. Változó munkaterhelés kezeléséhez, a rendszer több példánya is futtatható a fogyasztói szolgáltatás. A fogyasztók számára, össze kell hangolni győződjön meg arról, hogy minden üzenetet csak a rendszer egy-egy fogyasztó számára. A munkaterhelés kell szerepelnie terhelésű megakadályozhatja, hogy a példány a szűk keresztmetszetek váljon a felhasználók között.

## <a name="solution"></a>Megoldás

Az üzenet-várólista segítségével valósítja meg az alkalmazás és a fogyasztói szolgáltatás példányainak közötti kommunikációs csatornát. Az alkalmazás által a várólista-üzenetek formájában kéréseket, és a fogyasztói szolgáltatáspéldány üzenetek fogadása az üzenetsorból, és dolgozza fel őket. Ez a megközelítés lehetővé teszi, hogy a fogyasztói szolgáltatáspéldány kezelni az alkalmazás összes példányát üzeneteit azonos erőforráskészletben. Az ábra azt mutatja be, egy üzenet-várólista segítségével kiosztani a munkát egy szolgáltatás példányának.

![Üzenet-várólista használata egy szolgáltatás példányának munkát terjesztése](./_images/competing-consumers-diagram.png)

Ez a megoldás rendelkezik a következő előnyöket biztosítja:

- A betöltési simítva rendszer, amelyet kezelni tud a nagy mennyiségű kérést alkalmazáspéldányok által küldött szórás biztosít. A várólista pufferként a az alkalmazáspéldányok és a fogyasztói szolgáltatáspéldányok között. Ez segít a rendelkezésre állási és az alkalmazás és a szolgáltatáspéldány reakcióidőt gyakorolt hatás minimalizálása érdekében, szerint a [terhelés simítás várólista alapú mintát](queue-based-load-leveling.md). Egy üzenet egyes hosszan futó feldolgozási nem akadályozza más üzenetek egyidejűleg más esetekben a fogyasztói szolgáltatás által kezelt igénylő kezelése.

- Ez növeli a megbízhatóságot. Ha termelő közvetlenül kommunikál a fogyasztó helyett ezt a mintát használja, de nem figyelheti a fogyasztónak, akkor nagy valószínűséggel, hogy a üzenetek veszni sikerült-e vagy nem dolgozható fel, ha a fogyasztó nem sikerült. Ebben a mintában üzeneteket a megadott példánya nem küldeni. A sikertelen példánya nem blokkolja a gyártó, és bármely működő szolgáltatáspéldány feldolgozható üzenetek.

- Összetett koordinációs felhasználói között, vagy a gyártót és a felhasználói példányok nem igényel. Az üzenet-várólista biztosítja, hogy minden üzenet legalább egyszer kerül.

- Is méretezhető. A rendszer dinamikusan növelheti vagy csökkentheti a fogyasztói szolgáltatás példányainak száma, az üzenetek a kötet ingadozik.

- Rugalmasság azt javítható, ha az üzenet-várólista tranzakciós olvasási műveletek biztosít. Ha a fogyasztó példánya olvas, és feldolgozza az üzenetet egy tranzakciós művelet részeként, és a fogyasztói szolgáltatáspéldány meghibásodik, ebben a mintában bizonyosodjon meg arról, hogy az üzenet vissza a várólistában, felvenni, és egy másik példánya kezeli a ügyfélszolgálat.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- **Üzenet rendelési**. A sorrendet, amelyben a fogyasztó szolgáltatáspéldány üzeneteket fogadni, nem garantált, és nem tükrözi a sorrendben, ahol az üzenetek hozta létre. Tervezze meg a rendszer annak érdekében, hogy az üzenet feldolgozása-e az idempotent, mert ez hozzájárul a sorrendet, amelyben üzenetek kezelésének bármely függőség megszüntetéséhez. További információkért lásd: [idempotencia minták](http://blog.jonathanoliver.com/idempotency-patterns/) Jonathon Oliver blogjában.

    > A Microsoft Azure Service Bus-üzenetsorok garantált első-first out rendelési üzenetek üzenet munkamenetek használatával is létrehozható. További információkért lásd: [minták használatával munkamenetek üzenetküldési](https://msdn.microsoft.com/magazine/jj863132.aspx).

- **A rugalmasságot szolgáltatások tervezése**. Ha a rendszer célja, hogy észleli, és indítsa újra a sikertelen szolgáltatáspéldány, lehet, végre kell hajtani a szolgáltatáspéldány végzi el az idempotent műveletként, ha csökkenteni szeretné a lekérését és feldolgozott több különálló üzenet feldolgozása egynél többször.

- **Az elhalt üzenetek észlelésére**. Egy helytelenül formázott üzenetet, vagy erőforrásokat, amelyek nem érhetők el, a hozzáférést igénylő okozhat a szolgáltatáspéldány sikertelen lesz. A rendszer kell megakadályozza az ilyen üzeneteket ad vissza a várólistára, és helyette, majd tárolhatja ezeket az üzeneteket részleteit máshol, hogy az elemzett szükség esetén.

- **Eredmények kezelése**. A szolgáltatáspéldány, egy üzenet kezelése teljes mértékben a rendszer leválasztja az alkalmazáslogikát, amely létrehozza az üzenetet, és előfordulhat, hogy nem tudnak közvetlenül kommunikálnak. A szolgáltatáspéldány eredmények, amelyek kell átadni vissza az alkalmazáslogikát állít elő, ha ezeket az információkat egyaránt elérhető helyen kell tárolni. Leállítja, nehogy a úgy az alkalmazáslogikát lekérése során a rendszer tartalmaznia kell feldolgozásakor nem teljes adatokat befejeződött.

     > Azure használata, ha egy munkavégző folyamat képes továbbadni eredmények vissza az alkalmazáslogikát egy dedikált üzenet válasz várólista használatával. Az alkalmazás logikáját ezekkel az eredményekkel összefüggéseket az eredeti üzenet képesnek kell lennie. Ebben a forgatókönyvben a további részletes leírását lásd a [aszinkron üzenetkezelési ismertetése](https://msdn.microsoft.com/library/dn589781.aspx).

- **Az üzenetkezelési rendszeréhez skálázás**. A felügyeleti teendők központjaként megoldás egyetlen üzenet-várólista sikerült kell túlterhelik, az üzenetek száma és keresztmetszetet jelenthet a rendszerben. Ebben a helyzetben vegye figyelembe, az üzenetkezelési rendszeréhez adott gyártók üzeneteket küldeni egy adott várólistában particionálás, vagy a terheléselosztás segítségével több üzenetsorok üzenetek szét.

- **Az üzenetkezelési rendszeréhez megbízhatóságának biztosítása**. Egy megbízható üzenetkezelési rendszeréhez, hogy az alkalmazás enqueues után egy üzenetet, nem lesz elveszett biztosításához van szükség. Ez elengedhetetlen ahhoz, hogy minden üzenet legalább egyszer érkeznek.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- Az alkalmazás a munkaterhelés aszinkron módon futtatható tevékenységek van osztva.
- Feladatok független, és párhuzamosan futtatható.
- A munka mennyiségét magas változó, méretezhető megoldást igénylő.
- A megoldás magas rendelkezésre állást biztosítani kell, és rugalmas, ha egy tevékenység feldolgozása sikertelen lehet.

Ebben a mintában előfordulhat, hogy nem lehet hasznos:

- Nincs könnyen az alkalmazás munkaterhelést diszkrét feladatok külön, vagy nincs a tevékenységek közötti függőség magas fokú.
- Feladatok szinkron módon hajtható végre, és az alkalmazás logikáját kell várnia, hogy a feladat befejeződését.
- Feladatok meghatározott sorrendben kell végrehajtani.

> Néhány üzenetküldő rendszereket használnak a termelő üzenetek engedélyezése együtt, és győződjön meg arról, hogy azok még lehet kezelni az azonos ügyfél munkameneteket támogatják. A mechanizmus segítségével rangsorolt üzeneteket (ha azok támogatottak) üzenet, amely továbbítja üzenetek sorozatban a gyártó egy-egy fogyasztó rendelési űrlap megvalósítása.

## <a name="example"></a>Példa

Azure storage várólisták és működhet-ben Ez a kialakítás megvalósítása a Service Bus-üzenetsorok biztosít. Az alkalmazás logikáját beküldheti üzenetsorokba, és egy vagy több szerepkört a tevékenységként megvalósított fogyasztók üzeneteket beolvasni ebből a várólistából, és dolgozza fel őket. Rugalmasság, a Service Bus-üzenetsorba lehetővé teszi, hogy egy felhasználó számára `PeekLock` módra, amikor lekérdezi a üzenetet az üzenetsorból. Ebben a módban ténylegesen nem távolítja az üzenetet, de egyszerűen elrejti a más fogyasztók. Az eredeti felhasználó törölheti az üzenet feldolgozása után. Ha a fogyasztó nem sikerül, a betekintés zárolási időtúllépést okoz, és az üzenet újra láthatóvá válnak, így egy másik fogyasztó ennek lekéréséhez.

> Azure Service Bus-üzenetsorok használatával kapcsolatos részletes információkért lásd: [Service Bus-üzenetsorok, témakörök és előfizetések](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).
Azure tárolási sorok használatával kapcsolatos információkért lásd: [Ismerkedés az Azure Queue storage használatának .NET](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/).

A következő kódot a a `QueueManager` osztály CompetingConsumers megoldás elérhető a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) bemutatja, hogyan hozhat létre egy üzenetsort használatával egy `QueueClient` a példányt a `Start` eseménykezelő egy webes vagy feldolgozói szerepkörben.

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

A következő kódrészletet bemutatja, hogyan az alkalmazások létrehozása és az üzenetkötegek küldése az üzenetsorba.

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

A következő kód bemutatja, hogyan egy fogyasztói szolgáltatáspéldány is üzenetek fogadása az üzenetsorból egy eseményvezérelt módszert követve. A `processMessageTask` paramétert a `ReceiveMessages` metódust egy delegált fut egy üzenet jelenik meg, amikor a kód hivatkozó. Ez a kód aszinkron fut.

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

Vegye figyelembe, hogy használható-e automatikus skálázás funkciók, például az Azure-ban elérhető indítása és leállítása szerepkörpéldányt beállítani, mert a várólista hossza ingadozik. További információkért lásd: [automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx). Emellett nincs szükség a szerepkörpéldányok és a munkavégző folyamatok közötti egy az egyhez típusú egyezés karbantartása&mdash;egyetlen szerepkörpéldányt megvalósítható a több munkavégző folyamatot. További információkért lásd: [számítási erőforrás-összevonási mintát](compute-resource-consolidation.md).

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat előfordulhat, hogy használható legyen ebben a mintában végrehajtása során:

- [Aszinkron üzenetkezelési ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). Üzenet-várólistákból egy aszinkron kommunikációs mechanizmus. Ha a fogyasztó szolgáltatásnak kell egy alkalmazás válaszol, valamilyen válasz üzenetküldési végrehajtásához szükség lehet. Az aszinkron üzenetkezelési ismertetése információkat nyújt a kérelem/válasz üzenetküldési használatával megvalósításához üzenet-várólistákat.

- [Automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx). Esetleg indítása és leállítása egy fogyasztói szolgáltatás példányának óta a feladás egy vagy több üzeneteket változik várólista alkalmazások hosszát. Automatikus skálázás segíthet átviteli karbantartása csúcs feldolgozás esetén.

- [Számítási erőforrás-összevonási mintát](compute-resource-consolidation.md). Esetleg vonják össze a költségek és kezelési terhelés mellett csökkentése során egyetlen fogyasztói szolgáltatás több példánya. A számítási erőforrás-összevonási mintát előnyei és az ezt a módszert használja a következő mellékhatásokkal ismerteti.

- [Minta simítás várólista alapú terhelés](queue-based-load-leveling.md). Üzenet-várólista bevezetéséről adhat hozzá rugalmasság a rendszer, szolgáltatáspéldány, széles körben változó mennyiségű alkalmazáspéldányok kérelmeinek kezeléséhez. Az üzenet-várólista pufferként a, amely a terhelési szintek. A várólista-alapú terhelés simítás mintát ebben a forgatókönyvben részletesebben ismerteti.

- Ez a minta van egy [mintaalkalmazás](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) társítva.
