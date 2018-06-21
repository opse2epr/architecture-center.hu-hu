---
title: Statikus tartalom üzemeltetéséhez
description: Statikus tartalom központi telepítése a felhőalapú tároló szolgáltatás által biztosított közvetlenül az ügyfél számára.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541689"
---
# <a name="static-content-hosting-pattern"></a>Statikus tartalom üzemeltető minta

[!INCLUDE [header](../_includes/header.md)]

Statikus tartalom központi telepítése a felhőalapú tároló szolgáltatás által biztosított közvetlenül az ügyfél számára. Ezzel csökkenthető a jelentős mennyiségű számítási példányokért van szükség.

## <a name="context-and-problem"></a>A környezetben, és probléma

Webalkalmazások rendszerint a statikus tartalom néhány elemet. A statikus tartalom állhatnak HTML-lapok és más erőforrások, például a lemezképeket és az ügyfélhez, vagy a HTML-lapok (például a beágyazott képek, a stíluslapok és az ügyféloldali JavaScript-fájlt) részeként, vagy le kell tölteni (például PDF elérhető dokumentumok dokumentumok).

Bár a webkiszolgálók jól hangolt optimalizálása hatékony dinamikus lap programkód és a kimeneti gyorsítótár kéréseket, azok továbbra is fennáll a tanúsítványigénylések statikus tartalom letöltéséhez. Ez felhasználva feldolgozási ciklus, amelyek gyakran ütemezésbe helyezheti a hatékonyabb használatát.

## <a name="solution"></a>Megoldás

A legtöbb felhőalapú üzemeltetési környezeteket minimalizálni számítási példányokért szükség (például használja a kisebb példánya vagy kevesebb példányt), a tárolás szolgáltatás egy alkalmazás erőforrások és a statikus lapok néhány megkeresésével. A felhőben üzemeltetett tárolási költsége általában sokkal kevesebb mint számítási példányok.

Amikor egy alkalmazás egy tárolási szolgáltatás egyes részei üzemeltet, a fő szempontokat kapcsolódó az alkalmazás központi telepítését, és nem tekinthető a névtelen felhasználók számára elérhető erőforrások védelme.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- A szolgáltatott társzolgáltatás fel kell fednie a HTTP-végponttal, amely a felhasználók férhetnek hozzá a statikus erőforrások letöltéséhez. Bizonyos tárolási szolgáltatások HTTPS, is támogatja, így a gazdagép erőforrásokhoz a tárolási szolgáltatások, hogy megkövetelje az SSL Használatát.

- Az a legjobb teljesítmény és rendelkezésre állás érdemes lehet egy tartalomkézbesítési hálózat (CDN) világszerte több adatközpontokban a tároló tartalmának gyorsítótárazásához. Azonban valószínű összekapcsolta fizetni a CDN használatával.

- Georeplikált biztosítható a rugalmasság szemben, amelyek hatással lehetnek a datacenter események alapértelmezés szerint a Storage-fiókok gyakran történik. Ez azt jelenti, hogy az IP-cím változhatnak, de az URL-címe ugyanaz marad.

- Néhány tartalom található a tárfiók, és más tartalmak üzemeltetett számítási példányt lesz megnehezítheti a az alkalmazás központi telepítését, és frissíti azt. Lehetséges, hogy külön központi telepítések és verzió végrehajtásához az alkalmazás és a tartalom könnyebben felügyelje azt&mdash;különösen ha a statikus tartalom tartalmazza a parancsfájlok vagy felhasználói felületi összetevőket. Azonban ha csak statikus erőforrások frissítenie kell őket, akkor egyszerűen feltölthetők a tárfiók anélkül, hogy telepítse újra az alkalmazáscsomag.

- Tárolási szolgáltatások előfordulhat, hogy nem használható egyéni tartománynevek. Ebben az esetben fontos hivatkozások adja meg a teljes URL-CÍMÉT az erőforrások, mert egy másik tartományban hivatkozásokat tartalmazó dinamikusan generált tartalmából lesz.

- A tárolókban nyilvános olvasási hozzáféréssel kell konfigurálni, de elengedhetetlen győződjön meg arról, hogy azok nincsenek konfigurálva, hogy a felhasználók tudnak tartalom feltöltése nyilvános írás engedéllyel. Fontolja meg egy kulcs, vagy a token nem érhető el névtelen erőforrások elérésének szabályozására valet&mdash;tekintse meg a [Valet kulcs mintát](valet-key.md) további információt.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában a következőkhöz hasznos:

- Webhelyekhez és alkalmazásokhoz, amelyek bizonyos statikus erőforrásait tartalmazzák az üzemeltetési költségek minimalizálása.

- Minimalizálja a webhelyekhez, amely csak statikus tartalmat és erőforrásokat üzemeltetési költségek. Attól függően, hogy a szolgáltató tárolórendszer szolgáltatásait esetleg teljesen a tárfiókokban teljes statikus webhely üzemeltetésére.

