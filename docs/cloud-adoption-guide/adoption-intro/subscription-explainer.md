---
title: "Explainer: Mi az Azure-előfizetés?"
description: "Ismerteti az Azure-előfizetések, fiókok, és kínál"
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="5b8cb-103">Explainer: Mi az Azure-előfizetés?</span><span class="sxs-lookup"><span data-stu-id="5b8cb-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="5b8cb-104">Az a [Mi az az Azure Active Directory-bérlő?](tenant-explainer.md) explainer a cikkben megtanulta, hogy a szervezet digitális identitásokat az Azure Active Directory-bérlő van tárolva.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="5b8cb-105">Is megtanulta, hogy Azure megbízik az Azure Active Directory létrehozása, olvasása, frissítése vagy erőforrás törlése küldött felhasználói kérelmek hitelesítéséhez.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="5b8cb-106">Alapvetően megértéséhez miért szükség, ezeket a műveleteket, erőforrás - megakadályozhatja, hogy a nem hitelesített és a nem hitelesített felhasználók a hozzáférés az erőforrásokhoz való hozzáférés korlátozása.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="5b8cb-107">Azonban ezen erőforrás elvégzett művelet egy szervezet szeretne szabályozni, például az erőforrások száma a felhasználó vagy felhasználók csoportja számára megengedett létrehozása tulajdonságokat, és ezeket az erőforrásokat futtatásához költséget.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="5b8cb-108">Azure valósítja meg a vezérlő, és a neve egy **előfizetés**.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="5b8cb-109">Egy előfizetés csoportok felhasználókat és az erőforrásokat, amelyek azoknak a felhasználóknak hozott létre.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="5b8cb-110">Ezen erőforrások mindegyikének hozzájárul az [általános korlát] [ subscription-service-limits] az adott erőforráshoz.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="5b8cb-111">A szervezetek előfizetések segítségével kezelheti a költségeket és a felhasználók, csoportok, projektek, vagy sok más stratégiák segítségével erőforrást.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="5b8cb-112">Ezek a stratégiák a köztes és speciális Bevezetési fázis cikkek tárgyalja.</span><span class="sxs-lookup"><span data-stu-id="5b8cb-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="5b8cb-113">További lépések</span><span class="sxs-lookup"><span data-stu-id="5b8cb-113">Next steps</span></span>

* <span data-ttu-id="5b8cb-114">Most, hogy az Azure-előfizetések megismerte olvashat [-előfizetés létrehozása](subscription.md) az első Azure-erőforrások létrehozása előtt...</span><span class="sxs-lookup"><span data-stu-id="5b8cb-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
