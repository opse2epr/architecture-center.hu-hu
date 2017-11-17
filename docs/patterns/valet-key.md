---
title: Kulcs valet
description: "A token vagy a kulcs, amely egy meghatározott erőforrás vagy a szolgáltatás korlátozott közvetlen hozzáférést biztosít az ügyfelek használja."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- security
ms.openlocfilehash: 791132eabf926cc285567454c60f894efa286433
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="valet-key-pattern"></a>Valet kulcs minta

[!INCLUDE [header](../_includes/header.md)]

Egy, az ügyfelek egy adott erőforrás korlátozott közvetlen hozzáférést biztosít ahhoz, hogy az adatok átvitele az alkalmazás-kiszervezés jogkivonatot használja. Ez különösen fontos az alkalmazásokat, amelyek a felhőben üzemeltetett tárolórendszerek vagy várólisták, és kis méretűvé teheti költség és méretezhetőség és teljesítmény maximalizálása érdekében.

## <a name="context-and-problem"></a>A környezetben, és probléma

Ügyfél programok és a böngészők gyakran kell írási és olvasási fájlokat vagy adatokat adatfolyamokat, és onnan egy kiszolgálóalkalmazás-tároló. Általában az alkalmazás fogja kezelni az adatok mozgása &mdash; beolvasása, a tárolási és adatfolyamként történő továbbítását, az ügyfél számára, vagy a feltöltött adatfolyam olvasásakor az ügyfél és az adattár tárolásához. Azonban ez a megközelítés elnyeli értékes erőforrások, például a számítási, memória és a sávszélesség.

Adattároló képesek legyenek kezelni feltöltése és letöltéséhez az adatok közvetlenül, anélkül, hogy az alkalmazás hajtsa végre a feldolgozási ezek az adatok áthelyezése. De általában ehhez az ügyfél rendelkezik hozzáféréssel a tárolóhoz a hitelesítő adatokat. Ez akkor lehet hasznos módszer adatátvitel költségei és az alkalmazás horizontális, és a teljesítmény maximalizálásához a követelmény minimalizálása érdekében. Azt jelenti, azonban, hogy az alkalmazás már nem tudja kezelni az adatok biztonságát. Miután az ügyfél közvetlenül elérheti az adattárolóhoz kapcsolattal rendelkezik, az alkalmazás nem jogcímszabálykészlet(ek) a. Már nem az irányítást a folyamat, és sem akadályozható meg a későbbi feltöltések vagy letölti az adattárolóból.

Ez nem fog reális megközelítés elosztott rendszerek, amelyek nem megbízható ügyfelek kiszolgálásához szükséges. Ehelyett alkalmazások biztonságosan részletes módon az adatok hozzáférésének vezérléséhez, de továbbra is csökkenthető a kiszolgáló terhelése beállítása az ezt a kapcsolatot, és majd így az ügyfél közvetlenül kommunikálnak az adattároló hajtsa végre a szükséges olvasására vagy írására, képesnek kell lennie műveletek.

## <a name="solution"></a>Megoldás

Szabályozására számára egy adattárból, ahol nem tudja kezelni a tároló hitelesítési és engedélyezési az ügyfelek a probléma megoldásához szükséges. Egy tipikus megoldás, hogy korlátozza a hozzáférést az adattár nyilvános kapcsolat, és adja meg az ügyfél egy kulcs, vagy a jogkivonatot, amely az adattár azt is ellenőrzi.

A kulcs vagy a token általában nevezzük valet kulcs. Egy adott erőforráshoz időben korlátozott hozzáférést biztosít, és lehetővé teszi, hogy csak előre definiált műveletek, például a olvasása és írása tárolóra vagy várólisták, vagy feltöltését és egy webböngészőben letöltésekor. Alkalmazások is létrehozása és kiállítása valet kulcsok böngészők és ügyféleszközök gyorsan és egyszerűen, így az ügyfelek anélkül, hogy közvetlenül a az adatátvitel kezeléséhez szükséges műveleteket. Ezzel eltávolítja a feldolgozási terhelést, és a hatása a teljesítményre és méretezhetőségre, az alkalmazás és a kiszolgáló.

