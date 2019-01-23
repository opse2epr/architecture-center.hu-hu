---
title: Teljesítménnyel kapcsolatos és skálázhatósági minták
titleSuffix: Cloud Design Patterns
description: A teljesítmény egy rendszer válaszkészségét mutatja egy művelet adott időtartamon belül való végrehajtásának vonatkozásában, a skálázhatóság pedig a rendszer azon képessége, hogy tudja-e kezelni a terhelés növekedését a teljesítmény romlása nélkül, vagy tudja-e azonnal növelni a rendelkezésre álló erőforrásokat. A felhőalapú alkalmazások általában különböző munkaterheléseknek és aktivitási csúcspontoknak vannak kitéve. Ezeket előre megjósolni, főleg egy több-bérlős forgatókönyvben, szinte lehetetlen. Az alkalmazásoknak inkább a horizontális felskálázásra kell képesnek lenniük bizonyos korlátok között, hogy megfeleljenek a csúcspont idején az igényeknek, és elvégezzék a horizontálisan leskálázást is, ha csökken az igény. A skálázhatóság nem csak a számítási példányokat érinti, de más elemeket is, például az adattárolást, az üzenetkezelési infrastruktúrát stb.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a57194f70218a8294507bd389ea9526660e3041b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480725"
---
# <a name="performance-and-scalability-patterns"></a>Teljesítménnyel kapcsolatos és skálázhatósági minták

[!INCLUDE [header](../../_includes/header.md)]

A teljesítmény egy rendszer válaszkészségét mutatja egy művelet adott időtartamon belül való végrehajtásának vonatkozásában, a skálázhatóság pedig a rendszer azon képessége, hogy tudja-e kezelni a terhelés növekedését a teljesítmény romlása nélkül, vagy tudja-e azonnal növelni a rendelkezésre álló erőforrásokat. A felhőalapú alkalmazások általában különböző munkaterheléseknek és aktivitási csúcspontoknak vannak kitéve. Ezeket előre megjósolni, főleg egy több-bérlős forgatókönyvben, szinte lehetetlen. Az alkalmazásoknak inkább a horizontális felskálázásra kell képesnek lenniük bizonyos korlátok között, hogy megfeleljenek a csúcspont idején az igényeknek, és elvégezzék a horizontálisan leskálázást is, ha csökken az igény. A skálázhatóság nem csak a számítási példányokat érinti, de más elemeket is, például az adattárolást, az üzenetkezelési infrastruktúrát stb.

|                           Pattern                            |                                                                        Összefoglalás                                                                         |
|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|               [Cache-Aside](../cache-aside.md)               |                                                   Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból                                                   |
|                      [CQRS](../cqrs.md)                      |                           Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.                           |
|            [Event Sourcing](../event-sourcing.md)            |                     Használhat egy csak hozzáfűzéssel bővíthető tárat az egy tartomány adatain elvégzett műveleteket leíró események teljes sorozatának rögzítésére.                      |
|               [Index Table](../index-table.md)               |                                Indexeket hozhat létre a lekérdezések által gyakran hivatkozott adattárbeli mezőkről.                                |
|         [Materialized View](../materialized-view.md)         |       Létrehozhat előre kitöltött nézeteket egy vagy több adattár adataiból, ha az adatok formázása nem ideális a szükséges lekérdezési műveletekhez.        |
|            [Priority Queue](../priority-queue.md)            | Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat. |
| [Queue-Based Load Leveling](../queue-based-load-leveling.md) |              Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.               |
|                  [Sharding](../sharding.md)                  |                                           Egy adattárat horizontális partíció- vagy szilánkkészletté oszthat fel.                                           |
|    [Static Content Hosting](../static-content-hosting.md)    |                          A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.                          |
|                [Szabályozás](../throttling.md)                |                Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását.                 |
