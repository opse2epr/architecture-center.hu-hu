---
title: Alkalmazás-szerepkörök
description: Alkalmazás-szerepkörök használatával engedélyezési végrehajtása
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: a39c64f003c26f860086701dd988a8bb21fab5bf
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="application-roles"></a>Alkalmazás-szerepkörök

[![GitHub](../_images/github.png) példakód][sample application]

Alkalmazási szerepköröknek segítségével engedélyek hozzárendelése a felhasználókhoz. Például a [Dejójáték felmérések] [ Tailspin] alkalmazás határozza meg a következő szerepkörök:

* Rendszergazda. Minden CRUD-műveleteknek a bármely felmérés, amely arra a bérlőre tartozik hajthat végre.
* Létrehozó. Új felmérések hozhat létre.
* Olvasó. Bármely felmérések arra a bérlőre tartozó el tud olvasni.

Láthatja, hogy szerepkörök végső soron beolvasása lefordítani engedélyek során [engedélyezési]. De az első kérdést hozzárendelése és szerepkörök kezelése. Felismertük három fő lehetőség közül választhat:

* [Az Azure AD alkalmazás-szerepkörök](#roles-using-azure-ad-app-roles)
* [Azure AD biztonsági csoportokat](#roles-using-azure-ad-security-groups)
* [Alkalmazás-szerepkör kezelő](#roles-using-an-application-role-manager).

## <a name="roles-using-azure-ad-app-roles"></a>A szerepkörök Azure AD alkalmazás-szerepkörök használatával
Ez a megközelítés a Dejójáték felmérések alkalmazásban használt.

Ezt a módszert használja a Szolgáltatottszoftver-szolgáltató határozza meg az alkalmazás-szerepkörök hozzáadását az alkalmazás jegyzékében. Miután az ügyfél iratkozik fel, az ügyfél Active Directory rendszergazdája felhasználók rendel a szerepkörök. Amikor egy felhasználó bejelentkezik, a felhasználó hozzárendelt szerepkörök küldése a jogcímeket.

> [!NOTE]
> Ha az ügyfél rendelkezik az Azure AD Premium, a rendszergazda biztonsági csoport hozzárendelése a szerepkörhöz, és a csoport tagjai öröklik az alkalmazás-szerepkör. Ez kényelmes módja a kezelendő szerepköröket, mert a csoport tulajdonosának nem kell lenniük egy AD a rendszergazdának.
> 
> 

A módszer előnyei:

* Egyszerű programozási modell.
* Szerepkörök az alkalmazáshoz. A szerepkör jogcím egy alkalmazás nem kap meg egy másik alkalmazás.
* Ha az ügyfél az AD-bérlő eltávolítja az alkalmazást, a szerepkörök eltűnik.
* Az alkalmazás nem szükséges minden további Active Directory-engedélyek eltérő a felhasználói profil olvasása.

Hátrányai:

* Prémium szintű Azure AD nélkül az ügyfelek nem biztonsági csoportok hozzárendelése szerepkörökhöz. Ezek az ügyfelek minden felhasználó hozzárendelését AD rendszergazda kell elvégezni.
* Háttér webes API-k, amely nem csatlakozik a webalkalmazást, ha a webalkalmazáshoz tartozó szerepkör-hozzárendelések a webes API-k nem vonatkozik. Ezt a pontot további tárgyalását lásd: [a háttér webes API biztonságossá tétele].

### <a name="implementation"></a>Megvalósítás
**A Szerepkörök definiálása.** A Szolgáltatottszoftver-szolgáltató deklarálja az alkalmazás szerepkörök a [alkalmazásjegyzék]. Például itt bejegyzés a jegyzékfájl felmérések alkalmazás:

```
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

A `value` tulajdonság a szerepkör jogcím jelenik meg. A `id` tulajdonság a megadott szerepkör egyedi azonosítója. Mindig létrehozni egy új GUID azonosítót `id`.

**Felhasználók hozzárendelése**. Ha egy új ügyfél iratkozik fel, az alkalmazás regisztrálva van az ügyfél AD-bérlő. Ezen a ponton AD, hogy a bérlői rendszergazda megoldással felhasználókat rendelhet szerepköröket.

> [!NOTE]
> Ahogy azt korábban említettük, az Azure AD Premium ügyfelek csoportokat is rendelhet biztonsági szerepkörökhöz.
> 
> 

Az Azure-portálon az alábbi képernyőfelvételen látható felhasználóinak és csoportjainak a felmérést alkalmazás. A rendszergazda és a létrehozó olyan csoportok, illetve SurveyAdmin és SurveyCreator szerepkörökhöz hozzárendelt. Alice az a felhasználó, aki közvetlenül a SurveyAdmin szerepkör lett hozzárendelve. Bob és Charles, amely rendelkezik nem közvetlenül a szerepkörhöz hozzárendelt felhasználók.

![Felhasználók és csoportok](./images/running-the-app/users-and-groups.png)

Az alábbi képernyőképen látható, Charles része a felügyeleti csoport, hogy örökli a SurveyAdmin szerepkör. Bob, esetén Phil van még nincs hozzárendelve szerepkör.

![Felügyeleti csoport tagjai](./images/running-the-app/admin-members.png)


> [!NOTE]
> Egy másik módszert is van, az alkalmazás, amelyekhez szerepkörök rendelhetők programozott módon, az Azure AD Graph API-val. Azonban ehhez az alkalmazás beszerzése az ügyfél Active directory írási engedéllyel. Ezeket az engedélyeket az alkalmazás végrehajtja kárt számos &mdash; az ügyfél az alkalmazásnak, hogy nem a könyvtár mess delegálását. Előfordulhat, hogy sok ügyfél hajtja adja meg a hozzáférési szintet.
> 

**Szerepkör jogcím beolvasása**. Amikor egy felhasználó bejelentkezik, az alkalmazás típusú jogcímet a felhasználó hozzárendelt szerepkör(ök) fogadása `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.  

A felhasználó több szerepkört, vagy nincs szerepkör lehet. Az engedélyezési kód nem érdemes feltételezni a felhasználó rendelkezik-e jogcím pontosan egy szerepkört. Ehelyett kód írását, amely ellenőrzi, hogy jelen-e egy adott jogcím értéke:

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a>Szerepkörök az Azure AD biztonsági csoportokat használ
Ezt a módszert használja, a szerepkörök helyettesítik AD biztonsági csoportokat. Az alkalmazás engedélyeket rendeli, a felhasználóknak a biztonsági csoporttagságok alapján.

Előnyei:

* Prémium szintű Azure AD nem rendelkező ügyfelek Ez a megközelítés lehetővé teszi, hogy az ügyfél biztonsági csoportok segítségével kezelheti a szerepkör-hozzárendelések.

Hátrányait:

* Összetettségét. Mivel minden bérlő különböző csoportjogcímek küldi el, az alkalmazás kell nyomon követjük, hogy mely alkalmazás szerepköröket, az egyes bérlők számára megfelelő biztonsági csoportokat.
* Ha az ügyfél az AD-bérlő eltávolítja az alkalmazást, a biztonsági csoportokat az Active directory maradnak.

### <a name="implementation"></a>Megvalósítás
Az alkalmazásjegyzékben, állítsa be a `groupMembershipClaims` tulajdonság "a(z)"biztonsági csoporthoz. Ez szükséges tagsági csoportjogcímek lekérése az aad-ben.

```
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

Egy új ügyfél iratkozik fel, ha az alkalmazás utasítja az ügyfél a szerepköröket, az alkalmazás számára szükséges biztonsági csoportok létrehozása. Az ügyfél majd be kell írnia a csoportházirend-objektum azonosítók az alkalmazásba. Az alkalmazás tárolja ezeket, amely leképezhető csoportazonosítóval alkalmazási szerepköröknek bérlőnként táblázatban.

> [!NOTE]
> Másik lehetőségként az alkalmazás létrehozhat a csoportok programozott módon, az Azure AD Graph API-val.  Ez lehet kevesebb a hibalehetőség hiba. Az alkalmazás a "olvasása és írása az összes csoport" igényel, azonban az ügyfél Active directory engedélyeit. Előfordulhat, hogy sok ügyfél hajtja adja meg a hozzáférési szintet.
> 
> 

Ha felhasználó jelentkezik be:

1. Az alkalmazás a felhasználói csoportok jogcímekként kap. Az egyes jogcímek értéke egy csoport Objektumazonosítója.
2. Az Azure AD a jogkivonat küldött csoportok számának korlátozása. Ha csoportok száma meghaladja ezt a határt, a Azure AD egy különös "keretét" jogcímet küldi el. Ha meg adva, hogy a jogcímek, az alkalmazás az Azure AD Graph API minden a tartalmazó csoportok, hogy a felhasználó kell lekérdeznie. További információkért lásd: [felhőalapú alkalmazások AD-csoportok használata az engedélyezési], a "Csoportok igény túlhasználati" című szakaszban.
3. Az alkalmazás a saját adatbázisában található a megfelelő alkalmazás-szerepkörök hozzárendelése a felhasználóhoz a objektumazonosítók keres.
4. Az alkalmazás egy egyéni jogcím értékét hozzáadja az alkalmazás-szerepkör kifejezze egyszerű felhasználónév. Például: `survey_role` = "SurveyAdmin".

Engedélyezési házirendek használjon egyéni szerepkör jogcím jogcím-nem a csoport.

## <a name="roles-using-an-application-role-manager"></a>Egy alkalmazás-szerepkör kezelővel szerepkörök
Ezt a módszert alkalmazási szerepköröknek nem tárolja az Azure ad-ben minden. Ehelyett az alkalmazás tárolja a szerepkör-hozzárendelések az egyes felhasználók a saját DB &mdash; használata esetén például a **RoleManager** osztály ASP.NET-identitásban.

Előnyei:

* Az alkalmazás a szerepkörök és a felhasználó-hozzárendeléseket történő teljes körű vezérléssel rendelkezik.

Hátrányai:

* Összetettebb, nehezebb karbantartása.
* Active Directory biztonsági csoportokat nem használható a szerepkör-hozzárendelések kezeléséhez.
* Felhasználói adatokat tárol az alkalmazás adatbázisban, ahol előbbi beszerezheti a szinkronban a bérlő az Active directory, mivel a felhasználók hozzáadásakor vagy eltávolításakor.   


[**Következő**][engedélyezési]

<!-- Links -->
[Tailspin]: tailspin.md

[engedélyezési]: authorize.md
[a háttér webes API biztonságossá tétele]: web-api.md
[alkalmazásjegyzék]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
