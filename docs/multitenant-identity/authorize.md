---
title: "A több-bérlős alkalmazások engedélyezési"
description: "Egy több-bérlős alkalmazásban engedélyezési végrehajtása"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 86c308d21f19bb3ac2a4a2240a9a03a504de5cf4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="9e703-103">Szerepkör- és erőforrás-alapú engedélyezési</span><span class="sxs-lookup"><span data-stu-id="9e703-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="9e703-104">[![GitHub](../_images/github.png) példakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="9e703-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="9e703-105">A [megvalósítása hivatkozhat] ASP.NET Core-alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="9e703-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="9e703-106">Ebben a cikkben megnézzük, engedélyezési, két általános módszer az engedélyt az ASP.NET Core megadott API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="9e703-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="9e703-107">**Szerepkör-alapú engedélyezési**.</span><span class="sxs-lookup"><span data-stu-id="9e703-107">**Role-based authorization**.</span></span> <span data-ttu-id="9e703-108">A felhasználóhoz rendelt szerepkörök alapján műveletet engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="9e703-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="9e703-109">Például bizonyos műveleteket kell egy rendszergazdai szerepkört.</span><span class="sxs-lookup"><span data-stu-id="9e703-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="9e703-110">**Erőforrás-alapú engedélyezési**.</span><span class="sxs-lookup"><span data-stu-id="9e703-110">**Resource-based authorization**.</span></span> <span data-ttu-id="9e703-111">Egy adott erőforrás alapján művelet engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="9e703-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="9e703-112">Például minden erőforrás van tulajdonosa.</span><span class="sxs-lookup"><span data-stu-id="9e703-112">For example, every resource has an owner.</span></span> <span data-ttu-id="9e703-113">A tulajdonos törlése az erőforrás; más felhasználók nem.</span><span class="sxs-lookup"><span data-stu-id="9e703-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="9e703-114">A tipikus app vegyesen is megfelelő.</span><span class="sxs-lookup"><span data-stu-id="9e703-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="9e703-115">Például egy erőforrás törlése, a felhasználónak kell lennie az erőforrás tulajdonosa *vagy* egy rendszergazdával.</span><span class="sxs-lookup"><span data-stu-id="9e703-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="9e703-116">Szerepköralapú hitelesítés</span><span class="sxs-lookup"><span data-stu-id="9e703-116">Role-Based Authorization</span></span>
<span data-ttu-id="9e703-117">A [Dejójáték felmérések] [ Tailspin] alkalmazás határozza meg a következő szerepkörök:</span><span class="sxs-lookup"><span data-stu-id="9e703-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="9e703-118">Rendszergazda.</span><span class="sxs-lookup"><span data-stu-id="9e703-118">Administrator.</span></span> <span data-ttu-id="9e703-119">Minden CRUD-műveleteknek a bármely felmérés, amely arra a bérlőre tartozik hajthat végre.</span><span class="sxs-lookup"><span data-stu-id="9e703-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="9e703-120">Létrehozó.</span><span class="sxs-lookup"><span data-stu-id="9e703-120">Creator.</span></span> <span data-ttu-id="9e703-121">Új felmérések hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="9e703-121">Can create new surveys</span></span>
* <span data-ttu-id="9e703-122">Olvasó.</span><span class="sxs-lookup"><span data-stu-id="9e703-122">Reader.</span></span> <span data-ttu-id="9e703-123">Bármely felmérések arra a bérlőre tartozó olvashatja</span><span class="sxs-lookup"><span data-stu-id="9e703-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="9e703-124">Szerepkörök alkalmazása *felhasználók* az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="9e703-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="9e703-125">Egy felhasználó felmérések alkalmazás, egy rendszergazda, a létrehozó vagy az olvasó.</span><span class="sxs-lookup"><span data-stu-id="9e703-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="9e703-126">Bemutatja, hogyan határozhatja meg és kezelheti a szerepkörök tárgyalását lásd: [alkalmazási szerepköröknek].</span><span class="sxs-lookup"><span data-stu-id="9e703-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="9e703-127">Hogyan kezelheti a szerepkörök, függetlenül az engedélyezési kód alábbihoz hasonlóan fog megjelenni.</span><span class="sxs-lookup"><span data-stu-id="9e703-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="9e703-128">Az ASP.NET Core rendelkezik nevű absztrakciós [engedélyezési házirendek][policies].</span><span class="sxs-lookup"><span data-stu-id="9e703-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="9e703-129">Ezzel a szolgáltatással engedélyezési házirendek definiálása a kódban és a tartományvezérlő műveletek majd alkalmazza azokat a házirendeket.</span><span class="sxs-lookup"><span data-stu-id="9e703-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="9e703-130">A házirendet a rendszer leválasztja a tartományvezérlő.</span><span class="sxs-lookup"><span data-stu-id="9e703-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="9e703-131">Házirendek létrehozása</span><span class="sxs-lookup"><span data-stu-id="9e703-131">Create policies</span></span>
<span data-ttu-id="9e703-132">Adja meg a házirendet, először létre kell hoznia egy osztály, amely megvalósítja az `IAuthorizationRequirement`.</span><span class="sxs-lookup"><span data-stu-id="9e703-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="9e703-133">A legegyszerűbb kapcsolattípusokból származhatnak, amelyek `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="9e703-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="9e703-134">Az a `Handle` metódus, vizsgálja meg a megfelelő jogcím(ek).</span><span class="sxs-lookup"><span data-stu-id="9e703-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="9e703-135">Íme egy példa a Dejójáték felmérések alkalmazásból:</span><span class="sxs-lookup"><span data-stu-id="9e703-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="9e703-136">Ez az osztály a felhasználót, hogy hozzon létre egy új felmérés kapcsolatos követelmények meghatározása.</span><span class="sxs-lookup"><span data-stu-id="9e703-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="9e703-137">A felhasználó a SurveyAdmin vagy SurveyCreator szerepkörrel kell.</span><span class="sxs-lookup"><span data-stu-id="9e703-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="9e703-138">Az indítási osztály, amely tartalmaz egy vagy több követelmények elnevezett házirend definiálásához.</span><span class="sxs-lookup"><span data-stu-id="9e703-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="9e703-139">Ha több követelmények, a felhasználónak meg kell felelnie *minden* kell követelmény.</span><span class="sxs-lookup"><span data-stu-id="9e703-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="9e703-140">Az alábbi kód meghatároz két házirendek:</span><span class="sxs-lookup"><span data-stu-id="9e703-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="9e703-141">Ezt a kódot is meghatározza a hitelesítési séma, amely közli az ASP.NET mely hitelesítési köztes kell futtatnia, ha a hitelesítés sikertelen.</span><span class="sxs-lookup"><span data-stu-id="9e703-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="9e703-142">Ebben az esetben azt adjon meg a cookie-k hitelesítési köztes, mert a cookie-k hitelesítési köztes is átirányítja a felhasználót egy "Tiltott" lapon.</span><span class="sxs-lookup"><span data-stu-id="9e703-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="9e703-143">A tiltott lap helyének megadása a `AccessDeniedPath` választás, a cookie-k köztes; lásd: [konfigurálása a hitelesítési köztes].</span><span class="sxs-lookup"><span data-stu-id="9e703-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="9e703-144">A tartományvezérlő műveletek engedélyezése</span><span class="sxs-lookup"><span data-stu-id="9e703-144">Authorize controller actions</span></span>
<span data-ttu-id="9e703-145">Végül egy műveletet az MVC-vezérlőhöz engedélyezéséhez, állítsa be ezt a házirendet a `Authorize` attribútum:</span><span class="sxs-lookup"><span data-stu-id="9e703-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="9e703-146">Az ASP.NET korábbi verziójában állíthat a **szerepkörök** az attribútum a tulajdonság:</span><span class="sxs-lookup"><span data-stu-id="9e703-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

