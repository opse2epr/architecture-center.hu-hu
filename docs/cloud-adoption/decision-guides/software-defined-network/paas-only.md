---
title: 'CAF: Szoftveralapú hálózatok – PaaS csak'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A PaaS-felhő csak modell veszik górcső alá alapú hálózatkezelési funkció
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299417"
---
# <a name="software-defined-networks-paas-only"></a>A szoftveralapú hálózatok: Csak PaaS

Platformszolgáltatás (PaaS) erőforrás platformként megvalósításának, a telepítési folyamat automatikusan hálózatot hoz létre egy feltételezett alapul szolgáló vezérlők a hálózaton, többek között a terheléselosztást, blokkolja a port és egyéb PaaS kapcsolatokat csak korlátozott számú szolgáltatások.

Az Azure-ban, a PaaS-erőforrás számos különböző lehet [üzembe helyezett](/azure/virtual-network/virtual-network-for-azure-services) vagy [csatlakozik](/azure/virtual-network/virtual-network-service-endpoints-overview) egy virtuális hálózatot, lehetővé téve az ezekhez az erőforrásokhoz való integrációhoz a meglévő virtuális hálózati infrastruktúra. Azonban a sok esetben ezek hálózati natív módon a PaaS-erőforrásokhoz, által biztosított lehetőségeket alapértelmezés támaszkodva csak hálózati architektúrát, PaaS, elegendő számítási feladat igényeinek.

Ha egy PaaS, csak a hálózati architektúra használatát fontolgatja, mindenképpen ellenőrizze, hogy a szükséges előfeltételek összhangba kerüljenek a követelmények.

## <a name="paas-only-assumptions"></a>Csak PaaS feltételezések

A PaaS-csak hálózati architektúra üzembe helyezéséhez feltételezi, hogy a következő:

- Az alkalmazás üzembe helyezéséhez egy különálló alkalmazás, vagy egyéb PaaS-erőforrásokhoz függ.
- Az IT-üzemeltetési csapatokra frissítheti az eszközök, tanfolyamok és folyamatokat a felügyeleti, konfigurációs és központi telepítését a különálló PaaS-alkalmazások.
- A PaaS-alkalmazás nem szerepel egy szélesebb körű felhőalapú áttelepítési erőfeszítés, amely IaaS-erőforrásokat tartalmazza.

Ilyen Előfeltevések igazítva, és üzembe helyezését egy PaaS-csak hálózati minimális minősítők. Amíg ez a megközelítés lehetséges, hogy összhangba kerüljenek egy egyetlen alkalmazás telepítési követelményeinek, a felhő bevezetésének csapat vizsgálja meg ezeket a hosszú távú kérdéseket:

- A központi telepítés bővített a hatókörben vagy egyéb nem PaaS-erőforrásokhoz való hozzáférést igényelnek a méretezési csoport?
- Egyéb PaaS üzemelő példányok tervezett túl az aktuális megoldáshoz?
- Rendelkezik a szervezet más a migrálás a felhőbe jövőbeli terveket?

Az ezekre a kérdésekre adott válaszokat lenne nem zárja ki, hogy egy csapat egy PaaS egyetlen művelettel, de értendők, mielőtt az utolsó.
