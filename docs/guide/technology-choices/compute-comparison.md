---
title: "Feltételek kiválasztása az Azure számítási beállítás"
description: "Az Azure compute szolgáltatások összehasonlítása több tengely mentén."
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 640793b56c1713f63456bab75ab4b9289d22a53c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>Feltételek kiválasztása az Azure számítási beállítás

A kifejezés *számítási* a számítási erőforrásokért, az alkalmazásokat futtató üzemeltetési modell hivatkozik. Az alábbi táblázat összehasonlítja az Azure compute szolgáltatások több tengelyek között. Ezek a táblázatok vonatkoznak, ha az alkalmazás egy számítási lehetőséget választja.

## <a name="hosting-model"></a>Futtatási modell

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Alkalmazás összeállítás | Független | Alkalmazások | Szolgáltatások, a Vendég végrehajtható fájlok | Functions | Tárolók | Szerepkörök | Ütemezett feladatok  |
| Sűrűség | Független | Több alkalmazás keresztül alkalmazás tervek példányonként | Több szolgáltatás virtuális gépenként | Nincs kijelölt példány <a href="#note1"> <sup>1</sup></a> | Több tároló virtuális gépenként | Egy szerepkör példánya virtuális gépenként | Több alkalmazás, virtuális gépenként |
| Csomópontok minimális száma | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | A csomópontok nem dedikált <a href="#note1"> <sup>1</sup></a> | 3 | 2 | 1 <a href="#note4"><sup>4</sup></a> |
| Felügyeleti állapot | Állapot nélküli és állapotalapú | Állapot nélküli | Stateless vagy állapotalapú alkalmazások és szolgáltatások | Állapot nélküli | Állapot nélküli és állapotalapú | Állapot nélküli | Állapot nélküli |
| Webtároláshoz | Független | A beépített | Önálló gazdagép, az IIS-tárolókban lévő | Nem alkalmazható | Független | Beépített (IIS) | Nem |
| Operációs rendszer | Windows, Linux | Windows, Linux (előzetes verzió)  | Windows, Linux (előzetes verzió) | Nem alkalmazható | Windows, Linux | Windows | Windows, Linux |
| Telepíthetők dedikált virtuális hálózatot? | Támogatott | Támogatott <a href="#note5"> <sup>5</sup></a> | Támogatott | Nem támogatott | Támogatott | Támogatott <a href="#note6"> <sup>6</sup></a> | Támogatott |
| A hibrid kapcsolat | Támogatott | Támogatott <a href="#note1"> <sup>7</sup></a>  | Támogatott | Nem támogatott | Támogatott | Támogatott <a href="#note8"> <sup>8</sup></a> | Támogatott |

Megjegyzések

1. <span id="note1">Ha a felhasználási terv használatával. App Service-csomag használata funkciók futtassa az App Service-csomag számára lefoglalt virtuális gépeken. Lásd: [válassza ki a megfelelő service-csomag az Azure Functions][function-plans].</a>
2. <span id="note2">Két vagy több osztályt magasabb SLA-t.</a>
3. <span id="note3">Éles környezetben.</a>
4. <span id="note4">Skála le nulla után feladat befejezése is.</a>
5. <span id="note5">App Service Environment-környezet (ASE) szükséges.</a>
6. <span id="note6">Klasszikus virtuális hálózat csak.</a>
7. <span id="note7">ASE vagy BizTalk hibrid kapcsolatok igényel</a>
8. <span id="note8">Klasszikus virtuális hálózat vagy az erőforrás-kezelő virtuális hálózaton keresztül a Vnetben társviszony-létesítés</a>

