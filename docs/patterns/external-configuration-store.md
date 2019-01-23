---
title: Külső konfigurációs tár mintája
titleSuffix: Cloud Design Patterns
description: A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: fd006437aab934d951d0a0bc947d32878edbf9d8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482385"
---
# <a name="external-configuration-store-pattern"></a><span data-ttu-id="11566-104">Külső konfigurációs tár mintája</span><span class="sxs-lookup"><span data-stu-id="11566-104">External Configuration Store pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="11566-105">A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre.</span><span class="sxs-lookup"><span data-stu-id="11566-105">Move configuration information out of the application deployment package to a centralized location.</span></span> <span data-ttu-id="11566-106">Ezzel lehetőség nyílik a konfigurációs adatok egyszerűbb kezelésére és felügyeletére, valamint a konfigurációs adatok alkalmazások és alkalmazáspéldányok közötti megosztására.</span><span class="sxs-lookup"><span data-stu-id="11566-106">This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="11566-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="11566-107">Context and problem</span></span>

<span data-ttu-id="11566-108">A legtöbb alkalmazásfuttató környezetben találhatók az alkalmazással együtt üzembe helyezett fájlokban tárolt konfigurációs adatok.</span><span class="sxs-lookup"><span data-stu-id="11566-108">The majority of application runtime environments include configuration information that's held in files deployed with the application.</span></span> <span data-ttu-id="11566-109">Bizonyos esetekben ezek a fájlok szerkeszthetők, így az üzembe helyezés után is módosítható az alkalmazás viselkedése.</span><span class="sxs-lookup"><span data-stu-id="11566-109">In some cases, it's possible to edit these files to change the application behavior after it's been deployed.</span></span> <span data-ttu-id="11566-110">Az ilyen konfigurációmódosítások után viszont újból üzembe kell helyezni az alkalmazást, ez pedig gyakran elfogadhatatlan állásidőt és egyéb adminisztrációs terhelést jelenthet.</span><span class="sxs-lookup"><span data-stu-id="11566-110">However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.</span></span>

<span data-ttu-id="11566-111">A helyi konfigurációs fájlok ráadásul egy alkalmazásra korlátozzák a konfigurációt, pedig néha hasznos lenne megosztani több alkalmazás között is a konfigurációs beállításokat.</span><span class="sxs-lookup"><span data-stu-id="11566-111">Local configuration files also limit the configuration to a single application, but sometimes it would be useful to share configuration settings across multiple applications.</span></span> <span data-ttu-id="11566-112">Ilyenek például az adatbázis-kapcsolati sztringek, a felhasználói felület témájára vonatkozó információk vagy az alkalmazások kapcsolódó készlete által használt üzenetsorok és tárolók URL-címei.</span><span class="sxs-lookup"><span data-stu-id="11566-112">Examples include database connection strings, UI theme information, or the URLs of queues and storage used by a related set of applications.</span></span>

<span data-ttu-id="11566-113">A helyi konfigurációk változásainak az alkalmazás több futó példányán történő kezelése kihívást jelenthet, különösen egy felhőben üzembe helyezett forgatókönyv esetén.</span><span class="sxs-lookup"><span data-stu-id="11566-113">It's challenging to manage changes to local configurations across multiple running instances of the application, especially in a cloud-hosted scenario.</span></span> <span data-ttu-id="11566-114">Ez különböző konfigurációs beállításokat használó példányokat eredményezhet a frissítés üzembe helyezése során.</span><span class="sxs-lookup"><span data-stu-id="11566-114">It can result in instances using different configuration settings while the update is being deployed.</span></span>

<span data-ttu-id="11566-115">Ezenfelül az alkalmazások és összetevők frissítéséhez szükség lehet a konfigurációs sémák módosítására.</span><span class="sxs-lookup"><span data-stu-id="11566-115">In addition, updates to applications and components might require changes to configuration schemas.</span></span> <span data-ttu-id="11566-116">Számos konfigurációs rendszer nem támogatja a különböző konfigurációsadat-verziókat.</span><span class="sxs-lookup"><span data-stu-id="11566-116">Many configuration systems don't support different versions of configuration information.</span></span>

## <a name="solution"></a><span data-ttu-id="11566-117">Megoldás</span><span class="sxs-lookup"><span data-stu-id="11566-117">Solution</span></span>

