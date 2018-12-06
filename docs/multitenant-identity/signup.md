---
title: Regisztráció és a bérlők felvétele több-bérlős alkalmazásokban
description: Hogyan kell előkészíteni bérlők egy több-bérlős alkalmazásban
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: 541a4dd9abb2168eef4a60a0ec99e1e7c06049b5
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902476"
---
# <a name="tenant-sign-up-and-onboarding"></a><span data-ttu-id="2220c-103">Bérlői feliratkozás és előkészítés</span><span class="sxs-lookup"><span data-stu-id="2220c-103">Tenant sign-up and onboarding</span></span>

<span data-ttu-id="2220c-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="2220c-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="2220c-105">Ez a cikk bemutatja, hogyan valósíthat meg egy *előfizetési* a több-bérlős alkalmazás, amely lehetővé teszi, hogy egy ügyfél az alkalmazás szervezeti regisztráció folyamatát.</span><span class="sxs-lookup"><span data-stu-id="2220c-105">This article describes how to implement a *sign-up* process in a multi-tenant application, which allows a customer to sign up their organization for your application.</span></span>
<span data-ttu-id="2220c-106">A regisztrációs folyamat végrehajtásához több oka is van:</span><span class="sxs-lookup"><span data-stu-id="2220c-106">There are several reasons to implement a sign-up process:</span></span>

* <span data-ttu-id="2220c-107">AD rendszergazdai jóváhagyást az ügyfél a teljes cég számára, az alkalmazás használatának engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="2220c-107">Allow an AD admin to consent for the customer's entire organization to use the application.</span></span>
* <span data-ttu-id="2220c-108">Hitelkártyás fizetés vagy más ügyfél-információkat gyűjtése.</span><span class="sxs-lookup"><span data-stu-id="2220c-108">Collect credit card payment or other customer information.</span></span>
* <span data-ttu-id="2220c-109">Bármely az alkalmazás által egyszeri bérlőnkénti telepítést végre.</span><span class="sxs-lookup"><span data-stu-id="2220c-109">Perform any one-time per-tenant setup needed by your application.</span></span>

## <a name="admin-consent-and-azure-ad-permissions"></a><span data-ttu-id="2220c-110">Rendszergazdai jóváhagyás és az Azure AD-engedélyekről</span><span class="sxs-lookup"><span data-stu-id="2220c-110">Admin consent and Azure AD permissions</span></span>
<span data-ttu-id="2220c-111">Annak érdekében, hogy az Azure AD-hitelesítést, az alkalmazás a felhasználó hozzá kell férnie.</span><span class="sxs-lookup"><span data-stu-id="2220c-111">In order to authenticate with Azure AD, an application needs access to the user's directory.</span></span> <span data-ttu-id="2220c-112">Legalább az alkalmazás a felhasználói profil olvasása engedélyre van szüksége.</span><span class="sxs-lookup"><span data-stu-id="2220c-112">At a minimum, the application needs permission to read the user's profile.</span></span> <span data-ttu-id="2220c-113">Egy felhasználó bejelentkezik, először az Azure AD egy hozzájárulást kérő lap, amely megjeleníti a kért engedélyeket jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="2220c-113">The first time that a user signs in, Azure AD shows a consent page that lists the permissions being requested.</span></span> <span data-ttu-id="2220c-114">Kattintva **elfogadás**, a felhasználó engedélyt ad az alkalmazásnak.</span><span class="sxs-lookup"><span data-stu-id="2220c-114">By clicking **Accept**, the user grants permission to the application.</span></span>

