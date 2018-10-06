---
title: Azure-ban (CFD) szimuláció futtatja a numerikus áramlástani
description: Hajtsa végre a numerikus áramlástani (CFD) szimulációk Azure-ban.
author: mikewarr
ms.date: 09/20/2018
ms.openlocfilehash: 5734e6fe707e3beb5e23f2ad2b4344ba289803bb
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818575"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a><span data-ttu-id="4abf5-103">Azure-ban (CFD) szimuláció futtatja a numerikus áramlástani</span><span class="sxs-lookup"><span data-stu-id="4abf5-103">Running computational fluid dynamics (CFD) simulations on Azure</span></span>

<span data-ttu-id="4abf5-104">Folyadékdinamika (CFD) számítási szimulációkat szükséges speciális hardver-és jelentős számítási időért.</span><span class="sxs-lookup"><span data-stu-id="4abf5-104">Computational Fluid Dynamics (CFD) simulations require significant compute time along with specialized hardware.</span></span> <span data-ttu-id="4abf5-105">Ahogy fürt használati növekszik, szimuláció időpontokat, és átfogó grid használata növekszik, a tartalék kapacitás és a hosszú várakozási idők problémák vezető.</span><span class="sxs-lookup"><span data-stu-id="4abf5-105">As cluster usage increases, simulation times and overall grid use grow, leading to issues with spare capacity and long queue times.</span></span> <span data-ttu-id="4abf5-106">Fizikai hardver hozzáadása költséges lehet, és előfordulhat, hogy nem igazodnak a használati mutatkozó csúcsokhoz és völgyekhez, amely egy üzleti halad át.</span><span class="sxs-lookup"><span data-stu-id="4abf5-106">Adding physical hardware can be expensive, and may not align to the usage peaks and valleys that a business goes through.</span></span> <span data-ttu-id="4abf5-107">Az Azure előnyeit kihasználva számos, ezek a kihívások szükségességéből nem jár munkavállalónkénti együtt.</span><span class="sxs-lookup"><span data-stu-id="4abf5-107">By taking advantage of Azure, many of these challenges can be overcome with no capital expenditure.</span></span>

<span data-ttu-id="4abf5-108">Az Azure GPU- és CPU virtuális gépeken a CFD feladatok végrehajtásához szükséges hardverek biztosít.</span><span class="sxs-lookup"><span data-stu-id="4abf5-108">Azure provides the hardware you need to run your CFD jobs on both GPU and CPU virtual machines.</span></span> <span data-ttu-id="4abf5-109">RDMA (Remote Direct Memory Access) engedélyezett Virtuálisgép-méretek FDR InfiniBand-alapú hálózat, ami lehetővé teszi a kis késleltetésű kommunikációt MPI (Message Passing Interface) rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="4abf5-109">RDMA (Remote Direct Memory Access) enabled VM sizes have FDR InfiniBand-based networking which allows for low latency MPI (Message Passing Interface) communication.</span></span> <span data-ttu-id="4abf5-110">A Avere vFXT, amely egy vállalati szintű fürtözött fájlrendszert biztosít, kombinálva ügyfelek biztosítható a maximális átviteli sebesség az olvasási műveletek az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="4abf5-110">Combined with the Avere vFXT, which provides an enterprise-scale clustered file system, customers can ensure maximum throughput for read operations in Azure.</span></span>

<span data-ttu-id="4abf5-111">Egyszerűbb létrehozását, felügyeleti és HPC-fürtök optimalizálása, Azure CycleCloud-fürtök kiépítése és adatok felhőbeli és hibrid forgatókönyvekben is használható.</span><span class="sxs-lookup"><span data-stu-id="4abf5-111">To simplify the creation, management, and optimization of HPC clusters, Azure CycleCloud can be used to provision clusters and orchestrate data in both hybrid and cloud scenarios.</span></span> <span data-ttu-id="4abf5-112">A függőben lévő feladatok figyelésével, CycleCloud automatikusan elindul igény szerinti számítási erőforrásokat, ahol csak fizetnie kell fizetni, a számítási feladatok a scheduler a választott csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="4abf5-112">By monitoring the pending jobs, CycleCloud will automatically launch on-demand compute, where you only pay for what you use, connected to the workload scheduler of your choice.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="4abf5-113">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="4abf5-113">Relevant use cases</span></span>

