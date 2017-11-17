---
title: "Teljesítmény és méretezhetőség minták"
description: "Teljesítmény utalhat, hogy a rendszer egy adott időintervallumban belül bármely művelet végrehajtásához a figyelt, a során a rendszer azon képessége, a méretezhetőség azt vagy nő a terhelés nélkül kezelésére hatással lehet a teljesítményre, vagy a rendelkezésre álló erőforrásokat lehet könnyen nagyobb. A felhőalapú alkalmazásokhoz előforduló változó munkaterhelések és csúcsait általában tevékenységben. Ezek előrejelzésére, különösen a több-bérlős forgatókönyvek szinte lehetetlen. Ehelyett alkalmazások kell tudni horizontális felskálázás az igény szerinti csúcsait, és a skála teljesítéséhez igény szerinti csökkenése esetén határokon belül. Méretezhetőség aggályokat nem csak a példányok, de egyéb elemek, mint például az adatok tárolási és üzenetkezelési infrastruktúra több számítási."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 26e5fe0bc05bff7b9fb684795a2de3b57d945ae0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="performance-and-scalability-patterns"></a>Teljesítmény és méretezhetőség minták

[!INCLUDE [header](../../_includes/header.md)]

Teljesítmény utalhat, hogy a rendszer egy adott időintervallumban belül bármely művelet végrehajtásához a figyelt, a során a rendszer azon képessége, a méretezhetőség azt vagy nő a terhelés nélkül kezelésére hatással lehet a teljesítményre, vagy a rendelkezésre álló erőforrásokat lehet könnyen nagyobb. A felhőalapú alkalmazásokhoz előforduló változó munkaterhelések és csúcsait általában tevékenységben. Ezek előrejelzésére, különösen a több-bérlős forgatókönyvek szinte lehetetlen. Ehelyett alkalmazások kell tudni horizontális felskálázás az igény szerinti csúcsait, és a skála teljesítéséhez igény szerinti csökkenése esetén határokon belül. Méretezhetőség aggályokat nem csak a példányok, de egyéb elemek, mint például az adatok tárolási és üzenetkezelési infrastruktúra több számítási.

| Minta | Összefoglalás |
| ------- | ------- |
| [Gyorsítótár-Tartalékoljon](../cache-aside.md) | Adatok betöltése az igény szerinti egy adattárból a gyorsítótárba |
| [CQRS](../cqrs.md) | Elkülönítse műveletekkel kapcsolatos adatokat olvasni az operatív adatokat frissítő külön-felületek használatával. |
| [Esemény forrás](../event-sourcing.md) | Egy csak append tároló segítségével rögzítheti a teljes eseménysorozatokat, amelyek egy tartományban lévő adatokon műveleteit tartalmazzák. |
| [Index táblázat](../index-table.md) | Indexek létrehozása gyakran lekérdezések által hivatkozott adattárolókhoz mezőinek keresztül. |
| [Materializált nézet](../materialized-view.md) | Az adatokat egy vagy több adattárolókhoz keresztül előfeltöltött nézetek létrehozása az adatok nem ideális formázott kötelező lekérdezési műveletek. |
| [Prioritású várólistára.](../priority-queue.md) | Priorizálhatja azokat, hogy a magasabb prioritású kérelmek fogadott, és gyorsabban dolgozza fel a kisebb prioritással rendelkező szolgáltatások küldött kérelmeket. |
| [Simítás várólista alapú betöltése](../queue-based-load-leveling.md) | Egy feladat és egy szolgáltatás, amely meghívja a érdekében időszakos nagy terhelések közötti pufferként a várólista használja. |
| [Horizontális](../sharding.md) | Adattároló felosztani vízszintes partíciók vagy szilánkok készlete. |
| [Statikus tartalom üzemeltetéséhez](../static-content-hosting.md) | Statikus tartalom központi telepítése a felhőalapú tároló szolgáltatás által biztosított közvetlenül az ügyfél számára. |
| [Szabályozás](../throttling.md) | Egy alkalmazás, egy adott bérlő vagy az egész szolgáltatás egy példánya által használt erőforrások fogyasztásának szabályozzák. |