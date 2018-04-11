---
title: Hitelesítés a több-bérlős alkalmazásokhoz
description: Hogyan egy több-bérlős alkalmazást az Azure ad-felhasználókat hitelesítheti
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
ms.openlocfilehash: e85817626675cec4d126921c19a31a0983ecd62d
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2017
---
# <a name="authenticate-using-azure-ad-and-openid-connect"></a>Hitelesítés az Azure AD használatával és az OpenID Connect

[![GitHub](../_images/github.png) Mintakód][sample application]

A felmérések alkalmazás az OpenID Connect (OIDC) protokoll használatával hitelesíti a felhasználókat az Azure Active Directory (Azure AD). A felmérések alkalmazás, amelynek beépített köztes OIDC az ASP.NET Core használja. Az alábbi ábrán látható, hogy mi történik, ha a felhasználó bejelentkezik, magas szinten.

![Hitelesítési folyamat](./images/auth-flow.png)

1. Kattintás a "bejelentkezés" gombra az alkalmazásban. Ez a művelet az MVC-vezérlő kezeli.
2. Az MVC-vezérlő értéket ad vissza egy **ChallengeResult** művelet.
3. A köztes elfogja a **ChallengeResult** , és létrehoz egy 302-es választ, amely átirányítja a felhasználót az Azure AD bejelentkezési oldalára.
4. A felhasználók hitelesítése az Azure ad-val.
5. Az Azure AD egy azonosító tokent elküldi az alkalmazásnak.
6. A köztes érvényesíti a azonosító jogkivonatot. Ezen a ponton a felhasználói most már hitelesítve az alkalmazáson belüli.
7. A köztes átirányítja a felhasználói alkalmazás.

## <a name="register-the-app-with-azure-ad"></a>Az alkalmazás regisztrálása az Azure ad szolgáltatással
OpenID Connect engedélyezéséhez a Szolgáltatottszoftver-szolgáltató regisztrálja az alkalmazást a saját Azure AD-bérlő belül.

