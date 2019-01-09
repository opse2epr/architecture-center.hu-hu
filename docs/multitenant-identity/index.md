---
title: Identitáskezelés a több-bérlős alkalmazásokban
description: Ajánlott eljárások hitelesítéshez, engedélyezéshez, és identitáskezeléshez több-bérlős alkalmazások esetén.
author: MikeWasson
ms.date: 07/21/2017
ms.openlocfilehash: 864317cc98ee0211d4f4274253eda12b72beceed
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114147"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="a33a0-103">Identitáskezelés több-bérlős alkalmazásokban</span><span class="sxs-lookup"><span data-stu-id="a33a0-103">Manage identity in multitenant applications</span></span>

<span data-ttu-id="a33a0-104">A cikksorozat ajánlott eljárásokat ismertet a több-bérlős módhoz az Azure AD hitelesítésre és identitáskezelésre való használata esetén.</span><span class="sxs-lookup"><span data-stu-id="a33a0-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="a33a0-105">[![GitHub](../_images/github.png) Mintakód][sample-application]</span><span class="sxs-lookup"><span data-stu-id="a33a0-105">[![GitHub](../_images/github.png) Sample code][sample-application]</span></span>

<span data-ttu-id="a33a0-106">Több-bérlős alkalmazás létrehozásakor az elsőként jelentkező kihívások egyike a felhasználói identitások kezelése, mivel mostantól minden felhasználó egy bérlőhöz tartozik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="a33a0-107">Például:</span><span class="sxs-lookup"><span data-stu-id="a33a0-107">For example:</span></span>

- <span data-ttu-id="a33a0-108">A felhasználók bejelentkeznek a szervezeti hitelesítő adataikkal.</span><span class="sxs-lookup"><span data-stu-id="a33a0-108">Users sign in with their organizational credentials.</span></span>
- <span data-ttu-id="a33a0-109">A felhasználóknak hozzá kell férniük a szervezet adataihoz, de más bérlők adataihoz nem.</span><span class="sxs-lookup"><span data-stu-id="a33a0-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
- <span data-ttu-id="a33a0-110">A szervezet regisztrálhat az alkalmazásra, majd alkalmazás-szerepköröket rendelhet az egyes tagokhoz.</span><span class="sxs-lookup"><span data-stu-id="a33a0-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="a33a0-111">Az Azure Active Directory (Azure AD) remek funkciókkal rendelkezik, amelyek az összes itt felsorolt forgatókönyvet támogatják.</span><span class="sxs-lookup"><span data-stu-id="a33a0-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="a33a0-112">A cikksorozat kiegészítéseként elkészítettük egy több-bérlős alkalmazás [teljes körű megvalósítását][sample-application].</span><span class="sxs-lookup"><span data-stu-id="a33a0-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample-application] of a multitenant application.</span></span> <span data-ttu-id="a33a0-113">A cikkek az alkalmazás létrehozása során szerzett tapasztalatainkat dolgozzák fel.</span><span class="sxs-lookup"><span data-stu-id="a33a0-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="a33a0-114">Az alkalmazás létrehozásának első lépéseként tekintse meg [A Surveys alkalmazás futtatása][running-the-app] című cikket.</span><span class="sxs-lookup"><span data-stu-id="a33a0-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="a33a0-115">Bevezetés</span><span class="sxs-lookup"><span data-stu-id="a33a0-115">Introduction</span></span>

<span data-ttu-id="a33a0-116">Tegyük fel, hogy egy felhőalapú vállalati SaaS-alkalmazást szeretne elkészíteni.</span><span class="sxs-lookup"><span data-stu-id="a33a0-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="a33a0-117">Az alkalmazásnak természetesen lesznek felhasználói:</span><span class="sxs-lookup"><span data-stu-id="a33a0-117">Of course, the application will have users:</span></span>

![Felhasználók](./images/users.png)

<span data-ttu-id="a33a0-119">Ezek a felhasználók viszont különböző szervezetekhez tartoznak:</span><span class="sxs-lookup"><span data-stu-id="a33a0-119">But those users belong to organizations:</span></span>

![Szervezeti felhasználók](./images/org-users.png)

<span data-ttu-id="a33a0-121">Példa: A Tailspin előfizetéseket értékesít SaaS-alkalmazásához.</span><span class="sxs-lookup"><span data-stu-id="a33a0-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="a33a0-122">A Contoso és a Fabrikam előfizetnek erre az alkalmazásra.</span><span class="sxs-lookup"><span data-stu-id="a33a0-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="a33a0-123">Amikor Ágnes (`alice@contoso`) bejelentkezik, az alkalmazásnak fel kell ismernie, hogy Ágnes a Contoso vállalathoz tartozik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