<span data-ttu-id="2220c-115">Alapértelmezés szerint engedély felhasználónkénti alapon.</span><span class="sxs-lookup"><span data-stu-id="2220c-115">By default, consent is granted on a per-user basis.</span></span> <span data-ttu-id="2220c-116">Minden felhasználó, aki bejelentkezik a hozzájárulást kérő lap fog látni.</span><span class="sxs-lookup"><span data-stu-id="2220c-116">Every user who signs in sees the consent page.</span></span> <span data-ttu-id="2220c-117">Azonban, hogy az Azure AD támogatja-e is *rendszergazdai jóváhagyás*, amely lehetővé teszi, hogy egy AD-rendszergazdát, hogy engedélyt adjanak a teljes szervezet számára.</span><span class="sxs-lookup"><span data-stu-id="2220c-117">However, Azure AD also supports  *admin consent*, which allows an AD administrator to consent for an entire organization.</span></span>

<span data-ttu-id="2220c-118">Ha a rendszergazdai jóváhagyási folyamatot használja, a hozzájárulást kérő lap állapotok a, hogy az AD-rendszergazda a teljes bérlő nevében engedélyt biztosít:</span><span class="sxs-lookup"><span data-stu-id="2220c-118">When the admin consent flow is used, the consent page states that the AD admin is granting permission on behalf of the entire tenant:</span></span>

![Rendszergazdai jóváhagyás kérése](./images/admin-consent.png)

<span data-ttu-id="2220c-120">Miután a rendszergazda kattint **elfogadás**, az ugyanazon bérlőn belüli más felhasználók bejelentkezhetnek, és az Azure AD ezért kihagyja a jóváhagyást kérő képernyőt.</span><span class="sxs-lookup"><span data-stu-id="2220c-120">After the admin clicks **Accept**, other users within the same tenant can sign in, and Azure AD will skip the consent screen.</span></span>

<span data-ttu-id="2220c-121">Csak egy AD-rendszergazda engedélyezheti a rendszergazda, mert a teljes szervezet nevében engedélyt.</span><span class="sxs-lookup"><span data-stu-id="2220c-121">Only an AD administrator can give admin consent, because it grants permission on behalf of the entire organization.</span></span> <span data-ttu-id="2220c-122">Ha nem rendszergazdai próbálkozik a rendszergazdai jóváhagyási folyamatot, az Azure AD egy hibaüzenet jelenik meg:</span><span class="sxs-lookup"><span data-stu-id="2220c-122">If a non-administrator tries to authenticate with the admin consent flow, Azure AD displays an error:</span></span>

![Jóváhagyás hiba](./images/consent-error.png)

<span data-ttu-id="2220c-124">Ha az alkalmazás későbbi időpontban további engedélyeket igényel, az ügyfél kell újra regisztráljon, és hozzájárul az engedélyekkel.</span><span class="sxs-lookup"><span data-stu-id="2220c-124">If the application requires additional permissions at a later point, the customer will need to sign up again and consent to the updated permissions.</span></span>  

## <a name="implementing-tenant-sign-up"></a><span data-ttu-id="2220c-125">Bérlői feliratkozás megvalósítása</span><span class="sxs-lookup"><span data-stu-id="2220c-125">Implementing tenant sign-up</span></span>
<span data-ttu-id="2220c-126">Az a [Tailspin Surveys] [ Tailspin] alkalmazás, a regisztrációs folyamat számos követelményei meghatározott:</span><span class="sxs-lookup"><span data-stu-id="2220c-126">For the [Tailspin Surveys][Tailspin] application,  we defined several requirements for the sign-up process:</span></span>

* <span data-ttu-id="2220c-127">Egy bérlő regisztrálás felhasználóknak a bejelentkezéshez.</span><span class="sxs-lookup"><span data-stu-id="2220c-127">A tenant must sign up before users can sign in.</span></span>
* <span data-ttu-id="2220c-128">Regisztráció a rendszergazdai jóváhagyási folyamatot használ.</span><span class="sxs-lookup"><span data-stu-id="2220c-128">Sign-up uses the admin consent flow.</span></span>
* <span data-ttu-id="2220c-129">Az alkalmazás-adatbázis-előfizetési hozzáadja a felhasználó bérlőjéhez.</span><span class="sxs-lookup"><span data-stu-id="2220c-129">Sign-up adds the user's tenant to the application database.</span></span>
* <span data-ttu-id="2220c-130">Miután egy bérlő regisztrál, a az alkalmazás egy előkészítési lapját jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="2220c-130">After a tenant signs up, the application shows an onboarding page.</span></span>

<span data-ttu-id="2220c-131">Ebben a szakaszban végigvesszük megvalósítása a regisztrációs folyamat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="2220c-131">In this section, we'll walk through our implementation of the sign-up process.</span></span>
<span data-ttu-id="2220c-132">Fontos tudni, hogy a "regisztráció" és "sign-in" van egy alkalmazás fogalom.</span><span class="sxs-lookup"><span data-stu-id="2220c-132">It's important to understand that "sign up" versus "sign in" is an application concept.</span></span> <span data-ttu-id="2220c-133">A hitelesítési folyamat során az Azure AD nem természetüknél fogva tudja, hogy a felhasználó éppen regisztráló.</span><span class="sxs-lookup"><span data-stu-id="2220c-133">During the authentication flow, Azure AD does not inherently know whether the user is in process of signing up.</span></span> <span data-ttu-id="2220c-134">Az alkalmazás a helyi nyomon követéséhez van.</span><span class="sxs-lookup"><span data-stu-id="2220c-134">It's up to the application to keep track of the context.</span></span>

<span data-ttu-id="2220c-135">Névtelen felhasználó meglátogat a Surveys alkalmazás, amikor a felhasználó nem látható két gombot, egy bejelentkezni, és egy "regisztrálni a vállalat" (regisztrálás).</span><span class="sxs-lookup"><span data-stu-id="2220c-135">When an anonymous user visits the Surveys application, the user is shown two buttons, one to sign in, and one to "enroll your company" (sign up).</span></span>

![Alkalmazás-előfizetési lap](./images/sign-up-page.png)

<span data-ttu-id="2220c-137">Ezekre a gombokra hajthatók végre műveletek, az a `AccountController` osztály.</span><span class="sxs-lookup"><span data-stu-id="2220c-137">These buttons invoke actions in the `AccountController` class.</span></span>

<span data-ttu-id="2220c-138">A `SignIn` művelet értéket ad vissza egy **ChallegeResult**, amely hatására az OpenID Connect közbenső szoftvert átirányítani a hitelesítési végpontra.</span><span class="sxs-lookup"><span data-stu-id="2220c-138">The `SignIn` action returns a **ChallegeResult**, which causes the OpenID Connect middleware to redirect to the authentication endpoint.</span></span> <span data-ttu-id="2220c-139">Ez az eseményindító-hitelesítés az ASP.NET Core alapértelmezett módja.</span><span class="sxs-lookup"><span data-stu-id="2220c-139">This is the default way to trigger authentication in ASP.NET Core.</span></span>  

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

<span data-ttu-id="2220c-140">Hasonlítsa össze a `SignUp` művelet:</span><span class="sxs-lookup"><span data-stu-id="2220c-140">Now compare the `SignUp` action:</span></span>

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

<span data-ttu-id="2220c-141">Például `SignIn`, a `SignUp` művelet is adja vissza egy `ChallengeResult`.</span><span class="sxs-lookup"><span data-stu-id="2220c-141">Like `SignIn`, the `SignUp` action also returns a `ChallengeResult`.</span></span> <span data-ttu-id="2220c-142">Ebben az esetben hozzáadunk egy részét az állapotinformációkat, de a `AuthenticationProperties` a a `ChallengeResult`:</span><span class="sxs-lookup"><span data-stu-id="2220c-142">But this time, we add a piece of state information to the `AuthenticationProperties` in the `ChallengeResult`:</span></span>

* <span data-ttu-id="2220c-143">regisztráció: egy logikai jelzőt, amely jelzi, hogy a felhasználó elindult-e a regisztrációs folyamat.</span><span class="sxs-lookup"><span data-stu-id="2220c-143">signup: A Boolean flag, indicating that the user has started the sign-up process.</span></span>

