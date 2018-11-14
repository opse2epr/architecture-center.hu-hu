---
title: 3D-s videórenderelés az Azure-ban
description: Natív HPC számítási feladatokat futtathat az Azure-ban az Azure Batch szolgáltatás használatával.
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 1ffdaa5467fec73a01b8caa18b71c2bc4e49abbe
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610685"
---
# <a name="3d-video-rendering-on-azure"></a>3D-s videórenderelés az Azure-ban

Videó 3D renderelési az időigényes, amely jelentős mennyiségű Processzor időtartama igényel. Egyetlen gépen, létrehozni bloberőforrásokhoz videofájl származó statikus objektumokat is órákig vagy akár napokig eltarthat hossza és a videót, akkor állít elő összetettségétől függően. Számos vállalat vagy költséges, csúcskategóriás fogja megvásárolni ezeket a feladatokat, vagy be, amely is küldhetők be feladatok a nagy renderelő farmokat asztali számítógépeket. Azonban az Azure Batch előnyeit kihasználva Ez a teljesítmény érhetőek el, ha szükséges, és leállításakor magát, ha nem, akkor minden tőkebefektetés nélkül.

A Batch lehetővé teszi egy egységes felügyeleti élmény és a feladatütemezésben választja, a Windows Server vagy Linux számítási csomópontok. A Batch-Csel használhatja a meglévő Windows vagy Linux alkalmazásai, például AutoDesk Maya és a Blender, nagy léptékű futtatásához renderelési feladatok az Azure-ban.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

* 3D modellezés
* Vizuális FX (VFX) megjelenítési
* Videó átkódolása
* Képfeldolgozás, szín javítása és átméretezése

## <a name="architecture"></a>Architektúra

![A következő összetevők kapnak szerepet egy Azure Batch segítségével a Felhőbeli natív HPC-megoldást architektúrájának áttekintése][architecture]

Ebben a forgatókönyvben egy munkafolyamatot, amely használja az Azure Batch jeleníti meg. Az adatok folyamatok alábbiak szerint:

1. Töltse fel a bemeneti fájlok és feldolgozása ezeket a fájlokat az Azure Storage-fiókot az alkalmazásokkal.
2. Hozzon létre Batch-készlet, számítási csomópontok Batch-fiókban, egy feladatot, amely a készlethez, valamint a feladat a számítási feladatok futtatásához.
3. Töltse le a bemeneti fájlok és a Batch az alkalmazásokat.
4. Figyelheti a feladat a végrehajtás.
5. Tevékenység kimenetének feltöltése.
6. Kimeneti fájlok letöltése.

A folyamat leegyszerűsítése érdekében is használhatja a [Batch beépülő modulok, a Maya és a 3ds Max][batch-plugins]

### <a name="components"></a>Összetevők

Az Azure Batch-listaalkalmazásra épül a következő Azure-technológiák:

* [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) a fő csomópontot és a számítási erőforrásokat is használhatók.
* [Az Azure Storage-fiókok](/azure/storage/common/storage-introduction) szinkronizálás és az adatmegőrzést szolgálnak.
* [Virtual Machine Scale Sets] [ vmss] CycleCloud számítási erőforrásokat használ.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="machine-sizes-available-for-azure-batch"></a>Gépek méretei az Azure Batch számára elérhető

Renderelési a legtöbb ügyfél fog válassza ki a magas CPU-teljesítmény-erőforrásokat, miközben más számítási feladatok, a virtual machine scale sets használatával eltérően előfordulhat, hogy válassza ki virtuális gépeket, és számos tényezőtől függ:

* Van kötve a memória a futtatni az alkalmazást?
* Az alkalmazás gpu-k használatához nem kell? 
* A feladat típusok zavaróan párhuzamos vagy infiniband kapcsolatot igényelnek a szorosan összekapcsolt feladatokhoz?
* Gyors i/o számítási csomópontok tároló eléréséhez szükséges.

Egy számos különböző Virtuálisgép-méretek, amely az alkalmazás a fenti követelmények minden egy az Azure rendelkezik, néhány csak az adott HPC, de még a legkisebb méretű, amellyel egy hatékony rács implementáció:

* [HPC-Virtuálisgép-méretek] [ compute-hpc] renderelési kötött CPU jellege miatt általában javasolt az Azure H-sorozatú virtuális gépek. Az ilyen típusú virtuális gép kifejezetten a magas szintű számítási igényekre épül, 8 és 16 magos vCPU-mérettel érhető el, és szolgáltatások esetében DDR4 memóriával, ideiglenes SSD-tárolóval és a technológia az Intel Haswell E5 rendelkeznek.
* [GPU-Virtuálisgép-méretek] [ compute-gpu] GPU-optimalizált virtuális gépek méretek a következők specializált virtuális gépek egy vagy több NVIDIA gpu-k használatával érhető el. Ezeket a méreteket képi megjelenítés, nagy számítási igényű és magas grafikai igényű számítási feladatokhoz tervezték.
* Hálózati vezérlő, NCv2, az NCv3 és ND méretek nagy számítási és hálózatigényű alkalmazásokra és algoritmusokra, beleértve a CUDA és OpenCL-alapú alkalmazásokat és szimulációkat, mesterséges Intelligencia és a Deep Learning vannak optimalizálva. NV-méretek vannak kialakítva és optimalizálva távoli képi megjelenítés, streamelési, játék, kódolási és VDI-forgatókönyvekhez OpenGL, DirectX és hasonló keretrendszereket.
* [Memóriahasználatra optimalizált Virtuálisgép-méretek] [ compute-memory] több memóriára szükség, amikor a memóriahasználatra optimalizált Virtuálisgép-méretek kínál a nagyobb memória – Processzor memóriaarányt.
* [Általános célú virtuális gépek méreteit] [ compute-general] General-purpose Virtuálisgép-méretek is elérhetők, és adja meg a kiegyensúlyozott Processzor-memória arány.

