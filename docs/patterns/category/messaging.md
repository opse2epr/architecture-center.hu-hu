---
title: "Üzenetkezelési mintát"
description: "A felhőalapú alkalmazások elosztott jellege üzenetkezelési infrastruktúra, amely összeköti a összetevőit és szolgáltatásait, ideális méretezhetőség maximalizálása érdekében lazán összekapcsolt módon van szükség. Aszinkron üzenetkezelési széles körben használja, és számos előnnyel jár, de például üzenetek, az elhalt üzenet felügyeleti, idempotencia és több sorrendje kihívást is jelent."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6151f7f76fc7b3a953988122db75bdc25b49811f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="messaging-patterns"></a><span data-ttu-id="bac54-105">Üzenetkezelési mintát</span><span class="sxs-lookup"><span data-stu-id="bac54-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="bac54-106">A felhőalapú alkalmazások elosztott jellege üzenetkezelési infrastruktúra, amely összeköti a összetevőit és szolgáltatásait, ideális méretezhetőség maximalizálása érdekében lazán összekapcsolt módon van szükség.</span><span class="sxs-lookup"><span data-stu-id="bac54-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="bac54-107">Aszinkron üzenetkezelési széles körben használja, és számos előnnyel jár, de például üzenetek, az elhalt üzenet felügyeleti, idempotencia és több sorrendje kihívást is jelent.</span><span class="sxs-lookup"><span data-stu-id="bac54-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="bac54-108">Minta</span><span class="sxs-lookup"><span data-stu-id="bac54-108">Pattern</span></span> | <span data-ttu-id="bac54-109">Összefoglalás</span><span class="sxs-lookup"><span data-stu-id="bac54-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="bac54-110">Versengő fogyasztó számára</span><span class="sxs-lookup"><span data-stu-id="bac54-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="bac54-111">Engedélyezze a több egyidejű fogyasztók ugyanazt a üzenetkezelési csatornát a Beérkezett üzenetek feldolgozásához.</span><span class="sxs-lookup"><span data-stu-id="bac54-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="bac54-112">Adatcsatornák és a szűrők</span><span class="sxs-lookup"><span data-stu-id="bac54-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="bac54-113">Egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat lebontva.</span><span class="sxs-lookup"><span data-stu-id="bac54-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="bac54-114">Prioritású várólistára.</span><span class="sxs-lookup"><span data-stu-id="bac54-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="bac54-115">Priorizálhatja azokat, hogy a magasabb prioritású kérelmek fogadott, és gyorsabban dolgozza fel a kisebb prioritással rendelkező szolgáltatások küldött kérelmeket.</span><span class="sxs-lookup"><span data-stu-id="bac54-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="bac54-116">Simítás várólista alapú betöltése</span><span class="sxs-lookup"><span data-stu-id="bac54-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="bac54-117">Egy feladat és egy szolgáltatás, amely meghívja a érdekében időszakos nagy terhelések közötti pufferként a várólista használja.</span><span class="sxs-lookup"><span data-stu-id="bac54-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="bac54-118">A Feladatütemező ügynök felügyelő</span><span class="sxs-lookup"><span data-stu-id="bac54-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="bac54-119">Egy készletét műveletek között elosztott készlete szolgáltatások és az egyéb távoli erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="bac54-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |