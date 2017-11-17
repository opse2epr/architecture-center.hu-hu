---
title: "Áramköri megszakító"
description: "Kezeli olyan hárítsa el a távoli szolgáltatás vagy az erőforrás való csatlakozáskor a változó időt is igénybe vehet."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: ce110d0bbda600575d328895f2feca5aa253479d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="circuit-breaker-pattern"></a>Áramköri megszakító minta

Kezeli olyan helyreállítani, a változó időt is igénybe vehet a távoli szolgáltatás vagy az erőforrás történő csatlakozás során. Ez javítja a stabilitás és a rugalmasságot az alkalmazás.

## <a name="context-and-problem"></a>A környezetben, és probléma

Elosztott környezetben a távoli erőforrásokhoz és szolgáltatásokhoz hívások megakadályozhatják átmeneti hibák, például lassú hálózati kapcsolatoknál, időtúllépések vagy az erőforrásokat, amelyek túljegyzett vagy átmenetileg nem érhető el. Ezek a hibák általában javítsa magukat egy rövid időn belül, és a hatékony felhő alkalmazás stratégia használatával kezelendő kell készíteni a [újrapróbálkozási mintát][retry-pattern].

Azonban is lehet olyan helyzetekben, ahol hibák vannak a nem várt esemény miatt, és, hogy sokkal hosszabb ideig is tarthat megoldásával kapcsolatban. Ezek a hibák között szolgáltatás teljes hiba súlyossága a egy részleges a kapcsolat megszakadása lehet. Ezekben a helyzetekben előfordulhat, hogy az alkalmazás számára, amely nem valószínű, hogy a sikeres művelet folyamatosan újra értelmetlen, és ehelyett az alkalmazás kell gyorsan fogadja el, hogy a művelet sikertelen volt, és ennek megfelelően kezeli ezt a hibát.

Emellett, ha a szolgáltatás foglalt, a rendszer egy része sikertelen vezethet kaszkádolt hibák. Például egy művelet, amely meghívja a szolgáltatást sikerült konfigurálni egy időtúllépési bevezetéséhez, és válaszadás egy hibaüzenet, ha a szolgáltatás nem válaszol az ez idő alatt. Ezt a stratégiát azonban sok egyidejű kérés művelethez az időkorlát eléréséig letiltása esetén. Ezek a blokkolt kérelmek előfordulhat, hogy tartsa a kritikus rendszer-erőforrások, például a memória, a szálak, a Helyadatbázis-kapcsolatot és a stb. Ezért ezeket az erőforrásokat sikerült válnak kimerül, más szeretné használni, ugyanazokat az erőforrásokat a rendszer valószínűleg független részeinek hibája okozza. Ebben az esetben a művelet nem sikerül azonnal, és csak meghívására történt kísérlet a szolgáltatás valószínűleg sikeres lesz esetén előnyösebb lenne. Vegye figyelembe, hogy rövidebb időkorlátnak súgó a probléma, de az időtúllépés nem lehet, hogy a művelet sikertelen lesz az esetek többségében rövid akkor is, ha a a szolgáltatásnak küldött kérelemben volna végül sikeres lehet.

## <a name="solution"></a>Megoldás

