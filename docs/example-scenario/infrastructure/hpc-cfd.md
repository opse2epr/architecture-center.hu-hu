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
# <a name="running-computational-fluid-dynamics-cfd-on-azure"></a><span data-ttu-id="7e8e5-103">Azure-on futó numerikus áramlástani (CFD)</span><span class="sxs-lookup"><span data-stu-id="7e8e5-103">Running Computational Fluid Dynamics (CFD) on Azure</span></span>

<span data-ttu-id="7e8e5-104">Folyadékdinamika (CFD) számítási szimulációkat szükséges speciális hardver együtt egy jelentős számítási időért.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-104">Computational Fluid Dynamics (CFD) simulations require a significant compute time along with specialized hardware.</span></span> <span data-ttu-id="7e8e5-105">Ahogy fürt használati növekszik, szimuláció időpontokat, és átfogó grid használata növekszik, a tartalék kapacitás és a hosszú várakozási idők problémák vezető.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-105">As cluster usage increases, simulation times and overall grid use grow, leading to issues with spare capacity and long queue times.</span></span> <span data-ttu-id="7e8e5-106">További fizikai hardver hozzáadása költséges lehet, és előfordulhat, hogy nem igazodnak a használati csúcsok és völgyekhez üzleti halad át.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-106">Adding additional physical hardware can be expensive, and may not align to the usage peaks and valleys business goes through.</span></span> <span data-ttu-id="7e8e5-107">Az Azure előnyeit kihasználva számos, ezek a kihívások szükségességéből, bármely tőkeberuházás nélkül.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-107">By taking advantage of Azure, many of these challenges can be overcome, all without any capital expenditure.</span></span>

<span data-ttu-id="7e8e5-108">Az Azure GPU- és CPU virtuális gépeken a CFD feladatok végrehajtásához szükséges hardverek biztosít.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-108">Azure provides the hardware you need to run your CFD jobs on both GPU and CPU virtual machines.</span></span> <span data-ttu-id="7e8e5-109">RDMA (Remote Direct Memory Access) engedélyezve van a virtuális gépek mérete közel valós idejű MPI (Message Passing Interface) kommunikációt lehetővé tevő FDR InfiniBand-alapú hálózati rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-109">RDMA (Remote Direct Memory Access) enabled VM sizes have FDR InfiniBand based networking allowing for low latency MPI (Message Passing Interface) communication.</span></span> <span data-ttu-id="7e8e5-110">A Avere vFXT, amely egy vállalati szintű fürtözött fájlrendszert biztosít, kombinálva ügyfelek biztosítható a maximális átviteli sebesség az olvasási műveletek az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-110">Combined with the Avere vFXT, which provides an enterprise-scale clustered file system, customers can ensure maximum throughput for read operations in Azure.</span></span>

<span data-ttu-id="7e8e5-111">Egyszerűbb létrehozását, felügyeleti és HPC-fürtök optimalizálása, Azure CycleCloud-fürtök kiépítése és adatok felhőbeli és hibrid forgatókönyvekben is használható.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-111">To simplify the creation, management, and optimization of HPC clusters, Azure CycleCloud can be used to provision clusters and orchestrate data in both hybrid and cloud scenarios.</span></span> <span data-ttu-id="7e8e5-112">A függőben lévő feladatok figyelésével, CycleCloud automatikusan elindul igény szerinti számítási erőforrásokat, ahol csak fizetnie kell fizetni, a számítási feladatok a scheduler a választott csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-112">By monitoring the pending jobs, CycleCloud will automatically launch on-demand compute, where you only pay for what you use, connected to the workload scheduler of your choice.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="7e8e5-113">A lehetséges alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="7e8e5-113">Potential use cases</span></span>

<span data-ttu-id="7e8e5-114">Ebben a forgatókönyvben ezek iparágakban, ahol CFD alkalmazások használható, vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="7e8e5-114">Consider this scenario for these industries where CFD applications could be used:</span></span>