<span data-ttu-id="9e703-147">Ez az ASP.NET Core továbbra is támogatja, de azt hátrányokkal engedélyezési házirendek képest:</span><span class="sxs-lookup"><span data-stu-id="9e703-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="9e703-148">Egy adott jogcímtípushoz feltételezi.</span><span class="sxs-lookup"><span data-stu-id="9e703-148">It assumes a particular claim type.</span></span> <span data-ttu-id="9e703-149">Házirendek bármely jogcímtípus ellenőrizheti.</span><span class="sxs-lookup"><span data-stu-id="9e703-149">Policies can check for any claim type.</span></span> <span data-ttu-id="9e703-150">Szerepköröket használják, csak a jogcím típusának.</span><span class="sxs-lookup"><span data-stu-id="9e703-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="9e703-151">A szerepkör neve nem változtatható be az attribútumot.</span><span class="sxs-lookup"><span data-stu-id="9e703-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="9e703-152">Egyetlen helyen, van az engedélyezési logikai házirendekkel, így a frissítés, vagy még akkor is, load a konfigurációs beállítások.</span><span class="sxs-lookup"><span data-stu-id="9e703-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="9e703-153">Szabályzatok lehetővé összetettebb felhasználását engedélyezési döntésekhez (pl. elavulnak > = 21), amelyek nem fejezhetők egyszerű szerepkör tagsága.</span><span class="sxs-lookup"><span data-stu-id="9e703-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="9e703-154">Erőforrás-alapú engedélyezési</span><span class="sxs-lookup"><span data-stu-id="9e703-154">Resource based authorization</span></span>
<span data-ttu-id="9e703-155">*Erőforrás-alapú engedélyezési* akkor fordul elő, amikor az engedélyezési művelet által érintett adott erőforrás függ.</span><span class="sxs-lookup"><span data-stu-id="9e703-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="9e703-156">A Dejójáték felmérések alkalmazás minden felmérés van egy tulajdonost és nulla-a-többhöz közreműködők.</span><span class="sxs-lookup"><span data-stu-id="9e703-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="9e703-157">A tulajdonos olvasása, frissítése, közzétételét és a felmérés visszavonni.</span><span class="sxs-lookup"><span data-stu-id="9e703-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="9e703-158">A tulajdonos közreműködők rendelhet a felmérés.</span><span class="sxs-lookup"><span data-stu-id="9e703-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="9e703-159">A közreműködők olvashatja, és frissítse a felmérés.</span><span class="sxs-lookup"><span data-stu-id="9e703-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="9e703-160">Ne feledje, hogy "tulajdonos" és "közreműködő" nem alkalmazási szerepköröknek; Ezek felmérésekre, az alkalmazás adatbázisban tárolódnak.</span><span class="sxs-lookup"><span data-stu-id="9e703-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="9e703-161">Ellenőrizze, hogy a felhasználó törölhet egy felmérés, például az alkalmazás ellenőrzi, hogy a felhasználó adott felmérés tulajdonosaként.</span><span class="sxs-lookup"><span data-stu-id="9e703-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="9e703-162">Az ASP.NET Core valósítja meg az erőforrás-alapú engedélyezési által származó **AuthorizationHandler** és felülírása a **kezelni** metódust.</span><span class="sxs-lookup"><span data-stu-id="9e703-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="9e703-163">Figyelje meg, hogy ez az osztály erős típusmegadású felmérés objektumok.</span><span class="sxs-lookup"><span data-stu-id="9e703-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="9e703-164">Az osztály rögzítése DI indításakor:</span><span class="sxs-lookup"><span data-stu-id="9e703-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="9e703-165">Hitelesítési ellenőrzések elvégzéséhez használja a **IAuthorizationService** felület, amely akkor is szúrjon be a tartományvezérlőket.</span><span class="sxs-lookup"><span data-stu-id="9e703-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="9e703-166">Az alábbi kód ellenőrzi, hogy egy felhasználó egy felmérés olvashatja:</span><span class="sxs-lookup"><span data-stu-id="9e703-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="9e703-167">Mivel jelenleg adjon át egy `Survey` objektum, meghívják a hívás a `SurveyAuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="9e703-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="9e703-168">Az engedélyezési kód egy jó módszer, hogy a felhasználói szerepkör- és erőforrás-alapú engedélyeket, majd ellenőrizze a kért műveletet a összesítés meg összesített összes.</span><span class="sxs-lookup"><span data-stu-id="9e703-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="9e703-169">Íme egy példa a felmérések alkalmazásból.</span><span class="sxs-lookup"><span data-stu-id="9e703-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="9e703-170">Az alkalmazás több Engedélytípusok határozza meg:</span><span class="sxs-lookup"><span data-stu-id="9e703-170">The application defines several permission types:</span></span>

* <span data-ttu-id="9e703-171">Rendszergazda</span><span class="sxs-lookup"><span data-stu-id="9e703-171">Admin</span></span>
* <span data-ttu-id="9e703-172">Közreműködő</span><span class="sxs-lookup"><span data-stu-id="9e703-172">Contributor</span></span>
* <span data-ttu-id="9e703-173">Létrehozó</span><span class="sxs-lookup"><span data-stu-id="9e703-173">Creator</span></span>
* <span data-ttu-id="9e703-174">Tulajdonos</span><span class="sxs-lookup"><span data-stu-id="9e703-174">Owner</span></span>
* <span data-ttu-id="9e703-175">Olvasó</span><span class="sxs-lookup"><span data-stu-id="9e703-175">Reader</span></span>

<span data-ttu-id="9e703-176">Az alkalmazás azt is meghatározza, hogy a lehetséges műveletkészlet felméréséről:</span><span class="sxs-lookup"><span data-stu-id="9e703-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="9e703-177">Létrehozás</span><span class="sxs-lookup"><span data-stu-id="9e703-177">Create</span></span>
* <span data-ttu-id="9e703-178">Olvasás</span><span class="sxs-lookup"><span data-stu-id="9e703-178">Read</span></span>
* <span data-ttu-id="9e703-179">Frissítés</span><span class="sxs-lookup"><span data-stu-id="9e703-179">Update</span></span>
* <span data-ttu-id="9e703-180">Törlés</span><span class="sxs-lookup"><span data-stu-id="9e703-180">Delete</span></span>
* <span data-ttu-id="9e703-181">Közzététel</span><span class="sxs-lookup"><span data-stu-id="9e703-181">Publish</span></span>
* <span data-ttu-id="9e703-182">Unpublsh</span><span class="sxs-lookup"><span data-stu-id="9e703-182">Unpublsh</span></span>

<span data-ttu-id="9e703-183">Az alábbi kód létrehoz egy adott felhasználó és felmérés engedélyeinek a listáját.</span><span class="sxs-lookup"><span data-stu-id="9e703-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="9e703-184">Figyelje meg, hogy ez a kód ellenőrzi, hogy az alkalmazás szerepkörökhöz, mind a tulajdonos/közreműködő mezőket a felmérés.</span><span class="sxs-lookup"><span data-stu-id="9e703-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="9e703-185">Egy több-bérlős alkalmazásban gondoskodnia kell arról, hogy engedélyek nem "nyilvánosságra kerüljenek" más bérlőket adatokat.</span><span class="sxs-lookup"><span data-stu-id="9e703-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="9e703-186">A felmérések alkalmazás közreműködői engedély lehetővé teszi a keresztül bérlők &mdash; valaki más bérlőhöz, egy contriubutor rendelhet.</span><span class="sxs-lookup"><span data-stu-id="9e703-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contriubutor.</span></span> <span data-ttu-id="9e703-187">A más Engedélytípusok korlátozódnak, a felhasználó bérlői erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="9e703-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="9e703-188">Ezt a követelményt kényszerítéséhez a kód ellenőrzi a bérlő azonosítója az engedély megadása előtt.</span><span class="sxs-lookup"><span data-stu-id="9e703-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="9e703-189">(A `TenantId` rendeli, a felmérést létrehozásakor mezőben.)</span><span class="sxs-lookup"><span data-stu-id="9e703-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="9e703-190">A következő lépés, hogy ellenőrizze a műveletet (olvasás, frissítés, törlés, stb.) a engedélyekkel szemben.</span><span class="sxs-lookup"><span data-stu-id="9e703-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="9e703-191">A felmérések alkalmazás megvalósítja a ebben a lépésben egy keresési tábla funkciók használatával:</span><span class="sxs-lookup"><span data-stu-id="9e703-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="9e703-192">[**Következő**][web-api]</span><span class="sxs-lookup"><span data-stu-id="9e703-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[alkalmazási szerepköröknek]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[megvalósítása hivatkozhat]: tailspin.md
[konfigurálása a hitelesítési köztes]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