<span data-ttu-id="4abf5-114">Ebben a forgatókönyvben ezek iparágakban, ahol CFD alkalmazások használható, vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="4abf5-114">Consider this scenario for these industries where CFD applications could be used:</span></span>

* <span data-ttu-id="4abf5-115">Repüléstechnikai</span><span class="sxs-lookup"><span data-stu-id="4abf5-115">Aeronautics</span></span>
* <span data-ttu-id="4abf5-116">Autóipar</span><span class="sxs-lookup"><span data-stu-id="4abf5-116">Automotive</span></span>
* <span data-ttu-id="4abf5-117">HVAC létrehozása</span><span class="sxs-lookup"><span data-stu-id="4abf5-117">Building HVAC</span></span>
* <span data-ttu-id="4abf5-118">Olaj és gáz</span><span class="sxs-lookup"><span data-stu-id="4abf5-118">Oil and gas</span></span>
* <span data-ttu-id="4abf5-119">Élettudományok</span><span class="sxs-lookup"><span data-stu-id="4abf5-119">Life sciences</span></span>

## <a name="architecture"></a><span data-ttu-id="4abf5-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="4abf5-120">Architecture</span></span>

![Architektúradiagram][architecture]

<span data-ttu-id="4abf5-122">Az ábrán egy tipikus hibrid kialakítással és feladat figyelése az Azure-beli igény szerinti csomópontok biztosít magas szintű áttekintését:</span><span class="sxs-lookup"><span data-stu-id="4abf5-122">This diagram shows a high-level overview of a typical hybrid design providing job monitoring of the on-demand nodes in Azure:</span></span>

1. <span data-ttu-id="4abf5-123">Csatlakozzon a fürt konfigurálása az Azure CycleCloud kiszolgálóhoz.</span><span class="sxs-lookup"><span data-stu-id="4abf5-123">Connect to the Azure CycleCloud server to configure the cluster.</span></span>
2. <span data-ttu-id="4abf5-124">Állítsa be, és hozzon létre a fürt fő csomópontjának engedélyezve RDMA gépek MPI használatával.</span><span class="sxs-lookup"><span data-stu-id="4abf5-124">Configure and create the cluster head node, using RDMA enabled machines for MPI.</span></span>
3. <span data-ttu-id="4abf5-125">Adja hozzá, és konfigurálja a helyszíni fő csomópontot.</span><span class="sxs-lookup"><span data-stu-id="4abf5-125">Add and configure the on-premises head node.</span></span>
4. <span data-ttu-id="4abf5-126">Ha rendelkezésre áll elegendő erőforrás, Azure CycleCloud skálázza fel (vagy le) a számítási erőforrásokat az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="4abf5-126">If there are insufficient resources, Azure CycleCloud will scale up (or down) compute resources in Azure.</span></span> <span data-ttu-id="4abf5-127">Egy előre meghatározott korlát, hogy foglalási keresztül lehet definiálni.</span><span class="sxs-lookup"><span data-stu-id="4abf5-127">A predetermined limit can be defined to prevent over allocation.</span></span>
5. <span data-ttu-id="4abf5-128">A végrehajtás csomópontok számára kiosztott feladatokat.</span><span class="sxs-lookup"><span data-stu-id="4abf5-128">Tasks allocated to the execute nodes.</span></span>
6. <span data-ttu-id="4abf5-129">Az Azure-ban a helyszíni NFS-kiszolgáló a gyorsítótárazott adatok.</span><span class="sxs-lookup"><span data-stu-id="4abf5-129">Data cached in Azure from on-premises NFS server.</span></span>
7. <span data-ttu-id="4abf5-130">Az Azure cache szolgáltatáshoz a Avere vFXT beolvasott adatokat.</span><span class="sxs-lookup"><span data-stu-id="4abf5-130">Data read in from the Avere vFXT for Azure cache.</span></span>
8. <span data-ttu-id="4abf5-131">Az Azure CycleCloud kiszolgálóhoz továbbítón keresztüli feladatok és tevékenységek adatokat.</span><span class="sxs-lookup"><span data-stu-id="4abf5-131">Job and task information relayed to the Azure CycleCloud server.</span></span>

