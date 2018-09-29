---
title: Compute Resource Consolidation
description: Egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
ms.openlocfilehash: bd212b8b4406a08058f811db030843f732e08cdc
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428839"
---
# <a name="compute-resource-consolidation-pattern"></a>Számításierőforrás-konszolidálási minta

[!INCLUDE [header](../_includes/header.md)]

Egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet. Ez megnövelheti a számítási erőforrások kihasználtságát, valamint csökkentheti a felhőben üzemeltetett alkalmazásokban végzett számítási feldolgozások végrehajtásához kapcsolódó költségeket és munkaterhelést.

## <a name="context-and-problem"></a>Kontextus és probléma

A felhőalkalmazások gyakran számos különböző műveletet végeznek. Néhány megoldásban eleinte érdemes a kockázatok elkülönítésének tervezési alapelvét követni, és felosztani ezeket a műveleteket különálló számítási egységekre, amelyek üzembe helyezése és üzemeltetése egymástól független (például különálló App Service webalkalmazások, különálló virtuális gépek vagy különálló Cloud Service szerepkörök). Bár ez a stratégia segíthet leegyszerűsíteni a megoldás logikai kialakítását, ha több számítási egységet helyez üzembe ugyanannak az alkalmazásnak a részeként, az megnövelheti a futtatókörnyezet üzemeltetési költségeit, illetve a rendszer kezelését bonyolultabbá teheti.

Például az ábrán egy olyan, felhőben üzemeltetett megoldás egyszerűsített struktúrája látható, amely több mint egy számítási egység használatával lett megvalósítva. Mindegyik számítási egység a saját virtuális környezetét futtatja. Mindegyik funkció külön feladatként lett megvalósítva (A–E feladat), amely a saját számítási egységét futtatja.

![Feladatok futtatása egy felhőkörnyezetben dedikált számítási egységek készletének használatával](./_images/compute-resource-consolidation-diagram.png)


Mindegyik számítási egység felszámítható erőforrásokat használ, akkor is, ha tétlen vagy kisebb terhelésű. Ezért ez nem mindig a legköltséghatékonyabb megoldás.

Az Azure-ban ez a probléma a Cloud Service, az App Services és a Virtual Machines szerepköreire vonatkozik. Ezek az elemek a saját virtuális környezetükben futnak. Ha olyan különálló szerepkörök, webhelyek vagy virtuális gépek összességét futtatja, amelyek jól körülírt műveletek egy készletének végrehajtására lettek tervezve, azonban egy megoldás részeiként kell kommunikálniuk és együttműködniük, lehetséges, hogy erőforrásait nem hatékonyan használja fel.

## <a name="solution"></a>Megoldás

A költségek csökkentése, a kihasználtság optimalizálása, a kommunikációs sebesség javítása és a kezelési szükségletek csökkentése érdekében egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet.

