---
title: "Rendelkezésre állási minták"
description: "Rendelkezésre állási ideje, hogy a rendszer működési, és üzemel arányát határozza meg. Rendszerhibák, infrastrukturális problémák megoldását, rosszindulatú támadások, érinti, és a rendszer betölteni. Általában a hasznos üzemidő százalékaként értendő. A felhőalapú alkalmazásokhoz általában tájékoztatják a felhasználókat a szolgáltatásiszint-szerződéssel (SLA), ami azt jelenti, hogy alkalmazásokat úgy kell megtervezni és végrehajtani úgy, hogy a lehető legnagyobbra növeli a rendelkezésre állási."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a>Rendelkezésre állási minták

[!INCLUDE [header](../../_includes/header.md)]

Rendelkezésre állási ideje, hogy a rendszer működési, és üzemel arányát határozza meg. Rendszerhibák, infrastrukturális problémák megoldását, rosszindulatú támadások, érinti, és a rendszer betölteni. Általában a hasznos üzemidő százalékaként értendő. A felhőalapú alkalmazásokhoz általában tájékoztatják a felhasználókat a szolgáltatásiszint-szerződéssel (SLA), ami azt jelenti, hogy alkalmazásokat úgy kell megtervezni és végrehajtani úgy, hogy a lehető legnagyobbra növeli a rendelkezésre állási.

| Minta | Összefoglalás |
| ------- | ------- |
| [Végpont állapotfigyelés](../health-endpoint-monitoring.md) | Funkcionális ellenőrzések megvalósítását a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető alkalmazást. |
| [Simítás várólista alapú betöltése](../queue-based-load-leveling.md) | Egy feladat és egy szolgáltatás, amely meghívja a érdekében időszakos nagy terhelések közötti pufferként a várólista használja. |
| [Szabályozás](../throttling.md) | Egy alkalmazás, egy adott bérlő vagy az egész szolgáltatás egy példánya által használt erőforrások fogyasztásának szabályozzák. |