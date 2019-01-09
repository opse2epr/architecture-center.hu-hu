---
title: Egy több-bérlős alkalmazásban a háttéralkalmazás webes API biztonságossá tétele
description: Hogyan teheti biztonságossá a háttéralkalmazás webes API-t.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 517bdbb6e1a1063db9337b63905e2ff5f4bdd4d4
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114028"
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="eeefe-103">Egy háttéralkalmazás webes API biztonságossá tétele</span><span class="sxs-lookup"><span data-stu-id="eeefe-103">Secure a backend web API</span></span>

<span data-ttu-id="eeefe-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="eeefe-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="eeefe-105">A [Tailspin Surveys] alkalmazás felmérések CRUD-műveletek kezelésére egy háttéralkalmazás webes API-t használ.</span><span class="sxs-lookup"><span data-stu-id="eeefe-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="eeefe-106">Például ha egy felhasználó a "Saját felmérések" kattint, a webes alkalmazás HTTP-kérelmet küld a webes API-hoz:</span><span class="sxs-lookup"><span data-stu-id="eeefe-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```http
GET /users/{userId}/surveys
```

<span data-ttu-id="eeefe-107">A webes API-t egy JSON-objektumot ad vissza:</span><span class="sxs-lookup"><span data-stu-id="eeefe-107">The web API returns a JSON object:</span></span>

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

<span data-ttu-id="eeefe-108">A webes API nem engedélyezi a névtelen kérésekkel, így a web app hitelesítenie kell magát az OAuth 2 tulajdonosi jogkivonatok segítségével.</span><span class="sxs-lookup"><span data-stu-id="eeefe-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="eeefe-109">Ez a forgatókönyv kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="eeefe-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="eeefe-110">Az alkalmazás nem derül összes AJAX hívás a böngésző ügyfélről az API-hoz.</span><span class="sxs-lookup"><span data-stu-id="eeefe-110">The application does not make any AJAX calls to the API from the browser client.</span></span>

<span data-ttu-id="eeefe-111">Nincsenek két fő megközelítés közül választhat:</span><span class="sxs-lookup"><span data-stu-id="eeefe-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="eeefe-112">Felhasználói identitás delegált.</span><span class="sxs-lookup"><span data-stu-id="eeefe-112">Delegated user identity.</span></span> <span data-ttu-id="eeefe-113">A webes alkalmazás hitelesíti a felhasználó identitását.</span><span class="sxs-lookup"><span data-stu-id="eeefe-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="eeefe-114">Identita aplikace.</span><span class="sxs-lookup"><span data-stu-id="eeefe-114">Application identity.</span></span> <span data-ttu-id="eeefe-115">A webalkalmazás az ügyfél-azonosító, ügyfél-hitelesítő adatok folyamata OAuth2 használatával végzi a hitelesítést.</span><span class="sxs-lookup"><span data-stu-id="eeefe-115">The web application authenticates with its client ID, using OAuth2 client credential flow.</span></span>

<span data-ttu-id="eeefe-116">A Tailspin alkalmazás megvalósítja a felhasználó delegált identitása.</span><span class="sxs-lookup"><span data-stu-id="eeefe-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="eeefe-117">Az alábbiakban a fő különbség:</span><span class="sxs-lookup"><span data-stu-id="eeefe-117">Here are the main differences:</span></span>

<span data-ttu-id="eeefe-118">**Delegált felhasználói identitás:**</span><span class="sxs-lookup"><span data-stu-id="eeefe-118">**Delegated user identity:**</span></span>

