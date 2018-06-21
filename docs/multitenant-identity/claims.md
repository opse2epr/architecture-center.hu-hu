---
title: Jogcím-alapú identitásokat a több-bérlős alkalmazások használata
description: Hogyan használja a jogcím-a kibocsátó érvényesítése és az engedélyezés
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 61788d9759715b21ef1bdda59c5b54d923fd8f62
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541913"
---
# <a name="work-with-claims-based-identities"></a>Jogcímalapú identitás használata

[![GitHub](../_images/github.png) példakód][sample application]

## <a name="claims-in-azure-ad"></a>Jogcímek, az Azure ad-ben
Amikor egy felhasználó bejelentkezik, az Azure AD küld Azonosítót jogkivonatban a felhasználóval kapcsolatos jogcímek egy készletét tartalmazza. A jogcím egyszerűen egy, a kulcs/érték pár kifejezve. Például `email` = `bob@contoso.com`.  Jogcímek rendelkezik egy kibocsátó &mdash; ebben az esetben az Azure AD &mdash; Ez az az entitás, amely hitelesíti a felhasználót, és a jogcímeket hoz létre. A jogcímek megbízik, mivel a kibocsátó megbízik. (Ezzel szemben, ha nem bízik meg a kiállító, nem megbízható jogcímeket!)

Magas szinten:

1. A felhasználó hitelesíti magát.
2. Az IDP elküldi a jogcímek egy készletét.
3. Az alkalmazás normalizálja vagy szabályozva az információhasználat a jogcímeket (nem kötelező).
4. Az alkalmazás a jogcímek használja az engedélyezéshez.

Az OpenID Connect, a kapott jogcímek készlete vezérli a [hatókör-paramétert] a hitelesítési kérelem. Azonban az Azure AD kibocsát egy korlátozott körét jogcímek továbbítása OpenID Connect; Lásd: [támogatott jogkivonat és jogcímtípusok]. Ha azt szeretné, hogy a felhasználó további információt, meg kell az Azure AD Graph API-val.

Íme néhány a jogcímeket, előfordulhat, hogy általában fontos egy alkalmazás aad-ben:

| Az Azonosítót jogkivonatban a jogcím típusa | Leírás |
| --- | --- |
| és |Ki a token ki. Ez lesz az alkalmazás ügyfél-azonosítót. Általában nem szükséges foglalkoznia az ezt a kérelmet, mert a köztes automatikusan érvényesíti azt. Példa:`"91464657-d17a-4327-91f3-2ed99386406f"` |
| csoportok |AAD-csoportokat, amelyek a felhasználó tagja listáját. Példa:`["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]` |
| iss |A [kibocsátó] OIDC jogkivonat. Példa:`https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/` |
| név |A felhasználó megjelenített neve. Példa:`"Alice A."` |
| OID |Az aad-ben a felhasználó objektum azonosítója. Ez az érték a felhasználó nem módosítható, és egyszer használatos azonosító érték. Ezen a érték, nem e-mailt, egy egyedi azonosítóként felhasználó használhatja; e-mail címet használva módosítható. Ha az alkalmazás az Azure AD Graph API-t használ, objektumazonosító: adott profil adatlekérdezés használt érték. Példa:`"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"` |
| roles |A felhasználó alkalmazás szerepkörök listáját.    Példa:`["SurveyCreator"]` |
| TID |Bérlő azonosítója. Az értéke a bérlő egyedi azonosítója az Azure ad-ben. Példa:`"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"` |
| unique_name |A felhasználó emberi olvasható megjelenítendő neve. Példa:`"alice@contoso.com"` |
| egyszerű felhasználónév |Egyszerű felhasználónév. Példa:`"alice@contoso.com"` |

Ebben a táblázatban jelennek meg az Azonosítót jogkivonatban a jogcím-típusok található. Az ASP.NET Core az OpenID Connect köztes alakítja át a jogcímek valamelyikének, amikor a felhasználó egyszerű jogcímek gyűjteményében feltölt:

* OID >`http://schemas.microsoft.com/identity/claims/objectidentifier`
* TID >`http://schemas.microsoft.com/identity/claims/tenantid`
* unique_name >`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
* egyszerű felhasználónév >`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`

## <a name="claims-transformations"></a>A jogcímek átalakítása
A hitelesítési folyamat során előfordulhat, hogy szeretné a jogcímeket, hogy a kiállító terjesztési hely módosítása. Az ASP.NET Core jogcímek átalakítása belül végezheti el a **AuthenticationValidated** az OpenID Connect köztes eseményt. (Lásd: [hitelesítési események].)

Bármely során hozzáadott jogcím **AuthenticationValidated** a munkamenet hitelesítési cookie tárolja. Ezek nem get leküldött vissza az Azure AD.

Íme néhány példa a jogcímek átalakításáról:

* **Jogcím-normalizálási**, vagy a jogcímek konzisztens felhasználók között. Ez különösen fontos, ha több IDPs, használhatja a különböző jogcímtípusok hasonló információk a jogcímek kap.
  Az Azure AD például egy "" jogcím, amely tartalmazza a felhasználói e-mailt küld. Más IDPs előfordulhat, hogy az "e-mail" jogcímet küld ki. A következő kód egy "e-mail" jogcímet alakítja át a "" jogcím:
  
  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```
