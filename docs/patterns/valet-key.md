---
title: Pótkulcs minta
titleSuffix: Cloud Design Patterns
description: Jogkivonatot vagy kulcsot használhat, amely korlátozott közvetlen hozzáférést biztosít az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 09173717d499d524d4d5dad2c1202c1bf361b1e5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009865"
---
# <a name="valet-key-pattern"></a>Pótkulcs minta

[!INCLUDE [header](../_includes/header.md)]

Egy jogkivonat használatával korlátozott közvetlen hozzáférést biztosíthat az ügyfelek számára egy adott erőforráshoz, hogy így csökkentse egy alkalmazás adatátviteli követelményeit. Ez különösen hasznos olyan alkalmazások esetében, amelyek felhőben futtatott tárolórendszereket vagy üzenetsorokat használnak, mert minimalizálja a költségeket és maximalizálja a skálázhatóságot és a teljesítményt.

## <a name="context-and-problem"></a>Kontextus és probléma

Az ügyfélprogramoknak és a böngészőknek gyakran kell fájlokat vagy adatstreameket olvasniuk, illetve írniuk az alkalmazás tárolójában. Általában az alkalmazás fogja kezelni az adatok mozgását – vagy lekérdezi a tárolóból és streameli az ügyfélnek, vagy beolvassa az ügyfél által feltöltött streamet, és menti az adattárba. Azonban ez a megközelítés értékes erőforrásokat foglal le, például számítási, memória- és sávszélesség-erőforrásokat.

Az adattárak képesek közvetlenül kezelni az adatfeltöltést és -letöltést, így az alkalmazás részéről nincs szükség semmilyen feldolgozási tevékenységre az adatok mozgatásához. Ehhez viszont általában az ügyfélnek hozzá kell férnie a tároló biztonsági hitelesítő adataihoz. Ez hasznos módszer lehet, ha minimalizálni szeretné az adatátvitel költségeit és szükség van az alkalmazás horizontális felskálázására, valamint maximalizálni szeretné a teljesítményt. Azonban ez azt jelenti, hogy az alkalmazás már nem fogja tudni kezelni az adatok biztonságát. Miután az ügyfél közvetlen hozzáféréssel tud kapcsolódni az adattárhoz, az alkalmazás már nem töltheti be a forgalomirányító szerepét. Már nem az alkalmazás szabályozza a folyamatot, és nem akadályozhatja meg a későbbi feltöltéseket vagy letöltéseket az adattárolóban.

Ez nem reális megközelítés elosztott rendszerek esetében, amelyek nem megbízható ügyfeleket is kiszolgálnak. Ehelyett az alkalmazásoknak képesnek kell lenniük az adathozzáférést biztonságosan és részletesen szabályozni, ugyanakkor csökkenteniük kell a kiszolgáló terheltségét is, méghozzá úgy, hogy először létre kell hozniuk a kapcsolatot, majd engedélyezniük kell az ügyfél számára a közvetlen kommunikációt az adattárral, hogy az végrehajthassa a szükséges olvasási vagy írási műveleteket.

## <a name="solution"></a>Megoldás

Meg kell oldania az adattárhoz való hozzáférés szabályozását olyan esetben, amikor a tároló nem képes kezelni az ügyfelek hitelesítését és ellenőrzését. Egy tipikus megoldási lehetőség, hogy korlátozza a hozzáférést az adattár nyilvános kapcsolatához, és biztosít egy kulcsot vagy jogkivonatot az ügyfélnek, amelyet az adattár ellenőrizni tud.

Ezt a kulcsot vagy a jogkivonatot általában pótkulcsnak nevezzük. A kulcs időkorlátos hozzáférést biztosít bizonyos erőforrásokhoz, és csak előre meghatározott műveleteket tesz lehetővé, például írást és olvasást a tárolón vagy üzenetsorban, vagy fel- és letöltést böngészőn keresztül. Az alkalmazások gyorsan és könnyen létrehozhatnak és kioszthatnak pótkulcsokat ügyféleszközök és böngészők számára, amelyekkel az ügyfelek elvégezhetik a szükséges műveleteket anélkül, hogy az alkalmazásnak közvetlenül kezelni kellene az adatátvitelt. Ezzel megszűnik az alkalmazás és a kiszolgáló feldolgozási többletterhelése, a teljesítményre és skálázhatóságra gyakorolt hatásával együtt.

