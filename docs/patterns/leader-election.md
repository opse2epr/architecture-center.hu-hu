---
title: Vezetőválasztási minta
titleSuffix: Cloud Design Patterns
description: Koordinálhat egy elosztott alkalmazásban az együttműködő feladatpéldányokból álló gyűjtemény által végrehajtott műveleteket, ha vezetőnek választ meg egy példányt, amely vállalja a többi példány kezelésével járó felelősséget.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: cfc29e3490735c16b41c494e6cecbb8972cdc705
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010139"
---
# <a name="leader-election-pattern"></a>Vezetőválasztási minta

[!INCLUDE [header](../_includes/header.md)]

Az elosztott alkalmazásokban az együttműködő példányok gyűjteményei által végrehajtott műveletek koordinálhatók, ha megválaszt egy példányt vezetőnek, amely így felelős lesz a többi példány kezeléséért. Ez segít annak biztosításában, hogy a példányok nem ütköznek egymással, nem versengenek a megosztott erőforrásokért, és véletlenül sem zavarják meg a többi példány működését.

## <a name="context-and-problem"></a>Kontextus és probléma

Egy tipikus felhőalkalmazásban több, koordinált módon működő feladat található. Ezek a feladatok lehetnek ugyanazt a kódot futtató és ugyanahhoz az erőforráshoz hozzáférést igénylő példányok, vagy előfordulhat, hogy egy összetett számítás egyes részeit végzik párhuzamosan.

A feladatpéldányok az idő legnagyobb részében külön futhatnak, de szükség lehet az egyes példányok műveleteinek koordinálására, hogy a példányok ne ütközzenek, ne versengjenek a megosztott erőforrásokért, és véletlenül se zavarják meg a többi példány működését.

Példa:

- A horizontális skálázást alkalmazó felhőalapú rendszerekben egy feladat több példánya is futhat egyszerre úgy, hogy mindegyik példány egy különböző felhasználót szolgál ki. Ha ezek a példányok egy megosztott erőforrásba írnak, össze kell hangolni a műveleteiket, nehogy az egyik példány felülírja egy másik példány által elvégzett módosításokat.
- Ha a feladatok egy összetett számítás egyes elemeit párhuzamosan végzik, a feladatok befejezése után összesíteni kell az eredményeket.

A feladatpéldányok társviszonyban állnak, ezért nincs koordinátorként vagy összesítőként működő, vezető példány.

## <a name="solution"></a>Megoldás

Ki kell választani egy feladatpéldányt, amely vezetőként működik, és ez a példány fogja koordinálni a többi, alárendelt feladatpéldány műveleteit. Ha az összes feladatpéldány ugyanazt a kódot futtatja, akkor bármelyik működhet vezetőként. Ezért a választás során ügyelnie kell arra, nehogy két vagy több példány egyszerre vegye át a vezetői szerepkört.

A rendszernek robusztus mechanizmust kell biztosítania a vezető kiválasztásához. Ennek a módszernek tudnia kell kezelni az olyan eseményeket, mint például a hálózati kimaradások vagy a folyamathibák. Számos megoldásban az alárendelt folyamatpéldányok valamilyen szívveréses módszerrel vagy lekérdezéssel monitorozzák a vezetőt. Ha a kijelölt vezető váratlanul leáll, vagy az alárendelt feladatpéldányok hálózatkimaradás miatt nem tudják elérni, új vezetőt kell választani nekik.

Többféle módon is kiválasztható egy vezető egy elosztott környezet feladatainak készletéből, például:

- A legalacsonyabb besorolású példány- vagy folyamatazonosítóval rendelkező feladatpéldány kiválasztása.
- Verseny a közös, elosztott mutex beszerzéséért. Az a feladatpéldány lesz a vezető, amelyik elsőként szerzi be a mutexet. Azonban a rendszernek biztosítania kell a mutex felszabadítását, ha a vezető leáll vagy megszakad a kapcsolata a rendszer többi részével, hogy egy másik feladatpéldány vehesse át a vezető szerepet.
- Egyik széles körben használt vezetőválasztási algoritmus (például a [Bully algoritmus](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) vagy a [Ring algoritmus](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)) megvalósítása. Ezek az algoritmusok azt feltételezik, hogy a választásban részt vevő minden jelölt egyedi azonosítóval rendelkezik, és megbízhatóan kommunikál a többi jelölttel.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- A vezetőválasztási folyamatnak ellenállónak kell lennie az átmeneti és állandó hibákkal szemben.
- Észlelnie kell, ha a vezető leáll vagy bármely más okból elérhetetlenné válik (például egy kommunikációs hiba miatt). Az észlelés szükséges sebessége a rendszertől függ. Egyes rendszerek rövid ideig vezető nélkül is működőképesek lehetnek, és ez alatt az idő alatt az átmeneti hiba kijavítható. Más esetekben előfordulhat, hogy azonnal észlelni kell a vezető leállását, és új választást kell indítani.
- Az automatikus horizontális skálázást megvalósító rendszereknél a vezető eltávolítható, ha a rendszer leskálázza a számítási erőforrásokat, vagy néhányat leállít közülük.
- Közös, elosztott mutex használata esetén függőségi kapcsolat áll fenn a mutexet biztosító külső szolgáltatással. Ez a szolgáltatás kritikus hibapont lehet. Ha ez bármilyen okból elérhetetlenné válik, a rendszer nem fog tudni vezetőt választani.
- Egyetlen dedikált folyamat egyszerűen használható vezetőként. Ha azonban a folyamat leáll, jelentős késés léphet fel az újraindulásáig. Ez a késés hatással lehet a többi folyamat teljesítményére és válaszidejére, ha arra várnak, hogy a vezető koordináljon egy műveletet.
- Az egyik vezetőválasztási algoritmus manuális megvalósítása biztosítja a legnagyobb rugalmasságot a kód finomhangolásához és optimalizáláshoz.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Akkor használja ezt a mintát, ha egy elosztott alkalmazásban található feladathoz (például egy felhőben üzemeltetett megoldáshoz) gondos koordináció szükséges, és nincs természetes vezető.

> Kerülje el, hogy a vezető szűk keresztmetszet legyen a rendszerben. A vezető célja az alárendelt feladatok munkájának összehangolása. Magának a vezetőnek nem feltétlenül kell részt vennie a munkában, de képesnek kell lennie rá, mert előfordulhat, hogy nem ezt a feladatot választják meg vezetőnek.

Nem érdemes ezt a mintát használni, ha:

- Van természetes vezető vagy dedikált folyamat, amely mindig vezetőként működhet. Például lehetséges, hogy meg lehet valósítani egyetlen olyan folyamatot, amely a feladatpéldányokat koordinálja. Ha ez a folyamat meghibásodik, a rendszer le tudja állítani, és újra tudja indítani.
- A feladatok egy kisebb terhelést jelentő módszerrel is összehangolhatók. Ha például egyszerűen több feladatpéldánynak kell koordinált hozzáférést biztosítani egy közös erőforráshoz, akkor célszerűbb optimista vagy pesszimista zárolással vezérelni a hozzáférést.
- Egy külső megoldás megfelelőbb. Például az Apache Hadoopon alapuló Microsoft Azure HDInsight szolgáltatás az Apache Zookeeper szolgáltatásaival koordinálja a leképezési és csökkentési feladatokat, amelyek begyűjtik és összefoglalják az adatokat.

## <a name="example"></a>Példa

