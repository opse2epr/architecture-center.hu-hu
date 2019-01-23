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
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486797"
---
# <a name="availability-patterns"></a>Rendelkezésre állási minták

[!INCLUDE [header](../../_includes/header.md)]

A rendelkezésre állás azt az időarányt adja meg, amíg a rendszer működik és üzemel. Befolyásolhatják rendszerhibák, infrastruktúraproblémák, rosszindulatú támadások vagy a rendszer terhelése. Általában az üzemidő százalékaként értendő. A felhőalapú alkalmazások általában biztosítanak a felhasználók számára egy szolgáltatói szerződést (SLA), ami azt jelenti, hogy az alkalmazásokat a rendelkezésre állást maximalizálva kell kialakítani és megvalósítani.

|                            Pattern                             |                                                           Összefoglalás                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) | Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök. |
|  [Queue-Based Load Leveling](../queue-based-load-leveling.md)  | Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.  |
|                 [Szabályozás](../throttling.md)                 |   Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását.    |
