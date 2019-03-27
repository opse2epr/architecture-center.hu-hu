---
title: 'CAF: Erőforráshozzáférés-kezelés az Azure-ban'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: 'Erőforráshozzáférés-kezelés magyarázata hoz létre az Azure-ban: Az Azure Resource Manager, az előfizetések, erőforráscsoportok és erőforrások'
author: petertaylor9999
ms.openlocfilehash: b98cdc94d6d3a37c1e65da1d4de35d5d9520d6eb
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298761"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="be110-103">Erőforráshozzáférés-kezelés az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="be110-103">Resource access management in Azure</span></span>

<span data-ttu-id="be110-104">[A felhő Cégirányítási](../overview.md) felhőalapú irányítási, amely tartalmazza az erőforrás-kezelés öt állapítják ismerteti.</span><span class="sxs-lookup"><span data-stu-id="be110-104">[Cloud Governance](../overview.md) outlines the five disciplines of Cloud Governance, which includes Resource Management.</span></span>  <span data-ttu-id="be110-105">[Mi az erőforrás-hozzáférés szabályozása](overview.md) furthers azt ismerteti, hogyan illeszkedik a erőforráshozzáférés-kezelés az erőforrás-felügyeleti területen.</span><span class="sxs-lookup"><span data-stu-id="be110-105">[What is resource access governance](overview.md) furthers explains how resource access management fits into the resource management discipline.</span></span> <span data-ttu-id="be110-106">Áthelyezni a megtudhatja, hogyan kell irányítási mintájának, mielőtt fontos tudni, hogy az erőforrás-hozzáférési felügyeleti vezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="be110-106">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="be110-107">Erőforrás-hozzáférési felügyeleti vezérlőelemek konfigurációját a cégirányítási modell alapját.</span><span class="sxs-lookup"><span data-stu-id="be110-107">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="be110-108">Első lépésként forgalomnövekedésre közelebb hogyan erőforrások vannak üzembe helyezve, az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="be110-108">Begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="be110-109">Mi az Azure-erőforrásban?</span><span class="sxs-lookup"><span data-stu-id="be110-109">What is an Azure resource?</span></span>