A feladatokat a környezet által biztosított funkciók és a funkciókhoz kapcsolódó költségek szerint megállapított kritériumok alapján csoportosíthatja. Általánosan használt megközelítés olyan feladatok keresése, amelyek hasonló profillal rendelkeznek a skálázhatóságuk, élettartamuk és feldolgozási követelményeik szempontjából. A csoportosításukkal egy egységként skálázhatja őket. A több felhőkörnyezet által nyújtott rugalmasság lehetővé teszi, hogy a számítási feladattól függően egy számítási egység további példányait indítsa el vagy állítsa le. Például az Azure használatával automatikusan skálázhatja a Cloud Service, az App Services vagy a Virtual Machines szerepköreit. További információért lásd az [automatikus skálázás útmutatóját](https://msdn.microsoft.com/library/dn589774.aspx).

Az alábbi két feladat segítségével ellenpéldaként bemutatjuk, hogyan állapítható meg a skálázhatósággal, hogy mely műveleteket nem érdemes csoportosítanunk:

- Az 1. feladat az üzenetsorba küldött ritka, nem időérzékeny üzeneteket kérdezi le.
- A 2. feladat a hálózati forgalom csúcsterheléseit kezeli.

A második feladathoz rugalmasság szükséges, amelyhez hozzátartozik a számítási egység számos példányának elindítása és leállítása is. Amennyiben ugyanezt a skálázást alkalmazná az első feladatra, az egyszerűen azt eredményezné, hogy több feladat figyelne ritka üzenetekre ugyanabban az üzenetsorban, így feleslegesen használna erőforrásokat.

Számos felhőkörnyezetben a számítási egységek számára elérhető erőforrások meghatározhatók a processzormagok számával, a memória és a lemezterület méretével, illetve egyéb értékekkel. Általában minél több erőforrás van meghatározva, annál nagyobb a költség is. Pénz megtakarításához fontos maximalizálni a költséges számítási egységek által végrehajtott munkát, illetve biztosítani, hogy ne legyen hosszabb időre inaktív.

Ha előfordulnak olyan feladatok, amelyeknek jelentős processzorteljesítményre van szükségük rövidebb időintervallumokban, fontolja meg ezek konszolidálását egy számítási egységbe, amely biztosítja a szükséges teljesítményt. Fontos azonban megtalálni az egyensúlyt a költséges erőforrások folyamatos működtetése és a túlterhelés által előidézett versengés között. A hosszan futó, nagy számítási igényű feladatok például ne osztozzanak ugyanazon a számítási egységen.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

Vegye figyelembe az alábbi pontokat a minta megvalósításakor:

**Méretezhetőség és rugalmasság**. Számos felhőalapú megoldás a számítási egység szintjén a skálázhatóságot és a rugalmasságot az egységek példányainak elindításával és leállításával valósítja meg. Ne csoportosítsa azokat a feladatokat egy számítási egységbe, amelyek skálázási követelményei ütköznek.

**Élettartam**. A felhőalapú infrastruktúra rendszeres időközönként újraindítja a számítási egységeket üzemeltető virtuális környezeteket. Ha számos hosszú lefutású feladat található egy számítási egységben, szükséges lehet úgy konfigurálni az egységet, hogy megakadályozza az újraindítását, míg ezek a feladatok be nem fejeződnek. Másik megoldásként a feladatokat ellenőrzőpontos megközelítéssel is megtervezheti, amely lehetővé teszi, hogy megfelelően leálljanak, majd arról a pontról folytatódjanak, ahol az adott folyamat megszakadt a számítási egység újraindításakor.

**Kiadási ütem**. Ha egy feladat megvalósítása vagy konfigurációja gyakran változik, szükséges lehet a frissített kódot üzemeltető számítási egység leállítása, az egység újrakonfigurálása és újbóli üzembe helyezése, majd újraindítása. Ehhez a folyamathoz a számítási egységben futtatott összes egyéb feladatot is le kell állítani, újból üzembe kell helyezni, majd újra kell indítani.

**Biztonság**. Az azonos számítási egységekben futtatott feladatok közös biztonsági környezettel rendelkezhetnek, illetve ugyanazokhoz az erőforrásokhoz férhetnek hozzá. A feladatoknak magas szintű megbízhatósági kapcsolatban kell lenniük, valamint meg kell bizonyosodniuk arról, hogy az egyik feladat nem fogja károsítani vagy negatívan befolyásolni a másikat. Emellett egy adott számítási egységben futtatott feladatok számának emelése megnöveli az egység támadható felületét. Ha egy feladat nem biztonságos, az a többi feladat biztonságát is rontja.

**Hibatűrés**. Ha egy feladat a számítási egységben meghibásodik vagy rendellenesen viselkedik, az befolyásolhatja az azonos egységen futtatott egyéb feladatokat is. Ha például egy feladat képtelen megfelelően elindulni, az a számítási egység teljes indítási logikájának meghibásodásához vezethet, megakadályozva az egységen a többi feladat futtatását is.

**Versengés**. Ne idézzen elő versengést olyan feladatok létrehozásával, amelyek egy adott számítási egységben osztoznak az erőforrásokon. Ideális esetben az egy számítási egységben futtatott feladatok különböző erőforrás-felhasználási jellemzőkkel rendelkeznek. Ne futtasson például két nagy számítási igényű feladatot ugyanabban a számítási egységben, valamint két sok memóriát felhasználó feladatot se. Azonban egy nagy számítási igényű feladat és egy sok memóriát igénylő feladat közös üzemeltetése működőképes kombináció.

> [!NOTE]
>  Próbálja csak olyan rendszerek számítási erőforrásait konszolidálni, amelyek már egy ideje éles környezetben üzemelnek, hogy a kezelők és fejlesztők a rendszer monitorozásával létrehozhassanak egy _intenzitástérképet_, amely azonosítja, hogy az egyes feladatok hogyan használják a különböző erőforrásokat. A térkép segítségével megállapíthatja, hogy mely feladatok között érdemes megosztani a számítási egységeket.

**Összetettség**. Amennyiben több feladatot kombinál egy számítási egységben, az megnöveli a kód összetettségét, ami esetleg megnehezítheti a tesztelést, a hibakeresést és a karbantartást.

**Stabil logikai architektúra**. A kódot úgy tervezze és valósítsa meg az egyes feladatokban, hogy akkor se kelljen módosítani, ha a feladatot futtató fizikai környezet megváltozik.

**Egyéb stratégiák**. A számítási erőforrások konszolidálása csak egy módja a több feladat egyidejű futtatásával járó költségek csökkentésének. Csak gondos tervezéssel és monitorozással biztosítható, hogy hatékony megközelítés maradjon. A munka természetétől és a feladatokat futtató felhasználók tartózkodási helyétől függően egyéb stratégiák megfelelőbbek lehetnek. A számítási feladatok funkcionális felbontása (a [Compute-particionálási útmutatónak](https://msdn.microsoft.com/library/dn589773.aspx) megfelelően) például jobb megoldás lehet.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ezt a mintát olyan feladatok esetében alkalmazza, amelyek nem költséghatékonyak, ha a saját számítási egységeikben futnak. Ha egy feladat az idő nagy részében tétlen, a feladat futtatása egy dedikált egységen költséges lehet.

Ez a minta lehet, hogy nem megfelelő olyan feladatokhoz, amelyek kritikus hibatűrő műveleteket hajtanak végre, illetve rendkívül bizalmas vagy személyes adatokat dolgoznak fel, és saját biztonsági környezetre van szükségük. Ezeket a feladatokat a saját elkülönített környezetükben érdemes futtatni, különálló számítási egységben.

## <a name="example"></a>Példa

Amikor felhőszolgáltatást fejleszt az Azure-ban, a több feladat által végzett feldolgozást egyetlen szerepkörbe konszolidálhatja. Általában ez egy feldolgozói szerepkör, amely háttérben történő vagy aszinkron feldolgozási feladatokat végez.

> Néhány esetben a háttérben történő vagy aszinkron feldolgozási feladatokat belefoglalhatja a webes szerepkörbe. Ezzel a módszerrel csökkentheti a költségeket és egyszerűsítheti az üzembe helyezést, azonban ez befolyásolhatja a webes szerepkör által biztosított nyilvános felület skálázhatóságát és válaszképességét. 

Ez a szerepkör felelős a feladatok indításáért és leállításáért. Amikor az Azure Fabric Controller betölt egy szerepkört, kiváltja a szerepkör `Start` eseményét. Felülírhatja a `WebRole` vagy `WorkerRole` osztály `OnStart` metódusát, hogy kezelje az eseményt, például azoknak az adatoknak és egyéb erőforrásoknak az inicializálásával, amelyektől a metódus feladatai függnek.

Ha a `OnStart` metódus befejeződik, a szerepkör elkezdhet válaszolni a kérésekre. Az `OnStart` és a `Run` metódus szerepkörben történő használatáról további információkat és útmutatást az [alkalmazások felhőbe való áthelyezését tárgyaló](https://msdn.microsoft.com/library/ff728592.aspx) minta- és gyakorlati útmutató [alkalmazásindítási folyamatokról szóló](https://msdn.microsoft.com/library/ff803371.aspx#sec16) szakaszában talál.

> Az `OnStart` metódus kódja legyen a lehető legtömörebb. Az Azure nem határoz meg küszöbértéket a metódus befejezésére engedélyezett időtartamnak, azonban a szerepkör nem kezdi megválaszolni az elküldött hálózati kéréseket, amíg a metódus be nem fejeződik.

Ha az `OnStart` metódus befejeződött, a szerepkör végrehajtja a `Run` metódust. Ekkor a hálóvezérlő megkezdi a kérések küldését a szerepkörnek.

Helyezze a feladatokat ténylegesen létrehozó kódot a `Run` metódusba. Vegye figyelembe, hogy a `Run` metódus határozza meg a szerepkörpéldány élettartamát. A metódus befejezésekor a hálóvezérlő kezdeményezi a szerepkör leállítását.

Amikor egy szerepkör leáll vagy újraindul, a hálóvezérlő megakadályozza, hogy további bejövő kérések érkezzenek a terheléselosztóból, és kiváltja a `Stop` eseményt. Ezt az eseményt a szerepkör `OnStop` metódusának felülbírálásával rögzítheti, majd hajtsa végre a szükséges tisztítási műveleteket a szerepkör leállítása előtt.

> Az `OnStop` metódusban végrehajtott összes műveletet öt percen belül be kell fejezni (vagy 30 másodpercen belül, ha Azure emulátort használt helyi számítógépen). Ellenkező esetben az Azure Fabric Controller azt feltételezi, hogy a szerepkör elakadt, és leállásra kényszeríti.

A feladatokat a `Run` metódus indítja el, amely megvárja a feladatok befejezését. A feladatok megvalósítják a felhőszolgáltatás üzleti logikáját, és a szerepkörnek küldött üzeneteket az Azure Load Balanceren keresztül válaszolják meg. Az ábrán egy Azure-felhőszolgáltatás szerepköréhez tartozó feladatok és erőforrások életciklusa látható.

![Egy Azure-felhőszolgáltatás szerepköréhez tartozó feladatok és erőforrások életciklusa](./_images/compute-resource-consolidation-lifecycle.png)


A _WorkerRole.cs_ fájl a _ComputeResourceConsolidation.Worker_ projektben arra mutat be egy példát, hogyan valósíthatja meg ezt a mintát egy Azure felhőszolgáltatásban.

> A _ComputeResourceConsolidation.Worker_ projekt a _ComputeResourceConsolidation_ megoldás része, amely letölthető a [GitHubról](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).

A `MyWorkerTask1` és a `MyWorkerTask2` metódus bemutatja, hogyan hajthat végre különböző feladatokat ugyanabban a feldolgozói szerepkörben. Az alábbi kódban a `MyWorkerTask1` látható. Ez egy egyszerű feladat, amely 30 másodpercig alvó állapotba lép, majd elküld egy nyomkövetési üzenetet. Ez a folyamat a feladat megszakításáig ismétlődik. A `MyWorkerTask2` kódja hasonló.

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

> A mintakód a háttérfolyamatok egy gyakori megvalósítását mutatja be. Egy valós alkalmazásban követheti ugyanezt a struktúrát, azonban helyezze a saját feldolgozási logikáját a megszakítási kérelemre váró hurok törzsébe.

Miután a feldolgozói szerepkör inicializálta a felhasználni kívánt erőforrásokat, a `Run` metódus egyszerre elindítja a két feladatot az itt látható módon.

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

Ebben a példában a `Run` metódus a feladatok befejezésére vár. Ha a feladat meg lett szakítva, a `Run` metódus azt feltételezi, hogy a szerepkör leáll, és megvárja a hátralévő feladatok megszakítását a befejezés előtt (legfeljebb öt percet vár a leállításra). Ha a feladat egy várt kivétel miatt hiúsul meg, a `Run` metódus megszakítja a feladatot.

> Olyan átfogóbb monitorozási és kivételkezelési stratégiákat is megvalósíthat a `Run` metódusban, mint például a meghiúsult feladatok újraindítása, illetve olyan kód beillesztése, amely engedélyezi a szerepkör számára egyes feladatok leállítását és elindítását.

A következő kódban látható `Stop` metódust a rendszer akkor hívja meg, amikor a hálóvezérlő leállítja a szerepkörpéldányt (az `OnStop` metódusból indítva). A kód minden feladatot szabályosan állít le azok megszakításával. Ha a feladat befejezése több mint öt percet vesz igénybe, a `Stop` metódus megszakítási feldolgozása nem vár tovább, és a szerepkört leállítja.

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

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Útmutató az automatikus skálázáshoz](https://msdn.microsoft.com/library/dn589774.aspx). Az automatikus skálázás használatával szolgáltatásokat üzemeltető számítási erőforrásokat indíthat el vagy állíthat le a várható feldolgozási igényektől függően.

- [Compute-particionálási útmutató](https://msdn.microsoft.com/library/dn589773.aspx) Bemutatja, hogyan foglalhat le szolgáltatásokat és összetevőket egy felhőszolgáltatásban olyan módon, hogy az segítsen minimalizálni a futtatási költségeket a szolgáltatás skálázhatóságának, teljesítményének, rendelkezésre állásának és biztonságának megőrzése mellett.

- Ez a minta egy letölthető [mintaalkalmazást](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) tartalmaz.
