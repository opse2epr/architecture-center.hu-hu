---
title: "Számítási erőforrás összevonása"
description: "Több feladatokat vagy műveleteket egyetlen számítási egységbe összesítése"
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: design-implementation
ms.openlocfilehash: 85191fc630549559f8a1395e5a8622a7a6140a2d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="compute-resource-consolidation-pattern"></a>Számítási erőforrás-összevonási minta

[!INCLUDE [header](../_includes/header.md)]

Több feladatokat vagy műveleteket egyetlen számítási egységbe egyesíteni. Ez növeli a számítási erőforrások kihasználtságát, és csökkentik a költségeket és a felügyeleti terhelést hajt végre a felhőben üzemeltetett alkalmazások feldolgozás alatt álló számítási társított.

## <a name="context-and-problem"></a>A környezetben, és probléma

A felhőalapú alkalmazások gyakran valósítja meg a különféle műveletek. Néhány érdemes aggályokat elkülönítésének tervezési elvét kezdetben kövesse, és ossza meg ezeket a műveleteket az üzemeltetett és külön-külön (például külön App Service web apps – külön, telepített külön számítási egység Virtuális gépek, vagy külön Felhőszolgáltatási szerepkörök). Azonban ezt a stratégiát megkönnyítheti a logikai a megoldás kialakításának, bár telepítése számos számítási egység ugyanahhoz az alkalmazáshoz részeként is üzemeltetési költségek futásidejű növelése és győződjön a rendszer felügyeleti összetettebb.

Tegyük fel az ábrán látható az egyszerűsített struktúra a felhőben üzemeltetett megoldás, amely több számítási egység segítségével van megvalósítva. Minden számítási egység a saját virtuális környezetben futtatja. Minden egyes függvény van megvalósítva (– a feladat-E a tevékenység neve) külön feladatként futtatja a saját számítási egység.

![Az éppen futó feladatok egy felhőalapú környezetben használatával dedikált számítási egység](./_images/compute-resource-consolidation-diagram.png)


Minden számítási egység terhelhető erőforrásokat használ fel, akkor is, amikor az üresjárati vagy enyhén használt. Ezért ezt nem mindig a leginkább költséghatékony megoldás.

Ezen probléma Azure, a egy felhőalapú szolgáltatás, az App Service szolgáltatások és virtuális gépek szerepkörök vonatkozik. Ezeket az elemeket a saját virtuális környezetben futtatja. Külön szerepköröket, a webhelyek vagy a virtuális gépek, amelyek jól meghatározott műveletkészlet végrehajtásához, de a, amelyeknek szükségük van a kommunikációhoz, és egyetlen megoldás részeként együttműködnek gyűjteménye fut, lehet, hogy az erőforrások hatékony használatát.

## <a name="solution"></a>Megoldás

Segítségével csökkentheti a költségeket, növelje a kihasználtsági, kommunikációs sebesség növelése és csökkentse a felügyeleti rendszer több feladatokat vagy műveleteket egyetlen számítási egységbe vonják össze.

Feladatok a környezet által nyújtott szolgáltatásokat, és ezek a funkciók kapcsolódó költségeket alapján feltételek szerint csoportosíthatók. Egy közös megoldás, amelyet meg kíván keresni egy hasonló profilhoz, a méretezhetőséget, a élettartamát és az ügyféloldali bővítmények feldolgozási követelményeivel kapcsolatos feladatok. Csoportosítás ezen együtt lehetővé teszi a méretezési egységet. A sok felhőkörnyezetekben által biztosított rugalmassága lehetővé teszi, hogy egy számítási egység indítása és leállítása a munkaterhelés szerint további példányait. Például az Azure biztosít automatikus skálázás alkalmazható a szerepkörökhöz a egy felhőalapú szolgáltatás, az App Service szolgáltatások és virtuális gépek. További információkért lásd: [automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx).

Egy számláló példaként hogyan méretezhetőség segítségével határozza meg, mely műveletek nem csoportosított megjelenítendő vegye figyelembe az alábbi két feladatot:

- 1. feladat annak a várólistára küldött alkalomszerű, idő-és nagybetűk megkülönböztetése nélkül üzenet kérdezi le.
- 2. feladat nagy mennyiségű felszakadásáig hálózati forgalmat kezeli.