<span data-ttu-id="2220c-144">Az állapotinformációt `AuthenticationProperties` lekérdezi hozzáadni az OpenID Connect [állapot] paramétert, amelyre kerekíteni lelassítja a hitelesítési folyamat során.</span><span class="sxs-lookup"><span data-stu-id="2220c-144">The state information in `AuthenticationProperties` gets added to the OpenID Connect [state] parameter, which round trips during the authentication flow.</span></span>

![Állapot paraméter](./images/state-parameter.png)

<span data-ttu-id="2220c-146">Miután a felhasználó hitelesíti magát az Azure ad-ben, és átirányítja az alkalmazásnak, a hitelesítési jegy állapotát tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="2220c-146">After the user authenticates in Azure AD and gets redirected back to the application, the authentication ticket contains the state.</span></span> <span data-ttu-id="2220c-147">Ez a tény, hogy a "signup" érték továbbra is fennáll, az egész hitelesítési folyamat között használjuk.</span><span class="sxs-lookup"><span data-stu-id="2220c-147">We are using this fact to make sure the "signup" value persists across the entire authentication flow.</span></span>

## <a name="adding-the-admin-consent-prompt"></a><span data-ttu-id="2220c-148">A rendszergazda beleegyezést kérő hozzáadása</span><span class="sxs-lookup"><span data-stu-id="2220c-148">Adding the admin consent prompt</span></span>
<span data-ttu-id="2220c-149">Az Azure AD-ben a rendszergazdai jóváhagyás folyamathoz "parancssor" paraméter hozzáadása a lekérdezési karakterláncot a hitelesítési kérelem által aktivált:</span><span class="sxs-lookup"><span data-stu-id="2220c-149">In Azure AD, the admin consent flow is triggered by adding a "prompt" parameter to the query string in the authentication request:</span></span>

```
/authorize?prompt=admin_consent&...
```

<span data-ttu-id="2220c-150">A Surveys alkalmazás hozzáadása során a rendszer kéri a `RedirectToAuthenticationEndpoint` esemény.</span><span class="sxs-lookup"><span data-stu-id="2220c-150">The Surveys application adds the prompt during the `RedirectToAuthenticationEndpoint` event.</span></span> <span data-ttu-id="2220c-151">Ez az esemény jobb neve előtt a közbenső szoftver a hitelesítési végpontra irányítja át.</span><span class="sxs-lookup"><span data-stu-id="2220c-151">This event is called right before the middleware redirects to the authentication endpoint.</span></span>

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

<span data-ttu-id="2220c-152">Beállítás` ProtocolMessage.Prompt` arra utasítja a közbenső szoftverek, a "kérdés" paraméter hozzáadása a hitelesítési kérelmet.</span><span class="sxs-lookup"><span data-stu-id="2220c-152">Setting` ProtocolMessage.Prompt` tells the middleware to add the "prompt" parameter to the authentication request.</span></span>

<span data-ttu-id="2220c-153">Vegye figyelembe, hogy a rendszer csak akkor van szükség a regisztrációhoz.</span><span class="sxs-lookup"><span data-stu-id="2220c-153">Note that the prompt is only needed during sign-up.</span></span> <span data-ttu-id="2220c-154">Rendszeres bejelentkezés nem tartalmaznia kell azt.</span><span class="sxs-lookup"><span data-stu-id="2220c-154">Regular sign-in should not include it.</span></span> <span data-ttu-id="2220c-155">Megkülönböztetni őket, ellenőrzése a `signup` hitelesítési állapot értéke.</span><span class="sxs-lookup"><span data-stu-id="2220c-155">To distinguish between them, we check for the `signup` value in the authentication state.</span></span> <span data-ttu-id="2220c-156">Ezt az állapotot ellenőrzi a következő metódust:</span><span class="sxs-lookup"><span data-stu-id="2220c-156">The following extension method checks for this condition:</span></span>

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

