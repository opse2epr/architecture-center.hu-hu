---
title: 'CAF: Mi az a felhőerőforrás-szabályozás?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: MAGYARÁZAT felhőalapú erőforrás-szabályozása az Azure-ban
author: petertaylor9999
ms.openlocfilehash: 0989a5aad8a6290cce07fd71771c690bbd615e0d
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068854"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a><span data-ttu-id="3f031-103">Mi az a felhőerőforrás-szabályozás?</span><span class="sxs-lookup"><span data-stu-id="3f031-103">What is cloud resource governance?</span></span>

<span data-ttu-id="3f031-104">A [Azure működése?](what-is-azure.md), hogy megtanulta, hogy az Azure a kiszolgálókon és hálózati hardvereivel futtató virtualizált hardver- és a felhasználók nevében.</span><span class="sxs-lookup"><span data-stu-id="3f031-104">In [How does Azure work?](what-is-azure.md), you learned that Azure is a collection of servers and networking hardware running virtualized hardware and software on behalf of users.</span></span> <span data-ttu-id="3f031-105">Az Azure lehetővé teszi, hogy a munkahelyi alkalmazások fejlesztését és IT-részlegek számára egyszerűvé létrehozása, olvasása, frissítése és törlése az erőforrásokat, igény szerint, agilisek lehetnek.</span><span class="sxs-lookup"><span data-stu-id="3f031-105">Azure enables your organization's application development and IT departments to be agile by making it easy to create, read, update, and delete resources as needed.</span></span>

<span data-ttu-id="3f031-106">Azonban amíg erőforrásokhoz való korlátlan hozzáférés is, hogy a fejlesztők nagyon hatékony, is vezethet, váratlan költségek.</span><span class="sxs-lookup"><span data-stu-id="3f031-106">However, while unrestricted access to resources can make developers very agile, it can also lead to unexpected costs.</span></span> <span data-ttu-id="3f031-107">Ha például egy fejlesztői csapat előfordulhat, hogy jóvá kell hagyni tesztelési erőforrások üzembe helyezésének, de felejtse el törölni őket, ha a tesztelés.</span><span class="sxs-lookup"><span data-stu-id="3f031-107">For example, a development team might be approved to deploy a set of resources for testing but forget to delete them when testing is complete.</span></span> <span data-ttu-id="3f031-108">Ezek az erőforrások továbbra is lépheti túl a költségeket, annak ellenére, hogy azok nem lesznek jóváhagyott vagy szükséges.</span><span class="sxs-lookup"><span data-stu-id="3f031-108">These resources will continue to accrue costs even though they are no longer approved or necessary.</span></span>

<span data-ttu-id="3f031-109">A megoldás az erőforrás-hozzáférés szabályozása.</span><span class="sxs-lookup"><span data-stu-id="3f031-109">The solution is resource access governance.</span></span> <span data-ttu-id="3f031-110">**Cégirányítási** kezelését, megfigyelését és az Azure-erőforrások megfelelnek a szervezet követelményeinek használatának naplózása folyamatban van.</span><span class="sxs-lookup"><span data-stu-id="3f031-110">**Governance** is the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the requirements of your organization.</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="3f031-111">Ezek a követelmények az egyes cégeknek, egyedi, így a cégirányítási időkorlátokat nem hasznos.</span><span class="sxs-lookup"><span data-stu-id="3f031-111">These requirements are unique to each organization, so a one-size-fits-all approach to governance isn't helpful.</span></span> <span data-ttu-id="3f031-112">Ehelyett szolgáltatás minden szervezet számára, azok az Azure két elsődleges cégirányítási eszközökkel cégirányítási modell tervezéséhez: **szerepköralapú hozzáférés-vezérlés (RBAC)** és **erőforrás-szabályzat**.</span><span class="sxs-lookup"><span data-stu-id="3f031-112">Instead, it's up to each organization to design their governance model using Azure's two primary governance tools: **role-based access control (RBAC)** and **resource policy**.</span></span>