<span data-ttu-id="11566-118">A konfigurációs adatokat tárolja egy külső tárban, és biztosítson egy felületet, amely segítségével gyorsan és hatékonyan olvashat és frissíthet konfigurációs beállításokat.</span><span class="sxs-lookup"><span data-stu-id="11566-118">Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings.</span></span> <span data-ttu-id="11566-119">A külső tároló típusa az alkalmazás üzemeltetési és futtatókörnyezetétől függ.</span><span class="sxs-lookup"><span data-stu-id="11566-119">The type of external store depends on the hosting and runtime environment of the application.</span></span> <span data-ttu-id="11566-120">A felhőben üzembe helyezett forgatókönyv esetében ez általában egy felhőalapú tárolási szolgáltatás, de lehet üzemeltetett adatbázis vagy más rendszer is.</span><span class="sxs-lookup"><span data-stu-id="11566-120">In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.</span></span>

<span data-ttu-id="11566-121">A konfigurációs adatokhoz választott háttértárhoz szükség van egy felületre, amely stabil és könnyen használható hozzáférést biztosít.</span><span class="sxs-lookup"><span data-stu-id="11566-121">The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access.</span></span> <span data-ttu-id="11566-122">Az információkat típussal helyesen ellátott és strukturált formátumban kell elérhetővé tennie.</span><span class="sxs-lookup"><span data-stu-id="11566-122">It should expose the information in a correctly typed and structured format.</span></span> <span data-ttu-id="11566-123">Arra is szükség lehet, hogy a konfigurációs adatok védelme érdekében az implementáció engedélyezze a felhasználók hozzáférését, és elég rugalmas legyen ahhoz, hogy a konfiguráció több verzióját is tárolja (például fejlesztési, átmeneti, éles üzemi, mindegyiknél akár több kiadásverziót beleértve).</span><span class="sxs-lookup"><span data-stu-id="11566-123">The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration (such as development, staging, or production, including multiple release versions of each one).</span></span>

