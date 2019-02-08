---
title: 'CAF: Szoftver szoftveralapú hálózatok – hibrid hálózat'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Hogyan hibrid hálózat lehetővé teszi a felhőbeli virtuális hálózatot a helyszíni erőforrásokhoz történő csatlakozáshoz veszik górcső alá
author: rotycenh
ms.openlocfilehash: 02d181db0ae9baef3b453b8623d212b624f6b16a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899031"
---
# <a name="software-defined-networks-hybrid-network"></a>A szoftveralapú hálózatok: Hibrid hálózat

A hibrid hálózati architektúra lehetővé teszi, hogy a virtuális hálózatok szolgáltatások futtatását, WAN-kapcsolattal egy dedikált például az ExpressRoute- vagy egyéb kapcsolati módszer közvetlenül kapcsolódni a hálózatokhoz és a helyszíni erőforrásokhoz való hozzáférését.

![Hibrid hálózat](../../../reference-architectures/hybrid-networking/images/expressroute.png)

A felhőbeli natív virtuális hálózati architektúrát építve, hibrid virtuális hálózat el különítve létrehozásakor. Kapcsolat hozzáadása a helyszíni környezet hozzáférést biztosít, és a helyszíni hálózatról, bár minden egyéb bejövő forgalmat a virtuális hálózat célcsoport-kezelési erőforrások explicit módon kell engedélyezni kell. Biztosíthatja a kapcsolatot a virtuális tűzfal-eszközhöz használ, és útválasztási szabályok a hozzáférés korlátozásához, vagy megadhatja, hogy pontosan milyen szolgáltatásokat lehet használatával natív felhőalapú szolgáltatások útválasztást vagy a hálózati virtuális berendezések (nva-k) telepítése a két hálózat között érhető el forgalom kezelésére.

Bár a hibrid hálózati architektúra támogatja a VPN-kapcsolatok, például az ExpressRoute dedikált WAN-kapcsolaton használatosak általában nagyobb teljesítményt és nagyobb biztonság miatt.

## <a name="hybrid-assumptions"></a>Hibrid feltételezések

Hibrid virtuális hálózat üzembe helyezése feltételezi, hogy a következő:

- Az IT-biztonsági csoportoknak van igazítva a helyszíni és közvetlenül a helyszíni rendszerekkel való közölni megbízható felhőalapú hálózati biztonsági házirend annak biztosítása érdekében a cloud-alapú virtuális hálózatokat.
- A felhőalapú számítási feladatok hozzáférést igényelnek a storage, alkalmazások és a helyszíni vagy külső hálózatokon lévő üzemeltetett szolgáltatás, vagy a felhasználók vagy a helyszíni alkalmazások felhőben üzemeltetett erőforrásokhoz való hozzáférésre van szükségük.
- Szeretné áttelepíteni a meglévő alkalmazások és szolgáltatások, amelyek a helyszíni erőforrások függenek, de nem szeretne az erőforrások átalakítását eltávolítja ezeket a függőségeket a kiadás.
- Vállalati házirend, szabályozási követelmények vagy technikai kompatibilitási problémák nem akadályozza a megvalósítása a VPN-t vagy dedikált WAN-kapcsolat a helyszíni hálózatai és a felhőszolgáltató között.
- A számítási feladatok vagy nem igényelnek előfizetés erőforrás-korlátozások megkerülésére több előfizetést, vagy a számítási feladatok több előfizetést is jár, de nem igénylik a hálózati kapcsolat a központi felügyeleti vagy megosztott erőforrások közötti által használt szolgáltatások több előfizetés.

A felhőre való áttérés csapat kell figyelembe vennie az alábbi problémákat, ha megnézi a virtuális hibrid hálózati architektúra megvalósítása:

- A helyszíni hálózat csatlakoztatása felhőbeli hálózatok bonyolítják a a biztonsági követelményeknek. Mindkét hálózatot kell használnia az védeni kell külső biztonsági réseket és a hibrid környezet mindkét oldalról jogosulatlan hozzáférés ellen.
- A számát és méretét, egy hibrid felhőalapú környezetben futó alkalmazások és szolgáltatások méretezése is összetettebbé jelentős útválasztást és a forgalom kezelésére.
- Szüksége lesz kompatibilis felügyeleti fejlesztése és hozzáférés-vezérlési házirendeket a szervezeten belül konzisztens cégirányítási fenntartásához.

## <a name="learn-more"></a>Részletek

Hibrid hálózatkezelés az Azure platformon további információt a következő témakörben találhat.

- [Hibrid hálózatok referenciaarchitektúráját](../../../reference-architectures/hybrid-networking/expressroute.md). Virtuális hálózatok az Azure hybrid használja vagy ExpressRoute-kapcsolatcsoport, vagy a szervezet meglévő nem Azure-beli kapcsolódni a virtuális hálózat az Azure VPN üzemeltetett IT-eszközeivel. Ez a cikk ismerteti a lehetőségeket hibrid hálózat létrehozásakor az Azure-ban.
