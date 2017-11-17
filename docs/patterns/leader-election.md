---
title: "Vezető választás"
description: "Koordinálja az elosztott alkalmazásban lévő együttműködés task példányokat gyűjteménye egy példánya, amely azt feltételezi, hogy a többi példány felelős a vezetőjeként megválasztását által végrehajtott műveletekről."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- resiliency
ms.openlocfilehash: ddb61097ed3229ed0ed517b94c280d3ef892c999
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="leader-election-pattern"></a>Vezető választás minta

[!INCLUDE [header](../_includes/header.md)]

Koordinálja megválasztását egy példánya, amely a többi kezelése felelősséget a vezetőjeként a gyűjtemény egy elosztottalkalmazás-együttműködés példánya által végrehajtott műveletekről. Ez segít annak érdekében, hogy példányok nem ütköznek egymással, a versengés a megosztott erőforrások miatt, vagy véletlenül zavarja a munkát, amely a többi példány hajtja végre.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egy tipikus felhő alkalmazásnak összehangolt módon viselkedő számos feladatot. Ezeket a feladatokat az összes lehet kódot futtató és az erőforrásokhoz hozzáférést igénylő példányok, vagy azok működik együtt egy bonyolult számításhoz egyes részeire végre párhuzamosan.

A task példányokat lehet, hogy külön futtatja az az idő nagy, de is szükség lehet a műveleteket, győződjön meg arról, hogy nem ütköznek, a versengés a megosztott erőforrások miatt, vagy véletlenül zavarja a munkát, hogy a feladat minden egyes példányának koordinálására példányok hajt végre.

Példa:

- Egy felhőalapú rendszer, amely megvalósítja a horizontális skálázás ugyanezt a feladatot több példánya sikerült fut egyszerre, ha minden példánya egy másik felhasználót szolgál. Ha ezek a példányok írni egy megosztott erőforráson, fontos együttműködéséből feltünteti a mások által végrehajtott módosítások felülírják megelőzése érdekében.
- A feladatok egyes elemeit, egy bonyolult számításhoz párhuzamosan hajtja végre, ha az eredmények kell lennie összesítését, amikor az összes művelet befejeződik.

A task példányokat összes társak, így egy természetes vezető, amely működhet, és a koordinátor vagy a gyűjtő nem áll rendelkezésre.

## <a name="solution"></a>Megoldás

Egy adott feladat példány választják vezetője nevében járhasson el, és ezt a példányt kell koordinálja a többi alárendelt task példányokat a műveleteket. Minden a task példányokat futtassa ugyanazt a kódot, ha egyes képes a vezetőjeként működő. Ezért a választási folyamat felügyelni gondosan két vagy több példány egyszerre a vezető szerepkör tart megelőzése érdekében.

A rendszer meg kell adnia egy robusztus mechanizmus vezetője kijelöléséhez. Ez a módszer a folyamatosan esemény, például a hálózati kimaradások vagy folyamathibák rendelkezik. A sok megoldások az alsóbb szintű task példányokat figyelése valamilyen szívverés metódus, vagy lekérdezési keresztül vezetője. Ha a kijelölt vezető leáll váratlanul, vagy egy hálózati hiba miatt elérhetetlenné vezetője az alárendelt tevékenység példányokhoz, fontos egy új vezető dönthetnek úgy, hogy.

