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
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="cc556-104">Statikus tartalom üzemeltető minta</span><span class="sxs-lookup"><span data-stu-id="cc556-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="cc556-105">Statikus tartalom központi telepítése a felhőalapú tároló szolgáltatás által biztosított közvetlenül az ügyfél számára.</span><span class="sxs-lookup"><span data-stu-id="cc556-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="cc556-106">Ezzel csökkenthető a jelentős mennyiségű számítási példányokért van szükség.</span><span class="sxs-lookup"><span data-stu-id="cc556-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="cc556-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="cc556-107">Context and problem</span></span>

<span data-ttu-id="cc556-108">Webalkalmazások rendszerint a statikus tartalom néhány elemet.</span><span class="sxs-lookup"><span data-stu-id="cc556-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="cc556-109">A statikus tartalom állhatnak HTML-lapok és más erőforrások, például a lemezképeket és az ügyfélhez, vagy a HTML-lapok (például a beágyazott képek, a stíluslapok és az ügyféloldali JavaScript-fájlt) részeként, vagy le kell tölteni (például PDF elérhető dokumentumok dokumentumok).</span><span class="sxs-lookup"><span data-stu-id="cc556-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="cc556-110">Bár a webkiszolgálók jól hangolt optimalizálása hatékony dinamikus lap programkód és a kimeneti gyorsítótár kéréseket, azok továbbra is fennáll a tanúsítványigénylések statikus tartalom letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="cc556-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="cc556-111">Ez felhasználva feldolgozási ciklus, amelyek gyakran ütemezésbe helyezheti a hatékonyabb használatát.</span><span class="sxs-lookup"><span data-stu-id="cc556-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="cc556-112">Megoldás</span><span class="sxs-lookup"><span data-stu-id="cc556-112">Solution</span></span>

<span data-ttu-id="cc556-113">A legtöbb felhőalapú üzemeltetési környezeteket minimalizálni számítási példányokért szükség (például használja a kisebb példánya vagy kevesebb példányt), a tárolás szolgáltatás egy alkalmazás erőforrások és a statikus lapok néhány megkeresésével.</span><span class="sxs-lookup"><span data-stu-id="cc556-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="cc556-114">A felhőben üzemeltetett tárolási költsége általában sokkal kevesebb mint számítási példányok.</span><span class="sxs-lookup"><span data-stu-id="cc556-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="cc556-115">Amikor egy alkalmazás egy tárolási szolgáltatás egyes részei üzemeltet, a fő szempontokat kapcsolódó az alkalmazás központi telepítését, és nem tekinthető a névtelen felhasználók számára elérhető erőforrások védelme.</span><span class="sxs-lookup"><span data-stu-id="cc556-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="cc556-116">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="cc556-116">Issues and considerations</span></span>

<span data-ttu-id="cc556-117">Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="cc556-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="cc556-118">A szolgáltatott társzolgáltatás fel kell fednie a HTTP-végponttal, amely a felhasználók férhetnek hozzá a statikus erőforrások letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="cc556-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="cc556-119">Bizonyos tárolási szolgáltatások HTTPS, is támogatja, így a gazdagép erőforrásokhoz a tárolási szolgáltatások, hogy megkövetelje az SSL Használatát.</span><span class="sxs-lookup"><span data-stu-id="cc556-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="cc556-120">Az a legjobb teljesítmény és rendelkezésre állás érdemes lehet egy tartalomkézbesítési hálózat (CDN) világszerte több adatközpontokban a tároló tartalmának gyorsítótárazásához.</span><span class="sxs-lookup"><span data-stu-id="cc556-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="cc556-121">Azonban valószínű összekapcsolta fizetni a CDN használatával.</span><span class="sxs-lookup"><span data-stu-id="cc556-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="cc556-122">Georeplikált biztosítható a rugalmasság szemben, amelyek hatással lehetnek a datacenter események alapértelmezés szerint a Storage-fiókok gyakran történik.</span><span class="sxs-lookup"><span data-stu-id="cc556-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="cc556-123">Ez azt jelenti, hogy az IP-cím változhatnak, de az URL-címe ugyanaz marad.</span><span class="sxs-lookup"><span data-stu-id="cc556-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="cc556-124">Néhány tartalom található a tárfiók, és más tartalmak üzemeltetett számítási példányt lesz megnehezítheti a az alkalmazás központi telepítését, és frissíti azt.</span><span class="sxs-lookup"><span data-stu-id="cc556-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="cc556-125">Lehetséges, hogy külön központi telepítések és verzió végrehajtásához az alkalmazás és a tartalom könnyebben felügyelje azt&mdash;különösen ha a statikus tartalom tartalmazza a parancsfájlok vagy felhasználói felületi összetevőket.</span><span class="sxs-lookup"><span data-stu-id="cc556-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="cc556-126">Azonban ha csak statikus erőforrások frissítenie kell őket, akkor egyszerűen feltölthetők a tárfiók anélkül, hogy telepítse újra az alkalmazáscsomag.</span><span class="sxs-lookup"><span data-stu-id="cc556-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="cc556-127">Tárolási szolgáltatások előfordulhat, hogy nem használható egyéni tartománynevek.</span><span class="sxs-lookup"><span data-stu-id="cc556-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="cc556-128">Ebben az esetben fontos hivatkozások adja meg a teljes URL-CÍMÉT az erőforrások, mert egy másik tartományban hivatkozásokat tartalmazó dinamikusan generált tartalmából lesz.</span><span class="sxs-lookup"><span data-stu-id="cc556-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="cc556-129">A tárolókban nyilvános olvasási hozzáféréssel kell konfigurálni, de elengedhetetlen győződjön meg arról, hogy azok nincsenek konfigurálva, hogy a felhasználók tudnak tartalom feltöltése nyilvános írás engedéllyel.</span><span class="sxs-lookup"><span data-stu-id="cc556-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="cc556-130">Fontolja meg egy kulcs, vagy a token nem érhető el névtelen erőforrások elérésének szabályozására valet&mdash;tekintse meg a [Valet kulcs mintát](valet-key.md) további információt.</span><span class="sxs-lookup"><span data-stu-id="cc556-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="cc556-131">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="cc556-131">When to use this pattern</span></span>