* <span data-ttu-id="eeefe-119">A webes API-nak küldött tulajdonosi jogkivonat tartalmazza a felhasználó identitását.</span><span class="sxs-lookup"><span data-stu-id="eeefe-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="eeefe-120">A webes API lehetővé teszi a felhasználó identitása alapján felhasználását engedélyezési döntésekhez.</span><span class="sxs-lookup"><span data-stu-id="eeefe-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="eeefe-121">A webalkalmazás 403-as (tiltott) hibáinak kezelése a webes API-t, ha a felhasználó nem jogosult a művelet végrehajtásához szükséges.</span><span class="sxs-lookup"><span data-stu-id="eeefe-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="eeefe-122">Általában a webalkalmazás továbbra is lehetővé teszi bizonyos felhasználását engedélyezési döntésekhez, amelyek befolyásolják a felhasználói felület, például a Megjelenítés vagy elrejtés a felhasználói felületi elemeket).</span><span class="sxs-lookup"><span data-stu-id="eeefe-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="eeefe-123">A webes API potenciálisan nem megbízható ügyfelek, például egy JavaScript-alkalmazását vagy egy natív ügyfélalkalmazás által használható.</span><span class="sxs-lookup"><span data-stu-id="eeefe-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="eeefe-124">**Identita aplikace:**</span><span class="sxs-lookup"><span data-stu-id="eeefe-124">**Application identity:**</span></span>