A LeaderElection megoldás DistributedMutex projektje (a mintát bemutató mintakód elérhető a [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) bemutatja, hogyan használható egy Azure Storage-blobbérlet egy közös, elosztott mutex megvalósításához szükséges mechanizmus biztosítására. Ezzel a mutexszel kiválasztható a vezető egy Azure-felhőszolgáltatásban lévő szerepkörpéldányok csoportjából. A bérletet elsőként megszerző szerepkörpéldány lesz megválasztva vezetőnek, és addig az is marad, amíg fel nem szabadítja a bérletet, vagy nem tudja megújítani. A többi szerepkörpéldány folytathatja a blobbérlet monitorozását, ha a vezető nem érhető el.

> A blobbérlet egy blob exkluzív írási zárolása. Egy blob mindig csak egy bérlet tárgya lehet. Egy szerepkörpéldány kérelmezheti egy adott blob bérletét, és meg is kapja, ha egyetlen másik szerepkörpéldány sem rendelkezik ugyanennek a blobnak a bérletével. Egyéb esetben a kérelem kivételt ad vissza.
>
> Ha el szeretné kerülni, hogy a hibás szerepkörpéldány határozatlan időre szerezze be a bérletet, adja meg a bérlet élettartamát. Amikor ez lejár, a bérlet elérhetővé válik. Azonban amíg egy szerepkörpéldány rendelkezik a bérlettel, kérelmezheti a megújítását, és hosszabb időre megkaphatja a bérletet. A szerepkörpéldány rendszeresen megismételheti ezt a folyamatot, ha meg kívánja őrizni a bérletet.
> További információ a blobbérletekről: [Blobbérlet (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx).

Az alábbi C#-példában látható `BlobDistributedMutex` osztály tartalmazza a `RunTaskWhenMutexAquired` metódust, amely lehetővé teszi egy szerepkörpéldány számára, hogy megpróbálja beszerezni egy adott blob bérletét. A rendszer egy `BlobSettings` objektumban adja át a blob adatait (név, tároló és tárfiók) a konstruktornak a `BlobDistributedMutex` objektum létrehozásakor (ez az objektum egy egyszerű struktúra, amelyet a mintakód is tartalmaz). A konstruktor egy `Task` elemet is fogad, amely arra a kódra hivatkozik, amelyet a szerepkörpéldánynak futtatnia kell, ha sikeresen beszerzi a blobbérletet, és megválasztják vezetőnek. Vegye figyelembe, hogy a bérlet beszerzésének kevésbé fontos részleteit kezelő kód egy különálló, `BlobLeaseManager` nevű segítőosztályban szerepel.

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

A fenti kódmintában szereplő `RunTaskWhenMutexAquired` metódus meghívja a következő kódmintában látható `RunTaskWhenBlobLeaseAcquired` metódust a bérlet beszerzéséhez. A `RunTaskWhenBlobLeaseAcquired` metódus aszinkron módon fut. Ha sikeresen beszerezte a bérletet, a szerepkörpéldány lesz megválasztva vezetőnek. A `taskToRunWhenLeaseAcquired` delegált feladata a többi szerepkörpéldányt koordináló munka elvégzése. Ha nem sikerül beszereznie a bérletet, egy másik szerepkörpéldány lesz megválasztva vezetőnek, és az aktuális szerepkörpéldány alárendelt marad. Vegye figyelembe, hogy a `TryAcquireLeaseOrWait` metódus egy olyan segédmetódus, amely a `BlobLeaseManager` objektumot használja a bérlet beszerzéséhez.

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

A vezető által elindított feladat szintén aszinkron módon fut. Amíg ez a feladat fut, az alábbi kódmintában látható `RunTaskWhenBlobLeaseAquired` metódus időnként megkísérli a bérlet megújítását. Ezzel biztosítható, hogy ez a szerepkörpéldány marad a vezető. A mintamegoldásban a megújítási kérelmek közötti késés kevesebb, mint a bérlethez beállított időtartam, hogy ne lehessen egy másik szerepkörpéldányt megválasztani vezetőnek. Ha a megújítás bármilyen okból nem sikerül, a feladat megszakad.

Ha a bérlet megújítása nem sikerül vagy a feladat megszakad (például a szerepkörpéldány leállása miatt), felszabadul a bérlet. Ekkor ez vagy egy másik szerepkörpéldány is megválasztható vezetőként. Az alábbi kódkivonat a folyamat ezen részét mutatja.

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

A `KeepRenewingLease` metódus egy másik olyan segédmetódus, amely a `BlobLeaseManager` objektumot használja a bérlet megújításához. A `CancelAllWhenAnyCompletes` metódus megszakítja az első két paraméterként megadott feladatokat. A következő diagram azt mutatja be, hogyan választható egy vezető és futtatható egy műveleteket koordináló feladat a `BlobDistributedMutex` osztály használatával.

![Az 1. ábra a BlobDistributedMutex osztály funkcióit szemlélteti](./_images/leader-election-diagram.png)

Az alábbi példakód bemutatja, hogyan használható a `BlobDistributedMutex` osztály feldolgozói szerepkörben. Ez a kód beszerzi a fejlesztési tárterület bérlettárolójában lévő `MyLeaderCoordinatorTask` blob bérletét, és megadja, hogy a `MyLeaderCoordinatorTask` metódusban meghatározott kódnak akkor kell futnia, ha a szerepkörpéldányt megválasztják vezetőnek.

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

Vegye figyelembe a következő szempontokat a mintaként szolgáló megoldásról:

- A blob egy potenciálisan hibaérzékeny pont. Ha a blobszolgáltatás nem áll rendelkezésre vagy nem érhető el, a vezető nem tudja megújítani a bérletet, és a többi szerepkörpéldány sem fogja tudni beszerezni a bérletet. Ebben az esetben egyik szerepkörpéldány sem tud vezetőként működni. Azonban a blobszolgáltatást úgy tervezték, hogy rugalmas legyen, ezért a blobszolgáltatás teljes leállása nagyon valószínűtlen.
- Ha a vezető által elvégzett feladat megáll, a vezető folytathatja a bérlet megújítását, így a többi szerepkörpéldány nem tudja beszerezni a bérletet és átvenni a vezetői szerepkört a feladatok koordinálása érdekében. A való világban a vezető állapotát gyakori időközönként ellenőrizni kell.
- A választási folyamat nem determinált. Nem jósolható meg, hogy melyik szerepkörpéldány fogja beszerezni a blobbérletet és válik ezáltal vezetővé.
- A blobbérlet céljaként használt blobot ne használja más célra. Ha egy szerepkörpéldány megpróbál adatokat tárolni ebben a blobban, ezek az adatok csak akkor lesznek elérhetők, ha a szerepkörpéldány a vezető, és rendelkezik a blobbérlettel.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi útmutatók segíthetnek a minta megvalósításakor:

- Ez a minta egy letölthető [mintaalkalmazást](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election) tartalmaz.
- [Útmutató az automatikus skálázáshoz](https://msdn.microsoft.com/library/dn589774.aspx). A feladatot futtató gazdagépek példányai az alkalmazások terhelésének változásainak megfelelően elindíthatók vagy leállíthatók. Az automatikus skálázás segítségével a teljesítmény és az átviteli sebesség szinten tartható a feldolgozási csúcsidőszakban.
- [Compute-particionálási útmutató](https://msdn.microsoft.com/library/dn589773.aspx) Ez az útmutató ismerteti, hogyan rendelhet feladatokat a gazdagépekhez egy felhőszolgáltatásban olyan módon, hogy az segítsen minimalizálni a futtatási költségeket a szolgáltatás skálázhatóságának, teljesítményének, rendelkezésre állásának és biztonságának megőrzése mellett.
- [Feladatalapú aszinkron minta](https://msdn.microsoft.com/library/hh873175.aspx).
- A [Bully algoritmust](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) ábrázoló példa.
- A [Ring algoritmust](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html) ábrázoló példa.
- Az [Apache Curator](https://curator.apache.org/), amely az Apache ZooKeeper ügyfélkódtára.
- Az MSDN [blobbérleteket (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) ismertető cikke.