Az ügyfél a a token csak egy adott időszakban, és a hozzáférési engedélyeket adott korlátozások egy adott erőforrás szerepel az adattárban elérésére használt, az ábrán látható módon. A megadott időszak után a kulcs érvénytelenné válik, és nem engedélyezik az erőforrás elérésére.

![1. ábra – a minta áttekintése](./_images/valet-key-pattern.png)

Akkor is, amelyen más függőségek, például az adatok köre kulcs konfigurálása. Például attól függően, hogy az áruház képességek, a kulcs megadhat egy teljes tábla adattárat, vagy csak adott sorokra a táblában. A felhőalapú tárolás rendszerek egy tárolót, vagy csak olyan tárolóban egy cikket a kulcs adja meg.

A kulcs is az alkalmazás érvénytelenítve lenne. Ez az a hasznos megközelítést, abban az esetben, ha az ügyfél értesíti a kiszolgálót, hogy helyesek-e az adatátviteli művelet. A kiszolgáló majd érvénytelenné tehetik kulcs további megelőzése érdekében.

Ezt a mintát használja egyszerűbbé teheti az erőforrásokhoz való hozzáférés kezelése mert létrehozására és a felhasználó hitelesítéséhez, engedélyeket, és távolítsa el a felhasználó nem kötelező. Emellett a hellyel, az engedély és az érvényességi időszak szűkítse&mdash;mind szerint egyszerűen a futási időben egyik kulcs létrehozásakor. A fontos tényezők, melyeknek az érvényességi időszak, és különösen az erőforrás helye olyan szorosan korlátozása, hogy a címzett csak azt az adott célra.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

**Kezelje az érvényességi állapot és a kulcs időszak**. Ha kiszivárogniuk vagy sérült, a kulcs hatékonyan feloldja a cél elemet, és lehetővé teszi a rosszindulatú visszaélések az érvényességi időszak alatt. A kulcs általában visszavonva, vagy le van tiltva, attól függően, hogy hogyan állította ki. Kiszolgálóoldali házirendek módosítható, vagy a kiszolgáló kulccsal van aláírva érvénytelenítve lenne. Adja meg egy rövid érvényességi, ha engedélyezi a nem engedélyezett műveletek elleni az adattár csökkentése érdekében. Azonban ha az érvényességi időtartama túl rövid, az ügyfél nem feltétlenül tudja elvégezni a műveletet, a kulcs érvényességének lejárta előtt. A kulcs megújításához, mielőtt az érvényességi idejük lejárata, ha a védett erőforrásokhoz való több hozzáférések szükség a hitelesített felhasználóknak engedélyezik.

**Vezérelhető a nyújt a kulcs hozzáférési szint**. A kulcs általában a felhasználó csak műveleteket kell végrehajtani a műveletet, például ha az ügyfél nem lesz képes feltölteni az adatokat az adattár csak olvasási hozzáféréssel kell lehetővé. A fájlfeltöltés közös adható meg a kulcs, amely csak írási engedéllyel, valamint a hely és az érvényességi időtartam esetén. Nagyon fontos pontosan határozza meg az erőforrás vagy a kulcs hatálya alá tartozó erőforrások készletét.

**Vegye figyelembe a felhasználó viselkedésének vezérlése**. Ez a kialakítás megvalósítása azt jelenti, hogy az erőforrások felhasználók szabályozhatják adatvesztés hozzáférési engedéllyel. A képességek a házirendeket és az engedélyek érhető el a szolgáltatás vagy a cél-tárolót, amely képes lehet kifejtett felügyeleti korlátozza. Például nincs általában lehetőség, amely korlátozza az adatok tárolására, vagy a kulcs használható próbál hozzáférni egy fájlhoz hányszor írandó méretének kulcs létrehozásához. Ennek eredményeként hatalmas váratlan adatátvitel költségeit, akkor, amikor az adott ügyfél által használt, és a kód a hiba oka lehet, ami ismételt feltöltés és letöltés választható. Egy fájl is feltölthetők, ahol csak lehetséges száma korlátozza, force az ügyfél számára, hogy az alkalmazás értesítése, ha egy művelet befejeződött. Néhány adattárolókhoz például az alkalmazás kódjának segítségével műveletek figyelése, és szabályozhatja a felhasználó működését események ablakába. Azt azonban nem szigorú kvóták az egyes felhasználók olyan több-bérlős forgatókönyvekben, ahol a ugyanazzal a kulccsal egy bérlőhöz tartozó felhasználók kényszerítéséhez.

