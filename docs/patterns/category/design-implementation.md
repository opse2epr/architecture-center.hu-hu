---
title: Tervezési és implementálási minták
description: A jó tervezés olyan tényezőket is figyelembe vesz, mint a konzisztencia és a koherencia az összetevők tervezése és üzembe helyezése során, a karbantarthatóság az adminisztráció és a fejlesztés egyszerűsítéséhez, illetve az újrahasznosíthatóság, hogy az összetevők és alrendszerek más alkalmazásokban és más forgatókönyvekben is hasznosíthatók legyenek. A tervezés és az implementálás fázisában hozott döntések óriási hatással vannak a felhőalapú alkalmazások és szolgáltatások minőségére és teljes tulajdonlási költségére.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 861445ceeca62e5b1e62fd4cb33924c35e10c0b0
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="design-and-implementation-patterns"></a>Tervezési és implementálási minták

A jó tervezés olyan tényezőket is figyelembe vesz, mint a konzisztencia és a koherencia az összetevők tervezése és üzembe helyezése során, a karbantarthatóság az adminisztráció és a fejlesztés egyszerűsítéséhez, illetve az újrahasznosíthatóság, hogy az összetevők és alrendszerek más alkalmazásokban és más forgatókönyvekben is hasznosíthatók legyenek. A tervezés és az implementálás fázisában hozott döntések óriási hatással vannak a felhőalapú alkalmazások és szolgáltatások minőségére és teljes tulajdonlási költségére.


|                                Mintázat                                 |                                                                                                      Összegzés                                                                                                       |
|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Ambassador](../ambassador.md)                     |                                                         Olyan segítő szolgáltatásokat hozhat létre, amelyek egy otthoni használatra szánt szolgáltatás vagy alkalmazás nevében küldenek hálózati kéréseket.                                                          |
|          [Anti-Corruption Layer](../anti-corruption-layer.md)          |                                                               Egy előtér- vagy adapterréteget implementálhat egy korszerű alkalmazás és egy korábbi rendszer között.                                                                |
|         [Backends for Frontends](../backends-for-frontends.md)         |                                                          Elkülönített, adott előtérbeli alkalmazások vagy felületek által használt háttérszolgáltatásokat hozhat létre.                                                          |
|                           [CQRS](../cqrs.md)                           |                                                         Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.                                                         |
| [Compute Resource Consolidation](../compute-resource-consolidation.md) |                                                                     Egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet                                                                      |
|   [External Configuration Store](../external-configuration-store.md)   |                                                        A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre.                                                         |
|            [Gateway Aggregation](../gateway-aggregation.md)            |                                                                   Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével.                                                                   |
|             [Gateway Offloading](../gateway-offloading.md)             |                                                                      A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra.                                                                       |
|                [Gateway Routing](../gateway-routing.md)                |                                                                            Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.                                                                            |
|                [Leader Election](../leader-election.md)                | Koordinálhat egy elosztott alkalmazásban az együttműködő feladatpéldányokból álló gyűjtemény által végrehajtott műveleteket, ha vezetőnek választ meg egy példányt, amely vállalja a többi példány kezelésével járó felelősséget. |
|              [Pipes and Filters](../pipes-and-filters.md)              |                                                     Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.                                                      |
|                        [Sidecar](../sidecar.md)                        |                                                  Egy alkalmazás összetevőit külön folyamatban vagy tárolóban helyezheti üzembe, így elkülönítést és beágyazást biztosíthat.                                                  |
|         [Static Content Hosting](../static-content-hosting.md)         |                                                        A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.                                                        |
|                      [Strangler](../strangler.md)                      |                                         Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél.                                          |

