---
title: "Prioritású várólistára."
description: "Priorizálhatja azokat, hogy a magasabb prioritású kérelmek fogadott, és gyorsabban dolgozza fel a kisebb prioritással rendelkező szolgáltatások küldött kérelmeket."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- performance-scalability
ms.openlocfilehash: ecfbb38304bb95587e9ca15523ad9594898d9b32
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="priority-queue-pattern"></a>Prioritás várólista minta

[!INCLUDE [header](../_includes/header.md)]

Priorizálhatja azokat, hogy a magasabb prioritású kérelmek fogadott, és gyorsabban dolgozza fel a kisebb prioritással rendelkező szolgáltatások küldött kérelmeket. Ebben a mintában akkor hasznos, az alkalmazásokat, amelyek az egyes ügyfelek különböző szolgáltatási szint biztosítja, hogy biztosítson.

## <a name="context-and-problem"></a>A környezetben, és probléma

Alkalmazások hajtsa végre a háttérben történő feldolgozás, vagy más alkalmazásokhoz vagy szolgáltatásokhoz integrálása például delegálhatja más szolgáltatásokban meghatározott feladatok. A felhőben egy üzenet-várólista általában használt háttérben történő feldolgozás feladatok delegálására. Sok esetben egy szolgáltatás-kérelmek fogadásának a sorrendje fontos nem található. Egyes esetekben azonban fontos rangsorolására bizonyos kérésekre. Ezek a kérelmek korábbi, mint az alacsonyabb prioritású virtuális gép kérelmek az alkalmazás által korábban elküldött fel kell dolgozni.

## <a name="solution"></a>Megoldás

A várólista általában érkezési sorrendben történő kiküldési struktúra, és a fogyasztók ugyanabban a sorrendben, amely a várólistára könyvelés általában üzeneteket fogadni. Egyes üzenetsorok azonban támogatja a prioritás üzenetkezelés. Az alkalmazás, egy üzenet is rendelhet prioritást, valamint a várólistán lévő üzenetek automatikusan rendezi újra, így azok a magasabb prioritású fogadja az alacsonyabb prioritásúak előtt. Az ábrán láthatja a prioritás üzenetküldési rendelkező várólista.

![1. ábra – egy üzenetsor-kezelési mechanizmust, amely támogatja az üzenet rangsorolási használatával](./_images/priority-queue-pattern.png)

