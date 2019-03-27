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
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299610"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="d6295-104">Statikus tartalom üzemeltetési minta</span><span class="sxs-lookup"><span data-stu-id="d6295-104">Static Content Hosting pattern</span></span>

<span data-ttu-id="d6295-105">A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.</span><span class="sxs-lookup"><span data-stu-id="d6295-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="d6295-106">Ezzel csökkenthető a potenciálisan drága számítási példányok igénye.</span><span class="sxs-lookup"><span data-stu-id="d6295-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="d6295-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="d6295-107">Context and problem</span></span>

<span data-ttu-id="d6295-108">A Webalkalmazások rendszerint tartalmazzák a statikus tartalom néhány elemét.</span><span class="sxs-lookup"><span data-stu-id="d6295-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="d6295-109">A statikus tartalom magában foglalhat HTML-lapokat és más erőforrásokat, például az ügyfél számára hozzáférhető lemezképeket, amelyek vagy a HTML-lapok részei (például a beágyazott képek, a stíluslapok és az ügyféloldali JavaScript-fájlok), vagy külön letölthetőek (például PDF-dokumentumok).</span><span class="sxs-lookup"><span data-stu-id="d6295-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="d6295-110">Bár a webkiszolgálók dinamikus renderelési és a kimeneti gyorsítótárazás vannak optimalizálva, azok továbbra is kezelniük kell a kéréseket a statikus tartalom letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="d6295-110">Although web servers are optimized for dynamic rendering and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="d6295-111">Ez lefoglalja a feldolgozási ciklusokat, amelyek gyakran végezhetnének hasznosabb feladatokat.</span><span class="sxs-lookup"><span data-stu-id="d6295-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="d6295-112">Megoldás</span><span class="sxs-lookup"><span data-stu-id="d6295-112">Solution</span></span>

<span data-ttu-id="d6295-113">A legtöbb felhőalapú üzemeltetési környezeteket elhelyezhet néhányat az alkalmazás erőforrásait és statikus lapjait egy adattárolási szolgáltatásban.</span><span class="sxs-lookup"><span data-stu-id="d6295-113">In most cloud hosting environments, you can put some of an application's resources and static pages in a storage service.</span></span> <span data-ttu-id="d6295-114">A storage szolgáltatás kérelmeket ezekhez az erőforrásokhoz, csökkentve a terhelését, a számítási erőforrások, amelyek más webes kérések kezelésére is szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="d6295-114">The storage service can serve requests for these resources, reducing load on the compute resources that handle other web requests.</span></span> <span data-ttu-id="d6295-115">A felhőben üzemeltetett tárolás költsége általában sokkal alacsonyabb, mint a számítási példányoké.</span><span class="sxs-lookup"><span data-stu-id="d6295-115">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="d6295-116">Amikor egy alkalmazás egyes részeit egy tárolási szolgáltatásban üzemelteti, a fő szempontok az alkalmazás telepítésével és az olyan erőforrások védelmével kapcsolatosak, amelyekhez a névtelen felhasználóknak nem szabad hozzáférniük.</span><span class="sxs-lookup"><span data-stu-id="d6295-116">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="d6295-117">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="d6295-117">Issues and considerations</span></span>

