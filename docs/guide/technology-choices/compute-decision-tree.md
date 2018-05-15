---
title: Az Azure compute szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="52e5f-103">Az Azure compute szolgáltatások döntési fája</span><span class="sxs-lookup"><span data-stu-id="52e5f-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="52e5f-104">Az Azure számos módon az alkalmazás kódjában üzemeltetésére kínál.</span><span class="sxs-lookup"><span data-stu-id="52e5f-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="52e5f-105">A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut.</span><span class="sxs-lookup"><span data-stu-id="52e5f-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="52e5f-106">A következő folyamatábra segítséget az alkalmazás számítási szolgáltatás kiválasztása.</span><span class="sxs-lookup"><span data-stu-id="52e5f-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="52e5f-107">A folyamatábra végigvezeti azokon a fő döntési feltételcsoport ajánlást eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="52e5f-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="52e5f-108">**Ez a folyamatábra kezelni kiindulási pontként.**</span><span class="sxs-lookup"><span data-stu-id="52e5f-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="52e5f-109">Minden alkalmazás rendelkezik egyedi követelmények, így használja a javaslat kiindulási pontként.</span><span class="sxs-lookup"><span data-stu-id="52e5f-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="52e5f-110">Végezze el a részletes próbaverzióra, például megtekint szempontok:</span><span class="sxs-lookup"><span data-stu-id="52e5f-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="52e5f-111">Szolgáltatáskészlete</span><span class="sxs-lookup"><span data-stu-id="52e5f-111">Feature set</span></span>
- [<span data-ttu-id="52e5f-112">Szolgáltatási korlátozások</span><span class="sxs-lookup"><span data-stu-id="52e5f-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="52e5f-113">Költségek</span><span class="sxs-lookup"><span data-stu-id="52e5f-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="52e5f-114">SLA</span><span class="sxs-lookup"><span data-stu-id="52e5f-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="52e5f-115">Régiónkénti rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="52e5f-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="52e5f-116">Fejlesztői ökoszisztémától és a team képességek</span><span class="sxs-lookup"><span data-stu-id="52e5f-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="52e5f-117">Számítási összehasonlítás táblák</span><span class="sxs-lookup"><span data-stu-id="52e5f-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="52e5f-118">Ha az alkalmazás több munkaterhelés, külön-külön értékelje ki minden munkaterhelés.</span><span class="sxs-lookup"><span data-stu-id="52e5f-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="52e5f-119">A teljes megoldás tartalmazhatják a két vagy több számítási szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="52e5f-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

