---
title: Újrapróbálkozási minta
titleSuffix: Cloud Design Patterns
description: Engedélyezheti egy alkalmazás számára a szolgáltatásokhoz vagy hálózati erőforrásokhoz való csatlakozáskor jelentkező előre jelzett, átmeneti meghibásodások kezelését egy korábban meghiúsult művelet transzparens módon való ismételt megkísérlésével.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: edd73fd68ca4708faed92c6b1bcf5cfa3e6b2f07
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298589"
---
# <a name="retry-pattern"></a>Újrapróbálkozási minta

[!INCLUDE [header](../_includes/header.md)]

Engedélyezheti egy alkalmazás számára a szolgáltatásokhoz vagy hálózati erőforrásokhoz való csatlakozáskor jelentkező átmeneti meghibásodások kezelését egy meghiúsult művelet transzparens módon való ismételt megkísérlésével. Ez javíthatja az alkalmazás stabilitását.

## <a name="context-and-problem"></a>Kontextus és probléma

A felhőben futó elemekkel kommunikáló alkalmazásoknak érzékenynek kell lenniük az ebben a környezetben előforduló átmeneti hibákra. Ilyen hiba lehet az összetevők és szolgáltatások hálózati kapcsolatának pillanatnyi megszakadása, a szolgáltatások átmeneti elérhetetlensége, valamint a foglalt szolgáltatás miatti időtúllépés.

Ezek a hibák gyakran maguktól megoldódnak, és ha megfelelő idő múlva megismételik a hibát kiváltó műveletet, az valószínűleg sikeresen végbemegy. Egy számos egyidejű kérést feldolgozó adatbázis-szolgáltatás például olyan szabályozási stratégiát valósíthat meg, amely ideiglenesen elutasítja a további kéréseket, amíg nem enyhül a terhelés. Az adatbázist elérni próbáló alkalmazás csatlakozása sikertelen lehet, de ha később próbálkozik, sikerülhet a csatlakozás.

## <a name="solution"></a>Megoldás

A felhőben nem ritkák az átmeneti hibák, és az alkalmazásokat úgy kell kialakítani, hogy elegánsan és átlátható módon kezelje azokat. Így minimálisra csökkenthető a hibák az alkalmazás által végzett üzleti feladatokra gyakorolt hatása.

Ha egy alkalmazás hibát észlel, amikor kéréseket próbál küldeni egy távoli szolgáltatásnak, a következő stratégiákkal kezelheti a hibát:

- **Megszakítás**. Ha a hiba azt jelzi, hogy a probléma nem átmeneti vagy nem valószínű, hogy a művelet a megismétlésekor sikeres lesz, az alkalmazásnak meg kell szakítania a műveletet, és kivételt kell jelentenie. Az érvénytelen hitelesítő adatok megadása miatti hitelesítési hiba például valószínűleg további kísérletek esetén sem fog megoldódni.

- **Újrapróbálkozás**. Ha a jelentett hiba szokatlan vagy ritka, akkor előfordulhat, hogy szokatlan körülmények okozták, például egy hálózati csomag megsérült a továbbítás során. Ebben az esetben az alkalmazás azonnal újrapróbálkozhat a sikertelen kéréssel, mert nem valószínű, hogy ugyanez a hiba meg fog ismétlődni, és a kérés valószínűleg sikeres lesz.

- **Újrapróbálkozás később**. Ha a hibát egy vagy több gyakori kapcsolati vagy foglaltsági hiba okozza, lehet, hogy a hálózatnak vagy a szolgáltatásnak rövid időre van szüksége a kapcsolati hibák kijavításához vagy a várólistán lévő munkák kiürítéséhez. Az alkalmazásnak várnia kell egy ideig a kérés újbóli megkísérlése előtt.

A gyakori átmeneti hibák esetén úgy kell kiválasztani az újrapróbálkozások között eltelt időtartamot, hogy az alkalmazás több példányáról érkező kérések a lehető legegyenletesebben legyenek elosztva. Ez csökkenti annak az esélyét, hogy egy foglalt szolgáltatás folyamatosan túlterhelt legyen. Ha egy alkalmazás számos példánya folyamatosan túlterhel egy szolgáltatást a kérések újrapróbálásával, több időt vesz igénybe a szolgáltatás helyreállítása.

Ha a kérés továbbra is sikertelen, az alkalmazás várhat és megismételheti a kísérletet. Szükség esetén ez a folyamat megismételhető az újrapróbálkozások közötti késleltetések növelésével, amíg a rendszer el nem éri a kísérletek maximális számát. A késleltetés növekményesen vagy exponenciálisan növelhető a hiba típusától és annak valószínűségétől függően, hogy ezen idő alatt megoldódik-e.

A következő ábra szemlélteti egy művelet ezzel a mintával való aktiválását egy üzemeltetett szolgáltatásban. Ha a kérés a kísérletek előre meghatározott száma után is sikertelen, az alkalmazásnak kivételként kell kezelnie a hibát.

