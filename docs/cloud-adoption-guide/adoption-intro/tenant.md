---
title: 'Útmutató: Az Azure Active Directory-bérlő kialakítása'
description: Azure-bérlőhöz tervezési útmutató a legalapvetőbb felhő bevezetési stratégia részeként
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/27/2018
ms.locfileid: "29563455"
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="298d8-103">Útmutató: Az Azure Active Directory-bérlő kialakítása</span><span class="sxs-lookup"><span data-stu-id="298d8-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="298d8-104">Az Azure AD-bérlő biztosít a digitális identitásokat szolgáltatások és egy vagy több használt névterek [Azure-előfizetések](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="298d8-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="298d8-105">Ha a legalapvetőbb elfogadása vázlat, már megtanulta, [az Azure AD-bérlő beszerzése][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="298d8-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="298d8-106">Kialakítási szempontok</span><span class="sxs-lookup"><span data-stu-id="298d8-106">Design considerations</span></span>

- <span data-ttu-id="298d8-107">A legalapvetőbb elfogadása szakaszban el lehet kezdeni az egyetlen Azure AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="298d8-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="298d8-108">A szervezet meglévő Office 365-előfizetéssel, vagy az Azure-előfizetéssel rendelkezik, ha már rendelkezik az Azure AD-bérlő használhat.</span><span class="sxs-lookup"><span data-stu-id="298d8-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="298d8-109">Ha nem rendelkezik vagy vagy ezek részletesebb információk [az Azure AD-bérlő beszerzése][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="298d8-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="298d8-110">A köztes és speciális elfogadása fázisból áll akkor megtudhatja, hogyan szinkronizálásához, vagy összevonni a helyszíni címtárakat az Azure ad-val.</span><span class="sxs-lookup"><span data-stu-id="298d8-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="298d8-111">Ez lehetővé teszi a helyszíni digitális identitás használatára az Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="298d8-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="298d8-112">Azonban a legalapvetőbb szakaszban fel szeretné venni az új felhasználók, amelyek csak az egyetlen az identitással rendelkezik az Azure AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="298d8-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="298d8-113">Ezeket az identitásokat kezeléséért fogja.</span><span class="sxs-lookup"><span data-stu-id="298d8-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="298d8-114">Például, hogy az új Azure-bA a helyi Active Directory-felhasználók, az Azure AD az board-felhasználók, amelyek már nem szeretne van Azure-erőforrások eléréséhez, és egyéb felhasználói engedélyek vált.</span><span class="sxs-lookup"><span data-stu-id="298d8-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="298d8-115">További lépések</span><span class="sxs-lookup"><span data-stu-id="298d8-115">Next Steps</span></span>

* <span data-ttu-id="298d8-116">Most, hogy az Azure AD-bérlő, további [a felhasználó hozzáadása][azure-ad-add-user].</span><span class="sxs-lookup"><span data-stu-id="298d8-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="298d8-117">Miután egy vagy több új felhasználók felvétele az Azure AD-bérlő, a következő lépés megtanulni [Azure-előfizetések](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="298d8-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
