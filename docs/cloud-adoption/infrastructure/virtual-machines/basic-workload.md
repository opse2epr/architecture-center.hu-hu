---
title: 'CAF: egy alapvető számítási feladatok üzembe helyezése'
description: Ismerteti, hogyan helyezhet üzembe egy alapvető számítási feladatok Azure-bA
author: petertaylor9999
ms.date: 12/31/2018
ms.openlocfilehash: 15495f4306d816df5e54bb8a35819718791f6820
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298594"
---
# <a name="deploy-a-basic-workload-in-azure"></a>Az Azure-ban egy alapvető számítási feladatok üzembe helyezése

Az előfizetési időszak *munkaterhelés* általában funkciókat, például alkalmazások vagy szolgáltatások egy tetszőleges egysége típusúként van definiálva. Segít a kell foglalkoznia a számítási feladat tekintetében a kód összetevőket, amelyek egy kiszolgáló, valamint egyéb alkalmazásspecifikus szolgáltatások vannak telepítve. Ez akkor lehet hasznos definícióját egy helyszíni alkalmazást vagy szolgáltatást, de ki kell bővíteni kell a felhőalapú alkalmazásokhoz.

A felhőbeli számítási feladatok nem csupán magában foglalja a minden összetevő, de is tartalmaz a felhőbeli erőforrásokat is. Tartalmazza a felhőbeli erőforrások részeként az meghatározás miatt van más néven "infrastruktúra mint kód" fogalma. Amint már tudja, a [Azure működése?](../../getting-started/what-is-azure.md), hogy az orchestrator szolgáltatás által üzembe helyezett erőforrások az Azure-ban. Ez a szolgáltatás az orchestrator webes API-k segítségével funkciókkal rendelkezik, és meghívhatja a webes API-t több eszközök, például az Azure portal, PowerShell és az Azure parancssori felület (CLI) használatával. Ez azt jelenti, hogy megadhat egy gépi olvasásra alkalmas fájlban tárolt együtt a kód összetevőket az alkalmazáshoz kapcsolódó Azure-erőforrások.

Ez lehetővé teszi, hogy meghatározza a munkaterhelés kódja összetevők tekintetében, és a szükséges felhőbeli erőforrásokat, így további lehetővé téve a számítási feladatok elkülönítésére. Az erőforrások rendszerezésének módja, hálózati topológia, vagy egyéb tulajdonságok szerinti különítheti el számítási feladatokhoz. A számítási feladatok elkülönítése célja, hogy egy csapat a számítási feladat erőforrásoknál társítani, hogy a csapat egymástól függetlenül kezelheti ezeket az erőforrásokat minden aspektusát. Ez lehetővé teszi több a csapatok megoszthatnak erőforrás-kezelési szolgáltatások az Azure-ban megakadályozza, hogy az a véletlen törlés vagy módosítására a többi összes erőforrást.

Ez az elkülönítés is lehetővé teszi egy másik fogalom, mint DevOps. Fejlesztési és üzemeltetési magában foglalja a szoftverek fejlesztési és operatív informatikai is tartalmazó szoftverfejlesztési gyakorlatban, és hozzáadja a lehető legnagyobb mértékben automatizálást használni. A fejlesztési és üzemeltetési alapelveit egyik folyamatos integrációt és folyamatos készregyártás (CI/CD) néven ismert. Folyamatos integráció, amely minden alkalommal, amikor egy fejlesztő véglegesítések kódváltoztatást futnak automatizált összeállítási folyamatokat foglalja magában. Folyamatos készregyártás, hogy ez a kód üzembe helyezni különböző környezetekben, például egy tesztelési-fejlesztési környezetre vagy végleges üzembe helyezés éles környezetben automatizált folyamatokat foglalja magában.

## <a name="basic-workload"></a>Alapvető számítási feladatok

A *alapvető számítási feladatok* általában egy egyetlen webalkalmazás vagy virtuális gép (VM) rendelkező virtuális hálózaton (VNet) típusúként van definiálva.

> [!NOTE]
> Ez az útmutató nem tárgyalja alkalmazások fejlesztését. Az Azure-alkalmazások fejlesztésével kapcsolatos további információkért lásd: a [útmutató az Azure Alkalmazásarchitektúrájához](/azure/architecture/guide/).

Függetlenül attól, hogy a számítási feladatok egy webalkalmazás vagy egy virtuális Gépet, ezeket az üzemelő példányokat, szükséges egy *erőforráscsoport*. Egy erőforráscsoport létrehozásához szükséges engedéllyel rendelkező felhasználó el kell végeznie az alábbi lépések elvégzése előtt.

## <a name="basic-web-application-paas"></a>Alapszintű webalkalmazás (PaaS)

Alapszintű webalkalmazás, válassza az ötperces gyors ismertetőkkel az egyik a [web apps-dokumentáció](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) , és kövesse a lépéseket.

> [!NOTE]
> Alapértelmezés szerint a gyors útmutatók némelyike fogja telepíteni egy erőforráscsoportot. Ebben az esetben azt nem kell explicit módon hozzon létre egy erőforráscsoportot. Ellenkező esetben üzembe helyezése a webalkalmazás a fent létrehozott erőforráscsoportot.

Miután telepít egy egyszerű számítási feladat, többet is megtudhat telepítéséhez a bevált gyakorlatokat kapcsolatos egy [alapszintű webalkalmazás](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) az Azure-bA.

## <a name="single-windows-or-linux-vm-iaas"></a>Egyetlen Windows vagy Linux rendszerű virtuális gépek (IaaS)

Egy egyszerű számítási feladat, amely a virtuális gép fut az első lépés, hogy a virtuális hálózat üzembe helyezése. Az összes infrastruktúra-szolgáltatás (IaaS) az Azure-erőforrások virtuális gépeket, terheléselosztókat és átjárókat, például egy virtuális hálózat szükséges. Ismerje meg [Azure virtuális hálózatok](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), majd kövesse a lépéseket [üzembe helyezése egy virtuális hálózatot az Azure-ban a portál](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Az Azure Portalon a virtuális hálózatra vonatkozó beállításainak megadásakor ügyeljen arra, adja meg a fent létrehozott erőforráscsoport nevét.

A következő lépés, hogy egyetlen Windows vagy Linux rendszerű virtuális gép üzembe kell-e. Windows virtuális gép, kövesse a lépéseket a [Windows virtuális gép üzembe helyezése az Azure Portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Újra amikor az Azure Portalon adja meg a virtuális gép beállításait, adja meg a fent létrehozott erőforráscsoport nevét.

Miután követte a lépéseket, és a virtuális Gépet üzembe helyezett, megismerkedhet a [bevált eljárások az Azure-on futó Windows virtuális gép](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). Linux rendszerű virtuális gép esetén kövesse a lépéseket a [Linux virtuális gép üzembe helyezése az Azure-bA a portállal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Hogy még többet is megtudhat [bevált eljárások az Azure-on futó Linux rendszerű virtuális gép](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).

## <a name="next-steps"></a>További lépések

Lásd: [architektúrával kapcsolatos döntés útmutatók](../../decision-guides/overview.md) az alapvető infrastruktúra összetevői használata az Azure-felhőben.
