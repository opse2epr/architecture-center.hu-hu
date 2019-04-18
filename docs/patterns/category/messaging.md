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
ms.openlocfilehash: f838288e10acf9af5776c93972028b6b878bd7b0
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640668"
---
# <a name="messaging-patterns"></a>Üzenetkezelési minták

A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.

| Mintázat | Összegzés |
| ------- | ------- |
| [Claim Check](../claim-check.md) | A nagy méretű üzeneteket jogcímellenőrzésre és hasznos adatra oszthatja fel, hogy elkerülje az üzenetbusz túlterhelését. |
| [Competing Consumers](../competing-consumers.md) | Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket. |
| [Pipes and Filters](../pipes-and-filters.md) | Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává. |
| [Priority Queue](../priority-queue.md) | Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat. |
| [Publisher-Subscriber](../publisher-subscriber.md) | Engedélyezheti egy alkalmazás jelentjük be események több érdekelt fogyasztók aszinkron módon, a fogadók a feladók kapcsoló nélkül. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) | Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket. |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) | Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon. |
