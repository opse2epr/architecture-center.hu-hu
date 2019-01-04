---
title: Statikus tartalom üzemeltetési minta
titleSuffix: Cloud Design Patterns
description: A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: f3d4f544a22ec07a651dd1fb1f88bf957134254c
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009423"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="d1c55-104">Statikus tartalom üzemeltetési minta</span><span class="sxs-lookup"><span data-stu-id="d1c55-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="d1c55-105">A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.</span><span class="sxs-lookup"><span data-stu-id="d1c55-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="d1c55-106">Ezzel csökkenthető a potenciálisan drága számítási példányok igénye.</span><span class="sxs-lookup"><span data-stu-id="d1c55-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="d1c55-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="d1c55-107">Context and problem</span></span>

<span data-ttu-id="d1c55-108">A Webalkalmazások rendszerint tartalmazzák a statikus tartalom néhány elemét.</span><span class="sxs-lookup"><span data-stu-id="d1c55-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="d1c55-109">A statikus tartalom magában foglalhat HTML-lapokat és más erőforrásokat, például az ügyfél számára hozzáférhető lemezképeket, amelyek vagy a HTML-lapok részei (például a beágyazott képek, a stíluslapok és az ügyféloldali JavaScript-fájlok), vagy külön letölthetőek (például PDF-dokumentumok).</span><span class="sxs-lookup"><span data-stu-id="d1c55-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="d1c55-110">Bár a webkiszolgálók alkalmasak a kérések optimalizálására a hatékony dinamikus weblapkód-végrehajtás és a kimeneti gyorsítótárazás segítségével, továbbra is kezelniük kell a kéréseket a statikus tartalom letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="d1c55-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="d1c55-111">Ez lefoglalja a feldolgozási ciklusokat, amelyek gyakran végezhetnének hasznosabb feladatokat.</span><span class="sxs-lookup"><span data-stu-id="d1c55-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="d1c55-112">Megoldás</span><span class="sxs-lookup"><span data-stu-id="d1c55-112">Solution</span></span>

<span data-ttu-id="d1c55-113">A legtöbb felhőalapú üzemeltetési környezetben minimalizálni lehet a számítási példányokra való igényt (például használhat egy kisebb példányt vagy kevesebb példányt) azáltal, hogy megkeresi egy alkalmazás bizonyos erőforrásait és statikus lapjait egy adattárolási szolgáltatásban.</span><span class="sxs-lookup"><span data-stu-id="d1c55-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="d1c55-114">A felhőben üzemeltetett tárolás költsége általában sokkal alacsonyabb, mint a számítási példányoké.</span><span class="sxs-lookup"><span data-stu-id="d1c55-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="d1c55-115">Amikor egy alkalmazás egyes részeit egy tárolási szolgáltatásban üzemelteti, a fő szempontok az alkalmazás telepítésével és az olyan erőforrások védelmével kapcsolatosak, amelyekhez a névtelen felhasználóknak nem szabad hozzáférniük.</span><span class="sxs-lookup"><span data-stu-id="d1c55-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="d1c55-116">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="d1c55-116">Issues and considerations</span></span>

