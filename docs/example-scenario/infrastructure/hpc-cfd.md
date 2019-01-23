---
title: CFD szimuláció futtatása
titleSuffix: Azure Example Scenarios
description: Számítási folyadékdinamikai (CFD) szimulációkat hajthat végre az Azure-ban.
author: mikewarr
ms.date: 09/20/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
ms.openlocfilehash: 0bd0590ccc2975481e23ac2154c4f0998e1f0877
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488477"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a>Számítási folyadékdinamikai (CFD) szimulációk futtatása az Azure-ban

Folyadékdinamika (CFD) számítási szimulációkat szükséges speciális hardver-és jelentős számítási időért. Ahogy fürt használati növekszik, szimuláció időpontokat, és átfogó grid használata növekszik, a tartalék kapacitás és a hosszú várakozási idők problémák vezető. Fizikai hardver hozzáadása költséges lehet, és előfordulhat, hogy nem igazodnak a használati mutatkozó csúcsokhoz és völgyekhez, amely egy üzleti halad át. Az Azure előnyeit kihasználva számos, ezek a kihívások szükségességéből nem jár munkavállalónkénti együtt.

Az Azure GPU- és CPU virtuális gépeken a CFD feladatok végrehajtásához szükséges hardverek biztosít. RDMA (Remote Direct Memory Access) engedélyezett Virtuálisgép-méretek FDR InfiniBand-alapú hálózat, ami lehetővé teszi a kis késleltetésű kommunikációt MPI (Message Passing Interface) rendelkezik. A Avere vFXT, amely egy vállalati szintű fürtözött fájlrendszert biztosít, kombinálva ügyfelek biztosítható a maximális átviteli sebesség az olvasási műveletek az Azure-ban.

Egyszerűbb létrehozását, felügyeleti és HPC-fürtök optimalizálása, Azure CycleCloud-fürtök kiépítése és adatok felhőbeli és hibrid forgatókönyvekben is használható. A függőben lévő feladatok figyelésével, CycleCloud automatikusan elindul igény szerinti számítási erőforrásokat, ahol csak fizetnie kell fizetni, a számítási feladatok a scheduler a választott csatlakozik.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb releváns ágazatok CFD alkalmazásokhoz a következők:

- Repüléstechnikai
- Autóipar
- HVAC létrehozása
- Olaj és gáz
- Élettudományok

## <a name="architecture"></a>Architektúra

![Architektúradiagram][architecture]

Az ábrán egy tipikus hibrid kialakítással és feladat figyelése az Azure-beli igény szerinti csomópontok biztosít magas szintű áttekintését:

1. Csatlakozzon a fürt konfigurálása az Azure CycleCloud kiszolgálóhoz.
2. Állítsa be, és hozzon létre a fürt fő csomópontjának engedélyezve RDMA gépek MPI használatával.
3. Adja hozzá, és konfigurálja a helyszíni fő csomópontot.
4. Ha rendelkezésre áll elegendő erőforrás, Azure CycleCloud skálázza fel (vagy le) a számítási erőforrásokat az Azure-ban. Egy előre meghatározott korlát, hogy foglalási keresztül lehet definiálni.
5. A végrehajtás csomópontok számára kiosztott feladatokat.
6. Az Azure-ban a helyszíni NFS-kiszolgáló a gyorsítótárazott adatok.
7. Az Azure cache szolgáltatáshoz a Avere vFXT beolvasott adatokat.
8. Az Azure CycleCloud kiszolgálóhoz továbbítón keresztüli feladatok és tevékenységek adatokat.

### <a name="components"></a>Összetevők

- [Az Azure CycleCloud] [ cyclecloud] egy eszköz létrehozását, kezelését, üzemeltetési és optimalizálása az Azure-beli HPC és Big Compute-fürtöket.
- [Az Azure-ban Avere vFXT] [ avere] egy felhőbeli felhasználásra készült vállalati szintű fürtözött fájlrendszert biztosít.
- [Azure-beli virtuális gépek (VM)] [ vms] hozhatók létre a számítási példányok statikus készletét.
- [Virtual Machine Scale Sets (virtuálisgép-méretezési)] [ vmss] biztosítanak azonos virtuális gépek képes Azure CycleCloud szerint a felfelé és lefelé skálázva folyamatban van.
- [Az Azure Storage-fiókok](/azure/storage/common/storage-introduction) szinkronizálás és az adatmegőrzést szolgálnak.
- [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) engedélyezése a számos különböző típusú Azure-erőforrások, például az Azure Virtual Machines (VM), hogy biztonságosan kommunikálhassanak egymással, az internetes és a helyszíni hálózatok.

### <a name="alternatives"></a>Alternatív megoldások

Ügyfelek is használhatják Azure CycleCloud rács létrehozása teljes egészében az Azure-ban. Ezzel a beállítással az Azure CycleCloud kiszolgálón fut az Azure-előfizetésen belül.

