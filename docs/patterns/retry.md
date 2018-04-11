---
title: Retry
description: Engedélyezheti egy alkalmazás számára a szolgáltatásokhoz vagy hálózati erőforrásokhoz való csatlakozáskor jelentkező előre jelzett, átmeneti meghibásodások kezelését egy korábban meghiúsult művelet transzparens módon való ismételt megkísérlésével.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- resiliency
ms.openlocfilehash: 73fdcbcc2bd75593a4c8e33dc2259c90593e14db
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="retry-pattern"></a>Ismételje meg a minta

[!INCLUDE [header](../_includes/header.md)]

Átmeneti hibák kezeléséhez, ha csatlakozik egy szolgáltatás vagy a hálózati erőforráshoz, a sikertelen művelettel transzparens módon megpróbálásával alkalmazás engedélyezése. Ez javítja az alkalmazás stabilitását.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egy alkalmazás, amely kommunikál a felhőben futó elemek nem lehet az átmeneti hibák ebben a környezetben felmerülő-és nagybetűket. Hibák a következők: hálózati összetevőkkel és szolgáltatásokkal kapcsolatban a pillanatnyi elvesztését, egy szolgáltatás, vagy előforduló fordulhat elő, amikor egy szolgáltatás foglalt ideiglenes elérhetetlensége.

Ezek a hibák általában önállóan korrigálja, és ha a hibát kiváltó műveletet meg kell ismételni megfelelő késleltetéssel is valószínűleg sikeres lesz. Például egy adatbázis-szolgáltatás, amely sok egyidejű kérés feldolgozása folyamatban van a sávszélesség-szabályozási stratégia, amely ideiglenesen kódszegmensek semmilyen további mindaddig, amíg a munkaterhelés rendelkezik megkönnyítését is létrehozható. Egy alkalmazás, az adatbázis elérésére tett kísérlet sikertelen lehet a kapcsolat, de ismét várakozás után próbálja azt sikeres lehet.

## <a name="solution"></a>Megoldás

A felhőben átmeneti nem ritka, és egy alkalmazást úgy kell megtervezni, elegantly és transzparens módon kezelje őket. Ez minimalizálja a hibák lehet az üzleti feladatok működik-e az alkalmazás által okozott hatások.

Ha egy alkalmazás hibát észlel, amikor megpróbálja kérelmet küld a távoli szolgáltatás, a hiba a következő stratégiák használata is képes kezelni:

- **Szakítsa meg**. Ha a hiba azt jelzi, hogy a probléma nem átmeneti, vagy nem valószínű, hogy sikeres, ha ismétlődő, az alkalmazás kell a művelet megszakításához és kivétel jelentéséhez. Például érvénytelen hitelesítő adatok megadása által okozott hitelesítési hiba oka nem kísérlet sikeres függetlenül attól, hogy hány alkalommal azt.

- **Próbálja meg újra**. Ha a konkrét hibát jelentett szokatlan vagy ritka, akkor előfordulhat, hogy által okozott megszokottól például egy hálózati csomag sérülésének, amíg a továbbítás. Ebben az esetben az alkalmazás sikerült próbálja meg újra a sikertelen kérelem újra azonnal mert megszokott hiba valószínűleg nem ismételhető meg, és a kérés valószínűleg sikeres lesz.

- **Újrapróbálkozás késleltetése.** Ha egy több alkotómunkájának kapcsolat vagy foglalt hibák okozza a hibát, a hálózat vagy a szolgáltatás szükség lehet rövid időn belül során a problémák kijavítására sor vagy a hátralékos munkák nincs bejelölve. Az alkalmazás várakoznia kell egy megfelelő idő, mielőtt megpróbálná megismételni a kérelmet.

A gyakori átmeneti hibák újrapróbálkozások között az alkalmazás több példánya kérelmeinek lehetőség szerint egyenletes terjesztésére kell kiválasztani. Ez csökkenti a túlterhelt folytatása foglalt szolgáltatás esélyét. Ha egy alkalmazás hány példánya van folyamatosan overwhelming újrapróbálkozási által érintett szolgáltatás, ez vesz igénybe a szolgáltatás hosszabb helyreállításához.

Ha a kérés továbbra is sikertelen, az alkalmazás várja meg, és megismétléséhez. Ha szükséges, ez a folyamat megismételhető végezzenek újrapróbálkozások, amíg néhány kérelmek maximális számát kísérletek közötti üzenetváltás miatti késésekre. A késleltetés növelhető Növekményesen vagy exponenciálisan növekszik, attól függően, hogy milyen típusú hibával és a valószínűsége annak, hogy azt fogja javítani ebben az időszakban.

A következő ábra szemlélteti ezt a mintát használja egy üzemeltetett szolgáltatás művelet meghívása. Ha egy előre meghatározott számú kísérlet után a kérelem sikertelen, az alkalmazás kell kezelje a tartalék kivétel és ennek megfelelően kezelnie.

