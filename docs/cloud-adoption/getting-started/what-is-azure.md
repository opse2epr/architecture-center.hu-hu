---
title: 'CAF: Hogy működik az Azure?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
description: Az Azure belső működésére ismertetése
author: petertaylor9999
ms.date: 02/11/2019
ms.openlocfilehash: 215e4e4954a2f3e722e3ac865fea19508f6edadd
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068820"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-azure-work"></a>Hogy működik az Azure?

Az Azure a Microsoft nyilvános felhő platformja. Az Azure platformszolgáltatás (PaaS), szolgáltatott infrastruktúra (IaaS), adatbázis-szolgáltatás (DBaaS) képességeket, beleértve a szolgáltatások gyűjteménye kínál. De mi pontosan az Azure, és hogyan működik?

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo]

<!-- markdownlint-enable MD034 -->

Azure-ban, mint más felhőplatformokon támaszkodik egy technológia, más néven **virtualizálási**. A legtöbb számítógép hardvere emulálni a szoftver, mivel a legtöbb számítógép hardvere egyszerűen véglegesen vagy véglegesen pontosvesszővel titkosításúnak szilícium utasításokat egy készletét. Az emuláció réteg, amely leképezi a szoftver utasításokat utasításait a hardver használatával, virtualizált hardverrel is hajtsa végre a szoftvereket, mintha magára a hardverkövetelmények.

Alapvetően a felhőben, hajtsa végre a virtualizált hardverrel ügyfeleik nevében egy vagy több adatközpontokban fizikai kiszolgálók egy csoportja. Igen, hogyan nem a felhő létrehozása, indítása, leállítása és virtualizált hardverrel példánya több millió egyszerre törlése ügyfelek milliói számára?

Ennek megértéséhez Tekintsünk meg a hardver, az Adatközpont architektúráját.  Minden adatközponton belül visszamegyek a kiszolgáló állványt kiszolgálók gyűjteménye van. Minden kiszolgáló állványban található számos kiszolgáló **panelek** és a egy hálózati kapcsoló hálózati kapcsolatot és a egy power terjesztési egységek (PDU) biztosít a power. Nagyobb egységeket, más néven a állványt néha együtt vannak csoportosítva **fürtök**.

Minden állványt vagy fürt a kiszolgálók többsége vannak kijelölve, a felhasználó nevében a virtualizált hardverrel példányok futtatásához. Azonban az a kiszolgáló futtatása felhőalapú felügyeleti szoftverek a hálóvezérlő néven. A **hálóvezérlő** egy elosztott alkalmazás számos feladatokkal. Foglalja le a szolgáltatások, a kiszolgáló és a rajta futó szolgáltatások állapotát figyeli, és kijavítja a kiszolgálók, amikor meghiúsulnak.

Minden példánya a hálóvezérlő csatlakoztatva van egy másik hárompéldányos készletet felhőalapú vezénylési szoftvert, általában más néven futtató kiszolgálók egy **előtér**. Az előtér üzemelteti a webes szolgáltatásokat, RESTful API-k és a felhő hajtja végre a függvények használt belső Azure-adatbázisok.

Így például az előtér tárolja-e a szolgáltatásokat, amelyek az Azure-erőforrásokat lefoglalni a felhasználói kérések kezelésére [virtuális hálózatok](/azure/virtual-network/virtual-networks-overview), [virtuális gépek](/azure/virtual-machines), és a szolgáltatásokat, mint például [Cosmos DB](/azure/cosmos-db/introduction). Az előtér először ellenőrzi a felhasználó, és ellenőrzi a felhasználó jogosult-e a kért erőforrások lefoglalása. Ha igen, az előtér ellenőrzi az adatbázis keresse meg a kiszolgáló állvány-kapacitása elegendő, és majd arra utasítja a hálóvezérlő meg, hogy az erőforrás lefoglalásához rack.

Így alapvető Azure gyűjteménye hatalmas, kiszolgálók és a hálózatkezelési hardverek álló összetett készlettel, amellyel a konfiguráció az elosztott alkalmazások és a virtualizált hardver- és működésének futtatása ezeken a kiszolgálókon. Ennek vezénylését, amely az Azure ezért sokoldalúak &mdash; felhasználók nem lesznek felelős karbantartására és frissítésére hardver, mert az Azure összes ezt a háttérben hajtja végre.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett az Azure belső elemeinek, felhőalapú erőforrás-szabályozása ismerje.

> [!div class="nextstepaction"]
> [További tudnivalók az erőforrás-szabályozása](what-is-governance.md)

<!-- Links -->

[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
