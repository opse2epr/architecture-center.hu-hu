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
# <a name="role-based-and-resource-based-authorization"></a>Szerepkör- és erőforrás-alapú engedélyezési

[![GitHub](../_images/github.png) példakód][sample application]

A [megvalósítása hivatkozhat] ASP.NET Core-alkalmazás. Ebben a cikkben megnézzük, engedélyezési, két általános módszer az engedélyt az ASP.NET Core megadott API-k használatával.

* **Szerepkör-alapú engedélyezési**. A felhasználóhoz rendelt szerepkörök alapján műveletet engedélyezése. Például bizonyos műveleteket kell egy rendszergazdai szerepkört.
* **Erőforrás-alapú engedélyezési**. Egy adott erőforrás alapján művelet engedélyezése. Például minden erőforrás van tulajdonosa. A tulajdonos törlése az erőforrás; más felhasználók nem.

A tipikus app vegyesen is megfelelő. Például egy erőforrás törlése, a felhasználónak kell lennie az erőforrás tulajdonosa *vagy* egy rendszergazdával.

## <a name="role-based-authorization"></a>Szerepköralapú hitelesítés
A [Dejójáték felmérések] [ Tailspin] alkalmazás határozza meg a következő szerepkörök:

* Rendszergazda. Minden CRUD-műveleteknek a bármely felmérés, amely arra a bérlőre tartozik hajthat végre.
* Létrehozó. Új felmérések hozhat létre.
* Olvasó. Bármely felmérések arra a bérlőre tartozó olvashatja

Szerepkörök alkalmazása *felhasználók* az alkalmazás. Egy felhasználó felmérések alkalmazás, egy rendszergazda, a létrehozó vagy az olvasó.

Bemutatja, hogyan határozhatja meg és kezelheti a szerepkörök tárgyalását lásd: [alkalmazási szerepköröknek].

Hogyan kezelheti a szerepkörök, függetlenül az engedélyezési kód alábbihoz hasonlóan fog megjelenni. Az ASP.NET Core rendelkezik nevű absztrakciós [engedélyezési házirendek][policies]. Ezzel a szolgáltatással engedélyezési házirendek definiálása a kódban és a tartományvezérlő műveletek majd alkalmazza azokat a házirendeket. A házirendet a rendszer leválasztja a tartományvezérlő.

### <a name="create-policies"></a>Házirendek létrehozása
Adja meg a házirendet, először létre kell hoznia egy osztály, amely megvalósítja az `IAuthorizationRequirement`. A legegyszerűbb kapcsolattípusokból származhatnak, amelyek `AuthorizationHandler`. Az a `Handle` metódus, vizsgálja meg a megfelelő jogcím(ek).

Íme egy példa a Dejójáték felmérések alkalmazásból:

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

Ez az osztály a felhasználót, hogy hozzon létre egy új felmérés kapcsolatos követelmények meghatározása. A felhasználó a SurveyAdmin vagy SurveyCreator szerepkörrel kell.

Az indítási osztály, amely tartalmaz egy vagy több követelmények elnevezett házirend definiálásához. Ha több követelmények, a felhasználónak meg kell felelnie *minden* kell követelmény. Az alábbi kód meghatároz két házirendek:

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

Ezt a kódot is meghatározza a hitelesítési séma, amely közli az ASP.NET mely hitelesítési köztes kell futtatnia, ha a hitelesítés sikertelen. Ebben az esetben azt adjon meg a cookie-k hitelesítési köztes, mert a cookie-k hitelesítési köztes is átirányítja a felhasználót egy "Tiltott" lapon. A tiltott lap helyének megadása a `AccessDeniedPath` választás, a cookie-k köztes; lásd: [konfigurálása a hitelesítési köztes].

### <a name="authorize-controller-actions"></a>A tartományvezérlő műveletek engedélyezése
Végül egy műveletet az MVC-vezérlőhöz engedélyezéséhez, állítsa be ezt a házirendet a `Authorize` attribútum:

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

Az ASP.NET korábbi verziójában állíthat a **szerepkörök** az attribútum a tulajdonság:

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

Ez az ASP.NET Core továbbra is támogatja, de azt hátrányokkal engedélyezési házirendek képest:

* Egy adott jogcímtípushoz feltételezi. Házirendek bármely jogcímtípus ellenőrizheti. Szerepköröket használják, csak a jogcím típusának.
* A szerepkör neve nem változtatható be az attribútumot. Egyetlen helyen, van az engedélyezési logikai házirendekkel, így a frissítés, vagy még akkor is, load a konfigurációs beállítások.
* Szabályzatok lehetővé összetettebb felhasználását engedélyezési döntésekhez (pl. elavulnak > = 21), amelyek nem fejezhetők egyszerű szerepkör tagsága.

## <a name="resource-based-authorization"></a>Erőforrás-alapú engedélyezési
*Erőforrás-alapú engedélyezési* akkor fordul elő, amikor az engedélyezési művelet által érintett adott erőforrás függ. A Dejójáték felmérések alkalmazás minden felmérés van egy tulajdonost és nulla-a-többhöz közreműködők.

* A tulajdonos olvasása, frissítése, közzétételét és a felmérés visszavonni.
* A tulajdonos közreműködők rendelhet a felmérés.
* A közreműködők olvashatja, és frissítse a felmérés.

Ne feledje, hogy "tulajdonos" és "közreműködő" nem alkalmazási szerepköröknek; Ezek felmérésekre, az alkalmazás adatbázisban tárolódnak. Ellenőrizze, hogy a felhasználó törölhet egy felmérés, például az alkalmazás ellenőrzi, hogy a felhasználó adott felmérés tulajdonosaként.

Az ASP.NET Core valósítja meg az erőforrás-alapú engedélyezési által származó **AuthorizationHandler** és felülírása a **kezelni** metódust.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Figyelje meg, hogy ez az osztály erős típusmegadású felmérés objektumok.  Az osztály rögzítése DI indításakor:

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Hitelesítési ellenőrzések elvégzéséhez használja a **IAuthorizationService** felület, amely akkor is szúrjon be a tartományvezérlőket. Az alábbi kód ellenőrzi, hogy egy felhasználó egy felmérés olvashatja:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Mivel jelenleg adjon át egy `Survey` objektum, meghívják a hívás a `SurveyAuthorizationHandler`.

Az engedélyezési kód egy jó módszer, hogy a felhasználói szerepkör- és erőforrás-alapú engedélyeket, majd ellenőrizze a kért műveletet a összesítés meg összesített összes.
Íme egy példa a felmérések alkalmazásból. Az alkalmazás több Engedélytípusok határozza meg:

* Rendszergazda
* Közreműködő
* Létrehozó
* Tulajdonos
* Olvasó

Az alkalmazás azt is meghatározza, hogy a lehetséges műveletkészlet felméréséről:

* Létrehozás
* Olvasás
* Frissítés
* Törlés
* Közzététel
* Unpublsh

Az alábbi kód létrehoz egy adott felhasználó és felmérés engedélyeinek a listáját. Figyelje meg, hogy ez a kód ellenőrzi, hogy az alkalmazás szerepkörökhöz, mind a tulajdonos/közreműködő mezőket a felmérés.

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

Egy több-bérlős alkalmazásban gondoskodnia kell arról, hogy engedélyek nem "nyilvánosságra kerüljenek" más bérlőket adatokat. A felmérések alkalmazás közreműködői engedély lehetővé teszi a keresztül bérlők &mdash; valaki más bérlőhöz, egy contriubutor rendelhet. A más Engedélytípusok korlátozódnak, a felhasználó bérlői erőforrásokhoz. Ezt a követelményt kényszerítéséhez a kód ellenőrzi a bérlő azonosítója az engedély megadása előtt. (A `TenantId` rendeli, a felmérést létrehozásakor mezőben.)

A következő lépés, hogy ellenőrizze a műveletet (olvasás, frissítés, törlés, stb.) a engedélyekkel szemben. A felmérések alkalmazás megvalósítja a ebben a lépésben egy keresési tábla funkciók használatával:

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

[**Következő**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[alkalmazási szerepköröknek]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[megvalósítása hivatkozhat]: tailspin.md
[konfigurálása a hitelesítési köztes]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
