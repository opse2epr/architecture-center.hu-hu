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
ms.locfileid: "24541465"
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="4c402-103">A háttérrendszer webes API biztonságossá tétele</span><span class="sxs-lookup"><span data-stu-id="4c402-103">Secure a backend web API</span></span>

<span data-ttu-id="4c402-104">[![GitHub](../_images/github.png) példakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="4c402-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="4c402-105">A [Dejójáték felmérések] alkalmazás CRUD-műveleteknek a felmérések kezelésére egy háttér webes API-t használ.</span><span class="sxs-lookup"><span data-stu-id="4c402-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="4c402-106">Például ha a felhasználó a "Saját felmérések" kattint, a webalkalmazás egy HTTP-kérést küld a webes API-hoz:</span><span class="sxs-lookup"><span data-stu-id="4c402-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```
GET /users/{userId}/surveys
```

<span data-ttu-id="4c402-107">A webes API-t egy JSON-objektumot ad vissza:</span><span class="sxs-lookup"><span data-stu-id="4c402-107">The web API returns a JSON object:</span></span>

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

<span data-ttu-id="4c402-108">A webes API-k nem engedélyezi a névtelen kéréseknél, a web app hitelesítenie kell magát OAuth 2 tulajdonosi jogkivonatok használatával.</span><span class="sxs-lookup"><span data-stu-id="4c402-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="4c402-109">Ez egy webfarmos kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="4c402-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="4c402-110">Az alkalmazás nem tesz, bármely AJAX-hívások az API a böngésző ügyfélről.</span><span class="sxs-lookup"><span data-stu-id="4c402-110">The application does not make any AJAX calls to the API from the browser client.</span></span>
> 
> 

<span data-ttu-id="4c402-111">Nincsenek két fő megközelítés közül választhat:</span><span class="sxs-lookup"><span data-stu-id="4c402-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="4c402-112">Delegált felhasználói azonosító.</span><span class="sxs-lookup"><span data-stu-id="4c402-112">Delegated user identity.</span></span> <span data-ttu-id="4c402-113">A webes alkalmazás hitelesíti a felhasználó identitását.</span><span class="sxs-lookup"><span data-stu-id="4c402-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="4c402-114">Alkalmazás azonosítója.</span><span class="sxs-lookup"><span data-stu-id="4c402-114">Application identity.</span></span> <span data-ttu-id="4c402-115">A webalkalmazás az ügyfél-azonosító, OAuth2 ügyfél hitelesítő adatok folyamata segítségével végzi a hitelesítést.</span><span class="sxs-lookup"><span data-stu-id="4c402-115">The web application authenticates with its client ID, using OAuth2 client credential flow.</span></span>

<span data-ttu-id="4c402-116">A Dejójáték alkalmazás megvalósítja a felhasználó delegált identitása.</span><span class="sxs-lookup"><span data-stu-id="4c402-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="4c402-117">A fő különbség itt is:</span><span class="sxs-lookup"><span data-stu-id="4c402-117">Here are the main differences:</span></span>

<span data-ttu-id="4c402-118">**A felhasználó delegált identitása**</span><span class="sxs-lookup"><span data-stu-id="4c402-118">**Delegated user identity**</span></span>

* <span data-ttu-id="4c402-119">A tulajdonosi jogkivonattal, a webes API-t küldeni a felhasználói identitás tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="4c402-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="4c402-120">A webes API lehetővé teszi a felhasználó identitása alapján felhasználását engedélyezési döntésekhez.</span><span class="sxs-lookup"><span data-stu-id="4c402-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="4c402-121">A webalkalmazásnak kell kezelni a 403-as (tiltott) hibákat a webes API-t, ha a felhasználó nem jogosult a művelet végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="4c402-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="4c402-122">Általában a webes alkalmazás továbbra is lehetővé teszi bizonyos felhasználását engedélyezési döntésekhez, amelyek hatással vannak a felhasználói felület, például a Megjelenítés vagy elrejtés a felhasználói felületi elemei).</span><span class="sxs-lookup"><span data-stu-id="4c402-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="4c402-123">A webes API potenciálisan nem megbízható ügyfelek, például JavaScript-alkalmazását vagy egy natív ügyfélalkalmazás által használható.</span><span class="sxs-lookup"><span data-stu-id="4c402-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="4c402-124">**Alkalmazásazonosító**</span><span class="sxs-lookup"><span data-stu-id="4c402-124">**Application identity**</span></span>

* <span data-ttu-id="4c402-125">A webes API-k nem kap a felhasználó adatait.</span><span class="sxs-lookup"><span data-stu-id="4c402-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="4c402-126">A webes API-k bármilyen engedély a felhasználó identitása alapján nem végezhető el.</span><span class="sxs-lookup"><span data-stu-id="4c402-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="4c402-127">Az összes engedélyezési döntéseket a webes alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="4c402-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="4c402-128">A webes API nem használható egy nem megbízható ügyfelet (JavaScript vagy natív ügyfélalkalmazás).</span><span class="sxs-lookup"><span data-stu-id="4c402-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="4c402-129">Ez a megközelítés lehet, hogy némileg egyszerűbb alkalmazásához nem engedélyezési logika van a Web API.</span><span class="sxs-lookup"><span data-stu-id="4c402-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="4c402-130">Mindkét megközelítés a webalkalmazás be kell szereznie egy hozzáférési jogkivonatot, amely a webes API-k meghívásához szükséges hitelesítő adatait.</span><span class="sxs-lookup"><span data-stu-id="4c402-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="4c402-131">A delegált felhasználói azonosítót a jogkivonat van, a kiállító terjesztési hely, amely képes a felhasználó nevében jogkivonatok kiállítása származnia.</span><span class="sxs-lookup"><span data-stu-id="4c402-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="4c402-132">Az ügyfél hitelesítő adatait az alkalmazás lehet, hogy a jogkivonat beszerzése az IDP vagy a saját token gazdakiszolgálón.</span><span class="sxs-lookup"><span data-stu-id="4c402-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="4c402-133">(De nem teljesen új írni egy token kiszolgálót; például a tesztelt keretrendszerével [IdentityServer3].) Ha az Azure AD a hitelesítést, azt rendelkezik erősen ajánlott a hozzáférési jogkivonat beszerzése az Azure AD, akkor az ügyfél-hitelesítő adatok folyamata.</span><span class="sxs-lookup"><span data-stu-id="4c402-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer3].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="4c402-134">Ez a cikk többi azt feltételezi, hogy az alkalmazás az Azure AD hitelesíti magát.</span><span class="sxs-lookup"><span data-stu-id="4c402-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![A hozzáférési jogkivonat beolvasása](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="4c402-136">A webes API-k regisztrálni Azure AD-ben</span><span class="sxs-lookup"><span data-stu-id="4c402-136">Register the web API in Azure AD</span></span>
<span data-ttu-id="4c402-137">Ahhoz, hogy Azure AD-be, hogy kiállítson egy tulajdonosi jogkivonatot a webes API néhány dolog, az Azure AD konfigurálása szüksége.</span><span class="sxs-lookup"><span data-stu-id="4c402-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="4c402-138">Regisztrálja a webes API-t az Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="4c402-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="4c402-139">A webalkalmazás az ügyfél-azonosító hozzáadása a webes API-alkalmazás jegyzékfájlja a a `knownClientApplications` tulajdonság.</span><span class="sxs-lookup"><span data-stu-id="4c402-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="4c402-140">Lásd: [frissíteni az alkalmazásjegyzékeknek].</span><span class="sxs-lookup"><span data-stu-id="4c402-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="4c402-141">Engedélyt a webes alkalmazás a webes API hívásához.</span><span class="sxs-lookup"><span data-stu-id="4c402-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="4c402-142">Az Azure felügyeleti portálon, beállíthatja az engedélyek két típusú: "alkalmazás" alkalmazásidentitás (ügyfél-hitelesítő adatok folyamata), vagy engedélyeinek "Delegált engedélyek" delegált felhasználói identitás.</span><span class="sxs-lookup"><span data-stu-id="4c402-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>
   
   ![Delegált engedélyek](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="4c402-144">Egy hozzáférési jogkivonat beolvasása</span><span class="sxs-lookup"><span data-stu-id="4c402-144">Getting an access token</span></span>
<span data-ttu-id="4c402-145">Előtt hívja meg a webes API-t, a webes alkalmazás lekérdezi egy hozzáférési jogkivonat az Azure AD.</span><span class="sxs-lookup"><span data-stu-id="4c402-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="4c402-146">A .NET-alkalmazásokat, használja a [az Azure AD Authentication Library (ADAL) .NET-keretrendszerhez készült][ADAL].</span><span class="sxs-lookup"><span data-stu-id="4c402-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="4c402-147">Az OAuth 2 hitelesítésikód-folyamata, a az alkalmazás egy engedélyezési kódot, a hozzáférési token adatcseréihez használható.</span><span class="sxs-lookup"><span data-stu-id="4c402-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="4c402-148">Az alábbi kód adal-t használ a hozzáférési jogkivonat beszerzése.</span><span class="sxs-lookup"><span data-stu-id="4c402-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="4c402-149">Ez a kód neve alatt a `AuthorizationCodeReceived` esemény.</span><span class="sxs-lookup"><span data-stu-id="4c402-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

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

<span data-ttu-id="4c402-150">Az alábbiakban a különböző paraméterek szükséges:</span><span class="sxs-lookup"><span data-stu-id="4c402-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="4c402-151">`authority`.</span><span class="sxs-lookup"><span data-stu-id="4c402-151">`authority`.</span></span> <span data-ttu-id="4c402-152">A bejelentkezett felhasználó nevében Bérlőazonosító származik.</span><span class="sxs-lookup"><span data-stu-id="4c402-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="4c402-153">(Nem a bérlő azonosítója a Szolgáltatottszoftver-szolgáltató)</span><span class="sxs-lookup"><span data-stu-id="4c402-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="4c402-154">`authorizationCode`.</span><span class="sxs-lookup"><span data-stu-id="4c402-154">`authorizationCode`.</span></span> <span data-ttu-id="4c402-155">vissza kapott az IDP hitelesítési kódot.</span><span class="sxs-lookup"><span data-stu-id="4c402-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="4c402-156">`clientId`.</span><span class="sxs-lookup"><span data-stu-id="4c402-156">`clientId`.</span></span> <span data-ttu-id="4c402-157">A webes alkalmazás ügyfél-azonosítót.</span><span class="sxs-lookup"><span data-stu-id="4c402-157">The web application's client ID.</span></span>
* <span data-ttu-id="4c402-158">`clientSecret`.</span><span class="sxs-lookup"><span data-stu-id="4c402-158">`clientSecret`.</span></span> <span data-ttu-id="4c402-159">A webes alkalmazás titkos ügyfélkulcsot.</span><span class="sxs-lookup"><span data-stu-id="4c402-159">The web application's client secret.</span></span>
* <span data-ttu-id="4c402-160">`redirectUri`.</span><span class="sxs-lookup"><span data-stu-id="4c402-160">`redirectUri`.</span></span> <span data-ttu-id="4c402-161">Az átirányítási URI, amely az OpenID connect.</span><span class="sxs-lookup"><span data-stu-id="4c402-161">The redirect URI that you set for OpenID connect.</span></span> <span data-ttu-id="4c402-162">Ez azért, amelyben a kiállító terjesztési hely visszahívja a tokenhez.</span><span class="sxs-lookup"><span data-stu-id="4c402-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="4c402-163">`resourceID`.</span><span class="sxs-lookup"><span data-stu-id="4c402-163">`resourceID`.</span></span> <span data-ttu-id="4c402-164">Az App ID URI a Web API, amely akkor jön létre, amikor a webes API-t az Azure ad-ben regisztrált</span><span class="sxs-lookup"><span data-stu-id="4c402-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="4c402-165">`tokenCache`.</span><span class="sxs-lookup"><span data-stu-id="4c402-165">`tokenCache`.</span></span> <span data-ttu-id="4c402-166">Egy objektum, amely gyorsítótárazza a hozzáférési jogkivonatok.</span><span class="sxs-lookup"><span data-stu-id="4c402-166">An object that caches the access tokens.</span></span> <span data-ttu-id="4c402-167">Lásd: [Token gyorsítótárazás].</span><span class="sxs-lookup"><span data-stu-id="4c402-167">See [Token caching].</span></span>

<span data-ttu-id="4c402-168">Ha `AcquireTokenByAuthorizationCodeAsync` sikeres, ADAL gyorsítótárazza a jogkivonatot.</span><span class="sxs-lookup"><span data-stu-id="4c402-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="4c402-169">Később kaphat a jogkivonatot a gyorsítótárból AcquireTokenSilentAsync hívása:</span><span class="sxs-lookup"><span data-stu-id="4c402-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="4c402-170">Ha `userId` a felhasználó objektum azonosítója, amelyet az a `http://schemas.microsoft.com/identity/claims/objectidentifier` jogcímek.</span><span class="sxs-lookup"><span data-stu-id="4c402-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="4c402-171">A webes API hívásához a hozzáférési jogkivonat segítségével</span><span class="sxs-lookup"><span data-stu-id="4c402-171">Using the access token to call the web API</span></span>
<span data-ttu-id="4c402-172">Miután a jogkivonatot, elküldheti a HTTP-kérelmek engedélyezési fejlécében a webes API-k.</span><span class="sxs-lookup"><span data-stu-id="4c402-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="4c402-173">A következő kiterjesztésmetódus felmérések alkalmazása HTTP hitelesítési fejlécéhez beállítása kérni, használja a **HttpClient** osztály.</span><span class="sxs-lookup"><span data-stu-id="4c402-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

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

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="4c402-174">A webes API-t a hitelesítéshez</span><span class="sxs-lookup"><span data-stu-id="4c402-174">Authenticating in the web API</span></span>
<span data-ttu-id="4c402-175">A webes API-t a tulajdonosi jogkivonat hitelesítésére rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="4c402-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="4c402-176">Az ASP.NET Core, használhatja a [Microsoft.AspNet.Authentication.JwtBearer] [ JwtBearer] csomag.</span><span class="sxs-lookup"><span data-stu-id="4c402-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="4c402-177">Ez a csomag biztosít alkalmazott szoftver, amely lehetővé teszi az alkalmazások OpenID Connect tulajdonosi jogkivonatokat fogadni.</span><span class="sxs-lookup"><span data-stu-id="4c402-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="4c402-178">A köztes regisztrálja a webes API `Startup` osztály.</span><span class="sxs-lookup"><span data-stu-id="4c402-178">Register the middleware in your web API `Startup` class.</span></span>

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

* <span data-ttu-id="4c402-179">**A célközönség**.</span><span class="sxs-lookup"><span data-stu-id="4c402-179">**Audience**.</span></span> <span data-ttu-id="4c402-180">Állítsa be ezt App ID URL-címet a webes API, amely a webes API-t az Azure ad-vel való regisztrálásakor létrehozott.</span><span class="sxs-lookup"><span data-stu-id="4c402-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="4c402-181">**Szolgáltató**.</span><span class="sxs-lookup"><span data-stu-id="4c402-181">**Authority**.</span></span> <span data-ttu-id="4c402-182">Egy több-bérlős alkalmazáshoz állítsa ezt a beállítást `https://login.microsoftonline.com/common/`.</span><span class="sxs-lookup"><span data-stu-id="4c402-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="4c402-183">**TokenValidationParameters**.</span><span class="sxs-lookup"><span data-stu-id="4c402-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="4c402-184">Egy több-bérlős alkalmazás beállítása **ValidateIssuer** false értékre.</span><span class="sxs-lookup"><span data-stu-id="4c402-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="4c402-185">Ez azt jelenti, hogy az alkalmazás ellenőrzi a kibocsátó.</span><span class="sxs-lookup"><span data-stu-id="4c402-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="4c402-186">**Események** abból származó osztály **JwtBearerEvents**.</span><span class="sxs-lookup"><span data-stu-id="4c402-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="4c402-187">Kibocsátó érvényesítése</span><span class="sxs-lookup"><span data-stu-id="4c402-187">Issuer validation</span></span>
<span data-ttu-id="4c402-188">A token kibocsátó érvényesítése a **JwtBearerEvents.TokenValidated** esemény.</span><span class="sxs-lookup"><span data-stu-id="4c402-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="4c402-189">A kibocsátó a "iss" jogcímek küldése.</span><span class="sxs-lookup"><span data-stu-id="4c402-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="4c402-190">A felmérések alkalmazásban, a webes API-k nem kezeli az [előfizetési bérlői].</span><span class="sxs-lookup"><span data-stu-id="4c402-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="4c402-191">Ezért azt csak ellenőrzi, hogy a kibocsátó már az alkalmazás-adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="4c402-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="4c402-192">Ha nem, akkor kivételt jelez, amely azt eredményezi, a hitelesítés sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="4c402-192">If not, it throws an exception, which causes authentication to fail.</span></span>

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

<span data-ttu-id="4c402-193">Mivel ez a példa bemutatja, használhatja a **TokenValidated** esemény módosításához a rendszer a jogcímeket.</span><span class="sxs-lookup"><span data-stu-id="4c402-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="4c402-194">Ne feledje, hogy a jogcímek közvetlenül az Azure AD származnak.</span><span class="sxs-lookup"><span data-stu-id="4c402-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="4c402-195">Ha a webes alkalmazás módosítja a jogcímeket kap, ezeket a módosításokat nem jelenik meg a tulajdonosi jogkivonatot, amely megkapja a webes API-t.</span><span class="sxs-lookup"><span data-stu-id="4c402-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="4c402-196">További információkért lásd: [jogcím-átalakítással][claims-transformation].</span><span class="sxs-lookup"><span data-stu-id="4c402-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="4c402-197">Engedélyezés</span><span class="sxs-lookup"><span data-stu-id="4c402-197">Authorization</span></span>
<span data-ttu-id="4c402-198">Általános vitafórum engedély, lásd: [szerepkör- és erőforrás-alapú engedélyezési][Authorization].</span><span class="sxs-lookup"><span data-stu-id="4c402-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span> 

<span data-ttu-id="4c402-199">A JwtBearer köztes kezeli az engedélyezési válaszokat.</span><span class="sxs-lookup"><span data-stu-id="4c402-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="4c402-200">Például korlátozhatja a vezérlő művelet a hitelesített felhasználók számára, használja a **[engedélyezés]** atrribute, és adja meg **JwtBearerDefaults.AuthenticationScheme** hitelesítési sémát:</span><span class="sxs-lookup"><span data-stu-id="4c402-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** atrribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="4c402-201">Ez visszaadja a 401-es állapotkód, ha a felhasználó nincs hitelesítve.</span><span class="sxs-lookup"><span data-stu-id="4c402-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="4c402-202">A tartományvezérlő műveleti authorizaton házirend korlátozásához adja meg a házirend nevére a a **[engedélyezés]** attribútum:</span><span class="sxs-lookup"><span data-stu-id="4c402-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="4c402-203">Ez adja vissza a 401-es állapotkód: Ha a felhasználó nincs hitelesítve, és a 403-as, ha a felhasználó hitelesítése, de nem engedélyezett.</span><span class="sxs-lookup"><span data-stu-id="4c402-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="4c402-204">A házirend indítása regisztrálása:</span><span class="sxs-lookup"><span data-stu-id="4c402-204">Register the policy on startup:</span></span>

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

<span data-ttu-id="4c402-205">[**Következő**][token cache]</span><span class="sxs-lookup"><span data-stu-id="4c402-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Dejójáték felmérések]: tailspin.md
[Tailspin Surveys]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[frissíteni az alkalmazásjegyzékeknek]: ./run-the-app.md#update-the-application-manifests
[Update the application manifests]: ./run-the-app.md#update-the-application-manifests
[Token gyorsítótárazás]: token-cache.md
[Token caching]: token-cache.md
[előfizetési bérlői]: signup.md
[tenant sign-up]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