A második feladathoz, amely magába foglaló indítása és leállítása a nagy mennyiségű példánnyal a számítási egység rugalmasság. Az azonos méretezés alkalmazása az első lépése egyszerűen eredményeznének további feladatok a ugyanazon a várólistán alkalomszerű üzeneteket figyeli, és a pazarlás erőforrások.

Sok felhőalapú környezetben is lehet egy számítási egységhez CPU mag, memória, lemezterület, és így tovább számára elérhető erőforrások megadására. Általában a megadott, további erőforrások minél nagyobb a költségeket. Kevesebbet költeni, fontos egy drága számítási egység hajt végre, és azt válnak inaktívvá hosszú időn keresztül a munkahelyi maximalizálása érdekében.

Ha vannak, amelyek szükségesek a jelentős mértékben a CPU power rövid felszakadásáig, fontolja meg, ezek a szükséges teljesítményt biztosító egyetlen számítási egységbe konszolidálása. Fontos azonban drága erőforrások foglalt szemben a versengés, amely fordulhat elő, ha azok vannak keresztül hangsúlyozni tartása igénynek elosztása. Hosszan futó, a számítás-igényes feladatok például az azonos számítási egység nem szabad megosztani.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában végrehajtásakor, vegye figyelembe a következő szempontokat:

**Méretezhetőség és a rugalmasság**. Sok felhőalapú megoldás megvalósíthatja méretezhetőség és a rugalmasság a számítási egység szintjén indítása és leállítása egységek példányai. Kerülje a csoportosítási feladatok, amelyek azonos számítási egységben ütköző méretezhetőségi követelményeinek.

**Élettartam**. A felhőalapú infrastruktúra rendszeres időközönként újraindul egy számítási egység üzemeltető virtuális környezetben. Ha sok hosszan futó feladatokat egy számítási egység belül, ezáltal megakadályozhatja, hogy újra lesznek hasznosítva, amíg a feladatok befejezése egység konfigurálása szükség lehet. Azt is megteheti tervezze meg a feladatok a jelölőnégyzet mutató megközelítés, amely lehetővé teszi, hogy nem szabályszerűen, és azokat a számítási egység újraindításakor program megszakította a pontnál folytatják használatával.

**Kiadás ütemben történik**. Ha a végrehajtása vagy a feladat konfigurációját változik gyakran, állítsa le a frissített kódot futtató számítási egység, konfigurálja újra, és telepítse újra az egység, és indítsa újra szükség lehet. Ez a folyamat is szükséges, hogy az azonos számítási egység minden más feladat leállt, újratelepíteni, és újraindításra kerülnek.

**Biztonság**. Az azonos számítási egység feladatokat előfordulhat, hogy ossza meg ugyanazt a biztonsági környezetet, majd fogja tudni elérni a ugyanazokat az erőforrásokat. A feladatok, és abban, hogy közötti megbízhatósági kapcsolat magas fokú kell lennie, hogy egy tevékenység nem sérült, vagy egy másik kedvezőtlen hatással lesz. Emellett egy számítási egységben futó feladatok számának növelése növeli a támadási felületet a egység. Minden feladat csak olyan biztonságos, mint amelyen a legtöbb biztonsági rések.

**Hibatűrés**. Számítási egység egy feladat sikertelen lesz, vagy rendkívül viselkedik-e, ha ez befolyásolhatja a más, ugyanazon egységen belül futó feladatok. Például ha egy feladat nem indítható okozhat a számítási egység sikertelen lesz a teljes ügyfélindítási logikája, és más feladatok azonos egységben tiltsa le a futását.

**A versengés**. Kerülje a feladatokat, amelyek "versenyeznek" az számítási egységen belüli erőforrások közötti versengés bemutatása. Ideális esetben az azonos számítási egység tevékenységeket kell mutatnak a különböző Erőforrás kihasználtsága. Például két számítási-igényes feladatok valószínűleg nem találhatók a azonos számítási egység, és nem kell a két feladatot, amely nagy mennyiségű memóriát használ. Azonban jelentős mennyiségű memóriát igénylő számítási intenzív tevékenység keverése nem alkalmazható kombinációját.

> [!NOTE]
>  Vegye figyelembe a számítási erőforrásokat csak egy rendszer számára, hogy az operátorok és a fejlesztők a figyelheti a rendszer, és hozzon létre egy adott időn belül az éles környezetben ennyi ideig konszolidálása egy _hőtérkép_ , amely azonosítja, hogyan használja minden feladat eltérő erőforrások. Ez a térkép határozza meg, milyen feladatokat a számítási erőforrások megosztása esetén használható jól használható.