## <a name="devops"></a>DevOps

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Helyi hibakeresés | Független | Az IIS Express, mások <a href="#note1b"> <sup>1</sup></a> | Helyi csomópontot tartalmazó fürtben | Az Azure Functions parancssori felület | Helyi tároló futásidejű | Helyi emulátor | Nem támogatott |
| Programozási modell | Független | Webalkalmazás, webjobs-feladatok a háttérben feladatokhoz | Vendég végrehajtható, szolgáltatásmodellt, a modell szereplő tárolókhoz | Az eseményindítók funkciók | Független | Webes szerepkör, a feldolgozói szerepkör | Parancssori alkalmazás |
| Resource Manager | Támogatott | Támogatott | Támogatott | Támogatott | Támogatott | Korlátozott <a href="#note2b"> <sup>2</sup></a> | Támogatott |  
| Alkalmazás frissítése | Nincs beépített támogatása | Üzembehelyezési pontok | Működés közbeni frissítés száma (szolgáltatás) | Nincs beépített támogatása | Az orchestrator függ. A legtöbb támogatja a működés közbeni frissítések | Virtuális IP-címcsere vagy a működés közbeni frissítés | Nem alkalmazható |

Megjegyzések

1. <span id="note1b">A választható lehetőségek az IIS Express ASP.NET vagy node.js (iisnode); A PHP webkiszolgáló; Az IntelliJ, Eclipse Azure eszköztára Azure eszköztára. App Service is támogatja a telepített webalkalmazás távoli hibakeresés.</a>
2. <span id="note2b">Lásd: [Resource Manager szolgáltatók, régiók, API verziók és sémák][resource-manager-supported-services]. 


## <a name="scalability"></a>Méretezhetőség

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Automatikus skálázással | Virtuálisgép-méretezési készlet | Beépített szolgáltatás | Virtuális gépek méretezési csoportjai | Beépített szolgáltatás | Nem támogatott | Beépített szolgáltatás | N/A |
| Terheléselosztó | Azure Load Balancer | Integrált | Azure Load Balancer | Integrált | Azure Load Balancer | Integrált | Azure Load Balancer |
| Skála korlátot | Platformlemezképet: 1000-csomópontok ma_ximális száma VMSS, egyéni lemezkép: 100-csomópontok ma_ximális száma VMSS | példányok 20, 50 az App Service Environment-környezet | 100-csomópontok ma_ximális száma VMSS | Végtelen <a href="#note1c"> <sup>1</sup></a> | 100 | Nem meghatározott korláttal, 200 maximális ajánlott | Alapértelmezés szerint 20 core korlátot. Lépjen kapcsolatba az ügyfélszolgálat növekedést. |

Megjegyzések

1. <span id="note1c">Ha a felhasználási terv használatával. App Service-csomag használata esetén az App Service méretkorlátai alkalmazni. Lásd: [válassza ki a megfelelő service-csomag az Azure Functions][function-plans].</a>

## <a name="availability"></a>Rendelkezésre állás

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [SLA-t a virtuális gépek][sla-vm] | [Az App Service SLA][sla-app-service] | [A Service Fabric SLA][sla-sf] | [A funkciók SLA][sla-functions] | [Az Azure Tárolószolgáltatás SLA][sla-acs] | [SLA-t, a Cloud Services csomag][sla-cloud-service] | [Az Azure Batch SLA][sla-batch] |
| Több régióban feladatátvétel | Forgalomkezelő | Forgalomkezelő | A TRAFFIC manager, több területi fürt | Nem támogatott  | Forgalomkezelő | Forgalomkezelő | Nem támogatott |

## <a name="security"></a>Biztonság

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | A virtuális gép konfigurálva | Támogatott | Támogatott  | Támogatott | A virtuális gép konfigurálva | Támogatott | Támogatott |
| RBAC | Támogatott | Támogatott | Támogatott | Támogatott | Támogatott | Nem támogatott | Támogatott |

## <a name="other"></a>Egyéb

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Költségek | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [App Service díjszabás][cost-app-service] | [A Service Fabric díjszabása][cost-service-fabric] | [Az Azure Functions díjszabási][cost-functions] | [Azure Tárolószolgáltatás díjszabása][cost-acs] | [Cloud Services díjszabása][cost-cloud-services] | [Azure Batch díjszabása][cost-batch]
| Megfelelő architektúra stílusok | N szintű, nagy számítási (HPC) | Webalkalmazás-várólista-Worker | Mikroszolgáltatások, esemény-architektúra (EDA) | Mikroszolgáltatások, EDA | Mikroszolgáltatások, EDA | Webalkalmazás-várólista-Worker | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services