![1. ábra – Művelet meghívása üzemeltetett szolgáltatásban az újrapróbálkozási mintával](./_images/retry-pattern.png)

Az alkalmazásnak olyan kódba kell csomagolnia a távoli szolgáltatások elérési kísérleteit, amely a fent felsorolt stratégiák egyikének megfelelő újrapróbálkozási szabályzatot valósít meg. A különböző szolgáltatásoknak küldött kérésekre különböző szabályzatok lehetnek érvényesek. Néhány gyártó újrapróbálkozási szabályzatokat alkalmazó kódtárakat biztosít, ahol az alkalmazás meghatározhatja az újrapróbálkozások maximális számát, az újrapróbálkozási kísérletek között eltelt időt és más paramétereket.

Az alkalmazásoknak naplózniuk kell a hibák és a sikertelen műveletek részleteit. Ez az információ hasznos az operátorok számára. Ha egy szolgáltatás gyakran elérhetetlen vagy foglalt, azt leggyakrabban a szolgáltatás erőforrásainak kimerülése okozza. A szolgáltatás horizontális felskálázásával csökkentheti ezeknek a hibáknak a gyakoriságát. Ha például egy adatbázis-szolgáltatás folyamatosan túlterhelt, érdemes lehet particionálni az adatbázist, és több kiszolgáló között elosztani a terhelést.

> A [Microsoft Entity Framework](/ef) számos eszközt nyújt az adatbázis-műveletek újrapróbálásához. Emellett a legtöbb Azure-szolgáltatás és ügyféloldali SDK is tartalmaz újrapróbálkozási mechanizmust. További információkért tekintse meg [az adott szolgáltatásokra vonatkozó újrapróbálkozási útmutatást](/azure/architecture/best-practices/retry-service-specific).

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe.

Az újrapróbálkozási szabályzatot úgy kell beállítani, hogy megfeleljen az alkalmazás az üzleti követelményeinek és a hiba természetének. Egyes nem kritikus műveletek esetében jobb a gyors sikertelenség, mint a többszöri újrapróbálkozás, amely befolyásolja az alkalmazás teljesítményét. Egy távoli szolgáltatást elérő interaktív webes alkalmazásban például jobb a kevesebb újrapróbálkozás utáni sikertelen eredmény, ahol csak kis késleltetés van az újrapróbálkozási kísérletek között, és egy megfelelő üzenet jelenik meg a felhasználók számára (például „próbálkozzon újra később”). Kötegelt alkalmazásokhoz érdemes lehet növelni az újrapróbálkozási kísérletek számát, ezzel exponenciálisan növelve a kísérletek közötti késleltetést.

A kísérletek között minimális késleltetést használó agresszív újrapróbálkozási szabályzat és a számos újrapróbálkozás tovább ronthat az elfoglalt szolgáltatás állapotán, amely maximális vagy közel maximális kapacitással fut. Az újrapróbálkozási szabályzat az alkalmazás válaszkészségét is befolyásolhatja, ha folyamatosan sikertelen műveletet próbál végezni.

Ha egy kérés jelentős számú újrapróbálkozás után is sikertelen, akkor jobb, ha az alkalmazás meggátolja, hogy további kérések jussanak el ugyanarra az erőforrásra, és egyszerűen azonnal hibát jelent. Amikor az időszak lejár, az alkalmazás feltételesen átengedhet egy vagy több kérést, hogy megállapítsa, azok sikeresek-e. A stratégiáról további részletekért lásd az [áramkör-megszakítási mintát](./circuit-breaker.md).

Gondolja át, hogy a művelet idempotens-e. Ha igen, alapvetően biztonságos az újrapróbálkozás. Más esetekben az újrapróbálkozások miatt előfordulhat, hogy a művelet többször lesz végrehajtva, nem kívánt mellékhatásokkal. Egy szolgáltatás például fogadhatja a kérést, sikeresen feldolgozhatja a kérést, de lehet, hogy nem tud választ küldeni. Ezen a ponton az újrapróbálkozási logika újraküldheti a kérést, feltételezve, hogy a szolgáltatás nem kapta meg az első kérést.

A szolgáltatásoknak küldött kérések számos okból hiúsulhatnak meg, és a hiba természetétől függően különböző kivételeket váltanak ki. Néhány kivétel gyorsan megoldható hibát jelez, míg mások azt jelzik, hogy a hiba hosszabb ideig tart. Hasznos lehet, ha az újrapróbálkozási szabályzat a kivétel típusa alapján állítja be az újrapróbálkozási kísérletek közötti időt.

Fontolja meg, hogy egy tranzakció részét képező művelet újrapróbálása milyen hatással lesz a tranzakció teljes konzisztenciájára. Finomhangolja az újrapróbálkozási szabályzatot a tranzakciós műveletekhez, hogy a műveletek minél nagyobb eséllyel sikeresek legyenek, és ne kelljen visszavonni a tranzakció összes lépését.

