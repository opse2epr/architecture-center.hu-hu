---
title: 'Enterprise Cloud Adoption: Erőforráshozzáférés-kezelés az Azure-ban'
description: 'Erőforráshozzáférés-kezelés magyarázata hoz létre az Azure-ban: Az Azure resource manager, az előfizetések, erőforráscsoportok és erőforrások'
author: petertaylor9999
ms.date: 09/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 9f1d28cfeb062f44b2c58cd52c58f163d6de5b9d
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483071"
---
# <a name="enterprise-cloud-adoption-resource-access-management-in-azure"></a>Enterprise Cloud Adoption: Erőforráshozzáférés-kezelés az Azure-ban

A [Mi az erőforrás-szabályozás?](what-is-governance.md), a segítségével megtanulta, hogy cégirányítási vonatkozik-e folyamatban kezelését, megfigyelését és naplózás használata Azure-erőforrások megfelelnek a kitűzött célokat, és a szervezet követelményeinek. Áthelyezni a megtudhatja, hogyan kell irányítási mintájának, mielőtt fontos tudni, hogy az erőforrás-hozzáférési felügyeleti vezérlők az Azure-ban. Erőforrás-hozzáférési felügyeleti vezérlőelemek konfigurációját a cégirányítási modell alapját.

Lássunk is használatával hogyan erőforrások vannak üzembe helyezve, az Azure-ban közelebbről. 

## <a name="what-is-an-azure-resource"></a>Mi az Azure-erőforrásban?

Az Azure-ban az előfizetési időszak **erőforrás** egy Azure által kezelt entitásra hivatkozik. Ha például virtuális gépek, virtuális hálózatok és tárfiókok az összes nevezzük Azure-erőforrások.

![](../_images/governance-1-9.png)   
*1. ábra Erőforrás.*

## <a name="what-is-an-azure-resource-group"></a>Mi az Azure-erőforráscsoport?

