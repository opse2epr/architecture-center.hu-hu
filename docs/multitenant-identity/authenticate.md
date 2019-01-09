---
title: Hitelesítés a több-bérlős alkalmazásokban
description: Hogyan egy több-bérlős alkalmazás hitelesítheti a felhasználókat az Azure Active Directoryból.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
ms.openlocfilehash: c66fd80a539b507ee7a78c87a2ffa0d735b9def1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112430"
---
# <a name="authenticate-using-azure-ad-and-openid-connect"></a>Hitelesítés az Azure AD-vel és az OpenID Connect

[![GitHub](../_images/github.png) Mintakód][sample application]

A Surveys alkalmazás az OpenID Connect (OIDC) protokoll használatával hitelesíti a felhasználókat az Azure Active Directory (Azure AD). A Surveys alkalmazás használja az ASP.NET Core, amely beépített közbenső OIDC rendelkezik. Az alábbi ábrán látható, hogy mi történik, ha a felhasználó bejelentkezik, magas szinten.

![A hitelesítési folyamatból](./images/auth-flow.png)

1. A felhasználó az alkalmazásban a "bejelentkezés" gombra kattint. Ez a művelet egy MVC-vezérlő kezeli.
2. Az MVC-vezérlő értéket ad vissza egy **ChallengeResult** művelet.
3. A közbenső szoftver elfogja a **ChallengeResult** , és létrehoz egy 302 választ, amely átirányítja a felhasználót az Azure AD bejelentkezési oldalra.
4. A felhasználó hitelesíti magát az Azure ad-ben.
5. Az Azure AD elküldi az alkalmazás-azonosító jogkivonat.
6. A közbenső szoftver az azonosító jogkivonat érvényesíti. Ezen a ponton a felhasználó mostantól az alkalmazáson belüli hitelesítését.
7. A közbenső szoftver alkalmazásnak átirányítja a felhasználót.

## <a name="register-the-app-with-azure-ad"></a>Az alkalmazás regisztrálása az Azure ad-vel

OpenID Connect engedélyezéséhez az SaaS-szolgáltató regisztrálja az alkalmazás a saját Azure AD-bérlő belül.

