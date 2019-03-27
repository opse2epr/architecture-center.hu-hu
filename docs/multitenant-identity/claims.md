---
title: Több-bérlős alkalmazásokban jogcímalapú identitások használata
description: Hogyan használja a kibocsátó érvényesítése és az engedélyezés jogcímek.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 8b8cbd2b857493d94103e80f53f187207feaa11e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298433"
---
# <a name="work-with-claims-based-identities"></a>Jogcímalapú identitások használata

[![GitHub](../_images/github.png) Mintakód][sample application]

## <a name="claims-in-azure-ad"></a>Jogcímek, az Azure ad-ben

Amikor egy felhasználó bejelentkezik, az Azure AD elküldi egy azonosító jogkivonat, amely a felhasználóval kapcsolatos jogcímek egy készletét tartalmazza. Jogcím egyszerűen egy információt egy kulcs/érték pár kifejezve. Például: `email`=`bob@contoso.com`.  Jogcímek rendelkezik egy kibocsátó &mdash; ebben az esetben az Azure AD &mdash; Ez az a entitás, amely hitelesíti a felhasználót, és a jogcímeket hoz létre. Megbízik a jogcímeket, mivel a kibocsátó megbízik. (Ezzel szemben, ha a kiállító nem megbízható, nem megbízható jogcímeket!)

Magas szintű:

1. A felhasználó hitelesíti magát.
2. Az Identitásszolgáltató elküldi a jogcímeket.
3. Az alkalmazás normalizálja, vagy úgy bővíti a jogcímek (nem kötelező).
4. Az alkalmazás használ a jogcímek engedélyezési döntésekhez.

Az OpenID Connect, kap jogcímkészletet vezérli a [hatókör-paramétert] a hitelesítési kérelem. Azonban az Azure AD kibocsát egy korlátozott számú OpenID Connect; keresztül áramló jogcímeket Lásd: [Támogatott token- és jogcímtípusok]. Ha azt szeretné, hogy a felhasználó további információt, szüksége lesz az Azure AD Graph API.

Íme néhány a jogcímek, az aad-ből, amelyek az alkalmazás általában előfordulhat, hogy érdeklik:

| Azonosító jogkivonat a jogcím típusa | Leírás |
| --- | --- |
| AUD |Akik a jogkivonat van kiadva. Ez lesz az alkalmazás ügyfél-azonosítót. Általában nem kell aggódnia a kérelmet, mert az a közbenső automatikusan érvényesíti. Példa:  `"91464657-d17a-4327-91f3-2ed99386406f"` |
| csoportok |AAD-csoportokat, amely tagja a felhasználó listáját. Például: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]` |
| iss |A [Kiállító] OIDC jogkivonat. Például: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/` |
| név |A felhasználó megjelenített neve. Például: `"Alice A."` |
| objektumazonosító |Az aad-ben a felhasználói objektum azonosítója. Ez az érték a felhasználó azonosítója nem módosítható és nem újrahasználható. Az érték, email, használja a felhasználók számára; egyedi azonosítóként e-mail-címet használva módosítható. Az Azure AD Graph API használata az alkalmazásban, objektumazonosító:-e a lekérdezés profiladatok használt érték. Például: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"` |
| roles |A felhasználó alkalmazás-szerepkörök listáját.    Például: `["SurveyCreator"]` |
| TID |Bérlő azonosítója. Ezt az értéket az Azure AD-ben az egyedi azonosító a bérlő számára. Például: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"` |
| unique_name |A felhasználó az emberi olvasható megjelenített neve. Például: `"alice@contoso.com"` |
| egyszerű felhasználónév |Egyszerű felhasználónév. Például: `"alice@contoso.com"` |

Ez a táblázat felsorolja a jogcímtípusok, ahogy azok megjelennek a azonosító jogkivonat. Az ASP.NET Core az OpenID Connect közbenső szoftvert átalakítja a jogcím-típusok amikor feltölti a jogcím-gyűjtemény a felhasználó rendszerbiztonsági tag:

* oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`
* tid > `http://schemas.microsoft.com/identity/claims/tenantid`
* unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`
* egyszerű felhasználónév > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`

## <a name="claims-transformations"></a>A jogcímek átalakítása

A hitelesítési folyamat során érdemes módosítani a jogcímek, az Identitásszolgáltató képest. ASP.NET Core, a jogcímek átalakításáról található hajthat végre a **AuthenticationValidated** eseményt az OpenID Connect közbenső szoftvert. (Lásd: [hitelesítési események].)

Bármely során hozzáadott jogcím **AuthenticationValidated** a munkamenet hitelesítési cookie vannak tárolva. Ezek nem get küldi vissza az Azure ad-hez.

Íme néhány példa a jogcímek átalakításáról:

* **Jogcím-normalizálási**, vagy a jogcímszolgáltatói konzisztens felhasználók között. Ez akkor különösen fontos, ha a több identitásszolgáltató használatát, amely használható különböző jogcímtípusok hasonló információkat kap jogcímeket. Például az Azure AD elküldi egy "egyszerű felhasználónév" jogcímet, amely tartalmazza a felhasználó e-mail címe. Más identitásszolgáltató küldhet egy "e-mail" jogcímet. A következő kódot egy "e-mail" jogcímet a "egyszerű" jogcím alakítja át:

  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```

* Adjon hozzá **alapértelmezett jogcímértékek** , amelyek nincsenek jelen jogcímek &mdash; például egy felhasználó egy alapértelmezett szerepkör hozzárendelése. Bizonyos esetekben ez egyszerűsítheti a hitelesítési logikát.
* Adjon hozzá **egyéni jogcímtípusok** a felhasználó alkalmazás-specifikus információkat. Például előfordulhat, hogy tárolja a felhasználó adatait egy adatbázist. A hitelesítési jegy hozzáadhatja ezeket az információkat az egyéni jogcím. A jogcím tárolja a cookie-k, így csak az egyszeri bejelentkezési munkamenetenként adatbázisból való beolvasásához szükséges. Másrészről is érdemes kerülje a túl nagy a cookie-k, ezért érdemes kompromisszumot kötni a cookie-k méretének és adatbázis-keresések között kell.

A hitelesítési folyamat befejezése után a jogcímek érhetők el a `HttpContext.User`. Ezen a ponton kezelje őket egy csak olvasható gyűjteményként &mdash; például ezek segítségével engedélyezési döntésekhez.

## <a name="issuer-validation"></a>Kibocsátó érvényesítése

Az OpenID Connect a kiállító ("iss") jogcím azonosítja az Identitásszolgáltató az azonosító jogkivonat kibocsátó. OIDC hitelesítési folyamat része, hogy ellenőrizze, hogy a kibocsátó jogcím megegyezik-e a tényleges kibocsátó. A OIDC közbenső kezeli Ez az Ön számára.

Az Azure AD-ben a kibocsátó értékét az AD-bérlő minden egyedi (`https://sts.windows.net/<tenantID>`). Ezért egy alkalmazás egy további ellenőrzés, hogy a kibocsátó képviseli azt, hogy jelentkezzen be az alkalmazás egy bérlőt kell tennie.

Egybérlős alkalmazás esetében egyszerűen ellenőrizhető, hogy a kibocsátó-e a saját bérlő. Valójában a OIDC közbenső automatikusan elvégzi ezt alapértelmezés szerint. Egy több-bérlős alkalmazásban több kiállítók, a különböző bérlőkhöz tartozó engedélyeznie kell. Íme egy általános módszer használatához:

* A közbenső beállításokat OIDC, állítsa be **ValidateIssuer** hamis értékre. Ez az automatikus ellenőrzés kikapcsolása.
* Amikor egy bérlő regisztrál, a felhasználói adatbázis tárolja a bérlő és a kibocsátó.
* Minden alkalommal, amikor egy felhasználó bejelentkezik, keresse meg a kibocsátó adatbázisban. Ha a kiállító nem található, azt jelenti, hogy a bérlő még nem regisztrált. Átirányíthatja azokat egy bejelentkezési oldalára.
* Ön is volt tiltólistára egyes bérlők; például az ügyfelek, amelyek az előfizetés nem fizet.

További részletes tárgyalását lásd: [előfizetési és bérlőfelvétel egy több-bérlős alkalmazásban][signup].

## <a name="using-claims-for-authorization"></a>Engedélyezési jogcímek használata

A jogcímek, az a felhasználó identitását, már nem egy monolitikus entitást. Például egy felhasználó lehet e-mail cím, telefonszám, születésnap, nemét, stb. A felhasználó Identitásszolgáltató talán összes ezeket az információkat tárolja. De hitelesíteni a felhasználót, amikor általában kap egy része, ezek jogcímekként. Ebben a modellben a felhasználó identitását a jogcímek egyszerűen egy kötegelt. Amikor engedélyezéshez felhasználókkal kapcsolatos, adott részhalmazához jogcímek fogja keresni. Más szóval a kérdés végső soron "Nem felhasználói X rendelkezik igény Z" "X felhasználó művelet hajtható végre, Y" lesz.

Az alábbiakban néhány alapvető mintázatokból jogcímek ellenőrzéséhez.

* Ellenőrizze, hogy a felhasználó rendelkezik-e egy adott jogcím egy adott értékkel:

   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```

   Ez a kód ellenőrzi, hogy a felhasználó rendelkezik-e a szerepkör jogcím "Rendszergazda" értékű. Az eset, amelyhez a felhasználónak nincs szerepkör jogcím vagy több szerepkör jogcím van-e megfelelően kezeli.
  
   A **ClaimTypes** osztály jogcímtípusok gyakran használt állandókat határozza meg. Bármilyen karakterlánc típusú értéket használhatja azonban a jogcímtípushoz.
* Ha itt nem legfeljebb egy értéket várt egyetlen érték lekéréséhez a jogcím típusa:

  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```

* Egy jogcímtípust értékek lekéréséhez:

  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

További információkért lásd: [szerepkör- és erőforrás-alapú hitelesítés több-bérlős alkalmazásokban][authorization].

[**Tovább**][signup]

<!-- links -->

[hatókör-paramétert]: https://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[Támogatott token- és jogcímtípusok]: /azure/active-directory/active-directory-token-and-claims/
[Kiállító]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
[Hitelesítési események]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
