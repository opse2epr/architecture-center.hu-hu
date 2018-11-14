---
title: CAE-szolgáltatás az Azure-ban
description: SaaS-platformot biztosíthat CAE-projektekhez az Azure-ban.
author: alexbuckgit
ms.date: 08/22/2018
ms.openlocfilehash: 8bdf7198223f7194d0cd717949699bb3a508674e
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610549"
---
# <a name="a-computer-aided-engineering-service-on-azure"></a>CAE-szolgáltatás az Azure-ban

A példaforgatókönyv azt szemlélteti, hogy az Azure nagy teljesítményű feldolgozási (HPC-) funkcióit épülő szoftverek,-szolgáltatásként nyújtott (SaaS) platform kézbesítési. Ebben a forgatókönyvben egy mérnöki szoftvermegoldás alapul. Az architektúra azonban fontos egyéb HPC-erőforrások, például a képrenderelés, összetett modellezés és pénzügyi kockázat számítása igénylő iparágakban.

Ez a példa bemutatja egy mérnöki szoftverszolgáltató, amely számítógéppel támogatott mérnöki (CAE-) alkalmazások mérnöki cégek és vállalatok gyártási biztosít. CAE megoldások innovációnak, csökkentheti a fejlesztésre fordított időt és a egy termék tervezése során költségek csökkentését. Ezek a megoldások jelentős számítási erőforrásokat kér, és gyakran a nagy adatmennyiség feldolgozása. A magas költségeknek egy helyszíni HPC-berendezés vagy gyakran helyezni ezeket a technológiákat, összesen: csúcskategóriás munkaállomások mérnöki kisvállalkozások, a saját és a tanulók számára érhető el.

A vállalat úgy kívánja bontsa ki a rajtuk található alkalmazások számára a piaci alapját a HPC-technológiák felhőalapú SaaS-platformként létrehozásával. Az ügyfelek igény szerint kell fizetnie a számítási erőforrások és nagy számítási teljesítményt, amely egyébként lenne unaffordable érheti el képesnek kell lennie.

A vállalati célok a következők:

* Kihasználhatja a HPC-képességek az Azure-ban, a termék tervezési és a tesztelési folyamat meggyorsítása érdekében.
* Használja a legújabb hardveres innovációknak minimalizálhatók a költségek egyszerűbb szimulációkhoz összetett szimulációk futtatásához.
* Igaz élettartam Vizualizáció engedélyezése, és egy webböngészőben, anélkül, hogy a csúcskategóriás műszaki munkaállomás megjelenítése.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

* Genomikai kutatás
* Időjárás-szimuláció
* Alkalmazások számítási kémia

## <a name="architecture"></a>Architektúra

![A Szolgáltatottszoftver-megoldás, amely lehetővé teszi a HPC-képességek architektúrája][architecture]