Az ügyfél a jogkivonattal az adattár egy kijelölt erőforrásához férhet hozzá, csak meghatározott ideig, és a hozzáférési engedélyek bizonyos mértékű korlátozásával, mint ahogy az ábrán látható. A megadott időszak után a kulcs érvénytelenné válik, és többé nem ad hozzáférést az erőforráshoz.

![1. ábra – A minta áttekintése](./_images/valet-key-pattern.png)

Ezen felül lehetősége van más függőségekkel (például az adatok hatóköre) rendelkező kulcs konfigurálására. Például az adattár képességeihez mérten a kulcs meghatározhat egy teljes táblát az adattáron belül, vagy akár csak egy tábla bizonyos sorait. Felhőalapú tárolórendszerek esetében a kulcs megadhat egy tárolót, vagy csak egy adott elemet egy tárolón belül.

A kulcsot az alkalmazás is képes érvényteleníteni. Ez hasznos megközelítés abban az esetben, ha az ügyfél értesíti a kiszolgálót az adatátvitel befejeztéről. A kiszolgáló majd érvénytelenné tehetik a kulcsot a további hozzáférés megakadályozása.

A minta használatával leegyszerűsítheti az erőforrásokhoz való hozzáférés kezelését, mert nem kell létrehozni és hitelesíteni a felhasználót, engedélyeket megadni, majd végül eltávolítani a felhasználót. Emellett könnyű korlátozni a helyet, az engedélyt és az érvényességi időszakot – ehhez csupán létre kell hoznia egy kulcsot futtatáskor. A legfontosabb tényező az érvényességi időszak, és különösen az erőforrás helyének minél szigorúbb korlátozása, hogy a kulcs birtokosa azt csak rendeltetésszerű célokra használhassa.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

**Kezelje a kulcs érvényességi állapotát és időtartamát**. Ha a kulcs kiszivárog vagy megsérül, az gyakorlatilag szabaddá teszi a célelemet, amely az érvényességi idő alatt kártékony használat áldozata lehet. A kiállítás módjától függően a kulcsot általában visszavonhatja vagy letilthatja. A kiszolgálóoldali szabályzatok módosíthatók, illetve az aláírást biztosító kiszolgálói kulcs érvényteleníthető. Adjon meg rövid érvényességi időt, hogy a minimálisra csökkentse az adattárra irányuló jogosulatlan műveletek kockázatát. Ha azonban az érvényességi időtartam túl rövid, előfordulhat, hogy az ügyfél nem tudja elvégezni a kívánt műveletet a kulcs érvényességének lejárta előtt. Engedélyezze a jogosult felhasználók számára a kulcs megújítását az érvényesség lejárta előtt arra az esetre, ha többszöri hozzáférésre van szükség a védett erőforráshoz.

**Szabályozza a kulcs által biztosított hozzáférési szintet**. A kulcs általában csak a művelethez szükséges tevékenységek végrehajtását engedélyezi a felhasználó számára, például csak olvasható hozzáférést ad, ha az ügyfélnek nem szabadna adatot feltöltenie az adattárba. Fájlfeltöltés esetén olyan kulcsot szokás megadni, amely csak írási engedélyt biztosít, valamint meghatározza a helyet és az érvényességi időt. Döntően fontos pontosan meghatározni azt az erőforrást vagy azt az erőforráskészletet, amelyre a kulcs érvényes.

**Gondolja át, hogyan fogja szabályozni a felhasználó viselkedését**. Az ilyen minta megvalósításával együtt jár az, hogy kevésbé tudja szabályozni azokat az erőforrásokat, amelyekhez hozzáférést biztosít a felhasználóknak. A szabályozás mértékét korlátozzák a szolgáltatás vagy a célzott adattár esetében elérhető szabályzatok és engedélyek képességei. Például általában nincs lehetőség egy olyan kulcs létrehozására, amely korlátozza a tárolóra írható adat méretét, vagy azt, hogy egy kulccsal hányszor férhetnek hozzá egy adott fájlhoz. Ez váratlanul hatalmas adatátviteli költséget eredményezhet, még akkor is, ha az az ügyfél használja a kulcsot, akinek szánták, és a kód olyan hibájából adódhat, amely ismétlődő fel- vagy letöltést okoz. A fájlfeltöltések számának korlátozásához ahol lehetséges, kényszerítse az ügyfelet, hogy értesítse az alkalmazást a feltöltés befejezéséről. Néhány adattár például eseményeket hoz létre, amelyeket az alkalmazás felhasználhat a műveletek monitorozására és a felhasználói viselkedés szabályozására. Azonban nehéz kvótákat előírni az egyéni felhasználók számára egy több bérlős helyzet esetén, ahol egyetlen bérlő összes felhasználója ugyanazt a kulcsot használja.