Az alkalmazás regisztrálásához kövesse [alkalmazások integrálása az Azure Active Directoryval](/azure/active-directory/active-directory-integrating-applications/), szakaszában [egy alkalmazás hozzáadása](/azure/active-directory/active-directory-integrating-applications/#adding-an-application).

Lásd: [futtassa a felmérések alkalmazást](./run-the-app.md) a felmérések alkalmazására vonatkozó lépéseit. Vegye figyelembe a következőket:

- Egy több-bérlős alkalmazást konfigurálnia kell a multi-központjaként beállítást explicit módon. Ez lehetővé teszi, hogy más szervezetek számára, az alkalmazás eléréséhez.

- A válasz URL-címe: az URL-cím, ahol az Azure AD küldi a válaszokat OAuth 2.0-s. Az ASP.NET Core használata esetén ez egyezniük kell az elérési utat, amelyet megadtak a hitelesítési köztes (lásd a következő szakaszban), 

## <a name="configure-the-auth-middleware"></a>A hitelesítési köztes konfigurálása
Ez a szakasz ismerteti a hitelesítési köztes konfigurálása az ASP.NET Core az OpenID Connect több-bérlős hitelesítéshez.

Az a [indítási osztály](/aspnet/core/fundamentals/startup), az OpenID Connect köztes hozzáadása:

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

Figyelje meg, hogy a beállításokat a konfigurációs beállításai futásidejű művelet. Ez a köztes beállítások jelenti:

* **ClientId**. Az alkalmazás ügyfél-azonosító az Azure ad-ben az alkalmazás regisztrálásakor kapott.
* **Szolgáltató**. Egy több-bérlős alkalmazáshoz állítsa ezt a beállítást `https://login.microsoftonline.com/common/`. Ez az az Azure AD közös végpont, amely lehetővé teszi a felhasználók bármely Azure AD-bérlő jelentkezzen be az URL-CÍMÉT. A közös végpont kapcsolatos további információkért lásd: [ebben a blogbejegyzésben](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/).
* A **TokenValidationParameters**, beállíthatja **ValidateIssuer** false értékre. Ez azt jelenti, hogy az alkalmazás kell konfigurálnia a kibocsátó érték a Azonosítót jogkivonatban ellenőrzése. (A köztes továbbra is érvényesíti a jogkivonatot magát.) A kibocsátó ellenőrzésével kapcsolatos további információkért lásd: [kibocsátó érvényesítése](claims.md#issuer-validation).
* **PostLogoutRedirectUri**. Adja meg a felhasználók a kijelentkezés után átirányítási URL-CÍMÉT. Ez legyen egy oldal, amely lehetővé teszi a névtelen kérelmek &mdash; általában a kezdőlap.
* **SignInScheme**. Állítsa ezt a beállítást `CookieAuthenticationDefaults.AuthenticationScheme`. Ez a beállítás azt jelenti, hogy a felhasználó hitelesítése után a felhasználói jogcímeket kell tárolni helyileg egy cookie-ban. Ez a cookie, hogy a felhasználó bejelentkezett marad a böngésző-munkamenet során.
* **Események.** Esemény visszahívások; Lásd: [hitelesítési események](#authentication-events).

A hitelesítési cookie-k köztes is hozzáadhat a feldolgozási sor. A köztes írása egy cookie-t a felhasználói jogcímeket, és majd olvasása során a következő lapon terhelések a cookie-k felelős.

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

## <a name="initiate-the-authentication-flow"></a>A hitelesítési folyamat kezdeményezéséhez
A hitelesítési folyamat elindításához az ASP.NET mvc-ben, térjen vissza a **ChallengeResult** a a contoller:

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

Ennek hatására a közbenső szoftvert 302-es (található) választ, amely átirányítja a hitelesítési végpontra.

## <a name="user-login-sessions"></a>Felhasználói munkamenetek
Ahogy azt korábban említettük, ha a felhasználó először jelentkezik be, a hitelesítési cookie-k köztes ír egy cookie-t a felhasználói jogcímeket. Ezt követően HTTP-kérések hitelesítése a cookie-k olvasásakor.

Alapértelmezés szerint a cookie-k köztes ír egy [munkamenetcookie][session-cookie], mely lekérdezi törlése után a felhasználó bezárja a böngészőt. A felhasználó ezután felkeresi a webhelyet, amikor legközelebb rendelkeznek újból bejelentkezni. Azonban ha **IsPersistent** igaz értékű a **ChallengeResult**, a köztes ír egy állandó cookie-t, a böngésző bezárása után a felhasználó bejelentkezett marad. Beállíthatja, hogy a cookie lejárati; Lásd: [cookie-k beállítások szabályozása][cookie-options]. Állandó cookie-k a felhasználók kényelmesebb, de lehet, hogy egyes alkalmazások (szóbeli, egy banki alkalmazás) nem megfelelő Ha azt szeretné, hogy a felhasználót, hogy minden egyes bejelentkezés.

## <a name="about-the-openid-connect-middleware"></a>Az OpenID Connect köztes kapcsolatos
Az OpenID Connect köztes az ASP.NET elrejti a legtöbb protokoll részleteit. Ez a szakasz néhány megjegyzés, amely lehet hasznos, ha a protokoll folyamat ismertetése megvalósításával kapcsolatban.

Első lépésként vizsgáljuk meg a hitelesítési folyamat tekintetében az ASP.NET (az alkalmazás és az Azure AD közötti OIDC protokoll folyamat részleteit figyelmen kívül hagyva). Az alábbi ábrán látható, a folyamat.

![Bejelentkezés folyamata](./images/sign-in-flow.png)

Ezen a diagramon nincsenek két MVC-vezérlők. A fiók vezérlő bejelentkezési kéréseket kezeli, és az otthoni tartományvezérlő fel a kezdőlap szolgál.

A hitelesítési folyamat a következő:

1. A felhasználó a "Bejelentkezés" gombra kattint, és a böngésző GET kérés küldése. Például: `GET /Account/SignIn/`.
2. A fiók vezérlő értéket ad vissza egy `ChallengeResult`.
3. A OIDC köztes egy HTTP 302 választ, az Azure AD-átirányítás ad vissza.
4. A böngészőben a hitelesítési kérést küld az Azure AD
5. A felhasználó bejelentkezik az Azure AD és az Azure AD választ küld vissza hitelesítés.
6. A OIDC köztes rendszerbiztonsági tag jogcímeket hoz létre, és a hitelesítési cookie-k köztes számára továbbítja azokat.
7. A cookie-k köztes rendezi sorba a jogcímek rendszerbiztonsági tag, és beállítja egy cookie-t.
8. A OIDC köztes az alkalmazás visszahívási URL-címre irányítja át.
9. A böngésző követi az átirányítást, a cookie-k a kérelem küldése.
10. A cookie-k köztes deserializes egy egyszerű jogcímek a cookie-k, és beállítja a `HttpContext.User` egyenlő a jogcímek rendszerbiztonsági tag. A kérelem az MVC-vezérlőhöz történik.

### <a name="authentication-ticket"></a>Hitelesítési jegy
Ha a hitelesítés sikeres, a OIDC köztes hitelesítési jegyre, amely tartalmaz, amely tárolja a felhasználói jogcímek jogcímek rendszerbiztonsági tag hoz létre. A jegy belül végezheti el a **AuthenticationValidated** vagy **TicketReceived** esemény.

> [!NOTE]
> Az egész hitelesítési folyamat befejezéséig `HttpContext.User` továbbra is rendelkezik egy névtelen egyszerű **nem** a hitelesített felhasználó. A névtelen rendszerbiztonsági tag egy üres jogcímek gyűjteményt tartalmaz. Hitelesítés befejeződik, és az alkalmazás átirányítja a cookie-k köztes deserializes után a hitelesítési cookie-t és a készletek `HttpContext.User` a jogcímek rendszerbiztonsági tag, amely a hitelesített felhasználó jelöli.
> 
> 

### <a name="authentication-events"></a>Hitelesítési események
A hitelesítési folyamat során az OpenID Connect köztes események sorozatát riasztást:

* **Redirecttoidentityprovider metódus**. A köztes átirányítja a felhasználókat a hitelesítési végpontra hívni a jogosultsággal. Ez az esemény segítségével módosíthatja az átirányítási URL-címet; például adja meg a kérelem paramétereit. Lásd: [hozzáadása a felügyeleti beleegyezést kérő üzenete](signup.md#adding-the-admin-consent-prompt) példát.
* **AuthorizationCodeReceived**. Az engedélyezési kód hívása.
* **TokenResponseReceived**. Neve a köztes lekérése egy hozzáférési jogkivonat a kiállító terjesztési hely, de előtt érvényességét. Csak hitelesítésikód-folyamata vonatkozik.
* **TokenValidated**. Hívása után a köztes érvényesíti a azonosító jogkivonatot. Ezen a ponton az alkalmazás rendelkezik érvényesített, a felhasználóval kapcsolatos jogcímek egy készletét. Ezt az eseményt további érvényesítést hajt végre a rendszer a jogcímeket, illetve jogcímek átalakítására használható. Lásd: [jogcímek használata](claims.md).
* **UserInformationReceived**. Ha a köztes lekérése a felhasználói adatok végpont a felhasználói profil neve. Csak hitelesítésikód-folyamata, és csak akkor vonatkozik `GetClaimsFromUserInfoEndpoint = true` a köztes beállításai.
* **TicketReceived**. Hívható meg, ha a hitelesítés végbemegy. Ez az az utolsó esemény, feltéve, hogy a hitelesítés sikeres lesz. Után ez az esemény történik, a felhasználó jelentkezett be az alkalmazásba.
* **AuthenticationFailed**. Meghívni, ha a hitelesítés sikertelen. Ez az esemény használatával kezeli a hitelesítési hibák &mdash; például úgy, hogy egy hibalap irányít át.

Adja meg a visszahívások ezeket az eseményeket, állítsa be a **események** a köztes beállítást. Az eseménykezelők deklarálnia két különböző módja van: beágyazott lambda kifejezések, illetve az abból származó osztály **OpenIdConnectEvents**. A második megközelítést akkor ajánlott, ha az esemény visszahívások rendelkezik bármely jelentős logika, így azok nem megzavarhatják a indítási osztályt. A hivatkozás ezt a módszert használ.

### <a name="openid-connect-endpoints"></a>OpenID connect végpontok
Az Azure AD által támogatott [OpenID Connect felderítési](https://openid.net/specs/openid-connect-discovery-1_0.html), amelynek az identitásszolgáltató (IDP) egy a JSON-metaadatok dokumentumot ad vissza egy [jól ismert végpont](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig). A metaadat-dokumentum tartalmaz információkat, mint:

* Az engedélyezési végpont URL-CÍMÉT. Ez azért, ahol az alkalmazás átirányítja a felhasználókat a felhasználó hitelesítésére.
* A "munkamenet befejezése" végpont, ahol az alkalmazás jelentkezzen ki a felhasználó ugrik URL-CÍMÉT.
* Az aláíró kulcsok, amelyet az ügyfél használ, amely azt lekérése az IDP OIDC jogkivonatok érvényesítésére beolvasása az URL-címe.

Alapértelmezés szerint a OIDC köztes meg tudja a metaadatok beolvasása. Állítsa be a **hatóság** a köztes, és a köztes hoz létre a metaadatok URL-CÍMÉT. (A metaadatainak URL-CÍMÉT úgy, hogy lehet felülbírálni a **MetadataAddress** lehetőséget.)

### <a name="openid-connect-flows"></a>OpenID connect adatfolyamok
Alapértelmezés szerint a OIDC köztes hibrid folyamat használja az űrlap post válasz mód.

* *Hibrid folyamat* azt jelenti, hogy az ügyfél kérheti le egy azonosító jogkivonat és egy engedélyezési kód ugyanazon oda-vissza az engedélyezési kiszolgálóra.
* *Űrlap post válasz mód* azt jelenti, hogy a hitelesítési kiszolgáló egy HTTP POST kérelem használja a azonosító token és engedélyezési kódot küldeni az alkalmazás. Az értékek a következők űrlap-urlencoded (tartalomtípus = "application/x-www-form-urlencoded").

A OIDC köztes átirányítja az engedélyezési végpont, amikor az átirányítási URL-címet tartalmazza az összes OIDC által igényelt lekérdezési karakterlánc paramétert. A hibrid folyamata:

* client_id. Ez az érték meg a **ClientId** beállítás
* hatókör = "openid profil", ami azt jelenti, OIDC kérelmet, és azt szeretnénk, hogy a felhasználói profil.
* response_type = "code id_token". Azt határozza meg a hibrid folyamata.
* response_mode = "form_post". Azt határozza meg az űrlap post válasz.

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