- <span data-ttu-id="a33a0-124">Ágnesnek *hozzáférésre van szüksége* a Contoso adataihoz.</span><span class="sxs-lookup"><span data-stu-id="a33a0-124">Alice *should* have access to Contoso data.</span></span>
- <span data-ttu-id="a33a0-125">Ágnes *nem férhet hozzá* a Fabrikam adataihoz.</span><span class="sxs-lookup"><span data-stu-id="a33a0-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="a33a0-126">Ez az útmutató bemutatja, hogyan kezelheti a felhasználói identitásokat egy több-bérlős alkalmazásban, ha az [Azure Active Directoryt (Azure AD)](/azure/active-directory) használja a bejelentkezés és hitelesítés kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="a33a0-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory (Azure AD)](/azure/active-directory) to handle sign-in and authentication.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-multitenancy"></a><span data-ttu-id="a33a0-127">Mi az a több-bérlős mód?</span><span class="sxs-lookup"><span data-stu-id="a33a0-127">What is multitenancy?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="a33a0-128">A *bérlőt* felhasználók egy csoportja alkotja.</span><span class="sxs-lookup"><span data-stu-id="a33a0-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="a33a0-129">Egy SaaS-alkalmazásban a bérlő az alkalmazás egyik előfizetőjét vagy ügyfelét jelenti.</span><span class="sxs-lookup"><span data-stu-id="a33a0-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="a33a0-130">A *több-bérlős* egy olyan architektúra, ahol egy alkalmazás egy fizikai példányán több bérlő osztozik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="a33a0-131">Bár a bérlők osztoznak a fizikai erőforrásokon (például a virtuális gépeken vagy a tárterületen), minden bérlő saját logikai példánnyal rendelkezik az alkalmazásból.</span><span class="sxs-lookup"><span data-stu-id="a33a0-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="a33a0-132">Az alkalmazásadatokat a felhasználók egy bérlőn belül általában megosztják egymással, más bérlők felhasználóival azonban nem.</span><span class="sxs-lookup"><span data-stu-id="a33a0-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![Több-bérlős architektúra](./images/multitenant.png)

<span data-ttu-id="a33a0-134">Hasonlítsa össze ezt az architektúrát az egybérlős architektúrával, ahol minden egyes bérlő saját fizikai példánnyal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="a33a0-135">Egybérlős architektúra esetén bérlők hozzáadásakor új példányokat kell indítani az alkalmazásból.</span><span class="sxs-lookup"><span data-stu-id="a33a0-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![Egybérlős alkalmazás](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="a33a0-137">Több-bérlős architektúra és vízszintes méretezés</span><span class="sxs-lookup"><span data-stu-id="a33a0-137">Multitenancy and horizontal scaling</span></span>

<span data-ttu-id="a33a0-138">Ha méretezésre van szükség a felhőben, a leggyakoribb megoldás további fizikai példányok hozzáadása.</span><span class="sxs-lookup"><span data-stu-id="a33a0-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="a33a0-139">Ez más néven a *vízszintes méretezés* vagy *horizontális felskálázás*. Vegyünk példaként egy webalkalmazást.</span><span class="sxs-lookup"><span data-stu-id="a33a0-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="a33a0-140">Nagyobb forgalom kiszolgálásához hozzáadhat több virtuális gépet, és elhelyezheti azokat egy terheléselosztó mögé.</span><span class="sxs-lookup"><span data-stu-id="a33a0-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="a33a0-141">Minden virtuális gép a webalkalmazás egy külön fizikai példányát futtatja.</span><span class="sxs-lookup"><span data-stu-id="a33a0-141">Each VM runs a separate physical instance of the web app.</span></span>

![Terheléselosztás egy webhely esetében](./images/load-balancing.png)

<span data-ttu-id="a33a0-143">Bármelyik kérés átirányítható bármelyik példányra.</span><span class="sxs-lookup"><span data-stu-id="a33a0-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="a33a0-144">A rendszer minden része egyetlen logikai példányként működik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="a33a0-145">Leállíthat vagy el is indíthat egy virtuális gépeket anélkül, hogy ezzel hatással lenne a felhasználókra.</span><span class="sxs-lookup"><span data-stu-id="a33a0-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="a33a0-146">Ebben az architektúrában minden fizikai példány több-bérlős, és a méretezés további példányok hozzáadásával történik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="a33a0-147">Ha az egyik példány leáll, az nincs hatással egyik bérlőre sem.</span><span class="sxs-lookup"><span data-stu-id="a33a0-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="a33a0-148">Identitás több-bérlős alkalmazásokban</span><span class="sxs-lookup"><span data-stu-id="a33a0-148">Identity in a multitenant app</span></span>

<span data-ttu-id="a33a0-149">A több-bérlős alkalmazások esetében a felhasználókat a bérlők kontextusában kell figyelembe venni.</span><span class="sxs-lookup"><span data-stu-id="a33a0-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

### <a name="authentication"></a><span data-ttu-id="a33a0-150">Hitelesítés</span><span class="sxs-lookup"><span data-stu-id="a33a0-150">Authentication</span></span>

