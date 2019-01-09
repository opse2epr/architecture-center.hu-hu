---
title: Az Azure számítási szolgáltatások döntési fája
titleSuffix: Azure Application Architecture Guide
description: Számítási szolgáltatás kiválasztására szolgáló folyamatábra.
author: MikeWasson
ms.date: 11/03/2018
ms.custom: seojan19
ms.openlocfilehash: 905b9956c9dcddddb21a87ea588af0ad5160ae2a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114079"
---
# <a name="decision-tree-for-azure-compute-services"></a>Az Azure számítási szolgáltatások döntési fája

Az Azure számos módon üzemeltetni az alkalmazás kódjában kínál. A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. A következő folyamatábra szemlélteti, hogy az alkalmazás számítási szolgáltatás kiválasztása segít. A folyamatábra végigvezeti fő döntési feltételcsoport elérni egy javaslatot.

**Ez a folyamatábra kezeljük kiindulási pontként.** Minden alkalmazásnak egyedi követelményeit, ezért a kiindulási pontként a javaslatot. Hajtsa végre a részletesebb próbaverzióra, például Hibaoldal szempontok:

- Szolgáltatáskészlet
- [Szolgáltatási korlátozások](/azure/azure-subscription-service-limits)
- [Költségek](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [Régiónkénti rendelkezésre állás](https://azure.microsoft.com/global-infrastructure/services/)
- Fejlesztői ökoszisztéma és team képességei
- [Összehasonlító táblák COMPUTE](./compute-comparison.md)

Ha az alkalmazás több számítási feladatot tartalmaz, minden számítási feladat külön-külön kell kiértékelni. Teljes körű megoldást tartalmazhatják a két vagy több számítási szolgáltatást.

A tárolók az Azure-beli üzemeltetéséhez szükséges beállításokkal kapcsolatos további információkért lásd: [Azure tárolók](https://azure.microsoft.com/overview/containers/).

## <a name="flowchart"></a>Folyamatábra

![Az Azure számítási szolgáltatások döntési fája](../images/compute-decision-tree.svg)

## <a name="definitions"></a>Meghatározások

- **Lift- and -shift** stratégiát egy számítási feladat áttelepítését a felhőbe az alkalmazás újratervezése vagy a kód módosítása nélkül is. Más néven *újratárolása*. További információkért lásd: [Azure migrálási központ](https://azure.microsoft.com/migration/).

- **Optimalizált felhőalapú** van, az újrabontás natív funkciók és képességek kihasználásához egy alkalmazás által a felhőbe való migrálás stratégiája.

## <a name="next-steps"></a>További lépések

További szempontokat kell figyelembe venni: [számítási szolgáltatás Azure-beli kritériumai](./compute-comparison.md).
