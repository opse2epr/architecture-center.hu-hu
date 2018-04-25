---
title: Feltételek kiválasztása az Azure számítási szolgáltatás
description: Az Azure compute szolgáltatások összehasonlítása több tengely mentén
author: MikeWasson
layout: LandingPage
ms.date: 04/21/2018
ms.openlocfilehash: ff90ec41c56ae0ecb81bc82128f02fd06d02cb32
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/23/2018
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Feltételek kiválasztása az Azure számítási szolgáltatás

A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. Az alábbi táblázat az Azure számítási szolgáltatásokat hasonlítja össze több szempontból. A táblázatok segítségével kiválaszthatja a megfelelő számítási lehetőséget az alkalmazásához.

## <a name="hosting-model"></a>Futtatási modell

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Alkalmazás összeállítása | Független | Alkalmazások | Szolgáltatások, futtatható vendégalkalmazások, tárolók | Functions | Containers | Containers | Ütemezett feladatok  |
| Sűrűség | Független | Több alkalmazás példányonként az alkalmazásterveken keresztül | Több szolgáltatás virtuális gépenként | Nincsenek dedikált példányok <a href="#note1"><sup>1</sup></a> | Több tároló virtuális gépenként |Nincs kijelölt példány | Több alkalmazás virtuális gépenként |
| Csomópontok minimális száma | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Nincsenek dedikált csomópontok <a href="#note1"><sup>1</sup></a> | 3 | A csomópontok nem dedikált | 1 <a href="#note4"><sup>4</sup></a> |
| Állapotkezelés | Állapot nélküli vagy állapotalapú | Állapot nélküli | Állapot nélküli vagy állapotalapú | Állapot nélküli | Állapot nélküli vagy állapotalapú | Állapot nélküli | Állapot nélküli |
| Webes üzemeltetés | Független | Beépített | Független | Nem alkalmazható | Független | Független | Nem |
| Operációs rendszer | Windows, Linux | Windows, Linux  | Windows, Linux | Nem alkalmazható | Windows (előzetes verzió), Linux | Windows, Linux | Windows, Linux |
| Üzembe helyezhető dedikált virtuális hálózaton? | Támogatott | Támogatott <a href="#note5"><sup>5</sup></a> | Támogatott | Nem támogatott | Támogatott | Nem támogatott | Támogatott |
| Hibrid kapcsolat | Támogatott | Támogatott <a href="#note1"><sup>6</sup></a>  | Támogatott | Nem támogatott | Támogatott | Nem támogatott | Támogatott |

Megjegyzések

1. <span id="note1">Használatalapú csomag használata esetén. App Service-csomag használata esetén a függvények az App Service-csomag részeként kiosztott virtuális gépeken futnak. Lásd: [A megfelelő Azure Functions szolgáltatási csomag kiválasztása][function-plans].</a>
2. <span id="note2">Magasabb SLA két vagy több példánnyal.</a>
3. <span id="note3">Éles környezetekhez.</a>
4. <span id="note4">Leskálázható nullára a feladat befejezése után.</a>
5. <span id="note5">App Service-környezet (ASE) szükséges.</a>
6. <span id="note7">ASE vagy BizTalk Hybrid Connections szükséges</a>

## <a name="devops"></a>DevOps

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Helyi hibakeresés | Független | IIS Express, egyebek <a href="#note1b"><sup>1</sup></a> | Helyi fürtcsomópont | Azure Functions parancssori felület | Tároló helyi futtatókörnyezete | Tároló helyi futtatókörnyezete | Nem támogatott |
| A programozási modell | Független | Webalkalmazás, WebJobs-feladatok hozzáadása a háttérfeladatokhoz | Futtatható vendégalkalmazás, szolgáltatási modell, Actor modell, tárolók | Eseményindítókat használó függvények | Független | Független | Parancssori alkalmazás |
| Alkalmazás frissítése | Nincs beépített támogatás | Üzembehelyezési pontok | Működés közbeni frissítés (szolgáltatásonként) | Nincs beépített támogatás | A vezénylőtől függ. A legtöbb támogatja a működés közbeni frissítéseket | A tárolórendszerkép frissítése | Nem alkalmazható |

