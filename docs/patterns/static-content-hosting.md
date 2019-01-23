---
title: Statikus tartalom üzemeltetési minta
titleSuffix: Cloud Design Patterns
description: A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.
keywords: tervezési minta
author: dragon119
ms.date: 01/04/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 719f0221ecc8d52267cba3136eec20dadef30b99
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483445"
---
# <a name="static-content-hosting-pattern"></a>Statikus tartalom üzemeltetési minta

A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt. Ezzel csökkenthető a potenciálisan drága számítási példányok igénye.

## <a name="context-and-problem"></a>Kontextus és probléma

A Webalkalmazások rendszerint tartalmazzák a statikus tartalom néhány elemét. A statikus tartalom magában foglalhat HTML-lapokat és más erőforrásokat, például az ügyfél számára hozzáférhető lemezképeket, amelyek vagy a HTML-lapok részei (például a beágyazott képek, a stíluslapok és az ügyféloldali JavaScript-fájlok), vagy külön letölthetőek (például PDF-dokumentumok).

Bár a webkiszolgálók dinamikus renderelési és a kimeneti gyorsítótárazás vannak optimalizálva, azok továbbra is kezelniük kell a kéréseket a statikus tartalom letöltéséhez. Ez lefoglalja a feldolgozási ciklusokat, amelyek gyakran végezhetnének hasznosabb feladatokat.

## <a name="solution"></a>Megoldás

A legtöbb felhőalapú üzemeltetési környezeteket elhelyezhet néhányat az alkalmazás erőforrásait és statikus lapjait egy adattárolási szolgáltatásban. A storage szolgáltatás kérelmeket ezekhez az erőforrásokhoz, csökkentve a terhelését, a számítási erőforrások, amelyek más webes kérések kezelésére is szolgálnak. A felhőben üzemeltetett tárolás költsége általában sokkal alacsonyabb, mint a számítási példányoké.

Amikor egy alkalmazás egyes részeit egy tárolási szolgáltatásban üzemelteti, a fő szempontok az alkalmazás telepítésével és az olyan erőforrások védelmével kapcsolatosak, amelyekhez a névtelen felhasználóknak nem szabad hozzáférniük.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Az üzemeltetett társzolgáltatásnak fel kell fednie egy HTTP-végpontot, amelyhez a felhasználók hozzáférhetnek a statikus erőforrások letöltéséhez. Bizonyos tárolási szolgáltatások támogatják a HTTPS-t is, így lehetséges olyan erőforrásokat üzemeltetni az adattárolási szolgáltatásokban, amelyek megkövetelik az SSL használatát.

- A legjobb teljesítmény és rendelkezésre állás érdekében érdemes egy tartalomkézbesítési hálózatot (CDN-t) használni, amellyel a tároló tartalmát több különböző adatközpontban gyorsítótárazhatja világszerte. A CDN használatáért azonban általában díjat kell fizetni.

- A tárfiókok gyakran alapértelmezés szerint georeplikáltak, ami olyan események ellen biztosít védelmet, amelyek hatással lehetnek az adatközpontra. Ez azt jelenti, hogy az IP-cím változhat, de az URL-cím ugyanaz marad.

- Ha egyes tartalmak a storage-fiókban található, és más tartalmak pedig az üzemeltetett számítási példányban, megnehezítheti üzembe helyezéséhez és az alkalmazás frissítése. Lehetséges, hogy különálló üzembe helyezéseket kell végeznie, és verziószámmal kell ellátnia az alkalmazást és a tartalmakat, hogy egyszerűbben felügyelhesse azokat – különösen, ha a statikus tartalom parancsfájlokat vagy felhasználói felületi összetevőket is magában foglal. Ha azonban csak statikus erőforrásokat kell frissítenie, akkor egyszerűen feltöltheti őket a tárfiókra anélkül, hogy újból üzembe helyezné az alkalmazáscsomagot.

- Előfordulhat, hogy a tárolási szolgáltatások nem támogatják az egyéni tartománynevek használatát. Ebben az esetben fontos az erőforrások teljes URL-címének megadása, mert egy másik tartományban lesznek, mint a hivatkozásokat tartalmazó dinamikusan létrehozott tartalom.

- A tárolókat nyilvános olvasási hozzáféréssel kell konfigurálni, de elengedhetetlen az is, hogy azok ne legyenek nyilvános írási hozzáférésre konfigurálva, hogy a felhasználók ne tudjanak tartalmat feltölteni.

- Fontolja meg egy pótkulcs vagy egy tokent, amelyek nem érhetők el névtelenül erőforrásokhoz való hozzáférés vezérlése. Tekintse meg a [Pótkulcs minta](./valet-key.md) további információt.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi esetekben hasznos:

- A statikus erőforrásokat tartalmazó webhelyek és alkalmazások üzemeltetési költségeinek csökkentése.

- A csak statikus tartalomból és erőforrásokból álló webhelyek üzemeltetési költségeinek csökkentése. Az üzemeltető tárolórendszer képességeitől függően esetleg a storage-fiókban egy teljesen statikus webhely üzemeltetése.

- Más üzemeltetési környezetben vagy helyszíni kiszolgálókon futó alkalmazások statikus erőforrásainak és tartalmainak felfedése.

- Több földrajzi területen található adatok megkeresése egy tartalomkézbesítési hálózattal (CDN), amely világszerte több adatközpontban gyorsítótárazza a tároló tartalmát.

- A költségek és a sávszélesség-használat monitorozása. Egy külön tárfiók használata az összes vagy csak bizonyos statikus tartalmakhoz lehetővé teszi, hogy a költségek könnyebben elkülönüljenek az üzemeltetési és futtatási költségektől.

Nem érdemes ezt a mintát használni a következő helyzetekben:

- Az alkalmazásnak fel kell dolgoznia a statikus tartalmat, mielőtt elküldené az ügyfélnek. Szükség lehet például egy dokumentum időbélyeggel való ellátására.

- A statikus tartalom mennyisége elenyésző. A különálló tárolóról érkező tartalom fogadásával járó terhelés meghaladja a számítógép erőforrásaitól való elkülönítés előnyeit.

## <a name="example"></a>Példa

Az Azure Storage támogatja a kiszolgáló statikus tartalom közvetlenül a tárolót. Fájlok fájlnévkiterjesztései névtelen hozzáférés kérelmeket. Alapértelmezés szerint fájlokat URL-címe van tartozó altartományban található `core.windows.net`, mint például `https://contoso.z4.web.core.windows.net/image.png`. Egyéni tartománynév beállítása, és az Azure CDN használatával HTTPS-kapcsolaton keresztül a fájlok eléréséhez. További információkért lásd: [statikus webhely üzemeltetése az Azure Storage](/azure/storage/blobs/storage-blob-static-website).

![Az alkalmazás közvetlenül a storage szolgáltatás a statikus összetevők kézbesítése](./_images/static-content-hosting-pattern.png)

Statikus webhely üzemeltetése elérhetővé teszi a fájlok a névtelen hozzáférés. Szabályozza, ki férhet hozzá a fájlok van szüksége, ha fájlok tárolására az Azure blob storage-ban, és ezután hozza létre [közös hozzáférésű jogosultságkódot](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) a hozzáférés korlátozásához.

Továbbítja az ügyfélnek az oldalak hivatkozásai az erőforrás teljes URL-címet kell megadnia. Ha az erőforrás, például egy közös hozzáférésű jogosultságkód pótkulcs védi az aláírás az URL-címben szerepelnie kell.

Egy mintaalkalmazás, amely bemutatja a külső tároló a statikus erőforrásokkal érhető el a [GitHub][sample-app]. Ez a minta konfigurációs fájlok használatával adja meg a tárfiók és tároló, amely a statikus tartalmat tárolja.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

A StaticContentHosting.Web projekt Settings.cs fájljának `Settings` osztálya olyan metódusokat tartalmaz, amelyekkel ezek az adatok kinyerhetők, és létrehozható egy sztringérték, amely tartalmazza a felhőalapú tárfiók tárolójának URL-címét.

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

A StaticContentUrlHtmlHelper.cs fájl `StaticContentUrlHtmlHelper` osztálya tartalmaz egy `StaticContentUrl` nevű metódust, amely létrehoz egy, a felhőalapú tárfiók útvonalát tartalmazó URL-címet, ha az általa fogadott URL-cím az ASP.NET gyökérútvonal-karakterével (~) kezdődik.

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

A Views\Home mappa Index.cshtml fájlja tartalmaz egy képet, amely a `StaticContentUrl` metódus használatával létrehoz egy URL-címet az `src` attribútumához.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

- [Statikus tartalom üzemeltetési minta][sample-app]. Ez a minta bemutatja egy mintaalkalmazás.
- [Pótkulcs minta](./valet-key.md). A céloldali erőforrások nem lehet a névtelen felhasználók számára elérhető, ha ez a minta használatával korlátozza a közvetlen hozzáférést.
- [Az Azure kiszolgáló nélküli webalkalmazás](../reference-architectures/serverless/web-app.md). A referenciaarchitektúra által használt statikus webhely üzemeltetése az Azure Functions használatával kiszolgáló nélküli webalkalmazás megvalósításához.

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
