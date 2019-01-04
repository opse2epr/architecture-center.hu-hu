---
title: Rugalmassági minták
titleSuffix: Cloud Design Patterns
description: A rugalmasság a rendszer azon képessége, hogy könnyedén kezelje a hibákat, és gyorsan helyreálljon. A felhőalapú üzemeltetés jellegéből (ahol az alkalmazások gyakran több-bérlősek, megosztott platformszolgáltatásokat használnak, versenyeznek az erőforrásokért és a sávszélességért, az interneten keresztül kommunikálnak, és hagyományos hardvereken futnak) következik, hogy nagyobb a valószínűsége, hogy átmeneti és állandóbb hibák is megjelennek. A hibák észlelése és a gyors és hatékony helyreállás szükséges a rugalmasság fenntartásához.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d478c0fb42e89c6bb5d84b4d077259ff4bf35f16
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009712"
---
# <a name="resiliency-patterns"></a>Rugalmassági minták

A rugalmasság a rendszer azon képessége, hogy könnyedén kezelje a hibákat, és gyorsan helyreálljon. A felhőalapú üzemeltetés jellegéből (ahol az alkalmazások gyakran több-bérlősek, megosztott platformszolgáltatásokat használnak, versenyeznek az erőforrásokért és a sávszélességért, az interneten keresztül kommunikálnak, és hagyományos hardvereken futnak) következik, hogy nagyobb a valószínűsége, hogy átmeneti és állandóbb hibák is megjelennek. A hibák észlelése és a gyors és hatékony helyreállás szükséges a rugalmasság fenntartásához.

|                            Mintázat                             |                                                                                                      Összegzés                                                                                                       |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                   [Bulkhead](../bulkhead.md)                   |                                                     Készletekbe választja szét egy alkalmazás elemeit, hogy ha az egyik meghibásodna, a többi tovább üzemeljen.                                                      |
|            [Circuit Breaker](../circuit-breaker.md)            |                                                  Ha távoli szolgáltatáshoz vagy erőforráshoz csatlakozik, kezelheti azokat a hibákat, amelyek javítása esetleg sok időt venne igénybe.                                                   |
|   [Compensating Transaction](../compensating-transaction.md)   |                                                      Visszavonhat egy sorozatnyi, együttesen végül konzisztens műveletet meghatározó lépés által végrehajtott munkát.                                                       |
| [Health Endpoint Monitoring](../health-endpoint-monitoring.md) |                                            Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök.                                            |
|            [Leader Election](../leader-election.md)            | Koordinálhat egy elosztott alkalmazásban az együttműködő feladatpéldányokból álló gyűjtemény által végrehajtott műveleteket, ha vezetőnek választ meg egy példányt, amely vállalja a többi példány kezelésével járó felelősséget. |
|  [Queue-Based Load Leveling](../queue-based-load-leveling.md)  |                                            Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.                                             |
|                      [Retry](../retry.md)                      |             Engedélyezheti egy alkalmazás számára a szolgáltatásokhoz vagy hálózati erőforrásokhoz való csatlakozáskor jelentkező előre jelzett, átmeneti meghibásodások kezelését egy korábban meghiúsult művelet transzparens módon való ismételt megkísérlésével.             |
| [Scheduler Agent Supervisor](../scheduler-agent-supervisor.md) |                                                            Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon.                                                            |