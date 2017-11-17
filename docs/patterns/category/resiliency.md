---
title: "Rugalmasság minták"
description: "Rugalmasság azt a képességet szabályosan és kezelésére helyreállni hibák után a rendszer. A jellege üzemeltet a felhőben, amelyben alkalmazások is gyakran több-bérlős, megosztott platform szolgáltatásainak használatába, erőforrások és a sávszélesség, kommunikálnak az interneten keresztül, és futtassa a hagyományos hardver azt jelenti, hogy létezik egy nagyobb valószínűséggel \"versenyeznek\", amely mindkét átmeneti és állandó hibákat okozhat. Hibák észlelésére, és a helyreállítás gyors és hatékony, rugalmassági fenntartásához szükséges."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a3b9d72989e0de57c689bcec51e20653d0441d31
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="resiliency-patterns"></a>Rugalmasság minták

Rugalmasság azt a képességet szabályosan és kezelésére helyreállni hibák után a rendszer. A jellege üzemeltet a felhőben, amelyben alkalmazások is gyakran több-bérlős, megosztott platform szolgáltatásainak használatába, erőforrások és a sávszélesség, kommunikálnak az interneten keresztül, és futtassa a hagyományos hardver azt jelenti, hogy létezik egy nagyobb valószínűséggel "versenyeznek", amely mindkét átmeneti és állandó hibákat okozhat. Hibák észlelésére, és a helyreállítás gyors és hatékony, rugalmassági fenntartásához szükséges.

| Minta | Összefoglalás |
| ------- | ------- |
| [Válaszfal](../bulkhead.md) | Különítse el a készletekbe kérelem elemei, hogy ha nem sikerül, a többi tovább működik. |
| [Áramköri megszakító](../circuit-breaker.md) | Kezeli olyan hárítsa el a távoli szolgáltatás vagy az erőforrás való csatlakozáskor a változó időt is igénybe vehet. |
| [Tranzakció Compensating](../compensating-transaction.md) | Több lépésből áll, amelyek együtt határozza meg, hogy idővel konzisztenssé művelet által végzett munka visszavonása. |
| [Végpont állapotfigyelés](../health-endpoint-monitoring.md) | Funkcionális ellenőrzések megvalósítását a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető alkalmazást. |
| [Vezető választás](../leader-election.md) | Koordinálja az elosztott alkalmazásban lévő együttműködés task példányokat gyűjteménye egy példánya, amely azt feltételezi, hogy a többi példány felelős a vezetőjeként megválasztását által végrehajtott műveletekről. |
| [Simítás várólista alapú betöltése](../queue-based-load-leveling.md) | Egy feladat és egy szolgáltatás, amely meghívja a érdekében időszakos nagy terhelések közötti pufferként a várólista használja. |
| [Próbálja meg újra](../retry.md) | Lehetővé teszik az alkalmazások, szeretné kezelni a várható, ideiglenes hibák csatlakozik egy szolgáltatás vagy a hálózati erőforráshoz transzparens módon megpróbálásával korábban sikertelen műveletet. |
| [A Feladatütemező ügynök felügyelő](../scheduler-agent-supervisor.md) | Egy készletét műveletek között elosztott készlete szolgáltatások és az egyéb távoli erőforrásokhoz. |