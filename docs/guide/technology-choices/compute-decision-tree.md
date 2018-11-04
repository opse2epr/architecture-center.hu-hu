---
title: Az Azure számítási szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 11/03/2018
ms.openlocfilehash: cb074272b8d00a71223d8c5755ef8cde3a3f2592
ms.sourcegitcommit: 225251ee2dd669432a9c9abe3aa8cd84d9e020b7
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/04/2018
ms.locfileid: "50981980"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="1a1da-103">Az Azure számítási szolgáltatások döntési fája</span><span class="sxs-lookup"><span data-stu-id="1a1da-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="1a1da-104">Az Azure számos módon üzemeltetni az alkalmazás kódjában kínál.</span><span class="sxs-lookup"><span data-stu-id="1a1da-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="1a1da-105">A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut.</span><span class="sxs-lookup"><span data-stu-id="1a1da-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="1a1da-106">A következő folyamatábra szemlélteti, hogy az alkalmazás számítási szolgáltatás kiválasztása segít.</span><span class="sxs-lookup"><span data-stu-id="1a1da-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="1a1da-107">A folyamatábra végigvezeti fő döntési feltételcsoport elérni egy javaslatot.</span><span class="sxs-lookup"><span data-stu-id="1a1da-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="1a1da-108">**Ez a folyamatábra kezeljük kiindulási pontként.**</span><span class="sxs-lookup"><span data-stu-id="1a1da-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="1a1da-109">Minden alkalmazásnak egyedi követelményeit, ezért a kiindulási pontként a javaslatot.</span><span class="sxs-lookup"><span data-stu-id="1a1da-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="1a1da-110">Hajtsa végre a részletesebb próbaverzióra, például Hibaoldal szempontok:</span><span class="sxs-lookup"><span data-stu-id="1a1da-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="1a1da-111">A szolgáltatás beállítása</span><span class="sxs-lookup"><span data-stu-id="1a1da-111">Feature set</span></span>
- [<span data-ttu-id="1a1da-112">Szolgáltatási korlátozások</span><span class="sxs-lookup"><span data-stu-id="1a1da-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="1a1da-113">Költségek</span><span class="sxs-lookup"><span data-stu-id="1a1da-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="1a1da-114">SLA</span><span class="sxs-lookup"><span data-stu-id="1a1da-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="1a1da-115">Régiónkénti rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="1a1da-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="1a1da-116">Fejlesztői ökoszisztéma és team képességei</span><span class="sxs-lookup"><span data-stu-id="1a1da-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="1a1da-117">Összehasonlító táblák COMPUTE</span><span class="sxs-lookup"><span data-stu-id="1a1da-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="1a1da-118">Ha az alkalmazás több számítási feladatot tartalmaz, minden számítási feladat külön-külön kell kiértékelni.</span><span class="sxs-lookup"><span data-stu-id="1a1da-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="1a1da-119">Teljes körű megoldást tartalmazhatják a két vagy több számítási szolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="1a1da-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="1a1da-120">A tárolók az Azure-beli üzemeltetéséhez szükséges beállításokkal kapcsolatos további információkért lásd: https://azure.microsoft.com/overview/containers/.</span><span class="sxs-lookup"><span data-stu-id="1a1da-120">For more information about your options for hosting containers in Azure, see https://azure.microsoft.com/overview/containers/.</span></span>

## <a name="flowchart"></a><span data-ttu-id="1a1da-121">Folyamatábra</span><span class="sxs-lookup"><span data-stu-id="1a1da-121">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="1a1da-122">Meghatározások</span><span class="sxs-lookup"><span data-stu-id="1a1da-122">Definitions</span></span>

- <span data-ttu-id="1a1da-123">**Lift- and -shift** stratégiát egy számítási feladat áttelepítését a felhőbe az alkalmazás újratervezése vagy a kód módosítása nélkül is.</span><span class="sxs-lookup"><span data-stu-id="1a1da-123">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="1a1da-124">Más néven *újratárolása*.</span><span class="sxs-lookup"><span data-stu-id="1a1da-124">Also called *rehosting*.</span></span> <span data-ttu-id="1a1da-125">További információkért lásd: [Azure migrálási központ](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="1a1da-125">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="1a1da-126">**Optimalizált felhőalapú** van, az újrabontás natív funkciók és képességek kihasználásához egy alkalmazás által a felhőbe való migrálás stratégiája.</span><span class="sxs-lookup"><span data-stu-id="1a1da-126">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1a1da-127">További lépések</span><span class="sxs-lookup"><span data-stu-id="1a1da-127">Next steps</span></span>

<span data-ttu-id="1a1da-128">További szempontokat kell figyelembe venni: [számítási szolgáltatás Azure-beli kritériumai](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="1a1da-128">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
