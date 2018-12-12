---
title: Üzenetkezelési minták
description: A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.
keywords: tervezési minta
author: dragon119
ms.date: 12/07/2018
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 754e9a7dcad20dc1c4471af00f3f142f18022d62
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233880"
---
# <a name="messaging-patterns"></a><span data-ttu-id="f5276-105">Üzenetkezelési minták</span><span class="sxs-lookup"><span data-stu-id="f5276-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="f5276-106">A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással.</span><span class="sxs-lookup"><span data-stu-id="f5276-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="f5276-107">Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.</span><span class="sxs-lookup"><span data-stu-id="f5276-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="f5276-108">Mintázat</span><span class="sxs-lookup"><span data-stu-id="f5276-108">Pattern</span></span> | <span data-ttu-id="f5276-109">Összegzés</span><span class="sxs-lookup"><span data-stu-id="f5276-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="f5276-110">Competing Consumers</span><span class="sxs-lookup"><span data-stu-id="f5276-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="f5276-111">Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="f5276-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="f5276-112">Pipes and Filters</span><span class="sxs-lookup"><span data-stu-id="f5276-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="f5276-113">Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.</span><span class="sxs-lookup"><span data-stu-id="f5276-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="f5276-114">Priority Queue</span><span class="sxs-lookup"><span data-stu-id="f5276-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="f5276-115">Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat.</span><span class="sxs-lookup"><span data-stu-id="f5276-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="f5276-116">Közzétevő-előfizető</span><span class="sxs-lookup"><span data-stu-id="f5276-116">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="f5276-117">Engedélyezheti egy alkalmazás jelentjük be az eseményeket az érdekelt fogyasztók aynchronously több, a fogadók a feladói kapcsoló nélkül.</span><span class="sxs-lookup"><span data-stu-id="f5276-117">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="f5276-118">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="f5276-118">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="f5276-119">Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.</span><span class="sxs-lookup"><span data-stu-id="f5276-119">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="f5276-120">Scheduler Agent Supervisor</span><span class="sxs-lookup"><span data-stu-id="f5276-120">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="f5276-121">Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon.</span><span class="sxs-lookup"><span data-stu-id="f5276-121">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
