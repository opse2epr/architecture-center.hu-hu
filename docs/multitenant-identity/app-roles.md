---
title: Alkalmazás-szerepkörök
description: 'Hogyan végezheti el a hitelesítést: az alkalmazás-szerepkörök.'
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: 04749bff820132e40f3cbb5195bf65648ab39ab3
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112532"
---
# <a name="application-roles"></a><span data-ttu-id="0e38e-103">Alkalmazás-szerepkörök</span><span class="sxs-lookup"><span data-stu-id="0e38e-103">Application roles</span></span>

<span data-ttu-id="0e38e-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="0e38e-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="0e38e-105">Alkalmazás-szerepkörök segítségével engedélyek hozzárendelése a felhasználókhoz.</span><span class="sxs-lookup"><span data-stu-id="0e38e-105">Application roles are used to assign permissions to users.</span></span> <span data-ttu-id="0e38e-106">Ha például a [Tailspin Surveys] [ tailspin] alkalmazás határozza meg a következő szerepkörök:</span><span class="sxs-lookup"><span data-stu-id="0e38e-106">For example, the [Tailspin Surveys][tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="0e38e-107">Rendszergazda.</span><span class="sxs-lookup"><span data-stu-id="0e38e-107">Administrator.</span></span> <span data-ttu-id="0e38e-108">Bármely felmérés a bérlőhöz tartozó összes CRUD-műveletek hajthat végre.</span><span class="sxs-lookup"><span data-stu-id="0e38e-108">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="0e38e-109">Létrehozó.</span><span class="sxs-lookup"><span data-stu-id="0e38e-109">Creator.</span></span> <span data-ttu-id="0e38e-110">Új felmérések hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="0e38e-110">Can create new surveys.</span></span>
* <span data-ttu-id="0e38e-111">Olvasó.</span><span class="sxs-lookup"><span data-stu-id="0e38e-111">Reader.</span></span> <span data-ttu-id="0e38e-112">A bérlőhöz tartozó bármely felmérések olvashatja.</span><span class="sxs-lookup"><span data-stu-id="0e38e-112">Can read any surveys that belong to that tenant.</span></span>

<span data-ttu-id="0e38e-113">Láthatja, hogy szerepkörök végső soron első fordítja engedélyek során [engedélyezési].</span><span class="sxs-lookup"><span data-stu-id="0e38e-113">You can see that roles ultimately get translated into permissions, during [authorization].</span></span> <span data-ttu-id="0e38e-114">De az első kérdés, hogy miképpen lehet hozzárendelni és felügyelni a szerepkört.</span><span class="sxs-lookup"><span data-stu-id="0e38e-114">But the first question is how to assign and manage roles.</span></span> <span data-ttu-id="0e38e-115">Azt észleltük, hogy három fő beállítások:</span><span class="sxs-lookup"><span data-stu-id="0e38e-115">We identified three main options:</span></span>

* [<span data-ttu-id="0e38e-116">Az Azure AD alkalmazás-szerepkörök</span><span class="sxs-lookup"><span data-stu-id="0e38e-116">Azure AD App Roles</span></span>](#roles-using-azure-ad-app-roles)
* [<span data-ttu-id="0e38e-117">Az Azure AD biztonsági csoportok</span><span class="sxs-lookup"><span data-stu-id="0e38e-117">Azure AD security groups</span></span>](#roles-using-azure-ad-security-groups)
* <span data-ttu-id="0e38e-118">[Alkalmazás szerepkör-kezelő](#roles-using-an-application-role-manager).</span><span class="sxs-lookup"><span data-stu-id="0e38e-118">[Application role manager](#roles-using-an-application-role-manager).</span></span>

## <a name="roles-using-azure-ad-app-roles"></a><span data-ttu-id="0e38e-119">Szerepkörök az Azure AD alkalmazás-szerepkörök használatával</span><span class="sxs-lookup"><span data-stu-id="0e38e-119">Roles using Azure AD App Roles</span></span>

<span data-ttu-id="0e38e-120">Ez a megközelítés a Tailspin Surveys-alkalmazás is használt.</span><span class="sxs-lookup"><span data-stu-id="0e38e-120">This is the approach that we used in the Tailspin Surveys app.</span></span>

<span data-ttu-id="0e38e-121">Ez a megközelítés az SaaS-szolgáltató határozza meg az alkalmazás-szerepkörök hozzáadását az alkalmazásjegyzékben.</span><span class="sxs-lookup"><span data-stu-id="0e38e-121">In this approach, The SaaS provider defines the application roles by adding them to the application manifest.</span></span> <span data-ttu-id="0e38e-122">Miután egy ügyfél regisztrál, az ügyfél az Active Directory-rendszergazda rendeli hozzá a felhasználókat a szerepkörökhöz.</span><span class="sxs-lookup"><span data-stu-id="0e38e-122">After a customer signs up, an admin for the customer's AD directory assigns users to the roles.</span></span> <span data-ttu-id="0e38e-123">Ha egy felhasználó bejelentkezik, a felhasználó hozzárendelt szerepkörök jogcímekként küld.</span><span class="sxs-lookup"><span data-stu-id="0e38e-123">When a user signs in, the user's assigned roles are sent as claims.</span></span>

> [!NOTE]
> <span data-ttu-id="0e38e-124">Ha az ügyfél rendelkezik prémium szintű Azure AD, a rendszergazda hozzárendelheti egy biztonsági csoportot egy szerepkörhöz, és a csoport tagjai öröklik az alkalmazás-szerepkör.</span><span class="sxs-lookup"><span data-stu-id="0e38e-124">If the customer has Azure AD Premium, the admin can assign a security group to a role, and members of the group will inherit the app role.</span></span> <span data-ttu-id="0e38e-125">Ennek oka egyszerűen kezelheti a szerepköröket, a csoport tulajdonosának nem kell lennie a rendszergazda AD.</span><span class="sxs-lookup"><span data-stu-id="0e38e-125">This is a convenient way to manage roles, because the group owner doesn't need to be an AD admin.</span></span>

<span data-ttu-id="0e38e-126">Ez a módszer előnyei:</span><span class="sxs-lookup"><span data-stu-id="0e38e-126">Advantages of this approach:</span></span>

* <span data-ttu-id="0e38e-127">Egyszerű programozási modellen alapuló.</span><span class="sxs-lookup"><span data-stu-id="0e38e-127">Simple programming model.</span></span>
* <span data-ttu-id="0e38e-128">Szerepkörök kifejezetten az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="0e38e-128">Roles are specific to the application.</span></span> <span data-ttu-id="0e38e-129">A szerepkör jogcím egy alkalmazás nem kap egy másik alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="0e38e-129">The role claims for one application are not sent to another application.</span></span>
* <span data-ttu-id="0e38e-130">Ha az ügyfél eltávolítja az alkalmazást a saját AD-bérlőjében, azonnal nyissa meg a szerepköröket.</span><span class="sxs-lookup"><span data-stu-id="0e38e-130">If the customer removes the application from their AD tenant, the roles go away.</span></span>
* <span data-ttu-id="0e38e-131">Az alkalmazás nem kell semmilyen további Active Directory-engedélyek nem a felhasználói profil olvasása.</span><span class="sxs-lookup"><span data-stu-id="0e38e-131">The application doesn't need any extra Active Directory permissions, other than reading the user's profile.</span></span>

<span data-ttu-id="0e38e-132">Hátrányai:</span><span class="sxs-lookup"><span data-stu-id="0e38e-132">Drawbacks:</span></span>

* <span data-ttu-id="0e38e-133">Prémium szintű Azure AD nem rendelkező ügyfelek biztonsági csoportok nem rendelhető szerepköröket.</span><span class="sxs-lookup"><span data-stu-id="0e38e-133">Customers without Azure AD Premium cannot assign security groups to roles.</span></span> <span data-ttu-id="0e38e-134">Ezen ügyfelek számára egy AD-rendszergazda felhasználó összes hozzárendelést kell elvégezni.</span><span class="sxs-lookup"><span data-stu-id="0e38e-134">For these customers, all user assignments must be done by an AD administrator.</span></span>
* <span data-ttu-id="0e38e-135">Egy háttéralkalmazás webes API-t, amely a webalkalmazást ügynökellenőrzést, ha a webes alkalmazás szerepkör-hozzárendeléseit a webes API-k nem vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="0e38e-135">If you have a backend web API, which is separate from the web app, then role assignments for the web app don't apply to the web API.</span></span> <span data-ttu-id="0e38e-136">Ezen a ponton További ismertetéséhez lásd: [egy háttéralkalmazás webes API biztonságossá tétele].</span><span class="sxs-lookup"><span data-stu-id="0e38e-136">For more discussion of this point, see [Securing a backend web API].</span></span>

### <a name="implementation"></a><span data-ttu-id="0e38e-137">Megvalósítás</span><span class="sxs-lookup"><span data-stu-id="0e38e-137">Implementation</span></span>

<span data-ttu-id="0e38e-138">**A szerepkörök meghatározása.**</span><span class="sxs-lookup"><span data-stu-id="0e38e-138">**Define the roles.**</span></span> <span data-ttu-id="0e38e-139">Az SaaS-szolgáltatónak az alkalmazás-szerepkörök deklarálja a [alkalmazásjegyzék].</span><span class="sxs-lookup"><span data-stu-id="0e38e-139">The SaaS provider declares the app roles in the [application manifest].</span></span> <span data-ttu-id="0e38e-140">Ha például itt látható a jegyzékfájl bejegyzést a Surveys alkalmazás:</span><span class="sxs-lookup"><span data-stu-id="0e38e-140">For example, here is the manifest entry for the Surveys app:</span></span>

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

<span data-ttu-id="0e38e-141">A `value` tulajdonság a szerepkör jogcím szerepel.</span><span class="sxs-lookup"><span data-stu-id="0e38e-141">The `value`  property appears in the role claim.</span></span> <span data-ttu-id="0e38e-142">A `id` tulajdonság a megadott szerepkör egyedi azonosítója.</span><span class="sxs-lookup"><span data-stu-id="0e38e-142">The `id` property is the unique identifier for the defined role.</span></span> <span data-ttu-id="0e38e-143">Mindig hozzon létre egy új GUID azonosítót `id`.</span><span class="sxs-lookup"><span data-stu-id="0e38e-143">Always generate a new GUID value for `id`.</span></span>

<span data-ttu-id="0e38e-144">**Felhasználók hozzárendelése**.</span><span class="sxs-lookup"><span data-stu-id="0e38e-144">**Assign users**.</span></span> <span data-ttu-id="0e38e-145">Amikor új vevő regisztrál, az alkalmazás regisztrálva van az ügyfél AD-bérlőben.</span><span class="sxs-lookup"><span data-stu-id="0e38e-145">When a new customer signs up, the application is registered in the customer's AD tenant.</span></span> <span data-ttu-id="0e38e-146">Ezen a ponton egy AD rendszergazdai a bérlőhöz tartozó felhasználókat rendelhet szerepköröket.</span><span class="sxs-lookup"><span data-stu-id="0e38e-146">At this point, an AD admin for that tenant can assign users to roles.</span></span>

> [!NOTE]
> <span data-ttu-id="0e38e-147">Korábban feljegyzett ügyfelek az Azure AD Premium szolgáltatással is hozzárendelhetők biztonsági csoportok, szerepkörök.</span><span class="sxs-lookup"><span data-stu-id="0e38e-147">As noted earlier, customers with Azure AD Premium can also assign security groups to roles.</span></span>

<span data-ttu-id="0e38e-148">Az alábbi képernyőképen az Azure Portalon látható felhasználók és csoportok a felmérés alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="0e38e-148">The following screenshot from the Azure portal shows users and groups for the Survey application.</span></span> <span data-ttu-id="0e38e-149">Rendszergazdai és a szerző csoportjai, illetve SurveyAdmin és SurveyCreator szerepkörökhöz hozzárendelt.</span><span class="sxs-lookup"><span data-stu-id="0e38e-149">Admin and Creator are groups, assigned to SurveyAdmin and SurveyCreator roles respectively.</span></span> <span data-ttu-id="0e38e-150">Alice az egy felhasználó, aki közvetlenül az SurveyAdmin szerepkör lett hozzárendelve.</span><span class="sxs-lookup"><span data-stu-id="0e38e-150">Alice is a user who was assigned directly to the SurveyAdmin role.</span></span> <span data-ttu-id="0e38e-151">Bob és Charles nem szerepkörhöz közvetlenül hozzárendelt felhasználók közül ez.</span><span class="sxs-lookup"><span data-stu-id="0e38e-151">Bob and Charles are users that have not been directly assigned to a role.</span></span>

![Felhasználók és csoportok](./images/running-the-app/users-and-groups.png)

<span data-ttu-id="0e38e-153">Ahogy az az alábbi képernyőfelvételen is látható, Charles a felügyeleti csoport részét képezi, így ő örökli a SurveyAdmin szerepkör.</span><span class="sxs-lookup"><span data-stu-id="0e38e-153">As shown in the following screenshot, Charles is part of the Admin group, so he inherits the SurveyAdmin role.</span></span> <span data-ttu-id="0e38e-154">Bob, esetén Phil van még nincs hozzárendelve szerepkör.</span><span class="sxs-lookup"><span data-stu-id="0e38e-154">In the case of Bob, he has not been assigned a role yet.</span></span>

![Felügyeleti csoport tagjai](./images/running-the-app/admin-members.png)

> [!NOTE]
> <span data-ttu-id="0e38e-156">Egy alternatív módszer az alkalmazás rendelhet szerepköröket programozott módon, az Azure AD Graph API használatával is.</span><span class="sxs-lookup"><span data-stu-id="0e38e-156">An alternative approach is for the application to assign roles programmatically, using the Azure AD Graph API.</span></span> <span data-ttu-id="0e38e-157">Azonban ez csak írási engedéllyel az ügyfél AD-címtár beszerzése az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="0e38e-157">However, this requires the application to obtain write permissions for the customer's AD directory.</span></span> <span data-ttu-id="0e38e-158">Ezeket az engedélyeket az alkalmazás végrehajtja kárt rengeteg &mdash; az ügyfél nem az alkalmazás saját könyvtár elront delegálását.</span><span class="sxs-lookup"><span data-stu-id="0e38e-158">An application with those permissions could do a lot of mischief &mdash; the customer is trusting the app not to mess up their directory.</span></span> <span data-ttu-id="0e38e-159">Előfordulhat, hogy számos ügyfelünk nincs ilyen hozzáférési szint megadását.</span><span class="sxs-lookup"><span data-stu-id="0e38e-159">Many customers might be unwilling to grant this level of access.</span></span>

<span data-ttu-id="0e38e-160">**Első szerepkörjogcímeken**.</span><span class="sxs-lookup"><span data-stu-id="0e38e-160">**Get role claims**.</span></span> <span data-ttu-id="0e38e-161">Amikor egy felhasználó bejelentkezik, az alkalmazás fogad a felhasználó hozzárendelt szerepkör egy jogcím típusú `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span><span class="sxs-lookup"><span data-stu-id="0e38e-161">When a user signs in, the application receives the user's assigned role(s) in a claim with type `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.</span></span>

<span data-ttu-id="0e38e-162">Egy felhasználó több szerepkört, vagy nincs szerepkör rendelkezhetnek.</span><span class="sxs-lookup"><span data-stu-id="0e38e-162">A user can have multiple roles, or no role.</span></span> <span data-ttu-id="0e38e-163">Az engedélyezési kód nem érdemes feltételezni a felhasználó rendelkezik pontosan egy szerepkör jogcím.</span><span class="sxs-lookup"><span data-stu-id="0e38e-163">In your authorization code, don't assume the user has exactly one role claim.</span></span> <span data-ttu-id="0e38e-164">Ehelyett kód írása, amely ellenőrzi, hogy jelen-e egy adott jogcím értéke:</span><span class="sxs-lookup"><span data-stu-id="0e38e-164">Instead, write code that checks whether a particular claim value is present:</span></span>

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a><span data-ttu-id="0e38e-165">Szerepkörök az Azure AD biztonsági csoportok használata</span><span class="sxs-lookup"><span data-stu-id="0e38e-165">Roles using Azure AD security groups</span></span>

<span data-ttu-id="0e38e-166">Ebben a megközelítésben AD biztonsági csoportok, szerepkörök jelennek meg.</span><span class="sxs-lookup"><span data-stu-id="0e38e-166">In this approach, roles are represented as AD security groups.</span></span> <span data-ttu-id="0e38e-167">Az alkalmazás a felhasználók számára a biztonsági csoporttagságok alapján hozzárendeli az engedélyeket.</span><span class="sxs-lookup"><span data-stu-id="0e38e-167">The application assigns permissions to users based on their security group memberships.</span></span>

<span data-ttu-id="0e38e-168">Előnyök:</span><span class="sxs-lookup"><span data-stu-id="0e38e-168">Advantages:</span></span>

* <span data-ttu-id="0e38e-169">Azok a vásárlóknak, akik nem rendelkeznek Azure AD Premium szolgáltatással Ez a megközelítés lehetővé teszi a biztonsági csoportok használata a szerepkör-hozzárendelések kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="0e38e-169">For customers who do not have Azure AD Premium, this approach enables the customer to use security groups to manage role assignments.</span></span>

<span data-ttu-id="0e38e-170">Hátrányait:</span><span class="sxs-lookup"><span data-stu-id="0e38e-170">Disadvantages:</span></span>

* <span data-ttu-id="0e38e-171">Összetettségét.</span><span class="sxs-lookup"><span data-stu-id="0e38e-171">Complexity.</span></span> <span data-ttu-id="0e38e-172">Minden bérlőhöz küld a jogcímeket másik csoport, mert az alkalmazás kell nyomon követheti, amely az egyes bérlők számára mely alkalmazás-szerepkörök, biztonsági csoportok felel meg.</span><span class="sxs-lookup"><span data-stu-id="0e38e-172">Because every tenant sends different group claims, the app must keep track of which security groups correspond to which application roles, for each tenant.</span></span>
* <span data-ttu-id="0e38e-173">Ha az ügyfél eltávolítja az alkalmazást a saját AD-bérlőjében, a biztonsági csoportokat az AD-címtárban maradnak.</span><span class="sxs-lookup"><span data-stu-id="0e38e-173">If the customer removes the application from their AD tenant, the security groups are left in their AD directory.</span></span>

<!-- markdownlint-disable MD024 -->

### <a name="implementation"></a><span data-ttu-id="0e38e-174">Megvalósítás</span><span class="sxs-lookup"><span data-stu-id="0e38e-174">Implementation</span></span>

<!-- markdownlint-enable MD024 -->

<span data-ttu-id="0e38e-175">Az alkalmazásjegyzékben, állítsa be a `groupMembershipClaims` "SecurityGroup" tulajdonságot.</span><span class="sxs-lookup"><span data-stu-id="0e38e-175">In the application manifest, set the `groupMembershipClaims` property to "SecurityGroup".</span></span> <span data-ttu-id="0e38e-176">Erre azért van szükség, csoport tagsági jogcímek beolvasni az aad-ből.</span><span class="sxs-lookup"><span data-stu-id="0e38e-176">This is needed to get group membership claims from AAD.</span></span>

```json
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

<span data-ttu-id="0e38e-177">Amikor új vevő regisztrál, az alkalmazás utasítja az ügyfél a szerepköröket, az alkalmazás számára szükséges biztonsági csoportok létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="0e38e-177">When a new customer signs up, the application instructs the customer to create security groups for the roles needed by the application.</span></span> <span data-ttu-id="0e38e-178">Az ügyfél ezután meg kell adnia a csoportházirend-objektum azonosítóját az alkalmazás igényeinek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="0e38e-178">The customer then needs to enter the group object IDs into the application.</span></span> <span data-ttu-id="0e38e-179">Az alkalmazás tárolja ezeket egy táblában, amely leképezi a csoportazonosítók alkalmazás-szerepkörökhöz bérlőnként.</span><span class="sxs-lookup"><span data-stu-id="0e38e-179">The application stores these in a table that maps group IDs to application roles, per tenant.</span></span>

> [!NOTE]
> <span data-ttu-id="0e38e-180">Azt is megteheti az alkalmazás sikerült létrehozni a csoportok programozott módon, az Azure AD Graph API használatával.</span><span class="sxs-lookup"><span data-stu-id="0e38e-180">Alternatively, the application could create the groups programmatically, using the Azure AD Graph API.</span></span>  <span data-ttu-id="0e38e-181">Ez akkor lehet kisebb a hibalehetőségeket rejt magában.</span><span class="sxs-lookup"><span data-stu-id="0e38e-181">This would be less error prone.</span></span> <span data-ttu-id="0e38e-182">Azonban az alkalmazás a "olvasása és írása az összes csoport" szükséges engedélyeket az ügyfél AD-címtárhoz.</span><span class="sxs-lookup"><span data-stu-id="0e38e-182">However, it requires the application to obtain "read and write all groups" permissions for the customer's AD directory.</span></span> <span data-ttu-id="0e38e-183">Előfordulhat, hogy számos ügyfelünk nincs ilyen hozzáférési szint megadását.</span><span class="sxs-lookup"><span data-stu-id="0e38e-183">Many customers might be unwilling to grant this level of access.</span></span>

<span data-ttu-id="0e38e-184">Amikor egy felhasználó jelentkezik be:</span><span class="sxs-lookup"><span data-stu-id="0e38e-184">When a user signs in:</span></span>

1. <span data-ttu-id="0e38e-185">Az alkalmazás fogad a felhasználói csoportok jogcímekként.</span><span class="sxs-lookup"><span data-stu-id="0e38e-185">The application receives the user's groups as claims.</span></span> <span data-ttu-id="0e38e-186">Minden egyes jogcím értéke a csoport Objektumazonosítóját.</span><span class="sxs-lookup"><span data-stu-id="0e38e-186">The value of each claim is the object ID of a group.</span></span>
2. <span data-ttu-id="0e38e-187">Azure ad-ben a jogkivonatban elküldött csoportok számát korlátozza.</span><span class="sxs-lookup"><span data-stu-id="0e38e-187">Azure AD limits the number of groups sent in the token.</span></span> <span data-ttu-id="0e38e-188">Csoportok száma meghaladja ezt a korlátot, ha az Azure AD elküldi egy speciális "kereten túli" jogcímet.</span><span class="sxs-lookup"><span data-stu-id="0e38e-188">If the number of groups exceeds this limit, Azure AD sends a special "overage" claim.</span></span> <span data-ttu-id="0e38e-189">Ha a jogcím megtalálható, az alkalmazás az Azure AD Graph API-t, hogy a felhasználó csoportjának összes kell lekérdeznie.</span><span class="sxs-lookup"><span data-stu-id="0e38e-189">If that claim is present, the application must query the Azure AD Graph API to get all of the groups to which that user belongs.</span></span> <span data-ttu-id="0e38e-190">További információkért lásd: [az AD-csoportok használata a felhőalapú alkalmazások engedélyezési], a "Csoportok jogcím kerettúllépés" című szakaszában.</span><span class="sxs-lookup"><span data-stu-id="0e38e-190">For details, see [Authorization in Cloud Applications using AD Groups], under the section titled "Groups claim overage".</span></span>
3. <span data-ttu-id="0e38e-191">Az alkalmazás megkeresi az objektumazonosítók a saját adatbázisban található a megfelelő alkalmazás-szerepkörök hozzárendelése a felhasználóhoz.</span><span class="sxs-lookup"><span data-stu-id="0e38e-191">The application looks up the object IDs in its own database, to find the corresponding application roles to assign to the user.</span></span>
4. <span data-ttu-id="0e38e-192">Az alkalmazás egy egyéni jogcím értékét hozzáadja az egyszerű felhasználónév, amely kifejezi az alkalmazás-szerepkör.</span><span class="sxs-lookup"><span data-stu-id="0e38e-192">The application adds a custom claim value to the user principal that expresses the application role.</span></span> <span data-ttu-id="0e38e-193">Például: `survey_role` = "SurveyAdmin".</span><span class="sxs-lookup"><span data-stu-id="0e38e-193">For example: `survey_role` = "SurveyAdmin".</span></span>

<span data-ttu-id="0e38e-194">Engedélyezési házirendeket kell használnia az egyéni szerepkör jogcím nem a csoport jogcím.</span><span class="sxs-lookup"><span data-stu-id="0e38e-194">Authorization policies should use the custom role claim, not the group claim.</span></span>

## <a name="roles-using-an-application-role-manager"></a><span data-ttu-id="0e38e-195">Egy alkalmazás szerepkörkezelővel szerepkörök</span><span class="sxs-lookup"><span data-stu-id="0e38e-195">Roles using an application role manager</span></span>

<span data-ttu-id="0e38e-196">Ezt a módszert használja alkalmazás-szerepkörök nem tárolja az Azure ad-ben minden.</span><span class="sxs-lookup"><span data-stu-id="0e38e-196">With this approach, application roles are not stored in Azure AD at all.</span></span> <span data-ttu-id="0e38e-197">Ehelyett az alkalmazás tárolja a szerepkör-hozzárendeléseket az egyes felhasználók a saját DB &mdash; használata esetén például a **RoleManager** ASP.NET identitás osztályában.</span><span class="sxs-lookup"><span data-stu-id="0e38e-197">Instead, the application stores the role assignments for each user in its own DB &mdash; for example, using the **RoleManager** class in ASP.NET Identity.</span></span>

<span data-ttu-id="0e38e-198">Előnyök:</span><span class="sxs-lookup"><span data-stu-id="0e38e-198">Advantages:</span></span>

* <span data-ttu-id="0e38e-199">A szerepkörök és -hozzárendelések felhasználói teljes hozzáféréssel rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="0e38e-199">The app has full control over the roles and user assignments.</span></span>

<span data-ttu-id="0e38e-200">Hátrányai:</span><span class="sxs-lookup"><span data-stu-id="0e38e-200">Drawbacks:</span></span>

* <span data-ttu-id="0e38e-201">Összetettebb, nehezebben karbantartása.</span><span class="sxs-lookup"><span data-stu-id="0e38e-201">More complex, harder to maintain.</span></span>
* <span data-ttu-id="0e38e-202">AD biztonsági csoportok nem használható a szerepkör-hozzárendelések kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="0e38e-202">Cannot use AD security groups to manage role assignments.</span></span>
* <span data-ttu-id="0e38e-203">Az alkalmazás-adatbázis, honnan azt be nincs szinkronban a bérlő az Active directory, ahogy a felhasználók hozzáadásakor vagy eltávolításakor a felhasználói adatokat tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="0e38e-203">Stores user information in the application database, where it can get out of sync with the tenant's AD directory, as users are added or removed.</span></span>

<span data-ttu-id="0e38e-204">[**Tovább**][engedélyezési]</span><span class="sxs-lookup"><span data-stu-id="0e38e-204">[**Next**][authorization]</span></span>

<!-- links -->

[tailspin]: tailspin.md
[Engedélyezési]: authorize.md
[authorization]: authorize.md
[Egy háttéralkalmazás webes API biztonságossá tétele]: web-api.md
[Securing a backend web API]: web-api.md
[Alkalmazásjegyzék]: /azure/active-directory/active-directory-application-manifest/
[application manifest]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
