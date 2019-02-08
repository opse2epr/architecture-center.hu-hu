---
title: 'CAF: Szoftver szoftveralapú hálózatok – natív felhőalapú'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Felhőbeli natív virtuális hálózati szolgáltatásokat vitafórum
author: rotycenh
ms.openlocfilehash: c6200491bc9ba35a9f00e0003e51716b58628980
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898737"
---
# <a name="software-defined-networks-cloud-native"></a>A szoftveralapú hálózatok: Natív felhőalapú

Egy felhőbeli natív virtuális hálózati IaaS-erőforrások, például a virtuális gépek üzembe helyezése egy felhőalapú platform esetén szükséges. Virtuális hálózati hozzáférés a webes hasonló külső forrásból kell explicit módon ki kell építeni. Virtuális hálózatok ilyen típusú támogatja az alhálózatok, az útválasztási szabályokat és a virtuális tűzfal és a forgalom felügyeleti eszközök létrehozását.

Egy felhőbeli natív virtuális hálózati egy csoporttól sem a szervezet helyszíni vagy a felhőben futó számítási feladatok a felhőbe irányuló erőforrását. Az összes szükséges erőforrások kiépítése magát a virtuális hálózatban vagy felügyelt PaaS-ajánlatok használatával.

## <a name="cloud-native-assumptions"></a>Felhőbeli natív feltételezések

Egy felhőbeli natív virtuális hálózat üzembe helyezése feltételezi, hogy a következő:

- A számítási feladatok, a virtuális hálózaton üzembe helyezni kívánt alkalmazások és szolgáltatások elérhető csak a helyszíni hálózaton belüli nincs függőségekkel rendelkeznek. Kivéve, ha a nyilvános interneten, alkalmazások és szolgáltatások belsőleg elérhető végpontok biztosítanak a helyszíni számára nem érhetők el egy felhőalapú platformon futtatott erőforrásokkal.
- A számítási feladatok identitás és hozzáférés-vezérlési attól függ, a cloud platform identitás-szolgáltatások vagy a felhőalapú környezetében üzemeltetett IaaS-kiszolgálók. Nem kell közvetlenül csatlakozhat identitás-ban üzemeltetett szolgáltatások a helyszínen vagy más külső helyekről.
- Az identitás-szolgáltatások nem támogatja az egyszeri bejelentkezés (SSO) a helyszíni címtárak kell.

Felhőbeli natív virtuális hálózatok nem külső függőségekkel rendelkeznek. Így őket egyszerű üzembe helyezése és konfigurálása, és ennek eredményeképpen ezt az architektúrát gyakran kísérletek vagy egyéb kisebb önálló vagy gyorsan Léptetés a legjobb választás.

Egy felhőbeli natív virtuális hálózati architektúrát megvizsgálja a felhő bevezetésének csapat megfontolandó további hibák a következők:

- A számítási feladatokat egy helyszíni adatközpontban futtathatók kihasználásához felhőbeli funkciókat, például tárolás vagy hitelesítési szolgáltatások széles körű módosítása szükségessé.
- Felhőbeli natív hálózatok kizárólag a cloud platform felügyeleti eszközök kezelhető, és ezért névütközéseket okozhat, kezelése és házirend-eltérés a meglévő informatikai szabványok az idő előrehaladtával.

## <a name="learn-more"></a>Részletek

Tekintse meg a következő további információ a felhőbeli natív virtuális hálózat az Azure platformon.

- [Az Azure Virtual Network: Útmutatók](/azure/virtual-network/virtual-network-vnet-plan-design-arm). Az újonnan létrehozott Azure virtuális hálózatok natív alapértelmezés szerint. Ezek az útmutatók a segítségével tervezze meg a tervezési és központi telepítését a virtuális hálózatok.
- [Előfizetés korlátai: Hálózatkezelés](/azure/azure-subscription-service-limits?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Bármilyen egyetlen virtuális hálózat és a csatlakoztatott források csak akkor létezhet, egyetlen előfizetésben jön létre, és azok által előfizetési korlátozásait.