Az Azure-ban minden egyes erőforráscsoportba kell tartoznia a [erőforráscsoport](/azure/azure-resource-manager/resource-group-overview#resource-groups). Egy erőforráscsoport egyszerűen egy logikai szerkezet, amely több erőforrást együttesen csoportokat, hogy egyetlen egységként kezelhetők. Például egy hasonló az életciklusuk, például az erőforrások erőforrásokat egy [n szintű alkalmazás](/azure/architecture/guide/architecture-styles/n-tier) létrehozott vagy törölt csoportként. 

![](../_images/governance-1-10.png)   
*2. ábra Egy erőforráscsoport erőforrást tartalmaz.* 

Erőforráscsoportok és az erőforrások kapcsolódnak az Azure-beli **előfizetés**. 

## <a name="what-is-an-azure-subscription"></a>Mi az Azure-előfizetéssel?

Azure-előfizetés nagyon hasonlít egy erőforráscsoportot, abban, hogy egy logikai szerkezet, amely csoportosítja együtt erőforráscsoportokról és azok erőforrásokhoz. Azonban egy Azure-előfizetést is társítva az Azure resource manager által használt vezérlőkkel. Ez mit jelent? Vizsgáljuk meg közelebbről megismerheti és az Azure-előfizetések közötti kapcsolatot az Azure resource manager.

![](../_images/governance-1-11.png)   
*3. ábra Azure-előfizetés.*

## <a name="what-is-azure-resource-manager"></a>Mi az Azure resource manager?

A [Azure működése?](what-is-azure.md) megtanulta, hogy az Azure számos olyan szolgáltatás, amely az Azure minden funkcióját összehangolására az "előtér" tartalmaz. Ezek a szolgáltatások egyik [az Azure resource manager](/azure/azure-resource-manager/), és ez a szolgáltatás üzemelteti a RESTful API-ügyfelek által használt erőforrások kezeléséhez. 

![](../_images/governance-1-12.png)   
*4. ábra Az Azure resource manager.*

A következő ábrán látható három ügyfelek: [PowerShell](/powershell/azure/overview), [az Azure Portalon](https://portal.azure.com), és a [Azure parancssori felület (CLI)](/cli/azure):

![](../_images/governance-1-13.png)   
*5. ábra Az Azure-ügyfelek csatlakoznak az Azure resource manager REST API.*

Bár ezek az ügyfelek csatlakozni az Azure resource manager a RESTful API-val, az Azure resource manager nem tartalmaz közvetlenül kezelheti az erőforrásokat funkciót. Ehelyett a legtöbb erőforrástípusok az Azure-ban megvan a saját [ **erőforrás-szolgáltató**](/azure/azure-resource-manager/resource-group-overview#terminology). 

![](../_images/governance-1-14.png)   
*6. ábra Az Azure erőforrás-szolgáltatók.*

Amikor egy ügyfél kérést küld egy adott erőforrás kezeléséhez, az Azure resource manager kapcsolódik az erőforrás-szolgáltató az erőforrás típusát, a kérés teljesítéséhez. Például ha egy ügyfél kérést küld egy virtuális gép típusú erőforrást kezelésére, az Azure resource manager csatlakozik a **Microsoft.compute** erőforrás-szolgáltató. 

![](../_images/governance-1-15.png)   
*7. ábra Az Azure resource manager kapcsolódik a **Microsoft.compute** kezeléséhez az ügyfél kérésében megadott erőforrás erőforrás-szolgáltató.*

Az Azure resource manager megköveteli, hogy adja meg az előfizetés és az erőforráscsoport azonosítóját ahhoz, hogy a virtuális gép típusú erőforrást kezelése az ügyfél. 

Most, hogy az Azure resource manager works megismerése, nézzük térjen vissza a hozzászólás, hogyan Azure-előfizetéssel társítva az Azure resource manager által használt vezérlőkkel. Mielőtt bármilyen erőforrás felügyeleti kérelmet az Azure resource manager által hajtható végre, a vezérlők be van jelölve. 

Az első vezérlőelem, hogy a kérelmet kell kezdeményeznie, ha igazolt felhasználója, és a egy megbízható kapcsolattal rendelkezik az Azure resource manager [Azure Active Directory (Azure AD)](/azure/active-directory/) felhasználói identitáskezelési funkciókat biztosít.

![](../_images/governance-1-16.png)   
*8. ábra Azure Active Directory.*

Azure ad-felhasználók szegmentált be **bérlők**. Egy bérlő egy logikai szerkezet, amely általában egy szervezet társított Azure AD egy biztonságos, dedikált példányát jelöli. Minden egyes előfizetés társítva az Azure AD-bérlő.

![](../_images/governance-1-17.png)   
*9. ábra. Az egy előfizetéshez tartozó Azure AD-bérlővel.*

Minden ügyfélkérés kezelésére egy erőforrást egy adott előfizetés szükséges, hogy a felhasználó rendelkezik egy fiókot a társított Azure AD-bérlővel. 

A következő vezérlő, hogy a felhasználó rendelkezik-e megfelelő engedélye a kérés ellenőrzése. Engedélyek használatával rendelkező felhasználókhoz rendelt [szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/role-based-access-control/).

![](../_images/governance-1-18.png)   
*10. ábra. A bérlő minden felhasználója egy vagy több RBAC szerepkör van hozzárendelve.*

Az RBAC szerepkör engedélyekkel a felhasználó egy adott erőforráson eltarthat egy halmazát határozza meg. A szerepkör van rendelve a felhasználót, amikor a rendszer alkalmazza ezeket az engedélyeket. Például a [beépített **tulajdonosa** szerepkör](/azure/role-based-access-control/built-in-roles#owner) lehetővé teszi a felhasználóknak egy erőforráson bármely művelet elvégzésére.

A következő vezérlő annak ellenőrzése, hogy a megadott beállítások alapján engedélyezi a kérelmet [Azure erőforrás-szabályzat](/azure/governance/policy/). Az Azure erőforrás-házirendek adjon meg egy adott erőforrás engedélyezett műveletek. Azure erőforrás-szabályzat megadhatja például, hogy csak engedélyezett a felhasználók számára egy adott típusú virtuális gép üzembe helyezése.

![](../_images/governance-1-19.png)   
*11. ábra. Az Azure erőforrás-szabályzat.*

A következő vezérlő annak ellenőrzése, hogy nem haladja meg a kérelem egy [Azure előfizetésre vonatkozó korlát](/azure/azure-subscription-service-limits). Az egyes előfizetésekhez például 980 erőforráscsoportok előfizetésenként legfeljebb rendelkezik. Ha a kérelem érkezik egy további erőforráscsoport üzembe helyezéséhez, a korlát elérése után, a rendszer megtagadja.

![](../_images/governance-1-20.png)   
*12. ábra. Az Azure erőforrás-korlátait.* 

A végső vezérlő annak ellenőrzése, hogy a vonatkozó kérés, a pénzügyi kötelezettségvállalás, az előfizetéshez tartozó belül. Például ha a kérelem egy virtuális gép üzembe helyezése, az Azure resource manager ellenőrzi, hogy az előfizetés rendelkezik elegendő fizetési adatait.

![](../_images/governance-1-21.png)   
*13. ábra. A pénzügyi kötelezettségvállalás nélkül kapcsolódik előfizetés.*

## <a name="summary"></a>Összefoglalás

Ebből a cikkből megismerhette erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban az Azure resource manager használatával.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett az erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban, helyezze át megtudhatja, hogyan kell irányítási mintájának [egyetlen csapat](../governance/governance-single-team.md) vagy [több csapat](../governance/governance-multiple-teams.md) használja ezeket a szolgáltatásokat.

> [!div class="nextstepaction"]
> [Cégirányítási áttekintése](../governance/overview.md)