* <span data-ttu-id="7e8e5-115">Repüléstechnikai</span><span class="sxs-lookup"><span data-stu-id="7e8e5-115">Aeronautics</span></span>
* <span data-ttu-id="7e8e5-116">Autóipar</span><span class="sxs-lookup"><span data-stu-id="7e8e5-116">Automotive</span></span>
* <span data-ttu-id="7e8e5-117">HVAC létrehozása</span><span class="sxs-lookup"><span data-stu-id="7e8e5-117">Building HVAC</span></span>
* <span data-ttu-id="7e8e5-118">Olaj- és gázipar</span><span class="sxs-lookup"><span data-stu-id="7e8e5-118">Oil & Gas</span></span>
* <span data-ttu-id="7e8e5-119">Élettudományok</span><span class="sxs-lookup"><span data-stu-id="7e8e5-119">Life Sciences</span></span>

## <a name="architecture"></a><span data-ttu-id="7e8e5-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="7e8e5-120">Architecture</span></span>

![helyettesítő szöveg][cyclearch]

<span data-ttu-id="7e8e5-122">Az ábrán egy tipikus hibrid kialakítással és feladat figyelése az Azure-beli igény szerinti csomópontok biztosít magas szintű áttekintést nyújt:</span><span class="sxs-lookup"><span data-stu-id="7e8e5-122">The diagram provides a high-level overview of a typical hybrid design providing job monitoring of the on-demand nodes in Azure:</span></span>

1. <span data-ttu-id="7e8e5-123">Csatlakozzon az Azure CycleCloud kiszolgálóhoz a fürt konfigurálása</span><span class="sxs-lookup"><span data-stu-id="7e8e5-123">Connect to the Azure CycleCloud server to configure the cluster</span></span>
2. <span data-ttu-id="7e8e5-124">Konfigurálása és létrehozása a fürt fő csomópontjának engedélyezve RDMA gépek MPI használatával</span><span class="sxs-lookup"><span data-stu-id="7e8e5-124">Configure and create the cluster head node, using RDMA enabled machines for MPI</span></span> 
3. <span data-ttu-id="7e8e5-125">Felveheti és konfigurálhatja a helyszíni fő csomópontja</span><span class="sxs-lookup"><span data-stu-id="7e8e5-125">Add and configure the on-premise head node</span></span>
4. <span data-ttu-id="7e8e5-126">Ha rendelkezésre áll elegendő erőforrás, Azure CycleCloud skálázza fel (vagy le) a számítási erőforrásokat az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-126">If there are insufficient resources, Azure CycleCloud will scale up (or down) compute resources in Azure.</span></span> <span data-ttu-id="7e8e5-127">Egy előre meghatározott korlát, hogy foglalási keresztül lehet definiálni.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-127">A predetermined limit can be defined to prevent over allocation.</span></span>
5. <span data-ttu-id="7e8e5-128">A végrehajtás csomópontok feladatai</span><span class="sxs-lookup"><span data-stu-id="7e8e5-128">Tasks allocated to the execute nodes</span></span>
6. <span data-ttu-id="7e8e5-129">Az Azure-ban a helyszíni NFS-kiszolgáló a gyorsítótárazott adatok</span><span class="sxs-lookup"><span data-stu-id="7e8e5-129">Data cached in Azure from on-premises NFS server</span></span>
7. <span data-ttu-id="7e8e5-130">Az Azure cache szolgáltatáshoz a Avere vFXT beolvasott adatok</span><span class="sxs-lookup"><span data-stu-id="7e8e5-130">Data read in from the Avere vFXT for Azure cache</span></span>
8. <span data-ttu-id="7e8e5-131">Az Azure CycleCloud kiszolgálóhoz továbbítón keresztüli feladatok és tevékenységek információk</span><span class="sxs-lookup"><span data-stu-id="7e8e5-131">Job and task information relayed to the Azure CycleCloud server</span></span>


