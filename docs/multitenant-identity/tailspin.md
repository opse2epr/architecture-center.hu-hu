---
title: A Tailspin Surveys-alkalmazásról
description: A Tailspin Surveys alkalmazás áttekintése.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de2830b5c492e027c189a79e45ccc6634cab436b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483224"
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="af316-103">A Tailspin forgatókönyv</span><span class="sxs-lookup"><span data-stu-id="af316-103">The Tailspin scenario</span></span>

<span data-ttu-id="af316-104">[![GitHub](../_images/github.png) Mintakód][sample application]</span><span class="sxs-lookup"><span data-stu-id="af316-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="af316-105">Tailspin Surveys nevű SaaS-alkalmazás által fejlesztett, kitalált vállalat.</span><span class="sxs-lookup"><span data-stu-id="af316-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="af316-106">Ez az alkalmazás lehetővé teszi, hogy a szervezetek számára, hogy létrehozása és közzététele online felméréseket.</span><span class="sxs-lookup"><span data-stu-id="af316-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="af316-107">Egy szervezet regisztrálhat az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="af316-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="af316-108">A szervezethez regisztrált, miután a felhasználók az alkalmazásba a szervezeti hitelesítő adataikkal jelentkezhetnek.</span><span class="sxs-lookup"><span data-stu-id="af316-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="af316-109">Felhasználók létrehozása, szerkesztése és felmérések közzététele.</span><span class="sxs-lookup"><span data-stu-id="af316-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="af316-110">Az alkalmazás használatának megkezdéséhez lásd [a Surveys alkalmazás futtatása].</span><span class="sxs-lookup"><span data-stu-id="af316-110">To get started with the application, see [Run the Surveys application].</span></span>

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="af316-111">Felhasználók létrehozása, szerkesztése és felmérések megtekintése</span><span class="sxs-lookup"><span data-stu-id="af316-111">Users can create, edit, and view surveys</span></span>

<span data-ttu-id="af316-112">Egy hitelesített felhasználó megtekintheti a felmérések, ő hozott létre, vagy közreműködői jogosultságokkal rendelkezik, és hozzon létre új felmérések.</span><span class="sxs-lookup"><span data-stu-id="af316-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="af316-113">Figyelje meg, hogy a felhasználó szervezeti identitására van bejelentkezve `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="af316-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Felmérésekben app](./images/surveys-screenshot.png)

<span data-ttu-id="af316-115">Ezen a képernyőfelvételen a felmérés szerkesztése lapját jeleníti meg:</span><span class="sxs-lookup"><span data-stu-id="af316-115">This screenshot shows the Edit Survey page:</span></span>

![Felmérés szerkesztése](./images/edit-survey.png)

<span data-ttu-id="af316-117">Felhasználók is megtekintheti az ugyanazon bérlőn belüli más felhasználók által létrehozott bármely felmérések.</span><span class="sxs-lookup"><span data-stu-id="af316-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Bérlő felmérések](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="af316-119">Felmérés tulajdonosok közreműködők küldhetnek meghívót</span><span class="sxs-lookup"><span data-stu-id="af316-119">Survey owners can invite contributors</span></span>

<span data-ttu-id="af316-120">Amikor egy felhasználó létrehoz egy kérdőív, meghívhatja mások számára is a felmérést közreműködőkkel.</span><span class="sxs-lookup"><span data-stu-id="af316-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="af316-121">A közreműködők a felmérést, szerkesztése, de nem törölheti vagy tegye közzé.</span><span class="sxs-lookup"><span data-stu-id="af316-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>

![Adja hozzá a közreműködő](./images/add-contributor.png)

<span data-ttu-id="af316-123">A felhasználó hozzáadhatja a közreműködők más bérlőktől származó ami lehetővé teszi a több-bérlős erőforrás-megosztás.</span><span class="sxs-lookup"><span data-stu-id="af316-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="af316-124">Ezen a képernyőképen látható, Bob (`bob@contoso.com`) van Alice hozzáadása (`alice@fabrikam.com`) egy Bob által létrehozott felmérésre közreműködője.</span><span class="sxs-lookup"><span data-stu-id="af316-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="af316-125">Alice bejelentkezik, hitelkártyaadatokat "Felmérések is hozzájárul" részen a felmérést.</span><span class="sxs-lookup"><span data-stu-id="af316-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Felmérés közreműködője](./images/contributor.png)

<span data-ttu-id="af316-127">Vegye figyelembe, hogy Ágnes bejelentkezik a saját bérlőjében, nem a Contoso bérlő vendégként.</span><span class="sxs-lookup"><span data-stu-id="af316-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="af316-128">Alice csak a felmérés közreműködői engedélyekkel rendelkezik &mdash; nem látja a Contoso bérlőről más felmérések.</span><span class="sxs-lookup"><span data-stu-id="af316-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="af316-129">Architektúra</span><span class="sxs-lookup"><span data-stu-id="af316-129">Architecture</span></span>

<span data-ttu-id="af316-130">A Surveys alkalmazás webes kezelőfelületből és a webes API-háttérrendszer áll.</span><span class="sxs-lookup"><span data-stu-id="af316-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="af316-131">Mindkettő használatával megvalósított [ASP.NET Core].</span><span class="sxs-lookup"><span data-stu-id="af316-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="af316-132">A webalkalmazás az Azure Active Directory (Azure AD) használatával hitelesítheti a felhasználókat.</span><span class="sxs-lookup"><span data-stu-id="af316-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="af316-133">A webes alkalmazás is meghívja az Azure AD OAuth 2 hozzáférési jogkivonatok beolvasásához a webes API-hoz.</span><span class="sxs-lookup"><span data-stu-id="af316-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="af316-134">Hozzáférési jogkivonatok az Azure Redis Cache-ben lettek gyorsítótárazva.</span><span class="sxs-lookup"><span data-stu-id="af316-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="af316-135">A gyorsítótár lehetővé teszi több példány megosztása (például a kiszolgálófarmban) azonos tokengyorsítótárral.</span><span class="sxs-lookup"><span data-stu-id="af316-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Architektúra](./images/architecture.png)

<span data-ttu-id="af316-137">[**Tovább**][authentication]</span><span class="sxs-lookup"><span data-stu-id="af316-137">[**Next**][authentication]</span></span>

<!-- links -->

[authentication]: authenticate.md

[A Surveys alkalmazás futtatása]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
