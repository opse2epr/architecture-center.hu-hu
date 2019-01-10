---
title: 'Enterprise Cloud Adoption: Erőforráshozzáférés-kezelés az Azure-ban'
description: 'Erőforráshozzáférés-kezelés magyarázata hoz létre az Azure-ban: Az Azure resource manager, az előfizetések, erőforráscsoportok és erőforrások'
author: petertaylor9999
ms.date: 09/10/2018
ms.openlocfilehash: 8cb28d0dd5dbdfcef846786083d5197f435bf3ea
ms.sourcegitcommit: 7d9efe716e8c9e99f3fafa9d0213d48c23d9713d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/09/2019
ms.locfileid: "54160791"
---
# <a name="enterprise-cloud-adoption-resource-access-management-in-azure"></a><span data-ttu-id="15c65-103">Enterprise Cloud Adoption: Erőforráshozzáférés-kezelés az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="15c65-103">Enterprise Cloud Adoption: Resource access management in Azure</span></span>

<span data-ttu-id="15c65-104">A [Mi az erőforrás-szabályozás?](what-is-governance.md), a segítségével megtanulta, hogy cégirányítási vonatkozik-e folyamatban kezelését, megfigyelését és naplózás használata Azure-erőforrások megfelelnek a kitűzött célokat, és a szervezet követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="15c65-104">In [what is resource governance?](what-is-governance.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="15c65-105">Áthelyezni a megtudhatja, hogyan kell irányítási mintájának, mielőtt fontos tudni, hogy az erőforrás-hozzáférési felügyeleti vezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="15c65-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="15c65-106">Erőforrás-hozzáférési felügyeleti vezérlőelemek konfigurációját a cégirányítási modell alapját.</span><span class="sxs-lookup"><span data-stu-id="15c65-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="15c65-107">Lássunk is használatával hogyan erőforrások vannak üzembe helyezve, az Azure-ban közelebbről.</span><span class="sxs-lookup"><span data-stu-id="15c65-107">Let's begin by taking a closer look at how resources are deployed in Azure.</span></span> 

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="15c65-108">Mi az Azure-erőforrásban?</span><span class="sxs-lookup"><span data-stu-id="15c65-108">What is an Azure resource?</span></span>

<span data-ttu-id="15c65-109">Az Azure-ban az előfizetési időszak **erőforrás** egy Azure által kezelt entitásra hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="15c65-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="15c65-110">Ha például virtuális gépek, virtuális hálózatok és tárfiókok az összes nevezzük Azure-erőforrások.</span><span class="sxs-lookup"><span data-stu-id="15c65-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

![](../_images/governance-1-9.png)   
<span data-ttu-id="15c65-111">*1. ábra Erőforrás.*</span><span class="sxs-lookup"><span data-stu-id="15c65-111">*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="15c65-112">Mi az Azure-erőforráscsoport?</span><span class="sxs-lookup"><span data-stu-id="15c65-112">What is an Azure resource group?</span></span>

<span data-ttu-id="15c65-113">Az Azure-ban minden egyes erőforráscsoportba kell tartoznia a [erőforráscsoport](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="15c65-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="15c65-114">Egy erőforráscsoport egyszerűen egy logikai szerkezet, amely több erőforrást együttesen csoportokat, hogy egyetlen egységként kezelhetők.</span><span class="sxs-lookup"><span data-stu-id="15c65-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="15c65-115">Például egy hasonló az életciklusuk, például az erőforrások erőforrásokat egy [n szintű alkalmazás](/azure/architecture/guide/architecture-styles/n-tier) létrehozott vagy törölt csoportként.</span><span class="sxs-lookup"><span data-stu-id="15c65-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span> 

![](../_images/governance-1-10.png)   
<span data-ttu-id="15c65-116">*2. ábra Egy erőforráscsoport erőforrást tartalmaz.*</span><span class="sxs-lookup"><span data-stu-id="15c65-116">*Figure 2. A resource group contains a resource.*</span></span> 

<span data-ttu-id="15c65-117">Erőforráscsoportok és az erőforrások kapcsolódnak az Azure-beli **előfizetés**.</span><span class="sxs-lookup"><span data-stu-id="15c65-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span> 

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="15c65-118">Mi az Azure-előfizetéssel?</span><span class="sxs-lookup"><span data-stu-id="15c65-118">What is an Azure subscription?</span></span>

<span data-ttu-id="15c65-119">Azure-előfizetés nagyon hasonlít egy erőforráscsoportot, abban, hogy egy logikai szerkezet, amely csoportosítja együtt erőforráscsoportokról és azok erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="15c65-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="15c65-120">Azonban egy Azure-előfizetést is társítva az Azure resource manager által használt vezérlőkkel.</span><span class="sxs-lookup"><span data-stu-id="15c65-120">However, an Azure subscription is also associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="15c65-121">Ez mit jelent?</span><span class="sxs-lookup"><span data-stu-id="15c65-121">What does this mean?</span></span> <span data-ttu-id="15c65-122">Vizsgáljuk meg közelebbről megismerheti és az Azure-előfizetések közötti kapcsolatot az Azure resource manager.</span><span class="sxs-lookup"><span data-stu-id="15c65-122">Let's take a closer look at Azure resource manager to learn about the relationship between it and an Azure subscription.</span></span>

![](../_images/governance-1-11.png)   
<span data-ttu-id="15c65-123">*3. ábra Azure-előfizetés.*</span><span class="sxs-lookup"><span data-stu-id="15c65-123">*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="15c65-124">Mi az Azure resource manager?</span><span class="sxs-lookup"><span data-stu-id="15c65-124">What is Azure resource manager?</span></span>

<span data-ttu-id="15c65-125">A [Azure működése?](what-is-azure.md) megtanulta, hogy az Azure számos olyan szolgáltatás, amely az Azure minden funkcióját összehangolására az "előtér" tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="15c65-125">In [how does Azure work?](what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="15c65-126">Ezek a szolgáltatások egyik [az Azure resource manager](/azure/azure-resource-manager/), és ez a szolgáltatás üzemelteti a RESTful API-ügyfelek által használt erőforrások kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="15c65-126">One of these services is [Azure resource manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span> 

![](../_images/governance-1-12.png)   
<span data-ttu-id="15c65-127">*4. ábra Az Azure resource manager.*</span><span class="sxs-lookup"><span data-stu-id="15c65-127">*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="15c65-128">A következő ábrán látható három ügyfelek: [PowerShell](/powershell/azure/overview), [az Azure Portalon](https://portal.azure.com), és a [Azure parancssori felület (CLI)](/cli/azure):</span><span class="sxs-lookup"><span data-stu-id="15c65-128">The following figure shows three clients: [Powershell](/powershell/azure/overview), [the Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

![](../_images/governance-1-13.png)   
<span data-ttu-id="15c65-129">*5. ábra Az Azure-ügyfelek csatlakoznak az Azure resource manager REST API.*</span><span class="sxs-lookup"><span data-stu-id="15c65-129">*Figure 5. Azure clients connect to the Azure resource manager RESTful API.*</span></span>

<span data-ttu-id="15c65-130">Bár ezek az ügyfelek csatlakozni az Azure resource manager a RESTful API-val, az Azure resource manager nem tartalmaz közvetlenül kezelheti az erőforrásokat funkciót.</span><span class="sxs-lookup"><span data-stu-id="15c65-130">While these clients connect to Azure resource manager using the RESTful API, Azure resource manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="15c65-131">Ehelyett a legtöbb erőforrástípusok az Azure-ban megvan a saját [ **erőforrás-szolgáltató**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="15c65-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span> 

![](../_images/governance-1-14.png)   
<span data-ttu-id="15c65-132">*6. ábra Az Azure erőforrás-szolgáltatók.*</span><span class="sxs-lookup"><span data-stu-id="15c65-132">*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="15c65-133">Amikor egy ügyfél kérést küld egy adott erőforrás kezeléséhez, az Azure resource manager kapcsolódik az erőforrás-szolgáltató az erőforrás típusát, a kérés teljesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="15c65-133">When a client makes a request to manage a specific resource, Azure resource manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="15c65-134">Például ha egy ügyfél kérést küld egy virtuális gép típusú erőforrást kezelésére, az Azure resource manager csatlakozik a **Microsoft.compute** erőforrás-szolgáltató.</span><span class="sxs-lookup"><span data-stu-id="15c65-134">For example, if a client makes a request to manage a virtual machine resource, Azure resource manager connects to the **Microsoft.compute** resource provider.</span></span> 

![](../_images/governance-1-15.png)   
<span data-ttu-id="15c65-135">*7. ábra Az Azure resource manager kapcsolódik a **Microsoft.compute** kezeléséhez az ügyfél kérésében megadott erőforrás erőforrás-szolgáltató.*</span><span class="sxs-lookup"><span data-stu-id="15c65-135">*Figure 7. Azure resource manager connects to the **Microsoft.compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="15c65-136">Az Azure resource manager megköveteli, hogy adja meg az előfizetés és az erőforráscsoport azonosítóját ahhoz, hogy a virtuális gép típusú erőforrást kezelése az ügyfél.</span><span class="sxs-lookup"><span data-stu-id="15c65-136">Azure resource manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span> 

<span data-ttu-id="15c65-137">Most, hogy az Azure resource manager works megismerése, nézzük térjen vissza a hozzászólás, hogyan Azure-előfizetéssel társítva az Azure resource manager által használt vezérlőkkel.</span><span class="sxs-lookup"><span data-stu-id="15c65-137">Now that you have an understanding of how Azure resource manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="15c65-138">Mielőtt bármilyen erőforrás felügyeleti kérelmet az Azure resource manager által hajtható végre, a vezérlők be van jelölve.</span><span class="sxs-lookup"><span data-stu-id="15c65-138">Before any resource management request can be executed by Azure resource manager, a set of controls are checked.</span></span> 

<span data-ttu-id="15c65-139">Az első vezérlőelem, hogy a kérelmet kell kezdeményeznie, ha igazolt felhasználója, és a egy megbízható kapcsolattal rendelkezik az Azure resource manager [Azure Active Directory (Azure AD)](/azure/active-directory/) felhasználói identitáskezelési funkciókat biztosít.</span><span class="sxs-lookup"><span data-stu-id="15c65-139">The first control is that a request must be made by a validated user, and Azure resource manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

![](../_images/governance-1-16.png)   
<span data-ttu-id="15c65-140">*8. ábra Az Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="15c65-140">*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="15c65-141">Azure ad-felhasználók szegmentált be **bérlők**.</span><span class="sxs-lookup"><span data-stu-id="15c65-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="15c65-142">Egy bérlő egy logikai szerkezet, amely általában egy szervezet társított Azure AD egy biztonságos, dedikált példányát jelöli.</span><span class="sxs-lookup"><span data-stu-id="15c65-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="15c65-143">Minden egyes előfizetés társítva az Azure AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="15c65-143">Each subscription is associated with an Azure AD tenant.</span></span>

![](../_images/governance-1-17.png)   
<span data-ttu-id="15c65-144">*9. ábra. Az egy előfizetéshez tartozó Azure AD-bérlővel.*</span><span class="sxs-lookup"><span data-stu-id="15c65-144">*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="15c65-145">Minden ügyfélkérés kezelésére egy erőforrást egy adott előfizetés szükséges, hogy a felhasználó rendelkezik egy fiókot a társított Azure AD-bérlővel.</span><span class="sxs-lookup"><span data-stu-id="15c65-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span> 

<span data-ttu-id="15c65-146">A következő vezérlő, hogy a felhasználó rendelkezik-e megfelelő engedélye a kérés ellenőrzése.</span><span class="sxs-lookup"><span data-stu-id="15c65-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="15c65-147">Engedélyek használatával rendelkező felhasználókhoz rendelt [szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="15c65-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

![](../_images/governance-1-18.png)   
<span data-ttu-id="15c65-148">*10. ábra. A bérlő minden felhasználója egy vagy több RBAC szerepkör van hozzárendelve.*</span><span class="sxs-lookup"><span data-stu-id="15c65-148">*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="15c65-149">Az RBAC szerepkör engedélyekkel a felhasználó egy adott erőforráson eltarthat egy halmazát határozza meg.</span><span class="sxs-lookup"><span data-stu-id="15c65-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="15c65-150">A szerepkör van rendelve a felhasználót, amikor a rendszer alkalmazza ezeket az engedélyeket.</span><span class="sxs-lookup"><span data-stu-id="15c65-150">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="15c65-151">Például a [beépített **tulajdonosa** szerepkör](/azure/role-based-access-control/built-in-roles#owner) lehetővé teszi a felhasználóknak egy erőforráson bármely művelet elvégzésére.</span><span class="sxs-lookup"><span data-stu-id="15c65-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="15c65-152">A következő vezérlő annak ellenőrzése, hogy a megadott beállítások alapján engedélyezi a kérelmet [Azure erőforrás-szabályzat](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="15c65-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="15c65-153">Az Azure erőforrás-házirendek adjon meg egy adott erőforrás engedélyezett műveletek.</span><span class="sxs-lookup"><span data-stu-id="15c65-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="15c65-154">Azure erőforrás-szabályzat megadhatja például, hogy csak engedélyezett a felhasználók számára egy adott típusú virtuális gép üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="15c65-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

![](../_images/governance-1-19.png)   
<span data-ttu-id="15c65-155">*11. ábra. Az Azure erőforrás-szabályzat.*</span><span class="sxs-lookup"><span data-stu-id="15c65-155">*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="15c65-156">A következő vezérlő annak ellenőrzése, hogy nem haladja meg a kérelem egy [Azure előfizetésre vonatkozó korlát](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="15c65-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="15c65-157">Az egyes előfizetésekhez például 980 erőforráscsoportok előfizetésenként legfeljebb rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="15c65-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="15c65-158">Ha a kérelem érkezik egy további erőforráscsoport üzembe helyezéséhez, a korlát elérése után, a rendszer megtagadja.</span><span class="sxs-lookup"><span data-stu-id="15c65-158">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

![](../_images/governance-1-20.png)   
<span data-ttu-id="15c65-159">*12. ábra. Az Azure erőforrás-korlátait.*</span><span class="sxs-lookup"><span data-stu-id="15c65-159">*Figure 12. Azure resource limits.*</span></span> 

<span data-ttu-id="15c65-160">A végső vezérlő annak ellenőrzése, hogy a vonatkozó kérés, a pénzügyi kötelezettségvállalás, az előfizetéshez tartozó belül.</span><span class="sxs-lookup"><span data-stu-id="15c65-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="15c65-161">Például ha a kérelem egy virtuális gép üzembe helyezése, az Azure resource manager ellenőrzi, hogy az előfizetés rendelkezik elegendő fizetési adatait.</span><span class="sxs-lookup"><span data-stu-id="15c65-161">For example, if the request is to deploy a virtual machine, Azure resource manager verifies that the subscription has sufficient payment information.</span></span>

![](../_images/governance-1-21.png)   
<span data-ttu-id="15c65-162">*13. ábra. A pénzügyi kötelezettségvállalás nélkül kapcsolódik előfizetés.*</span><span class="sxs-lookup"><span data-stu-id="15c65-162">*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="15c65-163">Összegzés</span><span class="sxs-lookup"><span data-stu-id="15c65-163">Summary</span></span>

<span data-ttu-id="15c65-164">Ebből a cikkből megismerhette erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban az Azure resource manager használatával.</span><span class="sxs-lookup"><span data-stu-id="15c65-164">In this article, you learned about how resource access is managed in Azure using Azure resource manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="15c65-165">További lépések</span><span class="sxs-lookup"><span data-stu-id="15c65-165">Next steps</span></span>

<span data-ttu-id="15c65-166">Most, hogy megismerkedett az erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban, helyezze át megtudhatja, hogyan kell irányítási mintájának [egyetlen csapat](../governance/governance-single-team.md) vagy [több csapat](../governance/governance-multiple-teams.md) használja ezeket a szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="15c65-166">Now that you understand how resource access is managed in Azure, move on to learn how to design a governance model [for a single team](../governance/governance-single-team.md) or [multiple teams](../governance/governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="15c65-167">Cégirányítási áttekintése</span><span class="sxs-lookup"><span data-stu-id="15c65-167">An overview of governance</span></span>](../governance/overview.md)
