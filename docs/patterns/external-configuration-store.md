---
title: "Külső Beállítástárolóval"
description: "Az alkalmazás központi telepítési csomag kívül a konfigurációs adatok áthelyezése egy központi helyen."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="external-configuration-store-pattern"></a><span data-ttu-id="5e8f7-104">Külső a konfigurációs adattároló minta</span><span class="sxs-lookup"><span data-stu-id="5e8f7-104">External Configuration Store pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="5e8f7-105">Az alkalmazás központi telepítési csomag kívül a konfigurációs adatok áthelyezése egy központi helyen.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-105">Move configuration information out of the application deployment package to a centralized location.</span></span> <span data-ttu-id="5e8f7-106">Ez az egyszerűbb kezelés és konfigurációs adatok feletti ellenőrzési, és az alkalmazások és alkalmazáspéldányok közötti megosztás a konfigurációs adatokat biztosít lehetőségek.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-106">This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5e8f7-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="5e8f7-107">Context and problem</span></span>

<span data-ttu-id="5e8f7-108">A legtöbb alkalmazás futásidejű környezetek tartalmazza az alkalmazással együtt telepített fájlok használatban konfigurációs információkat.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-108">The majority of application runtime environments include configuration information that's held in files deployed with the application.</span></span> <span data-ttu-id="5e8f7-109">Néhány esetben szerkesztése után szeretné módosítani az alkalmazás viselkedését üzembe helyezésüket követően ezeket a fájlokat.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-109">In some cases, it's possible to edit these files to change the application behavior after it's been deployed.</span></span> <span data-ttu-id="5e8f7-110">A konfiguráció módosításai kérhetnek az alkalmazást újra kell telepíteni, ami gyakran elfogadhatatlan állásidő és egyéb adminisztratív terhelés mellett.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-110">However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.</span></span>

<span data-ttu-id="5e8f7-111">Helyi konfigurációs fájlok is korlátozza a konfiguráció egyetlen alkalmazás, de egyes esetekben célszerű konfigurációs beállítások megosztására több alkalmazás között.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-111">Local configuration files also limit the configuration to a single application, but sometimes it would be useful to share configuration settings across multiple applications.</span></span> <span data-ttu-id="5e8f7-112">Ilyen például az adatbázis-kapcsolati karakterláncok, a felhasználói felület téma információkat vagy a várólisták és a kapcsolódó alkalmazások által használt tároló URL-címei.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-112">Examples include database connection strings, UI theme information, or the URLs of queues and storage used by a related set of applications.</span></span>

<span data-ttu-id="5e8f7-113">Kezeléséhez a helyi beállításokat az alkalmazás, különösen a felhőben üzemeltetett forgatókönyvek több futó példánya kihívást is.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-113">It's challenging to manage changes to local configurations across multiple running instances of the application, especially in a cloud-hosted scenario.</span></span> <span data-ttu-id="5e8f7-114">Különböző konfigurációs beállítások használatával, a frissítés telepítésekor példányok eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-114">It can result in instances using different configuration settings while the update is being deployed.</span></span>

<span data-ttu-id="5e8f7-115">Alkalmazások és összetevők frissítéseit is követelhetnek sémák konfigurációs módosításait.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-115">In addition, updates to applications and components might require changes to configuration schemas.</span></span> <span data-ttu-id="5e8f7-116">Számos konfigurációs rendszer konfigurációs adatait különböző verziói nem támogatják.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-116">Many configuration systems don't support different versions of configuration information.</span></span>

## <a name="solution"></a><span data-ttu-id="5e8f7-117">Megoldás</span><span class="sxs-lookup"><span data-stu-id="5e8f7-117">Solution</span></span>

<span data-ttu-id="5e8f7-118">A konfigurációs adatok tárolása a külső tárhelyen, és gyors és hatékony olvassa el és konfigurációs beállításainak frissítése használható illesztőfelületet szolgáltasson.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-118">Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings.</span></span> <span data-ttu-id="5e8f7-119">Külső áruházban típusa attól függ, hogy a tároló és a futásidejű környezet, az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-119">The type of external store depends on the hosting and runtime environment of the application.</span></span> <span data-ttu-id="5e8f7-120">A felhőben üzemeltetett forgatókönyvek általában a felhőalapú társzolgáltatás, de lehet egy központi adatbázisba vagy más system.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-120">In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.</span></span>

