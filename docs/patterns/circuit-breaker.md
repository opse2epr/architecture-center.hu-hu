---
title: Circuit Breaker
description: Ha távoli szolgáltatáshoz vagy erőforráshoz csatlakozik, kezelheti azokat a hibákat, amelyek javítása esetleg sok időt venne igénybe.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 5a9c8254bf62488b46517ee3582c2323e206df8a
ms.sourcegitcommit: e9d9e214529edd0dc78df5bda29615b8fafd0e56
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/28/2018
ms.locfileid: "37090951"
---
# <a name="circuit-breaker-pattern"></a>Áramkör-megszakító minta

Ha távoli szolgáltatáshoz vagy erőforráshoz csatlakozik, kezelheti azokat a hibákat, amelyek helyreállítása esetleg sok időt venne igénybe. Ez javítja az alkalmazások stabilitását és rugalmasságát.

## <a name="context-and-problem"></a>Kontextus és probléma

Elosztott környezetben előfordulhat, hogy a távoli erőforrások és szolgáltatások meghívása átmeneti hibák miatt meghiúsul. Ilyen probléma lehet a lassú hálózati kapcsolat vagy időtúllépés, illetve ha az erőforrások nem fedezik az igényeket, vagy átmenetileg nem érhetők el. Ezek a hibák általában rövid idő alatt kijavítják magukat, és egy stabil felhőalapú alkalmazásnak fel kell készülnie az ilyesmi kezelésére, például egy [újrapróbálkozási mintához][retry-pattern] hasonló stratégiát alkalmazva.

Lehetnek azonban olyan helyzetek, amikor a hibákat nem várt események okozzák, amelyek helyrehozása tovább tarthat. Az ilyen hibák súlyossága a részleges kapcsolódási problémától a szolgáltatás teljes leállásáig terjedhet. Az ilyen helyzetekben lehet, hogy nincs értelme az alkalmazásnak folyamatosan újrapróbálkozni a művelettel, mert nem valószínű, hogy sikerrel járna. Ehelyett inkább az a szerencsés, ha az alkalmazás gyorsan tudomásul veszi, hogy a művelet nem sikerült, és ennek megfelelően kezeli a hibát.

Emellett, ha egy szolgáltatás nagyon elfoglalt, a rendszer egy részén előforduló hiba egész hibasorozatot indíthat be. Egy szolgáltatást meghívó műveletet például lehet úgy konfigurálni, hogy megvalósítson egy időtúllépést, és hibaüzenettel válaszoljon, ha a szolgáltatás nem válaszol a megadott időkereten belül. Ez a stratégia azonban eredményezheti azt, hogy az ugyanehhez a művelethez küldött sok párhuzamos kérés is blokkolva lesz az időkorlát lejáratáig. Ezek a blokkolt kérelmek olyan kritikus rendszererőforrásokat akadályozhatnak, mint a memória, szálak, adatbázis-kapcsolatok stb. Ebből következik, hogy ezek az erőforrások kimerülhetnek, ez pedig hibát okozhat a rendszer esetleg nem is kapcsolódó, de azonos erőforrásokat használó részeiben. Ezekben a helyzetekben az a jó, ha a művelet azonnal meghiúsul, és csak akkor érdemes újból meghívni a szolgáltatást, ha nagy esély van arra, hogy sikerül. Vegye figyelembe, hogy rövidebb időkorlát beállításával megoldható ez a probléma, de az időkorlát ne legyen olyan rövid, hogy a művelet szinte mindig meghiúsuljon, még akkor is, ha végül a szolgáltatáskérés sikeres.

## <a name="solution"></a>Megoldás