<span data-ttu-id="3f031-113">Az RBAC szerepkörök határozza meg, és a szerepkörök határozzák meg az egyes adott szerepkörhöz rendelt felhasználók képességeit.</span><span class="sxs-lookup"><span data-stu-id="3f031-113">RBAC defines roles, and roles define the capabilities of each user assigned that role.</span></span> <span data-ttu-id="3f031-114">Például a **tulajdonosa** szerepkör lehetővé teszi, hogy az összes képességek (létrehozása, olvasása, frissítése és törlése) egy erőforráshoz, amíg a **olvasó** szerepkör lehetővé teszi, hogy a csak olvasási képességet.</span><span class="sxs-lookup"><span data-stu-id="3f031-114">For example, the **owner** role allows all capabilites (create, read, update, and delete) for a resource, while the  **reader** role allows only the read capability.</span></span> <span data-ttu-id="3f031-115">Szerepkörök egy széles körét, amely számos erőforrástípusok vonatkozik, vagy vonatkozik néhány keskeny hatóköre lehet definiálni.</span><span class="sxs-lookup"><span data-stu-id="3f031-115">Roles can be defined with a broad scope that applies to many resource types, or a narrow scope that applies to a few.</span></span>

<span data-ttu-id="3f031-116">Erőforrás-házirendek erőforrás-létrehozás vonatkozó szabályok meghatározásához.</span><span class="sxs-lookup"><span data-stu-id="3f031-116">Resource policies define rules for resource creation.</span></span> <span data-ttu-id="3f031-117">Például egy erőforrás-szabályzat korlátozhatja az egy adott előre jóváhagyott méretű virtuális gép Termékváltozata.</span><span class="sxs-lookup"><span data-stu-id="3f031-117">For example, a resource policy can limit the SKU of a virtual machine to a particular pre-approved size.</span></span> <span data-ttu-id="3f031-118">Egy másik erőforrás-szabályzat egy hozzárendelt költséghely a címke alkalmazása sikerült kényszerítése, az erőforrás létrehozásához a kérelem elküldésekor.</span><span class="sxs-lookup"><span data-stu-id="3f031-118">Another resource policy could enforce the application of a tag for an assigned cost center when the request is made to create the resource.</span></span>

<span data-ttu-id="3f031-119">Ezek az eszközök konfigurálásakor fontos irányítási és szervezeti rugalmasságot.</span><span class="sxs-lookup"><span data-stu-id="3f031-119">When configuring these tools, it is important to balance governance and organizational agility.</span></span> <span data-ttu-id="3f031-120">A szigorúbb cégirányítási házirendjében a kevésbé rugalmas a fejlesztők és informatikai dolgozók lesz.</span><span class="sxs-lookup"><span data-stu-id="3f031-120">The more restrictive your governance policy, the less agile your developers and IT workers will be.</span></span> <span data-ttu-id="3f031-121">A cégirányítási korlátozó házirend további manuális lépések, például a fejlesztő töltsön ki egy űrlapot vagy e-mail üzenetet küldhet a cégirányítási csapat tagjai manuálisan hozzon létre egy erőforrást igénylő lehet szükség.</span><span class="sxs-lookup"><span data-stu-id="3f031-121">A restrictive governance policy may require more manual steps like requiring a developer to fill out a form or send an email to a member of the governance team to manually create a resource.</span></span> <span data-ttu-id="3f031-122">A cégirányítási csapata kapacitása véges, és előfordulhat, hogy megfelelően vannak megtervezve, eredményez a fejlesztői csapatok unproductively Várakozás keletkezhetnek költségek, törlés előtt létrehozott és a felesleges erőforrások az erőforrásaikat.</span><span class="sxs-lookup"><span data-stu-id="3f031-122">The governance team has finite capacity and may become a bottleneck, resulting in development teams waiting unproductively for their resources to be created or unneeded resources accruing costs before they are deleted.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3f031-123">További lépések</span><span class="sxs-lookup"><span data-stu-id="3f031-123">Next steps</span></span>

<span data-ttu-id="3f031-124">Most, hogy megismerkedett a felhőbeli erőforrás-szabályozás fogalmát, tudjon meg többet az erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3f031-124">Now that you understand the concept of cloud resource governance, learn more about how resource access is managed in Azure.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="3f031-125">További tudnivalók az Azure-beli erőforrás-hozzáférés</span><span class="sxs-lookup"><span data-stu-id="3f031-125">Learn about resource access in Azure</span></span>](azure-resource-access.md)