- <span data-ttu-id="a33a0-151">A felhasználók a szervezeti hitelesítő adatokkal jelentkeznek be az alkalmazásba.</span><span class="sxs-lookup"><span data-stu-id="a33a0-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="a33a0-152">Nem kell saját felhasználói fiókot létrehozniuk az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="a33a0-152">They don't have to create new user profiles for the app.</span></span>
- <span data-ttu-id="a33a0-153">Az egyazon szervezeten belüli felhasználók ugyanahhoz a bérlőhöz tartoznak.</span><span class="sxs-lookup"><span data-stu-id="a33a0-153">Users within the same organization are part of the same tenant.</span></span>
- <span data-ttu-id="a33a0-154">Ha egy felhasználó bejelentkezik, az alkalmazás tudja, hogy a felhasználó melyik bérlőhöz tartozik.</span><span class="sxs-lookup"><span data-stu-id="a33a0-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

### <a name="authorization"></a><span data-ttu-id="a33a0-155">Engedélyezés</span><span class="sxs-lookup"><span data-stu-id="a33a0-155">Authorization</span></span>

- <span data-ttu-id="a33a0-156">Egy felhasználó műveleteinek engedélyezésekor (például egy erőforrás megtekintésekor) az alkalmazásnak a felhasználó bérlőjét is figyelembe kell vennie.</span><span class="sxs-lookup"><span data-stu-id="a33a0-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
- <span data-ttu-id="a33a0-157">A felhasználók különböző szerepkörökkel rendelkezhetnek az alkalmazásban, mint például a „Rendszergazda” vagy az „Általános jogú felhasználó”.</span><span class="sxs-lookup"><span data-stu-id="a33a0-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="a33a0-158">A szerepkör-hozzárendeléseket az ügyfélnek kell kezelnie, nem az SaaS-szolgáltatónak.</span><span class="sxs-lookup"><span data-stu-id="a33a0-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="a33a0-159">**Példa:**</span><span class="sxs-lookup"><span data-stu-id="a33a0-159">**Example.**</span></span> <span data-ttu-id="a33a0-160">Ágnes, a Contoso alkalmazottja megnyitja az alkalmazást a böngészőből, és rákattint a „Bejelentkezés” gombra.</span><span class="sxs-lookup"><span data-stu-id="a33a0-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="a33a0-161">A rendszer átirányítja őt egy bejelentkezési képernyőre, ahol meg kell adnia a vállalati bejelentkezési adatokat (felhasználónevet és jelszót).</span><span class="sxs-lookup"><span data-stu-id="a33a0-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="a33a0-162">Ezen a ponton `alice@contoso.com` néven van bejelentkezve az alkalmazásba.</span><span class="sxs-lookup"><span data-stu-id="a33a0-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="a33a0-163">Az alkalmazás azt is felismeri, hogy Ágnes az alkalmazás rendszergazdai szintű felhasználója.</span><span class="sxs-lookup"><span data-stu-id="a33a0-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="a33a0-164">Rendszergazdaként láthatja a Contoso vállalathoz tartozó erőforrások teljes listáját.</span><span class="sxs-lookup"><span data-stu-id="a33a0-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="a33a0-165">A Fabrikam vállalat erőforrásait azonban nem látja, mivel csak a saját bérlőjében rendelkezik rendszergazdai jogosultságokkal.</span><span class="sxs-lookup"><span data-stu-id="a33a0-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="a33a0-166">Ebben az útmutatóban kifejezetten az Azure AD-vel való identitáskezeléssel foglalkozunk.</span><span class="sxs-lookup"><span data-stu-id="a33a0-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

- <span data-ttu-id="a33a0-167">Feltételezzük, hogy az ügyfél az Azure AD-ben tárolja felhasználói profiljait (az Office 365- és a Dynamics CRM-bérlőket is beleértve)</span><span class="sxs-lookup"><span data-stu-id="a33a0-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
- <span data-ttu-id="a33a0-168">A helyszíni Active Directoryval (AD) rendelkező ügyfelek az [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) segítségével szinkronizálhatják helyszíni AD-jüket az Azure AD-vel.</span><span class="sxs-lookup"><span data-stu-id="a33a0-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity) to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="a33a0-169">Ha egy helyszíni AD-vel rendelkező ügyfél (a vállalati informatikai házirend miatt vagy egyéb okból kifolyólag) nem tudja használni az Azure AD Connect szolgáltatást, az SaaS-szolgáltató az Active Directory összevonási szolgáltatások (AD FS) használatával is összevonható az ügyfél AD-jével.</span><span class="sxs-lookup"><span data-stu-id="a33a0-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="a33a0-170">Ezt a lehetőséget az [Összevonás az ügyfél AD FS szolgáltatásával](adfs.md) című rész ismerteti.</span><span class="sxs-lookup"><span data-stu-id="a33a0-170">This option is described in [Federating with a customer's AD FS](adfs.md).</span></span>

<span data-ttu-id="a33a0-171">Ez az útmutató más szempontból (pl. adatparticionálás, bérlőnkénti konfiguráció stb.) nem tárgyalja a több-bérlős architektúrákat.</span><span class="sxs-lookup"><span data-stu-id="a33a0-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

[<span data-ttu-id="a33a0-172">**Tovább**</span><span class="sxs-lookup"><span data-stu-id="a33a0-172">**Next**</span></span>](./tailspin.md)

<!-- links -->

[sample-application]: https://github.com/mspnp/multitenant-saas-guidance
[running-the-app]: ./run-the-app.md