<span data-ttu-id="d6295-118">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="d6295-118">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="d6295-119">Az üzemeltetett társzolgáltatásnak fel kell fednie egy HTTP-végpontot, amelyhez a felhasználók hozzáférhetnek a statikus erőforrások letöltéséhez.</span><span class="sxs-lookup"><span data-stu-id="d6295-119">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="d6295-120">Bizonyos tárolási szolgáltatások támogatják a HTTPS-t is, így lehetséges olyan erőforrásokat üzemeltetni az adattárolási szolgáltatásokban, amelyek megkövetelik az SSL használatát.</span><span class="sxs-lookup"><span data-stu-id="d6295-120">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="d6295-121">A legjobb teljesítmény és rendelkezésre állás érdekében érdemes egy tartalomkézbesítési hálózatot (CDN-t) használni, amellyel a tároló tartalmát több különböző adatközpontban gyorsítótárazhatja világszerte.</span><span class="sxs-lookup"><span data-stu-id="d6295-121">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="d6295-122">A CDN használatáért azonban általában díjat kell fizetni.</span><span class="sxs-lookup"><span data-stu-id="d6295-122">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="d6295-123">A tárfiókok gyakran alapértelmezés szerint georeplikáltak, ami olyan események ellen biztosít védelmet, amelyek hatással lehetnek az adatközpontra.</span><span class="sxs-lookup"><span data-stu-id="d6295-123">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="d6295-124">Ez azt jelenti, hogy az IP-cím változhat, de az URL-cím ugyanaz marad.</span><span class="sxs-lookup"><span data-stu-id="d6295-124">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="d6295-125">Ha egyes tartalmak a storage-fiókban található, és más tartalmak pedig az üzemeltetett számítási példányban, megnehezítheti üzembe helyezéséhez és az alkalmazás frissítése.</span><span class="sxs-lookup"><span data-stu-id="d6295-125">When some content is located in a storage account and other content is in a hosted compute instance, it becomes more challenging to deploy and update the application.</span></span> <span data-ttu-id="d6295-126">Lehetséges, hogy különálló üzembe helyezéseket kell végeznie, és verziószámmal kell ellátnia az alkalmazást és a tartalmakat, hogy egyszerűbben felügyelhesse azokat – különösen, ha a statikus tartalom parancsfájlokat vagy felhasználói felületi összetevőket is magában foglal.</span><span class="sxs-lookup"><span data-stu-id="d6295-126">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="d6295-127">Ha azonban csak statikus erőforrásokat kell frissítenie, akkor egyszerűen feltöltheti őket a tárfiókra anélkül, hogy újból üzembe helyezné az alkalmazáscsomagot.</span><span class="sxs-lookup"><span data-stu-id="d6295-127">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="d6295-128">Előfordulhat, hogy a tárolási szolgáltatások nem támogatják az egyéni tartománynevek használatát.</span><span class="sxs-lookup"><span data-stu-id="d6295-128">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="d6295-129">Ebben az esetben fontos az erőforrások teljes URL-címének megadása, mert egy másik tartományban lesznek, mint a hivatkozásokat tartalmazó dinamikusan létrehozott tartalom.</span><span class="sxs-lookup"><span data-stu-id="d6295-129">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="d6295-130">A tárolókat nyilvános olvasási hozzáféréssel kell konfigurálni, de elengedhetetlen az is, hogy azok ne legyenek nyilvános írási hozzáférésre konfigurálva, hogy a felhasználók ne tudjanak tartalmat feltölteni.</span><span class="sxs-lookup"><span data-stu-id="d6295-130">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span>

- <span data-ttu-id="d6295-131">Fontolja meg egy pótkulcs vagy egy tokent, amelyek nem érhetők el névtelenül erőforrásokhoz való hozzáférés vezérlése.</span><span class="sxs-lookup"><span data-stu-id="d6295-131">Consider using a valet key or token to control access to resources that shouldn't be available anonymously.</span></span> <span data-ttu-id="d6295-132">Tekintse meg a [Pótkulcs minta](./valet-key.md) további információt.</span><span class="sxs-lookup"><span data-stu-id="d6295-132">See the [Valet Key pattern](./valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="d6295-133">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="d6295-133">When to use this pattern</span></span>

<span data-ttu-id="d6295-134">Ez a minta az alábbi esetekben hasznos:</span><span class="sxs-lookup"><span data-stu-id="d6295-134">This pattern is useful for:</span></span>