### <a name="components"></a><span data-ttu-id="7e8e5-132">Összetevők</span><span class="sxs-lookup"><span data-stu-id="7e8e5-132">Components</span></span>

* <span data-ttu-id="7e8e5-133">[Az Azure CycleCloud] [ cyclecloud] egy eszköz létrehozását, kezelését, üzemeltetési és az Azure-beli HPC és Big Compute fürtök optimalizálása</span><span class="sxs-lookup"><span data-stu-id="7e8e5-133">[Azure CycleCloud][cyclecloud] a tool for creating, managing, operating, and optimizing HPC & Big Compute clusters in Azure</span></span>
* <span data-ttu-id="7e8e5-134">[Az Azure-ban Avere vFXT] [ avere] egy felhőbeli felhasználásra készült vállalati szintű fürtözött fájlrendszert biztosít</span><span class="sxs-lookup"><span data-stu-id="7e8e5-134">[Avere vFXT on Azure][avere] is used to provide an enterprise-scale clustered file system built for the cloud</span></span>
* <span data-ttu-id="7e8e5-135">[Virtual Machine Scale Sets (virtuálisgép-méretezési)] [ vmss] biztosítanak azonos virtuális gépek képes Azure CycleCloud szerint a felfelé és lefelé skálázva folyamatban</span><span class="sxs-lookup"><span data-stu-id="7e8e5-135">[Virtual Machine Scale Sets (virtual machine scale set)][vmss] provide a group of identical VMs capable of being scaled up or down by Azure CycleCloud</span></span>
* <span data-ttu-id="7e8e5-136">Használat [virtuális gépek] [ vms] egy statikus számítási példányok létrehozása</span><span class="sxs-lookup"><span data-stu-id="7e8e5-136">Use [Virtual Machines][vms] to create a static set of compute instances</span></span>
* <span data-ttu-id="7e8e5-137">[Tárolási] [ storage] fiókok használhatók szinkronizálási és adatmegőrzés</span><span class="sxs-lookup"><span data-stu-id="7e8e5-137">[Storage][storage] accounts are used for synchronization and data retention</span></span>
* <span data-ttu-id="7e8e5-138">[Virtuális hálózat] [ vnet] Azure Virtual Network lehetővé teszi számos különböző típusú Azure-erőforrások, például az Azure Virtual Machines (VM), biztonságosan kommunikálnak egymással, az internethez, és a helyszíni hálózatokkal</span><span class="sxs-lookup"><span data-stu-id="7e8e5-138">[Virtual Network][vnet] Azure Virtual Network enables many types of Azure resources, such as Azure Virtual Machines (VM), to securely communicate with each other, the internet, and on-premises networks</span></span>

### <a name="alternatives"></a><span data-ttu-id="7e8e5-139">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="7e8e5-139">Alternatives</span></span>

<span data-ttu-id="7e8e5-140">Ügyfelek is használhatják Azure CycleCloud rács létrehozása teljes egészében az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-140">Customers can also use Azure CycleCloud to create a grid entirely in Azure.</span></span>  <span data-ttu-id="7e8e5-141">Ezzel a beállítással az Azure CycleCloud kiszolgálón fut az Azure-előfizetésen belül.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-141">In this setup, the Azure CycleCloud server is run within your Azure subscription.</span></span>

