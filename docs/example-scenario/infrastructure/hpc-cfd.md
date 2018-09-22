---
title: Azure-on futó numerikus áramlástani (CFD)
description: Mintául szolgáló megoldás leíró futtatásáról számítási Folyadékdinamika (CFD) az Azure-ban
author: mikewarr
ms.date: 09/20/2018
ms.openlocfilehash: e765ca0e44ddb2ea4caa91afe4de675a16b43d35
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534550"
---
# <a name="running-computational-fluid-dynamics-cfd-on-azure"></a>Azure-on futó numerikus áramlástani (CFD)

Folyadékdinamika (CFD) számítási szimulációkat szükséges speciális hardver együtt egy jelentős számítási időért. Ahogy fürt használati növekszik, szimuláció időpontokat, és átfogó grid használata növekszik, a tartalék kapacitás és a hosszú várakozási idők problémák vezető. További fizikai hardver hozzáadása költséges lehet, és előfordulhat, hogy nem igazodnak a használati csúcsok és völgyekhez üzleti halad át. Az Azure előnyeit kihasználva számos, ezek a kihívások szükségességéből, bármely tőkeberuházás nélkül.

Az Azure GPU- és CPU virtuális gépeken a CFD feladatok végrehajtásához szükséges hardverek biztosít. RDMA (Remote Direct Memory Access) engedélyezve van a virtuális gépek mérete közel valós idejű MPI (Message Passing Interface) kommunikációt lehetővé tevő FDR InfiniBand-alapú hálózati rendelkezik. A Avere vFXT, amely egy vállalati szintű fürtözött fájlrendszert biztosít, kombinálva ügyfelek biztosítható a maximális átviteli sebesség az olvasási műveletek az Azure-ban.

Egyszerűbb létrehozását, felügyeleti és HPC-fürtök optimalizálása, Azure CycleCloud-fürtök kiépítése és adatok felhőbeli és hibrid forgatókönyvekben is használható. A függőben lévő feladatok figyelésével, CycleCloud automatikusan elindul igény szerinti számítási erőforrásokat, ahol csak fizetnie kell fizetni, a számítási feladatok a scheduler a választott csatlakozik.

## <a name="potential-use-cases"></a>A lehetséges alkalmazási helyzetek

Ebben a forgatókönyvben ezek iparágakban, ahol CFD alkalmazások használható, vegye figyelembe:

* Repüléstechnikai
* Autóipar
* HVAC létrehozása
* Olaj- és gázipar
* Élettudományok

## <a name="architecture"></a>Architektúra

![helyettesítő szöveg][cyclearch]

Az ábrán egy tipikus hibrid kialakítással és feladat figyelése az Azure-beli igény szerinti csomópontok biztosít magas szintű áttekintést nyújt:

1. Csatlakozzon az Azure CycleCloud kiszolgálóhoz a fürt konfigurálása
2. Konfigurálása és létrehozása a fürt fő csomópontjának engedélyezve RDMA gépek MPI használatával 
3. Felveheti és konfigurálhatja a helyszíni fő csomópontja
4. Ha rendelkezésre áll elegendő erőforrás, Azure CycleCloud skálázza fel (vagy le) a számítási erőforrásokat az Azure-ban. Egy előre meghatározott korlát, hogy foglalási keresztül lehet definiálni.
5. A végrehajtás csomópontok feladatai
6. Az Azure-ban a helyszíni NFS-kiszolgáló a gyorsítótárazott adatok
7. Az Azure cache szolgáltatáshoz a Avere vFXT beolvasott adatok
8. Az Azure CycleCloud kiszolgálóhoz továbbítón keresztüli feladatok és tevékenységek információk


### <a name="components"></a>Összetevők

* [Az Azure CycleCloud] [ cyclecloud] egy eszköz létrehozását, kezelését, üzemeltetési és az Azure-beli HPC és Big Compute fürtök optimalizálása
* [Az Azure-ban Avere vFXT] [ avere] egy felhőbeli felhasználásra készült vállalati szintű fürtözött fájlrendszert biztosít
* [Virtual Machine Scale Sets (virtuálisgép-méretezési)] [ vmss] biztosítanak azonos virtuális gépek képes Azure CycleCloud szerint a felfelé és lefelé skálázva folyamatban
* Használat [virtuális gépek] [ vms] egy statikus számítási példányok létrehozása
* [Tárolási] [ storage] fiókok használhatók szinkronizálási és adatmegőrzés
* [Virtuális hálózat] [ vnet] Azure Virtual Network lehetővé teszi számos különböző típusú Azure-erőforrások, például az Azure Virtual Machines (VM), biztonságosan kommunikálnak egymással, az internethez, és a helyszíni hálózatokkal

### <a name="alternatives"></a>Alternatív megoldások

Ügyfelek is használhatják Azure CycleCloud rács létrehozása teljes egészében az Azure-ban.  Ezzel a beállítással az Azure CycleCloud kiszolgálón fut az Azure-előfizetésen belül.

Ahol nincs szükség a számítási feladatok Scheduler felügyeleti, a modern alkalmazás megközelítésre [Azure Batch] [ batch] segítségével. Az Azure Batch hatékonyan futtathat nagy méretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat a felhőben. Az Azure batch lehetővé teszi, hogy a végrehajtása manuális konfigurálás vagy infrastruktúra kezelése nélkül az alkalmazások párhuzamos és nagy mennyiségű Azure-beli számítási erőforrásokat határoz meg. A Batch ütemezi a nagy számítási igényű feladatokat és dinamikusan hozzáadja és igényei alapján a számítási erőforrások eltávolítása

