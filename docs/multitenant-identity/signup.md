---
title: Regisztráció és a bérlők felvétele több-bérlős alkalmazásokban
description: Hogyan kell előkészíteni bérlők egy több-bérlős alkalmazásban.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: eb4e65b20ec3339b633b65d2adad768e98d1bdbb
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640600"
---
# <a name="tenant-sign-up-and-onboarding"></a>Bérlői feliratkozás és előkészítés

[![GitHub](../_images/github.png) Mintakód][sample application]

Ez a cikk bemutatja, hogyan valósíthat meg egy *előfizetési* a több-bérlős alkalmazás, amely lehetővé teszi, hogy egy ügyfél az alkalmazás szervezeti regisztráció folyamatát.
A regisztrációs folyamat végrehajtásához több oka is van:

* AD rendszergazdai jóváhagyást az ügyfél a teljes cég számára, az alkalmazás használatának engedélyezése.
* Hitelkártyás fizetés vagy más ügyfél-információkat gyűjtése.
* Bármely az alkalmazás által egyszeri bérlőnkénti telepítést végre.

## <a name="admin-consent-and-azure-ad-permissions"></a>Rendszergazdai jóváhagyás és az Azure AD-engedélyekről

Annak érdekében, hogy az Azure AD-hitelesítést, az alkalmazás a felhasználó hozzá kell férnie. Legalább az alkalmazás a felhasználói profil olvasása engedélyre van szüksége. Egy felhasználó bejelentkezik, először az Azure AD egy hozzájárulást kérő lap, amely megjeleníti a kért engedélyeket jeleníti meg. Kattintva **elfogadás**, a felhasználó engedélyt ad az alkalmazásnak.

Alapértelmezés szerint engedély felhasználónkénti alapon. Minden felhasználó, aki bejelentkezik a hozzájárulást kérő lap fog látni. Azonban, hogy az Azure AD támogatja-e is *rendszergazdai jóváhagyás*, amely lehetővé teszi, hogy egy AD-rendszergazdát, hogy engedélyt adjanak a teljes szervezet számára.

Ha a rendszergazdai jóváhagyási folyamatot használja, a hozzájárulást kérő lap állapotok a, hogy az AD-rendszergazda a teljes bérlő nevében engedélyt biztosít:

![Rendszergazdai jóváhagyás kérése](./images/admin-consent.png)

Miután a rendszergazda kattint **elfogadás**, az ugyanazon bérlőn belüli más felhasználók bejelentkezhetnek, és az Azure AD ezért kihagyja a jóváhagyást kérő képernyőt.

Csak egy AD-rendszergazda engedélyezheti a rendszergazda, mert a teljes szervezet nevében engedélyt. Ha nem rendszergazdai próbálkozik a rendszergazdai jóváhagyási folyamatot, az Azure AD egy hibaüzenet jelenik meg:

![Jóváhagyás hiba](./images/consent-error.png)

Ha az alkalmazás későbbi időpontban további engedélyeket igényel, az ügyfél kell újra regisztráljon, és hozzájárul az engedélyekkel.

## <a name="implementing-tenant-sign-up"></a>Bérlői feliratkozás megvalósítása

Az a [Tailspin Surveys] [ Tailspin] alkalmazás, a regisztrációs folyamat számos követelményei meghatározott:

* Egy bérlő regisztrálás felhasználóknak a bejelentkezéshez.
* Regisztráció a rendszergazdai jóváhagyási folyamatot használ.
* Az alkalmazás-adatbázis-előfizetési hozzáadja a felhasználó bérlőjéhez.
* Miután egy bérlő regisztrál, a az alkalmazás egy előkészítési lapját jeleníti meg.

Ebben a szakaszban végigvesszük megvalósítása a regisztrációs folyamat tartalmaz.
Fontos tudni, hogy a "regisztráció" és "sign-in" van egy alkalmazás fogalom. A hitelesítési folyamat során az Azure AD nem természetüknél fogva tudja, hogy a felhasználó éppen regisztráló. Az alkalmazás a helyi nyomon követéséhez van.

