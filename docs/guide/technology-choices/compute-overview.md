---
title: Az Azure számítási lehetőségeinek áttekintése
titleSuffix: Azure Application Architecture Guide
description: Az Azure számítási lehetőségeinek áttekintése.
author: MikeWasson
ms.date: 06/13/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 63754ce84f001226b6cddf1f30a152bcfce3e7ac
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299190"
---
# <a name="overview-of-azure-compute-options"></a>Az Azure számítási lehetőségeinek áttekintése

A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut.

## <a name="overview"></a>Áttekintés

A spektrum egyik végén **infrastruktúra--szolgáltatásként** (IaaS). IaaS használata esetén üzembe helyezi a szükséges virtuális gépeket, valamint a hozzájuk társított hálózati és tárolási összetevőket. Ezt követően üzembe helyezi azokat a szoftvereket és alkalmazásokat, amelyeket futtatni kíván a virtuális gépeken. Ez a modell áll a legközelebb a hagyományos helyszíni környezetekhez, azzal a különbséggel, hogy az infrastruktúrát a Microsoft kezeli. Az egyes virtuális gépeket továbbra is Ön felügyeli.

**Platformszolgáltatás (PaaS)** biztosít a felügyelt üzemeltetési környezetet, ahol telepítheti az alkalmazást anélkül, hogy a virtuális gépek vagy hálózati erőforrások kezelése. Például különálló virtuális gépek létrehozása helyett megadhat egy példányszámot, majd a szolgáltatás kiosztja, konfigurálja és kezeli a szükséges erőforrásokat. Az Azure App Service például egy PaaS-szolgáltatás.

Az IaaS és a tiszta PaaS közötti spektrum különböző pontjain is találhatók szolgáltatások. Például az Azure virtuális gépek automatikusan skálázhatók a virtuális gépek méretezési csoportjaival. Ez az automatikus skálázási képesség nem csak szigorúan a PaaS-ra jellemző, de az ilyen felügyeleti funkciók általában PaaS-szolgáltatásokban találhatók.

A **FaaS** (szolgáltatásként nyújtott funkciók) egy lépéssel továbbmegy, hogy Önnek ne kelljen foglalkoznia az üzemeltetési környezettel. Számítási példányok létrehozása, majd ezekben kódok üzembe helyezése helyett egyszerűen üzembe helyezi a kódot, és a szolgáltatás automatikusan lefuttatja. Így nem kell felügyelnie a számítási erőforrásokat. Ezek a szolgáltatások használata kiszolgáló nélküli architektúrával, és zökkenőmentesen méretezheti felfelé vagy lefelé a forgalom kezeléséhez szükséges szintre. Az Azure Functions egy FaaS-szolgáltatás.

Az IaaS nyújtja a legnagyobb irányítást, rugalmasságot és hordozhatóságot. A FaaS egyszerűséget, rugalmas skálázhatóságot és potenciális költségcsökkentést kínál, mivel csak annyi időért kell fizetnie, amennyit a kódja fut. A PaaS valahol a kettő közé esik. Általában minél nagyobb rugalmasságot biztosít egy szolgáltatás, annál inkább felelős Ön az erőforrások konfigurálásáért és kezeléséért. A FaaS-szolgáltatások automatikusan kezelik az alkalmazások futtatásának szinte minden aspektusát, míg az IaaS-megoldásokhoz Önnek kell kiosztania, konfigurálnia és kezelnie a létrehozott virtuális gépeket és hálózati összetevőket.

## <a name="azure-compute-options"></a>Az Azure számításüzemeltetési lehetőségei

Az Azure-ban jelenleg a következő fő számítási lehetőségek érhetők el:

- A [Virtual Machines](/azure/virtual-machines/) egy IaaS-szolgáltatás, amellyel virtuális gépeket helyezhet üzembe és kezelhet egy virtuális hálózaton (VNet).
- [App Service-ben](/azure/app-service/app-service-value-prop-what-is) felügyelt paas-megoldás kínál webalkalmazások, mobilalkalmazások háttérkomponenseit, RESTful API-kat vagy automatizált üzleti folyamatok üzemeltetésére.
- A [Service Fabric](/azure/service-fabric/service-fabric-overview) egy elosztott rendszerplatform, amely számos környezetben futtatható, például az Azure-ban vagy a helyszínen. A Service Fabric mikroszolgáltatásokat vezényel számítógépfürtökön.
- [Az Azure Kubernetes Service](/azure/aks/) kezeli az üzemeltetett Kubernetes-szolgáltatást a tárolóalapú alkalmazások futtatására.
- [Az Azure Container Instances](/azure/container-instances/container-instances-overview) kínálnak a leggyorsabb és legegyszerűbb módon futtathatók tárolók az Azure-ban, és a egy magasabb szintű szolgáltatást kellene minden olyan virtuális gépek üzembe helyezése nélkül.
- Az [Azure Functions](/azure/azure-functions/functions-overview) egy felügyelt FaaS-szolgáltatás.
- Az [Azure Batch](/azure/batch/batch-technical-overview) egy felügyelt szolgáltatás, amellyel nagy méretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat futtathat.
- A [Cloud Services](/azure/cloud-services/cloud-services-choose-me) egy felügyelt szolgáltatás, amellyel felhőalkalmazásokat futtathat. PaaS futtatási modellt használ.

A számítási lehetőség kiválasztásakor a következő tényezőket érdemes figyelembe venni:

- Futtatási modell. Hogyan üzemelteti a szolgáltatást? Ez az üzemeltetési környezet milyen követelményeket és korlátozásokat von magával?
- DevOps. Rendelkezik beépített támogatással az alkalmazások frissítéséhez? Milyen az üzemi modell?
- Méretezhetőség. Hogyan kezeli a szolgáltatás a példányok hozzáadását és eltávolítását? Képes automatikus skálázásra a terhelés és egyéb mérőszámok alapján?
- Rendelkezésre állás. Mi a szolgáltatás SLA-ja?
- Költségek. A szolgáltatás költsége mellett vegye figyelembe a szolgáltatásra épülő megoldás felügyeletének üzemeltetési költségét is. Az IaaS-megoldások például magasabb üzemeltetési költségekkel járhatnak.
- Mik az egyes szolgáltatások általános korlátozásai?
- Milyen jellegű alkalmazási architektúrák megfelelők a szolgáltatáshoz?

## <a name="next-steps"></a>További lépések

Az alkalmazás számítási szolgáltatás kiválasztása érdekében használjon a [döntési fát az Azure számítási szolgáltatások](./compute-decision-tree.md)

Az Azure számítási lehetőségeinek részletesebb összehasonlításáért lásd [számítási szolgáltatás Azure-beli kritériumai](./compute-comparison.md).
