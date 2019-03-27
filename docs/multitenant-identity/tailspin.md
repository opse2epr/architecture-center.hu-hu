---
title: A Tailspin Surveys-alkalmazásról
description: A Tailspin Surveys alkalmazás áttekintése.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de2830b5c492e027c189a79e45ccc6634cab436b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299357"
---
# <a name="the-tailspin-scenario"></a>A Tailspin forgatókönyv

[![GitHub](../_images/github.png) Mintakód][sample application]

Tailspin Surveys nevű SaaS-alkalmazás által fejlesztett, kitalált vállalat. Ez az alkalmazás lehetővé teszi, hogy a szervezetek számára, hogy létrehozása és közzététele online felméréseket.

* Egy szervezet regisztrálhat az alkalmazást.
* A szervezethez regisztrált, miután a felhasználók az alkalmazásba a szervezeti hitelesítő adataikkal jelentkezhetnek.
* Felhasználók létrehozása, szerkesztése és felmérések közzététele.

> [!NOTE]
> Az alkalmazás használatának megkezdéséhez lásd [a Surveys alkalmazás futtatása].

## <a name="users-can-create-edit-and-view-surveys"></a>Felhasználók létrehozása, szerkesztése és felmérések megtekintése

Egy hitelesített felhasználó megtekintheti a felmérések, ő hozott létre, vagy közreműködői jogosultságokkal rendelkezik, és hozzon létre új felmérések. Figyelje meg, hogy a felhasználó szervezeti identitására van bejelentkezve `bob@contoso.com`.

![Felmérésekben app](./images/surveys-screenshot.png)

Ezen a képernyőfelvételen a felmérés szerkesztése lapját jeleníti meg:

![Felmérés szerkesztése](./images/edit-survey.png)

Felhasználók is megtekintheti az ugyanazon bérlőn belüli más felhasználók által létrehozott bármely felmérések.

![Bérlő felmérések](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a>Felmérés tulajdonosok közreműködők küldhetnek meghívót

Amikor egy felhasználó létrehoz egy kérdőív, meghívhatja mások számára is a felmérést közreműködőkkel. A közreműködők a felmérést, szerkesztése, de nem törölheti vagy tegye közzé.

![Adja hozzá a közreműködő](./images/add-contributor.png)

A felhasználó hozzáadhatja a közreműködők más bérlőktől származó ami lehetővé teszi a több-bérlős erőforrás-megosztás. Ezen a képernyőképen látható, Bob (`bob@contoso.com`) van Alice hozzáadása (`alice@fabrikam.com`) egy Bob által létrehozott felmérésre közreműködője.

Alice bejelentkezik, hitelkártyaadatokat "Felmérések is hozzájárul" részen a felmérést.

![Felmérés közreműködője](./images/contributor.png)

Vegye figyelembe, hogy Ágnes bejelentkezik a saját bérlőjében, nem a Contoso bérlő vendégként. Alice csak a felmérés közreműködői engedélyekkel rendelkezik &mdash; nem látja a Contoso bérlőről más felmérések.

## <a name="architecture"></a>Architektúra

A Surveys alkalmazás webes kezelőfelületből és a webes API-háttérrendszer áll. Mindkettő használatával megvalósított [ASP.NET Core].

A webalkalmazás az Azure Active Directory (Azure AD) használatával hitelesítheti a felhasználókat. A webes alkalmazás is meghívja az Azure AD OAuth 2 hozzáférési jogkivonatok beolvasásához a webes API-hoz. Hozzáférési jogkivonatok az Azure Redis Cache-ben lettek gyorsítótárazva. A gyorsítótár lehetővé teszi több példány megosztása (például a kiszolgálófarmban) azonos tokengyorsítótárral.

![Architektúra](./images/architecture.png)

[**Tovább**][authentication]

<!-- links -->

[authentication]: authenticate.md

[A Surveys alkalmazás futtatása]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
