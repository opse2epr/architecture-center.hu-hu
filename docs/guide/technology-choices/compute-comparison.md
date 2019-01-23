---
title: Számítási szolgáltatás Azure-beli kritériumai
titleSuffix: Azure Application Architecture Guide
description: Azure számítási szolgáltatások összehasonlítása több szempontból.
author: MikeWasson
ms.date: 08/08/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 2b6b9b941bf7a3c0136b71ecb65bfe4b4a59e07b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484856"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Számítási szolgáltatás Azure-beli kritériumai

A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. Az alábbi táblázat az Azure számítási szolgáltatásokat hasonlítja össze több szempontból. A táblázatok segítségével kiválaszthatja a megfelelő számítási lehetőséget az alkalmazásához.

## <a name="hosting-model"></a>Futtatási modell

<!-- markdownlint-disable MD033 -->

| Feltételek | Virtuális gépek | App Service | Service Fabric-példány | Azure Functions | Azure Kubernetes Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Alkalmazás összeállítása | Független | Alkalmazások, tárolók | Szolgáltatások, futtatható vendégalkalmazások, tárolók | Funkciók | Tárolók | Tárolók | Ütemezett feladatok  |
| Sűrűség | Független | Több alkalmazás példányonként az app service-csomagok keresztül | Több szolgáltatás virtuális gépenként | Kiszolgáló nélküli <a href="#note1"> <sup>1</sup></a> | Csomópontonként több tároló |Nincsenek dedikált példányok | Több alkalmazás virtuális gépenként |
| Csomópontok minimális száma | 1 <a href="#note2"><sup>2</sup></a>  | 1. | 5 <a href="#note3"><sup>3</sup></a> | Kiszolgáló nélküli <a href="#note1"> <sup>1</sup></a> | 3 <a href="#note3"><sup>3</sup></a> | Nincsenek dedikált csomópontok | 1 <a href="#note4"><sup>4</sup></a> |
| Állapotkezelés | Állapot nélküli vagy állapotalapú | Állapot nélküli | Állapot nélküli vagy állapotalapú | Állapot nélküli | Állapot nélküli vagy állapotalapú | Állapot nélküli | Állapot nélküli |
| Webes üzemeltetés | Független | Beépített | Független | Nem alkalmazható | Független | Független | Nincs |
| Üzembe helyezhető dedikált virtuális hálózaton? | Támogatott | Támogatott<a href="#note5"><sup>5</sup></a> | Támogatott | Támogatott <a href="#note5"><sup>5</sup></a> | [Támogatott](/azure/aks/networking-overview) | Nem támogatott | Támogatott |
| Hibrid kapcsolat | Támogatott | Támogatott <a href="#note6"><sup>6</sup></a>  | Támogatott | Támogatott <a href="#node7"><sup>7</sup></a> | Támogatott | Nem támogatott | Támogatott |

Megjegyzések

1. <span id="note1">Használatalapú csomag használata esetén. App Service-csomag használata esetén a függvények az App Service-csomag részeként kiosztott virtuális gépeken futnak. Lásd: [A megfelelő Azure Functions szolgáltatási csomag kiválasztása][function-plans].</span>
2. <span id="note2">Magasabb SLA két vagy több példánnyal.</span>
3. <span id="note3">Éles környezetben ajánlott.</span>
4. <span id="note4">Leskálázható nullára a feladat befejezése után.</span>
5. <span id="note5">App Service-környezet (ASE) szükséges.</span>
6. <span id="note6">Használat [az Azure App Service hibrid kapcsolataira][app-service-hybrid].</span>
7. <span id="note7">App Service-csomag szükséges.</span>

## <a name="devops"></a>DevOps

| Feltételek | Virtuális gépek | App Service | Service Fabric-példány | Azure Functions | Azure Kubernetes Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Helyi hibakeresés | Független | IIS Express, egyebek <a href="#note1b"><sup>1</sup></a> | Helyi fürtcsomópont | A Visual Studio vagy az Azure Functions CLI-vel | Minikube, mások | Tároló helyi futtatókörnyezete | Nem támogatott |
| A programozási modell | Független | Webes és API-alkalmazások, webjobs-feladatok háttérben futó feladatok | Futtatható vendégalkalmazás, szolgáltatási modell, Actor modell, tárolók | Eseményindítókat használó függvények | Független | Független | Parancssori alkalmazás |
| Alkalmazás frissítése | Nincs beépített támogatás | Üzembe helyezési tárhelyek | Működés közbeni frissítés (szolgáltatásonként) | Üzembe helyezési tárhelyek | A működés közbeni frissítés | Nem alkalmazható |

