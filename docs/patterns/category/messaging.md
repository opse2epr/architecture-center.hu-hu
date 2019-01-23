---
title: Üzenetkezelési minták
titleSuffix: Cloud Design Patterns
description: A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.
keywords: tervezési minta
author: dragon119
ms.date: 12/07/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 2f8a34afc5e6c427bf5635b7a3be316719211112
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485162"
---
# <a name="messaging-patterns"></a>Üzenetkezelési minták

[!INCLUDE [header](../../_includes/header.md)]

A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.

| Pattern | Összefoglalás |
| ------- | ------- |
| [Competing Consumers](../competing-consumers.md) | Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket. |
| [Pipes and Filters](../pipes-and-filters.md) | Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává. |
| [Priority Queue](../priority-queue.md) | Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat. |
| [Publisher-Subscriber](../publisher-subscriber.md) | Engedélyezheti egy alkalmazás számára, hogy több érdeklődő fogyasztó számára aszinkron módon, a küldők és a fogadók összekapcsolása nélkül jelentsen be eseményeket. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) | Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket. |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) | Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon. |