### <a name="alternatives"></a>Alternatív megoldások

Ha jobban szabályozhatja az Azure-ban a renderelési környezet igényelnek, vagy egy hibrid megoldás kell, majd CycleCloud számítástechnika segítségével összehangolhatja a felhőben egy IaaS rács. Azure Batch a mögöttes Azure technológiákat használ, így létrehozását és karbantartását az IaaS rács hatékonyabbá teszi a folyamatot. További, és ismerje meg a tervezési alapelveket használja a következő hivatkozásra:

Az Azure-ban elérhető összes HPC-megoldások teljes körű áttekintést lásd a cikk [HPC, Batch és Big Compute megoldások az Azure virtuális gépekkel][hpc-alt-solutions]

### <a name="availability"></a>Rendelkezésre állás

Az Azure Batch-összetevők figyelésének, szolgáltatások, eszközök és API-k széles keresztül érhető el. Figyelés a következő cikkben található a [figyelő Batch-megoldások] [ batch-monitor] cikk.

### <a name="scalability"></a>Méretezhetőség

Belül egy fiók is, vagy a méretezési csoport manuális intézkedés révén, vagy egy képlet alapján az Azure Batch-mérőszámok, az Azure Batch Pools automatikusan skálázhatók. Méretezhetőség további információkért tekintse meg a cikket [hozzon létre egy Batch-készletben lévő csomópontok méretezése egy automatikus méretezési képlet][batch-scaling].

### <a name="security"></a>Biztonság

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Bár az Azure Batch szolgáltatásban jelenleg nem nincs feladatátvételi képességének, javasolt egy nem tervezett kimaradás esetén a rendelkezésre állás biztosítása érdekében az alábbi lépéseket követve:

* Egy másik Azure-beli helyen, egy másik tárfiókkal az Azure Batch-fiók létrehozása
* Az azonos csomópontkészletek ugyanazzal a névvel, nulla lefoglalt csomópontok létrehozása
* Győződjön meg, hogy az alkalmazások létrehozása és a másodlagos tárfiók frissítése
* Bemeneti fájlok feltöltése és a másodlagos Azure-Batch-fiók-feladatok elküldése

## <a name="deploy-this-scenario"></a>Ez a forgatókönyv megvalósítható

### <a name="creating-an-azure-batch-account-and-pools-manually"></a>Azure Batch-fiók és -készletek manuális létrehozása

Ebben a forgatókönyvben be az Azure Batch működése közben az Azure Batch Labs példaként Szolgáltatottszoftver-megoldás, amely az ügyfelek fejlesztette ki, amely azt mutatja be:

[Az Azure Batch Masterclass][batch-labs-masterclass]

### <a name="deploying-the-example-scenario-using-an-azure-resource-manager-template"></a>A példa egy Azure Resource Manager-sablonnal üzembe helyezése

A sablon telepíti:

* Egy új Azure Batch-fiók
* Storage-fiók
* A batch-fiókhoz társított csomópontkészletek
* A csomópont készlethez Canonical Ubuntu-lemezképekkel A2 v2 virtuális gép használatára lesz konfigurálva
* A csomópont készlethez virtuális gépek kezdetben tartalmazni fog, és el kell azt manuálisan hozzáadni a virtuális gépek méretezési

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

[További tudnivalók a Resource Manager-sablonok][azure-arm-templates]

## <a name="pricing"></a>Díjszabás

Azure Batch használatának költségét a készleteket, és mennyi ideig vannak lefoglalva a virtuális gépek által használt Virtuálisgép-méretek függ, és ott futtatja, nem egy Azure Batch-fiók létrehozása társított költségek nélkül. Tárolás és adatkezelés kimenő figyelembe kell venni, mivel ezek további költségekkel is érvényes lesz.

Az alábbi parancsok példák, amelyek használatával a kiszolgálókat különböző számú 8 órán belül befejeződik egy feladat sikerült felmerülő költségek:

* Nagy teljesítményű CPU 100 virtuális: [Költségbecslés][hpc-est-high]

  100 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom

* 50 nagy teljesítményű Processzorral virtuális: [Költségbecslés][hpc-est-med]

  50 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom

* 10, nagy teljesítményű CPU-alapú virtuális gépből: [Költségbecslés][hpc-est-low]

  10 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom

### <a name="pricing-for-low-priority-vms"></a>Alacsony prioritású virtuális gépek díjszabása

Az Azure Batch alacsony prioritású virtuális gépek használatát is támogatja a potenciálisan megadhat egy jelentős megtakarítás csomópontkészletek. További információk, beleértve a standard szintű virtuális gépek és az alacsony prioritású virtuális gépek, ár összehasonlítását: [Azure Batch szolgáltatás díjszabása][batch-pricing].

> [!NOTE] 
> Alacsony prioritású virtuális gépek csak olyan, megfelelő az egyes alkalmazások és számítási feladatok.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

[Az Azure Batch áttekintése][batch-overview]

[Az Azure Batch – dokumentáció][batch-doc]

[Tárolók az Azure Batch használatával][batch-containers]

<!-- links -->
[architecture]: ./media/architecture-video-rendering.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job