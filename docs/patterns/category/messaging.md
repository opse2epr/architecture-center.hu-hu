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
# <a name="messaging-patterns"></a>Üzenetkezelési mintát

[!INCLUDE [header](../../_includes/header.md)]

A felhőalapú alkalmazások elosztott jellege üzenetkezelési infrastruktúra, amely összeköti a összetevőit és szolgáltatásait, ideális méretezhetőség maximalizálása érdekében lazán összekapcsolt módon van szükség. Aszinkron üzenetkezelési széles körben használja, és számos előnnyel jár, de például üzenetek, az elhalt üzenet felügyeleti, idempotencia és több sorrendje kihívást is jelent.

| Minta | Összefoglalás |
| ------- | ------- |
| [Versengő fogyasztó számára](../competing-consumers.md) | Engedélyezze a több egyidejű fogyasztók ugyanazt a üzenetkezelési csatornát a Beérkezett üzenetek feldolgozásához. |
| [Adatcsatornák és a szűrők](../pipes-and-filters.md) | Egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat lebontva. |
| [Prioritású várólistára.](../priority-queue.md) | Priorizálhatja azokat, hogy a magasabb prioritású kérelmek fogadott, és gyorsabban dolgozza fel a kisebb prioritással rendelkező szolgáltatások küldött kérelmeket. |
| [Simítás várólista alapú betöltése](../queue-based-load-leveling.md) | Egy feladat és egy szolgáltatás, amely meghívja a érdekében időszakos nagy terhelések közötti pufferként a várólista használja. |
| [A Feladatütemező ügynök felügyelő](../scheduler-agent-supervisor.md) | Egy készletét műveletek között elosztott készlete szolgáltatások és az egyéb távoli erőforrásokhoz. |