![1. ábra – olyan üzemeltett szolgáltatásban, az újrapróbálkozási minta használatával a művelet meghívása](./_images/retry-pattern.png)

Az alkalmazás összes próbálja meg elérni a távoli szolgáltatás, amely megfelelő a fent felsorolt stratégiák egyikét újrapróbálkozási házirendje kódban sortörés. Különböző szolgáltatások küldött kérelmeket a különböző házirend lehet. Egyes szállítók adja meg a szalagtár szerepel, amely alkalmazhatja újrapróbálkozási házirendeket, ahol az alkalmazás adhat meg a maximális számú újrapróbálkozást, az ismételt kísérletek számát, és más paramétereket közötti idő.

Az alkalmazás adatainak hibák, de sikertelenül műveletek kell naplózása. Ez az információ operátorok. Egy szolgáltatás, a rendszer gyakran nem érhető el vagy foglalt esetén gyakran, mert a szolgáltatás kimerítette erőforrásait. A gyakorisága, ezek csökkentheti, ha a szolgáltatás kiterjesztése. Például ha egy adatbázis-szolgáltatás folyamatosan túl van terhelve, annak lehet hasznos az adatbázis partícióazonosító és a terhelés elosztva több kiszolgáló között.

> [Microsoft Entity Framework](https://docs.microsoft.com/ef/) újra próbálkozik a Helyadatbázis-műveletekhez szolgáltatásokat tartalmazza. Is az Azure-szolgáltatások és az ügyfél SDK-k tartalmaznak egy újrapróbálkozási mechanizmus. További információkért lásd: [ismételje meg az adott szolgáltatások útmutatást](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific).

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor vegye figyelembe a következő szempontokat.

Az újrapróbálkozási házirendet kell kell beállítani, hogy megfelel-e az alkalmazás az üzleti követelmények és a hibaüzeneteket a hiba természetének. Egyes nem kritikus műveletek esetében érdemes gyors sikertelen helyett több alkalommal újra, és hatással lehet az alkalmazás átviteli. Például egy interaktív webes alkalmazásban fér hozzá a távoli szolgáltatás, érdemes kisebb számú újrapróbálkozások újrapróbálkozások között csak rövid késleltetés után nem sikerül, és a megfelelő üzenetet jelenít meg, a felhasználónak (például "próbálkozzon újra később"). Kötegelt alkalmazáshoz akkor célszerű több exponenciálisan növekszik késleltetéssel a kísérletek között újrapróbálkozások számának növeléséhez.

Egy kísérletet, és számos, az újrapróbálkozásokat minimális késleltetés agresszív újrapróbálkozási házirendje további ronthatja a közeli vagy kapacitással futó foglalt szolgáltatást. Az újrapróbálkozási házirendje is befolyásolhatják a figyelt alkalmazás, ha folyamatosan próbál a sikertelen műveletet.

Egy kérelem jelentős számú ismételt próbálkozás után továbbra is sikertelen, akkor jobb, ha az alkalmazás további erőforrást fog kérelmek megakadályozása, és egyszerűen a hibáról jelentés azonnal. Az időszak lejár, ha az alkalmazás feltételesen lehetővé tehetik keresztül egy vagy több kérést a tudni, hogy azok sikeres. Ezt a stratégiát további részletekért lásd: a [áramköri megszakító mintát](circuit-breaker.md).

Vegye figyelembe, hogy-e a művelet az idempotent. Ha igen, az eleve biztonságos, majd ismételje meg. Ellenkező esetben az újrapróbálkozások támadó egynél többször nem kívánt mellékhatással működő hajthatnak végre a műveletet. Például egy szolgáltatás előfordulhat, hogy a kérés fogadásához, sikeresen feldolgozni a kérelmet, de nem választ küld. Ezen a ponton az újrapróbálkozási logika előfordulhat, hogy újra elküldeni a kérést, feltéve, hogy az első kérésre nem érkezett.

A szolgáltatásnak küldött kérelemben számos okból a hiba természetétől függően különböző kivételt váltson ki lehet sikertelen. Néhány kivétel, amely feloldható gyorsan, míg mások azt jelzi, hogy a hiba már tovább tartó hibájára utalhat. Akkor érdemes használni az újrapróbálkozási házirendet úgy, hogy a kivétel típusa alapján újrapróbálkozási kísérletek között eltelt idő.

Vegye figyelembe, hogy egy művelet, amely része egy tranzakció milyen hatással lesz a teljes tranzakció konzisztencia. Konfigurálva finomhangolhatják a tranzakciós műveletek maximalizálhatja a siker esélye, visszavonja a tranzakció lépéseket kell csökkentse az újrapróbálkozási házirendet.

Győződjön meg arról, hogy az összes újrapróbálkozási kód teljes körűen tesztelve meghibásodás számos ellen. Ellenőrizze, hogy nem súlyosan hatással lehet a teljesítmény- vagy az alkalmazás megbízhatóságát, a szolgáltatások és erőforrások túlterhelés miatt, vagy hozzon létre versenyhelyzetek vagy szűk keresztmetszetek.

Alkalmazzon újrapróbálkozási logika csak ha a sikertelen művelet teljes környezetében értendő. Például egy feladatot, amely tartalmazza az újrapróbálkozási házirendje hív meg egy másik feladat újrapróbálkozási házirendje is tartalmazó, a további réteget újrapróbálkozások adhat hozzá nagy késleltetéseket feldolgozását. Jobb, ha a gyors sikertelen, és a hiba okát jelentést a feladat aktiváló alacsonyabb szintű feladat lehet. A magasabb szintű tevékenység majd kezelik a saját házirend alapján hiba.

Fontos, hogy az azonosítható legyen az alapul szolgáló problémák az alkalmazáshoz, a szolgáltatások vagy az erőforrások ismételt próbálkozással okozó összes kapcsolathibái bejelentkezni.

Vizsgálja meg a hibákat, amelyek egy szolgáltatás vagy egy erőforrást derítsen fel, ha fontosságúak valószínű, hogy mennyi ideig tartós vagy a Terminálszolgáltatások fordulhat elő. Ha igen, célszerű a tartalék kivételként kezeli. Az alkalmazás jelentést vagy jelentkezzen a kivétel, és próbálja meghívása alternatív szolgáltatás (Ha ilyen), vagy funkciókat kínáló csökkentett teljesítményű folytatja. Észlelése és a hosszú ideig tart hibák kezeléséhez további információkért tekintse meg a [áramköri megszakító mintát](circuit-breaker.md).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ezt a mintát használja, ha egy alkalmazás sikerült átmeneti tapasztal, egy távoli szolgáltatással együttműködő, vagy egy távoli erőforráshoz fér hozzá. Ezek a hibák kellene lennie a rövid élt, majd ismételje meg a kérelmeket, amelyek korábban nem sikerült a következő kísérlet volt sikeres.

Ez a minta nem lehet hasznos:

- Amikor egy hiba valószínű, hogy hosszú ideig tartó, mert ez befolyásolhatja az alkalmazások válaszkészségét. Az alkalmazás előfordulhat, hogy jelentős, időt és erőforrásokat próbál ismételje meg a kérelmeket, amelyek valószínűleg sikertelen lesz.
- Kezelése, amelyek nem átmeneti, például a belső kivételek az üzleti logika egy alkalmazás szereplő hibák által okozott hibák miatt sikertelen.
- Méretezhetőségi problémájára címzési rendszerekben helyett. Ha egy alkalmazás gyakori foglalt hibákat észlel, akkor gyakran a jele, hogy a szolgáltatás vagy az éppen elért erőforrás kell kiterjesztett.

## <a name="example"></a>Példa

Ebben a példában a C# az újrapróbálkozási minta megvalósítását mutatja be. A `OperationWithBasicRetryAsync` metódust, az alábbi meghívja az aszinkron módon történik a külső szolgáltatást a `TransientOperationAsync` metódust. Részletes adatait a `TransientOperationAsync` metódus a szolgáltatásra vonatkozó lesz, és a mintakódot hiányoznak.

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

Egy try vagy catch blokkon csomagolni tartalmazza az utasítást, amely hívja meg ezt a módszert a hurok. Az a hurok kilépjen, ha hívása a `TransientOperationAsync` metódus végrehajtása sikeres, nem jelez kivételt. Ha a `TransientOperationAsync` metódus sikertelen, a catch blokk megvizsgálja a hiba okát. Ha az rendelkezik feltételezhetően olyan átmeneti hibát a kód megvárja, rövid késleltetés mielőtt megpróbálná megismételni a műveletet.

A hurok is nyomon követi a száma, hogy a művelet végrehajtására történt kísérlet, és ha a kód három alkalommal nem sikerül a kivétel adottnak további hosszú tartós. Ha a kivétel nem átmeneti, vagy hosszú tartós, a catch kezelő kivételt jelez. Ez a kivétel lép ki a hurok és kell lennie a kóddal, amely hívja meg a `OperationWithBasicRetryAsync` metódust.

A `IsTransient` metódust, az alábbi ellenőrzi az egy adott készletét, amely kapcsolódik a környezet a kódot a kivételeket futtatása. Egy átmeneti kivétel meghatározása az éppen elért erőforrás attól függően változnak, és a környezet a művelet végrehajtása folyamatban van a.

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

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

- [Áramköri megszakító mintát](circuit-breaker.md). A újrapróbálkozási minta nem hasznos, ha átmeneti kezelése. Ha hiba várhatóan több hosszú tartós, több megfelelő áramköri megszakító minta végrehajtásához lehet. Az újrapróbálkozási mintát is használható egy áramköri megszakító együtt arra, hogy egy átfogó megközelítés hibák kezelnek.
- [Ismételje meg az adott szolgáltatások útmutató](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific)
- [Kapcsolat rugalmassága](https://docs.microsoft.com/ef/core/miscellaneous/connection-resiliency)