### <a name="scalability-and-security"></a>Méretezhetőség és a biztonság

Tekintse meg az execute Azure CycleCloud csomópont is elvégezhető, manuális vagy automatikus skálázás, a méretezés [CycleCloud automatikus skálázás][cycle-scale]

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

## <a name="deploy-this-scenario"></a>Ez a forgatókönyv megvalósítható

Az Azure-beli üzembe helyezése előtt bizonyos Előfeltételek teljesülésére szükség, futtassa a fenti lépéseket a Resource Manager-sablon üzembe helyezése előtt:
1. [Egyszerű szolgáltatás] [ cycle-svcprin] az appId, displayname, jelszót és bérlői lekéréséhez
2. [SSH-kulcspár] [ cycle-ssh] való biztonságos bejelentkezéshez a CycleCloud kiszolgáló

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

3. [Jelentkezzen be a CycleCloud kiszolgáló] [ cycle-login] konfigurálásához és a egy új fürt létrehozása
4. [Fürt létrehozása][cycle-create] 

A Avere gyorsítótár nem egy nem kötelező megoldást, amely jelentősen felgyorsíthatja az alkalmazásadatok feladat olvasás átviteli sebességét. Avere vFXT Azure megoldja a problémát, ezek vállalati HPC-alkalmazások a felhőben futó költséghatékonyhatékonyságából, tárolt adatok a helyszíni vagy Azure Blob storage-ban.

A szervezet számára, amely a helyszíni tárolók és a felhő-számítástechnika a hibrid infrastruktúra helyzet tervez HPC-alkalmazások is "burst"-adatok tárolva NAS-eszközökön, és szükség szerint révén tetszőleges számú virtuális processzort elindítása az Azure-bA. Maga az adathalmaz teljesen soha nem helyezi át a felhőbe. A kért bájtok átmenetileg gyorsítótárazza a rendszer feldolgozása közben Avere fürtöt használ.

Beállítását, és a egy Avere vFXT telepítését konfigurálni, kövesse a [Avere beállítás és konfiguráció] [ avere] útmutató.

## <a name="pricing"></a>Díjszabás

Egy HPC-implementációhoz CycleCloud kiszolgáló használatával futtatásával járó költségeket számos tényezőtől függően eltérőek lehetnek. Ha például CycleCloud használt, a fő- és CycleCloud kiszolgáló foglalva és futnak általában alatt folyamatosan a számítási idő mennyisége alapján kell fizetnie. A végrehajtás csomópontok futtatásával járó költségeket függ, milyen hosszú az alábbiak helyezheti üzembe a emellett mérete szolgál. A normál Azure tárolási és hálózatkezelési is keletkezhet díjfizetési kötelezettség.  

Ez a forgatókönyv célja bemutatják, hogyan CFD alkalmazások is futtathatók az Azure-ban, így a gépek szükséges RDMA funkciója, amely csak az adott Virtuálisgép-méretek érhető el. Az alábbi parancsok példák egy méretezési csoport egy hónapban, az 1 TB-nyi adatot, egy kimenő forgalmi napi 8 órán folyamatosan lefoglalt sikerült felmerülő költségek. Ezenkívül tartalmazza a díjszabás az Azure CycleCloud server és az Azure-telepítés Avere vFXT:

* Régióban: Észak-Európa
* Azure CycleCloud Server: Standard D3 x 1 (4 x Standard HDD processzor, 14 GB memória, 32 GB)
* Az Azure CycleCloud főkiszolgáló: 1 x Standard D12 v (4 x processzor, 28 GB memória, Standard HDD 32 GB)
* Az Azure CycleCloud csomópont tömb: 10 Standard H16r x (16 x processzor, 112 GB memória)
* Az Azure-fürtön Avere vFXT: 3 x D16s v3 (200 GB-os OS, 1 TB méretű adatlemezzel prémium szintű SSD)
* Kimenő adatforgalom: 1 TB-ot

A következő hivatkozás használatával ellenőrizze a [a becsült ár] [ pricing] a fenti hardver

## <a name="next-steps"></a>További lépések

Miután üzembe helyezte a mintát, ismerje meg bővebben az Azure CycleCloud a [Azure CycleCloud dokumentációja][cyclecloud]

## <a name="related-resources"></a>Kapcsolódó erőforrások

* [RDMA-kompatibilis virtuálisgép-példányok][rdma]
* [Egy RDMA-példány virtuális gép testreszabása][rdma-custom]



<!-- links -->
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[cyclearch]: media/Hybrid-HPC-Ref-Arch.png
[vmss]: https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/overview
[cyclecloud]: https://docs.microsoft.com/en-us/azure/cyclecloud/
[rdma]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-hpc
[vms]: https://docs.microsoft.com/en-us/azure/virtual-machines/
[storage]: https://azure.microsoft.com/services/storage/
[low-pri]: https://docs.microsoft.com/en-ca/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: https://docs.microsoft.com/en-us/azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[vnet]: https://docs.microsoft.com/en-us/azure/virtual-network/
[cycle-prereq]: https://docs.microsoft.com/en-us/azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: https://docs.microsoft.com/en-us/azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: https://docs.microsoft.com/en-us/azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: https://docs.microsoft.com/en-us/azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: https://docs.microsoft.com/en-us/azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: https://docs.microsoft.com/en-us/azure/cyclecloud/autoscale
