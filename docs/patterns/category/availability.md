---
title: "Rendelkezésre állási minták"
description: "Rendelkezésre állási ideje, hogy a rendszer működési, és üzemel arányát határozza meg. Rendszerhibák, infrastrukturális problémák megoldását, rosszindulatú támadások, érinti, és a rendszer betölteni. Általában a hasznos üzemidő százalékaként értendő. A felhőalapú alkalmazásokhoz általában tájékoztatják a felhasználókat a szolgáltatásiszint-szerződéssel (SLA), ami azt jelenti, hogy alkalmazásokat úgy kell megtervezni és végrehajtani úgy, hogy a lehető legnagyobbra növeli a rendelkezésre állási."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a><span data-ttu-id="4e199-107">Rendelkezésre állási minták</span><span class="sxs-lookup"><span data-stu-id="4e199-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="4e199-108">Rendelkezésre állási ideje, hogy a rendszer működési, és üzemel arányát határozza meg.</span><span class="sxs-lookup"><span data-stu-id="4e199-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="4e199-109">Rendszerhibák, infrastrukturális problémák megoldását, rosszindulatú támadások, érinti, és a rendszer betölteni.</span><span class="sxs-lookup"><span data-stu-id="4e199-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="4e199-110">Általában a hasznos üzemidő százalékaként értendő.</span><span class="sxs-lookup"><span data-stu-id="4e199-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="4e199-111">A felhőalapú alkalmazásokhoz általában tájékoztatják a felhasználókat a szolgáltatásiszint-szerződéssel (SLA), ami azt jelenti, hogy alkalmazásokat úgy kell megtervezni és végrehajtani úgy, hogy a lehető legnagyobbra növeli a rendelkezésre állási.</span><span class="sxs-lookup"><span data-stu-id="4e199-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

| <span data-ttu-id="4e199-112">Minta</span><span class="sxs-lookup"><span data-stu-id="4e199-112">Pattern</span></span> | <span data-ttu-id="4e199-113">Összefoglalás</span><span class="sxs-lookup"><span data-stu-id="4e199-113">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="4e199-114">Végpont állapotfigyelés</span><span class="sxs-lookup"><span data-stu-id="4e199-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="4e199-115">Funkcionális ellenőrzések megvalósítását a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="4e199-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
| [<span data-ttu-id="4e199-116">Simítás várólista alapú betöltése</span><span class="sxs-lookup"><span data-stu-id="4e199-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="4e199-117">Egy feladat és egy szolgáltatás, amely meghívja a érdekében időszakos nagy terhelések közötti pufferként a várólista használja.</span><span class="sxs-lookup"><span data-stu-id="4e199-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="4e199-118">Szabályozás</span><span class="sxs-lookup"><span data-stu-id="4e199-118">Throttling</span></span>](../throttling.md) | <span data-ttu-id="4e199-119">Egy alkalmazás, egy adott bérlő vagy az egész szolgáltatás egy példánya által használt erőforrások fogyasztásának szabályozzák.</span><span class="sxs-lookup"><span data-stu-id="4e199-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span> |