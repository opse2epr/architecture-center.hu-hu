---
title: Regisztráció és a bérlők a több-bérlős alkalmazásokhoz bevezetése
description: Hogyan kell bevezetni bérlők egy több-bérlős alkalmazásban
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: dde577d5bab63fb436d52fb4548399d5bd8bb38f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="tenant-sign-up-and-onboarding"></a>A bérlők előfizetési és bevezetése

[![GitHub](../_images/github.png) példakód][sample application]

Ez a cikk ismerteti, hogyan végrehajtásához egy *előfizetési* egy több-bérlős alkalmazás, amely lehetővé teszi a regisztrációt a szervezet az alkalmazás ügyfél feldolgozása.
A regisztrációs folyamat végrehajtása több oka lehet:

* Egy AD-rendszergazdát, hogy az alkalmazás használatához az ügyfél a teljes szervezetben hozzájárulás engedélyezése.
* Gyűjti a hitelkártya fizetési vagy más ügyfél adatait.
* Az alkalmazás által igényelt egyszeri / bérlői telepítés végrehajtása.

## <a name="admin-consent-and-azure-ad-permissions"></a>Rendszergazda jóváhagyását és az Azure AD-engedélyek
Hitelesítéséhez az Azure ad-vel, az alkalmazás a felhasználó hozzáférésre van szüksége. Legalább az alkalmazást kell olvasni a felhasználói profil. Az első alkalommal a felhasználó bejelentkezik, az Azure AD egy hozzájárulási oldal, amely megjeleníti a kért engedélyeket jeleníti meg. Kattintva **elfogadás**, a felhasználó engedélyt ad az alkalmazást.

Alapértelmezés szerint engedély felhasználónkénti alapon. Minden felhasználó számára kérelmezni a hozzájárulási oldalon láthatja. Azonban, hogy az Azure AD támogatja-e is *admin hozzájárulási*, amely lehetővé teszi, hogy a rendszergazda számára, hogy a teljes szervezet hozzájárulás AD.

A rendszergazda hozzájárulási folyamat használata esetén a hozzájárulási lap szerint az, hogy az AD felügyeleti engedélyt a teljes bérlő nevében:

![Felügyeleti beleegyezést kérő üzenete](./images/admin-consent.png)

Miután rákattint a rendszergazda **elfogadás**, ugyanannak a bérlőnek belüli más felhasználók bejelentkezhetnek, és az Azure AD kihagyja a hozzájárulási képernyő.

Csak AD a rendszergazda biztosíthat a rendszergazda jóváhagyását, mert a teljes szervezet nevében engedélyt. Ha egy nem rendszergazdai próbálkozik a rendszergazda jóváhagyását folyamatot, az Azure AD egy hibaüzenet jelenik meg:

![Hozzájárulás hiba](./images/consent-error.png)

Ha az alkalmazás később további engedélyek, az ügyfél jelentkezzen újra, és hogy a frissített engedélyeket kell.  

## <a name="implementing-tenant-sign-up"></a>A bérlő előfizetés megvalósítása
Az a [Dejójáték felmérések] [ Tailspin] alkalmazás, a regisztrációs folyamat több követelményei meghatározott:

* A bérlő kell iratkozzon fel, mielőtt a felhasználók bejelentkezhetnek.
* Regisztráció az admin hozzájárulási folyamat használja.
* Az adatbázis-előfizetési hozzáadja a felhasználó bérlői.
* A bérlő előfizet, miután a az alkalmazás egy bevezetési lapját jeleníti meg.

Ebben a szakaszban végigvesszük az a regisztrációs folyamat végrehajtása.
Fontos tudni, hogy "regisztrációs" és "bejelentkezés" van egy alkalmazás fogalom. A hitelesítési folyamat során az Azure AD nem eredendően tudja, hogy a felhasználó regisztrál javításán. A környezet nyomon követéséhez az alkalmazáshoz van.

Névtelen felhasználó meglátogat felmérések alkalmazása, a felhasználó esetén látható két gomb, egy való bejelentkezéshez, és egy "regisztrálása a vállalati" (regisztrálás).

![Alkalmazás-előfizetési lap](./images/sign-up-page.png)

Az ilyen gombokat a műveletek meghívása a `AccountController` osztály.

A `SignIn` művelet értéket ad vissza egy **ChallegeResult**, amelyek hatására az OpenID Connect köztes átirányítani a hitelesítési végpontra. Ez az eseményindító hitelesítés az ASP.NET Core az alapértelmezett lehetőség.  

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

Most hasonlítsa össze a `SignUp` művelet:

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