## <a name="registering-a-tenant"></a><span data-ttu-id="2220c-157">A bérlő regisztrációja</span><span class="sxs-lookup"><span data-stu-id="2220c-157">Registering a Tenant</span></span>
<span data-ttu-id="2220c-158">A Surveys alkalmazás minden egyes bérlő némi információt és a felhasználó az alkalmazás-adatbázis tárolja.</span><span class="sxs-lookup"><span data-stu-id="2220c-158">The Surveys application stores some information about each tenant and user in the application database.</span></span>

![Bérlő táblában](./images/tenant-table.png)

<span data-ttu-id="2220c-160">A bérlő táblában IssuerValue az érték a kiállító jogcím a bérlő számára.</span><span class="sxs-lookup"><span data-stu-id="2220c-160">In the Tenant table, IssuerValue is the value of the issuer claim for the tenant.</span></span> <span data-ttu-id="2220c-161">Ez az Azure AD-hez `https://sts.windows.net/<tentantID>` és a egy egyedi értéket bérlőnként.</span><span class="sxs-lookup"><span data-stu-id="2220c-161">For Azure AD, this is `https://sts.windows.net/<tentantID>` and gives a unique value per tenant.</span></span>

<span data-ttu-id="2220c-162">Amikor egy új bérlő előfizet, a Surveys alkalmazás bérlői rekordot ír az adatbázis.</span><span class="sxs-lookup"><span data-stu-id="2220c-162">When a new tenant signs up, the Surveys application writes a tenant record to the database.</span></span> <span data-ttu-id="2220c-163">Ez belül történik a `AuthenticationValidated` esemény.</span><span class="sxs-lookup"><span data-stu-id="2220c-163">This happens inside the `AuthenticationValidated` event.</span></span> <span data-ttu-id="2220c-164">(Ne végezze el, ez az esemény előtt, mert a azonosító jogkivonat nem érvényesíthető, így nem megbízható a jogcímértékek.</span><span class="sxs-lookup"><span data-stu-id="2220c-164">(Don't do it before this event, because the ID token won't be validated yet, so you can't trust the claim values.</span></span> <span data-ttu-id="2220c-165">Lásd: [Hitelesítés].</span><span class="sxs-lookup"><span data-stu-id="2220c-165">See [Authentication].</span></span>

<span data-ttu-id="2220c-166">A megfelelő kódot a Surveys alkalmazás az itt látható:</span><span class="sxs-lookup"><span data-stu-id="2220c-166">Here is the relevant code from the Surveys application:</span></span>

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

<span data-ttu-id="2220c-167">Ez a kód a következőket teszi:</span><span class="sxs-lookup"><span data-stu-id="2220c-167">This code does the following:</span></span>

1. <span data-ttu-id="2220c-168">Ellenőrizze, hogy ha a bérlő kibocsátó értékét az adatbázisban már van.</span><span class="sxs-lookup"><span data-stu-id="2220c-168">Check if the tenant's issuer value is already in the database.</span></span> <span data-ttu-id="2220c-169">Ha a bérlő nem jelentkezett be, `FindByIssuerValueAsync` null értéket ad vissza.</span><span class="sxs-lookup"><span data-stu-id="2220c-169">If the tenant has not signed up, `FindByIssuerValueAsync` returns null.</span></span>
2. <span data-ttu-id="2220c-170">Ha a felhasználó jelentkezik:</span><span class="sxs-lookup"><span data-stu-id="2220c-170">If the user is signing up:</span></span>
   1. <span data-ttu-id="2220c-171">Adja hozzá a bérlő az adatbázishoz (`SignUpTenantAsync`).</span><span class="sxs-lookup"><span data-stu-id="2220c-171">Add the tenant to the database (`SignUpTenantAsync`).</span></span>
   2. <span data-ttu-id="2220c-172">A hitelesített felhasználó hozzáadása az adatbázishoz (`CreateOrUpdateUserAsync`).</span><span class="sxs-lookup"><span data-stu-id="2220c-172">Add the authenticated user to the database (`CreateOrUpdateUserAsync`).</span></span>
3. <span data-ttu-id="2220c-173">Ellenkező esetben a normál bejelentkezési folyamat befejezése:</span><span class="sxs-lookup"><span data-stu-id="2220c-173">Otherwise complete the normal sign-in flow:</span></span>
   1. <span data-ttu-id="2220c-174">Ha a bérlő kibocsátó nem található az adatbázisban, az azt jelenti, a bérlő nincs regisztrálva, és regisztrálnia kell az ügyfélnek.</span><span class="sxs-lookup"><span data-stu-id="2220c-174">If the tenant's issuer was not found in the database, it means the tenant is not registered, and the customer needs to sign up.</span></span> <span data-ttu-id="2220c-175">Ebben az esetben kivételt okoz a hitelesítés sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="2220c-175">In that case, throw an exception to cause the authentication to fail.</span></span>
   2. <span data-ttu-id="2220c-176">Máskülönben hozzon létre egy adatbázis-rekord a felhasználónál, ha nincs ilyen már (`CreateOrUpdateUserAsync`).</span><span class="sxs-lookup"><span data-stu-id="2220c-176">Otherwise, create a database record for this user, if there isn't one already (`CreateOrUpdateUserAsync`).</span></span>

<span data-ttu-id="2220c-177">Íme a `SignUpTenantAsync` metódushoz, amely hozzáad a bérlő az adatbázishoz.</span><span class="sxs-lookup"><span data-stu-id="2220c-177">Here is the `SignUpTenantAsync` method that adds the tenant to the database.</span></span>

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

<span data-ttu-id="2220c-178">Itt látható a teljes regisztrációs folyamatot a Surveys alkalmazás összefoglalása:</span><span class="sxs-lookup"><span data-stu-id="2220c-178">Here is a summary of the entire sign-up flow in the Surveys application:</span></span>

1. <span data-ttu-id="2220c-179">A felhasználó kattint a **regisztráció** gombra.</span><span class="sxs-lookup"><span data-stu-id="2220c-179">The user clicks the **Sign Up** button.</span></span>
2. <span data-ttu-id="2220c-180">A `AccountController.SignUp` művelet challege eredményt adja vissza.</span><span class="sxs-lookup"><span data-stu-id="2220c-180">The `AccountController.SignUp` action returns a challege result.</span></span>  <span data-ttu-id="2220c-181">A hitelesítési állapot "signup" értéket tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="2220c-181">The authentication state includes "signup" value.</span></span>
3. <span data-ttu-id="2220c-182">Az a `RedirectToAuthenticationEndpoint` esemény, adja hozzá a `admin_consent` parancssort.</span><span class="sxs-lookup"><span data-stu-id="2220c-182">In the `RedirectToAuthenticationEndpoint` event, add the `admin_consent` prompt.</span></span>
4. <span data-ttu-id="2220c-183">Az OpenID Connect közbenső szoftvert átirányítja a felhasználókat az Azure ad-ben, és a felhasználó hitelesíti magát.</span><span class="sxs-lookup"><span data-stu-id="2220c-183">The OpenID Connect middleware redirects to Azure AD and the user authenticates.</span></span>
5. <span data-ttu-id="2220c-184">Az a `AuthenticationValidated` esemény, keressen a "signup" állapot.</span><span class="sxs-lookup"><span data-stu-id="2220c-184">In the `AuthenticationValidated` event, look for the "signup" state.</span></span>
6. <span data-ttu-id="2220c-185">Adja hozzá a bérlő az adatbázishoz.</span><span class="sxs-lookup"><span data-stu-id="2220c-185">Add the tenant to the database.</span></span>

<span data-ttu-id="2220c-186">[**Tovább**][app roles]</span><span class="sxs-lookup"><span data-stu-id="2220c-186">[**Next**][app roles]</span></span>

<!-- Links -->
[app roles]: app-roles.md
[Tailspin]: tailspin.md

[állapot]: https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[state]: https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[Hitelesítés]: authenticate.md
[Authentication]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