> <span data-ttu-id="11566-124">Számos beépített konfigurációs rendszer beolvassa az adatokat az alkalmazás indulásakor, majd gyorsítótárazza az adatokat a memóriában, hogy gyors hozzáférést biztosítson, és a lehető legkisebbre csökkentse az alkalmazás teljesítményére gyakorolt hatást.</span><span class="sxs-lookup"><span data-stu-id="11566-124">Many built-in configuration systems read the data when the application starts up, and cache the data in memory to provide fast access and minimize the impact on application performance.</span></span> <span data-ttu-id="11566-125">A használt háttértár típusától és a tár késésétől függően hasznos lehet implementálni egy gyorsítótárazási mechanizmust a külső konfigurációs táron belül.</span><span class="sxs-lookup"><span data-stu-id="11566-125">Depending on the type of backing store used, and the latency of this store, it might be helpful to implement a caching mechanism within the external configuration store.</span></span> <span data-ttu-id="11566-126">További információt a [gyorsítótárazási útmutatóban](https://msdn.microsoft.com/library/dn589802.aspx) találhat.</span><span class="sxs-lookup"><span data-stu-id="11566-126">For more information, see the [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx).</span></span> <span data-ttu-id="11566-127">Az ábra a külső konfigurációs tár mintájának és az opcionális helyi gyorsítótárnak az áttekintését teszi lehetővé.</span><span class="sxs-lookup"><span data-stu-id="11566-127">The figure illustrates an overview of the External Configuration Store pattern with optional local cache.</span></span>

![A külső konfigurációs tár mintájának és az opcionális helyi gyorsítótárnak az áttekintése](./_images/external-configuration-store-overview.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="11566-129">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="11566-129">Issues and considerations</span></span>

<span data-ttu-id="11566-130">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="11566-130">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="11566-131">Válasszon olyan háttértárat, amely elfogadható teljesítményt, magas rendelkezésre állást és robusztus működést biztosít, valamint készíthető róla biztonsági másolat az alkalmazás karbantartási és adminisztrációs folyamatának részeként.</span><span class="sxs-lookup"><span data-stu-id="11566-131">Choose a backing store that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process.</span></span> <span data-ttu-id="11566-132">Egy felhőben üzemeltetett alkalmazásban egy felhőalapú tárolási mechanizmus használata általában jó választás a követelmények teljesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="11566-132">In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.</span></span>

<span data-ttu-id="11566-133">Úgy tervezze meg a háttértár sémáját, hogy rugalmasan tudja kezelni a tárolt adatok típusait.</span><span class="sxs-lookup"><span data-stu-id="11566-133">Design the schema of the backing store to allow flexibility in the types of information it can hold.</span></span> <span data-ttu-id="11566-134">Győződjön meg arról, hogy minden konfigurációs követelménynek megfeleljen (például típussal ellátott adatok, beállításgyűjtemények, több beállításverzió és bármely más olyan funkció, amelyre a háttértárt használó alkalmazásoknak szükségük lehet).</span><span class="sxs-lookup"><span data-stu-id="11566-134">Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require.</span></span> <span data-ttu-id="11566-135">Fontos, hogy a séma könnyen bővíthető legyen, hogy további beállításokat is támogasson, ha megváltoznak a követelmények.</span><span class="sxs-lookup"><span data-stu-id="11566-135">The schema should be easy to extend to support additional settings as requirements change.</span></span>

<span data-ttu-id="11566-136">Vegye figyelembe a háttértár fizikai képességeit, illetve azt, hogyan viszonyul a konfigurációs adatok tárolási módjához, továbbá a teljesítményre gyakorolt hatását.</span><span class="sxs-lookup"><span data-stu-id="11566-136">Consider the physical capabilities of the backing store, how it relates to the way configuration information is stored, and the effects on performance.</span></span> <span data-ttu-id="11566-137">Egy konfigurációs adatokat tartalmazó XML-dokumentum tárolásához például feltétel, hogy az egyes beállítások olvasásához a konfigurációs felület vagy az alkalmazás elemezze a dokumentumot.</span><span class="sxs-lookup"><span data-stu-id="11566-137">For example, storing an XML document containing configuration information will require either the configuration interface or the application to parse the document in order to read individual settings.</span></span> <span data-ttu-id="11566-138">Ettől egy beállítás frissítése bonyolultabbá válik, bár a beállítások gyorsítótárazása segíthet ellensúlyozni a lassabb olvasási teljesítmény.</span><span class="sxs-lookup"><span data-stu-id="11566-138">It'll make updating a setting more complicated, though caching the settings can help to offset slower read performance.</span></span>

<span data-ttu-id="11566-139">Gondolja át, hogyan fogja a konfigurációs felület lehetővé tenni a hatókör felügyeletét és a konfigurációs beállítások öröklését.</span><span class="sxs-lookup"><span data-stu-id="11566-139">Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.</span></span> <span data-ttu-id="11566-140">Követelmény lehet például a konfigurációs beállítások hatókörének korlátozása a cég, az alkalmazás vagy a gépek szintjén.</span><span class="sxs-lookup"><span data-stu-id="11566-140">For example, it might be a requirement to scope configuration settings at the organization, application, and the machine level.</span></span> <span data-ttu-id="11566-141">Előfordulhat, hogy támogatnia kell a különböző hatókörökhöz való hozzáférés fölötti felügyelet delegálását, valamint megakadályozni vagy engedélyezni az egyes alkalmazások számára a beállítások felülbírálását.</span><span class="sxs-lookup"><span data-stu-id="11566-141">It might need to support delegation of control over access to different scopes, and to prevent or allow individual applications to override settings.</span></span>

<span data-ttu-id="11566-142">Győződjön meg arról, hogy a konfigurációs felület képes a szükséges formátumban elérhetővé tenni a konfigurációs adatokat, például típussal ellátott értékekként, gyűjteményekként, a kulcs/érték-párokként vagy tulajdonságcsomagok formájában.</span><span class="sxs-lookup"><span data-stu-id="11566-142">Ensure that the configuration interface can expose the configuration data in the required formats such as typed values, collections, key/value pairs, or property bags.</span></span>

<span data-ttu-id="11566-143">Gondolja át, hogyan fog viselkedni a konfigurációs tár felülete, amikor a beállítások hibásak, vagy nem is léteznek a háttértárban.</span><span class="sxs-lookup"><span data-stu-id="11566-143">Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store.</span></span> <span data-ttu-id="11566-144">Érdemes lehet visszatérni az alapértelmezett beállításokhoz és naplózni a hibákat.</span><span class="sxs-lookup"><span data-stu-id="11566-144">It might be appropriate to return default settings and log errors.</span></span> <span data-ttu-id="11566-145">Az olyan aspektusokat is gondolja végig, mint hogy a konfigurációs beállítások kulcsai vagy nevei megkülönböztetik-e a kis- és nagybetűket, hogyan fogja tárolni és kezelni a bináris adatokat, valamint milyen módon fogja kezelni a nullértékű vagy üres értékeket.</span><span class="sxs-lookup"><span data-stu-id="11566-145">Also consider aspects such as the case sensitivity of configuration setting keys or names, the storage and handling of binary data, and the ways that null or empty values are handled.</span></span>

<span data-ttu-id="11566-146">Gondolja át, hogyan védheti meg a konfigurációs adatokat, hogy csak a megfelelő felhasználók és alkalmazások kapjanak hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="11566-146">Consider how to protect the configuration data to allow access to only the appropriate users and applications.</span></span> <span data-ttu-id="11566-147">A konfigurációs tár felülete valószínűleg nyújt ilyen szolgáltatást, de azt is biztosítani kell, hogy a háttértárban lévő adatokhoz ne lehessen közvetlenül hozzáférni a megfelelő engedély nélkül.</span><span class="sxs-lookup"><span data-stu-id="11566-147">This is likely a feature of the configuration store interface, but it's also necessary to ensure that the data in the backing store can't be accessed directly without the appropriate permission.</span></span> <span data-ttu-id="11566-148">Ügyeljen arra, hogy szigorúan elválassza a konfigurációs adatok olvasásához és írásához szükséges engedélyeket.</span><span class="sxs-lookup"><span data-stu-id="11566-148">Ensure strict separation between the permissions required to read and to write configuration data.</span></span> <span data-ttu-id="11566-149">Azt is vegye figyelembe, hogy titkosítania kell-e a konfigurációs beállítások egy részét vagy egészét, és hogy ez hogyan lesz implementálva a konfigurációs tár felületén.</span><span class="sxs-lookup"><span data-stu-id="11566-149">Also consider whether you need to encrypt some or all of the configuration settings, and how this'll be implemented in the configuration store interface.</span></span>

<span data-ttu-id="11566-150">A központilag tárolt, az alkalmazások viselkedését a futtatás közben módosító konfigurációk különösen fontosak, és ugyanazokkal a mechanizmusokkal kell üzembe helyezni, frissíteni és felügyelni, mint az alkalmazás kódját.</span><span class="sxs-lookup"><span data-stu-id="11566-150">Centrally stored configurations, which change application behavior during runtime, are critically important and should be deployed, updated, and managed using the same mechanisms as deploying application code.</span></span> <span data-ttu-id="11566-151">Az egynél több alkalmazást érintő módosításokat például teljes teszteléssel és szakaszos üzembe helyezéses megközelítéssel kell végrehajtani. Így biztosítható, hogy a módosítás minden, a konfigurációt használó alkalmazás számára megfelelő legyen.</span><span class="sxs-lookup"><span data-stu-id="11566-151">For example, changes that can affect more than one application must be carried out using a full test and staged deployment approach to ensure that the change is appropriate for all applications that use this configuration.</span></span> <span data-ttu-id="11566-152">Ha egy rendszergazda szerkeszt egy beállítást egy alkalmazás frissítéséhez, az negatív hatással lehet más, ugyanazt a beállítást használó alkalmazásokra.</span><span class="sxs-lookup"><span data-stu-id="11566-152">If an administrator edits a setting to update one application, it could adversely impact other applications that use the same setting.</span></span>

<span data-ttu-id="11566-153">Ha egy alkalmazás gyorsítótárazza a konfigurációs adatokat, az alkalmazást riasztani kell, ha módosul a konfiguráció.</span><span class="sxs-lookup"><span data-stu-id="11566-153">If an application caches configuration information, the application needs to be alerted if the configuration changes.</span></span> <span data-ttu-id="11566-154">Meg lehet esetleg valósítani egy, a gyorsítótárazott konfigurációs adatokra vonatkozó elévülési szabályzatot, hogy ez az információ rendszeres időközönként automatikusan frissüljön, és érvénybe lépjenek az esetleges változások.</span><span class="sxs-lookup"><span data-stu-id="11566-154">It might be possible to implement an expiration policy over cached configuration data so that this information is automatically refreshed periodically and any changes picked up (and acted on).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="11566-155">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="11566-155">When to use this pattern</span></span>

<span data-ttu-id="11566-156">Ez a minta az alábbi esetekben hasznos:</span><span class="sxs-lookup"><span data-stu-id="11566-156">This pattern is useful for:</span></span>

- <span data-ttu-id="11566-157">Több alkalmazás és alkalmazáspéldány között megosztott konfigurációs beállításoknál, vagy ahol a szabványos konfigurációt kell kényszeríteni több alkalmazásra és alkalmazáspéldányra.</span><span class="sxs-lookup"><span data-stu-id="11566-157">Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.</span></span>

- <span data-ttu-id="11566-158">Olyan szabványos konfigurációs rendszer esetében, amely nem támogatja az összes szükséges konfigurációs beállítást, például a képek tárolását vagy az összetett adattípusokat.</span><span class="sxs-lookup"><span data-stu-id="11566-158">A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.</span></span>

- <span data-ttu-id="11566-159">Kiegészítő tárolóként az alkalmazások bizonyos beállításaihoz, amely esetleg lehetővé teszi az alkalmazások számára néhány vagy az összes központilag tárolt beállítás felülbírálását.</span><span class="sxs-lookup"><span data-stu-id="11566-159">As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.</span></span>

- <span data-ttu-id="11566-160">Több alkalmazás adminisztrációjának egyszerűsítésére, valamint opcionálisan a konfigurációs beállítások használatának monitorozására a konfigurációs tár néhány vagy összes hozzáféréstípusának naplózása révén.</span><span class="sxs-lookup"><span data-stu-id="11566-160">As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.</span></span>

## <a name="example"></a><span data-ttu-id="11566-161">Példa</span><span class="sxs-lookup"><span data-stu-id="11566-161">Example</span></span>

<span data-ttu-id="11566-162">Egy Microsoft Azure által üzemeltetett alkalmazásban a konfigurációs adatok külső tárolására ideális választás az Azure Storage.</span><span class="sxs-lookup"><span data-stu-id="11566-162">In a Microsoft Azure hosted application, a typical choice for storing configuration information externally is to use Azure Storage.</span></span> <span data-ttu-id="11566-163">Rugalmas, és nagy teljesítményt tesz elérhetővé, ráadásul a magas rendelkezésre állás érdekében háromszor van replikálva, automatikus feladatátvétellel.</span><span class="sxs-lookup"><span data-stu-id="11566-163">This is resilient, offers high performance, and is replicated three times with automatic failover to offer high availability.</span></span> <span data-ttu-id="11566-164">Az Azure Table Storage egy kulcs/érték-tárt biztosít, amely képes rugalmas sémát használni az értékekhez.</span><span class="sxs-lookup"><span data-stu-id="11566-164">Azure Table storage provides a key/value store with the ability to use a flexible schema for the values.</span></span> <span data-ttu-id="11566-165">Az Azure Blob Storage hierarchikus, tárolóalapú tárt biztosít, amely egyénileg elnevezett blobokban bármilyen típusú adatot képes tárolni.</span><span class="sxs-lookup"><span data-stu-id="11566-165">Azure Blob storage provides a hierarchical, container-based store that can hold any type of data in individually named blobs.</span></span>

<span data-ttu-id="11566-166">Az alábbi példa bemutatja, hogyan lehet implementálni egy konfigurációs tárt a Blob Storage-on konfigurációs adatok tárolására és elérhetővé tételére.</span><span class="sxs-lookup"><span data-stu-id="11566-166">The following example shows how a configuration store can be implemented over Blob storage to store and expose configuration information.</span></span> <span data-ttu-id="11566-167">A `BlobSettingsStore` osztály absztrahálja a Blob Storage-ot a konfigurációs adatok tárolására, és implementálja az alábbi kódban látható `ISettingsStore` interfészt.</span><span class="sxs-lookup"><span data-stu-id="11566-167">The `BlobSettingsStore` class abstracts Blob storage for holding configuration information, and implements the `ISettingsStore` interface shown in the following code.</span></span>

> <span data-ttu-id="11566-168">Ez a kód a [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) elérhető _ExternalConfigurationStore_ megoldás _ExternalConfigurationStore.Cloud_ projektjében található meg.</span><span class="sxs-lookup"><span data-stu-id="11566-168">This code is provided in the _ExternalConfigurationStore.Cloud_ project in the _ExternalConfigurationStore_ solution, available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

<span data-ttu-id="11566-169">Ez az interfész a konfigurációs tárban tárolt konfigurációs beállítások lekéréséhez és frissítéséhez határoz meg metódusokat, és tartalmaz egy verziószámot, amely segítségével észlelhető, hogy módosult-e a közelmúltban bármely konfigurációs beállítás.</span><span class="sxs-lookup"><span data-stu-id="11566-169">This interface defines methods for retrieving and updating configuration settings held in the configuration store, and includes a version number that can be used to detect whether any configuration settings have been modified recently.</span></span> <span data-ttu-id="11566-170">A `BlobSettingsStore` osztály a blob `ETag` tulajdonságát használja a verziókövetés megvalósítására.</span><span class="sxs-lookup"><span data-stu-id="11566-170">The `BlobSettingsStore` class uses the `ETag` property of the blob to implement versioning.</span></span> <span data-ttu-id="11566-171">Az `ETag` tulajdonság a blob minden írása után frissül.</span><span class="sxs-lookup"><span data-stu-id="11566-171">The `ETag` property is updated automatically each time the blob is written.</span></span>

> <span data-ttu-id="11566-172">Ez az egyszerű megoldás úgy lett kialakítva, hogy minden konfigurációs beállítást típussal ellátott érték helyett sztringértékként tesz elérhetővé.</span><span class="sxs-lookup"><span data-stu-id="11566-172">By design, this simple solution exposes all configuration settings as string values rather than typed values.</span></span>

<span data-ttu-id="11566-173">Az `ExternalConfigurationManager` osztály burkolót biztosít egy `BlobSettingsStore` objektum körül.</span><span class="sxs-lookup"><span data-stu-id="11566-173">The `ExternalConfigurationManager` class provides a wrapper around a `BlobSettingsStore` object.</span></span> <span data-ttu-id="11566-174">Egy alkalmazás használhatja ezt az osztályt konfigurációs adatok tárolására és lekérésére.</span><span class="sxs-lookup"><span data-stu-id="11566-174">An application can use this class to store and retrieve configuration information.</span></span> <span data-ttu-id="11566-175">Ez az osztály a Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) kódtára segítségével teszi közzé a konfiguráció módosításait az `IObservable` interfész implementációján keresztül.</span><span class="sxs-lookup"><span data-stu-id="11566-175">This class uses the Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) library to expose any changes made to the configuration through an implementation of the `IObservable` interface.</span></span> <span data-ttu-id="11566-176">Ha egy beállítás a `SetAppSetting` metódus meghívása révén módosul, létrejön a `Changed` esemény, és az esemény minden előfizetője értesítést kap.</span><span class="sxs-lookup"><span data-stu-id="11566-176">If a setting is modified by calling the `SetAppSetting` method, the `Changed` event is raised and all subscribers to this event will be notified.</span></span>

