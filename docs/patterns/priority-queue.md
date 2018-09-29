---
title: Priority Queue
description: Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- performance-scalability
ms.openlocfilehash: 400bfbc03cf5640ff32a551636b01d60e6c0ec50
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428499"
---
# <a name="priority-queue-pattern"></a>Elsőbbségi üzenetsor mintája

[!INCLUDE [header](../_includes/header.md)]

Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat. Ez a minta olyan alkalmazások esetében hasznos, amelyek különböző szolgáltatási szintre vonatkozó garanciákat kínálnak az egyes ügyfelek számára.

## <a name="context-and-problem"></a>Kontextus és probléma

Az alkalmazások bizonyos feladatokat delegálhatnak más szolgáltatások számára, például a háttérfeldolgozás végrehajtását vagy a más alkalmazásokkal/szolgáltatásokkal való integrációt. A felhőben az üzenetsorokat jellemzően a feladatok háttérbeli feldolgozásra való delegálására használjuk. Sok esetben a szolgáltatások által fogadott rendelési kérelmek nem lényegesek. Néha azonban meg kell határozni bizonyos kérelmek prioritását. Ezeket a kérelmeket korábban kell feldolgozni, mint az alkalmazás által korábban küldött, alacsonyabb prioritású kérelmeket.

## <a name="solution"></a>Megoldás

Az üzenetsorok általában érkezési sorrend alapján végzik a feldolgozást, és a fogyasztók jellemzően az üzenetsorban való közzététel sorrendjében kapják meg az üzeneteket. Bizonyos üzenetsorok azonban támogatják a prioritásalapú üzenetkezelést is. Az üzenetet közzétevő alkalmazás prioritást rendelhet az üzenetekhez, és ekkor a rendszer az üzenetsorban lévő üzenetek sorrendjét automatikusan úgy rendezi át, hogy a magasabb prioritású üzenetek fogadása történjen meg előbb. Az ábrán egy prioritásalapú üzenetkezelést alkalmazó üzenetsor látható.

![1. ábra – Üzenetpriorizálást támogató üzenetsor-kezelési mechanizmus használata](./_images/priority-queue-pattern.png)