- <span data-ttu-id="d6295-135">A statikus erőforrásokat tartalmazó webhelyek és alkalmazások üzemeltetési költségeinek csökkentése.</span><span class="sxs-lookup"><span data-stu-id="d6295-135">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="d6295-136">A csak statikus tartalomból és erőforrásokból álló webhelyek üzemeltetési költségeinek csökkentése.</span><span class="sxs-lookup"><span data-stu-id="d6295-136">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="d6295-137">Az üzemeltető tárolórendszer képességeitől függően esetleg a storage-fiókban egy teljesen statikus webhely üzemeltetése.</span><span class="sxs-lookup"><span data-stu-id="d6295-137">Depending on the capabilities of the hosting provider's storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="d6295-138">Más üzemeltetési környezetben vagy helyszíni kiszolgálókon futó alkalmazások statikus erőforrásainak és tartalmainak felfedése.</span><span class="sxs-lookup"><span data-stu-id="d6295-138">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="d6295-139">Több földrajzi területen található adatok megkeresése egy tartalomkézbesítési hálózattal (CDN), amely világszerte több adatközpontban gyorsítótárazza a tároló tartalmát.</span><span class="sxs-lookup"><span data-stu-id="d6295-139">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="d6295-140">A költségek és a sávszélesség-használat monitorozása.</span><span class="sxs-lookup"><span data-stu-id="d6295-140">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="d6295-141">Egy külön tárfiók használata az összes vagy csak bizonyos statikus tartalmakhoz lehetővé teszi, hogy a költségek könnyebben elkülönüljenek az üzemeltetési és futtatási költségektől.</span><span class="sxs-lookup"><span data-stu-id="d6295-141">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="d6295-142">Nem érdemes ezt a mintát használni a következő helyzetekben:</span><span class="sxs-lookup"><span data-stu-id="d6295-142">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="d6295-143">Az alkalmazásnak fel kell dolgoznia a statikus tartalmat, mielőtt elküldené az ügyfélnek.</span><span class="sxs-lookup"><span data-stu-id="d6295-143">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="d6295-144">Szükség lehet például egy dokumentum időbélyeggel való ellátására.</span><span class="sxs-lookup"><span data-stu-id="d6295-144">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="d6295-145">A statikus tartalom mennyisége elenyésző.</span><span class="sxs-lookup"><span data-stu-id="d6295-145">The volume of static content is very small.</span></span> <span data-ttu-id="d6295-146">A különálló tárolóról érkező tartalom fogadásával járó terhelés meghaladja a számítógép erőforrásaitól való elkülönítés előnyeit.</span><span class="sxs-lookup"><span data-stu-id="d6295-146">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="d6295-147">Példa</span><span class="sxs-lookup"><span data-stu-id="d6295-147">Example</span></span>

<span data-ttu-id="d6295-148">Az Azure Storage támogatja a kiszolgáló statikus tartalom közvetlenül a tárolót.</span><span class="sxs-lookup"><span data-stu-id="d6295-148">Azure Storage supports serving static content directly from a storage container.</span></span> <span data-ttu-id="d6295-149">Fájlok fájlnévkiterjesztései névtelen hozzáférés kérelmeket.</span><span class="sxs-lookup"><span data-stu-id="d6295-149">Files are served through anonymous access requests.</span></span> <span data-ttu-id="d6295-150">Alapértelmezés szerint fájlokat URL-címe van tartozó altartományban található `core.windows.net`, mint például `https://contoso.z4.web.core.windows.net/image.png`.</span><span class="sxs-lookup"><span data-stu-id="d6295-150">By default, files have a URL in a subdomain of `core.windows.net`, such as `https://contoso.z4.web.core.windows.net/image.png`.</span></span> <span data-ttu-id="d6295-151">Egyéni tartománynév beállítása, és az Azure CDN használatával HTTPS-kapcsolaton keresztül a fájlok eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="d6295-151">You can configure a custom domain name, and use Azure CDN to access the files over HTTPS.</span></span> <span data-ttu-id="d6295-152">További információkért lásd: [statikus webhely üzemeltetése az Azure Storage](/azure/storage/blobs/storage-blob-static-website).</span><span class="sxs-lookup"><span data-stu-id="d6295-152">For more information, see [Static website hosting in Azure Storage](/azure/storage/blobs/storage-blob-static-website).</span></span>

![Az alkalmazás közvetlenül a storage szolgáltatás a statikus összetevők kézbesítése](./_images/static-content-hosting-pattern.png)

<span data-ttu-id="d6295-154">Statikus webhely üzemeltetése elérhetővé teszi a fájlok a névtelen hozzáférés.</span><span class="sxs-lookup"><span data-stu-id="d6295-154">Static website hosting makes the files available for anonymous access.</span></span> <span data-ttu-id="d6295-155">Szabályozza, ki férhet hozzá a fájlok van szüksége, ha fájlok tárolására az Azure blob storage-ban, és ezután hozza létre [közös hozzáférésű jogosultságkódot](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) a hozzáférés korlátozásához.</span><span class="sxs-lookup"><span data-stu-id="d6295-155">If you need to control who can access the files, you can store files in Azure blob storage and then generate [shared access signatures](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) to limit access.</span></span>

