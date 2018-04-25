---
title: Az Azure compute szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="5111b-103">Az Azure compute szolgáltatások döntési fája</span><span class="sxs-lookup"><span data-stu-id="5111b-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="5111b-104">Az Azure számos módon az alkalmazás kódjában üzemeltetésére kínál.</span><span class="sxs-lookup"><span data-stu-id="5111b-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="5111b-105">A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut.</span><span class="sxs-lookup"><span data-stu-id="5111b-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="5111b-106">A következő folyamatábra segítséget az alkalmazás számítási szolgáltatás kiválasztása.</span><span class="sxs-lookup"><span data-stu-id="5111b-106">The following flowchart will help you to choose a compute service for your application.</span></span>
 
![](../images/compute-decision-tree.svg)

<span data-ttu-id="5111b-107">A folyamatábra végigvezeti azokon a fő döntési feltételcsoport ajánlást eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="5111b-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> <span data-ttu-id="5111b-108">Minden alkalmazáshoz egyedi követelményekkel rendelkezik, ezért a javaslat kiindulási pontként kell tekinteni.</span><span class="sxs-lookup"><span data-stu-id="5111b-108">Every application has unique requirements, so you should treat the recommendation as a starting point.</span></span> <span data-ttu-id="5111b-109">Végezze el a részletesebb elemzés, például megtekint szempontok:</span><span class="sxs-lookup"><span data-stu-id="5111b-109">Then perform a more detailed analysis, looking at aspects such as:</span></span>
 
- <span data-ttu-id="5111b-110">Szolgáltatáskészlete</span><span class="sxs-lookup"><span data-stu-id="5111b-110">Feature set</span></span>
- [<span data-ttu-id="5111b-111">Szolgáltatási korlátozások</span><span class="sxs-lookup"><span data-stu-id="5111b-111">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="5111b-112">Költség</span><span class="sxs-lookup"><span data-stu-id="5111b-112">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="5111b-113">SLA</span><span class="sxs-lookup"><span data-stu-id="5111b-113">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="5111b-114">Régiónkénti rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="5111b-114">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="5111b-115">Fejlesztői ökoszisztémától és a team képességek</span><span class="sxs-lookup"><span data-stu-id="5111b-115">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="5111b-116">Számítási összehasonlítás táblák</span><span class="sxs-lookup"><span data-stu-id="5111b-116">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="5111b-117">Ha az alkalmazás több munkaterhelés, külön-külön értékelje ki minden munkaterhelés.</span><span class="sxs-lookup"><span data-stu-id="5111b-117">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="5111b-118">A teljes megoldás tartalmazhatják a két vagy több számítási szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="5111b-118">A complete solution may incorporate two or more compute services.</span></span>