<span data-ttu-id="cc556-132">Ebben a mintában a következőkhöz hasznos:</span><span class="sxs-lookup"><span data-stu-id="cc556-132">This pattern is useful for:</span></span>

- <span data-ttu-id="cc556-133">Webhelyekhez és alkalmazásokhoz, amelyek bizonyos statikus erőforrásait tartalmazzák az üzemeltetési költségek minimalizálása.</span><span class="sxs-lookup"><span data-stu-id="cc556-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="cc556-134">Minimalizálja a webhelyekhez, amely csak statikus tartalmat és erőforrásokat üzemeltetési költségek.</span><span class="sxs-lookup"><span data-stu-id="cc556-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="cc556-135">Attól függően, hogy a szolgáltató tárolórendszer szolgáltatásait esetleg teljesen a tárfiókokban teljes statikus webhely üzemeltetésére.</span><span class="sxs-lookup"><span data-stu-id="cc556-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="cc556-136">Az ilyen statikus erőforrások és egyéb üzemeltetési környezetekben vagy a helyszíni kiszolgálókon futó alkalmazások tartalmát.</span><span class="sxs-lookup"><span data-stu-id="cc556-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="cc556-137">Tartalom keresése a storage-fiókot világszerte több adatközpont tartalmát proxykiszolgálókhoz tartalomkézbesítési hálózat használatával több földrajzi területen.</span><span class="sxs-lookup"><span data-stu-id="cc556-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="cc556-138">Figyelés a költségeket és a sávszélesség-használat.</span><span class="sxs-lookup"><span data-stu-id="cc556-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="cc556-139">Külön tárfiókot használ vagy azok egy részét a statikus tartalom lehetővé teszi, hogy könnyebben különíteni üzemeltetési költségek és futásidejű költségeket.</span><span class="sxs-lookup"><span data-stu-id="cc556-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="cc556-140">Ez a minta nem lehet a következő esetekben lehet hasznos:</span><span class="sxs-lookup"><span data-stu-id="cc556-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="cc556-141">Az alkalmazás egyes feldolgozási elvégzéséhez a statikus tartalom előtt, az ügyfélnek szüksége van.</span><span class="sxs-lookup"><span data-stu-id="cc556-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="cc556-142">Például hogy szükség lehet időbélyeg hozzáadása a dokumentumhoz.</span><span class="sxs-lookup"><span data-stu-id="cc556-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="cc556-143">A statikus tartalom mennyisége nem nagyon nagy.</span><span class="sxs-lookup"><span data-stu-id="cc556-143">The volume of static content is very small.</span></span> <span data-ttu-id="cc556-144">Növeli a különálló tárhelyet a tartalom lekérése is járó elválasztó a számítási erőforrás költség előnyeit.</span><span class="sxs-lookup"><span data-stu-id="cc556-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="cc556-145">Példa</span><span class="sxs-lookup"><span data-stu-id="cc556-145">Example</span></span>