<span data-ttu-id="11566-177">Vegye figyelembe, hogy a rendszer a gyors hozzáférés érdekében minden beállítást gyorsítótáraz is egy `Dictionary` objektumban az `ExternalConfigurationManager` osztályon belül.</span><span class="sxs-lookup"><span data-stu-id="11566-177">Note that all settings are also cached in a `Dictionary` object inside the `ExternalConfigurationManager` class for fast access.</span></span> <span data-ttu-id="11566-178">A konfigurációs beállítások lekéréséhez használt `GetSetting` metódus beolvassa a gyorsítótárból az adatokat.</span><span class="sxs-lookup"><span data-stu-id="11566-178">The `GetSetting` method used to retrieve a configuration setting reads the data from the cache.</span></span> <span data-ttu-id="11566-179">Ha a beállítás nem található a gyorsítótárban, az alkalmazás a `BlobSettingsStore` objektumból kéri le helyette.</span><span class="sxs-lookup"><span data-stu-id="11566-179">If the setting isn't found in the cache, it's retrieved from the `BlobSettingsStore` object instead.</span></span>

<span data-ttu-id="11566-180">A `GetSettings` metódus meghívja a `CheckForConfigurationChanges` metódust, hogy az felderítse, hogy a Blob Storage-ban lévő konfigurációs adatok módosultak-e.</span><span class="sxs-lookup"><span data-stu-id="11566-180">The `GetSettings` method invokes the `CheckForConfigurationChanges` method to detect whether the configuration information in blob storage has changed.</span></span> <span data-ttu-id="11566-181">Ehhez megvizsgálja a verziószámot, és összehasonlítja az `ExternalConfigurationManager` objektumban tárolt aktuális verziószámmal.</span><span class="sxs-lookup"><span data-stu-id="11566-181">It does this by examining the version number and comparing it with the current version number held by the `ExternalConfigurationManager` object.</span></span> <span data-ttu-id="11566-182">Ha történt egy vagy több módosítás, létrejön a `Changed` esemény, és a rendszer frissíti a `Dictionary` objektumban gyorsítótárazott konfigurációs beállításokat.</span><span class="sxs-lookup"><span data-stu-id="11566-182">If one or more changes have occurred, the `Changed` event is raised and the configuration settings cached in the `Dictionary` object are refreshed.</span></span> <span data-ttu-id="11566-183">Ez a [gyorsítótár-feltöltési minta](./cache-aside.md) egy alkalmazása.</span><span class="sxs-lookup"><span data-stu-id="11566-183">This is an application of the [Cache-Aside pattern](./cache-aside.md).</span></span>

