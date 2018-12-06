---
title: Hitelesítés a több-bérlős alkalmazásokban
description: Engedélyezési végrehajtása egy több-bérlős alkalmazásban
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: bbf702fe6651625a1aeceff7e4e321dd08c38544
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902493"
---
# <a name="role-based-and-resource-based-authorization"></a>Szerepkör- és erőforrás-alapú hitelesítés

[![GitHub](../_images/github.png) Mintakód][sample application]

A [megvalósítási hivatkozhat] ASP.NET Core-alkalmazás. Ebben a cikkben áttekintjük engedélyezési, két általános megközelítése az engedélyt az ASP.NET Core API-k használatával.

* **Szerepkör-alapú hitelesítést**. A felhasználóhoz hozzárendelt szerepkörök alapján művelet engedélyezése. Ha például bizonyos műveleteket rendszergazdai szerepkör van szükség.
* **Erőforrás-alapú hitelesítést**. Engedélyező a művelet egy adott erőforrás alapján. Ha például minden erőforrás van tulajdonosa. A tulajdonos törölheti az erőforrást; nem végezhető el más felhasználók.

A tipikus app vegyesen is fogja alkalmazni. Ha például egy erőforrás törlése a felhasználónak kell lennie az erőforrás tulajdonosa *vagy* rendszergazda.

## <a name="role-based-authorization"></a>Szerepkör-alapú hitelesítést
A [Tailspin Surveys] [ Tailspin] alkalmazás határozza meg a következő szerepkörök:

* Rendszergazda. Bármely felmérés a bérlőhöz tartozó összes CRUD-műveletek hajthat végre.
* Létrehozó. Hozhat létre új felmérések
* Olvasó. Olvashatja a bérlőhöz tartozó bármely felmérések

Szerepkörök a alkalmazni *felhasználók* az alkalmazás. A Surveys alkalmazás egy felhasználó, egy rendszergazda, a létrehozó vagy a reader.

Bemutatja, hogyan határozhatja meg és kezelheti a szerepköröket, lásd: [Alkalmazási szerepkörök].

Függetlenül attól, hogyan kezelheti a szerepköröket az engedélyezési kódot fog hasonlítani. ASP.NET Core rendelkezik nevű absztrakciós [engedélyezési házirendek][policies]. Ezzel a funkcióval engedélyezési házirendeket határozhat meg a kódot, és majd alkalmazhatja őket vezérlő műveleteket hajthat végre. A szabályzat különválik a vezérlő.

### <a name="create-policies"></a>Szabályzatok létrehozása
Meghatározhat egy olyan szabályzatot, először hozzon létre egy osztályt, amely megvalósítja `IAuthorizationRequirement`. A legegyszerűbb származtassa `AuthorizationHandler`. Az a `Handle` metódus, vizsgálja meg a megfelelő jogcím.

Íme egy példa a Tailspin Surveys-alkalmazás:

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

Ez az osztály határozza meg a felhasználót, hogy hozzon létre egy új felmérés vonatkozó követelmény. A felhasználónak kell lennie a SurveyAdmin vagy SurveyCreator szerepkörben.

Az indítási osztályt határoz meg egy nevesített szabályzatot, amely tartalmaz egy vagy több követelménynek. Ha több követelmények, a felhasználónak meg kell felelnie *minden* követelmény, hogy engedélyezni kell. A következő kódot a két szabályzat határozza meg:

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

Ez a kód beállítja a hitelesítési sémát, amely közli az ASP.NET, mely hitelesítési közbenső kell futnia, ha a hitelesítés sikertelen. Ebben az esetben azt adja meg cookie-k közbenső hitelesítési szoftver, mert a cookie-k közbenső hitelesítési szoftver is átirányítja a felhasználót egy "Tiltott" lap. A tiltott lap helye van megadva a `AccessDeniedPath` első számú megoldás a cookie-k közbenső; lásd: [A közbenső hitelesítési szoftver konfigurálása].

### <a name="authorize-controller-actions"></a>Vezérlő műveletek engedélyezése
Végül az MVC-vezérlő művelet engedélyezéséhez, állítsa be ezt a házirendet a `Authorize` attribútum:

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

Az ASP.NET korábbi verzióiban, így állíthatja be a **szerepkörök** az attribútum a tulajdonság:

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

Ez továbbra is támogatja az ASP.NET Core, de azt hátrányokkal engedélyezési házirendek képest:

* Azt feltételezi, hogy egy adott jogcímtípushoz. Házirendek bármilyen jogcímtípust kereshet. Ezek a jogcím olyan típusú.
* A szerepkör neve nem változtatható, az attribútumot. Szabályzatok az engedélyezési logika érhető el minden egy helyen tudhatja, megkönnyítve a frissítés vagy a terhelést a konfigurációs beállítások.
* Szabályzatok lehetővé teszik a bonyolult felhasználását engedélyezési döntésekhez (pl. kor > = 21), amelyek nem fejezhetők egyszerű szerepköri tagság szerint.

## <a name="resource-based-authorization"></a>Erőforrás-alapú engedélyezési
*Erőforrás-alapú engedélyezési* következik be, a hitelesítést egy adott erőforrás-művelet által érintett függ. A Tailspin Surveys alkalmazás minden felmérés van tulajdonosa és nulla-a-többhöz közreműködő.

* A tulajdonos olvasása, frissítése, közzététel és közzétételének visszavonása a kérdőív kitöltését.
* A tulajdonos közreműködők rendelhet a kérdőív kitöltését.
* A közreműködők olvashatja, és frissítse a kérdőív kitöltését.

Vegye figyelembe, hogy "tulajdonos" és "közreműködő" ne legyenek; alkalmazás-szerepkörök az alkalmazás-adatbázis felmérésekre tárolódnak. Annak ellenőrzéséhez, hogy a felhasználók törölhetik a felmérés, például az alkalmazás ellenőrzi, hogy a felhasználó a tulajdonos, a felmérést.

Az ASP.NET Core, szerint alkalmazza az erőforrás-alapú hitelesítést kapcsolatból származtatott kapcsolatot **AuthorizationHandler** és felülírása a **kezelni** metódust.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Figyelje meg, hogy ez az osztály van listaobjektum felmérés objektumokhoz.  Az osztály regisztráljon DI indításakor:

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Hitelesítési ellenőrzések elvégzéséhez, használja a **IAuthorizationService** illesztőt, amely akkor is behelyezése a tartományvezérlőket. Az alábbi kód ellenőrzi-e egy felhasználó egy kérdőív olvashat:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Mivel adjuk át egy `Survey` objektumot, a hívás meghívja a `SurveyAuthorizationHandler`.

Az engedélyezési kódot jó módszer, hogy a felhasználói szerepkör- és erőforrás-alapú engedélyeket, majd az ellenőrzési összesített állítsa be a kívánt műveletet ellen összesített minden.
Íme egy példa a Surveys alkalmazással. Az alkalmazás több Engedélytípusok határozza meg:

* Adminisztratív körzet
* Közreműködő
* Létrehozó
* Tulajdonos
* Olvasó

Az alkalmazás is felmérések lehetséges műveletek egy csoportját határozza meg:

* Létrehozás
* Olvasás
* Frissítés
* Törlés
* Közzététel
* Unpublsh

Az alábbi kód létrehoz egy adott felhasználó és a felmérés vonatkozó engedélyek listája. Figyelje meg, hogy ez a kód megvizsgálja a felhasználó alkalmazás-szerepkörök, mind a tulajdonosa vagy közreműködője mezőket a kérdőív kitöltését.

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

Egy több-bérlős alkalmazásban gondoskodnia kell arról, hogy engedélyeket nem "nyilvánosságra kerüljenek" egy másik bérlő adatai. A Surveys alkalmazás közreműködői engedélye engedélyezett bérlők között &mdash; valaki más bérlők közreműködője is hozzárendelhet. Annak a felhasználónak a bérlőhöz tartozó erőforrásokhoz a többi Engedélytípusok korlátozódnak. Ezt a követelményt kényszerítéséhez a kód ellenőrzi a bérlő azonosítója az engedély megadása előtt. (A `TenantId` módon rendeli hozzá a felmérés létrehozásakor mezőben.)

A következő lépés, hogy ellenőrizze a műveletet (olvasás, update, delete stb.) ellen az engedélyeket. A Surveys alkalmazás ebben a lépésben egy keresési táblázat a functions használatával valósítja meg:

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

[**Tovább**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[Alkalmazási szerepkörök]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[megvalósítási hivatkozhat]: tailspin.md
[A közbenső hitelesítési szoftver konfigurálása]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