### <a name="components"></a><span data-ttu-id="4abf5-132">Összetevők</span><span class="sxs-lookup"><span data-stu-id="4abf5-132">Components</span></span>

* <span data-ttu-id="4abf5-133">[Az Azure CycleCloud] [ cyclecloud] egy eszköz létrehozását, kezelését, üzemeltetési és optimalizálása az Azure-beli HPC és Big Compute-fürtöket.</span><span class="sxs-lookup"><span data-stu-id="4abf5-133">[Azure CycleCloud][cyclecloud] a tool for creating, managing, operating, and optimizing HPC and Big Compute clusters in Azure.</span></span>
* <span data-ttu-id="4abf5-134">[Az Azure-ban Avere vFXT] [ avere] egy felhőbeli felhasználásra készült vállalati szintű fürtözött fájlrendszert biztosít.</span><span class="sxs-lookup"><span data-stu-id="4abf5-134">[Avere vFXT on Azure][avere] is used to provide an enterprise-scale clustered file system built for the cloud.</span></span>
* <span data-ttu-id="4abf5-135">[Azure-beli virtuális gépek (VM)] [ vms] hozhatók létre a számítási példányok statikus készletét.</span><span class="sxs-lookup"><span data-stu-id="4abf5-135">[Azure Virtual Machines (VMs)][vms] are used to create a static set of compute instances.</span></span>
* <span data-ttu-id="4abf5-136">[Virtual Machine Scale Sets (virtuálisgép-méretezési)] [ vmss] biztosítanak azonos virtuális gépek képes Azure CycleCloud szerint a felfelé és lefelé skálázva folyamatban van.</span><span class="sxs-lookup"><span data-stu-id="4abf5-136">[Virtual Machine Scale Sets (virtual machine scale set)][vmss] provide a group of identical VMs capable of being scaled up or down by Azure CycleCloud.</span></span>
* <span data-ttu-id="4abf5-137">[Az Azure Storage-fiókok](/azure/storage/common/storage-introduction) szinkronizálás és az adatmegőrzést szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="4abf5-137">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
* <span data-ttu-id="4abf5-138">[Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) engedélyezése a számos különböző típusú Azure-erőforrások, például az Azure Virtual Machines (VM), hogy biztonságosan kommunikálhassanak egymással, az internetes és a helyszíni hálózatok.</span><span class="sxs-lookup"><span data-stu-id="4abf5-138">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) enable many types of Azure resources, such as Azure Virtual Machines (VMs), to securely communicate with each other, the internet, and on-premises networks.</span></span>

### <a name="alternatives"></a><span data-ttu-id="4abf5-139">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="4abf5-139">Alternatives</span></span>

<span data-ttu-id="4abf5-140">Ügyfelek is használhatják Azure CycleCloud rács létrehozása teljes egészében az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="4abf5-140">Customers can also use Azure CycleCloud to create a grid entirely in Azure.</span></span> <span data-ttu-id="4abf5-141">Ezzel a beállítással az Azure CycleCloud kiszolgálón fut az Azure-előfizetésen belül.</span><span class="sxs-lookup"><span data-stu-id="4abf5-141">In this setup, the Azure CycleCloud server is run within your Azure subscription.</span></span>

