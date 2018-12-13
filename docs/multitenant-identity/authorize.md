---
title: Hitelesítés a több-bérlős alkalmazásokban
description: Engedélyezési végrehajtása egy több-bérlős alkalmazásban
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 8ff2317eb85197ed93e048b6a2d836405436cc17
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307163"
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="f528a-103">Szerepkör- és erőforrás-alapú hitelesítés</span><span class="sxs-lookup"><span data-stu-id="f528a-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="f528a-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="f528a-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="f528a-105">A [megvalósítási hivatkozhat] ASP.NET Core-alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="f528a-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="f528a-106">Ebben a cikkben áttekintjük engedélyezési, két általános megközelítése az engedélyt az ASP.NET Core API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="f528a-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="f528a-107">**Szerepkör-alapú hitelesítést**.</span><span class="sxs-lookup"><span data-stu-id="f528a-107">**Role-based authorization**.</span></span> <span data-ttu-id="f528a-108">A felhasználóhoz hozzárendelt szerepkörök alapján művelet engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="f528a-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="f528a-109">Ha például bizonyos műveleteket rendszergazdai szerepkör van szükség.</span><span class="sxs-lookup"><span data-stu-id="f528a-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="f528a-110">**Erőforrás-alapú hitelesítést**.</span><span class="sxs-lookup"><span data-stu-id="f528a-110">**Resource-based authorization**.</span></span> <span data-ttu-id="f528a-111">Engedélyező a művelet egy adott erőforrás alapján.</span><span class="sxs-lookup"><span data-stu-id="f528a-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="f528a-112">Ha például minden erőforrás van tulajdonosa.</span><span class="sxs-lookup"><span data-stu-id="f528a-112">For example, every resource has an owner.</span></span> <span data-ttu-id="f528a-113">A tulajdonos törölheti az erőforrást; nem végezhető el más felhasználók.</span><span class="sxs-lookup"><span data-stu-id="f528a-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="f528a-114">A tipikus app vegyesen is fogja alkalmazni.</span><span class="sxs-lookup"><span data-stu-id="f528a-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="f528a-115">Ha például egy erőforrás törlése a felhasználónak kell lennie az erőforrás tulajdonosa *vagy* rendszergazda.</span><span class="sxs-lookup"><span data-stu-id="f528a-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="f528a-116">Szerepkör-alapú hitelesítést</span><span class="sxs-lookup"><span data-stu-id="f528a-116">Role-Based Authorization</span></span>
<span data-ttu-id="f528a-117">A [Tailspin Surveys] [ Tailspin] alkalmazás határozza meg a következő szerepkörök:</span><span class="sxs-lookup"><span data-stu-id="f528a-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="f528a-118">Rendszergazda.</span><span class="sxs-lookup"><span data-stu-id="f528a-118">Administrator.</span></span> <span data-ttu-id="f528a-119">Bármely felmérés a bérlőhöz tartozó összes CRUD-műveletek hajthat végre.</span><span class="sxs-lookup"><span data-stu-id="f528a-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="f528a-120">Létrehozó.</span><span class="sxs-lookup"><span data-stu-id="f528a-120">Creator.</span></span> <span data-ttu-id="f528a-121">Hozhat létre új felmérések</span><span class="sxs-lookup"><span data-stu-id="f528a-121">Can create new surveys</span></span>
* <span data-ttu-id="f528a-122">Olvasó.</span><span class="sxs-lookup"><span data-stu-id="f528a-122">Reader.</span></span> <span data-ttu-id="f528a-123">Olvashatja a bérlőhöz tartozó bármely felmérések</span><span class="sxs-lookup"><span data-stu-id="f528a-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="f528a-124">Szerepkörök a alkalmazni *felhasználók* az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="f528a-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="f528a-125">A Surveys alkalmazás egy felhasználó, egy rendszergazda, a létrehozó vagy a reader.</span><span class="sxs-lookup"><span data-stu-id="f528a-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="f528a-126">Bemutatja, hogyan határozhatja meg és kezelheti a szerepköröket, lásd: [Alkalmazási szerepkörök].</span><span class="sxs-lookup"><span data-stu-id="f528a-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="f528a-127">Függetlenül attól, hogyan kezelheti a szerepköröket az engedélyezési kódot fog hasonlítani.</span><span class="sxs-lookup"><span data-stu-id="f528a-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="f528a-128">ASP.NET Core rendelkezik nevű absztrakciós [engedélyezési házirendek][policies].</span><span class="sxs-lookup"><span data-stu-id="f528a-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="f528a-129">Ezzel a funkcióval engedélyezési házirendeket határozhat meg a kódot, és majd alkalmazhatja őket vezérlő műveleteket hajthat végre.</span><span class="sxs-lookup"><span data-stu-id="f528a-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="f528a-130">A szabályzat különválik a vezérlő.</span><span class="sxs-lookup"><span data-stu-id="f528a-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="f528a-131">Szabályzatok létrehozása</span><span class="sxs-lookup"><span data-stu-id="f528a-131">Create policies</span></span>
<span data-ttu-id="f528a-132">Meghatározhat egy olyan szabályzatot, először hozzon létre egy osztályt, amely megvalósítja `IAuthorizationRequirement`.</span><span class="sxs-lookup"><span data-stu-id="f528a-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="f528a-133">A legegyszerűbb származtassa `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="f528a-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="f528a-134">Az a `Handle` metódus, vizsgálja meg a megfelelő jogcím.</span><span class="sxs-lookup"><span data-stu-id="f528a-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="f528a-135">Íme egy példa a Tailspin Surveys-alkalmazás:</span><span class="sxs-lookup"><span data-stu-id="f528a-135">Here is an example from the Tailspin Surveys application:</span></span>

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