<span data-ttu-id="cc556-146">Az Azure Blob Storage tárolóban található statikus tartalom elérése közvetlenül webböngészővel.</span><span class="sxs-lookup"><span data-stu-id="cc556-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="cc556-147">Azure keresztül, amely nyilvánosan elérhetővé tehető tárolási olyan HTTP-alapú felületet biztosít az ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="cc556-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="cc556-148">Például egy Azure Blob storage tárolók tartalma fel van fedve egy URL-cím használata a következő formában:</span><span class="sxs-lookup"><span data-stu-id="cc556-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="cc556-149">A tartalmak feltöltéséhez esetén egy vagy több blob tárolók ahhoz, hogy a fájlok és dokumentumok létrehozásához szükséges.</span><span class="sxs-lookup"><span data-stu-id="cc556-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="cc556-150">Vegye figyelembe, hogy az alapértelmezett engedélyt egy új tároló privát, és kell módosítani a nyilvános az ügyfelek tartalmához való hozzáféréshez.</span><span class="sxs-lookup"><span data-stu-id="cc556-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="cc556-151">Szükség esetén a tartalom védelmére a névtelen hozzáférés, Megvalósíthat a [Valet kulcs mintát](valet-key.md) , a felhasználók e egy érvényes tokent, az erőforrások letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="cc556-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="cc556-152">[BLOB-szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179376.aspx) a blob storage és, hogy hozzáférést, és ezzel kapcsolatos információkat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="cc556-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="cc556-153">Minden lapon lévő hivatkozásokra határozza meg az erőforrás URL-CÍMÉT, és közvetlenül a tárolószolgáltatásból az ügyfél fog hozzáférni.</span><span class="sxs-lookup"><span data-stu-id="cc556-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="cc556-154">Az ábra azt mutatja be, az alkalmazás közvetlenül a tárolás szolgáltatás statikus részének kézbesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="cc556-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![1. ábra – az alkalmazás közvetlenül a tárolás szolgáltatás statikus részének továbbítása](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="cc556-156">A hivatkozások a lap kézbesítve lenne az ügyfél a blobtárolót és az erőforrás teljes URL-címet kell megadnia.</span><span class="sxs-lookup"><span data-stu-id="cc556-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="cc556-157">Például a nyilvános tárolókban lévő egy képre mutató hivatkozást tartalmazó lap az alábbi HTML tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="cc556-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="cc556-158">Ha az erőforrások vannak védve, egy valet kulccsal, például egy Azure közös hozzáférésű jogosultságkódot az aláírás szerepelnie kell az URL-címek hivatkozásokra kattintva.</span><span class="sxs-lookup"><span data-stu-id="cc556-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="cc556-159">A megoldás, amely bemutatja a külső tárhelyet használ statikus erőforrások StaticContentHosting nevű érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="cc556-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="cc556-160">A StaticContentHosting.Cloud-projekt tartalmazza a konfigurációs fájlokat adja meg a tárfiók és tároló, amely a statikus tartalmat.</span><span class="sxs-lookup"><span data-stu-id="cc556-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="cc556-161">A `Settings` osztály Settings.cs fájlban a StaticContentHosting.Web projekt kibontása ezeket az értékeket, és a felhő tárolási fiók tároló URL-címet tartalmazó karakterlánc-érték létrehozása metódust tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="cc556-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="cc556-162">A `StaticContentUrlHtmlHelper` StaticContentUrlHtmlHelper.cs fájlban osztály nevű metódus közzététele `StaticContentUrl` létrehozó egy URL-címet tartalmazó elérési útját a felhőalapú társzolgáltatás fiókja, ha az ASP.NET legfelső szintű elérési út egy karaktere (~) kezdődő átadott URL-CÍMÉT.</span><span class="sxs-lookup"><span data-stu-id="cc556-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="cc556-163">A fájl a Views\Home mappában Index.cshtml által használt kép elemet tartalmaz a `StaticContentUrl` módszer az URL-címet létrehozni a `src` attribútum.</span><span class="sxs-lookup"><span data-stu-id="cc556-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="cc556-164">Útmutató és a kapcsolódó minták</span><span class="sxs-lookup"><span data-stu-id="cc556-164">Related patterns and guidance</span></span>

- <span data-ttu-id="cc556-165">Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="cc556-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="cc556-166">[Valet kulcs mintát](valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="cc556-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="cc556-167">A célként megadott erőforrások nem kellene lennie a névtelen felhasználók számára elérhető esetén a tároló, amely a statikus tartalmat tároló keresztül biztonsági végrehajtásához szükséges.</span><span class="sxs-lookup"><span data-stu-id="cc556-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="cc556-168">A token vagy ügyfelek adott erőforrásokhoz vagy szolgáltatásokhoz, például a felhőben üzemeltetett társzolgáltatás korlátozott közvetlen hozzáférést biztosító kulcs használatát ismerteti.</span><span class="sxs-lookup"><span data-stu-id="cc556-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="cc556-169">[Az Azure statikus webhely hatékonyan](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) a Infosys blogjában.</span><span class="sxs-lookup"><span data-stu-id="cc556-169">[An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) on the Infosys blog.</span></span>
- [<span data-ttu-id="cc556-170">A BLOB szolgáltatással kapcsolatos fogalmak</span><span class="sxs-lookup"><span data-stu-id="cc556-170">Blob Service Concepts</span></span>](https://msdn.microsoft.com/library/azure/dd179376.aspx)
