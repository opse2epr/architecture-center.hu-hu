---
title: Alkalmazás-szerepkörök
description: 'Hogyan végezheti el a hitelesítést: az alkalmazás-szerepkörök.'
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: 097b150f3706614b0814bb83d0af01f9b502c5ea
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483479"
---
# <a name="application-roles"></a>Alkalmazás-szerepkörök

[![GitHub](../_images/github.png) Mintakód][sample application]

Alkalmazás-szerepkörök segítségével engedélyek hozzárendelése a felhasználókhoz. Ha például a [Tailspin Surveys] [ tailspin] alkalmazás határozza meg a következő szerepkörök:

* Rendszergazda. Bármely felmérés a bérlőhöz tartozó összes CRUD-műveletek hajthat végre.
* Létrehozó. Új felmérések hozhat létre.
* Olvasó. A bérlőhöz tartozó bármely felmérések olvashatja.

Láthatja, hogy szerepkörök végső soron első fordítja engedélyek során [engedélyezési]. De az első kérdés, hogy miképpen lehet hozzárendelni és felügyelni a szerepkört. Azt észleltük, hogy három fő beállítások:

* [Az Azure AD alkalmazás-szerepkörök](#roles-using-azure-ad-app-roles)
* [Az Azure AD biztonsági csoportok](#roles-using-azure-ad-security-groups)
* [Alkalmazás szerepkör-kezelő](#roles-using-an-application-role-manager).

## <a name="roles-using-azure-ad-app-roles"></a>Szerepkörök az Azure AD alkalmazás-szerepkörök használatával

Ez a megközelítés a Tailspin Surveys-alkalmazás is használt.

Ez a megközelítés az SaaS-szolgáltató határozza meg az alkalmazás-szerepkörök hozzáadását az alkalmazásjegyzékben. Miután egy ügyfél regisztrál, az ügyfél az Active Directory-rendszergazda rendeli hozzá a felhasználókat a szerepkörökhöz. Ha egy felhasználó bejelentkezik, a felhasználó hozzárendelt szerepkörök jogcímekként küld.

> [!NOTE]
> Ha az ügyfél rendelkezik prémium szintű Azure AD, a rendszergazda hozzárendelheti egy biztonsági csoportot egy szerepkörhöz, és a csoport tagjai öröklik az alkalmazás-szerepkör. Ennek oka egyszerűen kezelheti a szerepköröket, a csoport tulajdonosának nem kell lennie a rendszergazda AD.

Ez a módszer előnyei:

* Egyszerű programozási modellen alapuló.
* Szerepkörök kifejezetten az alkalmazáshoz. A szerepkör jogcím egy alkalmazás nem kap egy másik alkalmazás.
* Ha az ügyfél eltávolítja az alkalmazást a saját AD-bérlőjében, azonnal nyissa meg a szerepköröket.
* Az alkalmazás nem kell semmilyen további Active Directory-engedélyek nem a felhasználói profil olvasása.

Hátrányai:

* Prémium szintű Azure AD nem rendelkező ügyfelek biztonsági csoportok nem rendelhető szerepköröket. Ezen ügyfelek számára egy AD-rendszergazda felhasználó összes hozzárendelést kell elvégezni.
* Egy háttéralkalmazás webes API-t, amely a webalkalmazást ügynökellenőrzést, ha a webes alkalmazás szerepkör-hozzárendeléseit a webes API-k nem vonatkoznak. Ezen a ponton További ismertetéséhez lásd: [egy háttéralkalmazás webes API biztonságossá tétele].

### <a name="implementation"></a>Implementáció

**A szerepkörök meghatározása.** Az SaaS-szolgáltatónak az alkalmazás-szerepkörök deklarálja a [alkalmazásjegyzék]. Ha például itt látható a jegyzékfájl bejegyzést a Surveys alkalmazás:

```json
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

A `value` tulajdonság a szerepkör jogcím szerepel. A `id` tulajdonság a megadott szerepkör egyedi azonosítója. Mindig hozzon létre egy új GUID azonosítót `id`.

**Felhasználók hozzárendelése**. Amikor új vevő regisztrál, az alkalmazás regisztrálva van az ügyfél AD-bérlőben. Ezen a ponton egy AD rendszergazdai a bérlőhöz tartozó felhasználókat rendelhet szerepköröket.

> [!NOTE]
> Korábban feljegyzett ügyfelek az Azure AD Premium szolgáltatással is hozzárendelhetők biztonsági csoportok, szerepkörök.

Az alábbi képernyőképen az Azure Portalon látható felhasználók és csoportok a felmérés alkalmazás. Rendszergazdai és a szerző csoportjai, illetve SurveyAdmin és SurveyCreator szerepkörökhöz hozzárendelt. Alice az egy felhasználó, aki közvetlenül az SurveyAdmin szerepkör lett hozzárendelve. Bob és Charles nem szerepkörhöz közvetlenül hozzárendelt felhasználók közül ez.

![Felhasználók és csoportok](./images/running-the-app/users-and-groups.png)

Ahogy az az alábbi képernyőfelvételen is látható, Charles a felügyeleti csoport részét képezi, így ő örökli a SurveyAdmin szerepkör. Bob, esetén Phil van még nincs hozzárendelve szerepkör.

![Felügyeleti csoport tagjai](./images/running-the-app/admin-members.png)

> [!NOTE]
> Egy alternatív módszer az alkalmazás rendelhet szerepköröket programozott módon, az Azure AD Graph API használatával is. Azonban ez csak írási engedéllyel az ügyfél AD-címtár beszerzése az alkalmazás. Ezeket az engedélyeket az alkalmazás végrehajtja kárt rengeteg &mdash; az ügyfél nem az alkalmazás saját könyvtár elront delegálását. Előfordulhat, hogy számos ügyfelünk nincs ilyen hozzáférési szint megadását.

**Első szerepkörjogcímeken**. Amikor egy felhasználó bejelentkezik, az alkalmazás fogad a felhasználó hozzárendelt szerepkör egy jogcím típusú `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`.

Egy felhasználó több szerepkört, vagy nincs szerepkör rendelkezhetnek. Az engedélyezési kód nem érdemes feltételezni a felhasználó rendelkezik pontosan egy szerepkör jogcím. Ehelyett kód írása, amely ellenőrzi, hogy jelen-e egy adott jogcím értéke:

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a>Szerepkörök az Azure AD biztonsági csoportok használata

Ebben a megközelítésben AD biztonsági csoportok, szerepkörök jelennek meg. Az alkalmazás a felhasználók számára a biztonsági csoporttagságok alapján hozzárendeli az engedélyeket.

Előnyök:

* Azok a vásárlóknak, akik nem rendelkeznek Azure AD Premium szolgáltatással Ez a megközelítés lehetővé teszi a biztonsági csoportok használata a szerepkör-hozzárendelések kezeléséhez.

Hátrányait:

* Összetettségét. Minden bérlőhöz küld a jogcímeket másik csoport, mert az alkalmazás kell nyomon követheti, amely az egyes bérlők számára mely alkalmazás-szerepkörök, biztonsági csoportok felel meg.
* Ha az ügyfél eltávolítja az alkalmazást a saját AD-bérlőjében, a biztonsági csoportokat az AD-címtárban maradnak.

<!-- markdownlint-disable MD024 -->

### <a name="implementation"></a>Implementáció

<!-- markdownlint-enable MD024 -->

Az alkalmazásjegyzékben, állítsa be a `groupMembershipClaims` "SecurityGroup" tulajdonságot. Erre azért van szükség, csoport tagsági jogcímek beolvasni az aad-ből.

```json
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

Amikor új vevő regisztrál, az alkalmazás utasítja az ügyfél a szerepköröket, az alkalmazás számára szükséges biztonsági csoportok létrehozásához. Az ügyfél ezután meg kell adnia a csoportházirend-objektum azonosítóját az alkalmazás igényeinek megfelelően. Az alkalmazás tárolja ezeket egy táblában, amely leképezi a csoportazonosítók alkalmazás-szerepkörökhöz bérlőnként.

> [!NOTE]
> Azt is megteheti az alkalmazás sikerült létrehozni a csoportok programozott módon, az Azure AD Graph API használatával.  Ez akkor lehet kisebb a hibalehetőségeket rejt magában. Azonban az alkalmazás a "olvasása és írása az összes csoport" szükséges engedélyeket az ügyfél AD-címtárhoz. Előfordulhat, hogy számos ügyfelünk nincs ilyen hozzáférési szint megadását.

Amikor egy felhasználó jelentkezik be:

1. Az alkalmazás fogad a felhasználói csoportok jogcímekként. Minden egyes jogcím értéke a csoport Objektumazonosítóját.
2. Azure ad-ben a jogkivonatban elküldött csoportok számát korlátozza. Csoportok száma meghaladja ezt a korlátot, ha az Azure AD elküldi egy speciális "kereten túli" jogcímet. Ha a jogcím megtalálható, az alkalmazás az Azure AD Graph API-t, hogy a felhasználó csoportjának összes kell lekérdeznie. További információkért lásd: [az AD-csoportok használata a felhőalapú alkalmazások engedélyezési], a "Csoportok jogcím kerettúllépés" című szakaszában.
3. Az alkalmazás megkeresi az objektumazonosítók a saját adatbázisban található a megfelelő alkalmazás-szerepkörök hozzárendelése a felhasználóhoz.
4. Az alkalmazás egy egyéni jogcím értékét hozzáadja az egyszerű felhasználónév, amely kifejezi az alkalmazás-szerepkör. Például: `survey_role` = "SurveyAdmin".

Engedélyezési házirendeket kell használnia az egyéni szerepkör jogcím nem a csoport jogcím.

## <a name="roles-using-an-application-role-manager"></a>Egy alkalmazás szerepkörkezelővel szerepkörök

Ezt a módszert használja alkalmazás-szerepkörök nem tárolja az Azure ad-ben minden. Ehelyett az alkalmazás tárolja a szerepkör-hozzárendeléseket az egyes felhasználók a saját DB &mdash; használata esetén például a **RoleManager** ASP.NET identitás osztályában.

Előnyök:

* A szerepkörök és -hozzárendelések felhasználói teljes hozzáféréssel rendelkezik.

Hátrányai:

* Összetettebb, nehezebben karbantartása.
* AD biztonsági csoportok nem használható a szerepkör-hozzárendelések kezeléséhez.
* Az alkalmazás-adatbázis, honnan azt be nincs szinkronban a bérlő az Active directory, ahogy a felhasználók hozzáadásakor vagy eltávolításakor a felhasználói adatokat tartalmazza.

[**Tovább**][engedélyezési]

<!-- links -->

[tailspin]: tailspin.md
[Engedélyezési]: authorize.md
[Egy háttéralkalmazás webes API biztonságossá tétele]: web-api.md
[Alkalmazásjegyzék]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