**Érvényesítsen és igény szerint vírusmentesítsen minden feltöltött adatot**. Ha egy kártevő felhasználó hozzáférést szerzett a kulcshoz, feltölthet olyan adatot, amelyet a rendszer károsítására terveztek. Más esetekben az engedélyezett felhasználók feltölthetnek érvénytelen adatokat, amelyek feldolgozáskor hibához vagy a rendszer meghibásodásához vezethetnek. Ez ellen úgy védekezhet, hogy gondoskodik az összes adat érvényesítéséről és rosszindulatú kódok ellenőrzéséről a használat előtt.

**Naplózzon minden műveletet**. Sok kulcsalapú mechanizmus képes olyan műveletek naplózására, mint a feltöltések, a letöltések és a hibák. Ezeket a naplókat általában beépítheti egy naplózási folyamatba, illetve számlázáshoz is használhatja, ha a felhasználónak fájlméret vagy adatmennyiség alapján számláz. A naplók segítségével észlelheti az olyan hitelesítési hibákat, amelyek esetlegesen a kulcsszolgáltató hibájából vagy egy tárolt hozzáférési szabályzat nem szándékos eltávolításából adódnak.

**A kulcsot biztonságosan adja át**. URL-be ágyazhatja, amelyet a felhasználó egy weblapon tud aktiválni, vagy használhatja egy kiszolgáló-átirányítási műveletben, így a letöltés automatikusan megindul. Mindig használjon HTTPS-t, hogy biztonságos csatornán keresztül adja át a kulcsot.

**Védje a bizalmas adatokat az átvitel során**. A bizalmas adatok szállítása az alkalmazáson keresztül általában SSL vagy TLS használatával történik, és ezt elő kell írni az olyan ügyfelek számára, akik közvetlen hozzáféréssel rendelkeznek az adattárhoz.

A minta megvalósításakor érdemes egyéb problémákról is tájékozódnia:

- Ha az ügyfél nem jelzi vagy nem tudja jelezni a kiszolgáló felé a művelet befejeződését, és csak a kulcs lejárati ideje korlátozza, az alkalmazás nem lesz képes naplózási műveletek végrehajtására, például a feltöltések vagy letöltések számának nyomon követésére, és nem képes megakadályozni a többszöri fel- vagy letöltést.

- A létrehozható kulcskezelési szabályzatok esetenként nem eléggé rugalmasak. Például bizonyos mechanizmusok csak időzített lejárati idő használatát engedik. Más mechanizmusok nem engedik az olvasási/írási engedélyek kellőképpen részletes meghatározását.

- Ha meg van adva a kulcs vagy a jogkivonat érvényességi időtartamának kezdeti időpontja, győződjön meg róla, hogy az a jelenlegi kiszolgálóidőnél egy kicsit korábbi időpont, hogy az ügyfelek akkor is kaphassanak hozzáférést, ha a rendszeróra nem teljesen szinkronizált. Az alapértelmezett érték a kiszolgáló aktuális rendszerideje, ha nincs egyéb megadva.

- A kulcsot tartalmazó URL-cím a kiszolgáló naplófájljaiban lesz rögzítve. Bár a kulcs általában már lejár, mire a naplófájlokat a rendszer felhasználná elemzések készítéséhez, gondoskodjon a hozzáférés korlátozásáról. Ha a naplózási adatokat továbbítja egy monitorozási rendszernek vagy más helyen tárolja őket, fontolja meg egy késleltetési idő bevezetését, amellyel megakadályozza a kulcsok kiszivárgását az érvényességi idejük lejárta előtt.

