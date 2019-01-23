---
title: Egy több-bérlős alkalmazásban a háttéralkalmazás webes API biztonságossá tétele
description: Hogyan teheti biztonságossá a háttéralkalmazás webes API-t.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: a895276a77c111e660f29397d250373bee53f29e
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480768"
---
# <a name="secure-a-backend-web-api"></a>Egy háttéralkalmazás webes API biztonságossá tétele

[![GitHub](../_images/github.png) Mintakód][sample application]

A [Tailspin Surveys] alkalmazás felmérések CRUD-műveletek kezelésére egy háttéralkalmazás webes API-t használ. Például ha egy felhasználó a "Saját felmérések" kattint, a webes alkalmazás HTTP-kérelmet küld a webes API-hoz:

```http
GET /users/{userId}/surveys
```

A webes API-t egy JSON-objektumot ad vissza:

```http
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

A webes API nem engedélyezi a névtelen kérésekkel, így a web app hitelesítenie kell magát az OAuth 2 tulajdonosi jogkivonatok segítségével.

> [!NOTE]
> Ez a forgatókönyv kiszolgálók. Az alkalmazás nem derül összes AJAX hívás a böngésző ügyfélről az API-hoz.

Nincsenek két fő megközelítés közül választhat:

* Felhasználói identitás delegált. A webes alkalmazás hitelesíti a felhasználó identitását.
* Identita aplikace. A webalkalmazás az ügyfél-azonosító, ügyfél-hitelesítő adatok folyamata OAuth2 használatával végzi a hitelesítést.

A Tailspin alkalmazás megvalósítja a felhasználó delegált identitása. Az alábbiakban a fő különbség:

**Delegált felhasználói identitás:**

* A webes API-nak küldött tulajdonosi jogkivonat tartalmazza a felhasználó identitását.
* A webes API lehetővé teszi a felhasználó identitása alapján felhasználását engedélyezési döntésekhez.
* A webalkalmazás 403-as (tiltott) hibáinak kezelése a webes API-t, ha a felhasználó nem jogosult a művelet végrehajtásához szükséges.
* Általában a webalkalmazás továbbra is lehetővé teszi bizonyos felhasználását engedélyezési döntésekhez, amelyek befolyásolják a felhasználói felület, például a Megjelenítés vagy elrejtés a felhasználói felületi elemeket).
* A webes API potenciálisan nem megbízható ügyfelek, például egy JavaScript-alkalmazását vagy egy natív ügyfélalkalmazás által használható.

**Identita aplikace:**

* A webes API-k nem kap a felhasználó adatait.
* A webes API-k nem hajtható végre a felhasználó identitása alapján engedélyezés. Az összes engedélyezési döntéseket a webes alkalmazás.  
* A webes API egy nem megbízható ügyfél (JavaScript- vagy natív ügyfélalkalmazás) nem használható.
* Ez a megközelítés implementálásához némileg egyszerűbb lehet, mert nincs engedélyezési logikai szerepel a webes API-t.

Mindkét megközelítés a webalkalmazás be kell szereznie egy hozzáférési jogkivonatot, amely a webes API-k meghívásához szükséges hitelesítő adatait.

* Delegált felhasználói identitás a jogkivonat kell meghatároznia az Identitásszolgáltató, amely a felhasználó nevében jogkivonatok kiállítása is rendelkezik.
* Az ügyfél-hitelesítő adatok egy alkalmazás előfordulhat, hogy a jogkivonat beszerzése az identitásszolgáltató vagy saját jogkivonat-kiszolgálót. (De nem teljesen új jogkivonat kiszolgálói írása; például egy tesztelt keretrendszert használ [IdentityServer4].) Ha az Azure AD-hitelesítést, Határozottan javasoljuk, hogy a hozzáférési jogkivonat beszerzése az Azure AD, még akkor is az ügyfél-hitelesítő adat folyamatát.

Ez a cikk többi része feltételezi, hogy az Azure AD hitelesíti magát az alkalmazást.

![A hozzáférési jogkivonat beolvasása](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a>A webes API regisztrálása az Azure ad-ben

Ahhoz, hogy a webes API tulajdonosi jogkivonatok kiállítása az Azure AD, a konfigurálása az Azure ad-ben a következőkre kell.

1. A webes API regisztrálása az Azure ad-ben.

2. A webalkalmazás az ügyfél-azonosító hozzáadása a webes API-k alkalmazásjegyzékhez a a `knownClientApplications` tulajdonság. Lásd: [Frissítse az alkalmazásjegyzékeket].

3. Engedélyezheti a webes alkalmazás számára a webes API hívása. Az Azure felügyeleti portálján, az engedélyek két típusú állíthatja be: "Alkalmazásengedélyek" identita aplikace (ügyfél-hitelesítő adat folyamat), vagy a "Delegált engedélyek" a delegált felhasználó identitását.

   ![Delegált engedélyek](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a>Hozzáférési jogkivonat beolvasása

Előtt hívja meg a webes API-t, a webes alkalmazás lekérése hozzáférési token Azure ad-ben. A .NET-alkalmazás, használja a [az Azure AD Authentication Library (ADAL) for .NET][ADAL].

Az OAuth 2 engedélyezési kód folyamata az alkalmazás hozzáférési jogkivonat egy engedélyezési kódjának adatcseréihez használható. A következő kódot használ adal-t beszerezni a hozzáférési jogkivonatot. Ez a kód nevezzük során a `AuthorizationCodeReceived` esemény.

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

Az alábbiakban a különböző paraméterek szükségesek:

* `authority`. A bejelentkezett felhasználó Bérlőazonosítója származik. (Nem a bérlő Azonosítóját az SaaS-szolgáltató)  
* `authorizationCode`. a hitelesítési kódot, amely az identitásszolgáltató vissza lett.
* `clientId`. A webes alkalmazás ügyfél-azonosítót.
* `clientSecret`. A webes alkalmazás titkos ügyfélkódja.
* `redirectUri`. Az átirányítási URI-t, amelyet beállított OpenID connect. Ez a, amelyben az Identitásszolgáltató visszahívja a jogkivonattal.
* `resourceID`. Az Alkalmazásazonosító URI a webes API-t, a webes API Azure AD-ben való regisztrálásakor létrehozott
* `tokenCache`. Gyorsítótárazza a hozzáférési jogkivonatok objektum. Lásd: [Token-gyorsítótárazási].

Ha `AcquireTokenByAuthorizationCodeAsync` sikeres, az adal-t gyorsítótárazza a jogkivonat. Később kérheti a jogkivonatot a gyorsítótárból AcquireTokenSilentAsync meghívásával:

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

ahol `userId` van a felhasználói objektum azonosítója, amely megtalálható a `http://schemas.microsoft.com/identity/claims/objectidentifier` jogcím.