<span data-ttu-id="7e8e5-142">Ahol nincs szükség a számítási feladatok Scheduler felügyeleti, a modern alkalmazás megközelítésre [Azure Batch] [ batch] segítségével.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-142">For a modern application approach where management of a workload scheduler is not needed, [Azure Batch][batch] can help.</span></span> <span data-ttu-id="7e8e5-143">Az Azure Batch hatékonyan futtathat nagy méretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat a felhőben.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-143">Azure Batch can run large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="7e8e5-144">Az Azure batch lehetővé teszi, hogy a végrehajtása manuális konfigurálás vagy infrastruktúra kezelése nélkül az alkalmazások párhuzamos és nagy mennyiségű Azure-beli számítási erőforrásokat határoz meg.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-144">Azure batch allows you define the Azure compute resources to execute your applications in parallel or at scale without manually configuring or managing infrastructure.</span></span> <span data-ttu-id="7e8e5-145">A Batch ütemezi a nagy számítási igényű feladatokat és dinamikusan hozzáadja és igényei alapján a számítási erőforrások eltávolítása</span><span class="sxs-lookup"><span data-stu-id="7e8e5-145">Batch schedules compute-intensive tasks and dynamically adds and removes compute resources based on your requirements</span></span>

### <a name="scalability-and-security"></a><span data-ttu-id="7e8e5-146">Méretezhetőség és a biztonság</span><span class="sxs-lookup"><span data-stu-id="7e8e5-146">Scalability, and Security</span></span>

<span data-ttu-id="7e8e5-147">Tekintse meg az execute Azure CycleCloud csomópont is elvégezhető, manuális vagy automatikus skálázás, a méretezés [CycleCloud automatikus skálázás][cycle-scale]</span><span class="sxs-lookup"><span data-stu-id="7e8e5-147">Scaling the execute nodes on Azure CycleCloud can be accomplished either manually or by using autoscaling, see [CycleCloud Autoscaling][cycle-scale]</span></span>

<span data-ttu-id="7e8e5-148">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="7e8e5-148">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="7e8e5-149">Ez a forgatókönyv megvalósítható</span><span class="sxs-lookup"><span data-stu-id="7e8e5-149">Deploy this scenario</span></span>

<span data-ttu-id="7e8e5-150">Az Azure-beli üzembe helyezése előtt bizonyos Előfeltételek teljesülésére szükség, futtassa a fenti lépéseket a Resource Manager-sablon üzembe helyezése előtt:</span><span class="sxs-lookup"><span data-stu-id="7e8e5-150">Before deploying in Azure, some pre-requisites are required, run through these steps before deploying the Resource Manager template:</span></span>
1. <span data-ttu-id="7e8e5-151">[Egyszerű szolgáltatás] [ cycle-svcprin] az appId, displayname, jelszót és bérlői lekéréséhez</span><span class="sxs-lookup"><span data-stu-id="7e8e5-151">[Service Principal][cycle-svcprin] to retrieve the appId, displayname, password, and tenant</span></span>
2. <span data-ttu-id="7e8e5-152">[SSH-kulcspár] [ cycle-ssh] való biztonságos bejelentkezéshez a CycleCloud kiszolgáló</span><span class="sxs-lookup"><span data-stu-id="7e8e5-152">[SSH Key pair][cycle-ssh] to sign in securely to the CycleCloud server</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

3. <span data-ttu-id="7e8e5-153">[Jelentkezzen be a CycleCloud kiszolgáló] [ cycle-login] konfigurálásához és a egy új fürt létrehozása</span><span class="sxs-lookup"><span data-stu-id="7e8e5-153">[Log into the CycleCloud server][cycle-login] to configure and create a new cluster</span></span>
4. <span data-ttu-id="7e8e5-154">[Fürt létrehozása][cycle-create]</span><span class="sxs-lookup"><span data-stu-id="7e8e5-154">[Create a cluster][cycle-create]</span></span> 

<span data-ttu-id="7e8e5-155">A Avere gyorsítótár nem egy nem kötelező megoldást, amely jelentősen felgyorsíthatja az alkalmazásadatok feladat olvasás átviteli sebességét.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-155">The Avere Cache is an optional solution, which can drastically speed up read throughput for the application job data.</span></span> <span data-ttu-id="7e8e5-156">Avere vFXT Azure megoldja a problémát, ezek vállalati HPC-alkalmazások a felhőben futó költséghatékonyhatékonyságából, tárolt adatok a helyszíni vagy Azure Blob storage-ban.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-156">Avere vFXT for Azure solves the problem of running these enterprise HPC applications in the cloud while leveraging data stored on-premises or in Azure Blob storage.</span></span>