**Ellenőrzi, és opcionálisan sanitize, minden feltöltött adatok**. Egy rosszindulatú felhasználó, amely hozzáfér a kulcs sikerült feltölteni az adatokat úgy tervezték, hogy okoz a rendszerben. Másik lehetőségként jogosult felhasználók előfordulhat, hogy feltölteni az adatokat, amely nem érvényes, és feldolgozásakor hiba vagy a rendszer hibát eredményezhet. Ez elleni védelem érdekében, győződjön meg arról, hogy minden feltöltött adatok érvényesítve, és be van jelölve, a használat előtt rosszindulatú tartalmat.

**Összes művelet naplózási**. Sok kulcs alapú mechanizmusok műveletek, például a feltöltések, a letöltéseket és a hibák jelentkezhetnek. Ezek a naplók általában lehet egy naplózási folyamat építve, és számlázási, ha a felhasználó a fájl mérete vagy az adatok kötet díjfizetéssel is használt. A naplók segítségével észleli a hitelesítési hibák száma, amelyek a kulcsszolgáltató, vagy egy tárolt házirend véletlen eltávolítását problémákat okozhatja.

**A kulcs biztonságos biztosításához**. A felhasználó egy weblapon aktiválja az URL-ágyazni, vagy használható server átirányítási műveletben, hogy a letöltés automatikusan megtörténik. Mindig HTTPS használatára képes biztosítani a kulcsot egy biztonságos csatornán keresztül.

**Bizalmas adatok védelmére átvitel**. Az alkalmazás keresztül érzékeny adatokat általában akkor kerül sor SSL vagy TLS használatával, és ez kell kikényszeríteni, az ügyfelek a-adattároló elérése közvetlenül.

Egyéb problémák az ebben a mintában végrehajtásakor a következők:

- Ha az ügyfél nem, vagy nem, értesítse a kiszolgáló, a művelet befejezése és a csak határérték a lejárati időt, a kulcs, az alkalmazás nem fog tudni elvégezni naplózási műveletek, például a leltározási feltöltések vagy letöltések számát, vagy megakadályozza az több feltöltések vagy letöltések.

- Kulcs házirendjei létrehozható rugalmasan korlátozott lehet. Például bizonyos mechanizmusok csak egy időzített lejárati idővel használatának engedélyezése. Mások számára nem adhat meg a megfelelő lépésköz legyen olvasási/írási engedéllyel.

- Ha a kulcs vagy a token érvényességi időtartam kezdési időpontja meg van adva, győződjön meg arról, hogy az kissé korábbi, hogy az ügyfél órák nem szinkronizált némileg elképzelhető, hogy engedélyezze az aktuális kiszolgáló időnél. Az alapértelmezett értéket, ha nincs megadva, az általában aktuális server idő.

- Az URL-címet a kulcsot tartalmazó kiszolgáló naplófájljainak lesznek rögzítve. Amíg a kulcs általában lejártak, előtt a naplófájlok használt elemzés, győződjön meg arról, hogy korlátozza a hozzáférést. Naplóadatok továbbított adatok köre a figyelési rendszerben vagy egy másik helyen tárolja, ha vegye fontolóra a késés, amíg kulcsok kiszivárgásának elkerülésére, az érvényességi időszak lejárta után.

- Ha az Ügyfélkód webböngészőben fut, a böngésző támogatja az eltérő eredetű erőforrások megosztása (CORS) engedélyezéséhez a kódot, amely végrehajtja az szolgáltatott a lap egy másik tartományban található adatok eléréséhez a böngészőben módosítania. Egyes régebbi böngészők és néhány adattároló nem támogatják a CORS, és előfordulhat, hogy a kód a böngészők futó tudja valet kulcsot használ egy másik tartományban, például a felhőalapú társzolgáltatás fiókja adatainak eléréséhez.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában akkor hasznos, ha az alábbi helyzetekben:

- Minimálisra csökkentése erőforrás betöltése, és maximalizálhatja a teljesítményét és méretezhetőségét. Valet kulccsal zárolva lesz az erőforrás nem igényel, nincs távoli kiszolgáló hívás szükséges, a kibocsáthatja valet kulcsok száma nincs korlátozva, és ezzel elkerülheti a hibaérzékeny pontok kialakulását, az alkalmazáson keresztül az adatátvitel végrehajtása eredő a kódot. Valet kulcs létrehozása az általában egy egyszerű kriptográfiai művelet karakterlánc aláírási kulccsal.

- Működési költségek csökkentése érdekében. Tárolók és a várólisták közvetlen hozzáférést tesz lehetővé a erőforrás és hatékony költség, a is kevesebb hálózati eredményez kerekíteni való adatváltások számát, és lehetővé teheti a szükséges számítási erőforrások számának csökkentése.

- Az ügyfelek mikor rendszeresen feltölteni, vagy adatletöltéshez, különösen akkor, ha van egy nagy méretű, vagy ha az egyes műveletek magában foglalja a nagy fájlok.

- Ha az alkalmazás csak korlátozott számítási erőforrásai, vagy üzemeltető korlátozásai miatt, vagy szempontok költsége. Ebben a forgatókönyvben a minta továbbiakat is lehet hasznos, ha sok egyidejű adatok feltöltését, vagy tölti le, mert azt mentesíti az alkalmazást az adatátvitel kezelése.

- Amikor az adatok tárolása egy távoli tárolót vagy egy ugyanabban az adatközpontban. Ha az alkalmazás jogcímszabálykészlet(ek) a kellett, valószínűleg az adatátvitel az Adatközpont között, vagy a saját vagy nyilvános hálózaton keresztül az ügyfél és az alkalmazás között, majd az alkalmazás között további sávszélességet díjat és az adatok tárolásához.

Ez a minta nem lehet a következő esetekben lehet hasznos:

- Ha az alkalmazás kell végezni néhány feladatot az adatok tárolása előtt vagy azt megelőzően azt az ügyfél küldte. Például ha az alkalmazás-ellenőrzéshez van szüksége, hozzáférés sikeres bejelentkezés, vagy hajtsa végre a átalakítás az adatokon. Azonban bizonyos adatokat tárolja és az ügyfelek képesek és például tömörítése és egyszerű átalakítások végez (például egy webes böngésző általában kezelni tud a GZip-formátumok).

- Ha egy meglévő alkalmazást kialakításának megnehezíti a minta tartalmaznia. Ebben a mintában általában használatához egy másik architekturális megközelítés továbbítása és az adatok fogadása.

- Ha elkérése karbantartása vagy hány alkalommal kell egy adatátviteli művelet végrehajtása, és használja a valet kulcs mechanizmus nem támogatja az értesítéseket, amelyek a kiszolgáló segítségével kezelheti ezeket a műveleteket.

- Ha szükséges, különösen a feltöltési művelet során az adatok méretének korlátozására. Ez az egyetlen megoldásként van, ellenőrizze az adatok méretét, a művelet befejezése után, vagy ellenőrizze a megadott időszak után, vagy egy ütemezés szerint feltöltések méretét az alkalmazásra vonatkozóan.

## <a name="example"></a>Példa

Az Azure által támogatott közös hozzáférésű jogosultságkód Azure Storage a blobot, táblát és üzenetsort adatainak részletes hozzáférés-vezérléshez, valamint a Service Bus-üzenetsorok és témakörök. Egy megosztott hozzáférési aláírást jogkivonat beállítható úgy, hogy egy adott táblához; például az olvasási, írási, frissítési és törlési speciális hozzáférési jogot biztosít a táblázatban lévő kulcs tartomány a várólista; egy blob; vagy a blob-tároló. Az érvényességi időszak vagy időkorlátozás nélkül lehet egy meghatározott ideig.