<span data-ttu-id="4abf5-142">Ahol nincs szükség a számítási feladatok Scheduler felügyeleti, a modern alkalmazás megközelítésre [Azure Batch] [ batch] segítségével.</span><span class="sxs-lookup"><span data-stu-id="4abf5-142">For a modern application approach where management of a workload scheduler is not needed, [Azure Batch][batch] can help.</span></span> <span data-ttu-id="4abf5-143">Az Azure Batch hatékonyan futtathat nagy méretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat a felhőben.</span><span class="sxs-lookup"><span data-stu-id="4abf5-143">Azure Batch can run large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="4abf5-144">Az Azure Batch lehetővé teszi, hogy meghatározza az Azure számítási erőforrásokat az alkalmazások párhuzamos és nagy mennyiségű manuális konfigurálás vagy infrastruktúra kezelése nélkül.</span><span class="sxs-lookup"><span data-stu-id="4abf5-144">Azure Batch allows you to define the Azure compute resources to execute your applications in parallel or at scale without manually configuring or managing infrastructure.</span></span> <span data-ttu-id="4abf5-145">Az Azure Batch ütemezi a nagy számítási igényű feladatokat és dinamikusan hozzáadja és eltávolítja a számítási erőforrások igényei alapján.</span><span class="sxs-lookup"><span data-stu-id="4abf5-145">Azure Batch schedules compute-intensive tasks and dynamically adds and removes compute resources based on your requirements.</span></span>

### <a name="scalability-and-security"></a><span data-ttu-id="4abf5-146">Méretezhetőség és a biztonság</span><span class="sxs-lookup"><span data-stu-id="4abf5-146">Scalability, and Security</span></span>

<span data-ttu-id="4abf5-147">Méretezés az Azure CycleCloud a csomópontok lehetnek execute érhető el vagy manuális vagy automatikus skálázás használatával.</span><span class="sxs-lookup"><span data-stu-id="4abf5-147">Scaling the execute nodes on Azure CycleCloud can be accomplished either manually or using autoscaling.</span></span> <span data-ttu-id="4abf5-148">További információkért lásd: [CycleCloud automatikus skálázás][cycle-scale].</span><span class="sxs-lookup"><span data-stu-id="4abf5-148">For more information, see [CycleCloud Autoscaling][cycle-scale].</span></span>