> A legtöbb üzenet-várólista megvalósítások támogatja, több felhasználóból (következő a [versengő fogyasztók mintát](https://msdn.microsoft.com/library/dn568101.aspx)), és a fogyasztói folyamatok száma is méretezhető felfelé vagy lefelé attól függően, hogy igény szerint.

A rendszer nem támogatja a prioritásalapú üzenetsorok egy alternatív megoldás, hogy minden egyes prioritás külön várólistájának karbantartása. Az alkalmazás felelős az üzenetek a megfelelő várólistát. Minden egyes üzenetsorok rendelkezhetnek fogyasztók külön készletét. Magasabb prioritású várólisták is rendelkezik, és nagyobb, mint az alacsonyabb prioritású virtuális gép várólisták gyorsabb hardveren futó fogyasztói. A következő ábra azt mutatja be, különálló üzenetsorokat használ minden egyes prioritás.

![2. ábra – külön üzenetsorokat használ minden egyes prioritás](./_images/priority-queue-separate.png)


Ezt a stratégiát egy változatát, hogy először ellenőrizze a magas prioritású várólisták üzeneteket, és csak ezután start üzenetek beolvasása alacsonyabb prioritású virtuális gép várólisták a fogyasztók egyetlen készletét. Vannak a fogyasztói folyamatok egyetlen készletét alkalmazó megoldás közötti eltérések szemantikai (vagy egy adott sorba, amely támogatja a különböző prioritásokba vagy több üzenetsorok üzenetek hogy minden egyes üzenetek kezeléséhez egyetlen prioritás), és a megoldás amely használ több várólisták egy másik készlet minden egyes sor.

Magasabb prioritású üzeneteket egyetlen készlet megközelítéssel mindig kapott és dolgozza fel előbb alacsonyabb prioritású üzenetek. Elméletileg üzeneteket, amelyek nagyon alacsony prioritást sikerült folyamatosan írhatók, és előfordulhat, hogy soha nem dolgozható fel. A több készlet módszert használja, az alacsonyabb prioritású üzenetek mindig dolgoz fel, nem csupán leggyorsabban a magasabb prioritású (attól függően, hogy a készletek és az erőforrások relatív méretét, hogy rendelkezik-e elérhető).

A prioritás Üzenetsor-kezelés alapján történő a következő előnyöket biztosítja:

- Ez lehetővé teszi, hogy az üzleti követelményeinek megfelelően, a rendelkezésre állás vagy a teljesítmény, például a különböző szintjei szolgáltatás ügyfelek adott csoportokhoz rangsorolási igénylő alkalmazások.

- Segíthet a működési költségek minimalizálása érdekében. Egyetlen sor megközelítéssel méretezheti vissza a felhasználók száma szükség esetén. Magas prioritású üzenetek továbbra is lesz elsőként feldolgozva (bár valószínűleg lassabban), és az alacsonyabb prioritású üzenetek előfordulhat, hogy később, az többé. Ha a több üzenetet várólista módszert, a fogyasztók egyes várólisták külön készletek bevezetése csökkentheti a fogyasztók alacsonyabb prioritású virtuális gép várólisták körét, vagy feldolgozási néhány nagyon kis prioritású virtuális gép várólisták fel is függesztheti a fogyasztók leállításával, Ezek a várólisták üzeneteket figyeli.

- Üzenet-várólista több megközelítés segítségével, maximalizálhatja az alkalmazás teljesítményét és méretezhetőségét felosztásával az üzenetek feldolgozási követelmények alapján. Például létfontosságú feladatok is prioritása fogadó számára azonnal, míg a kevésbé fontos háttérfeladatok fogadó számára kevésbé foglalt időszakokban ütemezett kell kezelnie kell kezelnie.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

Adja meg a prioritások a megoldás környezetében. Például magas prioritású azt is jelentheti, hogy üzenetek tíz másodpercen belül fel kell dolgozni. A magas prioritású elemeken, és a más erőforrások, az ezen feltételeknek kell kiosztani-követelményeinek azonosítása.

Döntse el, ha az összes magasabb prioritású elemekkel bármely alacsonyabb prioritású elemek előtt kell végrehajtani. Ha az üzenetek feldolgozott fogyasztók egyetlen készletét, hogy egy olyan mechanizmus biztosítása, amely megelőzik és egy feladatot, ha egy magasabb prioritású üzenet elérhetővé válik egy kis prioritású virtuális gép üzenet kezelő felfüggesztése.

A több várólista módszert használja, használatakor az összes várólistán figyelő fogyasztói folyamatok egyetlen készletét, nem pedig egy dedikált fogyasztói készlet minden egyes sor a fogyasztó biztosítja azt mindig magasabb prioritású várólisták szolgáltatások üzeneteit algoritmust kell alkalmaznia mielőtt azokat a alacsonyabb prioritású virtuális gép várólistákból.

A figyelő a feldolgozás sebessége a magas és alacsony prioritású várólisták kiválasztásával győződjön meg arról, hogy ilyen üzeneteket dolgozzák fel a várt sebességeket.

Kell garantálni, kis prioritású virtuális gép üzenetet dolgoz fel, akkor a több üzenetet várólista módszert, a fogyasztók több készletet végrehajtásához szükséges. Másik lehetőségként a sorhoz, amely támogatja az üzenet rangsorolási, is lehet dinamikusan életkorának az üzenetsorban található üzenet, mert a prioritás növelése. Ez a módszer azonban ezt a szolgáltatást nyújtó üzenet-várólista függ.

Egy külön várólista a minden üzenet prioritása működik a legjobban, amelyek néhány jól meghatározott prioritások rendszerekhez.

Üzenet prioritások logikailag lehet meghatározni a rendszer. Például ahelyett, hogy explicit magas és alacsony prioritású üzeneteket, azok sikerült jelölhetők "fizető díj ügyfél" vagy "nem díj fizető ügyfeleket." Attól függően, hogy az üzleti modell a a rendszer a további erőforrásokat kiosztani is díj fizető felhasználók nem díj fizető azokat, mint az üzenetek feldolgozása.

Költség feldolgozása társított ellenőrzése a várólista (néhány kereskedelmi üzenetkezelési rendszerek kell fizetni egy kis díj minden alkalommal, amikor egy üzenet közzé vagy lekérése, és minden alkalommal, amikor a várólista le kell kérdezni az üzenetek) üzenet, és lehet egy pénzügyi. Növeli a költségeket, több várólisták ellenőrzésekor.

Akkor lehet dinamikusan úgy, hogy a fogyasztók alapján, amely a készlet szervizeli a várólista hossza a készlet mérete. További információkért lásd: a [automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a minta olyan esetekben hasznos, ha:

- A rendszer több különböző prioritással rendelkező tevékenységek kell kezelni.

- Különböző felhasználók vagy a bérlők különböző prioritással rendelkező fel kell dolgozni.

## <a name="example"></a>Példa

A Microsoft Azure nem biztosít egy üzenetsor-kezelési mechanizmust, amely natív módon támogatja az automatikus rangsorolási üzenetek rendezés keresztül. Azonban valós Azure Service Bus-üzenettémák és előfizetések, amelyek az üzenetsor-kezelési módszer, amely üzenet szűrés együtt széles körét, amelyek hasznos legtöbb prioritású várólista implementációkban rugalmas lehetőségeket biztosít.

Egy Azure megoldás is létrehozható egy Service Bus, a témakör az alkalmazás beküldheti részére küldött üzenetek, a várólista azonos módon. Üzenetek metaadatok alkalmazás által definiált egyéni tulajdonságok formájában is tartalmazhat. A Service Bus-előfizetések társíthatók a témakört, és ezeket az előfizetéseket üzenetek tulajdonságaik alapján szűrhetők. Amikor egy alkalmazás egy üzenetet küld a témakör, az üzenet van irányítva a megfelelő előfizetés ahol annak olvasása a fogyasztó. Felhasználói folyamatokat is azonos szemantika használatával, egy üzenet-várólista (előfizetés egy logikai várólista) előfizetés üzeneteket beolvasni. A következő ábra azt mutatja be, az Azure Service Bus-üzenettémák és előfizetések prioritású várólista végrehajtási.

![3. ábra - végrehajtási egy prioritású várólistára. az Azure Service Bus-üzenettémák és előfizetések](./_images/priority-queue-service-bus.png)


A fenti ábrán az alkalmazások több üzenetek és rendeli hozzá az egyéni tulajdonságot, `Priority` minden üzenet értékű, vagy `High` vagy `Low`. Az alkalmazás ezen üzeneteket a témakörökbe küldi. A témakör azt, hogy mindkét üzenetek szűrése megvizsgálásával két kapcsolódó előfizetések a `Priority` tulajdonság. Egy előfizetés üzeneteket fogad ahol a `Priority` tulajdonsága `High`, és az egyéb üzeneteket fogad ahol a `Priority` tulajdonsága `Low`. Egyes fogyasztók üzenetek olvassa be az egyes előfizetések. A magas prioritású virtuális gép előfizetés nagyobb készlet rendelkezik, és a fogyasztók számára biztosítson a fogyasztók, kis prioritású virtuális gép-készletben-nál több erőforrást nagyobb teljesítményű számítógépek futhat.

Vegye figyelembe, hogy nincs szükség különleges kapcsolatos magas és alacsony prioritás üzenetek ebben a példában a jelölést. Egyszerűen tulajdonságként az egyes üzeneteket a megadott címkék van, és segítségével a megadott előfizetés-üzenetek. További prioritások szükség, akkor további létrehozásához előfizetések és -készletek kezelésére e prioritások fogyasztói folyamatok viszonylag egyszerű.

Az elérhető PriorityQueue megoldás [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue) tartalmazza ezt a módszert megvalósítása. Ez a megoldás tartalmaz két feldolgozói szerepkör projekt nevű `PriorityQueue.High` és `PriorityQueue.Low`. A feldolgozói szerepkörök örökli a `PriorityWorkerRole` osztály, amely tartalmazza a megadott előfizetés történő kapcsolódáshoz a szolgáltatásokat a `OnStart` metódust.

A `PriorityQueue.High` és `PriorityQueue.Low` feldolgozói szerepkörök csatlakozás különböző előfizetésekhez, a konfigurációs beállítások határozzák meg. Különböző számú minden egyes szerepkör futtatásához a rendszergazdák konfigurálhatják. Általában nem lesz több példánya a `PriorityQueue.High` feldolgozói szerepkör, mint a `PriorityQueue.Low` feldolgozói szerepkör.

A `Run` metódust a `PriorityWorkerRole` osztály rendezi a virtuális `ProcessMessage` metódus (is definiálva a `PriorityWorkerRole` osztály) tartozó minden üzenet érkezett a várólistát. A következő kódot tartalmazza a `Run` és `ProcessMessage` módszerek. A `QueueManager` PriorityQueue.Shared projektben meghatározott osztály Azure Service Bus-üzenetsorok használatával segítő metódusokat biztosít.

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
A `PriorityQueue.High` és `PriorityQueue.Low` mindkét feldolgozói szerepkörök bírálja felül az alapértelmezett funkcióit a `ProcessMessage` metódust. Az alábbi kódot a `ProcessMessage` módszer a `PriorityQueue.High` feldolgozói szerepkör.

```csharp
protected override async Task ProcessMessage(BrokeredMessage message)
{
  // Simulate message processing for High priority messages.
  await base.ProcessMessage(message);
  Trace.TraceInformation("High priority message processed by " +
    RoleEnvironment.CurrentRoleInstance.Id + " MessageId: " + message.MessageId);
}
```

Ha egy alkalmazás visszaküldés üzeneteket a témakör az előfizetések által használt társított a `PriorityQueue.High` és `PriorityQueue.Low` feldolgozói szerepkörök használatával adja meg a prioritás a `Priority` egyéni tulajdonság, a következő kódrészlet példa látható. Ezzel a kóddal (megvalósítva a `WorkerRole` osztály a PriorityQueue.Sender projekt), használja a `SendBatchAsync` segítő metódusában a `QueueManager` visszaküldeni az üzeneteket a témakörökbe kötegekben osztály.

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

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:

- Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue).

- [Aszinkron üzenetkezelési ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). A fogyasztó szolgáltatás, amely feldolgozza a kérést módosítania kell az alkalmazást, amely a kérelem közzétett példányának válaszol. Tájékoztatást nyújt a stratégia használható a kérelem/válasz üzenetküldéshez.

- [Versengő fogyasztók mintát](competing-consumers.md). A várólisták az átviteli sebesség növelése érdekében ugyanazon a várólistán figyelő több felhasználóból rendelkezik, és a párhuzamos tevékenységek feldolgozását. A felhasználókat a rendszer "versenyeznek" az üzeneteket, de csak egy tudják feldolgozni az egyes üzeneteket. További információt nyújt az előnyöket és mellékhatásokkal végrehajtási ezt a módszert használja.

- [Sávszélesség-szabályozási mintát](throttling.md). Sávszélesség-szabályozás várólisták használatával is létrehozható. Prioritás üzenetküldési segítségével győződjön meg arról, hogy kér a kritikus alkalmazások vagy a nagy értékű ügyfelei által futtatott alkalmazások elsőbbséget kérelmek keresztül a kevésbé fontos alkalmazások.

- [Automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx). Esetleg méretezése a kezelése attól függően, hogy a várólista hossza várólista fogyasztói folyamatok készlet mérete. Ezt a stratégiát a teljesítmény javítása érdekében különösen a magas prioritású üzenetkezelő készletek segítségével.

- [Vállalati integrációs minták a Service busszal](http://abhishekrlal.com/2013/01/11/enterprise-integration-patterns-with-service-bus-part-2/) Abhishek Lal blogjában.

