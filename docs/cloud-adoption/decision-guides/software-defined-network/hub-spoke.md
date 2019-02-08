---
title: 'CAF: Szoftver szoftveralapú hálózatok – natív felhőalapú'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Felhőbeli natív virtuális hálózati szolgáltatásokat vitafórum
author: rotycenh
ms.openlocfilehash: e0ad6803f2ddc982ea0c42c59fdf2486e1710433
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899170"
---
# <a name="software-defined-networks-hub-and-spoke"></a>A szoftveralapú hálózatok: Küllős topológiájú

A küllős hálózati modell rendezi az Azure-alapú felhőalapú hálózati infrastruktúra több csatlakoztatott virtuális hálózatokra. Ez a modell lehetővé teszi, hogy hatékonyabban kezelheti közös kommunikáció vagy biztonsági követelmények, és rejtőző lehetséges előfizetés korlátai.

A küllős modellben a *hub* egy virtuális hálózat, amely egy központi helyen kezeléséhez külső kapcsolatokat, és több számítási feladat által használt szolgáltatásokat üzemeltető funkcionál. A *küllők* , amely a számítási feladatokat üzemeltethet, és a központi agyhoz keresztül csatlakozhat virtuális hálózatok [virtuális hálózatok közötti társviszony](/virtual-network/virtual-network-peering-overview).

A számítási feladatok topológiájú hálózatokat vagy az összes forgalom a hub hálózat, ahol azt, irányítva, ellenőrzött vagy más módon kezeli a központilag felügyelt IT-szabályok és folyamatok továbbít.

Ez a modell az a célja, hogy az alábbi problémákat:

- Költségkímélés és felügyeleti hatékonyságát. Szolgáltatások, amelyek megoszthatók több számítási feladat, például a hálózati virtuális berendezések (nva-k) és a DNS-kiszolgálók, egyetlen helyen központosítása lehetővé teszi, hogy minimalizálja a redundáns információforrások és energiabefektetést igénylő felügyelet több számítási feladatok.
- Előfizetési korlátok leküzdése. Nagy méretű felhőalapú számítási feladatokhoz szükség lehet egy egyetlen Azure-előfizetésben engedélyezettnél több erőforrások használatával (lásd: [előfizetési korlátozásait](/azure/azure-subscription-service-limits)). Társviszony-létesítési munkaterhelés virtuális hálózatok különböző előfizetésekről csatlakoztat egy központi agyhoz-szel kiküszöbölhetők a ezeket a korlátokat.
- Kockázatok elkülönítése. Lehetővé teszi a központi IT-csoportoknak, és a számítási feladatok csapatok között az egyes számítási feladatok üzembe helyezéséhez.

Az alábbi ábrán látható egy példa hubot, és topológiájú architektúrák, beleértve a központilag felügyelt hibrid kapcsolatokat.

![Küllős hálózati architektúra](../../../reference-architectures/hybrid-networking/images/hub-spoke.png)

A küllős topológiájú architektúránk gyakran használják együtt a hibrid hálózati architektúra, a helyszíni környezet több számítási feladatok között megosztott központilag felügyelt kapcsolatot biztosító. Ebben a forgatókönyvben minden forgalom a számítási feladatok és a helyszíni között továbbított áthalad a hubhoz, ha felügyelt és biztonságos.

## <a name="hub-and-spoke-assumptions"></a>Küllős topológiájú feltételezések

Küllős topológiájú virtuális hálózati architektúra megvalósítása feltételezi, hogy a következő:

- A felhőalapú környezetekben magában foglalja például fejlesztési, tesztelési és éles környezetben, hogy az összes támaszkodjon közös szolgáltatásokat, például a DNS vagy a directory services külön működő környezetben futó számítási feladatokat.
- A számítási feladatok nem kell kommunikálnak egymással, de a gyakori külső kommunikációs rendelkezik, és a megosztott szolgáltatások követelményei.
- A munkaterhelések számára szükséges további erőforrásokat, mint amennyi rendelkezésre áll egy egyetlen Azure-előfizetésen belül.
- Meg kell adnia munkaterhelés csapatok delegált felügyeleti jogosultságokkal a saját erőforrások folyamatos kívülről központi biztonsági ellenőrzése.

## <a name="global-hub-and-spoke"></a>Globális küllős

Küllős topológiájú architektúrák gyakran a virtual Network hálózatok közötti késés minimalizálása érdekében azonos Azure-régióban üzembe helyezett vannak megvalósítva. Azonban nagy szervezeteknek globális elérhetőségű szükségessé számítási feladatot helyezhet üzembe több régióban rendelkezésre állás, a vész-helyreállítási vagy a szabályozási követelményeknek. Az Azure átívelő [globális virtuális társhálózatok](/azure/virtual-network/virtual-network-peering-overview), a küllős modell terjeszthetők ki központi felügyeletet és a megosztott szolgáltatások számítási feladatok, a világ különböző pontjain lévő régióban.

## <a name="learn-more"></a>Részletek

Küllős topológiájú hálózatokat megvalósítása az Azure-ban példákért lásd az alábbi példák az Azure-Referenciaarchitektúrák webhelyen:

- [Küllős hálózati topológia implementálása az Azure-ban](../../../reference-architectures/hybrid-networking/hub-spoke.md)
- [Közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban](../../../reference-architectures/hybrid-networking/shared-services.md)
