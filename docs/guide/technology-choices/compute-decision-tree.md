---
title: Az Azure compute szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a>Az Azure compute szolgáltatások döntési fája

Az Azure számos módon az alkalmazás kódjában üzemeltetésére kínál. A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. A következő folyamatábra segítséget az alkalmazás számítási szolgáltatás kiválasztása.
 
![](../images/compute-decision-tree.svg)

A folyamatábra végigvezeti azokon a fő döntési feltételcsoport ajánlást eléréséhez. Minden alkalmazáshoz egyedi követelményekkel rendelkezik, ezért a javaslat kiindulási pontként kell tekinteni. Végezze el a részletesebb elemzés, például megtekint szempontok:
 
- Szolgáltatáskészlete
- [Szolgáltatási korlátozások](/azure/azure-subscription-service-limits)
- [Költség](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [Régiónkénti rendelkezésre állás](https://azure.microsoft.com/global-infrastructure/services/)
- Fejlesztői ökoszisztémától és a team képességek
- [Számítási összehasonlítás táblák](./compute-comparison.md)

Ha az alkalmazás több munkaterhelés, külön-külön értékelje ki minden munkaterhelés. A teljes megoldás tartalmazhatják a két vagy több számítási szolgáltatások.

