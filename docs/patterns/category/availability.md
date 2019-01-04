---
title: Rendelkezésre állási minták
titleSuffix: Cloud Design Patterns
description: A rendelkezésre állás azt az időarányt adja meg, amíg a rendszer működik és üzemel. Befolyásolhatják rendszerhibák, infrastruktúraproblémák, rosszindulatú támadások vagy a rendszer terhelése. Általában az üzemidő százalékaként értendő. A felhőalapú alkalmazások általában biztosítanak a felhasználók számára egy szolgáltatói szerződést (SLA), ami azt jelenti, hogy az alkalmazásokat a rendelkezésre állást maximalizálva kell kialakítani és megvalósítani.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d256ddd39ca3e8a452162b7b75d67f26f7bb22c2
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009814"
---
# <a name="availability-patterns"></a>Rendelkezésre állási minták

[!INCLUDE [header](../../_includes/header.md)]

A rendelkezésre állás azt az időarányt adja meg, amíg a rendszer működik és üzemel. Befolyásolhatják rendszerhibák, infrastruktúraproblémák, rosszindulatú támadások vagy a rendszer terhelése. Általában az üzemidő százalékaként értendő. A felhőalapú alkalmazások általában biztosítanak a felhasználók számára egy szolgáltatói szerződést (SLA), ami azt jelenti, hogy az alkalmazásokat a rendelkezésre állást maximalizálva kell kialakítani és megvalósítani.

|                            Mintázat                             |                                                           Összegzés                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) | Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök. |
|  [Queue-Based Load Leveling](../queue-based-load-leveling.md)  | Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.  |
|                 [Szabályozás](../throttling.md)                 |   Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását.    |