## <a name="using-the-access-token-to-call-the-web-api"></a>A hozzáférési token használatával a webes API meghívásához

Miután a jogkivonatot, küldje el a HTTP-kérések az engedélyezési fejléc a webes API-hoz.

```http
Authorization: Bearer xxxxxxxxxx
```

A következő metódust a Surveys alkalmazás állítja be az engedélyeztetési fejléc egy HTTP-kérelem használatával a **HttpClient** osztály.

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

## <a name="authenticating-in-the-web-api"></a>A webes API hitelesítése

A webes API rendelkezik tulajdonosi jogkivonat hitelesítéséhez. Az ASP.NET Core, használhat a [Microsoft.AspNet.Authentication.JwtBearer] [ JwtBearer] csomagot. Ez a csomag biztosít közbenső szoftvert, amely lehetővé teszi az alkalmazás OpenID Connect tulajdonosi jogkivonatokat fogadni.

A webes API regisztrálása a közbenső szoftver `Startup` osztály.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });

    // ...
}
```

* **Célközönség**. Állítsa be ezt az azonosító URL-címét a webes API-t, a webes API-t az Azure AD-regisztrációja során létrehozott.
* **Szolgáltató**. Több-bérlős alkalmazás esetében állítsa ezt a beállítást `https://login.microsoftonline.com/common/`.
* **TokenValidationParameters**. Egy több-bérlős alkalmazás beállítása **ValidateIssuer** hamis értékre. Ez azt jelenti, hogy az alkalmazás ellenőrzi a kibocsátó.
* **Események** egy osztály, amely származó **JwtBearerEvents**.

### <a name="issuer-validation"></a>Kibocsátó érvényesítése

Az a tokenkiállító ellenőrzését a **JwtBearerEvents.TokenValidated** esemény. A kiállító a "iss" jogcímek zajlik.

A Surveys alkalmazás, az a webes API-k nem kezeli az [bérlői feliratkozás]. Ezért azt csak ellenőrzi, hogy a kibocsátó már az alkalmazás-adatbázis. Ha nem, akkor kivételt jelez, amely-hitelesítés sikertelen.

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

Mivel ez a példa bemutatja, is használhatja a **TokenValidated** eseményhez, és módosíthatja a jogcímeket. Ne feledje, hogy a jogcímek származnak, közvetlenül az Azure ad-ből. A webes alkalmazás módosítja a jogcímeket kap, ha ezek a módosítások nem jelennek meg a tulajdonosi jogkivonat, amely megkapja a webes API-t. További információkért lásd: [jogcím-átalakítást][claims-transformation].

## <a name="authorization"></a>Jogosultság

Általános engedélyezési, lásd: [szerepkör- és erőforrás-alapú hitelesítés][Authorization].

A JwtBearer közbenső kezeli a hitelesítési válaszokat. Például korlátozhatja egy vezérlő művelet a hitelesített felhasználók számára, használja a **[engedélyezés]** atrribute, és adja meg **JwtBearerDefaults.AuthenticationScheme** hitelesítési sémát:

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

401-es állapotkódot Ez ad vissza, ha a felhasználó nincs hitelesítve.

Korlátozza a tartományvezérlő műveleti authorizaton házirend, adja meg a házirend nevére a **[engedélyezés]** attribútum:

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

Ez adja vissza, ha a felhasználó hitelesítése nem 401-es állapotkódot, és a 403-as, ha a felhasználó hitelesítése, de nem jogosult. A szabályzat indításkor regisztrálása:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });

    // ...
}
```

[**Tovább**][token cache]

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer4]: https://github.com/IdentityServer/IdentityServer4
[Frissítse az alkalmazásjegyzékeket]: ./run-the-app.md#update-the-application-manifests
[Token-gyorsítótárazási]: token-cache.md
[bérlői feliratkozás]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