<span data-ttu-id="7e8e5-157">A szervezet számára, amely a helyszíni tárolók és a felhő-számítástechnika a hibrid infrastruktúra helyzet tervez HPC-alkalmazások is "burst"-adatok tárolva NAS-eszközökön, és szükség szerint révén tetszőleges számú virtuális processzort elindítása az Azure-bA.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-157">For organizations that are planning on a hybrid infrastructure situation with on-premises storage and cloud computing, HPC applications can “burst” into Azure using data stored in NAS devices and spin up as many virtual CPUs on an as needed basis.</span></span> <span data-ttu-id="7e8e5-158">Maga az adathalmaz teljesen soha nem helyezi át a felhőbe.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-158">The data set itself never completely moves into the cloud.</span></span> <span data-ttu-id="7e8e5-159">A kért bájtok átmenetileg gyorsítótárazza a rendszer feldolgozása közben Avere fürtöt használ.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-159">The requested bytes are temporarily cached using an Avere cluster while processing.</span></span>

<span data-ttu-id="7e8e5-160">Beállítását, és a egy Avere vFXT telepítését konfigurálni, kövesse a [Avere beállítás és konfiguráció] [ avere] útmutató.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-160">To set up and configure an Avere vFXT installation, follow the [Avere Setup and Configuration][avere] guide.</span></span>

## <a name="pricing"></a><span data-ttu-id="7e8e5-161">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="7e8e5-161">Pricing</span></span>

<span data-ttu-id="7e8e5-162">Egy HPC-implementációhoz CycleCloud kiszolgáló használatával futtatásával járó költségeket számos tényezőtől függően eltérőek lehetnek.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-162">The cost of running an HPC implementation using CycleCloud server will differ depending on a number of factors.</span></span> <span data-ttu-id="7e8e5-163">Ha például CycleCloud használt, a fő- és CycleCloud kiszolgáló foglalva és futnak általában alatt folyamatosan a számítási idő mennyisége alapján kell fizetnie.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-163">For example, CycleCloud is charged by the amount of compute time that is used, with the Master and CycleCloud server typically being constantly allocated and running.</span></span> <span data-ttu-id="7e8e5-164">A végrehajtás csomópontok futtatásával járó költségeket függ, milyen hosszú az alábbiak helyezheti üzembe a emellett mérete szolgál.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-164">The cost of running the Execute nodes will depend on how long these are up and running for in addition to which size is used.</span></span> <span data-ttu-id="7e8e5-165">A normál Azure tárolási és hálózatkezelési is keletkezhet díjfizetési kötelezettség.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-165">The normal Azure charges for Storage and Networking also apply.</span></span>  

<span data-ttu-id="7e8e5-166">Ez a forgatókönyv célja bemutatják, hogyan CFD alkalmazások is futtathatók az Azure-ban, így a gépek szükséges RDMA funkciója, amely csak az adott Virtuálisgép-méretek érhető el.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-166">This scenario is intended to show how CFD applications can be run in Azure, so the machines will require RDMA functionality, which is only available on specific VM sizes.</span></span> <span data-ttu-id="7e8e5-167">Az alábbi parancsok példák egy méretezési csoport egy hónapban, az 1 TB-nyi adatot, egy kimenő forgalmi napi 8 órán folyamatosan lefoglalt sikerült felmerülő költségek.</span><span class="sxs-lookup"><span data-stu-id="7e8e5-167">The following are examples of costs that could be incurred for a scale set that is allocated continuously for 8 hours a day for a month, with an egress of 1 TB of data.</span></span> <span data-ttu-id="7e8e5-168">Ezenkívül tartalmazza a díjszabás az Azure CycleCloud server és az Azure-telepítés Avere vFXT:</span><span class="sxs-lookup"><span data-stu-id="7e8e5-168">It also includes the pricing for the Azure CycleCloud server and the Avere vFXT for Azure install:</span></span>