<span data-ttu-id="f528a-136">Ez az osztály határozza meg a felhasználót, hogy hozzon létre egy új felmérés vonatkozó követelmény.</span><span class="sxs-lookup"><span data-stu-id="f528a-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="f528a-137">A felhasználónak kell lennie a SurveyAdmin vagy SurveyCreator szerepkörben.</span><span class="sxs-lookup"><span data-stu-id="f528a-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="f528a-138">Az indítási osztályt határoz meg egy nevesített szabályzatot, amely tartalmaz egy vagy több követelménynek.</span><span class="sxs-lookup"><span data-stu-id="f528a-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="f528a-139">Ha több követelmények, a felhasználónak meg kell felelnie *minden* követelmény, hogy engedélyezni kell.</span><span class="sxs-lookup"><span data-stu-id="f528a-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="f528a-140">A következő kódot a két szabályzat határozza meg:</span><span class="sxs-lookup"><span data-stu-id="f528a-140">The following code defines two policies:</span></span>

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

<span data-ttu-id="f528a-141">Ez a kód beállítja a hitelesítési sémát, amely közli az ASP.NET, mely hitelesítési közbenső kell futnia, ha a hitelesítés sikertelen.</span><span class="sxs-lookup"><span data-stu-id="f528a-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="f528a-142">Ebben az esetben azt adja meg cookie-k közbenső hitelesítési szoftver, mert a cookie-k közbenső hitelesítési szoftver is átirányítja a felhasználót egy "Tiltott" lap.</span><span class="sxs-lookup"><span data-stu-id="f528a-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="f528a-143">A tiltott lap helye van megadva a `AccessDeniedPath` első számú megoldás a cookie-k közbenső; lásd: [A közbenső hitelesítési szoftver konfigurálása].</span><span class="sxs-lookup"><span data-stu-id="f528a-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="f528a-144">Vezérlő műveletek engedélyezése</span><span class="sxs-lookup"><span data-stu-id="f528a-144">Authorize controller actions</span></span>
<span data-ttu-id="f528a-145">Végül az MVC-vezérlő művelet engedélyezéséhez, állítsa be ezt a házirendet a `Authorize` attribútum:</span><span class="sxs-lookup"><span data-stu-id="f528a-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="f528a-146">Az ASP.NET korábbi verzióiban, így állíthatja be a **szerepkörök** az attribútum a tulajdonság:</span><span class="sxs-lookup"><span data-stu-id="f528a-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="f528a-147">Ez továbbra is támogatja az ASP.NET Core, de azt hátrányokkal engedélyezési házirendek képest:</span><span class="sxs-lookup"><span data-stu-id="f528a-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="f528a-148">Azt feltételezi, hogy egy adott jogcímtípushoz.</span><span class="sxs-lookup"><span data-stu-id="f528a-148">It assumes a particular claim type.</span></span> <span data-ttu-id="f528a-149">Házirendek bármilyen jogcímtípust kereshet.</span><span class="sxs-lookup"><span data-stu-id="f528a-149">Policies can check for any claim type.</span></span> <span data-ttu-id="f528a-150">Ezek a jogcím olyan típusú.</span><span class="sxs-lookup"><span data-stu-id="f528a-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="f528a-151">A szerepkör neve nem változtatható, az attribútumot.</span><span class="sxs-lookup"><span data-stu-id="f528a-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="f528a-152">Szabályzatok az engedélyezési logika érhető el minden egy helyen tudhatja, megkönnyítve a frissítés vagy a terhelést a konfigurációs beállítások.</span><span class="sxs-lookup"><span data-stu-id="f528a-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="f528a-153">Szabályzatok lehetővé teszik a bonyolult felhasználását engedélyezési döntésekhez (pl. kor > = 21), amelyek nem fejezhetők egyszerű szerepköri tagság szerint.</span><span class="sxs-lookup"><span data-stu-id="f528a-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="f528a-154">Erőforrás-alapú engedélyezési</span><span class="sxs-lookup"><span data-stu-id="f528a-154">Resource based authorization</span></span>
<span data-ttu-id="f528a-155">*Erőforrás-alapú engedélyezési* következik be, a hitelesítést egy adott erőforrás-művelet által érintett függ.</span><span class="sxs-lookup"><span data-stu-id="f528a-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="f528a-156">A Tailspin Surveys alkalmazás minden felmérés van tulajdonosa és nulla-a-többhöz közreműködő.</span><span class="sxs-lookup"><span data-stu-id="f528a-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="f528a-157">A tulajdonos olvasása, frissítése, közzététel és közzétételének visszavonása a kérdőív kitöltését.</span><span class="sxs-lookup"><span data-stu-id="f528a-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="f528a-158">A tulajdonos közreműködők rendelhet a kérdőív kitöltését.</span><span class="sxs-lookup"><span data-stu-id="f528a-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="f528a-159">A közreműködők olvashatja, és frissítse a kérdőív kitöltését.</span><span class="sxs-lookup"><span data-stu-id="f528a-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="f528a-160">Vegye figyelembe, hogy "tulajdonos" és "közreműködő" ne legyenek; alkalmazás-szerepkörök az alkalmazás-adatbázis felmérésekre tárolódnak.</span><span class="sxs-lookup"><span data-stu-id="f528a-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="f528a-161">Annak ellenőrzéséhez, hogy a felhasználók törölhetik a felmérés, például az alkalmazás ellenőrzi, hogy a felhasználó a tulajdonos, a felmérést.</span><span class="sxs-lookup"><span data-stu-id="f528a-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="f528a-162">Az ASP.NET Core, szerint alkalmazza az erőforrás-alapú hitelesítést kapcsolatból származtatott kapcsolatot **AuthorizationHandler** és felülírása a **kezelni** metódust.</span><span class="sxs-lookup"><span data-stu-id="f528a-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="f528a-163">Figyelje meg, hogy ez az osztály van listaobjektum felmérés objektumokhoz.</span><span class="sxs-lookup"><span data-stu-id="f528a-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="f528a-164">Az osztály regisztráljon DI indításakor:</span><span class="sxs-lookup"><span data-stu-id="f528a-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="f528a-165">Hitelesítési ellenőrzések elvégzéséhez, használja a **IAuthorizationService** illesztőt, amely akkor is behelyezése a tartományvezérlőket.</span><span class="sxs-lookup"><span data-stu-id="f528a-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="f528a-166">Az alábbi kód ellenőrzi-e egy felhasználó egy kérdőív olvashat:</span><span class="sxs-lookup"><span data-stu-id="f528a-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="f528a-167">Mivel adjuk át egy `Survey` objektumot, a hívás meghívja a `SurveyAuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="f528a-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="f528a-168">Az engedélyezési kódot jó módszer, hogy a felhasználói szerepkör- és erőforrás-alapú engedélyeket, majd az ellenőrzési összesített állítsa be a kívánt műveletet ellen összesített minden.</span><span class="sxs-lookup"><span data-stu-id="f528a-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="f528a-169">Íme egy példa a Surveys alkalmazással.</span><span class="sxs-lookup"><span data-stu-id="f528a-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="f528a-170">Az alkalmazás több Engedélytípusok határozza meg:</span><span class="sxs-lookup"><span data-stu-id="f528a-170">The application defines several permission types:</span></span>

* <span data-ttu-id="f528a-171">Adminisztratív körzet</span><span class="sxs-lookup"><span data-stu-id="f528a-171">Admin</span></span>
* <span data-ttu-id="f528a-172">Közreműködő</span><span class="sxs-lookup"><span data-stu-id="f528a-172">Contributor</span></span>
* <span data-ttu-id="f528a-173">Létrehozó</span><span class="sxs-lookup"><span data-stu-id="f528a-173">Creator</span></span>
* <span data-ttu-id="f528a-174">Tulajdonos</span><span class="sxs-lookup"><span data-stu-id="f528a-174">Owner</span></span>
* <span data-ttu-id="f528a-175">Olvasó</span><span class="sxs-lookup"><span data-stu-id="f528a-175">Reader</span></span>

<span data-ttu-id="f528a-176">Az alkalmazás is felmérések lehetséges műveletek egy csoportját határozza meg:</span><span class="sxs-lookup"><span data-stu-id="f528a-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="f528a-177">Létrehozás</span><span class="sxs-lookup"><span data-stu-id="f528a-177">Create</span></span>
* <span data-ttu-id="f528a-178">Olvasás</span><span class="sxs-lookup"><span data-stu-id="f528a-178">Read</span></span>
* <span data-ttu-id="f528a-179">Frissítés</span><span class="sxs-lookup"><span data-stu-id="f528a-179">Update</span></span>
* <span data-ttu-id="f528a-180">Törlés</span><span class="sxs-lookup"><span data-stu-id="f528a-180">Delete</span></span>
* <span data-ttu-id="f528a-181">Közzététel</span><span class="sxs-lookup"><span data-stu-id="f528a-181">Publish</span></span>
* <span data-ttu-id="f528a-182">Közzététel visszavonása</span><span class="sxs-lookup"><span data-stu-id="f528a-182">Unpublish</span></span>

<span data-ttu-id="f528a-183">Az alábbi kód létrehoz egy adott felhasználó és a felmérés vonatkozó engedélyek listája.</span><span class="sxs-lookup"><span data-stu-id="f528a-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="f528a-184">Figyelje meg, hogy ez a kód megvizsgálja a felhasználó alkalmazás-szerepkörök, mind a tulajdonosa vagy közreműködője mezőket a kérdőív kitöltését.</span><span class="sxs-lookup"><span data-stu-id="f528a-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

<span data-ttu-id="f528a-185">Egy több-bérlős alkalmazásban gondoskodnia kell arról, hogy engedélyeket nem "nyilvánosságra kerüljenek" egy másik bérlő adatai.</span><span class="sxs-lookup"><span data-stu-id="f528a-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="f528a-186">A Surveys alkalmazás közreműködői engedélye engedélyezett bérlők között &mdash; valaki más bérlők közreműködője is hozzárendelhet.</span><span class="sxs-lookup"><span data-stu-id="f528a-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contributor.</span></span> <span data-ttu-id="f528a-187">Annak a felhasználónak a bérlőhöz tartozó erőforrásokhoz a többi Engedélytípusok korlátozódnak.</span><span class="sxs-lookup"><span data-stu-id="f528a-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="f528a-188">Ezt a követelményt kényszerítéséhez a kód ellenőrzi a bérlő azonosítója az engedély megadása előtt.</span><span class="sxs-lookup"><span data-stu-id="f528a-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="f528a-189">(A `TenantId` módon rendeli hozzá a felmérés létrehozásakor mezőben.)</span><span class="sxs-lookup"><span data-stu-id="f528a-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="f528a-190">A következő lépés, hogy ellenőrizze a műveletet (olvasás, update, delete stb.) ellen az engedélyeket.</span><span class="sxs-lookup"><span data-stu-id="f528a-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="f528a-191">A Surveys alkalmazás ebben a lépésben egy keresési táblázat a functions használatával valósítja meg:</span><span class="sxs-lookup"><span data-stu-id="f528a-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

<span data-ttu-id="f528a-192">[**Tovább**][web-api]</span><span class="sxs-lookup"><span data-stu-id="f528a-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[Alkalmazási szerepkörök]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[megvalósítási hivatkozhat]: tailspin.md
[reference implementation]: tailspin.md
[A közbenső hitelesítési szoftver konfigurálása]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