Megjegyzések

1. <span id="note1b">A lehetőségek a következők: ASP.NET-hez vagy node.js-hez (iisnode) készült IIS Express, PHP-webkiszolgáló, IntelliJ-hez készült Azure-eszközkészlet vagy Eclipse-hez készült Azure eszközkészlet. Az App Service ezenkívül támogatja az üzembe helyezett webalkalmazások távoli hibakeresését.</span>
2. <span id="note2b">Lásd: [Resource Manager-szolgáltatók, régiók, API-verziók és sémák][resource-manager-supported-services].</span>

## <a name="scalability"></a>Méretezhetőség

| Feltételek | Virtuális gépek | App Service | Service Fabric-példány | Azure Functions | Azure Kubernetes Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Automatikus méretezés | Virtuális gépek méretezési csoportjai | Beépített szolgáltatás | Virtuálisgép-méretezési csoportok | Beépített szolgáltatás | Nem támogatott | Nem támogatott | N. a. |
| Terheléselosztó | Azure Load Balancer | Integrált | Azure Load Balancer | Integrált | Integrált |  Nincs beépített támogatás | Azure Load Balancer |
| Méretezési korlát<a href="#note1c"><sup>1</sup></a> | Platformlemezkép: 1000 csomópont csoportonként, egyéni lemezkép: 100 csomópont virtuálisgép-méretezési csoportonként | 20 példány, 100 App Service environmenttel | 100 csomópont virtuálisgép-méretezési csoportonként | 200 példányok száma függvényalkalmazás | 100 csomópontok száma fürtönként (alapértelmezett korlát) |20 tárolócsoportok száma előfizetésenként (alapértelmezett korlát). | 20 magos korlát (alapértelmezett korlát). |

Megjegyzések

1. <span id="note1c">Lásd: [Azure-előfizetés és a szolgáltatások korlátozásai, kvótái és megkötései](/azure/azure-subscription-service-limits)</span>.

## <a name="availability"></a>Rendelkezésre állás

| Feltételek | Virtuális gépek | App Service | Service Fabric-példány | Azure Functions | Azure Kubernetes Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [A virtuális gépekre vonatkozó SLA][sla-vm] | [Az App Service-re vonatkozó SLA][sla-app-service] | [A Service Fabricre vonatkozó SLA][sla-sf] | [A Functionsre vonatkozó SLA][sla-functions] | [SLA-t az aks-ben][sla-acs] | [A Container Instances vonatkozó SLA](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Az Azure Batch-re vonatkozó SLA][sla-batch] |
| Többrégiós feladatátvétel | Traffic Manager | Traffic Manager | Traffic Manager, többrégiós fürt | Nem támogatott | Traffic Manager | Nem támogatott | Nem támogatott |

## <a name="other"></a>Egyéb

| Feltételek | Virtuális gépek | App Service | Service Fabric-példány | Azure Functions | Azure Kubernetes Service | Tárolópéldányok | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Virtuális gépen konfigurált | Támogatott | Támogatott  | Támogatott | [Bejövőforgalom-vezérlőt](/azure/aks/ingress) | Használat [oldalkocsi](../../patterns/sidecar.md) tároló | Támogatott |
| Költség | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Az App Service árképzése][cost-app-service] | [A Service Fabric árképzése][cost-service-fabric] | [Az Azure Functions árképzése][cost-functions] | [Az AKS díjszabása][cost-acs] | [Container Instances díjszabását](https://azure.microsoft.com/pricing/details/container-instances/) | [Az Azure Batch árképzése][cost-batch]
| Megfelelő architektúrastílusok | [N szintű][n-tier], [Big compute] [ big-compute] (HPC) | [Web-Queue-Worker][w-q-w], [N-Tier][n-tier] | [Mikroszolgáltatások][microservices], [eseményvezérelt architektúra][event-driven] | [Mikroszolgáltatások][microservices], [eseményvezérelt architektúra][event-driven] | [Mikroszolgáltatások][microservices], [eseményvezérelt architektúra][event-driven] | [Mikroszolgáltatások][microservices], feladat automatizálása, a batch-feladatok  | [Big compute] [ big-compute] (HPC) |

<!-- markdownlint-enable MD033 -->

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/kubernetes-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/kubernetes-service
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

[app-service-hybrid]: /azure/app-service/app-service-hybrid-connections