<span data-ttu-id="d1c55-117">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="d1c55-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="d1c55-118">Az üzemeltetett társzolgáltatásnak fel kell fednie egy HTTP-végpontot, amelyhez a felhasználók hozzáférhetnek a statikus erőforrások letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="d1c55-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="d1c55-119">Bizonyos tárolási szolgáltatások támogatják a HTTPS-t is, így lehetséges olyan erőforrásokat üzemeltetni az adattárolási szolgáltatásokban, amelyek megkövetelik az SSL használatát.</span><span class="sxs-lookup"><span data-stu-id="d1c55-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="d1c55-120">A legjobb teljesítmény és rendelkezésre állás érdekében érdemes egy tartalomkézbesítési hálózatot (CDN-t) használni, amellyel a tároló tartalmát több különböző adatközpontban gyorsítótárazhatja világszerte.</span><span class="sxs-lookup"><span data-stu-id="d1c55-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="d1c55-121">A CDN használatáért azonban általában díjat kell fizetni.</span><span class="sxs-lookup"><span data-stu-id="d1c55-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="d1c55-122">A tárfiókok gyakran alapértelmezés szerint georeplikáltak, ami olyan események ellen biztosít védelmet, amelyek hatással lehetnek az adatközpontra.</span><span class="sxs-lookup"><span data-stu-id="d1c55-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="d1c55-123">Ez azt jelenti, hogy az IP-cím változhat, de az URL-cím ugyanaz marad.</span><span class="sxs-lookup"><span data-stu-id="d1c55-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="d1c55-124">Ha egyes tartalmak a tárfiókban, más tartalmak pedig az üzemeltetett számítási példányban találhatók, az megnehezítheti az alkalmazás üzembe helyezését és frissítését.</span><span class="sxs-lookup"><span data-stu-id="d1c55-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="d1c55-125">Lehetséges, hogy különálló üzembe helyezéseket kell végeznie, és verziószámmal kell ellátnia az alkalmazást és a tartalmakat, hogy egyszerűbben felügyelhesse azokat – különösen, ha a statikus tartalom parancsfájlokat vagy felhasználói felületi összetevőket is magában foglal.</span><span class="sxs-lookup"><span data-stu-id="d1c55-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="d1c55-126">Ha azonban csak statikus erőforrásokat kell frissítenie, akkor egyszerűen feltöltheti őket a tárfiókra anélkül, hogy újból üzembe helyezné az alkalmazáscsomagot.</span><span class="sxs-lookup"><span data-stu-id="d1c55-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="d1c55-127">Előfordulhat, hogy a tárolási szolgáltatások nem támogatják az egyéni tartománynevek használatát.</span><span class="sxs-lookup"><span data-stu-id="d1c55-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="d1c55-128">Ebben az esetben fontos az erőforrások teljes URL-címének megadása, mert egy másik tartományban lesznek, mint a hivatkozásokat tartalmazó dinamikusan létrehozott tartalom.</span><span class="sxs-lookup"><span data-stu-id="d1c55-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="d1c55-129">A tárolókat nyilvános olvasási hozzáféréssel kell konfigurálni, de elengedhetetlen az is, hogy azok ne legyenek nyilvános írási hozzáférésre konfigurálva, hogy a felhasználók ne tudjanak tartalmat feltölteni.</span><span class="sxs-lookup"><span data-stu-id="d1c55-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="d1c55-130">Fontolja meg egy pótkulcs vagy egy token használatát, hogy szabályozhassa az olyan erőforrásokhoz való hozzáférést, amelyek nem érhetők el névtelenül – további információ: [Pótkulcs minta](./valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="d1c55-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](./valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="d1c55-131">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="d1c55-131">When to use this pattern</span></span>

<span data-ttu-id="d1c55-132">Ez a minta az alábbi esetekben hasznos:</span><span class="sxs-lookup"><span data-stu-id="d1c55-132">This pattern is useful for:</span></span>

- <span data-ttu-id="d1c55-133">A statikus erőforrásokat tartalmazó webhelyek és alkalmazások üzemeltetési költségeinek csökkentése.</span><span class="sxs-lookup"><span data-stu-id="d1c55-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="d1c55-134">A csak statikus tartalomból és erőforrásokból álló webhelyek üzemeltetési költségeinek csökkentése.</span><span class="sxs-lookup"><span data-stu-id="d1c55-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="d1c55-135">Az üzemeltető tárolórendszer képességeitől függően lehetséges egy teljesen statikus webhely üzemeltetése is egy tárfiókban.</span><span class="sxs-lookup"><span data-stu-id="d1c55-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="d1c55-136">Más üzemeltetési környezetben vagy helyszíni kiszolgálókon futó alkalmazások statikus erőforrásainak és tartalmainak felfedése.</span><span class="sxs-lookup"><span data-stu-id="d1c55-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="d1c55-137">Több földrajzi területen található adatok megkeresése egy tartalomkézbesítési hálózattal (CDN), amely világszerte több adatközpontban gyorsítótárazza a tároló tartalmát.</span><span class="sxs-lookup"><span data-stu-id="d1c55-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="d1c55-138">A költségek és a sávszélesség-használat monitorozása.</span><span class="sxs-lookup"><span data-stu-id="d1c55-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="d1c55-139">Egy külön tárfiók használata az összes vagy csak bizonyos statikus tartalmakhoz lehetővé teszi, hogy a költségek könnyebben elkülönüljenek az üzemeltetési és futtatási költségektől.</span><span class="sxs-lookup"><span data-stu-id="d1c55-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="d1c55-140">Nem érdemes ezt a mintát használni a következő helyzetekben:</span><span class="sxs-lookup"><span data-stu-id="d1c55-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="d1c55-141">Az alkalmazásnak fel kell dolgoznia a statikus tartalmat, mielőtt elküldené az ügyfélnek.</span><span class="sxs-lookup"><span data-stu-id="d1c55-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="d1c55-142">Szükség lehet például egy dokumentum időbélyeggel való ellátására.</span><span class="sxs-lookup"><span data-stu-id="d1c55-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="d1c55-143">A statikus tartalom mennyisége elenyésző.</span><span class="sxs-lookup"><span data-stu-id="d1c55-143">The volume of static content is very small.</span></span> <span data-ttu-id="d1c55-144">A különálló tárolóról érkező tartalom fogadásával járó terhelés meghaladja a számítógép erőforrásaitól való elkülönítés előnyeit.</span><span class="sxs-lookup"><span data-stu-id="d1c55-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="d1c55-145">Példa</span><span class="sxs-lookup"><span data-stu-id="d1c55-145">Example</span></span>

<span data-ttu-id="d1c55-146">Az Azure Blob Storage tárolóban található statikus tartalom közvetlenül elérhető webböngészővel.</span><span class="sxs-lookup"><span data-stu-id="d1c55-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="d1c55-147">Az Azure egy HTTP-alapú felületet biztosít a tárolóhoz, amely nyilvánosan megtekinthető az ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="d1c55-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="d1c55-148">Például az Azure Blob Storage-tárolók tartalma felfedhető egy URL-cím segítségével a következő formában:</span><span class="sxs-lookup"><span data-stu-id="d1c55-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`https://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`

<span data-ttu-id="d1c55-149">A tartalmak feltöltésekor szükséges egy vagy több blobtárolót létrehozni, amely a fájlokat és a dokumentumokat fogja tartalmazni.</span><span class="sxs-lookup"><span data-stu-id="d1c55-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="d1c55-150">Vegye figyelembe, hogy az új tárolók alapértelmezett engedélye Privát, ezért ezt módosítania kell Nyilvános engedélyre, hogy az ügyfelek hozzáférhessenek a tartalmakhoz.</span><span class="sxs-lookup"><span data-stu-id="d1c55-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="d1c55-151">Ha a tartalmat meg kell védeni a névtelen hozzáféréstől, használhatja a [Pótkulcs mintát](./valet-key.md), így a felhasználóknak egy érvényes tokent kell felmutatniuk az erőforrások letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="d1c55-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](./valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="d1c55-152">A [Blob szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179376.aspx) című rész további információt tartalmaz a Blob Storage-ról, valamint a használatáról és a hozzáféréséről.</span><span class="sxs-lookup"><span data-stu-id="d1c55-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="d1c55-153">Az oldalakon található hivatkozások határozzák meg az erőforrás URL-címét, és az ügyfél közvetlenül fog hozzáférni a társzolgáltatásból.</span><span class="sxs-lookup"><span data-stu-id="d1c55-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="d1c55-154">Az ábra azt mutatja be, hogyan történik a statikus összetevők kézbesítése közvetlenül a társzolgáltatásból.</span><span class="sxs-lookup"><span data-stu-id="d1c55-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![1. ábra – A statikus összetevők kézbesítése közvetlenül a társzolgáltatásból.](./_images/static-content-hosting-pattern.png)

<span data-ttu-id="d1c55-156">A lapokon található és az ügyfélnek kézbesített hivatkozásoknak a blobtároló és az erőforrás teljes URL-címét meg kell határozniuk.</span><span class="sxs-lookup"><span data-stu-id="d1c55-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="d1c55-157">Például egy olyan lap, amely egy nyilvános tárolóban lévő képre mutat, az alábbi HTML-címet tartalmazhatja.</span><span class="sxs-lookup"><span data-stu-id="d1c55-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="https://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="d1c55-158">Ha az erőforrások pótkulccsal, például egy Azure közös hozzáférésű jogosultságkóddal védettek, az aláírásnak szerepelnie kell a hivatkozások URL-címeiben.</span><span class="sxs-lookup"><span data-stu-id="d1c55-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="d1c55-159">A StaticContentHosting nevű megoldás, amely bemutatja a külső tároló a statikus erőforrásokkal való használatát, elérhető a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) oldaláról.</span><span class="sxs-lookup"><span data-stu-id="d1c55-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="d1c55-160">A StaticContentHosting.Cloud projekt tartalmazza a konfigurációs fájlokat, amelyek megadják a tárfiókot és a tárolót, amely a statikus tartalmat tárolja.</span><span class="sxs-lookup"><span data-stu-id="d1c55-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="d1c55-161">A StaticContentHosting.Web projekt Settings.cs fájljának `Settings` osztálya olyan metódusokat tartalmaz, amelyekkel ezek az adatok kinyerhetők, és létrehozható egy sztringérték, amely tartalmazza a felhőalapú tárfiók tárolójának URL-címét.</span><span class="sxs-lookup"><span data-stu-id="d1c55-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="d1c55-162">A StaticContentUrlHtmlHelper.cs fájl `StaticContentUrlHtmlHelper` osztálya tartalmaz egy `StaticContentUrl` nevű metódust, amely létrehoz egy, a felhőalapú tárfiók útvonalát tartalmazó URL-címet, ha az általa fogadott URL-cím az ASP.NET gyökérútvonal-karakterével (~) kezdődik.</span><span class="sxs-lookup"><span data-stu-id="d1c55-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="d1c55-163">A Views\Home mappa Index.cshtml fájlja tartalmaz egy képet, amely a `StaticContentUrl` metódus használatával létrehoz egy URL-címet az `src` attribútumához.</span><span class="sxs-lookup"><span data-stu-id="d1c55-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="d1c55-164">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="d1c55-164">Related patterns and guidance</span></span>

- <span data-ttu-id="d1c55-165">A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting) talál egy, a minta bemutatására szolgáló példát.</span><span class="sxs-lookup"><span data-stu-id="d1c55-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="d1c55-166">[Pótkulcs minta](./valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="d1c55-166">[Valet Key pattern](./valet-key.md).</span></span> <span data-ttu-id="d1c55-167">Ha nem szeretné, hogy a célként megadott erőforrásokhoz névtelen felhasználók is hozzáférhessenek, akkor konfigurálnia kell a biztonságot a statikus tartalom tárolóján.</span><span class="sxs-lookup"><span data-stu-id="d1c55-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="d1c55-168">Leírja, hogyan használhatók a tokenek vagy kulcsok, amelyek korlátozott közvetlen hozzáférést biztosítanak az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz, például egy felhős társzolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="d1c55-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- [<span data-ttu-id="d1c55-169">A Blob szolgáltatással kapcsolatos fogalmak</span><span class="sxs-lookup"><span data-stu-id="d1c55-169">Blob Service Concepts</span></span>](https://msdn.microsoft.com/library/azure/dd179376.aspx)
