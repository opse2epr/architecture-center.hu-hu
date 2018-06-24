---
title: Az Azure compute szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 689ec3f265e563273a75ad98268d03624a7b4536
ms.sourcegitcommit: ce2fa8ac2d310f7078317cade12f1b89db1ffe06
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/23/2018
ms.locfileid: "36338183"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="7c660-103">Az Azure compute szolgáltatások döntési fája</span><span class="sxs-lookup"><span data-stu-id="7c660-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="7c660-104">Az Azure számos módon az alkalmazás kódjában üzemeltetésére kínál.</span><span class="sxs-lookup"><span data-stu-id="7c660-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="7c660-105">A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut.</span><span class="sxs-lookup"><span data-stu-id="7c660-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="7c660-106">A következő folyamatábra segítséget az alkalmazás számítási szolgáltatás kiválasztása.</span><span class="sxs-lookup"><span data-stu-id="7c660-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="7c660-107">A folyamatábra végigvezeti azokon a fő döntési feltételcsoport ajánlást eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="7c660-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="7c660-108">**Ez a folyamatábra kezelni kiindulási pontként.**</span><span class="sxs-lookup"><span data-stu-id="7c660-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="7c660-109">Minden alkalmazás rendelkezik egyedi követelmények, így használja a javaslat kiindulási pontként.</span><span class="sxs-lookup"><span data-stu-id="7c660-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="7c660-110">Végezze el a részletes próbaverzióra, például megtekint szempontok:</span><span class="sxs-lookup"><span data-stu-id="7c660-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="7c660-111">Szolgáltatáskészlete</span><span class="sxs-lookup"><span data-stu-id="7c660-111">Feature set</span></span>
- [<span data-ttu-id="7c660-112">Szolgáltatási korlátozások</span><span class="sxs-lookup"><span data-stu-id="7c660-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="7c660-113">Költségek</span><span class="sxs-lookup"><span data-stu-id="7c660-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="7c660-114">SLA</span><span class="sxs-lookup"><span data-stu-id="7c660-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="7c660-115">Régiónkénti rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="7c660-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="7c660-116">Fejlesztői ökoszisztémától és a team képességek</span><span class="sxs-lookup"><span data-stu-id="7c660-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="7c660-117">Számítási összehasonlítás táblák</span><span class="sxs-lookup"><span data-stu-id="7c660-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="7c660-118">Ha az alkalmazás több munkaterhelés, külön-külön értékelje ki minden munkaterhelés.</span><span class="sxs-lookup"><span data-stu-id="7c660-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="7c660-119">A teljes megoldás tartalmazhatják a két vagy több számítási szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="7c660-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="7c660-120">Az Azure-ban tárolók üzemeltetéséhez a beállításokkal kapcsolatos további információkért lásd: https://azure.microsoft.com/overview/containers/.</span><span class="sxs-lookup"><span data-stu-id="7c660-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="7c660-121">Folyamatábra</span><span class="sxs-lookup"><span data-stu-id="7c660-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="7c660-122">Meghatározások</span><span class="sxs-lookup"><span data-stu-id="7c660-122">Definitions</span></span>

- <span data-ttu-id="7c660-123">**Greenfield** teljesen új, és teljesen új beépített szoftver projektet ismerteti.</span><span class="sxs-lookup"><span data-stu-id="7c660-123">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="7c660-124">Nem tartalmazza a hagyományos kódot.</span><span class="sxs-lookup"><span data-stu-id="7c660-124">It does not include legacy code.</span></span> 

- <span data-ttu-id="7c660-125">**Brownfield** ismerteti a szoftver projekt, amely egy meglévő alkalmazást épül.</span><span class="sxs-lookup"><span data-stu-id="7c660-125">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="7c660-126">Azt is öröklik örökölt kód vagy keretrendszert.</span><span class="sxs-lookup"><span data-stu-id="7c660-126">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="7c660-127">**Emelje fel, és az eltolás mértékét megadó** stratégiát a munkaterhelések áttelepítését a felhőbe újratervezése az alkalmazás vagy a kód módosítása nélkül is.</span><span class="sxs-lookup"><span data-stu-id="7c660-127">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="7c660-128">Más néven *áthelyezését*.</span><span class="sxs-lookup"><span data-stu-id="7c660-128">Also called *rehosting*.</span></span> <span data-ttu-id="7c660-129">További információkért lásd: [Azure áttelepítési center](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="7c660-129">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="7c660-130">**Optimalizált felhő** újrabontása felhő eredeti funkciók és képességek előnyeit alkalmazás által a felhőbe történő stratégiát is.</span><span class="sxs-lookup"><span data-stu-id="7c660-130">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="7c660-131">További lépések</span><span class="sxs-lookup"><span data-stu-id="7c660-131">Next steps</span></span>

<span data-ttu-id="7c660-132">Szempontokat kell figyelembe venni, lásd: [feltételek kiválasztása az Azure számítási szolgáltatás](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="7c660-132">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
