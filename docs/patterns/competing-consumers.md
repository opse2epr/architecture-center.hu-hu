---
title: Versengő felhasználók mintája
titleSuffix: Cloud Design Patterns
description: Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 77459ff42422969acdc83e66535197547d555de1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112107"
---
# <a name="competing-consumers-pattern"></a>Versengő felhasználók mintája

[!INCLUDE [header](../_includes/header.md)]

Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket. Ez lehetővé teszi, hogy a rendszer több üzenetet dolgozzon fel párhuzamosan a teljesítmény optimalizálása, a skálázhatóság és rendelkezésre állás javítása és a terhelés elosztása érdekében.

## <a name="context-and-problem"></a>Kontextus és probléma

Egy felhőben futó alkalmazásnak várhatóan sok kérelmet kell majd kezelnie. Az egyes kérelmek szinkron módon történő feldolgozása helyett az általános módszer az, hogy az alkalmazás egy üzenetkezelési rendszeren keresztül egy másik szolgáltatásba (feldolgozói szolgáltatásba) küldi őket, ahol a feldolgozásuk aszinkron módon történik. Ez a stratégia segít biztosítani, hogy az alkalmazáshoz kapcsolódó üzleti logikát ne akadályozza semmi, miközben a kérelmek feldolgozása zajlik.

Kérelmek száma idővel nagymértékben változhat, és ennek számos oka lehet. A felhasználói aktivitás hirtelen növekedése vagy az egyszerre több bérlőtől érkező összesített kérelmek kiszámíthatatlan mennyiségű számítási feladatot eredményezhetnek. Csúcsidőszakban előfordulhat, hogy a rendszernek másodpercenként több száz kérelmet kell feldolgoznia, míg más esetben ez a szám igen alacsony is lehet. Ezenkívül a kérelmek kezelése terén végzett munka jellege is igen változó lehet. Egyetlen feldolgozói szolgáltatás alkalmazása azt eredményezheti, hogy az adott példányt elárasztják a kérelmek, vagy az alkalmazás részéről érkező üzenetáradat túlterheli a levelezőrendszert. A változó mennyiségű számítási feladatok kezelése érdekében a rendszer a feldolgozói szolgáltatás több példányát is képes futtatni. Ezeket a feldolgozókat azonban össze kell hangolni annak érdekében, hogy egy üzenetet csak egyetlen feldolgozóhoz kézbesítsen a rendszer. A számítási feladatok jelentette terhelést is el kell osztani a feldolgozók között, nehogy egy példány szűk keresztmetszetté váljon.

## <a name="solution"></a>Megoldás

Az üzenetsor segítségével megteremtheti a kommunikációs csatornát az alkalmazás és a feldolgozói szolgáltatás példányai között. Az alkalmazás üzenetek formájában tesz közzé kérelmeket az üzenetsorban, a feldolgozói szolgáltatáspéldányok pedig fogadják és feldolgozzák ezeket az üzeneteket. Ez a megközelítés lehetővé teszi, hogy ugyanaz a feldolgozói szolgáltatáspéldány-készlet képes legyen kezelni az alkalmazás bármely példányától érkező üzeneteket. Az ábra bemutatja, hogyan lehet az üzenetsor használatával elosztani a munkát a szolgáltatás példányai között.

![A munka szétosztása egy szolgáltatás példányai között üzenetsor használatával](./_images/competing-consumers-diagram.png)

Ez a megoldás a következő előnyökkel jár:

- Kiegyenlített terhelésű rendszert biztosít, amely jelentősen eltérő mennyiségben is képes kezelni az alkalmazáspéldányok által küldött kérelmeket. Az üzenetsor pufferként viselkedik az alkalmazáspéldányok és a feldolgozói szolgáltatás példányai között. Ez segít minimalizálni a rendelkezésre állásra és a válaszképességre gyakorolt hatást mind az alkalmazás, mind a szolgáltatáspéldány esetében, az [Üzenetsor-alapú terheléskiegyenlítési minta](./queue-based-load-leveling.md) ismertetőjében foglaltak szerint. A hosszabb ideig tartó feldolgozást igénylő üzenetek nem akadályozzák a feldolgozói szolgáltatás más példányait a többi üzenet párhuzamos kezelésében.

