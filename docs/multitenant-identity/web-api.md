---
title: Egy több-bérlős alkalmazásban háttér webes API biztonságossá tétele
description: A háttérrendszer webes API biztonságossá tétele
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 65529280c5849e36ed7ff23de08a0b485034d0d8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="secure-a-backend-web-api"></a>A háttérrendszer webes API biztonságossá tétele

[![GitHub](../_images/github.png) példakód][sample application]

A [Dejójáték felmérések] alkalmazás CRUD-műveleteknek a felmérések kezelésére egy háttér webes API-t használ. Például ha a felhasználó a "Saját felmérések" kattint, a webalkalmazás egy HTTP-kérést küld a webes API-hoz:

```
GET /users/{userId}/surveys
```

A webes API-t egy JSON-objektumot ad vissza:

```
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

A webes API-k nem engedélyezi a névtelen kéréseknél, a web app hitelesítenie kell magát OAuth 2 tulajdonosi jogkivonatok használatával.

> [!NOTE]
> Ez egy webfarmos kiszolgálók. Az alkalmazás nem tesz, bármely AJAX-hívások az API a böngésző ügyfélről.
> 
> 

Nincsenek két fő megközelítés közül választhat:

* Delegált felhasználói azonosító. A webes alkalmazás hitelesíti a felhasználó identitását.
* Alkalmazás azonosítója. A webalkalmazás az ügyfél-azonosító, OAuth2 ügyfél hitelesítő adatok folyamata segítségével végzi a hitelesítést.

A Dejójáték alkalmazás megvalósítja a felhasználó delegált identitása. A fő különbség itt is:

**A felhasználó delegált identitása**

* A tulajdonosi jogkivonattal, a webes API-t küldeni a felhasználói identitás tartalmazza.
* A webes API lehetővé teszi a felhasználó identitása alapján felhasználását engedélyezési döntésekhez.
* A webalkalmazásnak kell kezelni a 403-as (tiltott) hibákat a webes API-t, ha a felhasználó nem jogosult a művelet végrehajtásához.
* Általában a webes alkalmazás továbbra is lehetővé teszi bizonyos felhasználását engedélyezési döntésekhez, amelyek hatással vannak a felhasználói felület, például a Megjelenítés vagy elrejtés a felhasználói felületi elemei).
* A webes API potenciálisan nem megbízható ügyfelek, például JavaScript-alkalmazását vagy egy natív ügyfélalkalmazás által használható.

**Alkalmazásazonosító**

* A webes API-k nem kap a felhasználó adatait.
* A webes API-k bármilyen engedély a felhasználó identitása alapján nem végezhető el. Az összes engedélyezési döntéseket a webes alkalmazás.  
* A webes API nem használható egy nem megbízható ügyfelet (JavaScript vagy natív ügyfélalkalmazás).
* Ez a megközelítés lehet, hogy némileg egyszerűbb alkalmazásához nem engedélyezési logika van a Web API.

Mindkét megközelítés a webalkalmazás be kell szereznie egy hozzáférési jogkivonatot, amely a webes API-k meghívásához szükséges hitelesítő adatait.

* A delegált felhasználói azonosítót a jogkivonat van, a kiállító terjesztési hely, amely képes a felhasználó nevében jogkivonatok kiállítása származnia.
* Az ügyfél hitelesítő adatait az alkalmazás lehet, hogy a jogkivonat beszerzése az IDP vagy a saját token gazdakiszolgálón. (De nem teljesen új írni egy token kiszolgálót; például a tesztelt keretrendszerével [IdentityServer3].) Ha az Azure AD a hitelesítést, azt rendelkezik erősen ajánlott a hozzáférési jogkivonat beszerzése az Azure AD, akkor az ügyfél-hitelesítő adatok folyamata.

Ez a cikk többi azt feltételezi, hogy az alkalmazás az Azure AD hitelesíti magát.

![A hozzáférési jogkivonat beolvasása](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a>A webes API-k regisztrálni Azure AD-ben
Ahhoz, hogy Azure AD-be, hogy kiállítson egy tulajdonosi jogkivonatot a webes API néhány dolog, az Azure AD konfigurálása szüksége.

1. Regisztrálja a webes API-t az Azure ad-ben.

2. A webalkalmazás az ügyfél-azonosító hozzáadása a webes API-alkalmazás jegyzékfájlja a a `knownClientApplications` tulajdonság. Lásd: [frissíteni az alkalmazásjegyzékeknek].

3. Engedélyt a webes alkalmazás a webes API hívásához. Az Azure felügyeleti portálon, beállíthatja az engedélyek két típusú: "alkalmazás" alkalmazásidentitás (ügyfél-hitelesítő adatok folyamata), vagy engedélyeinek "Delegált engedélyek" delegált felhasználói identitás.
   
   ![Delegált engedélyek](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a>Egy hozzáférési jogkivonat beolvasása
Előtt hívja meg a webes API-t, a webes alkalmazás lekérdezi egy hozzáférési jogkivonat az Azure AD. A .NET-alkalmazásokat, használja a [az Azure AD Authentication Library (ADAL) .NET-keretrendszerhez készült][ADAL].

Az OAuth 2 hitelesítésikód-folyamata, a az alkalmazás egy engedélyezési kódot, a hozzáférési token adatcseréihez használható. Az alábbi kód adal-t használ a hozzáférési jogkivonat beszerzése. Ez a kód neve alatt a `AuthorizationCodeReceived` esemény.

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

Az alábbiakban a különböző paraméterek szükséges:

* `authority`. A bejelentkezett felhasználó nevében Bérlőazonosító származik. (Nem a bérlő azonosítója a Szolgáltatottszoftver-szolgáltató)  
* `authorizationCode`. vissza kapott az IDP hitelesítési kódot.
* `clientId`. A webes alkalmazás ügyfél-azonosítót.
* `clientSecret`. A webes alkalmazás titkos ügyfélkulcsot.
* `redirectUri`. Az átirányítási URI, amely az OpenID connect. Ez azért, amelyben a kiállító terjesztési hely visszahívja a tokenhez.
* `resourceID`. Az App ID URI a Web API, amely akkor jön létre, amikor a webes API-t az Azure ad-ben regisztrált
* `tokenCache`. Egy objektum, amely gyorsítótárazza a hozzáférési jogkivonatok. Lásd: [Token gyorsítótárazás].

Ha `AcquireTokenByAuthorizationCodeAsync` sikeres, ADAL gyorsítótárazza a jogkivonatot. Később kaphat a jogkivonatot a gyorsítótárból AcquireTokenSilentAsync hívása:

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

Ha `userId` a felhasználó objektum azonosítója, amelyet az a `http://schemas.microsoft.com/identity/claims/objectidentifier` jogcímek.