Az alkalmazás regisztrálásához kövesse [alkalmazások az Azure Active Directoryval való integrálását](/azure/active-directory/active-directory-integrating-applications/), a szakaszban [egy alkalmazás hozzáadása](/azure/active-directory/active-directory-integrating-applications/#adding-an-application).

Lásd: [a Surveys alkalmazás futtatása](./run-the-app.md) lépései a Surveys alkalmazás számára. Vegye figyelembe a következőket:

- Egy több-bérlős alkalmazásban konfigurálnia kell a több-bérlős beállítás explicit módon. Ez lehetővé teszi, hogy más szervezetek számára az alkalmazás eléréséhez.

- A válasz-URL az URL-címe, ahol az Azure AD elküldi az OAuth 2.0-válaszok. Az ASP.NET Core használata esetén ez meg kell felelnie a közbenső hitelesítési szoftver konfigurált elérési útja (lásd a következő szakaszban).

## <a name="configure-the-auth-middleware"></a>A hitelesítési közbenső szoftver konfigurálása

Ez a szakasz ismerteti a közbenső hitelesítési szoftver konfigurálása az ASP.NET Core OpenID-kapcsolattal több-bérlős hitelesítéshez.

Az a [indítási osztályt](/aspnet/core/fundamentals/startup), az OpenID Connect közbenső szoftver hozzáadása:

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectOptions {
    ClientId = configOptions.AzureAd.ClientId,
    ClientSecret = configOptions.AzureAd.ClientSecret, // for code flow
    Authority = Constants.AuthEndpointPrefix,
    ResponseType = OpenIdConnectResponseType.CodeIdToken,
    PostLogoutRedirectUri = configOptions.AzureAd.PostLogoutRedirectUri,
    SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme,
    TokenValidationParameters = new TokenValidationParameters { ValidateIssuer = false },
    Events = new SurveyAuthenticationEvents(configOptions.AzureAd, loggerFactory),
});
```

Figyelje meg, hogy néhány beállítás, megnyílik a futásidejű konfigurációs beállításai. Íme a közbenső szoftver beállítások jelenti:

- **ClientId**. Az alkalmazás ügyfél-azonosító, amely az Azure ad-ben az alkalmazás regisztrálásakor kapott.
- **Szolgáltató**. Több-bérlős alkalmazás esetében állítsa ezt a beállítást `https://login.microsoftonline.com/common/`. Ez az az Azure AD közös végpont, amely lehetővé teszi a felhasználók bármilyen Azure AD-bérlővel, jelentkezzen be az URL-CÍMÉT. A közös végpontot kapcsolatos további információkért lásd: [ebben a blogbejegyzésben](https://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/).
- A **TokenValidationParameters**állítsa be **ValidateIssuer** hamis értékre. Ez azt jelenti, hogy az alkalmazás felelős a kibocsátó értékét az azonosító jogkivonat érvényesítése. (A közbenső szoftver továbbra is érvényesíti a jogkivonatot magát.) A kibocsátó ellenőrzésével kapcsolatos további információkért lásd: [kibocsátó érvényesítése](claims.md#issuer-validation).
- **PostLogoutRedirectUri**. Adja meg a felhasználók átirányítása után a kijelentkezési URL-címet. Ez lehet egy oldal, amely lehetővé teszi a névtelen kérelmek &mdash; általában a kezdőlapon.
- **SignInScheme**. Állítsa a bestattempt értékre `CookieAuthenticationDefaults.AuthenticationScheme`. Ez a beállítás azt jelenti, hogy a felhasználó hitelesítése után a felhasználói jogcímeket a rendszer helyben tárolja egy cookie-ban. Ez a cookie módját a felhasználó bejelentkezve marad a böngésző-munkamenet során.
- **Események.** Esemény visszahívások; Lásd: [hitelesítési események](#authentication-events).

A folyamat a hitelesítési cookie-k közbenső is hozzáadhat. A közbenső szoftver felelős a felhasználói jogcímeket a cookie-k írásakor, és a cookie-k során későbbi lapbetöltések olvassa.

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions {
    AutomaticAuthenticate = true,
    AutomaticChallenge = true,
    AccessDeniedPath = "/Home/Forbidden",
    CookieSecure = CookieSecurePolicy.Always,

    // The default setting for cookie expiration is 14 days. SlidingExpiration is set to true by default
    ExpireTimeSpan = TimeSpan.FromHours(1),
    SlidingExpiration = true
});
```

## <a name="initiate-the-authentication-flow"></a>A hitelesítési folyamat elindításához

A hitelesítési folyamat elindításához az ASP.NET mvc-ben, adja vissza egy **ChallengeResult** származó a contoller:

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

Ennek hatására a közbenső szoftvert választ 302 (található), amely átirányítja a felhasználókat a hitelesítési végpontra.

## <a name="user-login-sessions"></a>Felhasználói munkamenetek

Ahogy említettük, ha a felhasználó először jelentkezik be, a hitelesítési cookie-k közbenső ír egy cookie-t a felhasználói jogcímeket. Ezt követően a HTTP-kérések hitelesítése a cookie-k olvasása.

Alapértelmezés szerint a cookie-k közbenső ír egy [munkamenetcookie-t][session-cookie], mely beolvasása törlése után a felhasználó bezárja a böngészőt. A következő alkalommal, amikor a felhasználó ezután felkeresi a webhelyet, akkor kell jelentkezzen be újra. Azonban ha **IsPersistent** true-ra a **ChallengeResult**, a közbenső szoftver ír egy állandó cookie-t, ezért a böngésző bezárása után a felhasználó bejelentkezve marad. Beállíthatja, hogy a cookie lejáratának; Lásd: [cookie beállításai szabályozása][cookie-options]. Maradandó cookie-k a felhasználók kényelmesebb, de lehet, hogy nem megfelelő alkalmazások (például, egy banki alkalmazás) Amennyiben azt szeretné, hogy a felhasználót, hogy jelentkezzen be minden alkalommal.

## <a name="about-the-openid-connect-middleware"></a>Tudnivalók az OpenID Connect közbenső szoftvert

Az OpenID Connect közbenső szoftvert az ASP.NET elrejti a legtöbb protokoll részleteit. Ez a szakasz tartalmaz néhány megjegyzés, amely lehet hasznos, ha a protokoll folyamat ismertetése megvalósításával kapcsolatban.

Először is hozzunk vizsgálja meg a hitelesítési folyamat alapján (figyelmen kívül hagyja az alkalmazás és az Azure AD közötti OIDC protokoll folyamat részletei) az ASP.NET. Az alábbi ábrán látható a folyamat.

![Bejelentkezési folyamata](./images/sign-in-flow.png)

Ezen az ábrán két MVC-vezérlők vannak. A fiók vezérlő bejelentkezési kérelmet kezel, és a kezdőlapvezérlő kiszolgálja a kezdőlapon.

A hitelesítési folyamat a következő:

1. A felhasználó a "Sign-in" gombra kattint, és a böngészőben a GET-kérést küld. Például: `GET /Account/SignIn/`.
2. A fiók vezérlő értéket ad vissza egy `ChallengeResult`.
3. A OIDC közbenső HTTP 302 választ, az Azure AD-átirányítás adja vissza.
4. A böngészőben a hitelesítési kérést küld az Azure ad-ben
5. A felhasználó bejelentkezik az Azure ad-hez, és az Azure AD választ küld vissza hitelesítést.
6. A OIDC közbenső rendszerbiztonsági tag jogcímeket hoz létre, és továbbítja azt a hitelesítési cookie-k közbenső szoftver.
7. A cookie-k közbenső szerializálja a jogcímtag, és beállítja a cookie-k.
8. A OIDC közbenső átirányítja az alkalmazás visszahívási URL-CÍMÉT.
9. A böngészőben a cookie-t küld a kérelemben szereplő átirányítási követi.
10. A cookie-k közbenső deserializes egy egyszerű jogcímek a cookie-k, és beállítja a `HttpContext.User` a jogcímtag egyenlő. A kérelem egy MVC-vezérlő irányítja a rendszer.

### <a name="authentication-ticket"></a>Hitelesítési jegy

Ha a hitelesítés sikeres, a OIDC közbenső hitelesítési jegyre, amely tartalmazza a felhasználói jogcímeket tartalmazó jogcímeket rendszerbiztonsági tag hoz létre. A jegy belül hozzáférhet a **AuthenticationValidated** vagy **TicketReceived** esemény.

> [!NOTE]
> Az egész hitelesítési folyamat befejezéséig `HttpContext.User` továbbra is fogja tárolni egy névtelen egyszerű **nem** a hitelesített felhasználó. A névtelen egyszerű egy üres jogcímek gyűjteménye van. Hitelesítés befejeződik, és az alkalmazás átirányítja a cookie-k közbenső deserializes után a hitelesítési cookie-k és a csoportok `HttpContext.User` a jogcímek egyszerű, amely a hitelesített felhasználó jelöli.

### <a name="authentication-events"></a>Hitelesítési események

A hitelesítési folyamat során az OpenID Connect közbenső szoftvert események sorát vet fel:

- **RedirectToIdentityProvider**. Nevű jobb, előtt a közbenső szoftver a hitelesítési végpontra irányítja át. Használhatja ezt az eseményt az átirányítási URL-címet; Ha például a kérelem paramétereinek hozzáadása. Lásd: [hozzáadása a rendszergazda beleegyezést kérő](signup.md#adding-the-admin-consent-prompt) példaként.
- **AuthorizationCodeReceived**. Az engedélyezési kódot meghívva.
- **TokenResponseReceived**. Neve az a közbenső szoftver lekérése hozzáférési token az Identitásszolgáltató, de előtt érvényességét. Csak az engedélyezési kód folyamata vonatkozik.
- **TokenValidated**. Miután a közbenső szoftver ellenőrzi az azonosító jogkivonat neve. Ezen a ponton az alkalmazás rendelkezik a felhasználóval kapcsolatos ellenőrzött jogcímek egy készletét. Ezt az eseményt a további ellenőrzéshez a jogcímek, vagy a jogcímek átalakítására használható. Lásd: [használata](claims.md).
- **UserInformationReceived**. Ha a közbenső szoftver a felhasználói profil olvas be a felhasználói adatok végpont neve. Csak az engedélyezési kód folyamata, és csak akkor vonatkozik `GetClaimsFromUserInfoEndpoint = true` a közbenső szoftver beállítások között.
- **TicketReceived**. Meghívva, ha a hitelesítés végbemegy. Ez az az utolsó esemény, feltéve, hogy a hitelesítés sikeres lesz. Ez az esemény történik, ha a felhasználó bejelentkezett az alkalmazásba.
- **AuthenticationFailed**. Ha a hitelesítés sikertelen nevezik. Ez az esemény segítségével hitelesítési hibák kezelésére &mdash; például úgy, hogy egy hibalap átirányítása.

Adja meg visszahívások ezeket az eseményeket, állítsa be a **események** a közbenső szoftverek lehetőséget. Az eseménykezelőket deklarálnia két különböző módja van: Beágyazott lambdas, vagy az a osztálynak **OpenIdConnectEvents**. A második megközelítéssel ajánlott, ha az esemény-visszahívások bármely jelentős logikát, így azok feleslegesen ne tűzdelje tele az indítási osztályt. Referenciaimplementáció ezt a módszert használja.

### <a name="openid-connect-endpoints"></a>OpenID connect-végpontok

Az Azure AD támogatja [OpenID Connect-felderítési](https://openid.net/specs/openid-connect-discovery-1_0.html), viselkedésmintáit az identitásszolgáltató (IDP) adja vissza a JSON-metaadatok dokumentumok egy [jól ismert végpont](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig). A metaadat-dokumentum például tartalmaz információkat:

- Az engedélyezési végpont URL-címe Ez az, ahol az alkalmazás átirányítja a felhasználókat a felhasználó hitelesítéséhez.
- Az URL-címe az "a munkamenet befejezése" végpont, ahol az alkalmazás felhasználójának kijelentkeztetése ugrik.
- Az aláíró kulcsok, amelyek az ügyfél használatával, amely lekéri az identitásszolgáltató OIDC jogkivonatok érvényesítésére lekérése az URL-címe.

Alapértelmezés szerint a OIDC közbenső meg tudja beolvasni a metaadatok. Állítsa be a **hatóság** opcióhoz a közbenső szoftver, és a közbenső szoftver hoz létre a metaadatok URL-CÍMÉT. (Beállításával felülbírálhatja a metaadatok URL-címe a **MetadataAddress** beállítás.)

### <a name="openid-connect-flows"></a>OpenID connect folyamatok

Alapértelmezés szerint a OIDC közbenső hibrid folyamat használja az űrlap post Válaszmód.

- *Hibrid folyamat* azt jelenti, hogy az ügyfél kérheti egy azonosító jogkivonat és a egy engedélyezési kód ugyanazon oda-vissza, az engedélyezési kiszolgáló.
- *Űrlap post összesítéshez mód* azt jelenti, hogy az engedélyezési kiszolgáló HTTP-közzététel kérelmet használ az azonosító jogkivonat és engedélyezési kód küldése az alkalmazásnak. Az értékek a következők űrlap-kulcstárolókat (tartalomtípus = "application/x-www-form-urlencoded").

Ha a OIDC közbenső szoftver az engedélyezési végpontra irányítja, az átirányítási URL-cím tartalmazza az összes OIDC által igényelt, a lekérdezési karakterlánc paraméterei. A hibrid flow:

- client_id. Ez az érték meg a **ClientId** lehetőség
- hatókör = "openid profile", ami azt jelenti, OIDC kérelmet, és szeretnénk, hogy a felhasználó profilját.
- response_type = "code id_token". Azt határozza meg a hibrid folyamat.
- response_mode = "form_post". Azt határozza meg a post válasz.

Adjon meg egy másik folyamat, állítsa be a **ResponseType** tulajdonság a beállítások. Példa:

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.ResponseType = "code"; // Authorization code flow

    // Other options
}
```

[**Tovább**][claims]

[claims]: claims.md
[cookie-options]: /aspnet/core/security/authentication/cookie#controlling-cookie-options
[session-cookie]: https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