<span data-ttu-id="11566-184">A következő kódminta a `Changed` esemény, a `GetSettings` metódus és a `CheckForConfigurationChanges` metódus implementálását mutatja be:</span><span class="sxs-lookup"><span data-stu-id="11566-184">The following code sample shows how the `Changed` event, the `GetSettings` method, and the `CheckForConfigurationChanges` method are implemented:</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache.
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> <span data-ttu-id="11566-185">Az `ExternalConfigurationManager` osztály egy `Environment` nevű tulajdonságot is elérhetővé tesz.</span><span class="sxs-lookup"><span data-stu-id="11566-185">The `ExternalConfigurationManager` class also provides a property named `Environment`.</span></span> <span data-ttu-id="11566-186">Ez a tulajdonság támogatja a különböző konfigurációkat eltérő, például átmeneti és éles környezetben futó alkalmazásoknál.</span><span class="sxs-lookup"><span data-stu-id="11566-186">This property supports varying configurations for an application running in different environments, such as staging and production.</span></span>

<span data-ttu-id="11566-187">Egy `ExternalConfigurationManager` objektum a `BlobSettingsStore` objektumot is lekérdezheti rendszeres időközönként, hogy történt-e módosítás.</span><span class="sxs-lookup"><span data-stu-id="11566-187">An `ExternalConfigurationManager` object can also query the `BlobSettingsStore` object periodically for any changes.</span></span> <span data-ttu-id="11566-188">A következő kódrészletben a `StartMonitor` metódus meghatározott időközönként meghívja a `CheckForConfigurationChanges` metódust, hogy észlelje az esetleges módosításokat, és a korábban leírt módon létrehozza a `Changed` eseményt.</span><span class="sxs-lookup"><span data-stu-id="11566-188">In the following code, the `StartMonitor` method calls `CheckForConfigurationChanges` at an interval to detect any changes and raise the `Changed` event, as described earlier.</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