Nincsenek több stratégiák megválasztását között elosztott környezetben feladatokat egy vezető például:
- A feladat példány kiválasztja a legalacsonyabb rangsorolva példányt, vagy a folyamat azonosítója.
- Racing megosztott, elosztott mutex megszerzésére. Az első feladat, amely szerez be a mutex példány vezetője. A rendszer azonban biztosítja, hogy a vezető megszakítja, vagy megszakítja a kapcsolatot a rendszer a többi, ha a mutex átadott lehetővé válik a vezető egy másik feladat példány.
- Például a végrehajtási egyik közös vezető választás algoritmus a [Bully algoritmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) vagy a [Ring algoritmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html). Ezek az algoritmusok azt feltételezik, hogy minden jelölt a választás egy egyedi Azonosítóval rendelkezik-e, és, hogy képes legyen kommunikálni az egyéb jelöltek megbízhatóan.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:
- Egy vezető megválasztását folyamata rugalmas ideiglenes és állandó hibák kell lennie.
- Észlelés, ha a sikertelen vagy más módon elérhetetlenné vált elemezhetőnek kell lennie (például egy kommunikációs hiba miatt). Milyen gyorsan észlelési van szükség a rendszer függő. Egyes rendszerek lehet egy vezető, amely során egy átmeneti hiba előfordulhat, hogy rögzített nélkül rövid ideig működik. Más esetekben szükség lehet vezető hiba azonnali észlelésére, és indul el, egy új választás.
- A vízszintes automatikus skálázás megvalósító rendszer vezetője sikerült megszakítása, ha a rendszer méretezi vissza, és leállítja a számítási erőforrások.
- A külső szolgáltatás, amely a mutex függ a megosztott, elosztott mutex használatával vezet be. A szolgáltatás jelent a hibaérzékeny pontok kialakulását. Ha az elérhetetlenné válik a bármilyen okból, a rendszer nem lehet egy vezető kiválasztják.
- Egyetlen dedikált folyamat a vezetőjeként használata egyszerű megközelítése. Azonban ha a sikertelen lehetnek jelentős késés az újraindítás közben. Az eredményül kapott késés hatással lehet a teljesítmény és a válasz időpont más folyamatok, ha azok egy művelet. koordinálására vezetője eredménykészletre várakozik.
- Manuálisan végrehajtási egyik vezető választás algoritmus beállítás, és optimalizálja a kódot a legnagyobb rugalmasságot biztosít.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ezt a mintát használja, ha az elosztott alkalmazásokban, például a felhőben üzemeltetett megoldás, a feladatok óvatos koordinációs kell, és nem természetes vezető.

>  Elkerüli a vezető szűk keresztmetszet a rendszerben. A vezetője célja, hogy összehangolják a munkát az alsóbb szintű feladatokat, és nem feltétlenül kell részt vesz a munkát saját magát a&mdash;bár erre, ha a feladat nem döntött a vezetőjeként képesnek kell lennie.

Ez a minta nem lehet hasznos, ha:
- Legyen egy természetes vezető vagy dedikált folyamat, amely mindig működhet-e a vezetőjeként. Például előfordulhat, hogy legyen egy egypéldányos folyamat, amely koordinálja a task példányokat valósíthat meg. Ez a folyamat sikertelen lesz, vagy akkor kerül sérült állapotba, ha a rendszer leállítják a webhelyet, és indítsa újra.
- A feladatok összehangolását több egyszerűsített módszerrel lehet elérni. Például ha több task példányokat egyszerűen egy megosztott erőforráson koordinált hozzáférésre van szükségük, jobb megoldás, hogy használja a optimista vagy pesszimista zárolás való hozzáférés.
- Egy harmadik féltől származó megoldás több alkalmas. Például a Microsoft Azure HDInsight szolgáltatáson (Apache Hadoop) Apache Zookeeper által nyújtott szolgáltatások koordinálja a térkép és feladatokat, melyek begyűjtik és adatainak összefoglalója használja.

## <a name="example"></a>Példa

A LeaderElection DistributedMutex projektje (minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) bemutatja, hogyan lehet egy Azure Storage-blob a címbérlet segítségével végrehajtási egy megosztott egy olyan mechanizmus biztosítása elosztott mutex. A mutex segítségével választ egy vezető Azure-felhőszolgáltatás szerepkörpéldányt csoportja között. A bérleti jogot szerezni az első szerepkörpéldányt vezetője kijelölt, és a vezető marad, amíg kiadja a címbérlet, vagy nem tudja a címbérlet megújítása. Többi szerepkörpéldányon továbbra is figyelheti a blob bérleti abban az esetben, ha a vezető már nem érhető el.

