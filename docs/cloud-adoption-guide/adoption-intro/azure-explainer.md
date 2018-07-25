---
title: Hogyan működik az Azure?
description: Az Azure belső működésére ismertetése
author: petertay
ms.openlocfilehash: bf301a05d69ed66aa03727dde3968477c2337790
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/24/2018
ms.locfileid: "39228966"
---
# <a name="how-does-azure-work"></a>Hogyan működik az Azure?

Az Azure a Microsoft nyilvános felhő platformja. Az Azure gyűjteménye, többek között platform (PaaS), szolgáltatott infrastruktúra (IaaS), adatbázis-szolgáltatás (DBaaS), és sok más szolgáltatás szolgáltatásokat kínál. De mi pontosan az Azure, és hogyan működik?

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo] 

Azure-ban, mint más felhőplatformokon támaszkodik egy technológia, más néven **virtualizálási**. A legtöbb számítógép hardvere emulálni a szoftver, mivel a legtöbb számítógép hardvere egyszerűen véglegesen vagy véglegesen pontosvesszővel titkosításúnak szilícium utasításokat egy készletét. Az emuláció réteg, amely leképezi a szoftver utasításokat utasításait a hardver használatával, virtualizált hardverrel is hajtsa végre a szoftvereket, mintha magára a hardverkövetelmények.

Alapvetően a felhőben, hajtsa végre a virtualizált hardverrel ügyfeleik nevében egy vagy több adatközpontokban fizikai kiszolgálók egy csoportja. Igen, hogyan nem a felhő létrehozása, indítása, leállítása és virtualizált hardverrel példánya több millió egyszerre törlése ügyfelek milliói számára?

Ennek megértéséhez Tekintsünk meg a hardver, az Adatközpont architektúráját.  Minden adatközpontban olyan visszamegyek a kiszolgáló állványt kiszolgálók gyűjteménye. Minden kiszolgáló állványban található számos kiszolgáló **panelek** és a egy hálózati kapcsoló hálózati kapcsolatot és a egy power terjesztési egységek (PDU) biztosít a power. Nagyobb egységeket, más néven a állványt néha együtt vannak csoportosítva **fürtök**. 

Minden állványt vagy fürt a kiszolgálók többsége vannak kijelölve, a felhasználó nevében a virtualizált hardverrel példányok futtatásához. Azonban a kiszolgálók számos felhőjét felügyeleti szoftverek a hálóvezérlő néven. A **hálóvezérlő** egy elosztott alkalmazás számos feladatokkal. Foglalja le a szolgáltatások, a kiszolgáló és a rajta futó szolgáltatások állapotát figyeli, és kijavítja a kiszolgálók, amikor meghiúsulnak.

Minden példánya a hálóvezérlő csatlakoztatva van egy másik hárompéldányos készletet felhőalapú vezénylési szoftvert, általában más néven futtató kiszolgálók egy **előtér**. Az előtér üzemelteti a webes szolgáltatásokat, RESTful API-k és a felhő hajtja végre a függvények használt belső Azure-adatbázisok. 

Így például az előtér tárolja-e a szolgáltatásokat, amelyek az Azure-erőforrásokat lefoglalni a felhasználói kérések kezelésére [virtuális hálózatok][vnet], [virtuális gépek] [ vms], és a szolgáltatásokat, mint például [Cosmos DB][cosmosdb]. Az előtér először ellenőrzi a felhasználó, és ellenőrzi a felhasználó jogosult-e a kért erőforrások lefoglalása. Ha igen, az előtér olvas, keresse meg a kiszolgáló állvány elegendő kapacitással az adatbázis, és majd arra utasítja a hálóvezérlő a az állványra szerelt, az erőforrás lefoglalásához.

Tehát nagyon egyszerűen Azure gyűjteménye hatalmas, kiszolgálók és a hálózati eszközt, valamint összetett elosztott alkalmazások, amelyekkel a konfiguráció és a művelet a virtualizált hardver- és az ezeken a kiszolgálókon. És ennek vezénylését, amely az Azure ezért sokoldalúak - felhasználók nem lesznek karbantartásáért felelős és frissítése hardver, az Azure összes ezt a háttérben hajtja végre. 

## <a name="next-steps"></a>További lépések

* Most, hogy megismerkedett az Azure belső működésére, ismerje [erőforrás-hozzáférés szabályozása](governance-explainer.md). 

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