* A felhasználók NV-sorozat virtuális gépeken (VM) egy böngészőből egy HTML5-alapú RDP kapcsolat használatakor a [Apache Guacamole szolgáltatás](https://guacamole.apache.org/). Virtuálisgép-példány hatékony gpu-k képmegjelenítési, valamint által biztosított együttműködési környezettel feladatok adja meg. A felhasználók saját tervek szerkesztése és az eredmények megtekintése a nagy teljesítményű számítástechnikai mobileszközök vagy hordozható számítógépeken való hozzáférés nélkül. Az ütemező felhasználó által definiált heurisztika alapuló további virtuális gépeket indít el.
* Egy asztali CAD-munkamenetben a felhasználók elküldhetik a számítási feladatok végrehajtásához a rendelkezésre álló HPC-fürtcsomópontokon. Ilyen számítási feladat például feszültségelemzés vagy a folyadékdinamikai számítások, így nincs szükség helyszíni dedikált számítási fürtök feladatait. Ezek a fürtcsomópontok automatikus skálázásra a terhelés vagy üzenetsor mélységtől függ az aktív felhasználói igény szerinti számítási erőforrások konfigurálható.
* Az Azure Kubernetes Service (AKS) segítségével a végfelhasználók számára elérhető webes erőforrások üzemeltetéséhez.

### <a name="components"></a>Összetevők

* [H-sorozatú virtual machineshez](/azure/virtual-machines/linux/sizes-hpc) gombfolyamatokkal nagy számítási igényű szimulációk, például a molekuláris modellezés és folyadékdinamika. A megoldás is kihasználja a technológiákkal, mint például a távoli közvetlen memória-hozzáféréses (RDMA) kapcsolat és InfiniBand hálózatok.
* [NV-sorozat virtuális gépei](/azure/virtual-machines/windows/sizes-gpu) csúcskategóriás munkaállomás funkció mérnökök adjon egy standard webböngészőből. Ezek a virtuális gépek rendelkeznek az NVIDIA Tesla M60 gpu-kat, amely támogatja a speciális megjelenítés és különálló precíziós számítási feladatokat futtathat.
* [Általános célú virtuális gépek](/azure/virtual-machines/linux/sizes-general) futó CentOS például webalkalmazások hagyományosabb munkaterhelések kezelésére.
* [Az Application Gateway](/azure/application-gateway/overview) elosztja a terhelést a webkiszolgálók érkező kérelmeket.
* [Az Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) szimulációkhoz, amelyeknek nincs szükségük a HPC- vagy GPU virtuális gépek nagy képességeit alacsonyabb költségekkel méretezhető számítási feladatok futtatására szolgál.
* [Altair PBS Works Suite](https://www.pbsworks.com/PBSProduct.aspx?n=PBS-Works-Suite&c=Overview-and-Capabilities) koordinálja a HPC munkafolyamat, biztosítva, hogy elegendő virtuálisgép-példányok érhetők el a jelenlegi terhelés kezeléséhez. Is felszabadítja a virtuális gépeket, ha igény szerinti alacsonyabb költségek csökkentése érdekében.
* [A BLOB storage-](/azure/storage/blobs/storage-blobs-introduction) tárolja a fájlokat, amelyek támogatják az ütemezett feladatok. 

### <a name="alternatives"></a>Alternatív megoldások

* [Az Azure CycleCloud](/azure/cyclecloud/overview) egyszerűbbé teszi a létrehozását, kezelését, üzemeltetési és HPC-fürtök optimalizálása. Speciális szabályzatot és cégirányítási funkciókat kínál. CycleCloud bármely Feladatütemező vagy szoftververem támogatja.
* [HPC Pack](/azure/virtual-machines/windows/hpcpack-cluster-options) hozhat létre és kezelhet az Azure HPC-fürt, a Windows Server-alapú számítási feladatokhoz. HPC Pack nincs lehetőség a Linux-alapú számítási feladatokhoz.
* [Azure Automation Állapotkonfiguráció](/azure/automation/automation-dsc-overview) biztosít az infrastruktúra mint kód megközelítése a virtuális gépek és a segítségével központilag telepítendő szoftver. Virtuális gépek egy virtuális gép méretezési csoportot, az automatikus skálázási szabályokat a számítási csomópontok, a feladat-várólistában számára küldött számítási feladatok száma alapján részeként is telepíthető. Ha egy új virtuális gép van szükség, annak kiépítése az Azure lemezkép-katalógusában a legújabb javított lemezkép használatával, és majd a szükséges szoftverek telepítve és konfigurálva van a PowerShell DSC konfigurációs parancsfájl használatával.
* [Azure Functions](/azure/azure-functions/functions-overview)

## <a name="considerations"></a>Megfontolandó szempontok

* Az infrastruktúra mint kód megközelítéssel kiválóan alkalmas kezelheti a virtuális gép buildelési definíciókat, amíg egy parancsfájl segítségével új virtuális gép kiépítése hosszú ideig eltarthat. Ez a megoldás egy jó közel földön található készít rendszeres időközönként az arany lemezképen, amelyek ezután felhasználhatók egy új virtuális gép gyorsabb, mint a teljes mértékben létrehozása DSC használatával igény szerinti virtuális gép kiépítése a DSC-parancsfájl használatával. Az Azure DevOps-szolgáltatásokkal vagy más CI/CD-eszközök rendszeres időközönként frissítheti arany lemezképek DSC-parancsfájlok használatával.
* Teljes megoldás költségek elosztására a számítási erőforrások gyors elérhetősége a legfontosabb szempont. Az N-sorozat virtuálisgép-példányok készlet kiépítése és felszabadított állapotban lévő helyezné azokat csökkenti a működési költségek mellett. Ha egy további virtuális gép van szükség, újra-hozzárendelése egy meglévő példányát magában foglalja a virtuális gép, egy másik gazdagép működtetésére, de az operációs rendszer azonosításához, és telepítheti az illesztőprogramokat a grafikus Processzor számára szükséges PCI-busz észlelés ideje nem szűnik meg, mert egy virtuális gép, majd kiépíteni, és eltávolította a következő megtartja ugyanazt a PCI-busz az újraindítás után a GPU.
* Az eredeti architektúra teljesen szimulációk futtatásához az Azure virtual machines hivatkozni. Annak érdekében, hogy a virtuális gép összes funkcióját nem igénylő számítási feladatok költségeinek csökkentése érdekében ezeket a feladatokat is konténeralapú és üzembe helyezése az Azure Kubernetes Service (AKS).
* A munkaerő korábban meglévő ismereteit nyílt forráskódú technológiákat. Kihasználhatja az ezekkel a technológiákkal, mint például a Linux és a Kubernetes létrehozásával. 

## <a name="pricing"></a>Díjszabás

Felfedezheti a forgatókönyv futtatásával járó költségeket érdekében a szükséges szolgáltatások számos előre konfigurált a egy [költséget számító példa][calculator]. A költségek, hogy a megoldás száma és a követelmények teljesítéséhez szükséges szolgáltatások méretezése függenek.

Az alábbi szempontokat a költségek, a megoldás egy jelentős része lesz meghajtó:
* Azure virtuális gépek költségeit, további példányokat kiépített lineárisan növelése. Virtuális gépek felszabadított állapotban vannak, csak a tárolási költségekkel, és nem számítási költségeit. Ezek a gépek felszabadítva majd kell újra kiosztja, ha igény szerint magas.
* Az Azure Kubernetes-szolgáltatás költségeket a számítási a kiválasztott virtuális gép típus. A költségek megnöveli a költségráfordításokkal egyenes arányban alapján a virtuális gépek száma a fürtben.

## <a name="next-steps"></a>További lépések

* Olvassa el a [Altair vásárlói beszámolónk][source-document]. Ebben a példaforgatókönyvben architektúráját verzióján alapul.
* Tekintse át a többi [Big Compute-megoldások](https://azure.microsoft.com/solutions/big-compute) elérhető az Azure-ban.

<!-- links -->
[architecture]: ./media/architecture-hpc-saas.png
[source-document]: https://customers.microsoft.com/story/altair-manufacturing-azure
[calculator]: https://azure.com/e/3cb9ccdc893f41ffbcdb00c328178ccf
