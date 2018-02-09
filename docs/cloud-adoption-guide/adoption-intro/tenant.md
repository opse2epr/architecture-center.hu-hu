---
title: "Útmutató: Azure AD bérlő kialakítása"
description: "Azure-bérlőhöz tervezési útmutató a legalapvetőbb felhő bevezetési stratégia részeként"
author: telmosampaio
ms.openlocfilehash: 5bf710f74bde04e041f2896b4a638c3513e4aa44
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="8ebdb-103">Útmutató: Azure AD bérlő kialakítása</span><span class="sxs-lookup"><span data-stu-id="8ebdb-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="8ebdb-104">Az Azure AD-bérlő biztosít a digitális identitásokat szolgáltatások és egy vagy több használt névterek [Azure-előfizetések](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="8ebdb-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="8ebdb-105">Ha a legalapvetőbb elfogadása vázlat, már megtanulta, [az Azure AD-bérlő beszerzése][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="8ebdb-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="8ebdb-106">Kialakítási szempontok</span><span class="sxs-lookup"><span data-stu-id="8ebdb-106">Design considerations</span></span>

- <span data-ttu-id="8ebdb-107">A legalapvetőbb elfogadása szakaszban el lehet kezdeni az egyetlen Azure AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="8ebdb-108">A szervezet meglévő Office 365-előfizetéssel, vagy az Azure-előfizetéssel rendelkezik, ha már rendelkezik az Azure AD-bérlő használhat.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="8ebdb-109">Ha nem rendelkezik vagy vagy ezek részletesebb információk [az Azure AD-bérlő beszerzése][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="8ebdb-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="8ebdb-110">A köztes és speciális elfogadása fázisból áll akkor megtudhatja, hogyan szinkronizálásához, vagy összevonni a helyszíni címtárakat az Azure ad-val.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="8ebdb-111">Ez lehetővé teszi a helyszíni digitális identitás használatára az Azure ad-ben.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="8ebdb-112">Azonban a legalapvetőbb szakaszban fel szeretné venni az új felhasználók, amelyek csak az egyetlen az identitással rendelkezik az Azure AD-bérlő.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="8ebdb-113">Ezeket az identitásokat kezeléséért is lehet.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-113">You are be responsible for managing those identities.</span></span> <span data-ttu-id="8ebdb-114">Például, hogy az új Azure-bA a helyi Active Directory-felhasználók, az Azure AD az board-felhasználók, amelyek már nem szeretne van Azure-erőforrások eléréséhez, és egyéb felhasználói engedélyek vált.</span><span class="sxs-lookup"><span data-stu-id="8ebdb-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="8ebdb-115">További lépések</span><span class="sxs-lookup"><span data-stu-id="8ebdb-115">Next Steps</span></span>

* <span data-ttu-id="8ebdb-116">Most, hogy az Azure AD-bérlő, további [a felhasználó hozzáadása][azure-ad-add-user].</span><span class="sxs-lookup"><span data-stu-id="8ebdb-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="8ebdb-117">Miután egy vagy több új felhasználók felvétele az Azure AD-bérlő, a következő lépés megtanulni [Azure-előfizetések](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="8ebdb-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json