- Ez növeli a megbízhatóságot. Ha az előállító ennek a mintának a használata helyett közvetlenül kommunikál a feldolgozóval, de nem monitorozza azt, akkor nagy valószínűséggel el fognak veszni üzenetek, vagy a feldolgozásuk sikertelen lesz, ha a feldolgozó meghibásodik. Ebben a mintában a rendszer nem egy adott szolgáltatáspéldányra küldi az üzeneteket. A sikertelen szolgáltatáspéldány nem blokkolja az előállítót, az üzenetek feldolgozását pedig bármely működő szolgáltatáspéldány elvégezheti.

- Nincs szükség összetett koordinációra a feldolgozók, illetve az előállítói és a feldolgozói példányok között. Az üzenetsor biztosítja, hogy a rendszer minden üzenetet legalább egyszer elküldjön.

- Skálázható. A rendszer dinamikusan növelheti vagy csökkentheti a feldolgozó szolgáltatás példányainak számát az üzenetek mennyiségbeli változásainak megfelelően.

- Fokozható a rugalmasság, ha az üzenetsor tranzakciós olvasási műveleteket biztosít. Ha a feldolgozói szolgáltatáspéldány egy tranzakciós művelet részeként olvassa be és dolgozza fel az üzenetet, és a feldolgozói szolgáltatáspéldány meghibásodik, ez a minta gondoskodik arról, hogy az üzenet visszajusson az üzenetsorba, ahol a feldolgozói szolgáltatás egy másik példánya kezelheti.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- **Üzenetrendezés**. A sorrend, amelyben a fogyasztói szolgáltatáspéldány az üzeneteket fogadja, nem garantált, és nem feltétlenül tükrözi az üzenetek létrehozásának sorrendjét. Érdemes úgy megtervezni rendszert, hogy biztosítsa az üzenetek idempotens feldolgozását, mert ez segít kiküszöbölni minden olyan függőséget, amely az üzenetek kezelési sorrendjéhez kapcsolódik. További információkért lásd: [Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) (Jonathan Oliver blogjában).

    > A Microsoft Azure Service Bus-üzenetsorok üzenet-munkamenetek alkalmazásával képesek garantálni az üzenetek elsőnek-be, elsőnek-ki típusú kezelését. További információkért lásd: [Üzenetkezelési minták munkamenetek használatával](https://msdn.microsoft.com/magazine/jj863132.aspx).

- **Rugalmasságot biztosító szolgáltatások tervezése**. Ha a rendszert úgy tervezték, hogy észlelje és újraindítsa a sikertelen szolgáltatáspéldányokat, szükség lehet a szolgáltatáspéldányok által végrehajtott feldolgozás idempotens műveletekként történő alkalmazására, hogy egy adott üzenet többszöri lekéréséből és feldolgozásából eredő hatások minimalizálhatók legyenek.

- **Az ártalmas üzenetek észlelése**. Ha egy helytelenül formázott üzenet vagy feladat olyan erőforrásokhoz igényel hozzáférést, amelyek nem érhetők el, az a szolgáltatáspéldány sikertelen működéséhez vezethet. A rendszernek meg kell akadályoznia, hogy ezek az üzenetek visszajussanak az üzenetsorba. Ehelyett valahol máshol kell tárolnia ezeknek az üzeneteknek az adatait, hogy szükség esetén elemezni lehessen azokat.

- **Eredmények kezelése**. Az üzenetet kezelő szolgáltatáspéldányt a rendszer teljes mértékben leválasztja az üzenetet létrehozó alkalmazáslogikáról, így ezek valószínűleg nem tudnak közvetlenül kommunikálni egymással. Ha a szolgáltatáspéldány olyan eredményeket hoz létre, amelyeket vissza kell küldeni az alkalmazáslogikának, ezeket az információkat mindkét fél számára elérhető helyen kell tárolni. Annak érdekében, hogy az alkalmazáslogika ne hívhasson le hiányos adatokat, a rendszernek jeleznie kell, ha az adatok feldolgozása befejeződött.

     > Az Azure használata esetén egy feldolgozófolyamat vissza tudja küldeni az eredményeket az alkalmazáslogikának egy dedikált válaszadási üzenetsor használatával. Az alkalmazáslogikának össze kell kötnie ezeket az eredményeket az eredeti üzenettel. Ennek a forgatókönyvnek a részletes leírása [az aszinkron üzenetkezelés ismertetését](https://msdn.microsoft.com/library/dn589781.aspx) tartalmazó témakörben található.

- **Az üzenetkezelő rendszer skálázása**. Egy nagyméretű megoldás esetében egyetlen üzenetsort túlterhelhetnek az üzenetek, és így szűk keresztmetszetként jelenhet meg a rendszerben. Ebben a helyzetben érdemes megfontolni az üzenetkezelési rendszer particionálását, hogy az adott előállítók a meghatározott üzenetsorra küldjék az üzeneteket. Használhat terheléselosztást is, hogy több üzenetsor között oszthassa szét az üzeneteket.

- **Az üzenetkezelési rendszer megbízhatóságának biztosítása**. Egy megbízható üzenetkezelési rendszerre van szükség annak érdekében, hogy az alkalmazás által sorba helyezett üzenet biztosan ne vesszen el. Ez elengedhetetlen ahhoz, hogy a rendszer minden üzenetet legalább egyszer kézbesítsen.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Az alkalmazás számítási feladatai aszinkron módon futtatható feladatokra vannak osztva.
- A feladatok egymástól függetlenek, és párhuzamosan futtathatók.
- A munka mennyisége nagymértékben változó, ami skálázható megoldást kíván.
- A megoldásnak magas rendelkezésre állást kell biztosítania, továbbá rugalmasan kell kezelnie egy feladat feldolgozásának meghiúsulását.

Nem érdemes ezt a mintát használni, ha:

- Az alkalmazás számítási feladatait nem könnyű különálló feladatokká szétválasztani, vagy a feladatok között nagyfokú függőség áll fenn.
- A feladatokat szinkron módon kell végrehajtani, és az alkalmazáslogikának meg kell várnia a feladat befejeződését a továbblépés előtt.
- A feladatok meghatározott sorrendben kell végrehajtani.

> Néhány üzenetküldő rendszer olyan munkameneteket támogat, amelyek lehetővé teszik az előállító számára az üzenetek csoportosítását, illetve annak biztosítását, hogy mindet ugyanaz a feldolgozó kezelje. Ez a mechanizmus rangsorolt üzenetek esetében alkalmazható (ha azok támogatottak) egy olyan üzenetrendezés megvalósításához, amely sorrendben továbbítja az üzeneteket az előállítótól egyetlen fogyasztónak.

## <a name="example"></a>Példa

Az Azure olyan tárolási üzenetsorokat és Service Bus-üzenetsorokat biztosít, amelyek a minta alkalmazásának mechanizmusaként működhetnek. Az alkalmazáslogika üzeneteket küldhet az üzenetsorba, a feladatokként, egy vagy több szerepkörben megvalósított feldolgozók pedig beolvashatják az üzeneteket ebből az üzenetsorból, és feldolgozhatják azokat. A rugalmasság érdekében a Service Bus-üzenetsor lehetővé teszi, hogy a fogyasztó a `PeekLock` módot használja, amikor üzenetet kér le az üzenetsorból. Ez a mód ténylegesen nem távolítja el az üzenetet, hanem egyszerűen elrejti azt más feldolgozók elől. Az eredeti feldolgozó törölheti az üzenetet, amikor már végzett a feldolgozásával. Ha a feldolgozó nem jár sikerrel, a betekintési zárolás időtúllépést okoz, és az üzenet újra láthatóvá válik, így egy másik feldolgozó is lekérheti azt.

Az Azure Service Bus-üzenetsorok használatával kapcsolatos részletes információkért lásd: [Service Bus-üzenetsorok, témakörök és előfizetések](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx).

Az Azure tárolási üzenetsorok használatával kapcsolatos információkért lásd: [Az Azure Queue Storage használatának első lépései a .NET-keretrendszerrel](/azure/storage/queues/storage-dotnet-how-to-use-queues).

A [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers)-on elérhető CompetingConsumers megoldás `QueueManager` osztályából származó alábbi kód bemutatja, hogyan hozható létre egy `QueueClient`-példány a `Start` eseménykezelővel egy webes vagy feldolgozói szerepkörben.

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

A következő kódrészletet bemutatja, hogy egy alkalmazás hogyan hozhat létre és küldhet el egy üzenetköteget az üzenetsorba.

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

A következő kód bemutatja, hogy egy feldolgozói szolgáltatáspéldány hogyan fogadhat üzeneteket az üzenetsorból egy eseményvezérelt módszert követve. A `ReceiveMessages` metódus `processMessageTask` paramétere egy meghatalmazott, amely a futtatandó kódra hivatkozik egy üzenet beérkezésekor. Ez a kód aszinkron módon fut.

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

Érdemes megjegyezni, hogy az automatikus skálázási funkciók – például azok, amelyek az Azure-ban is elérhetők – a szerepkörpéldányok indításához és leállításához is használhatók, ahogy az üzenetsor hossza ingadozik. További információért lásd az [automatikus skálázás útmutatóját](https://msdn.microsoft.com/library/dn589774.aspx). Emellett nincs szükség a szerepkörpéldányok és a munkavégző folyamatok közötti egy az egyhez típusú egyezés fenntartására &mdash; egyetlen szerepkörpéldány több munkavégző folyamatot is megvalósíthat. További információkért lásd a [számításierőforrás-konszolidálási mintát](./compute-resource-consolidation.md).

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók hasznosak lehetnek a minta megvalósításakor:

- [Az aszinkron üzenetkezelés ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). Az üzenetsorok aszinkron típusú kommunikációs mechanizmusok. Ha a feldolgozói szolgáltatásnak választ kell küldenie egy alkalmazásnak, szükség lehet valamilyen válaszüzenet-küldés alkalmazására. Az aszinkron üzenetkezelés ismertetése információkat nyújt a kérelem/válasz típusú üzenetküldés üzenetsorok használatával történő megvalósításával kapcsolatban.

- [Útmutató az automatikus skálázáshoz](https://msdn.microsoft.com/library/dn589774.aspx). Egy feldolgozói szolgáltatás példányait el lehet indítani és le lehet állítani, mivel azon üzenetsorok hossza, amelyekre az alkalmazások üzeneteket küldenek, változó. Az automatikus skálázás segítségével az átviteli sebesség szinten tartható a feldolgozási csúcsidőszakban.

- [Számításierőforrás-konszolidálási minta](./compute-resource-consolidation.md). Egy feldolgozói szolgáltatás több példánya egyetlen folyamatba vonhatók össze a költségek és a felügyeleti terhek csökkentése érdekében. A számításierőforrás-konszolidálási minta ismerteti ennek a módszernek az előnyeit és a hátrányait.

- [Üzenetsor-alapú terheléskiegyenlítési minta](./queue-based-load-leveling.md). Az üzenetsor bevezetése rugalmassá teheti a rendszert, lehetővé téve, hogy a szolgáltatáspéldányok kezelni tudják alkalmazáspéldányoktól érkező, váltakozó mennyiségű kérelmeket. Az üzenetsor pufferként viselkedik, ami kiegyenlíti a terhelést. Az üzenetsor-alapú terheléskiegyenlítési minta részletesebben is ismerteti ezt a forgatókönyvet.

- Ehhez a mintához egy [mintaalkalmazás](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) is tartozik.