Az áramköri megszakító mintát, Michael Nygard által a könyv popularized [kiadás azt!](https://pragprog.com/book/mnee/release-it), megakadályozhatja, hogy az alkalmazás a ismételten valószínűleg hiba fog előfordulni egy művelet végrehajtása közben. Várakozás nélkül lehetővé téve a rögzített vagy általi lefoglalását CPU-ciklusok kell, amíg azt meghatározza, hogy a tartalék hosszú hiba tartó. Az áramköri megszakító mintát is lehetővé teszi a az alkalmazás észleléséhez, hogy megoldódott-e a hibát. Ha a probléma úgy tűnik, hogy a javított, az alkalmazás megpróbálja meghívni a műveletet.

> Az áramköri megszakító mintát célja az újrapróbálkozási mintát eltér. Az újrapróbálkozási mintát lehetővé teszi, hogy egy alkalmazást, majd ismételje meg a várt értéket, amely akkor lesz sikeres a művelet. Az áramköri megszakító mintát megakadályozza, hogy az alkalmazás, amely valószínűleg hiba fog előfordulni művelet végrehajtása a. Egy alkalmazás ezen két minták kombinálhatja újrapróbálkozási minta meghívni egy áramköri megszakító keresztül művelet használatával. Azonban az újrapróbálkozási logika legyen az áramköri megszakító által visszaadott kivételek-és nagybetűket, és abandon újrapróbálkozások, ha az áramköri megszakító azt jelzi, hogy a hibát nem átmeneti.

Egy áramköri megszakító proxyként viselkedik, előfordulhat, hogy eleget nem tevő műveletekhez. A proxy kell történt, és ezen információk használatával határozza meg, hogy a művelet folytatásához engedélyezze a legújabb hibák számának figyelése, vagy egyszerűen küldött azonnal kivételt.

A proxy, a következő állapotokkal egy elektromos áramköri megszakító működésére nézve, amelyek egy állapotgép valósítható meg:

- **Lezárt**: az alkalmazás a kérelem annak biztosítására, hogy a műveletet. A proxy fenntartja a száma a legutóbbi hibákat, és ha a művelet hívása sikertelen a proxy növeli a számláló értéke. A legújabb hibák száma meghaladja a megadott küszöbértéket egy adott időtartamon belül, ha a proxy elhelyezi a **nyitott** állapota. Ezen a ponton a proxy elindul egy időtúllépési számlálót, és idő letelte után a proxy elhelyezi a **félig nyitott** állapotát.

    > Az időtúllépés időzítő célja próbálja megoldani a problémát, mielőtt engedélyezné az alkalmazást újra a műveletet a hibát okozó rendszer ideje.

- **Nyissa meg**: A kérelmet az alkalmazás azonnal leáll, és kivételt küld vissza az alkalmazást.

- **Váltakozó nyitott**: az alkalmazás kérelmeinek korlátozott számú engedélyezettek továbbítja, és meghívja a műveletet. Ha ezek a kérelmek sikeres, feltételezzük, hogy a hiba, amely korábban következtében a hiba kijavítása, és az áramköri megszakító vált a **lezárva** állapota (a sikertelen kísérletek számlálója alaphelyzetbe állítása). Ha a kérelmet, sikertelen, az áramköri megszakító azt feltételezi, hogy a hiba továbbra is jelen-e, hogy a rendszer visszatér a **nyitott** állapot és az időtúllépés időzítő ahhoz, hogy megkapja a további időt állítsa helyre a hibát a rendszer újraindul.

    > A **félig nyitott** állapota akkor hasznos, ezáltal megakadályozhatja, hogy a helyreállítási szolgáltatás által érintett hirtelen túlterhelve. Szolgáltatás állítja helyre, akkor előfordulhat, hogy lehet támogatja a korlátozott mennyiségű kérést, amíg a helyreállítás, de a folyamatban lévő helyreállítás alatt a munkahelyi áramlik sok túllépi az időkorlátot a szolgáltatás, vagy végezzen ismét feladat.

![Áramköri megszakító állapotok](./_images/circuit-breaker-diagram.png)

Az ábrán a sikertelen kísérletek számlálója használják a **lezárva** állapota időpontokat. Automatikusan visszaáll a rendszeres időközönként. Ez segít megakadályozni az áramköri megszakító belépjen a **nyitott** állapot, ha azt észlel alkalmi hibák. A hiba küszöbértékét, amely utazás közben az áramköri megszakító be a **nyitott** állapot csak akkor érhető el, amikor adott számú hiba történt egy adott időszakban. A számláló által használt a **félig nyitott** állapotát rögzíti a hívási a művelet sikeres próbálkozások számát. Az áramköri megszakító visszatér a **lezárva** állapot után a megadott számú egymást követő műveletek meghívása sikeresek voltak. Bármely hívás sikertelen lesz, ha beírja-e az áramköri megszakító a **nyissa meg** azonnal állapotát, és a sikeres a számláló a következő alkalommal, akkor visszaáll a **félig nyitott** állapotát.

> Hogyan állítja helyre a rendszer kezeli kívülről, valószínűleg visszaállítása vagy újraindítása sikertelen összetevő vagy a hálózati kapcsolat javítása.

Az áramköri megszakító mintát stabilitását biztosít, miközben a rendszer egy történt hiba után állítja helyre, és minimálisra csökkenti a teljesítményre gyakorolt hatása. Ez a Súgó gombra a rendszer a válaszidő karbantartása gyorsan visszautasítja a kérelmet, amely valószínűleg hiba fog előfordulni művelet, hanem a Várakozás a művelet időtúllépés, vagy soha nem adja vissza. Az áramköri megszakító kivált egy eseményt minden alkalommal, amikor az állapot változik, ha ezt az információt a részét a rendszer az áramköri megszakító által védett állapotának figyelésére, illetve riasztást a rendszergazda, ha egy áramköri megszakító utazás közben történő használható-e a **megnyitása** állapotát.

A minta személyre szabható legyen, és ezeket szerint, a lehetséges hiba. Például egy növekvő időtúllépés időzítő egy áramköri megszakító alkalmazhat. Az áramköri megszakító a sikerült elhelyezni a **nyitott** néhány másodpercen belül kezdetben állapotát, és ezután Ha a hiba nem sikerült megoldani az időtúllépési néhány percet, és így tovább. Egyes esetekben nem pedig a **nyitott** állapot adatszolgáltató hiba és kivételt, ez például akkor lehet hasznos, amely elsősorban az alkalmazás az alapértelmezett értéket.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ez a kialakítás megvalósítása meghatározásakor a következő szempontokat kell figyelembe vennie:

**Kivételkezelés**. Egy alkalmazás egy áramköri megszakító keresztül művelet meghívása kezelni a kivételek jelenik meg, ha a művelet nem érhető el kell készíteni. Kivételek kezelése lesz az adott alkalmazás. Például egy alkalmazás sikerült ideiglenesen csökkentheti a funkciókat, végrehajthatja ugyanezt a feladatot, vagy az beszerzése ugyanazokat az adatokat, a másik művelet meghívása vagy jelentse a kivétel a felhasználó és kérje meg, hogy később próbálja meg újból.

**Kivételek típusú**. A kérelem sikertelen lehet, amelyek jelezhetik a többinél szigorúbb típusú hiba számos oka. Például egy kérelem meghiúsulhat, mert a távoli szolgáltatás összeomlott és helyreállítása, néhány percet vesz igénybe, vagy mert a szolgáltatás ideiglenesen túlterhelt alatt időtúllépés miatt. Egy áramköri megszakító megvizsgálhatja a kivételeket, és állítsa be ezeket a kivételeket természetétől függően a stratégia típusú lehet. Például akkor lehet szükség időtúllépési kivételek az áramköri megszakító való trip nagyobb számú a **nyitott** állapot, a hiba oka, hogy a szolgáltatás nem volt teljesen elérhető száma képest.

**Naplózás**. Egy áramköri megszakító kell naplófájl összes sikertelen kérések (és valószínűleg sikeres kérelmek) ahhoz, hogy a rendszergazda a művelet állapotának figyelése.

**Helyreállíthatósága**. Az áramköri megszakító az általa védett művelet valószínűleg helyreállítási szerkezet megfelelően konfigurálni kell. Például, ha az áramköri megszakító marad a **nyitott** állapot hosszú ideig felvethet kivételek akkor is, ha a hiba okát lett feloldva. Ehhez hasonlóan a áramköri megszakító ingadozik, és csökkenti az alkalmazások válaszidejét, ha vált a **nyissa meg** állapotát a **félig nyitott** túl gyorsan állapot.

**Tesztelése sikertelen műveletek**. Az a **nyissa meg** állapota, nem pedig egy időzítő segítségével határozza meg, váltson át a **félig nyitott** állapotba kerül, a áramköri megszakító helyette rendszeresen ping paranccsal elérhető a távoli szolgáltatás vagy az erőforrás annak meghatározásához, hogy rendelkezik ismét elérhetővé válik. A ping művelet, amely korábban nem sikerült meghívni kísérlet formájában is beletelhet, vagy azt egy különleges műveletet a távoli kifejezetten a teszt a szolgáltatás által biztosított szerint használhat a [állapotfigyelő végpont Figyelési mintát](health-endpoint-monitoring.md).

**Kézi felülbírálás**. A rendszer, ha a helyreállítási a sikertelen művelet ideje rendkívül változó hogy a rendszer kézi alaphelyzetbe állítása, amely lehetővé teszi a rendszergazdának, zárja be a áramköri megszakító (és alaphelyzetbe állítja a sikertelen kísérletek számlálója) beállítás számára előnyös. Hasonlóképpen, a rendszergazda kényszerítheti egy áramköri megszakító be a **nyissa meg** állapotban (Újraindítás a időtúllépés időzítő) Ha a művelet védi az áramköri megszakító: átmenetileg nem érhető el.

**Párhuzamossági**. Az azonos áramköri megszakító nagyszámú egyidejű alkalmazáspéldányra fért hozzá. Végrehajtása ne egyidejű-kérések blokkolása, vagy adja hozzá túlzott terhelés minden olyan művelet hívására.

**Erőforrás megkülönböztetési**. Ügyeljen arra, hogy ha egy egyetlen áramköri megszakító használ egy típusú erőforrás fennáll-e több alapul szolgáló független is. Például a tárolóban, amely több szilánkok tartalmazza, egy shard lehet teljes elérhető amíg egy másik ütközik egy ideiglenes probléma. A következő használati helyzetekben a hibaválaszok egyesítve lesznek, ha egy alkalmazás előfordulhat, hogy próbáljon meg hozzáférni bizonyos szilánkok akkor is, ha a hiba nagyon valószínű, amíg más szilánkok elérésére blokkolhatja, annak ellenére, hogy valószínűleg sikeres.

**Gyorsított áramkör legfrissebb**. Néha hibaválaszt út azonnal, és minimális időtartama tripped marad az áramköri megszakító elegendő információt tartalmazhat. Például egy megosztott erőforráson, amely túl van terhelve válaszát hiba oka lehet, hogy egy azonnali újrapróbálkozási nem ajánlott, és, hogy az alkalmazás kell helyette próbálkozzon újra néhány perc múlva.

> [!NOTE]
> A szolgáltatás adhat vissza HTTP 429 (túl sok kérelem) Ha az ügyfél szabályozás, vagy a HTTP 503-as (a szolgáltatás nem érhető el), ha a szolgáltatás már nem érhető el. A válasz tartalmazhatnak további adatokat, például az a késleltetés várható időtartama.

**Visszajátszását sikertelen kérelmek**. Az a **nyitott** állapotában ahelyett, hogy egyszerűen sikertelen gyorsan, egy áramköri megszakító sikerült is jegyezze fel az egyes kérelmek naplóba részleteit, és gondoskodik ezeket a kérelmeket a rendszer kell játssza, amikor a távoli erőforrás vagy a szolgáltatás elérhetővé válik.

**Külső szolgáltatások nem megfelelő időtúllépése**. Előfordulhat, hogy egy áramköri megszakító nem teljes védelme érdekében az alkalmazások külső szolgáltatások egy hosszabb időkorlát konfigurált teljesítő műveletek. Az időtúllépési érték túl hosszú, ha a szál fut egy áramköri megszakító blokkolhatja hosszú időn keresztül a áramköri megszakító azt jelzi, hogy sikerült-e a művelet előtt. Megadott idő példányok is megpróbálja meghívni az áramköri megszakító szolgáltatásba, és szálak előtt minden jelentős számú lefoglalják számos egyéb alkalmazás sikeres lesz.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a minta használata:

- Megakadályozhatja, hogy egy alkalmazás adott távoli szolgáltatás, vagy egy megosztott erőforrás elérésére, ha ez a művelet nagyon valószínű, sikertelen lesz.

Ez a minta nem ajánlott:

- Az alkalmazások, például a memória adatszerkezet helyi titkos erőforrásainak kezelése elérésére. Ebben a környezetben egy áramköri megszakító használatával volna hozzá protokollterhelés a rendszer.
- Az alkalmazások üzleti logikát a kivételek kezelése helyett.

## <a name="example"></a>Példa

A webalkalmazás a lapok számos feltöltött származó adatok egy külső szolgáltatás. A rendszer megvalósítja a minimális gyorsítótárazást, ha ezeken a lapokon a legtöbb találatok miatt a szolgáltatás oda-vissza. A szolgáltatás a webes alkalmazás közötti kapcsolatok sikerült kell konfigurálni a határidőn (jellemzően 60 másodperc), és ha a szolgáltatás ennyi időn belül nem válaszol az egyes weblapon logikai feltételezik, hogy a szolgáltatás nem érhető el, és kivételt jelez.

Azonban ha a szolgáltatás leáll és a rendszer túlzottan megnő, felhasználók sikerült kényszeríthető, várjon, amíg egy kivétel előtt 60 másodperc. Végül erőforrások, például a memória, a kapcsolatok és a szálak sikerült kell kimerül, akadályozza meg, hogy más felhasználók csatlakozzanak a rendszer akkor is, ha nem érik lapok, amelyek az adatok beolvasása a szolgáltatástól.

A rendszer további kiszolgálók hozzáadásával és végrehajtásával, terheléselosztás előfordulhat, hogy erőforrások válnak kimerül, de azt nem megoldani a problémát, mert a felhasználói kérelmek továbbra is lefagyottnak késleltetés és a webkiszolgálók végül továbbra is futtathat a skálázás erőforrások.

A logika, amely csatlakozik a szolgáltatáshoz, és lekéri az adatokat egy áramköri megszakító az alkalmazásburkoló segíthet a probléma megoldásához, és a szolgáltatás hibája több elegantly kezelését. Felhasználói kérelmek továbbra is sikertelen lesz, de és gyorsabban fog-e sikertelen, és az erőforrások nem tiltható le.

A `CircuitBreaker` osztály egy olyan objektum, amely megvalósítja az áramköri megszakító vonatkozó állapotadatokat megőrzi a `ICircuitBreakerStateStore` felülete az alábbi kódban látható.

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

A `State` tulajdonság jelzi az áramköri megszakító aktuális állapotát, és akkor **nyitott**, **HalfOpen**, vagy **lezárva** a általdefiniált`CircuitBreakerStateEnum`enumerálása. A `IsClosed` tulajdonság értéke true, ha az áramköri megszakító le van zárva, de hamis, ha megnyitott vagy félig nyitva kell. A `Trip` metódus a nyitott állapotba vált, az áramköri megszakító állapotát, és rögzíti a kivételt, ami miatt a módosítás állapotú, és a dátum és időpont, amikor a kivétel történt. A `LastException` és a `LastStateChangedDateUtc` tulajdonságok ezeket az információkat adnak vissza. A `Reset` metódus bezárja az áramköri megszakító és a `HalfOpen` módszert állítja be az áramköri megszakító fele megnyitásához.

A `InMemoryCircuitBreakerStateStore` példában osztály tartalmaz egy megvalósítása a `ICircuitBreakerStateStore` felületet. A `CircuitBreaker` osztály hoz létre a megszakító állapot tárolásához az osztály egy példányát.

A `ExecuteAction` metódust a `CircuitBreaker` osztály becsomagolja művelet, mint a megadott egy `Action` delegálni. Ha az áramköri megszakító le van zárva, `ExecuteAction` meghívja a `Action` delegálni. A művelet sikertelen lesz, ha meghívja-e egy kivételkezelőbe `TrackException`, amely megnyitásához áramköri megszakító állapotúra állítja. Az alábbi példakód mutatja be a folyamatot.

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

A következő példa bemutatja a (nincs megadva az előző példából) futtatott kód is, ha az áramköri megszakító nem lezárja. Először ellenőrzi, ha az áramköri megszakító van nyitva, a helyi által megadott időtartamnál hosszabb ideig `OpenToHalfOpenWaitTime` mező mellett a `CircuitBreaker` osztály. Ha ez a helyzet, a `ExecuteAction` metódus állítja be az áramköri megszakító váltakozó megnyitásához, majd megpróbálja végrehajtani a műveletet, amelyet a `Action` delegálni.

Ha a művelet sikeres, az áramköri megszakító lezárt állapotában lesz visszaállítva. A művelet sikertelen lesz, ha vissza a nyitott állapotban) és időpontot (a kivétel történt a frissül, hogy az áramköri megszakító további időtartamra újra a művelet végrehajtása előtt megvárja a azokat kioldják.

Ha az áramköri megszakító csak van nyitva egy rövid ideig kevesebb, mint `OpenToHalfOpenWaitTime` érték, a `ExecuteAction` metódus egyszerűen jelez a `CircuitBreakerOpenException` kivétel és a hibát, ami miatt az áramköri megszakító való áttéréssel megnyitott állapotát jelzi.

Emellett használ egy zároláshoz az áramköri megszakító megakadályozása egyidejű hívás a művelet végrehajtása, amíg félig nyitva. Egy párhuzamos próbáltak meghívni a művelet automatikusan elvégezhető, mintha az áramköri megszakító meg volt nyitva, és azt fogja sikertelen, és a kivételt, a későbbiekben olvashat.

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken)
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
          catch (Exception ex)
          {
            // If there's still an exception, trip the breaker again immediately.
            this.stateStore.Trip(ex);

            // Throw the exception so that the caller knows which exception occurred.
            throw;
          }
          finally
          {
            if (lockTaken)
            {
              Monitor.Exit(halfOpenSyncObject);
            }
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

Használatához a `CircuitBreaker` objektum védelméhez egy műveletet, az alkalmazás létrehoz egy új a `CircuitBreaker` osztályhoz, és meghívja a `ExecuteAction` metódus, a művelet lehet elvégezni, mivel a paraméter megadásával. Az alkalmazás tényleges kell készíteni a `CircuitBreakerOpenException` kivétel, ha a művelet sikertelen, mert az áramköri megszakító meg nyitva. A következő kód egy példát mutat be:

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő minták is hasznosak lehetnek ebben a mintában végrehajtása során:

- [Ismételje meg a minta][retry-pattern]. Ismerteti, hogyan alkalmazás is kezelni a várható ideiglenes hibák csatlakozik egy szolgáltatás vagy a hálózati erőforráshoz transzparens módon megpróbálásával korábban sikertelen műveletet.

- [Minta figyelési állapotfigyelő végpont](health-endpoint-monitoring.md). Előfordulhat, hogy egy áramköri megszakító tudja egy kérést küld a végpont által a szolgáltatás a szolgáltatás teszteléséhez. A szolgáltatás Megadja, hogy annak állapotát kell visszaadnia.


[retry-pattern]: ./retry.md