<span data-ttu-id="4abf5-149">Általános megoldások biztonságos, tekintse át a [az Azure security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="4abf5-149">For general guidance on designing secure solutions, see the [Azure security documentation][security].</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="4abf5-150">Ez a forgatókönyv megvalósítható</span><span class="sxs-lookup"><span data-stu-id="4abf5-150">Deploy this scenario</span></span>

<span data-ttu-id="4abf5-151">Az Azure-beli üzembe helyezése előtt bizonyos Előfeltételek teljesülésére szükség.</span><span class="sxs-lookup"><span data-stu-id="4abf5-151">Before deploying in Azure, some pre-requisites are required.</span></span> <span data-ttu-id="4abf5-152">A Resource Manager-sablon üzembe helyezése előtt kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4abf5-152">Follow these steps before deploying the Resource Manager template:</span></span>
1. <span data-ttu-id="4abf5-153">Hozzon létre egy [szolgáltatásnév] [ cycle-svcprin] az appId, displayName, name, jelszót és bérlői beolvasásakor.</span><span class="sxs-lookup"><span data-stu-id="4abf5-153">Create a [service principal][cycle-svcprin] for retrieving the appId, displayName, name, password, and tenant.</span></span>
2. <span data-ttu-id="4abf5-154">Hozzon létre egy [SSH-kulcspár] [ cycle-ssh] való biztonságos bejelentkezéshez a CycleCloud kiszolgáló.</span><span class="sxs-lookup"><span data-stu-id="4abf5-154">Generate an [SSH key pair][cycle-ssh] to sign in securely to the CycleCloud server.</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

3. <span data-ttu-id="4abf5-155">[Jelentkezzen be a CycleCloud kiszolgáló] [ cycle-login] konfigurálását, és hozzon létre egy új fürtöt.</span><span class="sxs-lookup"><span data-stu-id="4abf5-155">[Log into the CycleCloud server][cycle-login] to configure and create a new cluster.</span></span>
4. <span data-ttu-id="4abf5-156">[Hozzon létre egy fürtöt][cycle-create].</span><span class="sxs-lookup"><span data-stu-id="4abf5-156">[Create a cluster][cycle-create].</span></span>

<span data-ttu-id="4abf5-157">A Avere gyorsítótár nem egy nem kötelező megoldás, amely jelentősen növelheti a olvasási teljesítménye a feladat az alkalmazásadatok számára.</span><span class="sxs-lookup"><span data-stu-id="4abf5-157">The Avere Cache is an optional solution that can drastically increase read throughput for the application job data.</span></span> <span data-ttu-id="4abf5-158">Avere vFXT Azure megoldja a problémát, ezek vállalati HPC-alkalmazások a felhőben futó költséghatékonyhatékonyságából, tárolt adatok a helyszíni vagy Azure Blob storage-ban.</span><span class="sxs-lookup"><span data-stu-id="4abf5-158">Avere vFXT for Azure solves the problem of running these enterprise HPC applications in the cloud while leveraging data stored on-premises or in Azure Blob storage.</span></span>

<span data-ttu-id="4abf5-159">A szervezet számára, amely a helyszíni tárolók és a felhő-számítástechnika a hibrid infrastruktúra tervezéséhez HPC-alkalmazások is "burst" igény szerint NAS-eszközökön és a virtuális processzorok elindítása tárolt adatok Azure-bA.</span><span class="sxs-lookup"><span data-stu-id="4abf5-159">For organizations that are planning for a hybrid infrastructure with both on-premises storage and cloud computing, HPC applications can “burst” into Azure using data stored in NAS devices and spin up virtual CPUs as needed.</span></span> <span data-ttu-id="4abf5-160">Az adatkészlet soha nem teljesen áthelyezése a felhőbe.</span><span class="sxs-lookup"><span data-stu-id="4abf5-160">The data set is never moved completely into the cloud.</span></span> <span data-ttu-id="4abf5-161">A kért bájtok átmenetileg gyorsítótárazza a rendszer Avere fürtöt használ a feldolgozás során.</span><span class="sxs-lookup"><span data-stu-id="4abf5-161">The requested bytes are temporarily cached using an Avere cluster during processing.</span></span>

<span data-ttu-id="4abf5-162">Beállítását, és a egy Avere vFXT telepítését konfigurálni, kövesse a [Avere beállítási és konfigurációs útmutató][avere].</span><span class="sxs-lookup"><span data-stu-id="4abf5-162">To set up and configure an Avere vFXT installation, follow the [Avere Setup and Configuration guide][avere].</span></span>

## <a name="pricing"></a><span data-ttu-id="4abf5-163">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="4abf5-163">Pricing</span></span>

<span data-ttu-id="4abf5-164">Egy HPC-implementációhoz CycleCloud kiszolgáló használatával futtatásával járó költségeket számos tényezőtől függően változhat.</span><span class="sxs-lookup"><span data-stu-id="4abf5-164">The cost of running an HPC implementation using CycleCloud server will vary depending on a number of factors.</span></span> <span data-ttu-id="4abf5-165">Ha például CycleCloud használt, a fő- és CycleCloud kiszolgáló foglalva és futnak általában alatt folyamatosan a számítási idő mennyisége alapján kell fizetnie.</span><span class="sxs-lookup"><span data-stu-id="4abf5-165">For example, CycleCloud is charged by the amount of compute time that is used, with the Master and CycleCloud server typically being constantly allocated and running.</span></span> <span data-ttu-id="4abf5-166">Az Execute csomópontok függ, milyen hosszú az alábbiak fel, valamint milyen méretű rendszert futtat, a költség szolgál.</span><span class="sxs-lookup"><span data-stu-id="4abf5-166">The cost of running the Execute nodes will depend on how long these are up and running as well as what size is used.</span></span> <span data-ttu-id="4abf5-167">A normál Azure tárolási és hálózatkezelési is keletkezhet díjfizetési kötelezettség.</span><span class="sxs-lookup"><span data-stu-id="4abf5-167">The normal Azure charges for storage and networking also apply.</span></span>

<span data-ttu-id="4abf5-168">Ebből a forgatókönyvből megtudhatja, hogyan CFD alkalmazások is futtathatók az Azure-ban, így a gépek szükséges RDMA funkciója, amely csak az adott Virtuálisgép-méretek érhető el.</span><span class="sxs-lookup"><span data-stu-id="4abf5-168">This scenario shows how CFD applications can be run in Azure, so the machines will require RDMA functionality, which is only available on specific VM sizes.</span></span> <span data-ttu-id="4abf5-169">A következő példák, amely folyamatosan lefoglalt nyolc óra / nap egy hónapban, az 1 TB kimenő adatforgalom a méretezési csoport esetében sikerült felmerülő költségek.</span><span class="sxs-lookup"><span data-stu-id="4abf5-169">The following are examples of costs that could be incurred for a scale set that is allocated continuously for eight hours per day for one month, with data egress of 1 TB.</span></span> <span data-ttu-id="4abf5-170">Ezenkívül tartalmazza az Azure CycleCloud kiszolgáló-és az Azure-telepítés Avere vFXT:</span><span class="sxs-lookup"><span data-stu-id="4abf5-170">It also includes pricing for the Azure CycleCloud server and the Avere vFXT for Azure install:</span></span>

* <span data-ttu-id="4abf5-171">Régióban: Észak-Európa</span><span class="sxs-lookup"><span data-stu-id="4abf5-171">Region: North Europe</span></span>
* <span data-ttu-id="4abf5-172">Azure CycleCloud Server: Standard D3 x 1 (4 x Standard HDD processzor, 14 GB memória, 32 GB)</span><span class="sxs-lookup"><span data-stu-id="4abf5-172">Azure CycleCloud Server: 1 x Standard D3 (4 x CPUs, 14 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="4abf5-173">Az Azure CycleCloud főkiszolgáló: 1 x Standard D12 v (4 x processzor, 28 GB memória, Standard HDD 32 GB)</span><span class="sxs-lookup"><span data-stu-id="4abf5-173">Azure CycleCloud Master Server: 1 x Standard D12 v (4 x CPUs, 28 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="4abf5-174">Az Azure CycleCloud csomópont tömb: 10 Standard H16r x (16 x processzor, 112 GB memória)</span><span class="sxs-lookup"><span data-stu-id="4abf5-174">Azure CycleCloud Node Array: 10 x Standard H16r (16 x CPUs, 112 GB Memory)</span></span>
* <span data-ttu-id="4abf5-175">Az Azure-fürtön Avere vFXT: 3 x D16s v3 (200 GB-os OS, 1 TB méretű adatlemezzel prémium szintű SSD)</span><span class="sxs-lookup"><span data-stu-id="4abf5-175">Avere vFXT on Azure Cluster: 3 x D16s v3 (200 GB OS, Premium SSD 1-TB data disk)</span></span>
* <span data-ttu-id="4abf5-176">Kimenő adatforgalom: 1 TB-ot</span><span class="sxs-lookup"><span data-stu-id="4abf5-176">Data Egress: 1 TB</span></span>

<span data-ttu-id="4abf5-177">Tekintse át ezt [a becsült ár] [ pricing] a fent felsorolt hardver.</span><span class="sxs-lookup"><span data-stu-id="4abf5-177">Review this [price estimate][pricing] for the hardware listed above.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4abf5-178">További lépések</span><span class="sxs-lookup"><span data-stu-id="4abf5-178">Next Steps</span></span>

<span data-ttu-id="4abf5-179">Miután üzembe helyezte a mintát, tudjon meg többet [Azure CycleCloud][cyclecloud].</span><span class="sxs-lookup"><span data-stu-id="4abf5-179">Once you've deployed the sample, learn more about [Azure CycleCloud][cyclecloud].</span></span>

## <a name="related-resources"></a><span data-ttu-id="4abf5-180">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="4abf5-180">Related resources</span></span>

* <span data-ttu-id="4abf5-181">[RDMA-kompatibilis virtuálisgép-példányok][rdma]</span><span class="sxs-lookup"><span data-stu-id="4abf5-181">[RDMA Capable Machine Instances][rdma]</span></span>
* <span data-ttu-id="4abf5-182">[Egy RDMA-példány virtuális gép testreszabása][rdma-custom]</span><span class="sxs-lookup"><span data-stu-id="4abf5-182">[Customizing an RDMA Instance VM][rdma-custom]</span></span>

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
