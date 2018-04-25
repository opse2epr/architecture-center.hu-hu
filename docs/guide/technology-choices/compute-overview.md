---
title: Az Azure számítási lehetőségeinek áttekintése
description: Az Azure számítási lehetőségeinek áttekintése
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 6781e1f1f6e239e3853920cb544092538cccfc62
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/23/2018
---
# <a name="overview-of-azure-compute-options"></a>Az Azure számítási lehetőségeinek áttekintése

A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. 

A spektrum egyik végén található az **IaaS** (szolgáltatásként nyújtott infrastruktúra). IaaS használata esetén üzembe helyezi a szükséges virtuális gépeket, valamint a hozzájuk társított hálózati és tárolási összetevőket. Ezt követően üzembe helyezi azokat a szoftvereket és alkalmazásokat, amelyeket futtatni kíván a virtuális gépeken. Ez a modell áll a legközelebb a hagyományos helyszíni környezetekhez, azzal a különbséggel, hogy az infrastruktúrát a Microsoft kezeli. Az egyes virtuális gépeket továbbra is Ön felügyeli.  

A **PaaS** (szolgáltatásként nyújtott platform) egy olyan felügyelt üzemeltetési környezetet biztosít, amelyben az alkalmazásait virtuális gépek vagy hálózati erőforrások kezelése nélkül helyezheti üzembe. Például különálló virtuális gépek létrehozása helyett megadhat egy példányszámot, majd a szolgáltatás kiosztja, konfigurálja és kezeli a szükséges erőforrásokat. Az Azure App Service például egy PaaS-szolgáltatás.

Az IaaS és a tiszta PaaS közötti spektrum különböző pontjain is találhatók szolgáltatások. Például az Azure virtuális gépek automatikusan skálázhatók a virtuális gépek méretezési csoportjaival. Ez az automatikus skálázási képesség nem csak szigorúan a PaaS-ra jellemző, de az ilyen felügyeleti funkciók általában PaaS-szolgáltatásokban találhatók.

A **FaaS** (szolgáltatásként nyújtott funkciók) egy lépéssel továbbmegy, hogy Önnek ne kelljen foglalkoznia az üzemeltetési környezettel. Számítási példányok létrehozása, majd ezekben kódok üzembe helyezése helyett egyszerűen üzembe helyezi a kódot, és a szolgáltatás automatikusan lefuttatja. Így nem kell felügyelnie a számítási erőforrásokat. Ezek a szolgáltatások kiszolgáló nélküli architektúrát használnak, és probléma nélkül skálázzák magukat a forgalom kezeléséhez szükséges szintre. Az Azure Functions egy FaaS-szolgáltatás.

Az IaaS nyújtja a legnagyobb irányítást, rugalmasságot és hordozhatóságot. A FaaS egyszerűséget, rugalmas skálázhatóságot és potenciális költségcsökkentést kínál, mivel csak annyi időért kell fizetnie, amennyit a kódja fut. A PaaS valahol a kettő közé esik. Általában minél nagyobb rugalmasságot biztosít egy szolgáltatás, annál inkább felelős Ön az erőforrások konfigurálásáért és kezeléséért. A FaaS-szolgáltatások automatikusan kezelik az alkalmazások futtatásának szinte minden aspektusát, míg az IaaS-megoldásokhoz Önnek kell kiosztania, konfigurálnia és kezelnie a létrehozott virtuális gépeket és hálózati összetevőket.

Az Azure-ban jelenleg a következő fő számítási lehetőségek érhetők el:

- A [Virtual Machines](/azure/virtual-machines/) egy IaaS-szolgáltatás, amellyel virtuális gépeket helyezhet üzembe és kezelhet egy virtuális hálózaton (VNet).
- Az [App Service](/azure/app-service/app-service-value-prop-what-is) egy felügyelt szolgáltatás, amelynek segítségével webalkalmazásokat, mobilalkalmazások háttérkomponenseit, RESTful API-kat vagy automatizált üzleti folyamatokat üzemeltethet.
- A [Service Fabric](/azure/service-fabric/service-fabric-overview) egy elosztott rendszerplatform, amely számos környezetben futtatható, például az Azure-ban vagy a helyszínen. A Service Fabric mikroszolgáltatásokat vezényel számítógépfürtökön. 
- Az [Azure Container Service](/azure/container-service/container-service-intro) segítségével tárolóalapú alkalmazások futtatására konfigurált virtuális gépek fürtjeit hozhatja létre, konfigurálhatja és kezelheti.
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

Válassza ki az alkalmazás számítási szolgáltatás érdekében használjon a [Azure számítási szolgáltatások döntési fája](./compute-decision-tree.md)

Az Azure számítási lehetőségek részletes összehasonlítása, lásd: [feltételek kiválasztása az Azure számítási szolgáltatás](./compute-comparison.md).