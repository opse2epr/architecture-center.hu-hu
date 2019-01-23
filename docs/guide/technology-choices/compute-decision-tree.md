---
title: Az Azure számítási szolgáltatások döntési fája
titleSuffix: Azure Application Architecture Guide
description: Számítási szolgáltatás kiválasztására szolgáló folyamatábra.
author: MikeWasson
ms.date: 11/03/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: e3df1cbdd049e8c40597a85eca29899d8d0d1bc3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484652"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="9e49e-103">Az Azure számítási szolgáltatások döntési fája</span><span class="sxs-lookup"><span data-stu-id="9e49e-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="9e49e-104">Az Azure számos módon üzemeltetni az alkalmazás kódjában kínál.</span><span class="sxs-lookup"><span data-stu-id="9e49e-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="9e49e-105">A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut.</span><span class="sxs-lookup"><span data-stu-id="9e49e-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="9e49e-106">A következő folyamatábra szemlélteti, hogy az alkalmazás számítási szolgáltatás kiválasztása segít.</span><span class="sxs-lookup"><span data-stu-id="9e49e-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="9e49e-107">A folyamatábra végigvezeti fő döntési feltételcsoport elérni egy javaslatot.</span><span class="sxs-lookup"><span data-stu-id="9e49e-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span>

<span data-ttu-id="9e49e-108">**Ez a folyamatábra kezeljük kiindulási pontként.**</span><span class="sxs-lookup"><span data-stu-id="9e49e-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="9e49e-109">Minden alkalmazásnak egyedi követelményeit, ezért a kiindulási pontként a javaslatot.</span><span class="sxs-lookup"><span data-stu-id="9e49e-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="9e49e-110">Hajtsa végre a részletesebb próbaverzióra, például Hibaoldal szempontok:</span><span class="sxs-lookup"><span data-stu-id="9e49e-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>

- <span data-ttu-id="9e49e-111">Szolgáltatáskészlet</span><span class="sxs-lookup"><span data-stu-id="9e49e-111">Feature set</span></span>
- [<span data-ttu-id="9e49e-112">Szolgáltatási korlátozások</span><span class="sxs-lookup"><span data-stu-id="9e49e-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="9e49e-113">Költségek</span><span class="sxs-lookup"><span data-stu-id="9e49e-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="9e49e-114">SLA</span><span class="sxs-lookup"><span data-stu-id="9e49e-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="9e49e-115">Régiónkénti rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="9e49e-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="9e49e-116">Fejlesztői ökoszisztéma és team képességei</span><span class="sxs-lookup"><span data-stu-id="9e49e-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="9e49e-117">Összehasonlító táblák COMPUTE</span><span class="sxs-lookup"><span data-stu-id="9e49e-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="9e49e-118">Ha az alkalmazás több számítási feladatot tartalmaz, minden számítási feladat külön-külön kell kiértékelni.</span><span class="sxs-lookup"><span data-stu-id="9e49e-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="9e49e-119">Teljes körű megoldást tartalmazhatják a két vagy több számítási szolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="9e49e-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="9e49e-120">A tárolók az Azure-beli üzemeltetéséhez szükséges beállításokkal kapcsolatos további információkért lásd: [Azure tárolók](https://azure.microsoft.com/overview/containers/).</span><span class="sxs-lookup"><span data-stu-id="9e49e-120">For more information about your options for hosting containers in Azure, see [Azure Containers](https://azure.microsoft.com/overview/containers/).</span></span>

## <a name="flowchart"></a><span data-ttu-id="9e49e-121">Folyamatábra</span><span class="sxs-lookup"><span data-stu-id="9e49e-121">Flowchart</span></span>

![Az Azure számítási szolgáltatások döntési fája](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="9e49e-123">Definíciók</span><span class="sxs-lookup"><span data-stu-id="9e49e-123">Definitions</span></span>

- <span data-ttu-id="9e49e-124">**Lift- and -shift** stratégiát egy számítási feladat áttelepítését a felhőbe az alkalmazás újratervezése vagy a kód módosítása nélkül is.</span><span class="sxs-lookup"><span data-stu-id="9e49e-124">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="9e49e-125">Más néven *újratárolása*.</span><span class="sxs-lookup"><span data-stu-id="9e49e-125">Also called *rehosting*.</span></span> <span data-ttu-id="9e49e-126">További információkért lásd: [Azure migrálási központ](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="9e49e-126">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="9e49e-127">**Optimalizált felhőalapú** van, az újrabontás natív funkciók és képességek kihasználásához egy alkalmazás által a felhőbe való migrálás stratégiája.</span><span class="sxs-lookup"><span data-stu-id="9e49e-127">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9e49e-128">További lépések</span><span class="sxs-lookup"><span data-stu-id="9e49e-128">Next steps</span></span>

<span data-ttu-id="9e49e-129">További szempontokat kell figyelembe venni: [számítási szolgáltatás Azure-beli kritériumai](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="9e49e-129">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