* <span data-ttu-id="7e8e5-169">Régióban: Észak-Európa</span><span class="sxs-lookup"><span data-stu-id="7e8e5-169">Region: North Europe</span></span>
* <span data-ttu-id="7e8e5-170">Azure CycleCloud Server: Standard D3 x 1 (4 x Standard HDD processzor, 14 GB memória, 32 GB)</span><span class="sxs-lookup"><span data-stu-id="7e8e5-170">Azure CycleCloud Server: 1 x Standard D3 (4 x CPUs, 14 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="7e8e5-171">Az Azure CycleCloud főkiszolgáló: 1 x Standard D12 v (4 x processzor, 28 GB memória, Standard HDD 32 GB)</span><span class="sxs-lookup"><span data-stu-id="7e8e5-171">Azure CycleCloud Master Server: 1 x Standard D12 v (4 x CPUs, 28 GB Memory, Standard HDD 32 GB)</span></span>
* <span data-ttu-id="7e8e5-172">Az Azure CycleCloud csomópont tömb: 10 Standard H16r x (16 x processzor, 112 GB memória)</span><span class="sxs-lookup"><span data-stu-id="7e8e5-172">Azure CycleCloud Node Array: 10 x Standard H16r (16 x CPUs, 112 GB Memory)</span></span>
* <span data-ttu-id="7e8e5-173">Az Azure-fürtön Avere vFXT: 3 x D16s v3 (200 GB-os OS, 1 TB méretű adatlemezzel prémium szintű SSD)</span><span class="sxs-lookup"><span data-stu-id="7e8e5-173">Avere vFXT on Azure Cluster: 3 x D16s v3 (200 GB OS, Premium SSD 1-TB data disk)</span></span>
* <span data-ttu-id="7e8e5-174">Kimenő adatforgalom: 1 TB-ot</span><span class="sxs-lookup"><span data-stu-id="7e8e5-174">Data Egress: 1 TB</span></span>

<span data-ttu-id="7e8e5-175">A következő hivatkozás használatával ellenőrizze a [a becsült ár] [ pricing] a fenti hardver</span><span class="sxs-lookup"><span data-stu-id="7e8e5-175">Use the following link to check the [Price Estimate][pricing] for the above hardware</span></span>

## <a name="next-steps"></a><span data-ttu-id="7e8e5-176">További lépések</span><span class="sxs-lookup"><span data-stu-id="7e8e5-176">Next Steps</span></span>

<span data-ttu-id="7e8e5-177">Miután üzembe helyezte a mintát, ismerje meg bővebben az Azure CycleCloud a [Azure CycleCloud dokumentációja][cyclecloud]</span><span class="sxs-lookup"><span data-stu-id="7e8e5-177">Once you've deployed the sample, learn more information about Azure CycleCloud in the [Azure CycleCloud Documentation][cyclecloud]</span></span>

## <a name="related-resources"></a><span data-ttu-id="7e8e5-178">Kapcsolódó erőforrások</span><span class="sxs-lookup"><span data-stu-id="7e8e5-178">Related Resources</span></span>

* <span data-ttu-id="7e8e5-179">[RDMA-kompatibilis virtuálisgép-példányok][rdma]</span><span class="sxs-lookup"><span data-stu-id="7e8e5-179">[RDMA Capable Machine Instances][rdma]</span></span>
* <span data-ttu-id="7e8e5-180">[Egy RDMA-példány virtuális gép testreszabása][rdma-custom]</span><span class="sxs-lookup"><span data-stu-id="7e8e5-180">[Customizing an RDMA Instance VM][rdma-custom]</span></span>



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