- Ha az ügyfélkód webböngészőben fut, előfordulhat, hogy a böngészőnek támogatnia kell az eltérő eredetű erőforrások megosztását (CORS), így a webböngészőben lefuthatnak olyan kódok, amelyekkel a weblapot kiszolgáló tartományon keresztül hozzáférhet egy másik tartomány adataihoz. Egyes régebbi böngészők és néhány adattároló nem támogatja a CORS-t, és az ezekben a böngészőkben futó kódok képesek lehetnek pótkulcs használatára, hogy így biztosítsák a hozzáférést egy másik tartomány adataihoz, például egy felhőalapú tárfiók esetében.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi helyzetekben lehet hasznos:

- Ha az erőforrások terhelésének minimálisra csökkentésére, illetve a lehető legnagyobb teljesítményre és skálázhatóságra van szükség. Pótkulcs használatakor nem kell zárolni az erőforrást, nem szükséges a távoli kiszolgáló meghívása, nincs korlátozva a kiadható pótkulcsok száma, és elkerülheti, hogy az alkalmazáskódon keresztüli adatátvitel miatt egyetlen meghibásodási pont alakuljon ki. A pótkulcs létrehozása általában egy egyszerű kriptográfiai művelet, amelynek során aláír egy sztringet egy kulccsal.

- Ha az üzemeltetési költségek csökkentésére van szükség. A közvetlen hozzáférés engedélyezése a tárolókhoz és az üzenetsorokhoz erőforrás- és költséghatékony, lecsökkentheti a hálózaton belüli oda-vissza áramló adatok mennyiségét, és lehetővé teheti a szükséges számítási erőforrások számának lecsökkentését.

- Ha az ügyfelek rendszeresen töltenek fel vagy le adatot, különösen nagy forgalom esetén, vagy ha minden művelet nagy méretű fájlok mozgatásával jár.

- Ha az alkalmazásnak csak korlátozott számítási erőforrásai vannak az üzemeltetési korlátozások vagy költségvetési szempontok miatt. Ebben a helyzetben a minta még hasznosabb lehet akkor, ha sok az egyidejű adatfeltöltés és -letöltés, mert mentesíti az alkalmazást az adatátvitel kezelése alól.

- Ha az adatok tárolását egy távoli tárolóban vagy egy másik adatközpontban oldja meg. Ha az alkalmazásnak be kellett töltenie a forgalomirányító szerepét is, további költséget okozhat az a plusz sávszélesség, amely az adatközpontok közötti adatátvitelből fakad, vagy pedig az ügyfél és az alkalmazás közötti átvitelből nyilvános vagy magánhálózatokon keresztül, majd ezután az alkalmazás és az adatközpont közötti adatátvitelből adódik.

Nem érdemes ezt a mintát használni a következő helyzetekben:

- Ha az alkalmazásnak el kell végeznie még néhány feladatot az adatokon, mielőtt azokat a rendszer eltárolja vagy elküldi őket az ügyfélnek. Ha például az alkalmazásnak érvényesítést kell végeznie, feltétel a naplófájl sikeres elérése vagy átalakítást kell végezni az adatokon. Azonban bizonyos adattárak és ügyfelek képesek egyszerű átalakítások egyeztetésére és végrehajtására, például tömörítésre vagy kibontásra (például egy webböngésző általában tudja kezelni a Gzip formátumokat).

- Ha a meglévő alkalmazáskialakítás megnehezíti a minta beépítését. A minta használatához általában egy másik architekturális megközelítés szükséges az adatok szállításához és fogadásához.

- Ha szükség van auditnaplóra és az adatátviteli műveletek számának szabályozására, és az alkalmazott pótkulcs-mechanizmus nem támogatja azokat az értesítéseket, amelyeket a kiszolgáló ezen műveletek kezeléséhez használhatna.

- Ha korlátozni kell az adatok méretét, különösen feltöltési műveleteknél. Az egyetlen megoldás az, ha az alkalmazás a művelet befejezte után ellenőrzi az adatméretet, vagy ellenőrzi a feltöltések nagyságát egy megadott idő eltelte után vagy egy megadott ütemezés szerint.

## <a name="example"></a>Példa

