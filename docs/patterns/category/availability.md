---
title: Rendelkezésre állási minták
titleSuffix: Cloud Design Patterns
description: A rendelkezésre állás azt az időarányt adja meg, amíg a rendszer működik és üzemel. Befolyásolhatják rendszerhibák, infrastruktúraproblémák, rosszindulatú támadások vagy a rendszer terhelése. Általában az üzemidő százalékaként értendő. A felhőalapú alkalmazások általában biztosítanak a felhasználók számára egy szolgáltatói szerződést (SLA), ami azt jelenti, hogy az alkalmazásokat a rendelkezésre állást maximalizálva kell kialakítani és megvalósítani.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 897bc3c87beb23770220ef0fc3c5c4394d192f2a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298729"
---
# <a name="availability-patterns"></a><span data-ttu-id="c1534-107">Rendelkezésre állási minták</span><span class="sxs-lookup"><span data-stu-id="c1534-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="c1534-108">A rendelkezésre állás azt az időarányt adja meg, amíg a rendszer működik és üzemel.</span><span class="sxs-lookup"><span data-stu-id="c1534-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="c1534-109">Befolyásolhatják rendszerhibák, infrastruktúraproblémák, rosszindulatú támadások vagy a rendszer terhelése.</span><span class="sxs-lookup"><span data-stu-id="c1534-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="c1534-110">Általában az üzemidő százalékaként értendő.</span><span class="sxs-lookup"><span data-stu-id="c1534-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="c1534-111">A felhőalapú alkalmazások általában biztosítanak a felhasználók számára egy szolgáltatói szerződést (SLA), ami azt jelenti, hogy az alkalmazásokat a rendelkezésre állást maximalizálva kell kialakítani és megvalósítani.</span><span class="sxs-lookup"><span data-stu-id="c1534-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

|                            <span data-ttu-id="c1534-112">Mintázat</span><span class="sxs-lookup"><span data-stu-id="c1534-112">Pattern</span></span>                             |                                                           <span data-ttu-id="c1534-113">Összegzés</span><span class="sxs-lookup"><span data-stu-id="c1534-113">Summary</span></span>                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="c1534-114">Health Endpoint Monitoring</span><span class="sxs-lookup"><span data-stu-id="c1534-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="c1534-115">Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök.</span><span class="sxs-lookup"><span data-stu-id="c1534-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
|  [<span data-ttu-id="c1534-116">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="c1534-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  | <span data-ttu-id="c1534-117">Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.</span><span class="sxs-lookup"><span data-stu-id="c1534-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>  |
|                 [<span data-ttu-id="c1534-118">Szabályozás</span><span class="sxs-lookup"><span data-stu-id="c1534-118">Throttling</span></span>](../throttling.md)                 |   <span data-ttu-id="c1534-119">Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását.</span><span class="sxs-lookup"><span data-stu-id="c1534-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span>    |