* <span data-ttu-id="eeefe-125">A webes API-k nem kap a felhasználó adatait.</span><span class="sxs-lookup"><span data-stu-id="eeefe-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="eeefe-126">A webes API-k nem hajtható végre a felhasználó identitása alapján engedélyezés.</span><span class="sxs-lookup"><span data-stu-id="eeefe-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="eeefe-127">Az összes engedélyezési döntéseket a webes alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="eeefe-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="eeefe-128">A webes API egy nem megbízható ügyfél (JavaScript- vagy natív ügyfélalkalmazás) nem használható.</span><span class="sxs-lookup"><span data-stu-id="eeefe-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="eeefe-129">Ez a megközelítés implementálásához némileg egyszerűbb lehet, mert nincs engedélyezési logikai szerepel a webes API-t.</span><span class="sxs-lookup"><span data-stu-id="eeefe-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="eeefe-130">Mindkét megközelítés a webalkalmazás be kell szereznie egy hozzáférési jogkivonatot, amely a webes API-k meghívásához szükséges hitelesítő adatait.</span><span class="sxs-lookup"><span data-stu-id="eeefe-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="eeefe-131">Delegált felhasználói identitás a jogkivonat kell meghatároznia az Identitásszolgáltató, amely a felhasználó nevében jogkivonatok kiállítása is rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="eeefe-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="eeefe-132">Az ügyfél-hitelesítő adatok egy alkalmazás előfordulhat, hogy a jogkivonat beszerzése az identitásszolgáltató vagy saját jogkivonat-kiszolgálót.</span><span class="sxs-lookup"><span data-stu-id="eeefe-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="eeefe-133">(De nem teljesen új jogkivonat kiszolgálói írása; például egy tesztelt keretrendszert használ [IdentityServer4].) Ha az Azure AD-hitelesítést, Határozottan javasoljuk, hogy a hozzáférési jogkivonat beszerzése az Azure AD, még akkor is az ügyfél-hitelesítő adat folyamatát.</span><span class="sxs-lookup"><span data-stu-id="eeefe-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer4].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="eeefe-134">Ez a cikk többi része feltételezi, hogy az Azure AD hitelesíti magát az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="eeefe-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![A hozzáférési jogkivonat beolvasása](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="eeefe-136">A webes API regisztrálása az Azure ad-ben</span><span class="sxs-lookup"><span data-stu-id="eeefe-136">Register the web API in Azure AD</span></span>

<span data-ttu-id="eeefe-137">Ahhoz, hogy a webes API tulajdonosi jogkivonatok kiállítása az Azure AD, a konfigurálása az Azure ad-ben a következőkre kell.</span><span class="sxs-lookup"><span data-stu-id="eeefe-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="eeefe-138">A webes API regisztrálása az Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="eeefe-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="eeefe-139">A webalkalmazás az ügyfél-azonosító hozzáadása a webes API-k alkalmazásjegyzékhez a a `knownClientApplications` tulajdonság.</span><span class="sxs-lookup"><span data-stu-id="eeefe-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="eeefe-140">Lásd: [Frissítse az alkalmazásjegyzékeket].</span><span class="sxs-lookup"><span data-stu-id="eeefe-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="eeefe-141">Engedélyezheti a webes alkalmazás számára a webes API hívása.</span><span class="sxs-lookup"><span data-stu-id="eeefe-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="eeefe-142">Az Azure felügyeleti portálján, az engedélyek két típusú állíthatja be: "Alkalmazásengedélyek" identita aplikace (ügyfél-hitelesítő adat folyamat), vagy a "Delegált engedélyek" a delegált felhasználó identitását.</span><span class="sxs-lookup"><span data-stu-id="eeefe-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>

   ![Delegált engedélyek](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="eeefe-144">Hozzáférési jogkivonat beolvasása</span><span class="sxs-lookup"><span data-stu-id="eeefe-144">Getting an access token</span></span>

<span data-ttu-id="eeefe-145">Előtt hívja meg a webes API-t, a webes alkalmazás lekérése hozzáférési token Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="eeefe-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="eeefe-146">A .NET-alkalmazás, használja a [az Azure AD Authentication Library (ADAL) for .NET][ADAL].</span><span class="sxs-lookup"><span data-stu-id="eeefe-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="eeefe-147">Az OAuth 2 engedélyezési kód folyamata az alkalmazás hozzáférési jogkivonat egy engedélyezési kódjának adatcseréihez használható.</span><span class="sxs-lookup"><span data-stu-id="eeefe-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="eeefe-148">A következő kódot használ adal-t beszerezni a hozzáférési jogkivonatot.</span><span class="sxs-lookup"><span data-stu-id="eeefe-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="eeefe-149">Ez a kód nevezzük során a `AuthorizationCodeReceived` esemény.</span><span class="sxs-lookup"><span data-stu-id="eeefe-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

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

<span data-ttu-id="eeefe-150">Az alábbiakban a különböző paraméterek szükségesek:</span><span class="sxs-lookup"><span data-stu-id="eeefe-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="eeefe-151">`authority`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-151">`authority`.</span></span> <span data-ttu-id="eeefe-152">A bejelentkezett felhasználó Bérlőazonosítója származik.</span><span class="sxs-lookup"><span data-stu-id="eeefe-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="eeefe-153">(Nem a bérlő Azonosítóját az SaaS-szolgáltató)</span><span class="sxs-lookup"><span data-stu-id="eeefe-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="eeefe-154">`authorizationCode`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-154">`authorizationCode`.</span></span> <span data-ttu-id="eeefe-155">a hitelesítési kódot, amely az identitásszolgáltató vissza lett.</span><span class="sxs-lookup"><span data-stu-id="eeefe-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="eeefe-156">`clientId`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-156">`clientId`.</span></span> <span data-ttu-id="eeefe-157">A webes alkalmazás ügyfél-azonosítót.</span><span class="sxs-lookup"><span data-stu-id="eeefe-157">The web application's client ID.</span></span>
* <span data-ttu-id="eeefe-158">`clientSecret`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-158">`clientSecret`.</span></span> <span data-ttu-id="eeefe-159">A webes alkalmazás titkos ügyfélkódja.</span><span class="sxs-lookup"><span data-stu-id="eeefe-159">The web application's client secret.</span></span>
* <span data-ttu-id="eeefe-160">`redirectUri`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-160">`redirectUri`.</span></span> <span data-ttu-id="eeefe-161">Az átirányítási URI-t, amelyet beállított OpenID connect.</span><span class="sxs-lookup"><span data-stu-id="eeefe-161">The redirect URI that you set for OpenID connect.</span></span> <span data-ttu-id="eeefe-162">Ez a, amelyben az Identitásszolgáltató visszahívja a jogkivonattal.</span><span class="sxs-lookup"><span data-stu-id="eeefe-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="eeefe-163">`resourceID`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-163">`resourceID`.</span></span> <span data-ttu-id="eeefe-164">Az Alkalmazásazonosító URI a webes API-t, a webes API Azure AD-ben való regisztrálásakor létrehozott</span><span class="sxs-lookup"><span data-stu-id="eeefe-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="eeefe-165">`tokenCache`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-165">`tokenCache`.</span></span> <span data-ttu-id="eeefe-166">Gyorsítótárazza a hozzáférési jogkivonatok objektum.</span><span class="sxs-lookup"><span data-stu-id="eeefe-166">An object that caches the access tokens.</span></span> <span data-ttu-id="eeefe-167">Lásd: [Token-gyorsítótárazási].</span><span class="sxs-lookup"><span data-stu-id="eeefe-167">See [Token caching].</span></span>

<span data-ttu-id="eeefe-168">Ha `AcquireTokenByAuthorizationCodeAsync` sikeres, az adal-t gyorsítótárazza a jogkivonat.</span><span class="sxs-lookup"><span data-stu-id="eeefe-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="eeefe-169">Később kérheti a jogkivonatot a gyorsítótárból AcquireTokenSilentAsync meghívásával:</span><span class="sxs-lookup"><span data-stu-id="eeefe-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="eeefe-170">ahol `userId` van a felhasználói objektum azonosítója, amely megtalálható a `http://schemas.microsoft.com/identity/claims/objectidentifier` jogcím.</span><span class="sxs-lookup"><span data-stu-id="eeefe-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="eeefe-171">A hozzáférési token használatával a webes API meghívásához</span><span class="sxs-lookup"><span data-stu-id="eeefe-171">Using the access token to call the web API</span></span>

<span data-ttu-id="eeefe-172">Miután a jogkivonatot, küldje el a HTTP-kérések az engedélyezési fejléc a webes API-hoz.</span><span class="sxs-lookup"><span data-stu-id="eeefe-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```http
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="eeefe-173">A következő metódust a Surveys alkalmazás állítja be az engedélyeztetési fejléc egy HTTP-kérelem használatával a **HttpClient** osztály.</span><span class="sxs-lookup"><span data-stu-id="eeefe-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

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

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="eeefe-174">A webes API hitelesítése</span><span class="sxs-lookup"><span data-stu-id="eeefe-174">Authenticating in the web API</span></span>

<span data-ttu-id="eeefe-175">A webes API rendelkezik tulajdonosi jogkivonat hitelesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="eeefe-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="eeefe-176">Az ASP.NET Core, használhat a [Microsoft.AspNet.Authentication.JwtBearer] [ JwtBearer] csomagot.</span><span class="sxs-lookup"><span data-stu-id="eeefe-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="eeefe-177">Ez a csomag biztosít közbenső szoftvert, amely lehetővé teszi az alkalmazás OpenID Connect tulajdonosi jogkivonatokat fogadni.</span><span class="sxs-lookup"><span data-stu-id="eeefe-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="eeefe-178">A webes API regisztrálása a közbenső szoftver `Startup` osztály.</span><span class="sxs-lookup"><span data-stu-id="eeefe-178">Register the middleware in your web API `Startup` class.</span></span>

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

* <span data-ttu-id="eeefe-179">**Célközönség**.</span><span class="sxs-lookup"><span data-stu-id="eeefe-179">**Audience**.</span></span> <span data-ttu-id="eeefe-180">Állítsa be ezt az azonosító URL-címét a webes API-t, a webes API-t az Azure AD-regisztrációja során létrehozott.</span><span class="sxs-lookup"><span data-stu-id="eeefe-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="eeefe-181">**Szolgáltató**.</span><span class="sxs-lookup"><span data-stu-id="eeefe-181">**Authority**.</span></span> <span data-ttu-id="eeefe-182">Több-bérlős alkalmazás esetében állítsa ezt a beállítást `https://login.microsoftonline.com/common/`.</span><span class="sxs-lookup"><span data-stu-id="eeefe-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="eeefe-183">**TokenValidationParameters**.</span><span class="sxs-lookup"><span data-stu-id="eeefe-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="eeefe-184">Egy több-bérlős alkalmazás beállítása **ValidateIssuer** hamis értékre.</span><span class="sxs-lookup"><span data-stu-id="eeefe-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="eeefe-185">Ez azt jelenti, hogy az alkalmazás ellenőrzi a kibocsátó.</span><span class="sxs-lookup"><span data-stu-id="eeefe-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="eeefe-186">**Események** egy osztály, amely származó **JwtBearerEvents**.</span><span class="sxs-lookup"><span data-stu-id="eeefe-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="eeefe-187">Kibocsátó érvényesítése</span><span class="sxs-lookup"><span data-stu-id="eeefe-187">Issuer validation</span></span>

<span data-ttu-id="eeefe-188">Az a tokenkiállító ellenőrzését a **JwtBearerEvents.TokenValidated** esemény.</span><span class="sxs-lookup"><span data-stu-id="eeefe-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="eeefe-189">A kiállító a "iss" jogcímek zajlik.</span><span class="sxs-lookup"><span data-stu-id="eeefe-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="eeefe-190">A Surveys alkalmazás, az a webes API-k nem kezeli az [bérlői feliratkozás].</span><span class="sxs-lookup"><span data-stu-id="eeefe-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="eeefe-191">Ezért azt csak ellenőrzi, hogy a kibocsátó már az alkalmazás-adatbázis.</span><span class="sxs-lookup"><span data-stu-id="eeefe-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="eeefe-192">Ha nem, akkor kivételt jelez, amely-hitelesítés sikertelen.</span><span class="sxs-lookup"><span data-stu-id="eeefe-192">If not, it throws an exception, which causes authentication to fail.</span></span>

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

<span data-ttu-id="eeefe-193">Mivel ez a példa bemutatja, is használhatja a **TokenValidated** eseményhez, és módosíthatja a jogcímeket.</span><span class="sxs-lookup"><span data-stu-id="eeefe-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="eeefe-194">Ne feledje, hogy a jogcímek származnak, közvetlenül az Azure ad-ből.</span><span class="sxs-lookup"><span data-stu-id="eeefe-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="eeefe-195">A webes alkalmazás módosítja a jogcímeket kap, ha ezek a módosítások nem jelennek meg a tulajdonosi jogkivonat, amely megkapja a webes API-t.</span><span class="sxs-lookup"><span data-stu-id="eeefe-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="eeefe-196">További információkért lásd: [jogcím-átalakítást][claims-transformation].</span><span class="sxs-lookup"><span data-stu-id="eeefe-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="eeefe-197">Engedélyezés</span><span class="sxs-lookup"><span data-stu-id="eeefe-197">Authorization</span></span>

<span data-ttu-id="eeefe-198">Általános engedélyezési, lásd: [szerepkör- és erőforrás-alapú hitelesítés][Authorization].</span><span class="sxs-lookup"><span data-stu-id="eeefe-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span>

<span data-ttu-id="eeefe-199">A JwtBearer közbenső kezeli a hitelesítési válaszokat.</span><span class="sxs-lookup"><span data-stu-id="eeefe-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="eeefe-200">Például korlátozhatja egy vezérlő művelet a hitelesített felhasználók számára, használja a **[engedélyezés]** atrribute, és adja meg **JwtBearerDefaults.AuthenticationScheme** hitelesítési sémát:</span><span class="sxs-lookup"><span data-stu-id="eeefe-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** atrribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="eeefe-201">401-es állapotkódot Ez ad vissza, ha a felhasználó nincs hitelesítve.</span><span class="sxs-lookup"><span data-stu-id="eeefe-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="eeefe-202">Korlátozza a tartományvezérlő műveleti authorizaton házirend, adja meg a házirend nevére a **[engedélyezés]** attribútum:</span><span class="sxs-lookup"><span data-stu-id="eeefe-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="eeefe-203">Ez adja vissza, ha a felhasználó hitelesítése nem 401-es állapotkódot, és a 403-as, ha a felhasználó hitelesítése, de nem jogosult.</span><span class="sxs-lookup"><span data-stu-id="eeefe-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="eeefe-204">A szabályzat indításkor regisztrálása:</span><span class="sxs-lookup"><span data-stu-id="eeefe-204">Register the policy on startup:</span></span>

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

<span data-ttu-id="eeefe-205">[**Tovább**][token cache]</span><span class="sxs-lookup"><span data-stu-id="eeefe-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer4]: https://github.com/IdentityServer/IdentityServer4
[Frissítse az alkalmazásjegyzékeket]: ./run-the-app.md#update-the-application-manifests
[Update the application manifests]: ./run-the-app.md#update-the-application-manifests
[Token-gyorsítótárazási]: token-cache.md
[Token caching]: token-cache.md
[bérlői feliratkozás]: signup.md
[tenant sign-up]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