Győződjön meg arról, hogy az összes újrapróbálkozási kód teljes körűen tesztelve lett különböző hibafeltételekre. Ellenőrizze, hogy ne legyen jelentős hatással az alkalmazás teljesítményére vagy megbízhatóságára, ne okozzon túlzott terhelést a szolgáltatásokon és az erőforrásokon és ne hozzon létre versenyhelyzeteket vagy szűk keresztmetszeteket.

Csak ott alkalmazzon újrapróbálkozási logikát, ahol a meghiúsuló műveletek teljes kontextusa érthető. Ha például egy újrapróbálkozási szabályzatot tartalmazó feladat egy újrapróbálkozási szabályzatot tartalmazó másik feladatot hív meg, az újrapróbálkozások extra rétege miatt a feldolgozás során hosszú késések lesznek tapasztalhatóak. Érdemes úgy konfigurálni az alacsonyabb szintű feladatot, hogy gyorsan hiúsuljon meg és jelentse a hiba okát az azt elindító feladatnak. Ez a magasabb szintű feladat ezután a saját szabályzata alapján kezelheti a hibát.

Fontos az újrapróbálkozást okozó összes csatlakozási hibát naplózni, hogy azonosíthatók legyenek az alkalmazással, a szolgáltatásokkal vagy az erőforrásokkal kapcsolatos alapvető hibák.

Vizsgálja meg egy szolgáltatás vagy erőforrás legvalószínűbb hibáit annak felderítése érdekében, hogy valószínűleg tartósak vagy végzetesek-e. Ha igen, jobb kivételként kezelni a hibát. Az alkalmazás jelentheti vagy naplózhatja a kivételt, majd a folytatáshoz megpróbálhat alternatív szolgáltatást elindítani (ha van elérhető) vagy csökkentett teljesítményű funkciókat alkalmazni. A tartós hibák észlelésével és kezelésével kapcsolatos további információkért lásd az [áramkör-megszakítási mintát](./circuit-breaker.md).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ezt a mintát használja, ha egy alkalmazás átmeneti hibákat tapasztalhat, amikor távoli szolgáltatással kommunikál, vagy távoli erőforráshoz fér hozzá. Ezek a hibák várhatóan rövid életűek, és a korábban meghiúsult kérések megismétlése egy későbbi kísérlet során sikeres lehet.

Nem érdemes ezt a mintát használni a következő esetekben:

- Amikor egy hiba valószínűleg tartós, mert ez befolyásolhatja az alkalmazások válaszkészségét. Előfordulhat, hogy az alkalmazás időt és erőforrásokat pazarol egy olyan kérés megismétlésére, amely valószínűleg sikertelen lesz.
- A nem átmeneti hibák által okozott hibák kezeléséhez, például amikor egy alkalmazás üzleti logikájában lévő hibák belső kivételeket okoznak.
- A rendszer skálázhatósági hibáinak kezelési alternatívájaként. Ha egy alkalmazás gyakori foglaltsági hibákat észlel, az gyakran azt jelzi, hogy az elért szolgáltatást vagy erőforrást vertikálisan fel kell skálázni.

## <a name="example"></a>Példa

Ez a C# nyelven írt példa az újrapróbálkozási minta implementálását mutatja be. Az alább látható `OperationWithBasicRetryAsync` metódus aszinkron módon hív meg egy külső szolgáltatást a `TransientOperationAsync` metóduson keresztül. A `TransientOperationAsync` metódus részletei a szolgáltatásra jellemzőek, és kimaradnak a mintakódból.

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry,
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

Az ezt a metódust meghívó utasítás egy for ciklusba csomagolt try/catch blokkban szerepel. A for ciklus kilép, ha a `TransientOperationAsync` metódus hívása kivétel nélkül sikerül. Ha a `TransientOperationAsync` metódus sikertelen, a catch blokk megvizsgálja a hiba okát. Ha ez feltételezhetően átmeneti hiba, a kód vár egy rövid ideig a művelet újrapróbálása előtt.

A for ciklus a művelet megkísérléseinek számát is nyomon követi, és ha a kód három alkalommal sikertelen, tartósabbnak feltételezi a kivételt. Ha a kivétel nem átmeneti, vagy ha tartós, a catch kezelő kivételt jelez. Ez a kivétel kilép a for ciklusból, és az `OperationWithBasicRetryAsync` metódust meghívó kódnak el kell kapnia.

Az alább látható `IsTransient` metódus a kódot futtató környezetre jellemző adott kivételeket keres. Az átmeneti kivételek meghatározása az elért erőforrásoktól és a művelet elvégzéséhez használt környezettől függően változik.

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

- [Áramkör-megszakítási minta](./circuit-breaker.md). Az újrapróbálkozási minta átmeneti hibák kezeléséhez hasznos. Ha egy hiba várhatóan tartósabb, célszerűbb lehet az áramkör-megszakítási mintát implementálni. Az újrapróbálkozási minta áramkör-megszakítási mintával együtt is használható a hibák átfogóbb kezelése érdekében.
- [Újrapróbálkozási útmutatás adott szolgáltatásoknál](/azure/architecture/best-practices/retry-service-specific)
- [Kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency)