Megjegyzések

1. <span id="note1b">A lehetőségek a következők: ASP.NET-hez vagy node.js-hez (iisnode) készült IIS Express, PHP-webkiszolgáló, IntelliJ-hez készült Azure-eszközkészlet vagy Eclipse-hez készült Azure eszközkészlet. Az App Service ezenkívül támogatja az üzembe helyezett webalkalmazások távoli hibakeresését.</a>
2. <span id="note2b">Lásd: [Resource Manager-szolgáltatók, -régiók, API-verziók és -sémák][resource-manager-supported-services]. 


## <a name="scalability"></a>Méretezhetőség

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Automatikus méretezés | Virtuális gépek méretezési csoportjai | Beépített szolgáltatás | Virtuális gépek méretezési csoportjai | Beépített szolgáltatás | Nem támogatott | Nem támogatott | – |
| Terheléselosztó | Azure Load Balancer | Integrált | Azure Load Balancer | Integrált | Azure Load Balancer |  Nincs beépített támogatás | Azure Load Balancer |
| Méretkorlát | Platformlemezkép: 1000 csomópont virtuálisgép-méretezési csoportonként, Egyéni lemezkép: 100 csomópont virtuálisgép-méretezési csoportonként | 20 példány, 50 App Service Environmenttel | 100 csomópont virtuálisgép-méretezési csoportonként | Végtelen <a href="#note1c"><sup>1</sup></a> | 100 |előfizetésenként 20 tárolócsoportok <a href="#note2c"> <sup>2</sup></a> | Alapértelmezés szerint 20 magos korlát. Az értékek növeléséhez forduljon az ügyfélszolgálathoz. |

Megjegyzések

1. <span id="note1c">Használatalapú csomag használata esetén. App Service-csomag használata esetén az App Service méretkorlátai érvényesek. Lásd: [A megfelelő Azure Functions szolgáltatási csomag kiválasztása][function-plans].</a>
2. <span id="note2c">Lásd: [kvótái és az Azure-tároló példányok régiónkénti elérhetőség](/azure/container-instances/container-instances-quotas).</a>


## <a name="availability"></a>Rendelkezésre állás

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [A virtuális gépekre vonatkozó SLA][sla-vm] | [Az App Service-re vonatkozó SLA][sla-app-service] | [A Service Fabricre vonatkozó SLA][sla-sf] | [A Functionsre vonatkozó SLA][sla-functions] | [Az Azure Container Service-re vonatkozó SLA][sla-acs] | [SLA-t, tároló-példányok](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Az Azure Batch-re vonatkozó SLA][sla-batch] |
| Többrégiós feladatátvétel | Traffic Manager | Traffic Manager | Traffic Manager, többrégiós fürt | Nem támogatott  | Traffic Manager | Nem támogatott | Nem támogatott |

## <a name="other"></a>Egyéb

| Feltételek | Virtuális gépek | App Service | Service Fabric | Azure Functions | Azure Container Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Virtuális gépen konfigurált | Támogatott | Támogatott  | Támogatott | Virtuális gépen konfigurált | Nem támogatott | Támogatott |
| Költségek | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Az App Service árképzése][cost-app-service] | [A Service Fabric árképzése][cost-service-fabric] | [Az Azure Functions árképzése][cost-functions] | [Az Azure Container Service árképzése][cost-acs] | [Tároló példányok díjszabása](https://azure.microsoft.com/pricing/details/container-instances/) | [Az Azure Batch árképzése][cost-batch]
| Megfelelő architektúrastílusok | N szintű, Big Compute (HPC) | Webüzenetsor-feldolgozó | Mikroszolgáltatások, eseményvezérelt architektúra (EDA) | Mikroszolgáltatások | Mikroszolgáltatások | Feladatok automatizálása, kötegelt feladatok mikroszolgáltatások létrehozására  | Big Compute |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services