>  A blob címbérlet egy kizárólagos írási zárolás blob felett. Egy blob idő lehet bármely pontján csak egy címbérlet tárgyát. A szerepkör példánya a címbérlet kérheti a megadott blob keresztül, és azt fogja adható a címbérlet Ha nincs más szerepkörpéldányt tárolja a címbérlet azonos blob keresztül. A kérelem egyébként kivételt jelez.

> A hibás szerepkör példánya határozatlan ideig megtartja a címbérlet elkerülése érdekében adja meg a címbérlet-élettartamot. Amikor lejár, a címbérlet elérhetővé válik. Azonban amíg egy szerepkörpéldányt, amely tárolja a címbérlet kérheti, hogy a címbérlet megújulásakor, és azt fogja nyújtani a címbérlet további időn belül. A szerepkör példánya is folyamatosan ismételje ezt az eljárást, ha azt szeretné megőrizni a címbérlet.
Blob címbérlet kapcsolatos további információkért lásd: [bérleti Blob (REST API-t)](https://msdn.microsoft.com/library/azure/ee691972.aspx).

A `BlobDistributedMutex` osztály a C# az alábbi példa tartalmazza a `RunTaskWhenMutexAquired` módszer, amely lehetővé teszi, hogy egy sikertelen bejelentkezési kísérletet bérletet szerezni a megadott blob keresztül szerepkörpéldányt. A konstruktornak átadott a blobot (a neve, a tároló és a tárolási fiók) részleteit egy `BlobSettings` objektum amikor a `BlobDistributedMutex` objektum létrehozása (Ez az objektum, amely megtalálható a mintakódot egyszerű struktúra). A konstruktornak is fogad el egy `Task` , amely a kódot, amely a szerepkör példánya futhat, ha sikeresen szerez be a címbérlet a blob keresztül, és kijelölt vezetője hivatkozik. Vegye figyelembe, hogy a kódot, amely kezeli a kevésbé fontos részletek az beszerzése a címbérlet vezettek be egy külön segítőosztály nevű `BlobLeaseManager`.

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

A `RunTaskWhenMutexAquired` a fenti kódminta metódus meghívja a `RunTaskWhenBlobLeaseAcquired` mintában látható módon a következő kód ténylegesen bérleti jogot szerezni a metódust. A `RunTaskWhenBlobLeaseAcquired` metódus aszinkron módon futtatja. A bérlet sikeresen szerezte be, ha a szerepkör példánya úgy lett döntött, hogy a vezető. A célja a `taskToRunWhenLeaseAcquired` delegált, amely a többi szerepkörpéldányon koordinálja a munkájuk elvégzéséhez. A bérlet nem szerezte be, ha egy másik szerepkör példánya lett döntött a vezetőjeként, és az aktuális példányon beosztottja marad. Vegye figyelembe, hogy a `TryAcquireLeaseOrWait` metódus által használt egy segédmetódust a `BlobLeaseManager` objektum a bérleti jogot szerezni.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

A feladat elindult vezetője is aszinkron módon futtatja. Ez a feladat futása közben a `RunTaskWhenBlobLeaseAquired` metódus mintában látható módon a következő kód rendszeres időközönként megpróbálja a címbérlet megújítása. Ezzel biztosíthatja, hogy a szerepkör példánya továbbra is a vezető. A minta a megoldásban a közötti megújítási kérelmeket késleltetési idő legyen kisebb, mint a címbérlet időtartama leállítja, nehogy egy másik szerepkör példánya választanak vezetője megadott idő. A megújítás bármilyen okból nem sikerül, ha a feladat megszakadt.

Ha a címbérlet megújítását sikertelen, vagy megszakadt a feladat (valószínűleg miatt a szerepkörpéldányt leállítása), a címbérlet szabadul fel. Ez vagy egy másik szerepkör példánya ezen a ponton a vezetőjeként kell választani. A kód alatt mutat ez a folyamat során.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

A `KeepRenewingLease` metódus egy másik segítő módszerrel, amely a `BlobLeaseManager` a címbérlet megújítása objektum. A `CancelAllWhenAnyCompletes` metódus visszavonja a feladatokat az első két paraméterként megadva. A következő diagram azt ábrázolja, használja a `BlobDistributedMutex` osztályt választ egy vezető műveleteket koordináló feladat futtatása.

![1. ábra szemlélteti a funkciók BlobDistributedMutex osztály](./_images/leader-election-diagram.png)


Az alábbi példakód bemutatja, hogyan használható a `BlobDistributedMutex` osztály a feldolgozói szerepkörök. Ez a kód címbérletet keresztül nevű blob `MyLeaderCoordinatorTask` a címbérlet tárolóban fejlesztési tárolóban, és megadja, hogy a kód definiálva a `MyLeaderCoordinatorTask` metódus kell futtatni, ha a szerepkör példánya kijelölt vezetője.

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

Megjegyzés: a minta-megoldás a következő szempontokat:
- A blob nem lehetséges hibaérzékeny pontok kialakulását. A blob szolgáltatás nem érhető el, vagy nem érhető el, ha a kitöltés nem fog tudni a címbérlet megújítása, és nincs más szerepkörpéldányt tudják a bérleti jogot szerezni. Ebben az esetben nem szerepkörpéldányt tudnak vezetője nevében járhasson el. Azonban a blob szolgáltatás tervezték rugalmas, így teljesen meghibásodik, a blob szolgáltatás nagyon valószínű tekinthető.
- A feladat vezetője által végzett lefagy, ha a vezető előfordulhat, hogy továbbra is a címbérlet megújítása akadályozza meg, hogy más szerepkör példánya a címbérlet beszerzése és a vezető szerepkör feladatok koordinálására tovább tart. A valós életben vezetője állapotát ellenőrizni kell, rendszeres időközönként.
- A választási folyamat nem determinált. Nem hajtható végre semmilyen feltételezéseket szerepkörtől példányt fog szerezni a blob-címbérlet és vezetője válnak.
- A blob bérleti céljaként használt blob semmilyen más célra nem használható. Ha a szerepkör példánya megkísérli az adatok tárolása a blob, ezek az adatok nem érhető el kivéve, ha a szerepkör példánya vezetője, valamint a blob bérleti.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő útmutatást is lehet megfelelő, ebben a mintában végrehajtása során:
- Ez a minta van egy letölthető [mintaalkalmazás](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election).
- [Automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx). Akkor lehet indítása és leállítása a feladat gazdagépek példányai, mivel változik a terhelés, az alkalmazás. Automatikus skálázás segítségével átviteli sebesség és a maximális feldolgozási idők során.
- [Útmutatás particionálás számítási](https://msdn.microsoft.com/library/dn589773.aspx). Ez az útmutató ismerteti a feladatok lefoglalása egy felhőalapú szolgáltatás, ezáltal segít a méretezhetőséget, teljesítményt, rendelkezésre állási és a szolgáltatás biztonsági megőrzésével futó költségek minimalizálása érdekében a gazdagépek számára.
- A [feladatalapú aszinkron mintát](https://msdn.microsoft.com/library/hh873175.aspx).
- Egy példa ábrázoló a [Bully algoritmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html).
- Egy példa ábrázoló a [Ring algoritmus](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).
- A cikk [Apache Zookeeper a Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) a Microsoft Open Technologies webhelyen.
- [Apache gondnoka](http://curator.apache.org/) egy ügyfélkönyvtárat az Apache ZooKeeper.
- A cikk [bérleti Blob (REST API-t)](https://msdn.microsoft.com/library/azure/ee691972.aspx) az MSDN Webhelyén.
