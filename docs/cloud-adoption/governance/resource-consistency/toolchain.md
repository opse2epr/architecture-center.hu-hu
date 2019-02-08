---
title: 'CAF: Az Azure-ban konzisztencia erőforráseszközök'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Az Azure-ban konzisztencia erőforráseszközök
author: BrianBlanchard
ms.openlocfilehash: 68503289f60fbb3682264ff39546ca7b7700cef5
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899346"
---
# <a name="resource-consistency-tools-in-azure"></a>Az Azure-ban konzisztencia erőforráseszközök

[Erőforrás konzisztencia](overview.md) egyike a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md). Ez fegyelem módját a működési környezetben, alkalmazás vagy munkaterhelés kezelésével kapcsolatos házirendek összpontosít. Felhőalapú irányítási öt állapítják belül erőforrás konzisztencia tartalmaz, alkalmazások, számítási feladatok és eszköz teljesítményének figyelését. A skálázási igények figyelembevételével készült, teljesítménytúllépéseknek SLA-t, és kerülje el proaktív módon SLA teljesítménytúllépéseknek automatikus szervizelés útján szükséges feladatok is tartalmaz.

Az Azure-eszközök, amelyek segíthetnek a házirendek és a folyamatok, amelyek támogatják a cégirányítási fegyelem részletes listáját a következő:

|    | [Azure Portal](https://azure.microsoft.com/features/azure-portal/)  | [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview)  | [Azure Blueprints](/azure/governance/blueprints/overview) | [Azure Automation](/azure/automation/automation-intro) | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) |
|---------|---------|---------|---------|---------|---------|
| Erőforrások üzembe helyezése                             | Igen | Igen | Igen | Igen | Nem  |
| Erőforrások kezelése                             | Igen | Igen | Igen | Igen | Nem  |
| Erőforrások sablonok üzembe helyezése             | Nem  | Igen | Nem  | Igen | Nem  |
| Előkészített környezet üzembe helyezés          | Nem  | Nem  | Igen | Nem  | Nem  |
| Erőforrás-csoportok meghatározása                       | Igen | Igen | Igen | Nem  | Nem  |
| Számítási feladatok és a fiók tulajdonosainak kezelése           | Igen | Igen | Igen | Nem  | Nem  |
| Erőforrásokhoz való feltételes hozzáférés kezelése       | Igen | Igen | Igen | Nem  | Nem  |
| Az RBAC-felhasználók konfigurálása                         | Igen | Nem  | Nem  | Nem  | Igen |
| Szerepkörök és engedélyek hozzárendelése az erőforrások | Igen | Igen | Igen | Nem  | Igen |
| Erőforrások közötti függőségek megadása        | Nem  | Igen | Igen | Nem  | Nem  |
| Hozzáférés-vezérlés alkalmazása                         | Igen | Igen | Igen | Nem  | Igen |
| Mérje fel a rendelkezésre állás és méretezhetőség          | Nem  | Nem  | Nem  | Igen | Nem  |
| Erőforrások címkékkel                      | Igen | Igen | Igen | Nem  | Nem  |
| Rendelje hozzá az Azure Policy-szabályok                    | Igen | Igen | Igen | Nem  | Nem  |
| Vész-helyreállítási terv erőforrások         | Igen | Igen | Igen | Nem  | Nem  |
| A alkalmazni az automatikus szervizelés                  | Nem  | Nem  | Nem  | Igen | Nem  |
| A Számlázás kezelése                               | Igen | Nem  | Nem  | Nem  | Nem  |

Az erőforrás konzisztencia eszközök és szolgáltatások, valamint szüksége lesz a teljesítmény és állapotbeli problémák az üzembe helyezett erőforrások monitorozása. [Az Azure Monitor](/azure/azure-monitor/overview) az alapértelmezett figyelési és jelentéskészítési megoldás az Azure-ban. Az Azure Monitor számos olyan egyedi funkciók, amelyek segítségével a felhőbeli erőforrások figyelése és az alábbi lista mutatja, melyik funkció lehetővé teszi cím gyakori figyelési követelmények.

|                                                    | [Azure Portal](https://azure.microsoft.com/features/azure-portal/) | [Application Insights](/azure/application-insights/app-insights-overview) | [Log Analytics](/azure/azure-monitor/log-query/log-query-overview) | [Az Azure Monitor Rest API](/rest/api/monitor/) |
|----------------------------------------------------|--------------|----------------------|---------------|------------------------|
| Virtuális gép telemetriai naplóadatok                 | Nem           | Nem                   | Igen           | Nem                     |
| Virtuális hálózat a telemetriai adatok log              | Nem           | Nem                   | Igen           | Nem                     |
| PaaS szolgáltatások telemetriai adatok                   | Nem           | Nem                   | Igen           | Nem                     |
| Napló Alkalmazáshasználati adatokhoz                     | Nem           | Igen                  | Nem            | Nem                     |
| Jelentések és értesítések konfigurálása                       | Igen          | Nem                   | Nem            | Igen                    |
| Normál jelentések ütemezése vagy egyéni elemző        | Nem           | Nem                   | Nem            | Nem                     |
| Napló- és teljesítményadatokat adatok megjelenítése és elemzése     | Igen          | Nem                   | Nem            | Nem                     |
| Integráció a helyszíni vagy külső figyelési megoldás     | Nem           | Nem                   | Nem            | Igen                    |

Az üzembe helyezés tervezésekor érdemes figyelembe venni, naplózási adatok tárolására, és hogy hogyan integrálja, felhőalapú kell [jelentéskészítő és -szolgáltatások figyelésének](../../decision-guides/log-and-report/overview.md) a meglévő folyamatok és eszközök.

> [!NOTE]
> Szervezetek a számítási feladatok és erőforrások figyelése külső fejlesztési és üzemeltetési eszközök is használhatók. További információkért lásd: [DevOps-eszközök Integrációi](https://azure.microsoft.com/products/devops-tool-integrations/).

# <a name="next-steps"></a>További lépések

Ismerje meg, hogyan hozhat létre, rendelhet hozzá és kezelheti [szabályzatdefiníciók](/azure/governance/policy/) az Azure-ban.
