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
# <a name="tenant-sign-up-and-onboarding"></a><span data-ttu-id="f7477-103">A bérlők előfizetési és bevezetése</span><span class="sxs-lookup"><span data-stu-id="f7477-103">Tenant sign-up and onboarding</span></span>

<span data-ttu-id="f7477-104">[![GitHub](../_images/github.png) példakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="f7477-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="f7477-105">Ez a cikk ismerteti, hogyan végrehajtásához egy *előfizetési* egy több-bérlős alkalmazás, amely lehetővé teszi a regisztrációt a szervezet az alkalmazás ügyfél feldolgozása.</span><span class="sxs-lookup"><span data-stu-id="f7477-105">This article describes how to implement a *sign-up* process in a multi-tenant application, which allows a customer to sign up their organization for your application.</span></span>
<span data-ttu-id="f7477-106">A regisztrációs folyamat végrehajtása több oka lehet:</span><span class="sxs-lookup"><span data-stu-id="f7477-106">There are several reasons to implement a sign-up process:</span></span>

* <span data-ttu-id="f7477-107">Egy AD-rendszergazdát, hogy az alkalmazás használatához az ügyfél a teljes szervezetben hozzájárulás engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="f7477-107">Allow an AD admin to consent for the customer's entire organization to use the application.</span></span>
* <span data-ttu-id="f7477-108">Gyűjti a hitelkártya fizetési vagy más ügyfél adatait.</span><span class="sxs-lookup"><span data-stu-id="f7477-108">Collect credit card payment or other customer information.</span></span>
* <span data-ttu-id="f7477-109">Az alkalmazás által igényelt egyszeri / bérlői telepítés végrehajtása.</span><span class="sxs-lookup"><span data-stu-id="f7477-109">Perform any one-time per-tenant setup needed by your application.</span></span>

## <a name="admin-consent-and-azure-ad-permissions"></a><span data-ttu-id="f7477-110">Rendszergazda jóváhagyását és az Azure AD-engedélyek</span><span class="sxs-lookup"><span data-stu-id="f7477-110">Admin consent and Azure AD permissions</span></span>
<span data-ttu-id="f7477-111">Hitelesítéséhez az Azure ad-vel, az alkalmazás a felhasználó hozzáférésre van szüksége.</span><span class="sxs-lookup"><span data-stu-id="f7477-111">In order to authenticate with Azure AD, an application needs access to the user's directory.</span></span> <span data-ttu-id="f7477-112">Legalább az alkalmazást kell olvasni a felhasználói profil.</span><span class="sxs-lookup"><span data-stu-id="f7477-112">At a minimum, the application needs permission to read the user's profile.</span></span> <span data-ttu-id="f7477-113">Az első alkalommal a felhasználó bejelentkezik, az Azure AD egy hozzájárulási oldal, amely megjeleníti a kért engedélyeket jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="f7477-113">The first time that a user signs in, Azure AD shows a consent page that lists the permissions being requested.</span></span> <span data-ttu-id="f7477-114">Kattintva **elfogadás**, a felhasználó engedélyt ad az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="f7477-114">By clicking **Accept**, the user grants permission to the application.</span></span>

<span data-ttu-id="f7477-115">Alapértelmezés szerint engedély felhasználónkénti alapon.</span><span class="sxs-lookup"><span data-stu-id="f7477-115">By default, consent is granted on a per-user basis.</span></span> <span data-ttu-id="f7477-116">Minden felhasználó számára kérelmezni a hozzájárulási oldalon láthatja.</span><span class="sxs-lookup"><span data-stu-id="f7477-116">Every user who signs in sees the consent page.</span></span> <span data-ttu-id="f7477-117">Azonban, hogy az Azure AD támogatja-e is *admin hozzájárulási*, amely lehetővé teszi, hogy a rendszergazda számára, hogy a teljes szervezet hozzájárulás AD.</span><span class="sxs-lookup"><span data-stu-id="f7477-117">However, Azure AD also supports  *admin consent*, which allows an AD administrator to consent for an entire organization.</span></span>

<span data-ttu-id="f7477-118">A rendszergazda hozzájárulási folyamat használata esetén a hozzájárulási lap szerint az, hogy az AD felügyeleti engedélyt a teljes bérlő nevében:</span><span class="sxs-lookup"><span data-stu-id="f7477-118">When the admin consent flow is used, the consent page states that the AD admin is granting permission on behalf of the entire tenant:</span></span>

![Felügyeleti beleegyezést kérő üzenete](./images/admin-consent.png)

<span data-ttu-id="f7477-120">Miután rákattint a rendszergazda **elfogadás**, ugyanannak a bérlőnek belüli más felhasználók bejelentkezhetnek, és az Azure AD kihagyja a hozzájárulási képernyő.</span><span class="sxs-lookup"><span data-stu-id="f7477-120">After the admin clicks **Accept**, other users within the same tenant can sign in, and Azure AD will skip the consent screen.</span></span>

<span data-ttu-id="f7477-121">Csak AD a rendszergazda biztosíthat a rendszergazda jóváhagyását, mert a teljes szervezet nevében engedélyt.</span><span class="sxs-lookup"><span data-stu-id="f7477-121">Only an AD administrator can give admin consent, because it grants permission on behalf of the entire organization.</span></span> <span data-ttu-id="f7477-122">Ha egy nem rendszergazdai próbálkozik a rendszergazda jóváhagyását folyamatot, az Azure AD egy hibaüzenet jelenik meg:</span><span class="sxs-lookup"><span data-stu-id="f7477-122">If a non-administrator tries to authenticate with the admin consent flow, Azure AD displays an error:</span></span>

![Hozzájárulás hiba](./images/consent-error.png)

<span data-ttu-id="f7477-124">Ha az alkalmazás később további engedélyek, az ügyfél jelentkezzen újra, és hogy a frissített engedélyeket kell.</span><span class="sxs-lookup"><span data-stu-id="f7477-124">If the application requires additional permissions at a later point, the customer will need to sign up again and consent to the updated permissions.</span></span>  

## <a name="implementing-tenant-sign-up"></a><span data-ttu-id="f7477-125">A bérlő előfizetés megvalósítása</span><span class="sxs-lookup"><span data-stu-id="f7477-125">Implementing tenant sign-up</span></span>
<span data-ttu-id="f7477-126">Az a [Dejójáték felmérések] [ Tailspin] alkalmazás, a regisztrációs folyamat több követelményei meghatározott:</span><span class="sxs-lookup"><span data-stu-id="f7477-126">For the [Tailspin Surveys][Tailspin] application,  we defined several requirements for the sign-up process:</span></span>

* <span data-ttu-id="f7477-127">A bérlő kell iratkozzon fel, mielőtt a felhasználók bejelentkezhetnek.</span><span class="sxs-lookup"><span data-stu-id="f7477-127">A tenant must sign up before users can sign in.</span></span>
* <span data-ttu-id="f7477-128">Regisztráció az admin hozzájárulási folyamat használja.</span><span class="sxs-lookup"><span data-stu-id="f7477-128">Sign-up uses the admin consent flow.</span></span>
* <span data-ttu-id="f7477-129">Az adatbázis-előfizetési hozzáadja a felhasználó bérlői.</span><span class="sxs-lookup"><span data-stu-id="f7477-129">Sign-up adds the user's tenant to the application database.</span></span>
* <span data-ttu-id="f7477-130">A bérlő előfizet, miután a az alkalmazás egy bevezetési lapját jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="f7477-130">After a tenant signs up, the application shows an onboarding page.</span></span>

<span data-ttu-id="f7477-131">Ebben a szakaszban végigvesszük az a regisztrációs folyamat végrehajtása.</span><span class="sxs-lookup"><span data-stu-id="f7477-131">In this section, we'll walk through our implementation of the sign-up process.</span></span>
<span data-ttu-id="f7477-132">Fontos tudni, hogy "regisztrációs" és "bejelentkezés" van egy alkalmazás fogalom.</span><span class="sxs-lookup"><span data-stu-id="f7477-132">It's important to understand that "sign up" versus "sign in" is an application concept.</span></span> <span data-ttu-id="f7477-133">A hitelesítési folyamat során az Azure AD nem eredendően tudja, hogy a felhasználó regisztrál javításán.</span><span class="sxs-lookup"><span data-stu-id="f7477-133">During the authentication flow, Azure AD does not inherently know whether the user is in process of signing up.</span></span> <span data-ttu-id="f7477-134">A környezet nyomon követéséhez az alkalmazáshoz van.</span><span class="sxs-lookup"><span data-stu-id="f7477-134">It's up to the application to keep track of the context.</span></span>

<span data-ttu-id="f7477-135">Névtelen felhasználó meglátogat felmérések alkalmazása, a felhasználó esetén látható két gomb, egy való bejelentkezéshez, és egy "regisztrálása a vállalati" (regisztrálás).</span><span class="sxs-lookup"><span data-stu-id="f7477-135">When an anonymous user visits the Surveys application, the user is shown two buttons, one to sign in, and one to "enroll your company" (sign up).</span></span>

![Alkalmazás-előfizetési lap](./images/sign-up-page.png)

<span data-ttu-id="f7477-137">Az ilyen gombokat a műveletek meghívása a `AccountController` osztály.</span><span class="sxs-lookup"><span data-stu-id="f7477-137">These buttons invoke actions in the `AccountController` class.</span></span>

<span data-ttu-id="f7477-138">A `SignIn` művelet értéket ad vissza egy **ChallegeResult**, amelyek hatására az OpenID Connect köztes átirányítani a hitelesítési végpontra.</span><span class="sxs-lookup"><span data-stu-id="f7477-138">The `SignIn` action returns a **ChallegeResult**, which causes the OpenID Connect middleware to redirect to the authentication endpoint.</span></span> <span data-ttu-id="f7477-139">Ez az eseményindító hitelesítés az ASP.NET Core az alapértelmezett lehetőség.</span><span class="sxs-lookup"><span data-stu-id="f7477-139">This is the default way to trigger authentication in ASP.NET Core.</span></span>  

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

<span data-ttu-id="f7477-140">Most hasonlítsa össze a `SignUp` művelet:</span><span class="sxs-lookup"><span data-stu-id="f7477-140">Now compare the `SignUp` action:</span></span>

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

<span data-ttu-id="f7477-141">Például `SignIn`, a `SignUp` művelet is adja vissza egy `ChallengeResult`.</span><span class="sxs-lookup"><span data-stu-id="f7477-141">Like `SignIn`, the `SignUp` action also returns a `ChallengeResult`.</span></span> <span data-ttu-id="f7477-142">Most, azt hozzá egy adatait az adat a `AuthenticationProperties` a a `ChallengeResult`:</span><span class="sxs-lookup"><span data-stu-id="f7477-142">But this time, we add a piece of state information to the `AuthenticationProperties` in the `ChallengeResult`:</span></span>

* <span data-ttu-id="f7477-143">regisztráció: egy logikai jelző, amely jelzi, hogy a felhasználó elindult-e a regisztrációs folyamat.</span><span class="sxs-lookup"><span data-stu-id="f7477-143">signup: A Boolean flag, indicating that the user has started the sign-up process.</span></span>

<span data-ttu-id="f7477-144">Az állapotadatokat az `AuthenticationProperties` bekerül az OpenID Connect [állapot] paraméter, amely kerekíteni való adatváltások számát a hitelesítési folyamat során.</span><span class="sxs-lookup"><span data-stu-id="f7477-144">The state information in `AuthenticationProperties` gets added to the OpenID Connect [state] parameter, which round trips during the authentication flow.</span></span>

![Állapot paraméter](./images/state-parameter.png)

<span data-ttu-id="f7477-146">Miután a felhasználó az Azure AD hitelesíti, és lekérdezi visszairányítja az alkalmazást, a hitelesítési jegy állapotát tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="f7477-146">After the user authenticates in Azure AD and gets redirected back to the application, the authentication ticket contains the state.</span></span> <span data-ttu-id="f7477-147">Győződjön meg arról, hogy a "signup" érték továbbra is fennáll, az egész hitelesítési folyamat között a használjuk a tény.</span><span class="sxs-lookup"><span data-stu-id="f7477-147">We are using this fact to make sure the "signup" value persists across the entire authentication flow.</span></span>

## <a name="adding-the-admin-consent-prompt"></a><span data-ttu-id="f7477-148">A rendszergazda beleegyezést kérő üzenete hozzáadása</span><span class="sxs-lookup"><span data-stu-id="f7477-148">Adding the admin consent prompt</span></span>
<span data-ttu-id="f7477-149">Az Azure ad-ben a rendszergazda hozzájárulási folyamat elindítja a sématelepítési "prompt" paramétert adna a lekérdezési karakterláncot a hitelesítési kérelem:</span><span class="sxs-lookup"><span data-stu-id="f7477-149">In Azure AD, the admin consent flow is triggered by adding a "prompt" parameter to the query string in the authentication request:</span></span>

```
/authorize?prompt=admin_consent&...
```

<span data-ttu-id="f7477-150">A felmérések alkalmazás hozzáadása során a parancssorba a `RedirectToAuthenticationEndpoint` esemény.</span><span class="sxs-lookup"><span data-stu-id="f7477-150">The Surveys application adds the prompt during the `RedirectToAuthenticationEndpoint` event.</span></span> <span data-ttu-id="f7477-151">Ez az esemény jobb nevezik, előtt a köztes átirányítja a felhasználókat a hitelesítési végpontra.</span><span class="sxs-lookup"><span data-stu-id="f7477-151">This event is called right before the middleware redirects to the authentication endpoint.</span></span>

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

<span data-ttu-id="f7477-152">Beállítás` ProtocolMessage.Prompt` közli a közbenső szoftvert adja hozzá a "kérdés" paramétert a hitelesítési kérelmet.</span><span class="sxs-lookup"><span data-stu-id="f7477-152">Setting` ProtocolMessage.Prompt` tells the middleware to add the "prompt" parameter to the authentication request.</span></span>

<span data-ttu-id="f7477-153">Vegye figyelembe, hogy a kérés csak akkor szükséges, regisztráció során.</span><span class="sxs-lookup"><span data-stu-id="f7477-153">Note that the prompt is only needed during sign-up.</span></span> <span data-ttu-id="f7477-154">Rendszeres bejelentkezés nem foglalja azt.</span><span class="sxs-lookup"><span data-stu-id="f7477-154">Regular sign-in should not include it.</span></span> <span data-ttu-id="f7477-155">Különbségtétel, a ellenőrizzük a `signup` hitelesítési állapot értéke.</span><span class="sxs-lookup"><span data-stu-id="f7477-155">To distinguish between them, we check for the `signup` value in the authentication state.</span></span> <span data-ttu-id="f7477-156">A következő kiterjesztésmetódus ellenőrzi a feltétel:</span><span class="sxs-lookup"><span data-stu-id="f7477-156">The following extension method checks for this condition:</span></span>

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

## <a name="registering-a-tenant"></a><span data-ttu-id="f7477-157">A bérlő regisztrálása</span><span class="sxs-lookup"><span data-stu-id="f7477-157">Registering a Tenant</span></span>
<span data-ttu-id="f7477-158">A felmérések alkalmazás mindegyik bérlő kapcsolatos információkat és a felhasználó az alkalmazás-adatbázisban tárolja.</span><span class="sxs-lookup"><span data-stu-id="f7477-158">The Surveys application stores some information about each tenant and user in the application database.</span></span>

![Bérlői tábla](./images/tenant-table.png)

<span data-ttu-id="f7477-160">A bérlői tábla IssuerValue az érték a kibocsátó jogcím a bérlő számára.</span><span class="sxs-lookup"><span data-stu-id="f7477-160">In the Tenant table, IssuerValue is the value of the issuer claim for the tenant.</span></span> <span data-ttu-id="f7477-161">Ez az az Azure AD `https://sts.windows.net/<tentantID>` és bérlőnként egy egyedi értéket ad.</span><span class="sxs-lookup"><span data-stu-id="f7477-161">For Azure AD, this is `https://sts.windows.net/<tentantID>` and gives a unique value per tenant.</span></span>

<span data-ttu-id="f7477-162">Amikor egy új bérlő előfizet, a felmérések alkalmazás bérlői rekordot ír az adatbázis.</span><span class="sxs-lookup"><span data-stu-id="f7477-162">When a new tenant signs up, the Surveys application writes a tenant record to the database.</span></span> <span data-ttu-id="f7477-163">Ez akkor történik meg belül a `AuthenticationValidated` esemény.</span><span class="sxs-lookup"><span data-stu-id="f7477-163">This happens inside the `AuthenticationValidated` event.</span></span> <span data-ttu-id="f7477-164">(Ne elvégezhető előtt ezt az eseményt, mert a azonosító jogkivonat nem érvényesítése még, így a jogcímértékek nem megbízható.</span><span class="sxs-lookup"><span data-stu-id="f7477-164">(Don't do it before this event, because the ID token won't be validated yet, so you can't trust the claim values.</span></span> <span data-ttu-id="f7477-165">Lásd: [hitelesítési].</span><span class="sxs-lookup"><span data-stu-id="f7477-165">See [Authentication].</span></span>

<span data-ttu-id="f7477-166">Ez a megfelelő kódot a felmérések alkalmazásból:</span><span class="sxs-lookup"><span data-stu-id="f7477-166">Here is the relevant code from the Surveys application:</span></span>

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

<span data-ttu-id="f7477-167">Ez a kód a következőket teszi:</span><span class="sxs-lookup"><span data-stu-id="f7477-167">This code does the following:</span></span>

1. <span data-ttu-id="f7477-168">Ellenőrzés, hogy a bérlő kibocsátó értéket már az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="f7477-168">Check if the tenant's issuer value is already in the database.</span></span> <span data-ttu-id="f7477-169">Ha a bérlő nem írta alá, `FindByIssuerValueAsync` null értéket ad vissza.</span><span class="sxs-lookup"><span data-stu-id="f7477-169">If the tenant has not signed up, `FindByIssuerValueAsync` returns null.</span></span>
2. <span data-ttu-id="f7477-170">Ha a felhasználó regisztrál:</span><span class="sxs-lookup"><span data-stu-id="f7477-170">If the user is signing up:</span></span>
   1. <span data-ttu-id="f7477-171">A bérlő hozzáadása az adatbázishoz (`SignUpTenantAsync`).</span><span class="sxs-lookup"><span data-stu-id="f7477-171">Add the tenant to the database (`SignUpTenantAsync`).</span></span>
   2. <span data-ttu-id="f7477-172">A hitelesített felhasználó hozzáadása az adatbázishoz (`CreateOrUpdateUserAsync`).</span><span class="sxs-lookup"><span data-stu-id="f7477-172">Add the authenticated user to the database (`CreateOrUpdateUserAsync`).</span></span>
3. <span data-ttu-id="f7477-173">Ellenkező esetben hajtsa végre a szokásos bejelentkezési folyamata:</span><span class="sxs-lookup"><span data-stu-id="f7477-173">Otherwise complete the normal sign-in flow:</span></span>
   1. <span data-ttu-id="f7477-174">Ha a bérlő kibocsátó nem található az adatbázisban, akkor a bérlő nincs regisztrálva, és regisztrálni kell az ügyfél.</span><span class="sxs-lookup"><span data-stu-id="f7477-174">If the tenant's issuer was not found in the database, it means the tenant is not registered, and the customer needs to sign up.</span></span> <span data-ttu-id="f7477-175">Ebben az esetben idéznie egy kivételt a hitelesítés sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="f7477-175">In that case, throw an exception to cause the authentication to fail.</span></span>
   2. <span data-ttu-id="f7477-176">Máskülönben hozzon létre egy adatbázis-rekord ehhez a felhasználóhoz, ha nincs ilyen már (`CreateOrUpdateUserAsync`).</span><span class="sxs-lookup"><span data-stu-id="f7477-176">Otherwise, create a database record for this user, if there isn't one already (`CreateOrUpdateUserAsync`).</span></span>

<span data-ttu-id="f7477-177">Itt a `SignUpTenantAsync` módszer, amely a bérlői hozzáadása az adatbázishoz.</span><span class="sxs-lookup"><span data-stu-id="f7477-177">Here is the `SignUpTenantAsync` method that adds the tenant to the database.</span></span>

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

<span data-ttu-id="f7477-178">Ez a teljes regisztrációs folyamat az felmérések alkalmazásban összefoglalása:</span><span class="sxs-lookup"><span data-stu-id="f7477-178">Here is a summary of the entire sign-up flow in the Surveys application:</span></span>

1. <span data-ttu-id="f7477-179">A felhasználó kattint a **regisztráció** gombra.</span><span class="sxs-lookup"><span data-stu-id="f7477-179">The user clicks the **Sign Up** button.</span></span>
2. <span data-ttu-id="f7477-180">A `AccountController.SignUp` művelet challege eredményt adja vissza.</span><span class="sxs-lookup"><span data-stu-id="f7477-180">The `AccountController.SignUp` action returns a challege result.</span></span>  <span data-ttu-id="f7477-181">A hitelesítési állapot "signup" értéket tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="f7477-181">The authentication state includes "signup" value.</span></span>
3. <span data-ttu-id="f7477-182">Az a `RedirectToAuthenticationEndpoint` esemény, vegye fel a `admin_consent` kérdés.</span><span class="sxs-lookup"><span data-stu-id="f7477-182">In the `RedirectToAuthenticationEndpoint` event, add the `admin_consent` prompt.</span></span>
4. <span data-ttu-id="f7477-183">Az OpenID Connect köztes átirányítja az Azure AD és a felhasználó hitelesíti magát.</span><span class="sxs-lookup"><span data-stu-id="f7477-183">The OpenID Connect middleware redirects to Azure AD and the user authenticates.</span></span>
5. <span data-ttu-id="f7477-184">Az a `AuthenticationValidated` esemény, keresse meg a "signup" állapotban.</span><span class="sxs-lookup"><span data-stu-id="f7477-184">In the `AuthenticationValidated` event, look for the "signup" state.</span></span>
6. <span data-ttu-id="f7477-185">A bérlő hozzáadása az adatbázishoz.</span><span class="sxs-lookup"><span data-stu-id="f7477-185">Add the tenant to the database.</span></span>

<span data-ttu-id="f7477-186">[**Következő**][app roles]</span><span class="sxs-lookup"><span data-stu-id="f7477-186">[**Next**][app roles]</span></span>

<!-- Links -->
[app roles]: app-roles.md
[Tailspin]: tailspin.md

[állapot]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[state]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[hitelesítési]: authenticate.md
[Authentication]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