<span data-ttu-id="d6295-156">Továbbítja az ügyfélnek az oldalak hivatkozásai az erőforrás teljes URL-címet kell megadnia.</span><span class="sxs-lookup"><span data-stu-id="d6295-156">The links in the pages delivered to the client must specify the full URL of the resource.</span></span> <span data-ttu-id="d6295-157">Ha az erőforrás, például egy közös hozzáférésű jogosultságkód pótkulcs védi az aláírás az URL-címben szerepelnie kell.</span><span class="sxs-lookup"><span data-stu-id="d6295-157">If the resource is protected with a valet key, such as a shared access signature, this signature must be included in the URL.</span></span>

<span data-ttu-id="d6295-158">Egy mintaalkalmazás, amely bemutatja a külső tároló a statikus erőforrásokkal érhető el a [GitHub][sample-app].</span><span class="sxs-lookup"><span data-stu-id="d6295-158">A sample application that demonstrates using external storage for static resources is available on [GitHub][sample-app].</span></span> <span data-ttu-id="d6295-159">Ez a minta konfigurációs fájlok használatával adja meg a tárfiók és tároló, amely a statikus tartalmat tárolja.</span><span class="sxs-lookup"><span data-stu-id="d6295-159">This sample uses configuration files to specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="d6295-160">A StaticContentHosting.Web projekt Settings.cs fájljának `Settings` osztálya olyan metódusokat tartalmaz, amelyekkel ezek az adatok kinyerhetők, és létrehozható egy sztringérték, amely tartalmazza a felhőalapú tárfiók tárolójának URL-címét.</span><span class="sxs-lookup"><span data-stu-id="d6295-160">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="d6295-161">A StaticContentUrlHtmlHelper.cs fájl `StaticContentUrlHtmlHelper` osztálya tartalmaz egy `StaticContentUrl` nevű metódust, amely létrehoz egy, a felhőalapú tárfiók útvonalát tartalmazó URL-címet, ha az általa fogadott URL-cím az ASP.NET gyökérútvonal-karakterével (~) kezdődik.</span><span class="sxs-lookup"><span data-stu-id="d6295-161">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="d6295-162">A Views\Home mappa Index.cshtml fájlja tartalmaz egy képet, amely a `StaticContentUrl` metódus használatával létrehoz egy URL-címet az `src` attribútumához.</span><span class="sxs-lookup"><span data-stu-id="d6295-162">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="d6295-163">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="d6295-163">Related patterns and guidance</span></span>

- <span data-ttu-id="d6295-164">[Statikus tartalom üzemeltetési minta][sample-app].</span><span class="sxs-lookup"><span data-stu-id="d6295-164">[Static Content Hosting sample][sample-app].</span></span> <span data-ttu-id="d6295-165">Ez a minta bemutatja egy mintaalkalmazás.</span><span class="sxs-lookup"><span data-stu-id="d6295-165">A sample application that demonstrates this pattern.</span></span>
- <span data-ttu-id="d6295-166">[Pótkulcs minta](./valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="d6295-166">[Valet Key pattern](./valet-key.md).</span></span> <span data-ttu-id="d6295-167">A céloldali erőforrások nem lehet a névtelen felhasználók számára elérhető, ha ez a minta használatával korlátozza a közvetlen hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="d6295-167">If the target resources aren't supposed to be available to anonymous users, use this pattern to restrict direct access.</span></span>
- <span data-ttu-id="d6295-168">[Az Azure kiszolgáló nélküli webalkalmazás](../reference-architectures/serverless/web-app.md).</span><span class="sxs-lookup"><span data-stu-id="d6295-168">[Serverless web application on Azure](../reference-architectures/serverless/web-app.md).</span></span> <span data-ttu-id="d6295-169">A referenciaarchitektúra által használt statikus webhely üzemeltetése az Azure Functions használatával kiszolgáló nélküli webalkalmazás megvalósításához.</span><span class="sxs-lookup"><span data-stu-id="d6295-169">A reference architecture that uses static website hosting with Azure Functions to implement a serverless web app.</span></span>

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