Névtelen felhasználó meglátogat a Surveys alkalmazás, amikor a felhasználó nem látható két gombot, egy bejelentkezni, és egy "regisztrálni a vállalat" (regisztrálás).

![Alkalmazás-előfizetési lap](./images/sign-up-page.png)

Ezekre a gombokra hajthatók végre műveletek, az a `AccountController` osztály.

A `SignIn` művelet értéket ad vissza egy **ChallengeResult**, amely hatására az OpenID Connect közbenső szoftvert átirányítani a hitelesítési végpontra. Ez az eseményindító-hitelesítés az ASP.NET Core alapértelmezett módja.

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

Hasonlítsa össze a `SignUp` művelet:

```csharp
[AllowAnonymous]
public IActionResult SignUp()
{
    var state = new Dictionary<string, string> { { "signup", "true" }};
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties(state)
        {
            RedirectUri = Url.Action(nameof(SignUpCallback), "Account")
        });
}
```

Például `SignIn`, a `SignUp` művelet is adja vissza egy `ChallengeResult`. Ebben az esetben hozzáadunk egy részét az állapotinformációkat, de a `AuthenticationProperties` a a `ChallengeResult`:

* signup: Egy logikai jelző, amely jelzi, hogy a felhasználó elindult-e a regisztrációs folyamat.

Az állapotinformációt `AuthenticationProperties` lekérdezi hozzáadni az OpenID Connect [állapot] paramétert, amelyre kerekíteni lelassítja a hitelesítési folyamat során.

![Állapot paraméter](./images/state-parameter.png)

Miután a felhasználó hitelesíti magát az Azure ad-ben, és átirányítja az alkalmazásnak, a hitelesítési jegy állapotát tartalmazza. Ez a tény, hogy a "signup" érték továbbra is fennáll, az egész hitelesítési folyamat között használjuk.

## <a name="adding-the-admin-consent-prompt"></a>A rendszergazda beleegyezést kérő hozzáadása

Az Azure AD-ben a rendszergazdai jóváhagyás folyamathoz "parancssor" paraméter hozzáadása a lekérdezési karakterláncot a hitelesítési kérelem által aktivált:

<!-- markdownlint-disable MD040 -->

```
/authorize?prompt=admin_consent&...
```

<!-- markdownlint-enable MD040 -->

A Surveys alkalmazás hozzáadása során a rendszer kéri a `RedirectToAuthenticationEndpoint` esemény. Ez az esemény jobb neve előtt a közbenső szoftver a hitelesítési végpontra irányítja át.

```csharp
public override Task RedirectToAuthenticationEndpoint(RedirectContext context)
{
    if (context.IsSigningUp())
    {
        context.ProtocolMessage.Prompt = "admin_consent";
    }

    _logger.RedirectToIdentityProvider();
    return Task.FromResult(0);
}
```

Beállítás `ProtocolMessage.Prompt` arra utasítja a közbenső szoftverek, a "kérdés" paraméter hozzáadása a hitelesítési kérelmet.

Vegye figyelembe, hogy a rendszer csak akkor van szükség a regisztrációhoz. Rendszeres bejelentkezés nem tartalmaznia kell azt. Megkülönböztetni őket, ellenőrzése a `signup` hitelesítési állapot értéke. Ezt az állapotot ellenőrzi a következő metódust:

```csharp
internal static bool IsSigningUp(this BaseControlContext context)
{
    Guard.ArgumentNotNull(context, nameof(context));

    string signupValue;
    // Check the HTTP context and convert to string
    if ((context.Ticket == null) ||
        (!context.Ticket.Properties.Items.TryGetValue("signup", out signupValue)))
    {
        return false;
    }

    // We have found the value, so see if it's valid
    bool isSigningUp;
    if (!bool.TryParse(signupValue, out isSigningUp))
    {
        // The value for signup is not a valid boolean, throw

        throw new InvalidOperationException($"'{signupValue}' is an invalid boolean value");
    }

    return isSigningUp;
}
```

## <a name="registering-a-tenant"></a>A bérlő regisztrációja

A Surveys alkalmazás minden egyes bérlő némi információt és a felhasználó az alkalmazás-adatbázis tárolja.

![Bérlő táblában](./images/tenant-table.png)

