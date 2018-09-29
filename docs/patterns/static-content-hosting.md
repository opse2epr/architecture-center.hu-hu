---
title: Static Content Hosting
description: A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 450d0c4c08098c1ba48e4c0dac3d058a46e3709b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428210"
---
# <a name="static-content-hosting-pattern"></a>Statikus tartalom üzemeltetési minta

[!INCLUDE [header](../_includes/header.md)]

A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt. Ezzel csökkenthető a potenciálisan drága számítási példányok igénye.

## <a name="context-and-problem"></a>Kontextus és probléma

A Webalkalmazások rendszerint tartalmazzák a statikus tartalom néhány elemét. A statikus tartalom magában foglalhat HTML-lapokat és más erőforrásokat, például az ügyfél számára hozzáférhető lemezképeket, amelyek vagy a HTML-lapok részei (például a beágyazott képek, a stíluslapok és az ügyféloldali JavaScript-fájlok), vagy külön letölthetőek (például PDF-dokumentumok).

Bár a webkiszolgálók alkalmasak a kérések optimalizálására a hatékony dinamikus weblapkód-végrehajtás és a kimeneti gyorsítótárazás segítségével, továbbra is kezelniük kell a kéréseket a statikus tartalom letöltéséhez. Ez lefoglalja a feldolgozási ciklusokat, amelyek gyakran végezhetnének hasznosabb feladatokat.

## <a name="solution"></a>Megoldás

A legtöbb felhőalapú üzemeltetési környezetben minimalizálni lehet a számítási példányokra való igényt (például használhat egy kisebb példányt vagy kevesebb példányt) azáltal, hogy megkeresi egy alkalmazás bizonyos erőforrásait és statikus lapjait egy adattárolási szolgáltatásban. A felhőben üzemeltetett tárolás költsége általában sokkal alacsonyabb, mint a számítási példányoké.

Amikor egy alkalmazás egyes részeit egy tárolási szolgáltatásban üzemelteti, a fő szempontok az alkalmazás telepítésével és az olyan erőforrások védelmével kapcsolatosak, amelyekhez a névtelen felhasználóknak nem szabad hozzáférniük.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Az üzemeltetett társzolgáltatásnak fel kell fednie egy HTTP-végpontot, amelyhez a felhasználók hozzáférhetnek a statikus erőforrások letöltéséhez. Bizonyos tárolási szolgáltatások támogatják a HTTPS-t is, így lehetséges olyan erőforrásokat üzemeltetni az adattárolási szolgáltatásokban, amelyek megkövetelik az SSL használatát.

- A legjobb teljesítmény és rendelkezésre állás érdekében érdemes egy tartalomkézbesítési hálózatot (CDN-t) használni, amellyel a tároló tartalmát több különböző adatközpontban gyorsítótárazhatja világszerte. A CDN használatáért azonban általában díjat kell fizetni.

- A tárfiókok gyakran alapértelmezés szerint georeplikáltak, ami olyan események ellen biztosít védelmet, amelyek hatással lehetnek az adatközpontra. Ez azt jelenti, hogy az IP-cím változhat, de az URL-cím ugyanaz marad.

- Ha egyes tartalmak a tárfiókban, más tartalmak pedig az üzemeltetett számítási példányban találhatók, az megnehezítheti az alkalmazás üzembe helyezését és frissítését. Lehetséges, hogy különálló üzembe helyezéseket kell végeznie, és verziószámmal kell ellátnia az alkalmazást és a tartalmakat, hogy egyszerűbben felügyelhesse azokat – különösen, ha a statikus tartalom parancsfájlokat vagy felhasználói felületi összetevőket is magában foglal. Ha azonban csak statikus erőforrásokat kell frissítenie, akkor egyszerűen feltöltheti őket a tárfiókra anélkül, hogy újból üzembe helyezné az alkalmazáscsomagot.

- Előfordulhat, hogy a tárolási szolgáltatások nem támogatják az egyéni tartománynevek használatát. Ebben az esetben fontos az erőforrások teljes URL-címének megadása, mert egy másik tartományban lesznek, mint a hivatkozásokat tartalmazó dinamikusan létrehozott tartalom.

- A tárolókat nyilvános olvasási hozzáféréssel kell konfigurálni, de elengedhetetlen az is, hogy azok ne legyenek nyilvános írási hozzáférésre konfigurálva, hogy a felhasználók ne tudjanak tartalmat feltölteni. Fontolja meg egy pótkulcs vagy egy token használatát, hogy szabályozhassa az olyan erőforrásokhoz való hozzáférést, amelyek nem érhetők el névtelenül – további információ: [Pótkulcs minta](valet-key.md).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi esetekben hasznos:

- A statikus erőforrásokat tartalmazó webhelyek és alkalmazások üzemeltetési költségeinek csökkentése.

- A csak statikus tartalomból és erőforrásokból álló webhelyek üzemeltetési költségeinek csökkentése. Az üzemeltető tárolórendszer képességeitől függően lehetséges egy teljesen statikus webhely üzemeltetése is egy tárfiókban.

- Más üzemeltetési környezetben vagy helyszíni kiszolgálókon futó alkalmazások statikus erőforrásainak és tartalmainak felfedése.

- Több földrajzi területen található adatok megkeresése egy tartalomkézbesítési hálózattal (CDN), amely világszerte több adatközpontban gyorsítótárazza a tároló tartalmát.

- A költségek és a sávszélesség-használat monitorozása. Egy külön tárfiók használata az összes vagy csak bizonyos statikus tartalmakhoz lehetővé teszi, hogy a költségek könnyebben elkülönüljenek az üzemeltetési és futtatási költségektől.

Nem érdemes ezt a mintát használni a következő helyzetekben:

- Az alkalmazásnak fel kell dolgoznia a statikus tartalmat, mielőtt elküldené az ügyfélnek. Szükség lehet például egy dokumentum időbélyeggel való ellátására.

- A statikus tartalom mennyisége elenyésző. A különálló tárolóról érkező tartalom fogadásával járó terhelés meghaladja a számítógép erőforrásaitól való elkülönítés előnyeit.

## <a name="example"></a>Példa

Az Azure Blob Storage tárolóban található statikus tartalom közvetlenül elérhető webböngészővel. Az Azure egy HTTP-alapú felületet biztosít a tárolóhoz, amely nyilvánosan megtekinthető az ügyfelek számára. Például az Azure Blob Storage-tárolók tartalma felfedhető egy URL-cím segítségével a következő formában:

`https://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


A tartalmak feltöltésekor szükséges egy vagy több blobtárolót létrehozni, amely a fájlokat és a dokumentumokat fogja tartalmazni. Vegye figyelembe, hogy az új tárolók alapértelmezett engedélye Privát, ezért ezt módosítania kell Nyilvános engedélyre, hogy az ügyfelek hozzáférhessenek a tartalmakhoz. Ha a tartalmat meg kell védeni a névtelen hozzáféréstől, használhatja a [Pótkulcs mintát](valet-key.md), így a felhasználóknak egy érvényes tokent kell felmutatniuk az erőforrások letöltéséhez.

> A [Blob szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179376.aspx) című rész további információt tartalmaz a Blob Storage-ról, valamint a használatáról és a hozzáféréséről.

Az oldalakon található hivatkozások határozzák meg az erőforrás URL-címét, és az ügyfél közvetlenül fog hozzáférni a társzolgáltatásból. Az ábra azt mutatja be, hogyan történik a statikus összetevők kézbesítése közvetlenül a társzolgáltatásból.

![1. ábra – A statikus összetevők kézbesítése közvetlenül a társzolgáltatásból.](./_images/static-content-hosting-pattern.png)


A lapokon található és az ügyfélnek kézbesített hivatkozásoknak a blobtároló és az erőforrás teljes URL-címét meg kell határozniuk. Például egy olyan lap, amely egy nyilvános tárolóban lévő képre mutat, az alábbi HTML-címet tartalmazhatja.

```html
<img src="https://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> Ha az erőforrások pótkulccsal, például egy Azure közös hozzáférésű jogosultságkóddal védettek, az aláírásnak szerepelnie kell a hivatkozások URL-címeiben.

A StaticContentHosting nevű megoldás, amely bemutatja a külső tároló a statikus erőforrásokkal való használatát, elérhető a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) oldaláról. A StaticContentHosting.Cloud projekt tartalmazza a konfigurációs fájlokat, amelyek megadják a tárfiókot és a tárolót, amely a statikus tartalmat tárolja.

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

- A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) talál egy, a minta bemutatására szolgáló példát.
- [Pótkulcs minta](valet-key.md). Ha nem szeretné, hogy a célként megadott erőforrásokhoz névtelen felhasználók is hozzáférhessenek, akkor konfigurálnia kell a biztonságot a statikus tartalom tárolóján. Leírja, hogyan használhatók a tokenek vagy kulcsok, amelyek korlátozott közvetlen hozzáférést biztosítanak az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz, például egy felhős társzolgáltatáshoz.
- [A Blob szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179376.aspx)