> A legtöbb üzenetsor-implementáció támogatja a többfogyasztós megoldásokat (a [Versengő felhasználókat ismertető minta](https://msdn.microsoft.com/library/dn568101.aspx) szerint), és a fogyasztói folyamatok száma igény szerint skálázható vertikálisan.

Olyan rendszerekben, amelyek nem támogatják a prioritásalapú üzenetsorokat, alternatív megoldásként külön sorokat tarthatunk fenn a prioritások számára. Az alkalmazás feladata az üzenetek megfelelő üzenetsorban történő közzététele. Minden egyes üzenetsorhoz külön fogyasztók tartozhatnak. A magasabb prioritású üzenetsorokhoz több, gyorsabb hardveren futtatott fogyasztó tartozhat, mint az alacsonyabb prioritású üzenetsorokhoz. A következő ábra azt mutatja be, amikor külön üzenetsorokat használunk minden egyes prioritáshoz.

![2. ábra – Külön üzenetsor használata minden egyes prioritáshoz](./_images/priority-queue-separate.png)


Ennek a stratégiának egy variációja, ha egyetlen fogyasztókészlet először a magas prioritású üzenetsorok üzeneteit kérdezi le, majd azután az alacsonyabb prioritású üzenetsorok üzeneteit. Szemantikai különbségek vannak az egyetlen fogyasztófolyamat-készletet alkalmazó megoldások (ezek használhatnak egyetlen, különböző prioritású üzeneteket támogató üzenetsort vagy több üzenetsort, amelyek mindegyike adott prioritású üzeneteket kezel), illetve az olyan, több üzenetsort használó megoldások között, ahol minden egyes üzenetsorhoz külön készlet tartozik.

Az egykészletes módszer esetén a magasabb prioritású üzeneteket fogadása és feldolgozása mindig az alacsonyabb prioritású üzenetek előtt történik. Elméletileg előfordulhat, hogy a nagyon alacsony prioritású üzenetek folyamatosan kiszorulnak, és soha nem lesznek feldolgozva. A többkészletes módszerben az alacsonyabb prioritású üzenetek minden esetben fel lesznek dolgozva, csak lassabban, mint a magasabb prioritásúak (a feldolgozás sebessége a készletek relatív méretétől és a rendelkezésre álló erőforrásoktól függ).

A prioritásalapú üzenetsor-kezelés alkalmazása a következő előnyökkel jár:

- Lehetővé teszi az alkalmazások számára, hogy megfeleljenek azon üzleti követelményeknek, amelyek a rendelkezésre állás vagy a teljesítmény priorizálását igénylik (például különböző szintű szolgáltatást nyújtanak adott ügyfélcsoportok számára).

- Hozzájárulhat az üzemeltetési költségek minimálisra csökkentéséhez. Az egykészletes módszerben szükség esetén csökkenthetjük a fogyasztók számát. Továbbra is először a magas prioritású üzenetek lesznek feldolgozva (bár esetleg lassabban), az alacsonyabb prioritású üzenetek pedig hosszabb ideig késhetnek. Ha több üzenetsoros megközelítést használ, ahol minden egyes üzenetsorhoz külön fogyasztókészlet tartozik, lecsökkentheti az alacsonyabb prioritású üzenetsorokhoz tartozó fogyasztókészlet nagyságát, vagy akár fel is függesztheti egyes nagyon alacsony prioritású üzenetsorok feldolgozását úgy, hogy leállítja a kérdéses üzenetsorok üzeneteit figyelő fogyasztókat.

- A többüzenetsoros megközelítés révén maximalizálható az alkalmazások teljesítménye és skálázhatósága az üzenetek feldolgozási követelményeken alapuló particionálásával. Például az alapvető fontosságú feladatok priorizálásával elérhető, hogy azokat azonnal futó fogadók dolgozzák fel, míg a kevésbé fontos háttérfeladatokat a kisebb terhelésű időszakokban futó fogadók kezelhetik.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

Határozza meg a prioritásokat a megoldás tekintetében. A magas prioritás például azt jelentheti, hogy az üzeneteket tíz másodpercen belül fel kell dolgozni. Határozza meg a magas prioritású elemek kezelésére vonatkozó követelményeket, valamint az ezen kritériumoknak való megfelelés céljából lefoglalni kívánt egyéb forrásokat.

Döntse el, hogy minden magas prioritású elemet az alacsonyabb prioritású elemek előtt kíván-e feldolgozni. Ha az üzeneteket egyetlen fogyasztókészlet dolgozza fel, ehhez olyan mechanizmust kell biztosítani, amely képes visszasorolni és felfüggeszteni az alacsony prioritású üzeneteket kezelő feladatot, ha magasabb prioritású üzenet válik elérhetővé.

Ha a több üzenetsoros megközelítésben egyetlen olyan fogyasztófolyamat-készletet használunk, amely az összes üzenetsort, nem pedig az egyes üzenetsorokhoz tartozó dedikált fogyasztókészletet figyeli, a fogyasztónak olyan algoritmust kell alkalmaznia, amely biztosítja, hogy a rendszer mindig előbb a magasabb prioritású üzenetsorok üzeneteit szolgálja ki.

A magas és alacsony prioritású üzenetsorok feldolgozási sebességének monitorozásával biztosítható, hogy az ezen üzenetsorokban lévő üzenetek feldolgozása a várt sebességgel történjen.

Ha garantálni kell, hogy az alacsony prioritású üzenetek feldolgozása megtörténik, a több üzenetsoros megközelítést több fogyasztókészlettel kell implementálni. Alternatív megoldásként az üzenetpriorizálást támogató üzenetsorokban lehetséges az üzenetsorban található üzenetek prioritásának dinamikus növelése az idő előrehaladtával. Ezen megközelítés megvalósíthatósága azonban a funkciót biztosító üzenetsortól függ.

Külön üzenetsor használata az egyes üzenetprioritásokhoz olyan rendszerek esetében működik a legjobban, amelyek kevés, jól meghatározott prioritással rendelkeznek.

A rendszer képes logikai alapon meghatározni az üzenetprioritásokat. Például ahelyett, hogy explicit módon magas és alacsony prioritású üzenetekről beszélnénk, ezek lehetnek „fizető ügyfelek” és „nem fizető ügyfelek”. Az üzleti modelltől függően a rendszer több erőforrást foglalhat le a fizető ügyfelektől származó üzenetek feldolgozására, mint a nem fizető ügyfelek esetében.

Előfordulhat, hogy pénzügyi és feldolgozási költséggel jár egy üzenet lekérdezése az üzenetsorból (egyes kereskedelmi forgalomban lévő üzenetkezelési rendszerek kis összegű díjat számítanak fel az üzenetek közzétételekor vagy lekérdezésekor, illetve az üzenetek üzenetsorból való lekérésekor). Több üzenetsor lekérdezésekor a költségek is növekednek.

A készlet által kiszolgált üzenetsor hossza alapján lehetséges a fogyasztókészlet méretének dinamikus módosítása. További információ: [Automatikus skálázási útmutató](https://msdn.microsoft.com/library/dn589774.aspx).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi forgatókönyvekben hasznos:

- A rendszernek több olyan feladatot kell kezelnie, amelyek eltérő prioritásokkal rendelkeznek.

- A különböző felhasználókat vagy bérlőket különböző prioritással kell kiszolgálni.

## <a name="example"></a>Példa

A Microsoft Azure nem biztosít olyan üzenetsor-kezelési mechanizmust, amely natív módon támogatja az üzenetek rendezéssel történő automatikus priorizálását. Biztosít azonban olyan Azure Service Bus-témaköröket és -előfizetéseket, amelyek támogatják az üzenetszűrési szolgáltatást nyújtó üzenetsor-kezelési mechanizmust, valamint számos olyan rugalmas képességet, amelyek ideálissá teszik azt a legtöbb prioritásalapú üzenetsor implementálásakor.

Az Azure-megoldások képesek olyan Service Bus-témakörök implementálására, amelybe az alkalmazások az üzenetsorokkal azonos módon üzeneteket tehetnek közzé. Az üzenetek alkalmazások által meghatározott egyéni tulajdonságok formájában metaadatokat tartalmazhatnak. Az adott témakörhöz Service Bus-előfizetések társíthatók, és ezek az előfizetések a tulajdonságaik alapján szűrhetik az üzeneteket. Amikor egy alkalmazás üzenetet küld egy témakörnek, az üzenetet a rendszer a megfelelő előfizetéshez irányítja, ahol a fogyasztó elolvashatja azt. A fogyasztófolyamatok az üzenetek előfizetésből történő lekérdezésére ugyanazt a szemantikát használhatják, mint az üzenetsorok (az előfizetés logikai üzenetsornak tekinthető). A következő ábrán egy prioritásalapú üzenetsor Azure Service Bus-témakörök és -előfizetések felhasználásával történő implementálása látható.

![3. ábra – Prioritásalapú üzenetsor Azure Service Bus-témakörök és -előfizetések felhasználásával történő implementálása](./_images/priority-queue-service-bus.png)


A fenti ábrán szereplő alkalmazás több üzenetet hoz létre, és hozzájuk rendel egy `Priority` nevű egyéni tulajdonságot és egy értéket (ez `High` vagy `Low` lehet). Az alkalmazás ezeket az üzeneteket közzéteszi egy témakörbe. Ez a témakör két társított előfizetéssel rendelkezik, amelyek a `Priority` tulajdonság megvizsgálásával szűrik az üzeneteket. Az egyik előfizetés akkor fogad üzeneteket, ha a `Priority` tulajdonság értéke `High`, a másik pedig akkor, ha a `Priority` tulajdonság értéke `Low`. Az egyes előfizetésekből fogyasztókészletek olvassák az üzeneteket. A magas prioritású előfizetések nagyobb készlettel rendelkeznek, és ezek a fogyasztók nagyobb teljesítményű, több erőforrással rendelkező számítógépeken futhatnak, mint az alacsony prioritású készletek fogyasztói.

Vegye figyelembe, hogy ebben a példában nincs jelentősége a magas és alacsony prioritású üzenetek megjelölésének. Ezek egyszerűen az egyes üzenetekben tulajdonságokként megadott címkék, amelyeket a rendszer arra használ, hogy az üzeneteket egy adott előfizetéshez irányítsák. Ha további priorizálásra van szükség, viszonylag könnyű létrehozni további előfizetéseket fogyasztófolyamat-készleteket ezen prioritások kezeléséhez.

A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue) elérhető PriorityQueue megoldás ezen megközelítés egyik implementálását tartalmazza. Ez a megoldás a következő két feldolgozóiszerepkör-projektet tartalmazza: `PriorityQueue.High` és `PriorityQueue.Low`. Ezek a feldolgozói szerepkörök a `PriorityWorkerRole` osztály tulajdonságait öröklik, amely az adott előfizetéshez való csatlakozási funkciót tartalmazza az `OnStart` metódusban.

A `PriorityQueue.High` és a `PriorityQueue.Low` feldolgozói szerepkör különböző előfizetésekhez csatlakozik, amelyet a konfigurációs beállítások határoznak meg. A rendszergazdák külön-külön konfigurálhatják a futtatni kívánt szerepkörök számát. Jellemzően a `PriorityQueue.High` feldolgozói szerepkör több példánya fog futni, mint a `PriorityQueue.Low` feldolgozói szerepkörből.

A `PriorityWorkerRole` osztály `Run` metódusa kezeli az üzenetsorban fogadott üzenetek esetében futtatandó virtuális `ProcessMessage` metódust (amely szintén a `PriorityWorkerRole` osztályban van meghatározva). A következő kód a `Run` és a `ProcessMessage` metódust mutatja be. A PriorityQueue.Shared projektben meghatározott `QueueManager` osztály Azure Service Bus-üzenetsorok használatához biztosít segédmetódusokat.

```csharp
public class PriorityWorkerRole : RoleEntryPoint
{
  private QueueManager queueManager;
  ...

  public override void Run()
  {
    // Start listening for messages on the subscription.
    var subscriptionName = CloudConfigurationManager.GetSetting("SubscriptionName");
    this.queueManager.ReceiveMessages(subscriptionName, this.ProcessMessage);
    ...;
  }
  ...

  protected virtual async Task ProcessMessage(BrokeredMessage message)
  {
    // Simulating processing.
    await Task.Delay(TimeSpan.FromSeconds(2));
  }
}
```
A `PriorityQueue.High` és a `PriorityQueue.Low` feldolgozói szerepkör egyaránt felülírja a `ProcessMessage` metódus alapértelmezett funkcióját. Az alábbi kód a `PriorityQueue.High` feldolgozói szerepkör `ProcessMessage` metódusát mutatja be.

```csharp
protected override async Task ProcessMessage(BrokeredMessage message)
{
  // Simulate message processing for High priority messages.
  await base.ProcessMessage(message);
  Trace.TraceInformation("High priority message processed by " +
    RoleEnvironment.CurrentRoleInstance.Id + " MessageId: " + message.MessageId);
}
```

Ha egy alkalmazás a `PriorityQueue.High` és a `PriorityQueue.Low` feldolgozói szerepkör által használt előfizetésekhez rendelt témakörökben tesz közzé üzeneteket, akkor a `Priority` egyéni tulajdonság használatával adja meg azok prioritását, ahogyan az az alábbi mintakód esetében is látható. Ez a (PriorityQueue.Sender projekt `WorkerRole` osztályában implementált) kód a `QueueManager` osztály `SendBatchAsync` segédmetódusát használja az üzenetek témakörökben történő kötegelt közzétételéhez.

```csharp
// Send a low priority batch.
var lowMessages = new List<BrokeredMessage>();

for (int i = 0; i < 10; i++)
{
  var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
  message.Properties["Priority"] = Priority.Low;
  lowMessages.Add(message);
}

this.queueManager.SendBatchAsync(lowMessages).Wait();
...

// Send a high priority batch.
var highMessages = new List<BrokeredMessage>();

for (int i = 0; i < 10; i++)
{
  var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
  message.Properties["Priority"] = Priority.High;
  highMessages.Add(message);
}

this.queueManager.SendBatchAsync(highMessages).Wait();
```

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue) talál egy, a minta bemutatására szolgáló példát.

- [Az aszinkron üzenetkezelés ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). Lehetséges, hogy a kérelmet feldolgozó fogyasztói szolgáltatásnak választ kell küldenie a kérelmet közzétevő alkalmazáspéldány számára. Információkat nyújt a kérelem/válasz típusú üzenetkezelés implementálásakor használható stratégiákról.

- [Versengő felhasználókat ismertető minta](competing-consumers.md). Az üzenetsorok átviteli sebességének növelése érdekében lehetséges, hogy több fogyasztó figyelje ugyanazt az üzenetsort, és egyidejűleg dolgozza fel a feladatokat. Ezek a fogyasztók versenyezni fognak az üzenetekért, de egy adott üzenetet csak egy dolgozhat fel. További információkat biztosít ezen megközelítés implementálásának előnyeiről és hátrányairól.

- [Szabályozási minta](throttling.md). A szabályozás üzenetsorok használatával valósítható meg. A prioritásalapú üzenetkezelés annak biztosítására használható, hogy a kritikus vagy a fontos ügyfelek által futtatott alkalmazások elsőbbséget élvezzenek a kevésbé fontos alkalmazások kérelmeihez képest.

- [Útmutató az automatikus skálázáshoz](https://msdn.microsoft.com/library/dn589774.aspx). Lehetségessé válhat az üzenetsorokat kezelő fogyasztófolyamat-készletek méretének skálázása az üzenetsor hosszától függően. Ezzel a stratégiával javítható a teljesítmény, különösen a magas prioritású üzeneteket kezelő készletek esetében.

- [Vállalati integrációs minták a Service Bus használatával](https://abhishekrlal.com/2013/01/11/enterprise-integration-patterns-with-service-bus-part-2/) Abhishek Lal blogjában.