Ahol nincs szükség a számítási feladatok Scheduler felügyeleti, a modern alkalmazás megközelítésre [Azure Batch] [ batch] segítségével. Az Azure Batch hatékonyan futtathat nagy méretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat a felhőben. Az Azure Batch lehetővé teszi, hogy meghatározza az Azure számítási erőforrásokat az alkalmazások párhuzamos és nagy mennyiségű manuális konfigurálás vagy infrastruktúra kezelése nélkül. Az Azure Batch ütemezi a nagy számítási igényű feladatokat és dinamikusan hozzáadja és eltávolítja a számítási erőforrások igényei alapján.

### <a name="scalability-and-security"></a>Méretezhetőség és a biztonság

Méretezés az Azure CycleCloud a csomópontok lehetnek execute érhető el vagy manuális vagy automatikus skálázás használatával. További információkért lásd: [CycleCloud automatikus skálázás][cycle-scale].

Általános megoldások biztonságos, tekintse át a [az Azure security dokumentációja][security].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

### <a name="prerequisites"></a>Előfeltételek

A Resource Manager-sablon üzembe helyezése előtt kövesse az alábbi lépéseket:

1. Hozzon létre egy [szolgáltatásnév] [ cycle-svcprin] az appId, displayName, name, jelszót és bérlői beolvasásakor.
2. Hozzon létre egy [SSH-kulcspár] [ cycle-ssh] való biztonságos bejelentkezéshez a CycleCloud kiszolgáló.

    <!-- markdownlint-disable MD033 -->

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
        <img src="https://azuredeploy.net/deploybutton.png"/>
    </a>

    <!-- markdownlint-enable MD033 -->

3. [Jelentkezzen be a CycleCloud kiszolgáló] [ cycle-login] konfigurálását, és hozzon létre egy új fürtöt.
4. [Hozzon létre egy fürtöt][cycle-create].

A Avere gyorsítótár nem egy nem kötelező megoldás, amely jelentősen növelheti a olvasási teljesítménye a feladat az alkalmazásadatok számára. Avere vFXT Azure megoldja a problémát, ezek vállalati HPC-alkalmazások a felhőben futó költséghatékonyhatékonyságából, tárolt adatok a helyszíni vagy Azure Blob storage-ban.

A szervezet számára, amely a helyszíni tárolók és a felhő-számítástechnika a hibrid infrastruktúra tervezéséhez HPC-alkalmazások is "burst" igény szerint NAS-eszközökön és a virtuális processzorok elindítása tárolt adatok Azure-bA. Az adatkészlet soha nem teljesen áthelyezése a felhőbe. A kért bájtok átmenetileg gyorsítótárazza a rendszer Avere fürtöt használ a feldolgozás során.

Beállítását, és a egy Avere vFXT telepítését konfigurálni, kövesse a [Avere beállítási és konfigurációs útmutató][avere].

## <a name="pricing"></a>Díjszabás

Egy HPC-implementációhoz CycleCloud kiszolgáló használatával futtatásával járó költségeket számos tényezőtől függően változhat. Ha például CycleCloud használt, a fő- és CycleCloud kiszolgáló foglalva és futnak általában alatt folyamatosan a számítási idő mennyisége alapján kell fizetnie. Az Execute csomópontok függ, milyen hosszú az alábbiak fel, valamint milyen méretű rendszert futtat, a költség szolgál. A normál Azure tárolási és hálózatkezelési is keletkezhet díjfizetési kötelezettség.

Ebből a forgatókönyvből megtudhatja, hogyan CFD alkalmazások is futtathatók az Azure-ban, így a gépek szükséges RDMA funkciója, amely csak az adott Virtuálisgép-méretek érhető el. A következő példák, amely folyamatosan lefoglalt nyolc óra / nap egy hónapban, az 1 TB kimenő adatforgalom a méretezési csoport esetében sikerült felmerülő költségek. Ezenkívül tartalmazza az Azure CycleCloud kiszolgáló-és az Azure-telepítés Avere vFXT:

- Régió: Észak-Európa
- Azure CycleCloud Server: Standard D3 x 1 (4 x Standard HDD processzor, 14 GB memória, 32 GB)
- Azure CycleCloud Master Server: Standard D12 v (4 x processzor, 28 GB memória, Standard HDD 32 GB) x 1
- Azure CycleCloud Node Array: Standard H16r x 10 (16 x processzor, 112 GB memória)
- Az Azure-fürtön vFXT Avere: 3 x D16s v3 (200 GB-os OS, 1 TB méretű adatlemezzel prémium szintű SSD)
- Kimenő adatforgalom: 1 TB

Tekintse át ezt [a becsült ár] [ pricing] a fent felsorolt hardver.

## <a name="next-steps"></a>Következő lépések

Miután üzembe helyezte a mintát, tudjon meg többet [Azure CycleCloud][cyclecloud].

## <a name="related-resources"></a>Kapcsolódó erőforrások

- [RDMA-kompatibilis virtuálisgép-példányok][rdma]
- [Egy RDMA-példány virtuális gép testreszabása][rdma-custom]

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
