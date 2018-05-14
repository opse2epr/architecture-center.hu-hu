---
title: Az Azure compute szolgáltatások döntési fája
description: A számítási szolgáltatás kiválasztásának folyamatábrája
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/11/2018
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

![](../images/compute-decision-tree.svg)