## <a name="using-the-access-token-to-call-the-web-api"></a>A webes API hívásához a hozzáférési jogkivonat segítségével
Miután a jogkivonatot, elküldheti a HTTP-kérelmek engedélyezési fejlécében a webes API-k.

```
Authorization: Bearer xxxxxxxxxx
```

A következő kiterjesztésmetódus felmérések alkalmazása HTTP hitelesítési fejlécéhez beállítása kérni, használja a **HttpClient** osztály.

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

## <a name="authenticating-in-the-web-api"></a>A webes API-t a hitelesítéshez
A webes API-t a tulajdonosi jogkivonat hitelesítésére rendelkezik. Az ASP.NET Core, használhatja a [Microsoft.AspNet.Authentication.JwtBearer] [ JwtBearer] csomag. Ez a csomag biztosít alkalmazott szoftver, amely lehetővé teszi az alkalmazások OpenID Connect tulajdonosi jogkivonatokat fogadni.

A köztes regisztrálja a webes API `Startup` osztály.

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

* **A célközönség**. Állítsa be ezt App ID URL-címet a webes API, amely a webes API-t az Azure ad-vel való regisztrálásakor létrehozott.
* **Szolgáltató**. Egy több-bérlős alkalmazáshoz állítsa ezt a beállítást `https://login.microsoftonline.com/common/`.
* **TokenValidationParameters**. Egy több-bérlős alkalmazás beállítása **ValidateIssuer** false értékre. Ez azt jelenti, hogy az alkalmazás ellenőrzi a kibocsátó.
* **Események** abból származó osztály **JwtBearerEvents**.

### <a name="issuer-validation"></a>Kibocsátó érvényesítése
A token kibocsátó érvényesítése a **JwtBearerEvents.TokenValidated** esemény. A kibocsátó a "iss" jogcímek küldése.

A felmérések alkalmazásban, a webes API-k nem kezeli az [előfizetési bérlői]. Ezért azt csak ellenőrzi, hogy a kibocsátó már az alkalmazás-adatbázisban. Ha nem, akkor kivételt jelez, amely azt eredményezi, a hitelesítés sikertelen lesz.

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

Mivel ez a példa bemutatja, használhatja a **TokenValidated** esemény módosításához a rendszer a jogcímeket. Ne feledje, hogy a jogcímek közvetlenül az Azure AD származnak. Ha a webes alkalmazás módosítja a jogcímeket kap, ezeket a módosításokat nem jelenik meg a tulajdonosi jogkivonatot, amely megkapja a webes API-t. További információkért lásd: [jogcím-átalakítással][claims-transformation].

## <a name="authorization"></a>Engedélyezés
Általános vitafórum engedély, lásd: [szerepkör- és erőforrás-alapú engedélyezési][Authorization]. 

A JwtBearer köztes kezeli az engedélyezési válaszokat. Például korlátozhatja a vezérlő művelet a hitelesített felhasználók számára, használja a **[engedélyezés]** atrribute, és adja meg **JwtBearerDefaults.AuthenticationScheme** hitelesítési sémát:

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

Ez visszaadja a 401-es állapotkód, ha a felhasználó nincs hitelesítve.

A tartományvezérlő műveleti authorizaton házirend korlátozásához adja meg a házirend nevére a a **[engedélyezés]** attribútum:

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

Ez adja vissza a 401-es állapotkód: Ha a felhasználó nincs hitelesítve, és a 403-as, ha a felhasználó hitelesítése, de nem engedélyezett. A házirend indítása regisztrálása:

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

[**Következő**][token cache]

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Dejójáték felmérések]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[frissíteni az alkalmazásjegyzékeknek]: ./run-the-app.md#update-the-application-manifests
[Token gyorsítótárazás]: token-cache.md
[előfizetési bérlői]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