<span data-ttu-id="11566-189">Az `ExternalConfigurationManager` osztályt a rendszer egyszeri példányként példányosítja az `ExternalConfiguration` osztállyal, ahogy az alább is látható.</span><span class="sxs-lookup"><span data-stu-id="11566-189">The `ExternalConfigurationManager` class is instantiated as a singleton instance by the `ExternalConfiguration` class shown below.</span></span>

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

<span data-ttu-id="11566-190">Az alábbi kód az _ExternalConfigurationStore.Cloud_ projekt `WorkerRole` osztályából származik.</span><span class="sxs-lookup"><span data-stu-id="11566-190">The following code is taken from the `WorkerRole` class in the _ExternalConfigurationStore.Cloud_ project.</span></span> <span data-ttu-id="11566-191">Bemutatja, hogyan használja az alkalmazás az `ExternalConfiguration` osztályt egy beállítás beolvasására.</span><span class="sxs-lookup"><span data-stu-id="11566-191">It shows how the application uses the `ExternalConfiguration` class to read a setting.</span></span>

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

<span data-ttu-id="11566-192">A következő, szintén a `WorkerRole` osztályból származó kód bemutatja, hogyan iratkozik fel egy alkalmazás konfigurációs eseményekre.</span><span class="sxs-lookup"><span data-stu-id="11566-192">The following code, also from the `WorkerRole` class, shows how the application subscribes to configuration events.</span></span>

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="11566-193">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="11566-193">Related patterns and guidance</span></span>

- <span data-ttu-id="11566-194">A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) talál egy, a minta bemutatására szolgáló példát.</span><span class="sxs-lookup"><span data-stu-id="11566-194">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>
