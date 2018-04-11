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
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="92905-103">A Dejójáték forgatókönyv</span><span class="sxs-lookup"><span data-stu-id="92905-103">The Tailspin scenario</span></span>

<span data-ttu-id="92905-104">[![GitHub](../_images/github.png) példakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="92905-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="92905-105">Dejójáték egy fiktív cég, amely által fejlesztett felmérések nevű SaaS-alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="92905-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="92905-106">Ez az alkalmazás lehetővé teszi a szervezetek létrehozása és közzététele online felmérések.</span><span class="sxs-lookup"><span data-stu-id="92905-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="92905-107">Egy szervezet iratkozzon fel az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="92905-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="92905-108">Miután regisztrált a szervezet, felhasználók bejelentkezhetnek a szervezeti hitelesítő adataikkal az alkalmazásba.</span><span class="sxs-lookup"><span data-stu-id="92905-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="92905-109">Felhasználók létrehozása, szerkesztése és felmérések közzététele.</span><span class="sxs-lookup"><span data-stu-id="92905-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="92905-110">Első lépésként az alkalmazással együtt, lásd: [futtassa a felmérések alkalmazást].</span><span class="sxs-lookup"><span data-stu-id="92905-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="92905-111">Felhasználók létrehozása, szerkesztése és felmérések megtekintése</span><span class="sxs-lookup"><span data-stu-id="92905-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="92905-112">Hitelesített felhasználók ő hozott létre, vagy még közreműködői jogokat minden felmérések megtekintheti, és hozzon létre új felmérések.</span><span class="sxs-lookup"><span data-stu-id="92905-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="92905-113">Figyelje meg, hogy a felhasználó van bejelentkezve, szervezeti identitására `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="92905-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Felmérések alkalmazás](./images/surveys-screenshot.png)

<span data-ttu-id="92905-115">Ezen a képernyőfelvételen látható a Szerkesztés felmérés lapját jeleníti meg:</span><span class="sxs-lookup"><span data-stu-id="92905-115">This screenshot shows the Edit Survey page:</span></span>

![Felmérés szerkesztése](./images/edit-survey.png)

<span data-ttu-id="92905-117">Felhasználók ugyanannak a bérlőnek belüli más felhasználók által létrehozott összes felmérések is megtekintheti.</span><span class="sxs-lookup"><span data-stu-id="92905-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Bérlői felmérések](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="92905-119">Felmérés tulajdonosok kérhetnek közreműködők</span><span class="sxs-lookup"><span data-stu-id="92905-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="92905-120">Amikor egy felhasználó létrehoz egy felmérés, hívhat mások számára is a felmérés lévő közreműködőkkel rendelkeznek.</span><span class="sxs-lookup"><span data-stu-id="92905-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="92905-121">A közreműködők a felmérés szerkesztése, de nem törölheti vagy tegye közzé.</span><span class="sxs-lookup"><span data-stu-id="92905-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![A közreműködői hozzáadása](./images/add-contributor.png)

<span data-ttu-id="92905-123">Egy felhasználó adhat hozzá közreműködők a többi bérlőtől, amely lehetővé teszi az erőforrás-megosztás kereszt-bérlő.</span><span class="sxs-lookup"><span data-stu-id="92905-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="92905-124">Ezen a képernyőfelvételen látható Bálint (`bob@contoso.com`) van fel Alice felhasználóját (`alice@fabrikam.com`) egy Bob által létrehozott felmérésre közreműködőként.</span><span class="sxs-lookup"><span data-stu-id="92905-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="92905-125">Alice jelentkezik be, ha azt látja, a "Felmérések is hozzájáruljanak" alatt szereplő felmérés.</span><span class="sxs-lookup"><span data-stu-id="92905-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Felmérés közreműködő](./images/contributor.png)

<span data-ttu-id="92905-127">Figyelje meg, hogy Alice saját bérlőt, nem a Contoso bérlő vendégként bejelentkezik.</span><span class="sxs-lookup"><span data-stu-id="92905-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="92905-128">Alice csak adott felmérés közreműködői engedélyekkel rendelkezik &mdash; ő nem lehet megtekinteni, más felmérésekkel, a Contoso-bérlőhöz.</span><span class="sxs-lookup"><span data-stu-id="92905-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="92905-129">Architektúra</span><span class="sxs-lookup"><span data-stu-id="92905-129">Architecture</span></span>
<span data-ttu-id="92905-130">A felmérések alkalmazás egy előtér-webkiszolgáló és a webes API háttéralkalmazás áll.</span><span class="sxs-lookup"><span data-stu-id="92905-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="92905-131">Mindkettő használatával megvalósított [ASP.NET Core].</span><span class="sxs-lookup"><span data-stu-id="92905-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="92905-132">A webalkalmazás az Azure Active Directory (Azure AD) a felhasználók hitelesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="92905-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="92905-133">A webes alkalmazás is meghívja az Azure AD a webes API-t OAuth 2 hozzáférési jogkivonatok lekérésére.</span><span class="sxs-lookup"><span data-stu-id="92905-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="92905-134">Az Azure Redis Cache hozzáférési jogkivonatok lettek gyorsítótárazva.</span><span class="sxs-lookup"><span data-stu-id="92905-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="92905-135">A gyorsítótár lehetővé teszi, hogy több példány megosztása (pl. a kiszolgálófarm) azonos jogkivonatok gyorsítótárát.</span><span class="sxs-lookup"><span data-stu-id="92905-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Architektúra](./images/architecture.png)

<span data-ttu-id="92905-137">[**Következő**][authentication]</span><span class="sxs-lookup"><span data-stu-id="92905-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[futtassa a felmérések alkalmazást]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
