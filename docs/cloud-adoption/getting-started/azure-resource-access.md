---
title: 'CAF: Erőforráshozzáférés-kezelés az Azure-ban'
description: Erőforráshozzáférés-kezelés ismertetése az Azure-ban – Azure Resource Manager, az előfizetések, erőforráscsoportok és erőforrásokat hoz létre.
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
author: petertaylor9999
ms.openlocfilehash: a72e9fbd6f5f320440d63d55d4e0f2aa2009a2d1
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298454"
---
# <a name="resource-access-management-in-azure"></a>Erőforráshozzáférés-kezelés az Azure-ban

A [Mi az erőforrás-szabályozás?](what-is-governance.md), a segítségével megtanulta, hogy cégirányítási vonatkozik-e folyamatban kezelését, megfigyelését és naplózás használata Azure-erőforrások megfelelnek a kitűzött célokat, és a szervezet követelményeinek. Áthelyezni a megtudhatja, hogyan kell irányítási mintájának, mielőtt fontos tudni, hogy az erőforrás-hozzáférési felügyeleti vezérlők az Azure-ban. Erőforrás-hozzáférési felügyeleti vezérlőelemek konfigurációját a cégirányítási modell alapját.

Lássunk is használatával hogyan erőforrások vannak üzembe helyezve, az Azure-ban közelebbről.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a>Mi az Azure-erőforrásban?

Az Azure-ban az előfizetési időszak **erőforrás** egy Azure által kezelt entitásra hivatkozik. Ha például virtuális gépek, virtuális hálózatok és tárfiókok az összes nevezzük Azure-erőforrások.

![](../_images/governance-1-9.png)
*1. ábra. Erőforrás.*

## <a name="what-is-an-azure-resource-group"></a>Mi az Azure-erőforráscsoport?

Az Azure-ban minden egyes erőforráscsoportba kell tartoznia a [erőforráscsoport](/azure/azure-resource-manager/resource-group-overview#resource-groups). Egy erőforráscsoport egyszerűen egy logikai szerkezet, amely több erőforrást együttesen csoportokat, hogy egyetlen egységként kezelhetők. Például egy hasonló az életciklusuk, például az erőforrások erőforrásokat egy [n szintű alkalmazás](/azure/architecture/guide/architecture-styles/n-tier) létrehozott vagy törölt csoportként.

![](../_images/governance-1-10.png)
*2. ábra. Egy erőforráscsoport erőforrást tartalmaz.*

Erőforráscsoportok és az erőforrások kapcsolódnak az Azure-beli **előfizetés**.

## <a name="what-is-an-azure-subscription"></a>Mi az Azure-előfizetéssel?

Azure-előfizetés nagyon hasonlít egy erőforráscsoportot, abban, hogy egy logikai szerkezet, amely csoportosítja együtt erőforráscsoportokról és azok erőforrásokhoz. Azonban egy Azure-előfizetést is társítva az Azure Resource Manager által használt vezérlőkkel. Ez mit jelent? Vizsgáljuk meg közelebbről a Resource Manager és az Azure-előfizetések közötti kapcsolatot kapcsolatos.

![](../_images/governance-1-11.png)
*3. ábra. Azure-előfizetés.*

## <a name="what-is-azure-resource-manager"></a>Mi az Azure Resource Manager?

A [Azure működése?](what-is-azure.md) megtanulta, hogy az Azure számos olyan szolgáltatás, amely az Azure minden funkcióját összehangolására az "előtér" tartalmaz. Ezek a szolgáltatások egyik [Resource Manager](/azure/azure-resource-manager/), és ez a szolgáltatás üzemelteti a RESTful API-ügyfelek által használt erőforrások kezeléséhez.

![](../_images/governance-1-12.png)
*4. ábra. Az Azure Resource Manager.*

A következő ábrán látható három ügyfelek: [PowerShell](/powershell/azure/overview), a [az Azure portal](https://portal.azure.com), és a [Azure parancssori felület (CLI)](/cli/azure):

![](../_images/governance-1-13.png)
*5. ábra. A Resource Manager RESTful API az Azure-ügyfelek csatlakoznak.*

Bár ezek az ügyfelek csatlakozni a Resource Managernek a RESTful API-val, erőforrás-kezelő nem tartalmaz közvetlenül kezelheti az erőforrásokat funkciót. Ehelyett a legtöbb erőforrástípusok az Azure-ban megvan a saját [ **erőforrás-szolgáltató**](/azure/azure-resource-manager/resource-group-overview#terminology).

![](../_images/governance-1-14.png)
*6. ábra. Az Azure erőforrás-szolgáltatók.*

Amikor egy ügyfél kérést küld egy adott erőforrás kezeléséhez, a Resource Manager csatlakozik az erőforrás-szolgáltató az erőforrás típusát, a kérés teljesítéséhez. Például ha egy ügyfél kérést küld egy virtuális gép típusú erőforrást kezeléséhez, erőforrás-kezelő csatlakozik a **Microsoft.Compute** erőforrás-szolgáltató.

![](../_images/governance-1-15.png)
*7. ábra. Erőforrás-kezelő csatlakozik a **Microsoft.Compute** kezeléséhez az ügyfél kérésében megadott erőforrás erőforrás-szolgáltató.*

Resource Manager megköveteli, hogy adja meg az előfizetés és az erőforráscsoport azonosítóját ahhoz, hogy a virtuális gép típusú erőforrást kezelése az ügyfél.

Most, hogy hogyan működik a Resource Manager megértését, nézzük térjen vissza a hozzászólás, milyen Azure-előfizetéssel társítva az Azure Resource Manager által használt vezérlőkkel. Mielőtt bármely erőforrás felügyeleti kérelem Resource Manager által hajtható végre, a vezérlők be van jelölve.

Az első vezérlőelem, hogy a kérelmet kell kezdeményeznie, ha igazolt felhasználója, és a Resource Manager megbízható kapcsolattal rendelkezik [Azure Active Directory](/azure/active-directory/) (Azure AD) felhasználói identitáskezelési funkciókat biztosít.

![](../_images/governance-1-16.png)
*8. ábra. Azure Active Directory.*

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

A végső vezérlő annak ellenőrzése, hogy a vonatkozó kérés, a pénzügyi kötelezettségvállalás, az előfizetéshez tartozó belül. Például ha a kérelem egy virtuális gép üzembe helyezése, Resource Manager ellenőrzi, hogy az előfizetés rendelkezik elegendő fizetési adatait.

![](../_images/governance-1-21.png)
*13. ábra. A pénzügyi kötelezettségvállalás nélkül kapcsolódik előfizetés.*

## <a name="summary"></a>Összegzés

Ebből a cikkből megismerhette erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban Azure Resource Manager használatával.

## <a name="next-steps"></a>További lépések

Most, hogy már tudja, hogyan erőforrás-hozzáférés felügyelt az Azure-ban, helyezze át a megtudhatja, hogyan tervezhet egy cégirányítási modellt.

> [!div class="nextstepaction"]
> [Felhő-irányítás](../governance/overview.md)
