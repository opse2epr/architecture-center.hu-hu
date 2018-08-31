---
title: A Big Compute architektúrastílus
description: Ismerteti a Big Compute-architektúrák előnyeit, kihívásait és ajánlott eljárásait az Azure-ban.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: aca2221faf1fbf47de2fd81c8909dfe8aef46bea
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326173"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="89f4a-103">A Big Compute architektúrastílus</span><span class="sxs-lookup"><span data-stu-id="89f4a-103">Big compute architecture style</span></span>

<span data-ttu-id="89f4a-104">A *Big Compute* kifejezés olyan nagyméretű számítási feladatokat jelent, amelyekhez nagy mennyiségű, akár a több száz vagy több ezer mag szükséges.</span><span class="sxs-lookup"><span data-stu-id="89f4a-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="89f4a-105">Ilyen forgatókönyv például a képrenderelés, a folyadékdinamika, a pénzügyi kockázatok modellezése, az olajfeltárás, a gyógyszerfejlesztés vagy a műszaki feszültségelemzés.</span><span class="sxs-lookup"><span data-stu-id="89f4a-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="89f4a-106">A Big Compute-alkalmazások néhány jellemzője:</span><span class="sxs-lookup"><span data-stu-id="89f4a-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="89f4a-107">A munka felosztható önálló feladatokra, amelyek egyidejűleg futtathatók több magon.</span><span class="sxs-lookup"><span data-stu-id="89f4a-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="89f4a-108">Mindegyik feladat véges.</span><span class="sxs-lookup"><span data-stu-id="89f4a-108">Each task is finite.</span></span> <span data-ttu-id="89f4a-109">Bemeneti adatokat igényel, feldolgozást végez és létrehoz egy kimenetet.</span><span class="sxs-lookup"><span data-stu-id="89f4a-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="89f4a-110">A teljes alkalmazás véges ideig fut (ez néhány perctől napokig terjedhet).</span><span class="sxs-lookup"><span data-stu-id="89f4a-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="89f4a-111">Gyakori minta nagy mennyiségű mag kiépítése egyszerre, majd az alkalmazás befejeződésével a magok számának nullára csökkentése.</span><span class="sxs-lookup"><span data-stu-id="89f4a-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="89f4a-112">Az alkalmazásnak nem kell folyamatosan futnia.</span><span class="sxs-lookup"><span data-stu-id="89f4a-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="89f4a-113">Azonban a rendszernek tudnia kell kezelni a csomóponthibákat és az alkalmazás-összeomlásokat.</span><span class="sxs-lookup"><span data-stu-id="89f4a-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="89f4a-114">Egyes alkalmazások esetében a feladatok egymástól függetlenek, és párhuzamosan futtathatók.</span><span class="sxs-lookup"><span data-stu-id="89f4a-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="89f4a-115">Más esetekben a feladatok szorosan kapcsolódnak egymáshoz, azaz kommunikálniuk kell és meg kell osztaniuk egymással a köztes eredményeket.</span><span class="sxs-lookup"><span data-stu-id="89f4a-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="89f4a-116">Ebben az esetben érdemes olyan nagy sebességű hálózatkezelési technológiákat használni, mint az InfiniBand, és a távoli közvetlen memória-hozzáférés (RDMA).</span><span class="sxs-lookup"><span data-stu-id="89f4a-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="89f4a-117">A számítási feladattól függően nagy számítási igényekre szabott virtuálisgép-méretek is használhatók (H16r, H16mr és A9).</span><span class="sxs-lookup"><span data-stu-id="89f4a-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="89f4a-118">Mikor érdemes ezt az architektúrát használni?</span><span class="sxs-lookup"><span data-stu-id="89f4a-118">When to use this architecture</span></span>

- <span data-ttu-id="89f4a-119">Nagy számítási igényű műveletek esetében, mint a szimuláció és a számokkal való munka.</span><span class="sxs-lookup"><span data-stu-id="89f4a-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="89f4a-120">Nagy számítási igényű szimulációk esetében, amelyeket fel kell osztani több számítógép és több CPU között (tíztől több ezres nagyságrendig).</span><span class="sxs-lookup"><span data-stu-id="89f4a-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="89f4a-121">Olyan szimulációk esetében, amelyeknek túl nagy a memóriaigénye egyetlen számítógép számára, ezért fel kell őket osztani több számítógép között.</span><span class="sxs-lookup"><span data-stu-id="89f4a-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="89f4a-122">Hosszan futó számítások esetében, amelyeket egyetlen számítógépen túl sokáig tartana elvégezni.</span><span class="sxs-lookup"><span data-stu-id="89f4a-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="89f4a-123">Kisebb számítások esetében, amelyeket több száz vagy több ezer alkalommal el kell végezni, ilyenek például a Monte Carlo-szimulációk.</span><span class="sxs-lookup"><span data-stu-id="89f4a-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="89f4a-124">Előnyök</span><span class="sxs-lookup"><span data-stu-id="89f4a-124">Benefits</span></span>

- <span data-ttu-id="89f4a-125">Nagy teljesítmény és „[zavaróan párhuzamos][embarrassingly-parallel]” feldolgozás.</span><span class="sxs-lookup"><span data-stu-id="89f4a-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="89f4a-126">Egyszerre több száz vagy több ezer számítógépmag hasznosítása a nagyobb problémák gyorsabb megoldásához.</span><span class="sxs-lookup"><span data-stu-id="89f4a-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="89f4a-127">Hozzáférés speciális nagy teljesítményű hardverekhez dedikált nagy sebességű InfiniBand-hálózatokkal.</span><span class="sxs-lookup"><span data-stu-id="89f4a-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="89f4a-128">A virtuális gépek igény szerint helyezhetők üzembe és építhetők le.</span><span class="sxs-lookup"><span data-stu-id="89f4a-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="89f4a-129">Problémák</span><span class="sxs-lookup"><span data-stu-id="89f4a-129">Challenges</span></span>

