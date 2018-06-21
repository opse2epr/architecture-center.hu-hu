---
title: 'Útmutató: Azure-erőforrás csoportjainak terve'
description: Útmutató az Azure erőforrás-csoport kialakítása a eligazodást felhő bevezetési stratégia részeként
author: petertay
ms.openlocfilehash: ac6cbb03be8cdba020641d3b9034ad9d20101acf
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
ms.locfileid: "29060841"
---
# <a name="guidance-azure-resource-group-design"></a>Útmutató: Azure-erőforrás csoportjainak terve

Az Azure a [erőforráscsoport](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#resource-groups) , amelyben az erőforrások vannak csoportosítva logikai tárolója. Az egyes Azure szolgáltatásba telepített erőforrások kell telepíthető az egyetlen erőforráscsoportként működnek.

## <a name="design-considerations"></a>Kialakítási szempontok

- Összes erőforrást erőforráscsoportban kell megosztani az azonos életciklussal. A gyakorlatban ez azt jelenti, hogy a központi telepítése, frissítése és törlése az egy csoportként azonos életciklussal rendelkező erőforrásokat kell lennie. Például egy egyetlen webalkalmazás számítási erőforrások általában telepítése egyetlen egységként. Azonban más webes alkalmazásokat a megosztott adatbázis egy másik életciklussal valószínűleg szeretné kezelni, és a saját erőforráscsoportban kell lennie.
- Az erőforráscsoportok tartalmazhatnak olyan erőforrásokat, amelyek különböző régiókban találhatók.
- Egyetlen erőforráscsoportként működnek az összes erőforrást egy-egy előfizetéshez társítva kell lennie. 
- Egy erőforrás helyezhető, erőforráscsoport-sablonok között, azonban nem telepített egy másik előfizetésben található erőforrás tartalmazó csoport.
- Egy erőforrás csoport-hozzárendelés nem befolyásolja a kapcsolatot vagy egyéb erőforráscsoportok erőforrásaihoz való együttműködéshez. Például egy virtuális gépet egy erőforráscsoporthoz rendelt tud kapcsolódni egy adatbázishoz rendelve egy másik erőforráscsoportban, ha van egymás közötti hálózati kapcsolat.
- Az erőforráscsoport segítségével meghatározhatja a hozzáférés-vezérlési hatókört felügyeleti műveletekhez. Szerepköralapú hozzáférés-vezérlést (RBAC) engedélyekkel az előfizetések szintjén, illetve az erőforráscsoport szintjén is alkalmazhat. Az előfizetés szintjén jogosultságait örökölt, az erőforráscsoport szintjén. További információk az RBAC-és erőforrás megtanulhatja a köztes és speciális elfogadása szakaszaiban.

## <a name="proven-practices"></a>Bevált gyakorlatok

- A legalapvetőbb fázisban kezelt valószínűleg csak néhány a koncepció igazolása (POC) projektre, az erőforrások kis számú. Mert Koncepció erőforrások általában ugyanazt az azonos életciklussal, az egyes projektek is létrehozhat egyetlen erőforráscsoportként működnek.
- A köztes Bevezetési fázis során felügyelni kívánt több projektet. Különböző típusú projektek is kihasználhatja a le más erőforrás csoport tervek. Ha szeretné léptetni a kezdeti Koncepció projektek üzemi bármelyikét, áthelyezheti erőforrásokat egy másik erőforráscsoportban Ha ugyanahhoz az előfizetéshez tartozik. Ezért ezen a ponton kell telepít ezeket az erőforrásokat ugyanazt az előfizetést, a jövőben erőforrások átrendezéséhez.

## <a name="next-steps"></a>További lépések

* Most, hogy rendelkezik megismerte a bevált gyakorlat a legalapvetőbb elfogadása. szakaszra vonatkozóan, akkor képesek erőforráscsoportokat létrehozni és erőforrások hozzáadása. Ezen a ponton kezelt erőforrások kis számú, amíg azok válik összetettebb, mert további erőforrások hozzáadása. További tudnivalók [Azure elnevezési konvenciók és címkézés](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json) nevet, és az erőforrások címkét a köztes Bevezetési fázis előkészítésekor.