* Adja hozzá **alapértelmezett jogcímértékek** a jogcímek, amelyek nem &mdash; például egy felhasználó hozzárendelése egy alapértelmezett szerepkör. Egyes esetekben ez egyszerűbbé teszi hitelesítési logikát.
* Adja hozzá **egyéni jogcímtípusok** az a felhasználó az alkalmazás-specifikus adatait. Például előfordulhat, hogy tárolja a felhasználó adatait adatbázisban. A hitelesítési jegy hozzáadhatja ezeket az adatokat egy egyéni jogcímleírásokat. A jogcím egy cookie-ban tárolja, így csak akkor kell töltse le innen bejelentkezési munkamenetenként egyszer az adatbázisból. Másrészről érdemes ne hozzon létre túl nagy a cookie-k, ezért figyelembe kell vennie a kompromisszum közötti adatbázis-keresések és cookie-méretet.   

A hitelesítési folyamat befejezése után a rendszer a jogcímeket érhetők el `HttpContext.User`. Ezen a ponton kell tekinteni őket egy csak olvasható gyűjteményként &mdash; pl., amelyekkel engedélyezési döntésekhez.

## <a name="issuer-validation"></a>Kibocsátó érvényesítése
Az OpenID Connect a kibocsátó jogcím ("iss") a kiállító terjesztési hely a azonosító jogkivonatot kibocsátó azonosítja. Győződjön meg arról, hogy a kibocsátó jogcím egyezik a tényleges kibocsátó OIDC hitelesítési folyamatának részeként is. A OIDC köztes kezeli ezt meg.

Az Azure ad-ben, a kibocsátó értéke egyedi AD bérlőnként (`https://sts.windows.net/<tenantID>`). Ezért az alkalmazás tegyen egy további ellenőrzést, győződjön meg arról, hogy a kibocsátó képviseli azt, hogy jelentkezzen be az alkalmazás a bérlő számára.

Single-bérlő alkalmazáshoz csak ellenőrizheti, hogy a kibocsátó-e saját bérlőt. Valójában a OIDC köztes automatikusan elvégzi ezt alapértelmezés szerint. Egy több-bérlős alkalmazásban, engedélyeznie kell a több kiállítók, a különböző bérlőkhöz tartozó. Ez egy általános módszer használatához:

* Állítsa be az OIDC köztes beállításai **ValidateIssuer** false értékre. Ez kikapcsolja az automatikus ellenőrzést.
* Amikor a bérlő előfizet, a felhasználói adatbázis tárolja a bérlő és a kibocsátó.
* Amikor egy felhasználó bejelentkezik, keresse meg a kibocsátó adatbázisban. Ha a kibocsátó nem található, az azt jelenti, hogy a bérlő még nem regisztrált. Akkor is átirányítja őket egy bejelentkezési oldalára.
* Ön is volt tiltólistára egyes bérlők; például az ügyfelek, amelyek nem kell fizetnie az előfizetését.

További részletes tárgyalását lásd: [-előfizetés és a bérlők egy több-bérlős alkalmazásban bevezetési][signup].

## <a name="using-claims-for-authorization"></a>A hitelesítéshez jogcímeket használó
A jogcímek, az a felhasználói identitás már nincs egységes entitás. A felhasználó Előfordulhat például, e-mail címét, telefonszámát, születési, nemét, stb. Lehet, hogy a felhasználó IDP az összes olyan tárolja ezeket az információkat. De ha hitelesíteni a felhasználót, általában kap egy részét, ezek jogcímekként. Ebben a modellben a felhasználó identitását a jogcímek egyszerűen egy kötegelt. Amikor engedélyezéshez felhasználókkal kapcsolatos, adott beállítása jogcímek fogja keresni. Más szóval a kérdés végső soron "Nem felhasználói X rendelkezik igény Z" "X felhasználói művelet végrehajtása Y" lesz.

Az alábbiakban néhány alapvető mintázatokból jogcímek ellenőrzéséhez.

* Ellenőrizze, hogy a felhasználó rendelkezik-e egy adott jogcímet, és az egy adott értéket:
  
   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```
   Ez a kód ellenőrzi, hogy a felhasználó rendelkezik-e a szerepkör jogcím "Rendszergazda" értékű. Megfelelően kezeli az esetben, ha a felhasználó nincs szerepkör jogcím vagy több szerepkör jogcím rendelkezik-e.
  
   A **ClaimTypes** osztály állandók a gyakran használt jogcímtípusok meghatározása. Azonban a jogcímtípus bármilyen karakterlánc típusú értéket is használhatja.
* Ha több érték nagyobb, mint tudjanak várhatóan egyetlen érték lekérése a jogcím típusa:
  
  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```
* Összes érték lekérése a jogcím típusa:
  
  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

További információkért lásd: [több-bérlős alkalmazásokhoz a szerepkör- és erőforrás-alapú engedélyezési][authorization].

[**Következő**][signup]


<!-- Links -->

[hatókör-paramétert]: http://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[támogatott jogkivonat és jogcímtípusok]: /azure/active-directory/active-directory-token-and-claims/
[kibocsátó]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[hitelesítési események]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