Például `SignIn`, a `SignUp` művelet is adja vissza egy `ChallengeResult`. Most, azt hozzá egy adatait az adat a `AuthenticationProperties` a a `ChallengeResult`:

* regisztráció: egy logikai jelző, amely jelzi, hogy a felhasználó elindult-e a regisztrációs folyamat.

Az állapotadatokat az `AuthenticationProperties` bekerül az OpenID Connect [állapot] paraméter, amely kerekíteni való adatváltások számát a hitelesítési folyamat során.

![Állapot paraméter](./images/state-parameter.png)

Miután a felhasználó az Azure AD hitelesíti, és lekérdezi visszairányítja az alkalmazást, a hitelesítési jegy állapotát tartalmazza. Győződjön meg arról, hogy a "signup" érték továbbra is fennáll, az egész hitelesítési folyamat között a használjuk a tény.

## <a name="adding-the-admin-consent-prompt"></a>A rendszergazda beleegyezést kérő üzenete hozzáadása
Az Azure ad-ben a rendszergazda hozzájárulási folyamat elindítja a sématelepítési "prompt" paramétert adna a lekérdezési karakterláncot a hitelesítési kérelem:

```
/authorize?prompt=admin_consent&...
```

A felmérések alkalmazás hozzáadása során a parancssorba a `RedirectToAuthenticationEndpoint` esemény. Ez az esemény jobb nevezik, előtt a köztes átirányítja a felhasználókat a hitelesítési végpontra.

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

Beállítás` ProtocolMessage.Prompt` közli a közbenső szoftvert adja hozzá a "kérdés" paramétert a hitelesítési kérelmet.

Vegye figyelembe, hogy a kérés csak akkor szükséges, regisztráció során. Rendszeres bejelentkezés nem foglalja azt. Különbségtétel, a ellenőrizzük a `signup` hitelesítési állapot értéke. A következő kiterjesztésmetódus ellenőrzi a feltétel:

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

## <a name="registering-a-tenant"></a>A bérlő regisztrálása
A felmérések alkalmazás mindegyik bérlő kapcsolatos információkat és a felhasználó az alkalmazás-adatbázisban tárolja.

![Bérlői tábla](./images/tenant-table.png)

A bérlői tábla IssuerValue az érték a kibocsátó jogcím a bérlő számára. Ez az az Azure AD `https://sts.windows.net/<tentantID>` és bérlőnként egy egyedi értéket ad.

Amikor egy új bérlő előfizet, a felmérések alkalmazás bérlői rekordot ír az adatbázis. Ez akkor történik meg belül a `AuthenticationValidated` esemény. (Ne elvégezhető előtt ezt az eseményt, mert a azonosító jogkivonat nem érvényesítése még, így a jogcímértékek nem megbízható. Lásd: [hitelesítési].

Ez a megfelelő kódot a felmérések alkalmazásból:

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

1. Ellenőrzés, hogy a bérlő kibocsátó értéket már az adatbázisban. Ha a bérlő nem írta alá, `FindByIssuerValueAsync` null értéket ad vissza.
2. Ha a felhasználó regisztrál:
   1. A bérlő hozzáadása az adatbázishoz (`SignUpTenantAsync`).
   2. A hitelesített felhasználó hozzáadása az adatbázishoz (`CreateOrUpdateUserAsync`).
3. Ellenkező esetben hajtsa végre a szokásos bejelentkezési folyamata:
   1. Ha a bérlő kibocsátó nem található az adatbázisban, akkor a bérlő nincs regisztrálva, és regisztrálni kell az ügyfél. Ebben az esetben idéznie egy kivételt a hitelesítés sikertelen lesz.
   2. Máskülönben hozzon létre egy adatbázis-rekord ehhez a felhasználóhoz, ha nincs ilyen már (`CreateOrUpdateUserAsync`).

Itt a `SignUpTenantAsync` módszer, amely a bérlői hozzáadása az adatbázishoz.

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

Ez a teljes regisztrációs folyamat az felmérések alkalmazásban összefoglalása:

1. A felhasználó kattint a **regisztráció** gombra.
2. A `AccountController.SignUp` művelet challege eredményt adja vissza.  A hitelesítési állapot "signup" értéket tartalmaz.
3. Az a `RedirectToAuthenticationEndpoint` esemény, vegye fel a `admin_consent` kérdés.
4. Az OpenID Connect köztes átirányítja az Azure AD és a felhasználó hitelesíti magát.
5. Az a `AuthenticationValidated` esemény, keresse meg a "signup" állapotban.
6. A bérlő hozzáadása az adatbázishoz.

[**Következő**][app roles]

<!-- Links -->
[app roles]: app-roles.md
[Tailspin]: tailspin.md

[állapot]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[hitelesítési]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