Az Azure támogatja a közös hozzáférésű jogosultságkódot az Azure Storage-en, hogy lehetővé tegye az adatok részletes hozzáférés-szabályozását a blobokban, a táblákban és az üzenetsorokban, valamint Service Bus-üzenetsorok és -témakörök esetében. A közös hozzáférésű jogosultságkód konfigurálható úgy, hogy meghatározott hozzáférést biztosítson, például olvasási, írási, frissítési és törlési engedélyt egy adott táblához, egy táblán belüli kulcstartományhoz, egy üzenetsorhoz, egy blobhoz vagy egy blobtárolóhoz. Az érvényesség lehet egy adott időszak, vagy akár lejárati idő nélküli is.

Az Azure közös hozzáférésű jogosultságkódja emellett támogatja a kiszolgálón tárolt hozzáférési szabályzatokat, amelyeket összekapcsolhat egy adott erőforrással, például egy táblával vagy blobbal. Ez a szolgáltatás további szabályozhatóságot és rugalmasságot biztosít, szemben az alkalmazás által létrehozott közös hozzáférésű jogosultságkódok jogkivonataival. Amikor csak lehetséges, javasolt a használata. A kiszolgálón tárolt szabályzatokban módosíthatók a meghatározott beállítások, és a jogkivonatban megjelennek a változások anélkül, hogy új jogkivonatot kellene kibocsátani. A jogkivonatban meghatározott beállítások azonban nem módosíthatók új jogkivonat kiadása nélkül. Ez a módszer azt is lehetővé teszi, hogy visszavonja egy érvényes közös hozzáférésű jogosultságkód jogkivonatát még annak lejárta előtt.

> További információkat [a táblákra és üzenetsorokra vonatkozó SAS (közös hozzáférésű jogosultságkód) bemutatását és a blobokra vonatkozó SAS frissítését leíró cikkben](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/) és az MSDN [közös hozzáférésű jogosultságkódok használatát ismertető cikkében](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) talál.

Az alábbi kód bemutatja, hogyan hozhat létre egy olyan jogkivonatot egy közös hozzáférésű jogosultságkódhoz, amely öt percig érvényes. A `GetSharedAccessReferenceForUpload` metódus a közös hozzáférésű jogosultságkód olyan jogkivonatát adja vissza, amellyel feltölthet egy fájlt az Azure Blob Storage-ba.

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

> A teljes minta megtalálható a ValetKey megoldásban, amelyet a [GitHubról](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key) tölthet le. Az ebben a megoldásban található ValetKey.Web projekt egy olyan webalkalmazást tartalmaz, amely tartalmazza a fent említett `ValuesController` osztályt. A ValetKey.Client projektben egy olyan ügyfélalkalmazásra láthat példát, amely ezzel a webalkalmazással kéri le a közös hozzáférésű jogosultságkódhoz tartozó kulcsot, és feltölt egy fájlt a blobtárolóba.

## <a name="next-steps"></a>További lépések

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/valet-key) talál egy, a minta bemutatására szolgáló példát.
- [Forgalomirányító minta](./gatekeeper.md). Ez a minta a Pótkulcs mintával együtt használható fel arra, hogy az alkalmazások és szolgáltatások védelmét egy dedikált gazdapéldánnyal valósítsa meg, amely közvetítőként szolgál az ügyfelek és az alkalmazás vagy szolgáltatás között. A forgalomirányító érvényesíti és vírusmentesíti a kéréseket, valamint közvetíti a kéréseket és az adatokat az ügyfél és az alkalmazás között. Ez egy további biztonsági réteget biztosíthat, és csökkenti a számítógép támadható felületét.
- [Statikus tartalom üzemeltetési minta](./static-content-hosting.md). Leírja, hogyan kell üzembe helyezni statikus tartalmakat egy felhőalapú társzolgáltatásban, amely közvetlenül az ügyfélnek közvetíti az erőforrásokat, hogy ezzel visszaszorítsa a költséges számítási példányok szükségességét. Abban az esetben, ha az erőforrásoknak nem kell nyilvánosan elérhetőnek lenniük, azokat a Pótkulcs minta segítségével védheti meg.
- [Táblákra és üzenetsorokra vonatkozó SAS (közös hozzáférésű jogosultságkód) bemutatása és a blobokra vonatkozó SAS frissítése](https://blogs.msdn.microsoft.com/windowsazurestorage/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas/)
- [Közös hozzáférésű jogosultságkódok használata](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
- [Közös hozzáférésű jogosultságkódos hitelesítés Service Bus használatával](/azure/service-bus-messaging/service-bus-sas)
