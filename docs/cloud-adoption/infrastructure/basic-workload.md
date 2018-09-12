---
title: 'Vállalati felhő bevezetésének: egy alapvető számítási feladatok üzembe helyezése'
description: Ismerteti, hogyan helyezhet üzembe egy alapvető számítási feladatok Azure-bA
author: petertaylor9999
ms.date: 09/10/2018
ms.openlocfilehash: e615ba33fb713278a3695057e61d99c92b72b3f2
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389313"
---
# <a name="enterprise-cloud-adoption-deploy-a-basic-workload"></a>Enterprise Cloud Adoption: egy alapvető számítási feladatok üzembe helyezése

Az előfizetési időszak **munkaterhelés** általában érthető funkciók, például alkalmazások vagy szolgáltatások egy tetszőleges egysége meghatározásához. Azt gondolja végig egy számítási feladat tekintetében a kód-összetevők a kiszolgálóra telepített, hanem a szükséges szolgáltatások tekintetében. Ez hasznos definícióját egy helyszíni alkalmazást vagy szolgáltatást, de a felhőben, bontsa ki kell.

A felhőben a munkaterhelés nem csak magában foglalja a minden összetevő, de is tartalmaz a felhőbeli erőforrásokat is. A fogalom, az infrastruktúra mint kód miatt a definíció részeként tartalmazza a felhőbeli erőforrások. Amint már tudja, az a [Azure működése?](../getting-started/what-is-azure.md), hogy az orchestrator szolgáltatás által üzembe helyezett erőforrások az Azure-ban. Az orchestrator szolgáltatás elérhetővé teszi ezt a funkciót, a webes API-k és a webes API nem hívható meg több eszközök, például az Azure portal, Powershell és az Azure parancssori felület (CLI) használatával. Ez azt jelenti, hogy adható meg az erőforrásokat a kód összetevőket az alkalmazáshoz társított együtt tárolt gépi olvasásra alkalmas fájlban.

Ez lehetővé teszi számunkra, hogy meghatározása egy számítási feladat tekintetében kód összetevők és a szükséges felhőbeli erőforrásokat, és a további lehetővé teszi számunkra, hogy a számítási feladatok elkülönítésére. Számítási feladatok lehet elkülönített egyébként erőforrások vannak rendszerezve, hálózati topológia, vagy egyéb attribútumokkal. A számítási feladatok elkülönítése célja, hogy egy csapat a számítási feladat erőforrásoknál társítani, a csapat egymástól függetlenül kezelheti ezeket az erőforrásokat minden aspektusát. Ez lehetővé teszi több a csapatok megoszthatnak erőforrás-kezelési szolgáltatások az Azure-ban megakadályozza, hogy az a véletlen törlés vagy módosítására a többi összes erőforrást.

Ez az elkülönítés is lehetővé teszi a DevOps más néven egy másik fogalom. Fejlesztési és üzemeltetési a szoftverfejlesztési gyakorlatban, amely tartalmazza a szoftverek fejlesztési és operatív informatikai is tartalmaz, de hozzáadja a lehető legnagyobb mértékben automatizálást használni. A fejlesztési és üzemeltetési alapelveit egyik folyamatos integrációt és folyamatos készregyártás (CI/CD) néven ismert. Folyamatos integráció hivatkozik az automatizált összeállítási folyamatairól minden alkalommal, amikor egy fejlesztő véglegesítések kódváltoztatást futnak, és folyamatos készregyártás, hogy ez a kód üzembe helyezni különböző környezetekben, például fejlesztési környezetének automatizált folyamatokat foglalja magában tesztelési vagy éles környezetben a végleges üzembe helyezés.

## <a name="basic-workload"></a>Alapvető számítási feladatok

A **alapvető számítási feladatok** általában egyetlen webalkalmazás vagy virtuális gép (VM) rendelkező virtuális hálózaton (VNet) típusúként van definiálva. 

> [!NOTE]
> Ez az útmutató nem tárgyalja alkalmazások fejlesztését. Az Azure-alkalmazások fejlesztésével kapcsolatos további információkért lásd: a [útmutató az Azure Alkalmazásarchitektúrájához](/azure/architecture/guide/).

Függetlenül attól, hogy a számítási feladatok egy webalkalmazás vagy egy virtuális Gépet, ezeket az üzemelő példányokat, szükséges egy **erőforráscsoport**. Egy erőforráscsoport létrehozásához szükséges engedéllyel rendelkező felhasználó kell tennie, amely előtt az alábbi lépéseket.

## <a name="basic-web-application-paas"></a>Alapszintű webalkalmazás (PaaS)

Alapszintű webalkalmazás, válassza az 5 perces gyors útmutatók az egyik a [web apps-dokumentáció](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) , és kövesse a lépéseket. 

> [!NOTE]
> Alapértelmezés szerint a rövid útmutató egyes fogja telepíteni egy erőforráscsoportot. Ebben az esetben azt nem kell explicit módon hozzon létre egy erőforráscsoportot. Ellenkező esetben üzembe helyezése a webalkalmazás a fent létrehozott erőforráscsoportot.

Az egyszerű alkalmazások és szolgáltatások üzembe helyezését követően többet is megtudhat telepítéséhez a bevált gyakorlatokat kapcsolatos egy [alapszintű webalkalmazás](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) az Azure-bA.

## <a name="single-windows-or-linux-vm-iaas"></a>Egyetlen Windows vagy Linux rendszerű virtuális gépek (IaaS)

Egy egyszerű számítási feladat, amely a virtuális gépen fut az első lépés, hogy a virtuális hálózat üzembe helyezése. Például a virtual machines, a terheléselosztók és átjárók minden IaaS-erőforrásokat az Azure-beli virtuális hálózat szükséges. Ismerje meg [Azure virtuális hálózatok](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), majd kövesse a lépéseket [üzembe helyezése egy virtuális hálózatot az Azure-ban a portál](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Az Azure Portalon a virtuális hálózatra vonatkozó beállításainak megadásakor adja meg a fent létrehozott erőforráscsoport nevét.

A következő lépés, hogy egyetlen Windows vagy Linux rendszerű virtuális gép üzembe kell-e. Windows virtuális gép esetén kövesse a lépéseket a [Windows virtuális gép üzembe helyezése az Azure Portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Újra amikor az Azure Portalon adja meg a virtuális gép beállításait, adja meg a fent létrehozott erőforráscsoport nevét.

Miután követte a lépéseket, és a virtuális Gépet üzembe helyezett, megismerkedhet a [bevált eljárások az Azure-on futó Windows virtuális gép](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). Linux rendszerű virtuális gép esetén kövesse a lépéseket a [Linux virtuális gép üzembe helyezése az Azure-bA a portállal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Hogy még többet is megtudhat [bevált eljárások az Azure-on futó Linux rendszerű virtuális gép](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).