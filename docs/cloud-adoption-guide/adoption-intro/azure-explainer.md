---
title: 'Explainer: Azure működése'
description: Azure belső működését ismerteti
author: petertay
ms.openlocfilehash: 88514ba01efabe38a1b92d135321c2e44cef004e
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206463"
---
# <a name="explainer-how-does-azure-work"></a>Explainer: Azure működése

Azure a Microsoft nyilvános felhő platformja. Az Azure kínál, többek közt a platform (PaaS), infrastruktúrák (IaaS), egy szolgáltatás, és számos más adatbázis szolgáltatás nagy gyűjteménye. De mi pontosan Azure, és hogyan működik?

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo] 

Azure, például más felhőplatformokkal támaszkodik a technológiát néven **virtualizálási**. A legtöbb számítógép hardvere emulálni a szoftvert, mert a legtöbb számítógép hardvere olyan egyszerűen véglegesen vagy pontosvesszővel véglegesen kódolt szilícium utasításokat. Az emuláció réteg, amely leképezhető szoftver utasításokat hardver utasítások használatával, virtualizált hardverrel is végre lehet hajtani szoftver, mintha a tényleges hardver.

A felhő alapvetően, egy vagy több olyan adatközpontokban, amelyek végrehajtása a virtualizált hardverrel ügyfeleik nevében fizikai kiszolgálók egy csoportja. Igen, hogyan nem a felhő létrehozása, indítása, leállítása, és egyidejűleg törlése virtualizált hardverrel példánya több millió felhasználók millióit?

Ennek megértéséhez vizsgáljuk meg a hardver az adatközpontban architektúrájának.  Minden adatközponton belül olyan óta várakozik a kiszolgáló rackszekrények kiszolgálók gyűjteménye. Minden kiszolgáló állványban található több kiszolgáló **paneleken** továbbá egy hálózati kapcsoló hálózati kapcsolat és egy terjesztési teljesítménye (PDU) power biztosítása. Például rackszekrények néha sorolhatók néven nagyobb egységekre **fürtök**. 

Minden állványt vagy fürt a kiszolgálók többsége vannak kijelölve, a felhasználó nevében a virtualizált hardverrel példányán fusson. Azonban a kiszolgálók számos futtatni a felhő háló tartományvezérlőként ismert felügyeleti szoftver. A **fabric controller** egy elosztott alkalmazás számos feladatokkal. Azt foglal le, szolgáltatások, a kiszolgáló és a rajta futó szolgáltatások állapotát követi és kiszolgálók heals, ha azt elmulasztják.

A fabric controller minden példánya csatlakozik-e egy másik készlet felhő vezénylési szoftvert, általában néven futtató kiszolgálók egy **előtér**. Az előtér üzemelteti a webes szolgáltatások RESTful API-k és a felhő végez összes funkciójának használt belső Azure adatbázisok. 

Így például az előtér tárolja a szolgáltatásokat, amelyeket az ügyfél-tanúsítványigénylések Azure-erőforrásokat lefoglalni [virtuális hálózatok][vnet], [virtuális gépek] [ vms], például a szolgáltatások és [Cosmos DB][cosmosdb]. Az előtér először ellenőrzi a felhasználó, és ellenőrzi, hogy a felhasználó jogosult-e a kért erőforrásokat. Ha igen, az előtér egy kiszolgálószekrény található elegendő kapacitással rendelkező adatbázis olvas, és majd arra utasítja a fabric controller az állványra lefoglalni az erőforrás.

Igen nagyon egyszerűen Azure gyűjteménye túl nagy a kiszolgálók és a hálózati hardver, valamint álló összetett készlettel, amely a konfigurációs levezényelni elosztott alkalmazások és az ezeken a kiszolgálókon a virtualizált hardver- és működését. És így hatékony Azure teszi lehetővé a vezénylési – felhasználó már nem felelős és frissítése hardver, Azure összes ezt a háttérben végzi. 

## <a name="next-steps"></a>További lépések

* Most, hogy megismerte az Azure belső működésére, megismerése [erőforrás hozzáférés-irányítási](governance-explainer.md). Majd lépjen az első Azure, ami történő elfogadásával [megismerése az Azure-ban digitális identitásokat](tenant-explainer.md). A lépés befejezése után készen áll [az első felhasználó létrehozása az Azure AD][docs-add-users-to-aad].

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