A Michael Nygard [Release it!](https://pragprog.com/book/mnee/release-it) című könyvében népszerűsített áramkör-megszakítási minta megakadályozhatja, hogy egy alkalmazás ismételten megpróbáljon végrehajtani egy nagy valószínűséggel meghiúsuló műveletet. Így lehetővé teszi a folytatást, nem kell a hiba kijavítására várni, vagy CPU-ciklusokat pazarolni arra, hogy kiderüljön, hosszan tartó hibáról van szó. Az áramkör-megszakítási minta azt is lehetővé teszi az alkalmazás számára, hogy észlelje, ha a hibát kijavították. Ha úgy tűnik, hogy a hiba megoldódott, az alkalmazás megpróbálhatja meghívni a műveletet.

> Az áramkör-megszakító minta nem azonos az újrapróbálkozási mintával. Az újrapróbálkozási minta lehetővé teszi az alkalmazás számára, hogy újból megpróbálkozzon egy művelettel, ha arra számít, hogy sikeres lesz. Az áramkör-megszakítási minta megakadályozza, hogy egy alkalmazás olyan műveletet próbáljon többször végrehajtani, amely nagy eséllyel lesz sikertelen. Az alkalmazások kombinálhatják ezt a két mintát úgy, hogy az újrapróbálkozási mintát használják egy művelet áramkör-megszakítón keresztüli meghívására. Az azonban fontos, hogy az újrapróbálkozási minta érzékeny legyen az áramkör-megszakító által visszaadott kivételekre, és abbahagyja az újrapróbálkozási kísérleteket, ha az áramkör-megszakító azt jelzi, hogy a hiba nem átmeneti.

Az áramkör-megszakító proxyként viselkedik az esetlegesen meghiúsuló műveleteknél. A proxynak monitoroznia kell a legutóbbi jelentkezett hibák számát, ez az információ segít eldönteni, hogy engedje-e a művelet továbbhaladását, vagy egyszerűen azonnal küldjön egy kivételt.

A proxy egy, az alábbi, egy elektromos áramkör-megszakító működését utánzó állapotokkal bíró állapotgépként valósítható meg:

- **Zárt**: Az alkalmazásból érkező kérelmet a rendszer a művelethez irányítja. A proxy számlálóban követi a közelmúltbeli hibák számát, és ha a művelet meghívása nem sikeres, a proxy növeli a számláló értékét. Ha a közelmúltbeli hibák száma meghalad egy megadott küszöbértéket egy adott időtartamon belül, a proxy **Nyitott** állapotba kerül. Ezen a ponton a proxy elindít egy időtúllépési időzítőt, és annak lejártakor a proxy **Félig nyitott** állapotba kerül.

    > Az időtúllépési időzítő célja az, hogy időt adjon a rendszernek a hibát okozó probléma megoldására, mielőtt engedélyezné az alkalmazás számára, hogy újra megpróbálja elvégezni a műveletet.

- **Nyitott**: Az alkalmazásból érkező kérelem azonnal meghiúsul, az alkalmazás kivételt kap vissza.

- **Félig nyitott**: Az alkalmazásból érkező kérelmek korlátozott számban átjuthatnak, és meghívhatják a műveletet. Ha ezek a kérelmek sikeresek, feltételezhető, hogy a korábban meghibásodást okozó hiba megszűnt, és az áramkör-megszakító átvált **Zárt** állapotra (a hibaszámláló alaphelyzetbe áll). Ha a kérelem meghiúsul, az áramkör-megszakító feltételezi, hogy a hiba továbbra is fennáll, ezért visszaáll **Nyitott** állapotba, és újraindítja az időtúllépési időzítőt, hogy további időt biztosítson a rendszernek a meghibásodásból való helyreállásra.

    > A **Félig nyitott** állapot azért hasznos, mert megakadályozza, hogy a helyreálló szolgáltatást hirtelen elárasszák a kérelmek. Előfordul, hogy miközben zajlik a helyreállítás, a szolgáltatás már képes korlátozott számban kiszolgálni kéréseket a teljes helyreállásig, de amíg folyamatban van a helyreállítás, a nagy mennyiségű feladattól időtúllépés fordulhat elő, vagy esetleg újra meghibásodik a szolgáltatás.

![Az áramkör-megszakító állapotai](./_images/circuit-breaker-diagram.png)

Az ábrán a **Zárt** állapot által használt hibaszámláló időalapú. Rendszeres időközönként automatikusan alaphelyzetbe áll. Ez segít megakadályozni, hogy az áramkör-megszakító belépjen a **Nyitott** állapotba, ha csak alkalmanként észlel hibát. Az áramkör-megszakítót **Nyitott** állapotba léptető küszöbérték csak akkor érhető el, hogy a megadott számú hiba megadott időtartam alatt következik be. A **Félig nyitott** állapot által használt számláló a művelet meghívására tett sikeres kísérletek számát rögzít. Az áramkör-megszakító akkor tér vissza a **Zárt** állapotba, ha adott számú egymást követő műveletmeghívás sikeres volt. Ha bármely meghívás meghiúsul, az áramkör-megszakító azonnal belép a **Nyitott** állapotba, és a sikeres meghívások számlálója alaphelyzetbe áll, amikor a megszakító újra **Félig nyitott** állapotba lép.

> A rendszer helyreállásának kezelése kívülről történik, esetleg egy meghibásodott összetevő visszaállításával vagy újraindításával, vagy a hálózati kapcsolat megjavításával.

Az áramkör-megszakító minta stabilitást biztosít arra az időszakra, amíg a rendszer helyreáll egy meghibásodás után, és minimálisra csökkenti a hiba teljesítményre gyakorolt hatását. A segítségével fenntartható a rendszer válaszideje, mert gyorsan visszautasítja a várhatóan sikertelen műveletekre irányuló kéréseket, nem várja meg a művelet időtúllépését, vagy azt, hogy egyáltalán ne küldjön választ. Ha az áramkör-megszakító minden állapotváltáskor létrehoz egy eseményt, ez az információ segít monitorozni a rendszer áramkör-megszakító által védett részének állapotát, vagy riasztást küldeni egy rendszergazdának, ha az áramkör-megszakító **Nyitott** állapotba kerül.

A minta testreszabható, és a lehetséges hiba típusának megfelelően alkalmazható. Alkalmazhat például növekvő időtúllépési időzítőt egy áramkör-megszakítón. Az áramkör-megszakítót először néhány másodpercre helyezheti **Nyitott** állapotba, majd, ha a hiba nem oldódott meg, növelheti az időkorlátot néhány percre, és így tovább. Bizonyos esetekben hasznos lehet egy, az alkalmazás számára jelentéssel bíró alapértelmezett értéket visszaadni az alkalmazásnak ahelyett, hogy a **Nyitott** állapot hibát adna vissza, és kivételt hozna létre.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

**Kivételkezelés**. A műveletet áramkör-megszakítón keresztüli meghívó alkalmazást fel kell készíteni azon kivételek kezelésére, amelyek akkor jönnek létre, amikor a művelet nem érhető el. A kivételek kezelése alkalmazásspecifikus lesz. Az alkalmazás például átmenetileg csökkentheti a működési teljesítményét, meghívhat egy másik műveletet, hogy az megpróbálja végrehajtani ugyanazt a feladatot, vagy beszerezni ugyanazokat az adatokat, illetve jelentheti a kivételt a felhasználónak, és megkérheti, hogy próbálkozzon később újra.

**Kivételek típusai**. Egy kérés sok okból meghiúsulhat, ezek némelyike komolyabb meghibásodást jelez, mint mások. Egy kérelem meghiúsulhat például azért, mert egy távoli szolgáltatás összeomlott, és a helyreállása eltart pár percig, vagy meghiúsulhat időtúllépés miatt, ha a szolgáltatás átmenetileg túl lett terhelve. Egy áramkör-megszakító képes lehet a felmerülő kivételek típusának vizsgálatára, és képes az adott kivételek természetéhez igazítani a stratégiáját. Előfordulhat például, hogy nagyobb számú időtúllépési kivételre lesz szüksége az áramkör-megszakító **Nyitott** állapotba váltásához ahhoz képest, mint amikor a szolgáltatás teljes leállása miatt jelentkeznek a hibák.

**Naplózás**. Az áramkör-megszakítónak minden meghiúsult kérelmet naplóznia kell (és esetleg a sikereseket is), hogy a rendszergazda monitorozni tudja a művelet állapotát.

**Helyreállíthatóság**. Az áramkör-megszakítót úgy konfigurálja, hogy kövesse a védett művelet várható helyreállítási mintáját. Ha például az áramkör-megszakító hosszan **Nyitott** állapotban marad, akkor is létrehozhat kivételeket, ha a hiba oka már megszűnt. Ehhez hasonlóan az áramkör-megszakító ingadozhat és csökkentheti az alkalmazások válaszidejeit, ha túl gyorsan vált a **Nyitott** állapotból **Félig nyitott** állapotba.

**Sikertelen műveletek tesztelése**. A **Nyitott** állapotban lévő áramkör-megszakító ahelyett, hogy időzítővel határozná meg, mikor váltson **Félig nyitott** állapotba, rendszeres időközönként pingelheti a távoli szolgáltatást vagy erőforrást annak megállapításához, hogy az mikor válik újra elérhetővé. Ez a pingelés történhet egy korában meghiúsult művelet meghívásának formájában, vagy lehet a távoli szolgáltatás által kifejezetten a szolgáltatás állapotának tesztelésére biztosított speciális műveletet használni, mint például az [állapot végponti monitorozását végző minta](health-endpoint-monitoring.md).

**Kézi felülbírálás**. Olyan rendszerben, ahol a meghiúsuló művelet helyreállítási ideje kifejezetten változó, érdemes manuális visszaállítási lehetőséget biztosítani, hogy a rendszergazda zárhassa az áramkör-megszakítót (és alaphelyzetbe állíthassa a hibaszámlálót). Ehhez hasonlóan a rendszergazda **Nyitott** állapotba kényszerítheti az áramkör-megszakítót (és alaphelyzetbe állíthatja az időtúllépési időzítőt), ha az áramkör-megszakító által védett művelet átmenetileg nem érhető el.

**Egyidejűség**. Ugyanazt az áramkör-megszakítót számos párhuzamos alkalmazáspéldány is elérheti. Ez a megvalósítás valószínűleg nem blokkolja a párhuzamos kéréseket, és nem ad túlzott többletterhelést az egyes műveletmeghívásokhoz.

**Erőforrás-megkülönböztetés**. Legyen óvatos, ha egy áramkör-megszakítót használ egy erőforrástípushoz, amennyiben több alapul szolgáló független szolgáltató is előfordulhat. Egy több szilánkot tartalmazó adattárban előfordulhat, hogy az egyik szilánk teljesen hozzáférhető, de egy másiknál átmenetileg probléma jelentkezik. Ha ezekben a forgatókönyvekben egyesülnek a hibaválaszok, előfordulhat, hogy egy alkalmazás akkor is megpróbál hozzáférni egy szilánkhoz, amikor nagy valószínűséggel hiba jelentkezek, miközben más szilánkokat blokkol, noha valószínűleg sikeres lenne a művelet.

**Gyorsított áramkör-megszakítás**. Néha egy hibaválasz elég információt tartalmaz ahhoz, hogy az áramkör-megszakító azonnal váltson, és átváltva maradjon a lehető legrövidebb ideig. Egy túlterhelt megosztott erőforrástól érkező hibaüzenet például jelezheti azt, hogy az azonnali újrapróbálkozás nem ajánlott, és érdemes inkább pár perccel később újrapróbálkoznia az alkalmazásnak.

> [!NOTE]
> A szolgáltatás adhat vissza HTTP 429-es hibát (Túl sok kérés), ha szabályozza az ügyfelet, vagy HTTP 503-as hibát (A szolgáltatás nem érhető el), ha a szolgáltatás jelenleg nem érhető el. A válasz tartalmazhat további információkat is, például a késleltetés várható időtartamát.

**Sikertelen kérelmek visszajátszása**. Egy **Nyitott** állapotú áramkör-megszakító az egyszerű és gyors meghiúsulás helyett rögzítheti az egyes kérések adatait egy naplóba, és beállítja a kérések megismétlését, ha a távoli erőforrás vagy szolgáltatás elérhetővé válik.

**Külső szolgáltatások nem megfelelő időtúllépései**. Előfordulhat, hogy egy áramkör-megszakító nem képes teljesen megvédeni az alkalmazásokat a hosszú időkorláttal konfigurált külső szolgáltatásokon meghiúsuló műveletektől. Ha az időkorlát túl hosszú, egy áramkör-megszakítót futtató szál blokkolása hosszú ideig is tarthat, mielőtt az áramkör-megszakító a művelet meghiúsulását jelezné. Lehet, hogy ezen idő alatt sok más alkalmazáspéldány is megpróbálja meghívni a szolgáltatást az áramkör-megszakítón keresztül, és jelentős számú szálat lefoglalhatnak, mielőtt meghiúsulnak.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja a következő mintát a következő helyzetekben:

- Ha meg kell akadályozni, hogy egy alkalmazás meghívjon egy távoli szolgáltatást vagy hozzáférjen egy megosztott erőforráshoz, mert a művelet nagy valószínűséggel meghiúsul.

Ez a minta nem ajánlott a következő helyzetekben:

- Ha helyi személyes erőforrások hozzáférését kell kezelnie egy alkalmazásban, például egy memórián belüli adatstruktúrát. Ebben a környezetben az áramkör-megszakító csak többletterhelést jelentene a rendszer számára.
- Ha helyettesíteni szeretné a kivételkezelést az alkalmazásai üzleti logikájában.

## <a name="example"></a>Példa

Egy webalkalmazásban számos oldal külső szolgáltatásból származó adatokkal van feltöltve. Ha a rendszer minimális gyorsítótárazást valósít meg, ezen oldalak legtöbb megkeresése a szolgáltatással való adatváltást eredményez. A webalkalmazás és a szolgáltatás közötti kapcsolatokat lehet egy időkorláttal (általában 60 mp) konfigurálni, és ha a szolgáltatás nem válaszol ennyi idő alatt, az egyes weboldalak logikája feltételezi, hogy a szolgáltatás nem érhető el, ezért kivételt jelez.

Ha azonban a szolgáltatás meghiúsul, és a rendszer nagyon elfoglalt, a felhasználóknak akár 60 másodpercig is várakozniuk kell a kivétel létrejötte előtt. Végül a memória, a kapcsolatok, a szálak és a hasonló erőforrások kimerülhetnek, és megakadályozhatják, hogy más felhasználók csatlakozzanak a rendszerhez, még ha azok nem is a szolgáltatástól adatokat lekérő oldalakhoz akarnának hozzáférni.

A rendszer méretezése további webkiszolgálók hozzáadásával és terhelés-kiegyenlítés megvalósításával késleltetheti az erőforrások kimerülését, de nem fogja megoldani a problémát, mert a felhasználói kérelmek továbbra sem fognak válaszolni, és a végül az összes webkiszolgáló kifogyhat az erőforrásokból.

Ha a szolgáltatáshoz kapcsolódó és az adatokat lekérő adatokat egy áramkör-megszakítóba burkoljuk, az segít megoldani ezt a problémát, és a szolgáltatás-meghibásodás kezelése elegánsabb lesz. A felhasználói kérések továbbra is meghiúsulnak, de gyorsabban, és az erőforrások nem lesznek blokkolva.

A `CircuitBreaker` osztály az áramkör-megszakítók állapotinformációit egy olyan objektumban tartalmazza, amely megvalósítja az alábbi kódban látható `ICircuitBreakerStateStore` interfészt.

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

A `State` tulajdonság jelzi az áramkör-megszakító aktuális állapotát, és **Open** (Nyitott), **HalfOpen** (Félig nyitott) vagy **Closed** (Zárt) értéket vehet fel a `CircuitBreakerStateEnum` enumerálásban megadottak szerint. Az `IsClosed` tulajdonság értéke akkor legyen true (igaz), ha az áramkör-megszakító le van zárva, és akkor false (hamis), ha a megszakító nyitva vagy félig nyitva van. A `Trip` metódus az áramköri megszakított nyitott állapotba váltja, valamint rögzíti az állapotban változást okozó kivételt a kivétel létrejöttének dátumával és időpontjával együtt. A `LastException` és a `LastStateChangedDateUtc` tulajdonságok adják vissza ezeket az információkat. A `Reset` metódus zárja az áramkör-megszakítót, a `HalfOpen` metódus pedig félig nyitott állapotba helyezi az áramkör-megszakítót.

A példában szereplő `InMemoryCircuitBreakerStateStore` osztály tartalmazza az `ICircuitBreakerStateStore` interfész megvalósítását. A `CircuitBreaker` osztály létrehozza ennek az osztálynak egy példányát, hogy megőrizze az áramkör-megszakító állapotát.

A `CircuitBreaker` osztály `ExecuteAction` metódusa egy, az `Action` delegáltjaként megadott műveletet burkol be. Ha az áramkör-megszakító zárt, az `ExecuteAction` meghívja az `Action` delegáltat. Ha a művelet meghiúsul, egy kivételkezelő meghívja a `TrackException` metódust, amely az áramkör-megszakító állapotát nyitottra állítja. A folyamatot az itt következő kód mutatja be.

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

A következő példa bemutatja azt az előző példából kihagyott kódrészt, amely akkor lesz végrehajtva, ha az áramkör-megszakító nincs zárva. Először ellenőrzi, hogy az áramkör-megszakító hosszabb ideje van-e nyitva, mint ami a helyi `OpenToHalfOpenWaitTime` mezőben van megadva a `CircuitBreaker` osztályban. Ha ez a helyzet, az `ExecuteAction` metódus félig nyitott állapotba állítja az áramkör-megszakítót, majd megpróbálja végrehajtani az `Action` delegált által meghatározott műveletet.

Ha a művelet sikeres, az áramkör-megszakító visszavált zárt állapotba. Ha a művelet meghiúsul, visszavált nyitott állapotba, és frissül a kivétel létrejöttének időpontja, így az áramkör-megszakító kivár egy újabb időtartamot, mielőtt újra megpróbálja végrehajtani a műveletet.

Ha az áramkör-megszakító csak egy rövid ideje van nyitva, az `OpenToHalfOpenWaitTime` értéknél nem régebben, az `ExecuteAction` metódus egyszerűen `CircuitBreakerOpenException` kivételt jelez, és visszaadja azt a hibát, amely miatt az áramkör-megszakító nyitott állapotba került.

Ezen felül egy zár segítségével akadályozza meg, hogy az áramkör-megszakító félig nyitott állapotban megpróbáljon párhuzamos hívásokat indítani a művelethez. A művelet meghívására indított egyidejű kísérleteket a rendszer úgy kezeli, mintha az áramkör-megszakító nyitott állapotban lenne, és a kísérlet meghiúsulna a később ismertetett kivétellel.

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
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
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
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

Ha `CircuitBreaker` objektumot használ egy művelet védelmére, egy alkalmazás létrehoz egy példányt a `CircuitBreaker` osztályból, és meghívja az `ExecuteAction` metódust, amelynek paramétere a végrehajtani kívánt művelet. Az alkalmazást elő kell készíteni a `CircuitBreakerOpenException` kivétel rögzítésére arra az esetre, ha a művelet az áramkör-megszakító nyitott állapota miatt meghiúsulna. Az alábbi kód példa erre:

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

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

A következő minták is hasznosak lehetnek ennek a mintának a végrehajtása során:

- [Újrapróbálkozási minta][retry-pattern]. Ismerteti egy alkalmazás számára a szolgáltatásokhoz vagy hálózati erőforrásokhoz való csatlakozáskor jelentkező előre jelzett, átmeneti meghibásodások kezelését egy korábban meghiúsult művelet transzparens módon való ismételt megkísérlésével.

- [Állapot végponti monitorozását végző minta](health-endpoint-monitoring.md). Egy áramkör-megszakító képes lehet egy szolgáltatás állapotának tesztelésére oly módon, hogy kérést küld egy, a szolgáltatás által közzétett végpontnak. A szolgáltatásnak az állapotát jelző adatokat kell visszaadnia.


[retry-pattern]: ./retry.md