**Összetettsége**. Több feladat egyetlen számítási egységbe egyesítése összetettségét hozzáadja a kódot a egységben, valószínűleg megnehezíti tesztelése, hibakeresését és karbantartása.

**Logikai architektúra stabil**. A tervezést, és minden feladat valósítja meg a kódot, így azt nem kell módosítania, még akkor is, ha a feladat fut a fizikai környezet módosítása.

**Más stratégiák**. Számítási erőforrások konszolidálása csak egy módja több feladat egyidejű futtatásával kapcsolatos költségek csökkentése érdekében. Körültekintően megtervezve és figyelés győződjön meg arról, hogy marad egy hatékony módszert igényel. Előfordulhat, hogy más stratégiák pontosabb, attól függően, hogy a munkahelyi és hol találhatók a felhasználók, a feladatok futnak. A munkaterhelés például működési felbontás ellen (szerint a [particionálás útmutatást számítási](https://msdn.microsoft.com/library/dn589773.aspx)) lehet, hogy jobb megoldás.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában nincsenek költséghatékony, ha futtatja a saját számítási egység feladatokhoz használhatja. Ha egy feladat nagy részét az üresjárati időt tölt, költséges lehet ez a feladat fut egy dedikált egységben.

Előfordulhat, hogy ez a minta nem megfelelő-e a feladatokat, kritikus hibatűrő műveleteket, illetve a feladatokat, amelyek rendkívül bizalmas vagy személyes adatok feldolgozása és a saját biztonsági környezet szükséges. Ezeket a feladatokat egy külön számítási egység a saját elkülönített környezetben fusson.

## <a name="example"></a>Példa

Azure cloud Service szolgáltatásra fejlesztéskor is lehet összevonása akkor történjen meg több feladatok, irányítson át egyetlen szerepkör feldolgozását. Ez általában egy feldolgozói szerepkört, amely a háttér vagy aszinkron feladatokat hajt végre.

> Néhány esetben lehetőség a háttér vagy aszinkron feldolgozási feladatok a webes szerepkör. Ez a módszer segítségével csökkentheti a költségeket és egyszerűbbé telepítését, bár ez hatással lehet a méretezhetőség és a nyilvánosan elérhető felülete, a webes szerepkör válaszképességét. A cikk [egyesítésével több Azure feldolgozói szerepkörök be egy Azure webes szerepkör](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) háttér vagy aszinkron feldolgozási feladatainak valósít meg a webes szerepkör részletes leírását tartalmazza.

A szerepkör indítása és leállítása a feladatok felelős. Egy szerepkört az Azure fabric controller betöltésekor, a riasztást a `Start` esemény a szerepkörhöz. Felülírhatja a `OnStart` metódusában a `WebRole` vagy `WorkerRole` osztály kezelni ezt az eseményt, lehet, hogy inicializálni az adatokat és más erőforrások függenek a feladatokat az ezzel a módszerrel.

Ha a `OnStart `metódus befejeződött, a szerepkör elindíthatja válaszol a kérelmekre. További információt és útmutatást segítségével megtalálhatja a `OnStart` és `Run` módszerek a szerepkörnek a [alkalmazás indítási folyamat](https://msdn.microsoft.com/library/ff803371.aspx#sec16) a minták és gyakorlatok guide című [alkalmazások áthelyezése a felhőbe](https://msdn.microsoft.com/library/ff728592.aspx).

> A kód tartsa a `OnStart` , pontos sikerkritériumokkal biztosíthatja a lehető metódust. Azure nem ugyanazok a metódus végrehajtásához szükséges idő a bármely korlátozását, de a szerepkör nem indítható el, amíg befejeződik ez a módszer küldött hálózati kérésekre válaszol.

Ha a `OnStart` metódus befejeződött, a szerepkör végrehajtja a `Run` metódust. Ezen a ponton a fabric controller elindíthatja kérelmek küldése a szerepkört.

A kódot, amely ténylegesen hoz létre a feladatokat a `Run` metódust. Vegye figyelembe, hogy a `Run` metódus a szerepkörpéldányt élettartamát határozza meg. Ez a módszer befejezését követően a fabric controller szervez le kell állítani a szerepkörhöz.

Szerepkör leáll vagy újrahasznosítása esetén a fabric controller megakadályozza, hogy a terheléselosztó és vált ki fogadott több bejövő kérelmek a `Stop` esemény. Ez az esemény felülbírálásával rögzítheti a `OnStop` metódus a szerepkör, és végezze el minden tidying előtt megszakítja a szerepkör szükséges.

> Semmilyen műveletet végez a `OnStop` metódus öt perc (vagy 30 másodperces használata az Azure emulátorban a helyi számítógép) belül el kell végezni. Ellenkező esetben az Azure fabric controller feltételezi, hogy a szerepkör leállását észlelte, és arra kényszeríti, hogy állítsa le.

A feladatok által elindított a `Run` módszer, amely a feladatok befejezésére vár. A feladatok valósítja meg az üzleti logika, a felhőalapú szolgáltatás, és képes válaszolni a szerepkört az Azure load balancer keresztül küldött üzeneteket. Az ábrán láthatók az életciklus feladatok és erőforrások olyan szerepkörben, az Azure-felhőszolgáltatás.

![Az életciklus feladatok és erőforrások olyan szerepkörben, az Azure-felhőszolgáltatás](./_images/compute-resource-consolidation-lifecycle.png)


A _WorkerRole.cs_ fájlt a _ComputeResourceConsolidation.Worker_ projekt szemlélteti, hogyan lehet, hogy implementálja ebben a mintában az Azure-felhőszolgáltatás.

> A _ComputeResourceConsolidation.Worker_ projekt része a _ComputeResourceConsolidation_ letölthető megoldás [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).

A `MyWorkerTask1` és a `MyWorkerTask2` módszerek bemutatják, hogyan lehet belül az azonos feldolgozói szerepkör különböző feladatok végrehajtására. A következő kódot tartalmazza `MyWorkerTask1`. Ez egy egyszerű feladat 30 másodpercig alvó állapotba kerül, majd exportálja a nyomkövetési üzenet. Ez a folyamat ismétlődik, amíg a feladat meg lett szakítva. A kód `MyWorkerTask2` hasonló.

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> A mintakód bemutatja egy közös végrehajtása háttérfolyamatként. Egy valós alkalmazás követésével Ez ugyanaz a struktúra, azzal a különbséggel, hogy a saját feldolgozó logika helyezze a hurok, amely megvárja a megszakítási kérés törzsében.

A feldolgozói szerepkör inicializálta a által használt erőforrások után a `Run` metódus elindítja a két feladatot egyszerre, ahogy az itt látható.

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

Ebben a példában a `Run` metódus megvárja-e a feladatot végrehajtani. Ha a feladat meg lett szakítva, a `Run` metódus feltételezi, hogy a szerepkör leállítása folyamatban van, és megvárja-e a befejezése előtt megszakítandó feladataiban (megvárja, legfeljebb öt perc megszakítása előtt). Ha egy feladat nem várt kivétel miatt a `Run` metódus megszakítja a feladatot.

> Szélesebb körű figyelési és stratégiák a kivételkezelő sikerült megvalósítása a `Run` módszer például a sikertelen feladatok újraindítása, vagy beleértve a kódot, amely lehetővé teszi a szerepkör leállítása és elindítása az egyes feladatok.

A `Stop` az alábbi kódban látható metódus lehívásra kerül, amikor a szerepkör példánya leáll a háló vezérlő (a meghívták a `OnStop` metódus). A kód minden feladat szabályosan leállítja törli azt. Ha a feladat több mint öt percet is igénybe vehet, feldolgozás alatt álló ezzel a `Stop` metódus várakozási megszűnik, és a szerepkör a rendszer megszakítja.

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:

- [Automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx). Automatikus skálázás indítása és leállítása a számítási erőforrásokat, attól függően, hogy a várható igény szerinti feldolgozásra üzemeltetési szolgáltatás példányának használható.

- [Útmutatás particionálás számítási](https://msdn.microsoft.com/library/dn589773.aspx). Ismerteti, hogyan lehet a szolgáltatások és összetevők a felhőalapú szolgáltatás, ezáltal segít a méretezhetőséget, teljesítményt, rendelkezésre állási és a szolgáltatás biztonsági megőrzésével futó költségek minimalizálása érdekében a lefoglalni.

- Ebben a mintában tartalmaz egy letölthető [mintaalkalmazás](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).