Azure közös hozzáférésű jogosultságkód is támogatja a kiszolgálón tárolt hozzáférési házirendeket, amelyek egy adott erőforrás, például egy táblázat vagy a blob társítható. Ez a szolgáltatás vezérelhetőséget és rugalmasságot alkalmazás által létrehozott megosztott hozzáférési aláírást jogkivonatok képest, és kell használni, amikor csak lehetséges. Egy kiszolgálón tárolja a házirendben meghatározott beállítások módosíthatók, és anélkül, hogy egy új jogkivonatot kell végezni a token megjelennek, de a jogkivonat megadott beállítások nem módosítható egy új jogkivonatot kiadása nélkül. Ezt a módszert is lehetővé teszi egy érvényes megosztott hozzáférési aláírást jogkivonat visszavonása lejárta előtt.

> További információ: [bevezetéséről tábla SAS (közös hozzáférésű Jogosultságkód), üzenetsor SAS és a Blob SAS-frissítés](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/) és [megosztott hozzáférési aláírásokkal használatával](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/) az MSDN Webhelyén.

A következő kód bemutatja, hogyan hozzon létre egy megosztott hozzáférési aláírást tokent, amely érvényes 5 percig. A `GetSharedAccessReferenceForUpload` módszer segítségével feltölteni a fájlt az Azure Blob Storage közös hozzáférésű aláírások tokenre adja vissza.

```csharp
public class ValuesController : ApiController
{
  private readonly CloudStorageAccount account;
  private readonly string blobContainer;
  ...
  /// <summary>
  /// Return a limited access key that allows the caller to upload a file
  /// to this specific destination for a defined period of time.
  /// </summary>
  private StorageEntitySas GetSharedAccessReferenceForUpload(string blobName)
  {
    var blobClient = this.account.CreateCloudBlobClient();
    var container = blobClient.GetContainerReference(this.blobContainer);

    var blob = container.GetBlockBlobReference(blobName);

    var policy = new SharedAccessBlobPolicy
    {
      Permissions = SharedAccessBlobPermissions.Write,

      // Specify a start time five minutes earlier to allow for client clock skew.
      SharedAccessStartTime = DateTime.UtcNow.AddMinutes(-5),

      // Specify a validity period of five minutes starting from now.
      SharedAccessExpiryTime = DateTime.UtcNow.AddMinutes(5)
    };

    // Create the signature.
    var sas = blob.GetSharedAccessSignature(policy);

    return new StorageEntitySas
    {
      BlobUri = blob.Uri,
      Credentials = sas,
      Name = blobName
    };
  }

  public struct StorageEntitySas
  {
    public string Credentials;
    public Uri BlobUri;
    public string Name;
  }
}
```

> A teljes minta érhető el a letölthető ValetKey megoldás [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key). Ebben a megoldásban a ValetKey.Web projekt tartalmazza a webes alkalmazás, amely tartalmazza a `ValuesController` osztály fent látható. Ügyfél mintaalkalmazás, amely a webes alkalmazás használ az egy megosztott hozzáférési aláírásokkal kulcs lekérése, valamint a fájl feltöltése a blob storage a ValetKey.Client projekt érhető el.

## <a name="next-steps"></a>Következő lépések

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:
- Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key).
- [Forgalomirányító mintát](gatekeeper.md). Ebben a mintában a Valet kulcs mintát együtt használható alkalmazások és szolgáltatások védelmét, hogy az ügyfelek és az alkalmazás vagy szolgáltatás közötti közvetítőként dedikált gazdagép példánnyal. A forgalomirányító ellenőrzi kérelmek sanitizes, és továbbítja a kérelmek és az adatok az ügyfél és az alkalmazás között. Adjon meg egy további biztonsági réteget és csökkenteni a támadási felületet, a rendszer.
- [Statikus tartalom üzemeltető mintát](static-content-hosting.md). Egy felhőalapú tárolási szolgáltatás által biztosított ezeket az erőforrásokat közvetlenül az ügyfél, olcsóbbá számítási példányokért kapcsolatos követelmények csökkentése érdekében a statikus erőforrások telepítésének módját ismerteti. Ha az erőforrások nem kell nyilvánosan elérhető, a Valet kulcs minta segítségével biztonságos őket.
- [Tábla SAS (közös hozzáférésű Jogosultságkód), üzenetsor SAS és a Blob SAS-frissítés bemutatása](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/)
- [Közös hozzáférésű Jogosultságkód használatával](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/)
- [A Service busszal aláírás hitelesítés megosztott](https://azure.microsoft.com/documentation/articles/service-bus-shared-access-signature-authentication/)
