---
title: Az Azure compute beállítások áttekintése
description: Az Azure compute beállítások áttekintése
author: MikeWasson
ms.openlocfilehash: a23dd49f24bc52db6f357540e3ebccb19e0497ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="overview-of-azure-compute-options"></a>Az Azure compute beállítások áttekintése

A kifejezés *számítási* a számítási erőforrásokért, az alkalmazást futtató üzemeltetési modell hivatkozik. 

Egy végén a **Intrastructure,--szolgáltatás** (IaaS). Az infrastruktúra-szolgáltatási a virtuális gépeket, amelyekre szüksége van, valamint társított hálózati és tárolási összetevőinek telepítéséhez. Ezután telepítse központilag bármilyen szoftver- és virtuális gépek kívánt alkalmazásokat. Ez a modell egy hagyományos helyszíni környezetben, a legközelebb, azzal a különbséggel, hogy az infrastruktúra a Microsoft felügyeli. Továbbra is kezelheti az egyes virtuális gépeken.  

**Platform,--szolgáltatás** (PaaS) biztosít a felügyelt üzemeltetési környezetben, ahol telepítheti az alkalmazást anélkül, hogy a virtuális gépek vagy a hálózati erőforrások kezelése. Például helyett az egyes virtuális gépek létrehozására, adjon meg egy példányok száma, és a szolgáltatás telepítéséhez, konfigurálása, és a szükséges erőforrások kezelése. Az Azure App Service egy PaaS szolgáltatás példája.

Az IaaS a tiszta PaaS számos van. Azure virtuális gépek például Virtuálisgép-méretezési készlet használatával is az automatikus skálázása. Az automatikus méretezési funkció nem feltétlenül PaaS, de kezelési funkció, amely egy PaaS szolgáltatásban található előfordulhat, hogy milyen típusú.

**Funkciók,--szolgáltatás** (FaaS) még tovább szükség az üzemeltetési környezetben foglalkoznia a kerül. Létrehozása helyett példányok számítási és a kód telepítésének logikailemez, egyszerűen telepítse kódját, és a szolgáltatás automatikusan futtatja. Nem kell felügyelheti a számítási erőforrásokat. Ezek a szolgáltatások ellenőrizze a kiszolgáló nélküli architektúrát használó, és problémamentesen vertikális vagy bármilyen a forgalom kezeléséhez szükséges szintre. Az Azure Functions FaaS szolgáltatás.

Infrastruktúra-szolgáltatási lehetőséget a legtöbb, rugalmasan, és hordozhatóság. FaaS biztosít az egyszerűség, rugalmasan méretezhető, és lehetséges költségmegtakarításhoz, mert a kódja fut. időért kell fizetnie. A PaaS valahol a két közé esik. Általában a nagyobb rugalmasságot biztosít a szolgáltatás, annál telepítésért felelős, konfigurálásához és kezeléséhez az erőforrásokat. FaaS szolgáltatások automatikusan kezelheti egy alkalmazás fut, miközben IaaS-megoldásokat kell kiépíteni, konfigurálása és kezelése, a virtuális gépek és a hálózati összetevők hoz létre szinte minden szempontját.

Itt érhetők el a fő számítási beállítások jelenleg az Azure-ban:

- [Virtuális gépek](/azure/virtual-machines/) IaaS szolgáltatásnak, hogy lehetővé teszi a központi telepítését és felügyeletét a virtuális gépek virtuális hálózatot (VNet) belül vannak.
- [App Service](/azure/app-service/app-service-value-prop-what-is) egy felügyelt szolgáltatás webalkalmazások, mobilalkalmazásokból, RESTful API-k vagy automatizált üzleti folyamatok üzemeltetéséhez.
- [A Service Fabric](/azure/service-fabric/service-fabric-overview) egy elosztott rendszerek platform, amely sok környezetben, beleértve az Azure, illetve a helyszíni futtathatók. A Service Fabric egy orchestrator mikroszolgáltatások létrehozására a gépet fürtön belül. 
- [Az Azure Tárolószolgáltatás](/azure/container-service/container-service-intro) lehetővé teszi, hogy létrehozására, konfigurálására és egy fürt indexelése alkalmazások futtatásához az előre konfigurált virtuális gépek kezeléséhez.
- [Az Azure Functions](/azure/azure-functions/functions-overview) egy felügyelt FaaS szolgáltatás.
- [Az Azure Batch](/azure/batch/batch-technical-overview) egy olyan felügyelt szolgáltatás futtatásához a nagyméretű, párhuzamos és nagy teljesítményű számítástechnikai (HPC) alkalmazások.
- [A felhőalapú szolgáltatások](/azure/cloud-services/cloud-services-choose-me) egy felügyelt szolgáltatás, a felhőalapú alkalmazások futtatásához. A PaaS üzemeltetési modell használ. 

Ha egy számítási lehetőséget választja, az alábbiakban néhány tényezőket kell figyelembe venni:

- Futtatási modell. Hogyan tárolódik a szolgáltatást? Milyen követelmények és korlátozások elő az üzemeltetési környezetben? 
- DevOps. Alkalmazásfrissítések beépített támogatása van? Mi az a telepítési modell?
- Méretezhetőség. Hogyan kezeli a a szolgáltatás a hozzáadásával vagy eltávolításával példányok? Képes az automatikus méretezése a terhelési és más metrikák alapján? 
- A rendelkezésre állás. Mi az a szolgáltatás SLA-t? 
- Költség. A szolgáltatás költségének mellett vegye figyelembe a kezelése, a megoldás, a szolgáltatás beépített műveletek költsége. IaaS-megoldásokat Előfordulhat például, a költség magasabb műveletek.
- Mik az egyes szolgáltatások általános korlátozások? 
- Milyen jellegű alkalmazási architektúrákban megfelelőek ezt a szolgáltatást? 

Az Azure számítási lehetőségek részletes összehasonlítása, lásd: [feltételek kiválasztása az Azure számítási beállítás](./compute-comparison.md).