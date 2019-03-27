---
title: Üzenetkezelési minták
titleSuffix: Cloud Design Patterns
description: A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.
keywords: tervezési minta
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: e2079d8ed33dd205c155348d455f6380fff660a3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299617"
---
# <a name="messaging-patterns"></a><span data-ttu-id="f2b6c-105">Üzenetkezelési minták</span><span class="sxs-lookup"><span data-stu-id="f2b6c-105">Messaging patterns</span></span>

<span data-ttu-id="f2b6c-106">A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="f2b6c-107">Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="f2b6c-108">Mintázat</span><span class="sxs-lookup"><span data-stu-id="f2b6c-108">Pattern</span></span> | <span data-ttu-id="f2b6c-109">Összegzés</span><span class="sxs-lookup"><span data-stu-id="f2b6c-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="f2b6c-110">Jogcím-ellenőrzés</span><span class="sxs-lookup"><span data-stu-id="f2b6c-110">Claim Check</span></span>](../claim-check.md) | <span data-ttu-id="f2b6c-111">Nagy méretű üzenet felosztása egy jogcím-ellenőrzést és a egy hasznos egy üzenetbusz túlterhelésének elkerülése érdekében.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-111">Split a large message into a claim check and a payload to avoid overwhelming a message bus.</span></span> |
| [<span data-ttu-id="f2b6c-112">Competing Consumers</span><span class="sxs-lookup"><span data-stu-id="f2b6c-112">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="f2b6c-113">Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-113">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="f2b6c-114">Pipes and Filters</span><span class="sxs-lookup"><span data-stu-id="f2b6c-114">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="f2b6c-115">Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-115">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="f2b6c-116">Priority Queue</span><span class="sxs-lookup"><span data-stu-id="f2b6c-116">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="f2b6c-117">Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-117">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="f2b6c-118">Publisher-Subscriber</span><span class="sxs-lookup"><span data-stu-id="f2b6c-118">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="f2b6c-119">Engedélyezheti egy alkalmazás számára, hogy több érdeklődő fogyasztó számára aszinkron módon, a küldők és a fogadók összekapcsolása nélkül jelentsen be eseményeket.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-119">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="f2b6c-120">Queue-Based Load Leveling</span><span class="sxs-lookup"><span data-stu-id="f2b6c-120">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="f2b6c-121">Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-121">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="f2b6c-122">Scheduler Agent Supervisor</span><span class="sxs-lookup"><span data-stu-id="f2b6c-122">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="f2b6c-123">Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon.</span><span class="sxs-lookup"><span data-stu-id="f2b6c-123">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