- <span data-ttu-id="89f4a-130">A virtuálisgép-infrastruktúra kezelése.</span><span class="sxs-lookup"><span data-stu-id="89f4a-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="89f4a-131">A számokkal való munka mennyiségének kezelése.</span><span class="sxs-lookup"><span data-stu-id="89f4a-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="89f4a-132">Többezer mag időben történő kiépítése.</span><span class="sxs-lookup"><span data-stu-id="89f4a-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="89f4a-133">Szorosan összekapcsolt feladatok esetén több mag hozzáadása ronthatja a teljesítményt.</span><span class="sxs-lookup"><span data-stu-id="89f4a-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="89f4a-134">A magok megfelelő számának megtalálása lehet, hogy kísérletezést igényel.</span><span class="sxs-lookup"><span data-stu-id="89f4a-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="89f4a-135">Big Compute az Azure Batch használatával</span><span class="sxs-lookup"><span data-stu-id="89f4a-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="89f4a-136">Az [Azure Batch][batch] egy felügyelt szolgáltatás, amellyel nagyméretű és nagy teljesítményű feldolgozási (HPC) alkalmazásokat futtathat.</span><span class="sxs-lookup"><span data-stu-id="89f4a-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="89f4a-137">Az Azure Batch használatakor konfigurálnia kell egy VM-készletet, és fel kell töltenie az alkalmazásokat és adatfájlokat.</span><span class="sxs-lookup"><span data-stu-id="89f4a-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="89f4a-138">Ezután a Batch szolgáltatás kiosztja a virtuális gépeket, feladatokat rendel azokhoz, futtatja a feladatokat, és monitorozza az állapotot.</span><span class="sxs-lookup"><span data-stu-id="89f4a-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="89f4a-139">A Batch a számítási feladat igényeinek megfelelően automatikusan méretezi a virtuális gépeket.</span><span class="sxs-lookup"><span data-stu-id="89f4a-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="89f4a-140">A Batch emellett a feladatütemezésről is gondoskodik.</span><span class="sxs-lookup"><span data-stu-id="89f4a-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="89f4a-141">Big Compute a virtuális gépeken</span><span class="sxs-lookup"><span data-stu-id="89f4a-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="89f4a-142">A [Microsoft HPC Packkel][hpc-pack] felügyelhet egy virtuális gépekből álló fürtöt, valamint ütemezheti és monitorozhatja a HPC-feladatokat.</span><span class="sxs-lookup"><span data-stu-id="89f4a-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="89f4a-143">Ennél a módszernél az Ön feladata a virtuális gépek és a hálózati infrastruktúra kiépítése és felügyelete.</span><span class="sxs-lookup"><span data-stu-id="89f4a-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="89f4a-144">A módszert akkor érdemes használni, ha a meglévő HPC számítási feladatait vagy azok egy részét át szeretné helyezni az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="89f4a-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="89f4a-145">Áthelyezheti a teljes HPC-fürtöt az Azure-ba, vagy megtarthatja a fürtöt a helyszínen és az Azure-t használhatja a kapacitáscsúcsok esetén.</span><span class="sxs-lookup"><span data-stu-id="89f4a-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="89f4a-146">További információkért lásd: [Batch- és HPC-megoldások nagyméretű számítási feladatokhoz][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="89f4a-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="89f4a-147">Az Azure-ban telepített HPC Pack</span><span class="sxs-lookup"><span data-stu-id="89f4a-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="89f4a-148">Ebben a forgatókönyvben a HPC-fürtöt teljes egészében az Azure-ban hozzuk létre.</span><span class="sxs-lookup"><span data-stu-id="89f4a-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="89f4a-149">Az átjárócsomópont felügyeleti és feladatütemezési szolgáltatásokat nyújt a fürthöz.</span><span class="sxs-lookup"><span data-stu-id="89f4a-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="89f4a-150">Szorosan összekapcsolt feladatokhoz használjon egy RDMA-hálózatot, amely rendkívül nagy sávszélességű, kis késleltetésű kommunikációt biztosít a virtuális gépek között.</span><span class="sxs-lookup"><span data-stu-id="89f4a-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="89f4a-151">További információkat a [HPC Pack 2016-fürt az Azure-ban való üzembe helyezését][deploy-hpc-azure] ismertető cikkben olvashat.</span><span class="sxs-lookup"><span data-stu-id="89f4a-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="89f4a-152">HPC-fürt löketszerű átvitele az Azure-ba</span><span class="sxs-lookup"><span data-stu-id="89f4a-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="89f4a-153">Ebben a forgatókönyvben a vállalat a helyszínen futtatja a HPC Packet, az Azure-beli virtuális gépeket pedig a kapacitáscsúcsok esetén használja.</span><span class="sxs-lookup"><span data-stu-id="89f4a-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="89f4a-154">A fürt helyszíni átjárócsomóponttal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="89f4a-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="89f4a-155">A helyszíni hálózat ExpressRoute vagy VPN Gateway használatával csatlakozik az Azure VNethez.</span><span class="sxs-lookup"><span data-stu-id="89f4a-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
