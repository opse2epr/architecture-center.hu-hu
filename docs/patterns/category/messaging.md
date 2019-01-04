---
title: Üzenetkezelési minták
titleSuffix: Cloud Design Patterns
description: A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.
keywords: tervezési minta
author: dragon119
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 4619d30c152f050f3f95aee3f9983b8fe85911ed
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011327"
---
# <a name="messaging-patterns"></a>Üzenetkezelési minták

[!INCLUDE [header](../../_includes/header.md)]

A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.

| Mintázat | Összegzés |
| ------- | ------- |
| [Competing Consumers](../competing-consumers.md) | Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket. |
| [Pipes and Filters](../pipes-and-filters.md) | Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává. |
| [Priority Queue](../priority-queue.md) | Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat. |
| [Közzétevő-előfizető](../publisher-subscriber.md) | Engedélyezheti egy alkalmazás számára, hogy több érdeklődő fogyasztó számára aszinkron módon, a küldők és a fogadók összekapcsolása nélkül jelentsen be eseményeket. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) | Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket. |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) | Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon. |
