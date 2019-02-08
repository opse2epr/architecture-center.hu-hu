---
title: 'CAF: Szoftveralapú hálózatok - felhő szegélyhálózat (DMZ)'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A hálózati architektúra lehetővé teszi, hogy korlátozott hozzáférés a helyszíni és felhőalapú hálózatok között
author: rotycenh
ms.openlocfilehash: a192541dcfb0f3d713f4139a2ab0541d0c7202db
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899218"
---
# <a name="software-defined-networks-cloud-dmz"></a>A szoftveralapú hálózatok: Szegélyhálózat (DMZ) felhő

A felhő DMZ hálózati architektúra lehetővé teszi, hogy korlátozott hozzáférés a helyszíni és felhőalapú hálózatok, virtuális magánhálózat (VPN) használatával való csatlakozáshoz a hálózatok között. DMZ helyezünk üzembe a felhőben való biztonságos hozzáférést a helyszíni hálózathoz a felhőbeli erőforrásokat.

![Biztonságos hibrid hálózati architektúra](../../../reference-architectures/dmz/images/dmz-private.png)

Ez az architektúra úgy tervezték, hogy ahol a szervezet által támogatni kívánt indítsa el a számítási feladatok felhőbeli integrálása a helyszíni számítási feladatok esetében, de előfordulhat, hogy nincs teljesen érett felhőbeli biztonsági házirendek vagy beszerzett egy biztonságos, dedikált WAN kapcsolat között a két környezet. Felhőalapú hálózatokhoz, például egy demilitarizált zóna annak biztosítása érdekében, a helyszíni szolgáltatások biztonságos kell kezelni.

A DMZ hálózati virtuális berendezések (nva-k) a különböző biztonsági funkciókat, például a tűzfalakat és csomagvizsgálatot helyez üzembe. A helyszíni és felhőalapú alkalmazások vagy szolgáltatások közötti forgalom a DMZ-n keresztül kell átadni, ahol naplózható. VPN-kapcsolatok és a szabályok, amely meghatározza, hogy milyen adatforgalom haladjon át a DMZ-hálózatokban szigorúan szabályozza IT-biztonsági csoportokat.

## <a name="cloud-dmz-assumptions"></a>Szegélyhálózat (DMZ) felhő feltételezések

Üzembe helyezése Felhőbeli DMZ feltételezi, hogy a következő:

- A biztonsági csoportok nem rendelkezik teljes igazodik, a helyszíni és felhőalapú biztonsági követelmények és szabályzatok.
- A felhőalapú számítási feladatokat a helyszíni vagy külső hálózatokon lévő üzemeltetett szolgáltatások korlátozott hozzáférést igényelnek, vagy a felhőben futó erőforrások korlátozott hozzáférésre van szükségük a felhasználók vagy alkalmazások a helyszíni környezetben.
- Vállalati házirend, szabályozási követelmények vagy technikai kompatibilitási problémák nem akadályozza a végrehajtási egy VPN-kapcsolatot a helyszíni hálózatai és a felhőszolgáltató között.
- A számítási feladatok vagy nem igényelnek előfizetés erőforrás-korlátozások megkerülésére több előfizetést, vagy azok több előfizetés között, de nem igénylik a hálózati kapcsolat a központi felügyeleti vagy megosztott erőforrásokat több elosztva által használt szolgáltatások az előfizetések.

A felhőre való áttérés csapat kell figyelembe vennie a következő problémák, ha megnézi a felhő szegélyhálózat (DMZ) virtuális hálózati architektúra megvalósítása:

- A helyszíni hálózat csatlakoztatása felhőbeli hálózatok bonyolítják a a biztonsági követelményeknek. Annak ellenére, hogy biztosított felhőalapú hálózatokhoz és a helyszíni környezet közötti kapcsolat, továbbra is szeretné biztosítani a felhőbeli erőforrások vannak biztosítva.
- A felhő szegélyhálózat (DMZ) az architektúrát gyakran használják a ke krokování kő kapcsolat további pedig a védett és a helyszíni és a felhő között igazítva biztonsági házirend hálózatokat, így egy teljes körű hibrid hálózati architektúra szélesebb körű elfogadását.

## <a name="learn-more"></a>Részletek

Tekintse meg a következő további információt a végrehajtási felhőalapú DMZ az Azure platformon.

- [Az Azure és a helyszíni adatközpont közötti DMZ implementálása](../../../reference-architectures/dmz/secure-vnet-hybrid.md). Ez a cikk ismerteti egy biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