A bérlő táblában IssuerValue az érték a kiállító jogcím a bérlő számára. Ez az Azure AD-hez `https://sts.windows.net/<tentantID>` és a egy egyedi értéket bérlőnként.

Amikor egy új bérlő előfizet, a Surveys alkalmazás bérlői rekordot ír az adatbázis. Ez belül történik a `AuthenticationValidated` esemény. (Ne végezze el, ez az esemény előtt, mert a azonosító jogkivonat nem érvényesíthető, így nem megbízható a jogcímértékek. Lásd: [Hitelesítés].

A megfelelő kódot a Surveys alkalmazás az itt látható:

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var userId = principal.GetObjectIdentifierValue();
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    _logger.AuthenticationValidated(userId, issuerValue);

    // Normalize the claims first.
    NormalizeClaims(principal);
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue)
        .ConfigureAwait(false);

    if (context.IsSigningUp())
    {
        if (tenant == null)
        {
            tenant = await SignUpTenantAsync(context, tenantManager)
                .ConfigureAwait(false);
        }

        // In this case, we need to go ahead and set up the user signing us up.
        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
    else
    {
        if (tenant == null)
        {
            _logger.UnregisteredUserSignInAttempted(userId, issuerValue);
            throw new SecurityTokenValidationException($"Tenant {issuerValue} is not registered");
        }

        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
}
```

Ez a kód a következőket teszi:

1. Ellenőrizze, hogy ha a bérlő kibocsátó értékét az adatbázisban már van. Ha a bérlő nem jelentkezett be, `FindByIssuerValueAsync` null értéket ad vissza.
2. Ha a felhasználó jelentkezik:
   1. Adja hozzá a bérlő az adatbázishoz (`SignUpTenantAsync`).
   2. A hitelesített felhasználó hozzáadása az adatbázishoz (`CreateOrUpdateUserAsync`).
3. Ellenkező esetben a normál bejelentkezési folyamat befejezése:
   1. Ha a bérlő kibocsátó nem található az adatbázisban, az azt jelenti, a bérlő nincs regisztrálva, és regisztrálnia kell az ügyfélnek. Ebben az esetben kivételt okoz a hitelesítés sikertelen lesz.
   2. Máskülönben hozzon létre egy adatbázis-rekord a felhasználónál, ha nincs ilyen már (`CreateOrUpdateUserAsync`).

Íme a `SignUpTenantAsync` metódushoz, amely hozzáad a bérlő az adatbázishoz.

```csharp
private async Task<Tenant> SignUpTenantAsync(BaseControlContext context, TenantManager tenantManager)
{
    Guard.ArgumentNotNull(context, nameof(context));
    Guard.ArgumentNotNull(tenantManager, nameof(tenantManager));

    var principal = context.Ticket.Principal;
    var issuerValue = principal.GetIssuerValue();
    var tenant = new Tenant
    {
        IssuerValue = issuerValue,
        Created = DateTimeOffset.UtcNow
    };

    try
    {
        await tenantManager.CreateAsync(tenant)
            .ConfigureAwait(false);
    }
    catch(Exception ex)
    {
        _logger.SignUpTenantFailed(principal.GetObjectIdentifierValue(), issuerValue, ex);
        throw;
    }

    return tenant;
}
```

Itt látható a teljes regisztrációs folyamatot a Surveys alkalmazás összefoglalása:

1. A felhasználó kattint a **regisztráció** gombra.
2. A `AccountController.SignUp` művelet challenge eredményt adja vissza.  A hitelesítési állapot "signup" értéket tartalmaz.
3. Az a `RedirectToAuthenticationEndpoint` esemény, adja hozzá a `admin_consent` parancssort.
4. Az OpenID Connect közbenső szoftvert átirányítja a felhasználókat az Azure ad-ben, és a felhasználó hitelesíti magát.
5. Az a `AuthenticationValidated` esemény, keressen a "signup" állapot.
6. Adja hozzá a bérlő az adatbázishoz.

[**Tovább**][app roles]

<!-- links -->

[app roles]: app-roles.md
[Tailspin]: tailspin.md

[állapot]: https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[Hitelesítés]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