<span data-ttu-id="be110-110">Az Azure-ban az előfizetési időszak **erőforrás** egy Azure által kezelt entitásra hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="be110-110">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="be110-111">Ha például virtuális gépek, virtuális hálózatok és tárfiókok az összes nevezzük Azure-erőforrások.</span><span class="sxs-lookup"><span data-stu-id="be110-111">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="be110-112">![Egy erőforrás ábrája](../../_images/governance-1-9.png)
*1. ábra. Erőforrás.*</span><span class="sxs-lookup"><span data-stu-id="be110-112">![Diagram of a resource](../../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="be110-113">Mi az Azure-erőforráscsoport?</span><span class="sxs-lookup"><span data-stu-id="be110-113">What is an Azure resource group?</span></span>

<span data-ttu-id="be110-114">Az Azure-ban minden egyes erőforráscsoportba kell tartoznia a [erőforráscsoport](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="be110-114">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="be110-115">Egy erőforráscsoport egyszerűen egy logikai szerkezet, amely több erőforrást együttesen csoportokat, hogy egyetlen egységként kezelhetők.</span><span class="sxs-lookup"><span data-stu-id="be110-115">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="be110-116">Például egy hasonló az életciklusuk, például az erőforrások erőforrásokat egy [n szintű alkalmazás](/azure/architecture/guide/architecture-styles/n-tier) létrehozott vagy törölt csoportként.</span><span class="sxs-lookup"><span data-stu-id="be110-116">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="be110-117">![A diagram egy erőforrást tartalmazó erőforráscsoport](../../_images/governance-1-10.png)
*2. ábra. Egy erőforráscsoport erőforrást tartalmaz.*</span><span class="sxs-lookup"><span data-stu-id="be110-117">![Diagram of a resource group containing a resource](../../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="be110-118">Erőforráscsoportok és az erőforrások kapcsolódnak az Azure-beli **előfizetés**.</span><span class="sxs-lookup"><span data-stu-id="be110-118">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="be110-119">Mi az Azure-előfizetéssel?</span><span class="sxs-lookup"><span data-stu-id="be110-119">What is an Azure subscription?</span></span>

<span data-ttu-id="be110-120">Azure-előfizetés nagyon hasonlít egy erőforráscsoportot, abban, hogy egy logikai szerkezet, amely csoportosítja együtt erőforráscsoportokról és azok erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="be110-120">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="be110-121">Azonban egy Azure-előfizetést is társítva az Azure Resource Manager által használt vezérlőkkel.</span><span class="sxs-lookup"><span data-stu-id="be110-121">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="be110-122">Ez mit jelent?</span><span class="sxs-lookup"><span data-stu-id="be110-122">What does this mean?</span></span> <span data-ttu-id="be110-123">Jelenleg Azure Resource Manager és az Azure-előfizetések közötti kapcsolatot kapcsolatos közelebbről is.</span><span class="sxs-lookup"><span data-stu-id="be110-123">Take a closer look at Azure Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="be110-124">![Azure-előfizetés diagram](../../_images/governance-1-11.png)
*3. ábra. Azure-előfizetés.*</span><span class="sxs-lookup"><span data-stu-id="be110-124">![Diagram of an Azure subscription](../../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="be110-125">Mi az Azure Resource Manager?</span><span class="sxs-lookup"><span data-stu-id="be110-125">What is Azure Resource Manager?</span></span>

<span data-ttu-id="be110-126">A [Azure működése?](../../getting-started/what-is-azure.md) megtanulta, hogy az Azure számos olyan szolgáltatás, amely az Azure minden funkcióját összehangolására az "előtér" tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="be110-126">In [how does Azure work?](../../getting-started/what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="be110-127">Ezek a szolgáltatások egyik [Azure Resource Manager](/azure/azure-resource-manager/), és ez a szolgáltatás üzemelteti a RESTful API-ügyfelek által használt erőforrások kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="be110-127">One of these services is [Azure Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="be110-128">![Az Azure Resource Manager-diagram](../../_images/governance-1-12.png)
*4. ábra. Az Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="be110-128">![Diagram of Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*</span></span>

<span data-ttu-id="be110-129">A következő ábrán látható három ügyfelek: [PowerShell](/powershell/azure/overview), a [az Azure portal](https://portal.azure.com), és a [az Azure parancssori felület (CLI)](/cli/azure):</span><span class="sxs-lookup"><span data-stu-id="be110-129">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command-line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="be110-130">![Az Azure-ügyfelek az Azure Resource Manager API-t kapcsolódás diagram](../../_images/governance-1-13.png)
*5. ábra. Az Azure-ügyfelek csatlakoznak az Azure Resource Manager REST API.*</span><span class="sxs-lookup"><span data-stu-id="be110-130">![Diagram of Azure clients connecting to the Azure Resource Manager API](../../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Azure Resource Manager RESTful API.*</span></span>

<span data-ttu-id="be110-131">Bár ezek az ügyfelek csatlakozni az Azure Resource Manager a RESTful API-val, az Azure Resource Manager nem tartalmaz közvetlenül kezelheti az erőforrásokat funkciót.</span><span class="sxs-lookup"><span data-stu-id="be110-131">While these clients connect to Azure Resource Manager using the RESTful API, Azure Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="be110-132">Ehelyett a legtöbb erőforrástípusok az Azure-ban megvan a saját [ **erőforrás-szolgáltató**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="be110-132">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="be110-133">![Az Azure erőforrás-szolgáltatók](../../_images/governance-1-14.png)
*6. ábra. Az Azure erőforrás-szolgáltatók.*</span><span class="sxs-lookup"><span data-stu-id="be110-133">![Azure resource providers](../../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="be110-134">Amikor egy ügyfél kérést küld egy adott erőforrás kezeléséhez, Azure Resource Manager kapcsolódik az erőforrás-szolgáltató az erőforrás típusát, a kérés teljesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="be110-134">When a client makes a request to manage a specific resource, Azure Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="be110-135">Például ha egy ügyfél kérést küld egy virtuális gép típusú erőforrást kezelése, Azure Resource Manager csatlakozik a **Microsoft.Compute** erőforrás-szolgáltató.</span><span class="sxs-lookup"><span data-stu-id="be110-135">For example, if a client makes a request to manage a virtual machine resource, Azure Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="be110-136">![A Microsoft.Compute erőforrás-szolgáltatóhoz való csatlakozás az Azure Resource Manager](../../_images/governance-1-15.png)
*7. ábra. Az Azure Resource Manager csatlakozik a **Microsoft.Compute** kezeléséhez az ügyfél kérésében megadott erőforrás erőforrás-szolgáltató.*</span><span class="sxs-lookup"><span data-stu-id="be110-136">![Azure Resource Manager connecting to the Microsoft.Compute resource provider](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="be110-137">Az Azure Resource Manager megköveteli, hogy adja meg az előfizetés és az erőforráscsoport azonosítóját ahhoz, hogy a virtuális gép típusú erőforrást kezelése az ügyfél.</span><span class="sxs-lookup"><span data-stu-id="be110-137">Azure Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="be110-138">Most, hogy az Azure Resource Manager működésének megismerése, térjen vissza a téma bemutatásához, hogyan Azure-előfizetéssel társítva az Azure Resource Manager által használt vezérlőkkel.</span><span class="sxs-lookup"><span data-stu-id="be110-138">Now that you have an understanding of how Azure Resource Manager works, return to the discussion of how an Azure subscription is associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="be110-139">Mielőtt bármely erőforrás felügyeleti kérelem Azure Resource Manager által hajtható végre, a vezérlők be van jelölve.</span><span class="sxs-lookup"><span data-stu-id="be110-139">Before any resource management request can be executed by Azure Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="be110-140">Az első vezérlőelem, hogy a kérelmet kell kezdeményeznie, ha igazolt felhasználója, és a egy megbízható kapcsolattal rendelkezik az Azure Resource Manager [Azure Active Directory (Azure AD)](/azure/active-directory/) felhasználói identitáskezelési funkciókat biztosít.</span><span class="sxs-lookup"><span data-stu-id="be110-140">The first control is that a request must be made by a validated user, and Azure Resource Manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

<span data-ttu-id="be110-141">![Az Azure Active Directory](../../_images/governance-1-16.png)
*8. ábra. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="be110-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="be110-142">Azure ad-felhasználók szegmentált be **bérlők**.</span><span class="sxs-lookup"><span data-stu-id="be110-142">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="be110-143">Egy bérlő egy logikai szerkezet, amely általában egy szervezet társított Azure AD egy biztonságos, dedikált példányát jelöli.</span><span class="sxs-lookup"><span data-stu-id="be110-143">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="be110-144">Minden egyes előfizetés társítva az Azure AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="be110-144">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="be110-145">![Az egy előfizetéshez tartozó Azure AD-bérlővel](../../_images/governance-1-17.png)
*9. ábra. Az egy előfizetéshez tartozó Azure AD-bérlővel.*</span><span class="sxs-lookup"><span data-stu-id="be110-145">![An Azure AD tenant associated with a subscription](../../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="be110-146">Minden ügyfélkérés kezelésére egy erőforrást egy adott előfizetés szükséges, hogy a felhasználó rendelkezik egy fiókot a társított Azure AD-bérlővel.</span><span class="sxs-lookup"><span data-stu-id="be110-146">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="be110-147">A következő vezérlő, hogy a felhasználó rendelkezik-e megfelelő engedélye a kérés ellenőrzése.</span><span class="sxs-lookup"><span data-stu-id="be110-147">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="be110-148">Engedélyek használatával rendelkező felhasználókhoz rendelt [szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="be110-148">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="be110-149">![RBAC-szerepkörökhöz rendelt felhasználók](../../_images/governance-1-18.png)
*10. ábra. A bérlő minden felhasználója egy vagy több RBAC szerepkör van hozzárendelve.*</span><span class="sxs-lookup"><span data-stu-id="be110-149">![Users assigned to RBAC roles](../../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="be110-150">Az RBAC szerepkör engedélyekkel a felhasználó egy adott erőforráson eltarthat egy halmazát határozza meg.</span><span class="sxs-lookup"><span data-stu-id="be110-150">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="be110-151">A szerepkör van rendelve a felhasználót, amikor a rendszer alkalmazza ezeket az engedélyeket.</span><span class="sxs-lookup"><span data-stu-id="be110-151">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="be110-152">Például a [beépített **tulajdonosa** szerepkör](/azure/role-based-access-control/built-in-roles#owner) lehetővé teszi a felhasználóknak egy erőforráson bármely művelet elvégzésére.</span><span class="sxs-lookup"><span data-stu-id="be110-152">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="be110-153">A következő vezérlő annak ellenőrzése, hogy a megadott beállítások alapján engedélyezi a kérelmet [Azure erőforrás-szabályzat](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="be110-153">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="be110-154">Az Azure erőforrás-házirendek adjon meg egy adott erőforrás engedélyezett műveletek.</span><span class="sxs-lookup"><span data-stu-id="be110-154">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="be110-155">Azure erőforrás-szabályzat megadhatja például, hogy csak engedélyezett a felhasználók számára egy adott típusú virtuális gép üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="be110-155">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="be110-156">![Az Azure erőforrás-szabályzat](../../_images/governance-1-19.png)
*11. ábra. Az Azure erőforrás-szabályzat.*</span><span class="sxs-lookup"><span data-stu-id="be110-156">![Azure resource policy](../../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="be110-157">A következő vezérlő annak ellenőrzése, hogy nem haladja meg a kérelem egy [Azure előfizetésre vonatkozó korlát](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="be110-157">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="be110-158">Az egyes előfizetésekhez például 980 erőforráscsoportok előfizetésenként legfeljebb rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="be110-158">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="be110-159">Ha a kérelem érkezik egy további erőforráscsoport üzembe helyezéséhez, a korlát elérése után, a rendszer megtagadja.</span><span class="sxs-lookup"><span data-stu-id="be110-159">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="be110-160">![Az Azure erőforrás-korlátozások](../../_images/governance-1-20.png)
*12. ábra. Az Azure erőforrás-korlátait.*</span><span class="sxs-lookup"><span data-stu-id="be110-160">![Azure resource limits](../../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="be110-161">A végső vezérlő annak ellenőrzése, hogy a vonatkozó kérés, a pénzügyi kötelezettségvállalás, az előfizetéshez tartozó belül.</span><span class="sxs-lookup"><span data-stu-id="be110-161">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="be110-162">Például ha a kérelem egy virtuális gép üzembe helyezése, Azure Resource Manager ellenőrzi, hogy az előfizetés rendelkezik elegendő fizetési adatait.</span><span class="sxs-lookup"><span data-stu-id="be110-162">For example, if the request is to deploy a virtual machine, Azure Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="be110-163">![Az egy előfizetéshez tartozó pénzügyi kötelezettségvállalás](../../_images/governance-1-21.png)
*13. ábra. A pénzügyi kötelezettségvállalás nélkül kapcsolódik előfizetés.*</span><span class="sxs-lookup"><span data-stu-id="be110-163">![A financial commitment associated with a subscription](../../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="be110-164">Összegzés</span><span class="sxs-lookup"><span data-stu-id="be110-164">Summary</span></span>

<span data-ttu-id="be110-165">Ebből a cikkből megismerhette erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban Azure Resource Manager használatával.</span><span class="sxs-lookup"><span data-stu-id="be110-165">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="be110-166">További lépések</span><span class="sxs-lookup"><span data-stu-id="be110-166">Next steps</span></span>

<span data-ttu-id="be110-167">Most, hogy megismerkedett az Azure-beli erőforrás-hozzáférés kezelése, helyezze át megtudhatja, hogyan kell irányítási mintájának [egy egyszerű számítási feladat](governance-simple-workload.md) vagy [több csapat](governance-multiple-teams.md) használja ezeket a szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="be110-167">Now that you understand how to manage resource access in Azure, move on to learn how to design a governance model [for a simple workload](governance-simple-workload.md) or for [multiple teams](governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="be110-168">Cégirányítási áttekintése</span><span class="sxs-lookup"><span data-stu-id="be110-168">An overview of governance</span></span>](../overview.md)
