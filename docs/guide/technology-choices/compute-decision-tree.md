---
title: Az Azure compute szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 60bb84d4bf210888d3d43498db043b6e452f6a80
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206940"
---
# <a name="decision-tree-for-azure-compute-services"></a>Az Azure compute szolgáltatások döntési fája

Az Azure számos módon az alkalmazás kódjában üzemeltetésére kínál. A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. A következő folyamatábra segítséget az alkalmazás számítási szolgáltatás kiválasztása. A folyamatábra végigvezeti azokon a fő döntési feltételcsoport ajánlást eléréséhez. 

**Ez a folyamatábra kezelni kiindulási pontként.** Minden alkalmazás rendelkezik egyedi követelmények, így használja a javaslat kiindulási pontként. Végezze el a részletes próbaverzióra, például megtekint szempontok:
 
- Szolgáltatáskészlete
- [Szolgáltatási korlátozások](/azure/azure-subscription-service-limits)
- [Költségek](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [Régiónkénti rendelkezésre állás](https://azure.microsoft.com/global-infrastructure/services/)
- Fejlesztői ökoszisztémától és a team képességek
- [Számítási összehasonlítás táblák](./compute-comparison.md)

Ha az alkalmazás több munkaterhelés, külön-külön értékelje ki minden munkaterhelés. A teljes megoldás tartalmazhatják a két vagy több számítási szolgáltatások.

## <a name="flowchart"></a>Folyamatábra

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a>Definíciók

- **Greenfield** teljesen új, és teljesen új beépített szoftver projektet ismerteti. Nem tartalmazza a hagyományos kódot. 

- **Brownfield** ismerteti a szoftver projekt, amely egy meglévő alkalmazást épül. Azt is öröklik örökölt kód vagy keretrendszert.

- **Emelje fel, és az eltolás mértékét megadó** stratégiát a munkaterhelések áttelepítését a felhőbe újratervezése az alkalmazás vagy a kód módosítása nélkül is. Más néven *áthelyezését*. További információkért lásd: [Azure áttelepítési center](https://azure.microsoft.com/migration/).

- **Optimalizált felhő** újrabontása felhő eredeti funkciók és képességek előnyeit alkalmazás által a felhőbe történő stratégiát is.

## <a name="next-steps"></a>További lépések

Szempontokat kell figyelembe venni, lásd: [feltételek kiválasztása az Azure számítási szolgáltatás](./compute-comparison.md).
