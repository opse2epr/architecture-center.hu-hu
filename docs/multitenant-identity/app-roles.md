---
title: Alkalmazás-szerepkörök
description: Alkalmazás-szerepkörök használatával engedélyezési végrehajtása
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: ec563936e5f00aba79d65844762feeed97ad547d
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/21/2018
---
# <a name="application-roles"></a><span data-ttu-id="c9faf-103">Alkalmazás-szerepkörök</span><span class="sxs-lookup"><span data-stu-id="c9faf-103">Application roles</span></span>

<span data-ttu-id="c9faf-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="c9faf-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="c9faf-105">Alkalmazási szerepköröknek segítségével engedélyek hozzárendelése a felhasználókhoz.</span><span class="sxs-lookup"><span data-stu-id="c9faf-105">Application roles are used to assign permissions to users.</span></span> <span data-ttu-id="c9faf-106">Például a [Dejójáték felmérések] [ Tailspin] alkalmazás határozza meg a következő szerepkörök:</span><span class="sxs-lookup"><span data-stu-id="c9faf-106">For example, the [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="c9faf-107">Rendszergazda.</span><span class="sxs-lookup"><span data-stu-id="c9faf-107">Administrator.</span></span> <span data-ttu-id="c9faf-108">Minden CRUD-műveleteknek a bármely felmérés, amely arra a bérlőre tartozik hajthat végre.</span><span class="sxs-lookup"><span data-stu-id="c9faf-108">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="c9faf-109">Létrehozó.</span><span class="sxs-lookup"><span data-stu-id="c9faf-109">Creator.</span></span> <span data-ttu-id="c9faf-110">Új felmérések hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="c9faf-110">Can create new surveys.</span></span>
* <span data-ttu-id="c9faf-111">Olvasó.</span><span class="sxs-lookup"><span data-stu-id="c9faf-111">Reader.</span></span> <span data-ttu-id="c9faf-112">Bármely felmérések arra a bérlőre tartozó el tud olvasni.</span><span class="sxs-lookup"><span data-stu-id="c9faf-112">Can read any surveys that belong to that tenant.</span></span>

<span data-ttu-id="c9faf-113">Láthatja, hogy szerepkörök végső soron beolvasása lefordítani engedélyek során [engedélyezési].</span><span class="sxs-lookup"><span data-stu-id="c9faf-113">You can see that roles ultimately get translated into permissions, during [authorization].</span></span> <span data-ttu-id="c9faf-114">De az első kérdést hozzárendelése és szerepkörök kezelése.</span><span class="sxs-lookup"><span data-stu-id="c9faf-114">But the first question is how to assign and manage roles.</span></span> <span data-ttu-id="c9faf-115">Felismertük három fő lehetőség közül választhat:</span><span class="sxs-lookup"><span data-stu-id="c9faf-115">We identified three main options:</span></span>

* [<span data-ttu-id="c9faf-116">Az Azure AD alkalmazás-szerepkörök</span><span class="sxs-lookup"><span data-stu-id="c9faf-116">Azure AD App Roles</span></span>](#roles-using-azure-ad-app-roles)
* [<span data-ttu-id="c9faf-117">Azure AD biztonsági csoportokat</span><span class="sxs-lookup"><span data-stu-id="c9faf-117">Azure AD security groups</span></span>](#roles-using-azure-ad-security-groups)
* <span data-ttu-id="c9faf-118">[Alkalmazás-szerepkör kezelő](#roles-using-an-application-role-manager).</span><span class="sxs-lookup"><span data-stu-id="c9faf-118">[Application role manager](#roles-using-an-application-role-manager).</span></span>

## <a name="roles-using-azure-ad-app-roles"></a><span data-ttu-id="c9faf-119">A szerepkörök Azure AD alkalmazás-szerepkörök használatával</span><span class="sxs-lookup"><span data-stu-id="c9faf-119">Roles using Azure AD App Roles</span></span>
<span data-ttu-id="c9faf-120">Ez a megközelítés a Dejójáték felmérések alkalmazásban használt.</span><span class="sxs-lookup"><span data-stu-id="c9faf-120">This is the approach that we used in the Tailspin Surveys app.</span></span>

<span data-ttu-id="c9faf-121">Ezt a módszert használja a Szolgáltatottszoftver-szolgáltató határozza meg az alkalmazás-szerepkörök hozzáadását az alkalmazás jegyzékében.</span><span class="sxs-lookup"><span data-stu-id="c9faf-121">In this approach, The SaaS provider defines the application roles by adding them to the application manifest.</span></span> <span data-ttu-id="c9faf-122">Miután az ügyfél iratkozik fel, az ügyfél Active Directory rendszergazdája felhasználók rendel a szerepkörök.</span><span class="sxs-lookup"><span data-stu-id="c9faf-122">After a customer signs up, an admin for the customer's AD directory assigns users to the roles.</span></span> <span data-ttu-id="c9faf-123">Amikor egy felhasználó bejelentkezik, a felhasználó hozzárendelt szerepkörök küldése a jogcímeket.</span><span class="sxs-lookup"><span data-stu-id="c9faf-123">When a user signs in, the user's assigned roles are sent as claims.</span></span>

> [!NOTE]
> <span data-ttu-id="c9faf-124">Ha az ügyfél rendelkezik az Azure AD Premium, a rendszergazda biztonsági csoport hozzárendelése a szerepkörhöz, és a csoport tagjai öröklik az alkalmazás-szerepkör.</span><span class="sxs-lookup"><span data-stu-id="c9faf-124">If the customer has Azure AD Premium, the admin can assign a security group to a role, and members of the group will inherit the app role.</span></span> <span data-ttu-id="c9faf-125">Ez kényelmes módja a kezelendő szerepköröket, mert a csoport tulajdonosának nem kell lenniük egy AD a rendszergazdának.</span><span class="sxs-lookup"><span data-stu-id="c9faf-125">This is a convenient way to manage roles, because the group owner doesn't need to be an AD admin.</span></span>
> 
> 

<span data-ttu-id="c9faf-126">A módszer előnyei:</span><span class="sxs-lookup"><span data-stu-id="c9faf-126">Advantages of this approach:</span></span>

* <span data-ttu-id="c9faf-127">Egyszerű programozási modell.</span><span class="sxs-lookup"><span data-stu-id="c9faf-127">Simple programming model.</span></span>
* <span data-ttu-id="c9faf-128">Szerepkörök az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="c9faf-128">Roles are specific to the application.</span></span> <span data-ttu-id="c9faf-129">A szerepkör jogcím egy alkalmazás nem kap meg egy másik alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="c9faf-129">The role claims for one application are not sent to another application.</span></span>
* <span data-ttu-id="c9faf-130">Ha az ügyfél az AD-bérlő eltávolítja az alkalmazást, a szerepkörök eltűnik.</span><span class="sxs-lookup"><span data-stu-id="c9faf-130">If the customer removes the application from their AD tenant, the roles go away.</span></span>
* <span data-ttu-id="c9faf-131">Az alkalmazás nem szükséges minden további Active Directory-engedélyek eltérő a felhasználói profil olvasása.</span><span class="sxs-lookup"><span data-stu-id="c9faf-131">The application doesn't need any extra Active Directory permissions, other than reading the user's profile.</span></span>

<span data-ttu-id="c9faf-132">Hátrányai:</span><span class="sxs-lookup"><span data-stu-id="c9faf-132">Drawbacks:</span></span>

* <span data-ttu-id="c9faf-133">Prémium szintű Azure AD nélkül az ügyfelek nem biztonsági csoportok hozzárendelése szerepkörökhöz.</span><span class="sxs-lookup"><span data-stu-id="c9faf-133">Customers without Azure AD Premium cannot assign security groups to roles.</span></span> <span data-ttu-id="c9faf-134">Ezek az ügyfelek minden felhasználó hozzárendelését AD rendszergazda kell elvégezni.</span><span class="sxs-lookup"><span data-stu-id="c9faf-134">For these customers, all user assignments must be done by an AD administrator.</span></span>
* <span data-ttu-id="c9faf-135">Háttér webes API-k, amely nem csatlakozik a webalkalmazást, ha a webalkalmazáshoz tartozó szerepkör-hozzárendelések a webes API-k nem vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="c9faf-135">If you have a backend web API, which is separate from the web app, then role assignments for the web app don't apply to the web API.</span></span> <span data-ttu-id="c9faf-136">Ezt a pontot további tárgyalását lásd: [a háttér webes API biztonságossá tétele].</span><span class="sxs-lookup"><span data-stu-id="c9faf-136">For more discussion of this point, see [Securing a backend web API].</span></span>

### <a name="implementation"></a><span data-ttu-id="c9faf-137">Megvalósítás</span><span class="sxs-lookup"><span data-stu-id="c9faf-137">Implementation</span></span>
<span data-ttu-id="c9faf-138">**A Szerepkörök definiálása.**</span><span class="sxs-lookup"><span data-stu-id="c9faf-138">**Define the roles.**</span></span> <span data-ttu-id="c9faf-139">A Szolgáltatottszoftver-szolgáltató deklarálja az alkalmazás szerepkörök a [alkalmazásjegyzék].</span><span class="sxs-lookup"><span data-stu-id="c9faf-139">The SaaS provider declares the app roles in the [application manifest].</span></span> <span data-ttu-id="c9faf-140">Például itt bejegyzés a jegyzékfájl felmérések alkalmazás:</span><span class="sxs-lookup"><span data-stu-id="c9faf-140">For example, here is the manifest entry for the Surveys app:</span></span>

```json
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

<span data-ttu-id="c9faf-141">A `value` tulajdonság a szerepkör jogcím jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="c9faf-141">The `value`  property appears in the role claim.</span></span> <span data-ttu-id="c9faf-142">A `id` tulajdonság a megadott szerepkör egyedi azonosítója.</span><span class="sxs-lookup"><span data-stu-id="c9faf-142">The `id` property is the unique identifier for the defined role.</span></span> <span data-ttu-id="c9faf-143">Mindig létrehozni egy új GUID azonosítót `id`.</span><span class="sxs-lookup"><span data-stu-id="c9faf-143">Always generate a new GUID value for `id`.</span></span>

<span data-ttu-id="c9faf-144">**Felhasználók hozzárendelése**.</span><span class="sxs-lookup"><span data-stu-id="c9faf-144">**Assign users**.</span></span> <span data-ttu-id="c9faf-145">Ha egy új ügyfél iratkozik fel, az alkalmazás regisztrálva van az ügyfél AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="c9faf-145">When a new customer signs up, the application is registered in the customer's AD tenant.</span></span> <span data-ttu-id="c9faf-146">Ezen a ponton AD, hogy a bérlői rendszergazda megoldással felhasználókat rendelhet szerepköröket.</span><span class="sxs-lookup"><span data-stu-id="c9faf-146">At this point, an AD admin for that tenant can assign users to roles.</span></span>

> [!NOTE]
> <span data-ttu-id="c9faf-147">Ahogy azt korábban említettük, az Azure AD Premium ügyfelek csoportokat is rendelhet biztonsági szerepkörökhöz.</span><span class="sxs-lookup"><span data-stu-id="c9faf-147">As noted earlier, customers with Azure AD Premium can also assign security groups to roles.</span></span>
> 
> 

<span data-ttu-id="c9faf-148">Az Azure-portálon az alábbi képernyőfelvételen látható felhasználóinak és csoportjainak a felmérést alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="c9faf-148">The following screenshot from the Azure portal shows users and groups for the Survey application.</span></span> <span data-ttu-id="c9faf-149">A rendszergazda és a létrehozó olyan csoportok, illetve SurveyAdmin és SurveyCreator szerepkörökhöz hozzárendelt.</span><span class="sxs-lookup"><span data-stu-id="c9faf-149">Admin and Creator are groups, assigned to SurveyAdmin and SurveyCreator roles respectively.</span></span> <span data-ttu-id="c9faf-150">Alice az a felhasználó, aki közvetlenül a SurveyAdmin szerepkör lett hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="c9faf-150">Alice is a user who was assigned directly to the SurveyAdmin role.</span></span> <span data-ttu-id="c9faf-151">Bob és Charles, amely rendelkezik nem közvetlenül a szerepkörhöz hozzárendelt felhasználók.</span><span class="sxs-lookup"><span data-stu-id="c9faf-151">Bob and Charles are users that have not been directly assigned to a role.</span></span>

![Felhasználók és csoportok](./images/running-the-app/users-and-groups.png)

<span data-ttu-id="c9faf-153">Az alábbi képernyőképen látható, Charles része a felügyeleti csoport, hogy örökli a SurveyAdmin szerepkör.</span><span class="sxs-lookup"><span data-stu-id="c9faf-153">As shown in the following screenshot, Charles is part of the Admin group, so he inherits the SurveyAdmin role.</span></span> <span data-ttu-id="c9faf-154">Bob, esetén Phil van még nincs hozzárendelve szerepkör.</span><span class="sxs-lookup"><span data-stu-id="c9faf-154">In the case of Bob, he has not been assigned a role yet.</span></span>

![Felügyeleti csoport tagjai](./images/running-the-app/admin-members.png)


> [!NOTE]
> <span data-ttu-id="c9faf-156">Egy másik módszert is van, az alkalmazás, amelyekhez szerepkörök rendelhetők programozott módon, az Azure AD Graph API-val.</span><span class="sxs-lookup"><span data-stu-id="c9faf-156">An alternative approach is for the application to assign roles programmatically, using the Azure AD Graph API.</span></span> <span data-ttu-id="c9faf-157">Azonban ehhez az alkalmazás beszerzése az ügyfél Active directory írási engedéllyel.</span><span class="sxs-lookup"><span data-stu-id="c9faf-157">However, this requires the application to obtain write permissions for the customer's AD directory.</span></span> <span data-ttu-id="c9faf-158">Ezeket az engedélyeket az alkalmazás végrehajtja kárt számos &mdash; az ügyfél az alkalmazásnak, hogy nem a könyvtár mess delegálását.</span><span class="sxs-lookup"><span data-stu-id="c9faf-158">An application with those permissions could do a lot of mischief &mdash; the customer is trusting the app not to mess up their directory.</span></span> <span data-ttu-id="c9faf-159">Előfordulhat, hogy sok ügyfél hajtja adja meg a hozzáférési szintet.</span><span class="sxs-lookup"><span data-stu-id="c9faf-159">Many customers might be unwilling to grant this level of access.</span></span>
> 

<span data-ttu-id="c9faf-160">**Szerepkör jogcím beolvasása**.</span><span class="sxs-lookup"><span data-stu-id="c9faf-160">**Get role claims**.</span></span> <span data-ttu-id="c9faf-161">Amikor egy felhasználó bejelentkezik, az alkalmazás típusú jogcímet a felhasználó hozzárendelt szerepkör(ök) fogadása `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span><span class="sxs-lookup"><span data-stu-id="c9faf-161">When a user signs in, the application receives the user's assigned role(s) in a claim with type `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span></span>  

<span data-ttu-id="c9faf-162">A felhasználó több szerepkört, vagy nincs szerepkör lehet.</span><span class="sxs-lookup"><span data-stu-id="c9faf-162">A user can have multiple roles, or no role.</span></span> <span data-ttu-id="c9faf-163">Az engedélyezési kód nem érdemes feltételezni a felhasználó rendelkezik-e jogcím pontosan egy szerepkört.</span><span class="sxs-lookup"><span data-stu-id="c9faf-163">In your authorization code, don't assume the user has exactly one role claim.</span></span> <span data-ttu-id="c9faf-164">Ehelyett kód írását, amely ellenőrzi, hogy jelen-e egy adott jogcím értéke:</span><span class="sxs-lookup"><span data-stu-id="c9faf-164">Instead, write code that checks whether a particular claim value is present:</span></span>

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a><span data-ttu-id="c9faf-165">Szerepkörök az Azure AD biztonsági csoportokat használ</span><span class="sxs-lookup"><span data-stu-id="c9faf-165">Roles using Azure AD security groups</span></span>
<span data-ttu-id="c9faf-166">Ezt a módszert használja, a szerepkörök helyettesítik AD biztonsági csoportokat.</span><span class="sxs-lookup"><span data-stu-id="c9faf-166">In this approach, roles are represented as AD security groups.</span></span> <span data-ttu-id="c9faf-167">Az alkalmazás engedélyeket rendeli, a felhasználóknak a biztonsági csoporttagságok alapján.</span><span class="sxs-lookup"><span data-stu-id="c9faf-167">The application assigns permissions to users based on their security group memberships.</span></span>

<span data-ttu-id="c9faf-168">Előnyei:</span><span class="sxs-lookup"><span data-stu-id="c9faf-168">Advantages:</span></span>

* <span data-ttu-id="c9faf-169">Prémium szintű Azure AD nem rendelkező ügyfelek Ez a megközelítés lehetővé teszi, hogy az ügyfél biztonsági csoportok segítségével kezelheti a szerepkör-hozzárendelések.</span><span class="sxs-lookup"><span data-stu-id="c9faf-169">For customers who do not have Azure AD Premium, this approach enables the customer to use security groups to manage role assignments.</span></span>

<span data-ttu-id="c9faf-170">Hátrányait:</span><span class="sxs-lookup"><span data-stu-id="c9faf-170">Disadvantages:</span></span>

* <span data-ttu-id="c9faf-171">Összetettségét.</span><span class="sxs-lookup"><span data-stu-id="c9faf-171">Complexity.</span></span> <span data-ttu-id="c9faf-172">Mivel minden bérlő különböző csoportjogcímek küldi el, az alkalmazás kell nyomon követjük, hogy mely alkalmazás szerepköröket, az egyes bérlők számára megfelelő biztonsági csoportokat.</span><span class="sxs-lookup"><span data-stu-id="c9faf-172">Because every tenant sends different group claims, the app must keep track of which security groups correspond to which application roles, for each tenant.</span></span>
* <span data-ttu-id="c9faf-173">Ha az ügyfél az AD-bérlő eltávolítja az alkalmazást, a biztonsági csoportokat az Active directory maradnak.</span><span class="sxs-lookup"><span data-stu-id="c9faf-173">If the customer removes the application from their AD tenant, the security groups are left in their AD directory.</span></span>

### <a name="implementation"></a><span data-ttu-id="c9faf-174">Megvalósítás</span><span class="sxs-lookup"><span data-stu-id="c9faf-174">Implementation</span></span>
<span data-ttu-id="c9faf-175">Az alkalmazásjegyzékben, állítsa be a `groupMembershipClaims` tulajdonság "a(z)"biztonsági csoporthoz.</span><span class="sxs-lookup"><span data-stu-id="c9faf-175">In the application manifest, set the `groupMembershipClaims` property to "SecurityGroup".</span></span> <span data-ttu-id="c9faf-176">Ez szükséges tagsági csoportjogcímek lekérése az aad-ben.</span><span class="sxs-lookup"><span data-stu-id="c9faf-176">This is needed to get group membership claims from AAD.</span></span>

```json
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

<span data-ttu-id="c9faf-177">Egy új ügyfél iratkozik fel, ha az alkalmazás utasítja az ügyfél a szerepköröket, az alkalmazás számára szükséges biztonsági csoportok létrehozása.</span><span class="sxs-lookup"><span data-stu-id="c9faf-177">When a new customer signs up, the application instructs the customer to create security groups for the roles needed by the application.</span></span> <span data-ttu-id="c9faf-178">Az ügyfél majd be kell írnia a csoportházirend-objektum azonosítók az alkalmazásba.</span><span class="sxs-lookup"><span data-stu-id="c9faf-178">The customer then needs to enter the group object IDs into the application.</span></span> <span data-ttu-id="c9faf-179">Az alkalmazás tárolja ezeket, amely leképezhető csoportazonosítóval alkalmazási szerepköröknek bérlőnként táblázatban.</span><span class="sxs-lookup"><span data-stu-id="c9faf-179">The application stores these in a table that maps group IDs to application roles, per tenant.</span></span>

> [!NOTE]
> <span data-ttu-id="c9faf-180">Másik lehetőségként az alkalmazás létrehozhat a csoportok programozott módon, az Azure AD Graph API-val.</span><span class="sxs-lookup"><span data-stu-id="c9faf-180">Alternatively, the application could create the groups programmatically, using the Azure AD Graph API.</span></span>  <span data-ttu-id="c9faf-181">Ez lehet kevesebb a hibalehetőség hiba.</span><span class="sxs-lookup"><span data-stu-id="c9faf-181">This would be less error prone.</span></span> <span data-ttu-id="c9faf-182">Az alkalmazás a "olvasása és írása az összes csoport" igényel, azonban az ügyfél Active directory engedélyeit.</span><span class="sxs-lookup"><span data-stu-id="c9faf-182">However, it requires the application to obtain "read and write all groups" permissions for the customer's AD directory.</span></span> <span data-ttu-id="c9faf-183">Előfordulhat, hogy sok ügyfél hajtja adja meg a hozzáférési szintet.</span><span class="sxs-lookup"><span data-stu-id="c9faf-183">Many customers might be unwilling to grant this level of access.</span></span>
> 
> 

<span data-ttu-id="c9faf-184">Ha felhasználó jelentkezik be:</span><span class="sxs-lookup"><span data-stu-id="c9faf-184">When a user signs in:</span></span>

1. <span data-ttu-id="c9faf-185">Az alkalmazás a felhasználói csoportok jogcímekként kap.</span><span class="sxs-lookup"><span data-stu-id="c9faf-185">The application receives the user's groups as claims.</span></span> <span data-ttu-id="c9faf-186">Az egyes jogcímek értéke egy csoport Objektumazonosítója.</span><span class="sxs-lookup"><span data-stu-id="c9faf-186">The value of each claim is the object ID of a group.</span></span>
2. <span data-ttu-id="c9faf-187">Az Azure AD a jogkivonat küldött csoportok számának korlátozása.</span><span class="sxs-lookup"><span data-stu-id="c9faf-187">Azure AD limits the number of groups sent in the token.</span></span> <span data-ttu-id="c9faf-188">Ha csoportok száma meghaladja ezt a határt, a Azure AD egy különös "keretét" jogcímet küldi el.</span><span class="sxs-lookup"><span data-stu-id="c9faf-188">If the number of groups exceeds this limit, Azure AD sends a special "overage" claim.</span></span> <span data-ttu-id="c9faf-189">Ha meg adva, hogy a jogcímek, az alkalmazás az Azure AD Graph API minden a tartalmazó csoportok, hogy a felhasználó kell lekérdeznie.</span><span class="sxs-lookup"><span data-stu-id="c9faf-189">If that claim is present, the application must query the Azure AD Graph API to get all of the groups to which that user belongs.</span></span> <span data-ttu-id="c9faf-190">További információkért lásd: [felhőalapú alkalmazások AD-csoportok használata az engedélyezési], a "Csoportok igény túlhasználati" című szakaszban.</span><span class="sxs-lookup"><span data-stu-id="c9faf-190">For details, see [Authorization in Cloud Applications using AD Groups], under the section titled "Groups claim overage".</span></span>
3. <span data-ttu-id="c9faf-191">Az alkalmazás a saját adatbázisában található a megfelelő alkalmazás-szerepkörök hozzárendelése a felhasználóhoz a objektumazonosítók keres.</span><span class="sxs-lookup"><span data-stu-id="c9faf-191">The application looks up the object IDs in its own database, to find the corresponding application roles to assign to the user.</span></span>
4. <span data-ttu-id="c9faf-192">Az alkalmazás egy egyéni jogcím értékét hozzáadja az alkalmazás-szerepkör kifejezze egyszerű felhasználónév.</span><span class="sxs-lookup"><span data-stu-id="c9faf-192">The application adds a custom claim value to the user principal that expresses the application role.</span></span> <span data-ttu-id="c9faf-193">Például: `survey_role` = "SurveyAdmin".</span><span class="sxs-lookup"><span data-stu-id="c9faf-193">For example: `survey_role` = "SurveyAdmin".</span></span>

<span data-ttu-id="c9faf-194">Engedélyezési házirendek használjon egyéni szerepkör jogcím jogcím-nem a csoport.</span><span class="sxs-lookup"><span data-stu-id="c9faf-194">Authorization policies should use the custom role claim, not the group claim.</span></span>

## <a name="roles-using-an-application-role-manager"></a><span data-ttu-id="c9faf-195">Egy alkalmazás-szerepkör kezelővel szerepkörök</span><span class="sxs-lookup"><span data-stu-id="c9faf-195">Roles using an application role manager</span></span>
<span data-ttu-id="c9faf-196">Ezt a módszert alkalmazási szerepköröknek nem tárolja az Azure ad-ben minden.</span><span class="sxs-lookup"><span data-stu-id="c9faf-196">With this approach, application roles are not stored in Azure AD at all.</span></span> <span data-ttu-id="c9faf-197">Ehelyett az alkalmazás tárolja a szerepkör-hozzárendelések az egyes felhasználók a saját DB &mdash; használata esetén például a **RoleManager** osztály ASP.NET-identitásban.</span><span class="sxs-lookup"><span data-stu-id="c9faf-197">Instead, the application stores the role assignments for each user in its own DB &mdash; for example, using the **RoleManager** class in ASP.NET Identity.</span></span>

<span data-ttu-id="c9faf-198">Előnyei:</span><span class="sxs-lookup"><span data-stu-id="c9faf-198">Advantages:</span></span>

* <span data-ttu-id="c9faf-199">Az alkalmazás a szerepkörök és a felhasználó-hozzárendeléseket történő teljes körű vezérléssel rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="c9faf-199">The app has full control over the roles and user assignments.</span></span>

<span data-ttu-id="c9faf-200">Hátrányai:</span><span class="sxs-lookup"><span data-stu-id="c9faf-200">Drawbacks:</span></span>

* <span data-ttu-id="c9faf-201">Összetettebb, nehezebb karbantartása.</span><span class="sxs-lookup"><span data-stu-id="c9faf-201">More complex, harder to maintain.</span></span>
* <span data-ttu-id="c9faf-202">Active Directory biztonsági csoportokat nem használható a szerepkör-hozzárendelések kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="c9faf-202">Cannot use AD security groups to manage role assignments.</span></span>
* <span data-ttu-id="c9faf-203">Felhasználói adatokat tárol az alkalmazás adatbázisban, ahol előbbi beszerezheti a szinkronban a bérlő az Active directory, mivel a felhasználók hozzáadásakor vagy eltávolításakor.</span><span class="sxs-lookup"><span data-stu-id="c9faf-203">Stores user information in the application database, where it can get out of sync with the tenant's AD directory, as users are added or removed.</span></span>   


<span data-ttu-id="c9faf-204">[**Következő**][engedélyezési]</span><span class="sxs-lookup"><span data-stu-id="c9faf-204">[**Next**][authorization]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[engedélyezési]: authorize.md
[authorization]: authorize.md
[a háttér webes API biztonságossá tétele]: web-api.md
[Securing a backend web API]: web-api.md
[alkalmazásjegyzék]: /azure/active-directory/active-directory-application-manifest/
[application manifest]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
