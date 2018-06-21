---
title: A Dejójáték felmérések alkalmazással kapcsolatos
description: Dejójáték felmérések – áttekintés
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540057"
---
# <a name="the-tailspin-scenario"></a>A Dejójáték forgatókönyv

[![GitHub](../_images/github.png) példakód][sample application]

Dejójáték egy fiktív cég, amely által fejlesztett felmérések nevű SaaS-alkalmazáshoz. Ez az alkalmazás lehetővé teszi a szervezetek létrehozása és közzététele online felmérések.

* Egy szervezet iratkozzon fel az alkalmazáshoz.
* Miután regisztrált a szervezet, felhasználók bejelentkezhetnek a szervezeti hitelesítő adataikkal az alkalmazásba.
* Felhasználók létrehozása, szerkesztése és felmérések közzététele.

> [!NOTE]
> Első lépésként az alkalmazással együtt, lásd: [futtassa a felmérések alkalmazást].
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a>Felhasználók létrehozása, szerkesztése és felmérések megtekintése
Hitelesített felhasználók ő hozott létre, vagy még közreműködői jogokat minden felmérések megtekintheti, és hozzon létre új felmérések. Figyelje meg, hogy a felhasználó van bejelentkezve, szervezeti identitására `bob@contoso.com`.

![Felmérések alkalmazás](./images/surveys-screenshot.png)

Ezen a képernyőfelvételen látható a Szerkesztés felmérés lapját jeleníti meg:

![Felmérés szerkesztése](./images/edit-survey.png)

Felhasználók ugyanannak a bérlőnek belüli más felhasználók által létrehozott összes felmérések is megtekintheti.

![Bérlői felmérések](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a>Felmérés tulajdonosok kérhetnek közreműködők
Amikor egy felhasználó létrehoz egy felmérés, hívhat mások számára is a felmérés lévő közreműködőkkel rendelkeznek. A közreműködők a felmérés szerkesztése, de nem törölheti vagy tegye közzé.  

![A közreműködői hozzáadása](./images/add-contributor.png)

Egy felhasználó adhat hozzá közreműködők a többi bérlőtől, amely lehetővé teszi az erőforrás-megosztás kereszt-bérlő. Ezen a képernyőfelvételen látható Bálint (`bob@contoso.com`) van fel Alice felhasználóját (`alice@fabrikam.com`) egy Bob által létrehozott felmérésre közreműködőként.

Alice jelentkezik be, ha azt látja, a "Felmérések is hozzájáruljanak" alatt szereplő felmérés.

![Felmérés közreműködő](./images/contributor.png)

Figyelje meg, hogy Alice saját bérlőt, nem a Contoso bérlő vendégként bejelentkezik. Alice csak adott felmérés közreműködői engedélyekkel rendelkezik &mdash; ő nem lehet megtekinteni, más felmérésekkel, a Contoso-bérlőhöz.

## <a name="architecture"></a>Architektúra
A felmérések alkalmazás egy előtér-webkiszolgáló és a webes API háttéralkalmazás áll. Mindkettő használatával megvalósított [ASP.NET Core].

A webalkalmazás az Azure Active Directory (Azure AD) a felhasználók hitelesítéséhez. A webes alkalmazás is meghívja az Azure AD a webes API-t OAuth 2 hozzáférési jogkivonatok lekérésére. Az Azure Redis Cache hozzáférési jogkivonatok lettek gyorsítótárazva. A gyorsítótár lehetővé teszi, hogy több példány megosztása (pl. a kiszolgálófarm) azonos jogkivonatok gyorsítótárát.

![Architektúra](./images/architecture.png)

[**Következő**][authentication]

<!-- Links -->

[authentication]: authenticate.md

[futtassa a felmérések alkalmazást]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