<span data-ttu-id="5e8f7-121">A biztonsági tár úgy dönt, hogy a konfigurációs adatokat egy felület, amely egységes és könnyen használható hozzáféréssel kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-121">The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access.</span></span> <span data-ttu-id="5e8f7-122">Akkor kell láthassa helytelenül beírt és strukturált formátumban.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-122">It should expose the information in a correctly typed and structured format.</span></span> <span data-ttu-id="5e8f7-123">Végrehajtására is szükség lehet ahhoz, hogy a konfigurációs adatok védelmét, és lehet vajon elég rugalmas az tároló (például a fejlesztés, átmeneti vagy üzemi, beleértve a több kiadási konfigurációs több verziójának engedélyezése a felhasználói hozzáférés engedélyezésére verziója minden egyes).</span><span class="sxs-lookup"><span data-stu-id="5e8f7-123">The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration (such as development, staging, or production, including multiple release versions of each one).</span></span>

> <span data-ttu-id="5e8f7-124">Számos beépített konfigurációs rendszer beolvasni az adatokat, amikor az alkalmazás elindul, és gyors hozzáférést, és az alkalmazás teljesítményére gyakorolt hatásának minimalizálása érdekében a memóriában adatok gyorsítótárazásához.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-124">Many built-in configuration systems read the data when the application starts up, and cache the data in memory to provide fast access and minimize the impact on application performance.</span></span> <span data-ttu-id="5e8f7-125">Típusától biztonsági használt tárolási és a Tiltás késése a ezt a tárolót hasznos lehet a külső a konfigurációs adattároló belül gyorsítótárazást végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-125">Depending on the type of backing store used, and the latency of this store, it might be helpful to implement a caching mechanism within the external configuration store.</span></span> <span data-ttu-id="5e8f7-126">További információkért lásd: a [gyorsítótárazás útmutatást](https://msdn.microsoft.com/library/dn589802.aspx).</span><span class="sxs-lookup"><span data-stu-id="5e8f7-126">For more information, see the [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx).</span></span> <span data-ttu-id="5e8f7-127">Az ábra azt mutatja be, a választható helyi gyorsítótár külső a konfigurációs adattároló mintát áttekintését.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-127">The figure illustrates an overview of the External Configuration Store pattern with optional local cache.</span></span>

![A választható helyi gyorsítótár külső a konfigurációs adattároló mintát áttekintése](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a><span data-ttu-id="5e8f7-129">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="5e8f7-129">Issues and considerations</span></span>

<span data-ttu-id="5e8f7-130">Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="5e8f7-130">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="5e8f7-131">Válassza ki a biztonsági tároló, amely elfogadható teljesítményt, a magas rendelkezésre állás, a szolgáltatás megbízhatóságára nyújt, és a biztonsági másolat készíthető a karbantartási és felügyeleti folyamatának részeként.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-131">Choose a backing store that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process.</span></span> <span data-ttu-id="5e8f7-132">A felhőben üzemeltetett alkalmazásban a felhő tárolási mechanizmus segítségével, akkor a általában ezek a követelmények teljesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-132">In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.</span></span>

<span data-ttu-id="5e8f7-133">Tervezze meg a séma, a biztonsági tároló, hogy lehetővé tegyék a rugalmasságot biztosít a típusú adatok tárolására képes.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-133">Design the schema of the backing store to allow flexibility in the types of information it can hold.</span></span> <span data-ttu-id="5e8f7-134">Győződjön meg arról, hogy az összes konfigurációs követelmények például típusos adatok, beállítások, beállítások különböző verzióinak és más szolgáltatások, az azt használó alkalmazások igénylő biztosít.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-134">Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require.</span></span> <span data-ttu-id="5e8f7-135">A séma kell könnyen terjessze ki további beállításokat támogatja, a követelmények változnak.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-135">The schema should be easy to extend to support additional settings as requirements change.</span></span>

<span data-ttu-id="5e8f7-136">Vegye figyelembe, hogy a mögöttes fizikai képességeit tárolja, hogyan felügyeletében a konfigurációs adatokat tárolja, és a teljesítményre gyakorolt hatás.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-136">Consider the physical capabilities of the backing store, how it relates to the way configuration information is stored, and the effects on performance.</span></span> <span data-ttu-id="5e8f7-137">Például egy beállításait tartalmazó XML-dokumentum tárolásához szükséges konfigurációs felületen vagy az alkalmazás a dokumentum értelmezhető ahhoz, hogy egyes beállítások beolvasása.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-137">For example, storing an XML document containing configuration information will require either the configuration interface or the application to parse the document in order to read individual settings.</span></span> <span data-ttu-id="5e8f7-138">Fog létrehozni, akkor egy összetettebb, beállítás frissítése ellenére gyorsítótárazási a beállítások segítséget eltolási lassabb olvasási teljesítmény.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-138">It'll make updating a setting more complicated, though caching the settings can help to offset slower read performance.</span></span>

<span data-ttu-id="5e8f7-139">Vegye figyelembe, hogyan ad engedélyt a konfigurációs felület a hatókör-vezérlés és a konfigurációs beállítások öröklési.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-139">Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.</span></span> <span data-ttu-id="5e8f7-140">Például lehet a követelmény a hatókör beállításait a szervezetben, alkalmazás és a gép szint.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-140">For example, it might be a requirement to scope configuration settings at the organization, application, and the machine level.</span></span> <span data-ttu-id="5e8f7-141">Módosítania, hogy támogatják a különböző hatóköröket történő hozzáférés delegálására és a beállítások felülírják az egyes alkalmazások engedélyezése vagy tiltása.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-141">It might need to support delegation of control over access to different scopes, and to prevent or allow individual applications to override settings.</span></span>

<span data-ttu-id="5e8f7-142">Győződjön meg arról, hogy a konfigurációs felület is elérhetővé teheti a konfigurációs adatokat, például gépelt értékek, a gyűjtemények, a kulcs/érték párok vagy a tulajdonságcsomagok szükséges formátumban.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-142">Ensure that the configuration interface can expose the configuration data in the required formats such as typed values, collections, key/value pairs, or property bags.</span></span>

<span data-ttu-id="5e8f7-143">Vegye figyelembe, hogy a konfigurációs tároló felület beállítások hibákat tartalmaz, vagy nem található a biztonsági tárolóban működése.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-143">Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store.</span></span> <span data-ttu-id="5e8f7-144">Akkor célszerű térjen vissza az alapértelmezett beállításokat, és a hibák naplózására.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-144">It might be appropriate to return default settings and log errors.</span></span> <span data-ttu-id="5e8f7-145">A nagybetűk konfigurációs beállítás kulcsok vagy nevek, a tárolási és a bináris adatok kezelése és a módon, hogy kezeli-e a null vagy üres értékeket szempontjai figyelembe venni.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-145">Also consider aspects such as the case sensitivity of configuration setting keys or names, the storage and handling of binary data, and the ways that null or empty values are handled.</span></span>

<span data-ttu-id="5e8f7-146">Vegye figyelembe, hogyan védi meg a konfigurációs adatokat, hogy csak a megfelelő felhasználók és az alkalmazások hozzáférjenek.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-146">Consider how to protect the configuration data to allow access to only the appropriate users and applications.</span></span> <span data-ttu-id="5e8f7-147">Ez valószínűleg egy olyan beállítás, a konfigurációs tároló illesztőfelület, de az is biztosítani kell, hogy az adatokat a biztonsági tároló nem érhető el közvetlenül a megfelelő engedély nélkül.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-147">This is likely a feature of the configuration store interface, but it's also necessary to ensure that the data in the backing store can't be accessed directly without the appropriate permission.</span></span> <span data-ttu-id="5e8f7-148">Győződjön meg arról, szigorú elkülönítése is megvalósuljon olvasni és írni a konfigurációs adatok szükséges engedélyekkel.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-148">Ensure strict separation between the permissions required to read and to write configuration data.</span></span> <span data-ttu-id="5e8f7-149">Is vegye figyelembe, hogy vagy azok egy részét a konfigurációs beállítások titkosítani kell, és hogyan ez fogja kell végrehajtani a konfigurációs tároló felületen.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-149">Also consider whether you need to encrypt some or all of the configuration settings, and how this'll be implemented in the configuration store interface.</span></span>

<span data-ttu-id="5e8f7-150">Központilag tárolt konfigurációk, amelyek alkalmazás megváltozzon futásidőben, különösen fontos kell telepíteni, frissíteni, és felügyelt alkalmazás kód telepítésének ugyanazt a mechanizmust használva.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-150">Centrally stored configurations, which change application behavior during runtime, are critically important and should be deployed, updated, and managed using the same mechanisms as deploying application code.</span></span> <span data-ttu-id="5e8f7-151">Például több alkalmazást is érintő változások végre kell hajtani egy teljes vizsgálatot, és győződjön meg arról, hogy a módosítás megfelelő minden olyan alkalmazásnál, amely használja ezt a konfigurációt a telepítési módszerre előkészítése.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-151">For example, changes that can affect more than one application must be carried out using a full test and staged deployment approach to ensure that the change is appropriate for all applications that use this configuration.</span></span> <span data-ttu-id="5e8f7-152">A rendszergazda szerkesztése egy alkalmazás frissítése egy beállítást, ha más alkalmazásokat, amelyek ugyanazt a beállítást használják kedvezőtlen hatással lehetett.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-152">If an administrator edits a setting to update one application, it could adversely impact other applications that use the same setting.</span></span>

<span data-ttu-id="5e8f7-153">Ha egy alkalmazás gyorsítótárba helyezi azt a konfigurációs adatokat, az alkalmazást kell riasztást kap, ha a konfiguráció módosításait.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-153">If an application caches configuration information, the application needs to be alerted if the configuration changes.</span></span> <span data-ttu-id="5e8f7-154">Előfordulhat, hogy a valósíthat meg egy elévülési szabályzatának keresztül gyorsítótárazott konfigurációs adatokat, hogy ezeket az információkat rendszeres időközönként automatikusan frissülnek, és módosításokat felvételre (és a intézkedni).</span><span class="sxs-lookup"><span data-stu-id="5e8f7-154">It might be possible to implement an expiration policy over cached configuration data so that this information is automatically refreshed periodically and any changes picked up (and acted on).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="5e8f7-155">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="5e8f7-155">When to use this pattern</span></span>

<span data-ttu-id="5e8f7-156">Ebben a mintában a következőkhöz hasznos:</span><span class="sxs-lookup"><span data-stu-id="5e8f7-156">This pattern is useful for:</span></span>

- <span data-ttu-id="5e8f7-157">Konfigurációs beállítások, amelyek között vannak megosztva, több alkalmazás és alkalmazáspéldányok, vagy ha normál konfigurációban kell kényszeríthető több alkalmazások és alkalmazáspéldányok.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-157">Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.</span></span>

- <span data-ttu-id="5e8f7-158">Egy szabványos konfigurációs rendszer nem támogatja a szükséges konfigurációs beállítások, például képeket vagy összetett adattípusú tárolásához.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-158">A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.</span></span>

- <span data-ttu-id="5e8f7-159">Az egyes alkalmazások beállításai kiegészítő tárolóként lehet, hogy így alkalmazások vagy azok egy részét a központilag tárolt beállításainak felülbírálására.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-159">As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.</span></span>

- <span data-ttu-id="5e8f7-160">Egyszerű kezelés érdekében több alkalmazást, és opcionálisan ellenőrzési úgy használja a konfigurációs beállítások történő naplózásának révén a konfigurációs adattároló néhány vagy az összes típusú hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-160">As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.</span></span>

## <a name="example"></a><span data-ttu-id="5e8f7-161">Példa</span><span class="sxs-lookup"><span data-stu-id="5e8f7-161">Example</span></span>

<span data-ttu-id="5e8f7-162">A Microsoft Azure szolgáltatásban futtatott alkalmazás egy tipikus választott külsőleg konfigurációs adatok tárolásához, hogy használja az Azure Storage.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-162">In a Microsoft Azure hosted application, a typical choice for storing configuration information externally is to use Azure Storage.</span></span> <span data-ttu-id="5e8f7-163">Ez rugalmas, magas teljesítményt nyújt, és automatikus feladatátvétel magas rendelkezésre állást biztosítani a rendszer háromszor replikálja.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-163">This is resilient, offers high performance, and is replicated three times with automatic failover to offer high availability.</span></span> <span data-ttu-id="5e8f7-164">Az Azure Table storage egy kulcs-érték tárolóban biztosít az értékek egy rugalmas séma használhatja.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-164">Azure Table storage provides a key/value store with the ability to use a flexible schema for the values.</span></span> <span data-ttu-id="5e8f7-165">Az Azure Blob storage biztosítja, hogy külön-külön elnevezett blobok bármilyen típusú adat tárolására képes hierarchikus, tároló-alapú tárolását.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-165">Azure Blob storage provides a hierarchical, container-based store that can hold any type of data in individually named blobs.</span></span>

<span data-ttu-id="5e8f7-166">A következő példa bemutatja, hogyan a konfigurációs adattároló keresztül tárolja, és tegye elérhetővé a konfigurációs adatokat a Blob-tároló implementálhatók.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-166">The following example shows how a configuration store can be implemented over Blob storage to store and expose configuration information.</span></span> <span data-ttu-id="5e8f7-167">A `BlobSettingsStore` osztály kivonatolja a Blob-tároló rendelkezik a konfigurációs adatokat, és megvalósítja az `ISettingsStore` felülete az alábbi kódban látható.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-167">The `BlobSettingsStore` class abstracts Blob storage for holding configuration information, and implements the `ISettingsStore` interface shown in the following code.</span></span>

> <span data-ttu-id="5e8f7-168">Ezt a kódot a rendszer biztosítja a _ExternalConfigurationStore.Cloud_ a projektre a _ExternalConfigurationStore_ az elérhető megoldás [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span><span class="sxs-lookup"><span data-stu-id="5e8f7-168">This code is provided in the _ExternalConfigurationStore.Cloud_ project in the _ExternalConfigurationStore_ solution, available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

<span data-ttu-id="5e8f7-169">Ez az interfész beolvasása és a konfigurációs adattároló tárolt konfigurációs beállításainak frissítése módszerek meghatározása, és egy verziószámot, amely segítségével észleli, hogy nemrég módosították minden olyan konfigurációs beállítást is.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-169">This interface defines methods for retrieving and updating configuration settings held in the configuration store, and includes a version number that can be used to detect whether any configuration settings have been modified recently.</span></span> <span data-ttu-id="5e8f7-170">A `BlobSettingsStore` osztályt használja a `ETag` tulajdonság a BLOB versioning végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-170">The `BlobSettingsStore` class uses the `ETag` property of the blob to implement versioning.</span></span> <span data-ttu-id="5e8f7-171">A `ETag` tulajdonság frissítése automatikusan történik minden alkalommal, amikor a blob írása.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-171">The `ETag` property is updated automatically each time the blob is written.</span></span>

> <span data-ttu-id="5e8f7-172">Úgy lett kialakítva a legegyszerűbb megoldás gépelt értékek, hanem karakterlánc-értékek összes konfigurációs beállítás közzétesz.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-172">By design, this simple solution exposes all configuration settings as string values rather than typed values.</span></span>

<span data-ttu-id="5e8f7-173">A `ExternalConfigurationManager` osztály kínál egy egy `BlobSettingsStore` objektum.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-173">The `ExternalConfigurationManager` class provides a wrapper around a `BlobSettingsStore` object.</span></span> <span data-ttu-id="5e8f7-174">Az alkalmazás az osztály használatával tárolásához és konfigurációs információk lekéréséhez.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-174">An application can use this class to store and retrieve configuration information.</span></span> <span data-ttu-id="5e8f7-175">Ez az osztály használja a Microsoft [reaktív bővítmények](https://msdn.microsoft.com/library/hh242985.aspx) könyvtár keresztül megvalósítása a konfiguráció módosításai teszi közzé a `IObservable` felületet.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-175">This class uses the Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) library to expose any changes made to the configuration through an implementation of the `IObservable` interface.</span></span> <span data-ttu-id="5e8f7-176">Ha egy beállítás meghívásával módosítják a `SetAppSetting` metódus, a `Changed` egy esemény jelenik meg, és ezt az eseményt minden előfizetők értesítést fog kapni.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-176">If a setting is modified by calling the `SetAppSetting` method, the `Changed` event is raised and all subscribers to this event will be notified.</span></span>

<span data-ttu-id="5e8f7-177">Vegye figyelembe, hogy az összes beállítást is a gyorsítótárba helyezett egy `Dictionary` objektum belül a `ExternalConfigurationManager` osztály a gyors hozzáférés.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-177">Note that all settings are also cached in a `Dictionary` object inside the `ExternalConfigurationManager` class for fast access.</span></span> <span data-ttu-id="5e8f7-178">A `GetSetting` egy konfigurációs beállítás beolvasásához használt módszer a adatokat olvas a gyorsítótárból.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-178">The `GetSetting` method used to retrieve a configuration setting reads the data from the cache.</span></span> <span data-ttu-id="5e8f7-179">Ha a beállítás nem található a gyorsítótárban, azt veszi át a `BlobSettingsStore` objektum helyett.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-179">If the setting isn't found in the cache, it's retrieved from the `BlobSettingsStore` object instead.</span></span>

<span data-ttu-id="5e8f7-180">A `GetSettings` metódus meghívja a `CheckForConfigurationChanges` metódus észleli, hogy változott-e a konfigurációs adatokat a blob Storage tárolóban.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-180">The `GetSettings` method invokes the `CheckForConfigurationChanges` method to detect whether the configuration information in blob storage has changed.</span></span> <span data-ttu-id="5e8f7-181">Ennek érdekében a verziószám vizsgálata és összehasonlítva azt az a jelenlegi verziószám birtokában a `ExternalConfigurationManager` objektum.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-181">It does this by examining the version number and comparing it with the current version number held by the `ExternalConfigurationManager` object.</span></span> <span data-ttu-id="5e8f7-182">Ha egy vagy több változtatások történtek, a `Changed` egy esemény jelenik meg, és tárolja a rendszer a konfigurációs beállításokat a `Dictionary` objektum frissítése.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-182">If one or more changes have occurred, the `Changed` event is raised and the configuration settings cached in the `Dictionary` object are refreshed.</span></span> <span data-ttu-id="5e8f7-183">Ez az az alkalmazás a [gyorsítótár-Tartalékoljon mintát](cache-aside.md).</span><span class="sxs-lookup"><span data-stu-id="5e8f7-183">This is an application of the [Cache-Aside pattern](cache-aside.md).</span></span>

<span data-ttu-id="5e8f7-184">A következő kódot a minta azt mutatja be az `Changed` esemény, a `GetSettings` metódust, és a `CheckForConfigurationChanges` metódus valósíthatók meg:</span><span class="sxs-lookup"><span data-stu-id="5e8f7-184">The following code sample shows how the `Changed` event, the `GetSettings` method, and the `CheckForConfigurationChanges` method are implemented:</span></span>

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

> <span data-ttu-id="5e8f7-185">A `ExternalConfigurationManager` osztály is biztosít nevű tulajdonság `Environment`.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-185">The `ExternalConfigurationManager` class also provides a property named `Environment`.</span></span> <span data-ttu-id="5e8f7-186">Ez a tulajdonság támogatja a különböző konfigurációkat különböző környezetekben, például az átmeneti és üzemi futó alkalmazásokra vonatkozóan.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-186">This property supports varying configurations for an application running in different environments, such as staging and production.</span></span>

<span data-ttu-id="5e8f7-187">Egy `ExternalConfigurationManager` objektum is lekérdezheti a `BlobSettingsStore` objektum rendszeres időközönként a módosításokat.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-187">An `ExternalConfigurationManager` object can also query the `BlobSettingsStore` object periodically for any changes.</span></span> <span data-ttu-id="5e8f7-188">A következő kódrészlet a `StartMonitor` metódushívások `CheckForConfigurationChanges` ismeri fel a módosításokat, és indítson egy időközönként a `Changed` esemény, a fentebb leírt módon.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-188">In the following code, the `StartMonitor` method calls `CheckForConfigurationChanges` at an interval to detect any changes and raise the `Changed` event, as described earlier.</span></span>

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

<span data-ttu-id="5e8f7-189">A `ExternalConfigurationManager` osztály által egypéldányos példányként létrejön a `ExternalConfiguration` osztály alább látható.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-189">The `ExternalConfigurationManager` class is instantiated as a singleton instance by the `ExternalConfiguration` class shown below.</span></span>

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

<span data-ttu-id="5e8f7-190">Az alábbi kód származik a `WorkerRole` osztályt a _ExternalConfigurationStore.Cloud_ projekt.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-190">The following code is taken from the `WorkerRole` class in the _ExternalConfigurationStore.Cloud_ project.</span></span> <span data-ttu-id="5e8f7-191">Azt mutatja, hogy az alkalmazás használja a `ExternalConfiguration` osztály beolvasni egy beállítását.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-191">It shows how the application uses the `ExternalConfiguration` class to read a setting.</span></span>

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

<span data-ttu-id="5e8f7-192">A következő kódot a is a `WorkerRole` osztály, bemutatja, hogyan számítógépcsoportra fizetett elő az alkalmazás konfigurációs események.</span><span class="sxs-lookup"><span data-stu-id="5e8f7-192">The following code, also from the `WorkerRole` class, shows how the application subscribes to configuration events.</span></span>

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

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="5e8f7-193">Útmutató és a kapcsolódó minták</span><span class="sxs-lookup"><span data-stu-id="5e8f7-193">Related patterns and guidance</span></span>

- <span data-ttu-id="5e8f7-194">Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span><span class="sxs-lookup"><span data-stu-id="5e8f7-194">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>