- Az ilyen statikus erőforrások és egyéb üzemeltetési környezetekben vagy a helyszíni kiszolgálókon futó alkalmazások tartalmát.

- Tartalom keresése a storage-fiókot világszerte több adatközpont tartalmát proxykiszolgálókhoz tartalomkézbesítési hálózat használatával több földrajzi területen.

- Figyelés a költségeket és a sávszélesség-használat. Külön tárfiókot használ vagy azok egy részét a statikus tartalom lehetővé teszi, hogy könnyebben különíteni üzemeltetési költségek és futásidejű költségeket.

Ez a minta nem lehet a következő esetekben lehet hasznos:

- Az alkalmazás egyes feldolgozási elvégzéséhez a statikus tartalom előtt, az ügyfélnek szüksége van. Például hogy szükség lehet időbélyeg hozzáadása a dokumentumhoz.

- A statikus tartalom mennyisége nem nagyon nagy. Növeli a különálló tárhelyet a tartalom lekérése is járó elválasztó a számítási erőforrás költség előnyeit.

## <a name="example"></a>Példa

Az Azure Blob Storage tárolóban található statikus tartalom elérése közvetlenül webböngészővel. Azure keresztül, amely nyilvánosan elérhetővé tehető tárolási olyan HTTP-alapú felületet biztosít az ügyfelek számára. Például egy Azure Blob storage tárolók tartalma fel van fedve egy URL-cím használata a következő formában:

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


A tartalmak feltöltéséhez esetén egy vagy több blob tárolók ahhoz, hogy a fájlok és dokumentumok létrehozásához szükséges. Vegye figyelembe, hogy az alapértelmezett engedélyt egy új tároló privát, és kell módosítani a nyilvános az ügyfelek tartalmához való hozzáféréshez. Szükség esetén a tartalom védelmére a névtelen hozzáférés, Megvalósíthat a [Valet kulcs mintát](valet-key.md) , a felhasználók e egy érvényes tokent, az erőforrások letöltéséhez.

> [BLOB-szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179376.aspx) a blob storage és, hogy hozzáférést, és ezzel kapcsolatos információkat tartalmaz.

Minden lapon lévő hivatkozásokra határozza meg az erőforrás URL-CÍMÉT, és közvetlenül a tárolószolgáltatásból az ügyfél fog hozzáférni. Az ábra azt mutatja be, az alkalmazás közvetlenül a tárolás szolgáltatás statikus részének kézbesítéséhez.

![1. ábra – az alkalmazás közvetlenül a tárolás szolgáltatás statikus részének továbbítása](./_images/static-content-hosting-pattern.png)


A hivatkozások a lap kézbesítve lenne az ügyfél a blobtárolót és az erőforrás teljes URL-címet kell megadnia. Például a nyilvános tárolókban lévő egy képre mutató hivatkozást tartalmazó lap az alábbi HTML tartalmazhat.

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> Ha az erőforrások vannak védve, egy valet kulccsal, például egy Azure közös hozzáférésű jogosultságkódot az aláírás szerepelnie kell az URL-címek hivatkozásokra kattintva.

A megoldás, amely bemutatja a külső tárhelyet használ statikus erőforrások StaticContentHosting nevű érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting). A StaticContentHosting.Cloud-projekt tartalmazza a konfigurációs fájlokat adja meg a tárfiók és tároló, amely a statikus tartalmat.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

A `Settings` osztály Settings.cs fájlban a StaticContentHosting.Web projekt kibontása ezeket az értékeket, és a felhő tárolási fiók tároló URL-címet tartalmazó karakterlánc-érték létrehozása metódust tartalmaz.

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

A `StaticContentUrlHtmlHelper` StaticContentUrlHtmlHelper.cs fájlban osztály nevű metódus közzététele `StaticContentUrl` létrehozó egy URL-címet tartalmazó elérési útját a felhőalapú társzolgáltatás fiókja, ha az ASP.NET legfelső szintű elérési út egy karaktere (~) kezdődő átadott URL-CÍMÉT.

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

A fájl a Views\Home mappában Index.cshtml által használt kép elemet tartalmaz a `StaticContentUrl` módszer az URL-címet létrehozni a `src` attribútum.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

- Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).
- [Valet kulcs mintát](valet-key.md). A célként megadott erőforrások nem kellene lennie a névtelen felhasználók számára elérhető esetén a tároló, amely a statikus tartalmat tároló keresztül biztonsági végrehajtásához szükséges. A token vagy ügyfelek adott erőforrásokhoz vagy szolgáltatásokhoz, például a felhőben üzemeltetett társzolgáltatás korlátozott közvetlen hozzáférést biztosító kulcs használatát ismerteti.
- [Az Azure statikus webhely hatékonyan](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) a Infosys blogjában.
- [A BLOB szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179376.aspx)
