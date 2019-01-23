---
title: A Big Compute architektúrastílus
titleSuffix: Azure Application Architecture Guide
description: Előnyeit, kihívásait és ajánlott eljárások a Big Compute-architektúrák ismerteti az Azure-ban.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, HPC
ms.openlocfilehash: 56bd2ce010b56880e769ada4c6397391a73bbd1e
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485757"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="58d8f-103">A Big Compute architektúrastílus</span><span class="sxs-lookup"><span data-stu-id="58d8f-103">Big compute architecture style</span></span>

<span data-ttu-id="58d8f-104">A *Big Compute* kifejezés olyan nagyméretű számítási feladatokat jelent, amelyekhez nagy mennyiségű, akár a több száz vagy több ezer mag szükséges.</span><span class="sxs-lookup"><span data-stu-id="58d8f-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="58d8f-105">Ilyen forgatókönyv például a képrenderelés, a folyadékdinamika, a pénzügyi kockázatok modellezése, az olajfeltárás, a gyógyszerfejlesztés vagy a műszaki feszültségelemzés.</span><span class="sxs-lookup"><span data-stu-id="58d8f-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![Big compute architektúrastílus logikai diagramja](./images/big-compute-logical.png)

<span data-ttu-id="58d8f-107">A Big Compute-alkalmazások néhány jellemzője:</span><span class="sxs-lookup"><span data-stu-id="58d8f-107">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="58d8f-108">A munka felosztható önálló feladatokra, amelyek egyidejűleg futtathatók több magon.</span><span class="sxs-lookup"><span data-stu-id="58d8f-108">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="58d8f-109">Mindegyik feladat véges.</span><span class="sxs-lookup"><span data-stu-id="58d8f-109">Each task is finite.</span></span> <span data-ttu-id="58d8f-110">Bemeneti adatokat igényel, feldolgozást végez és létrehoz egy kimenetet.</span><span class="sxs-lookup"><span data-stu-id="58d8f-110">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="58d8f-111">A teljes alkalmazás véges ideig fut (ez néhány perctől napokig terjedhet).</span><span class="sxs-lookup"><span data-stu-id="58d8f-111">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="58d8f-112">Gyakori minta nagy mennyiségű mag kiépítése egyszerre, majd az alkalmazás befejeződésével a magok számának nullára csökkentése.</span><span class="sxs-lookup"><span data-stu-id="58d8f-112">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span>
- <span data-ttu-id="58d8f-113">Az alkalmazásnak nem kell folyamatosan futnia.</span><span class="sxs-lookup"><span data-stu-id="58d8f-113">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="58d8f-114">Azonban a rendszernek tudnia kell kezelni a csomóponthibákat és az alkalmazás-összeomlásokat.</span><span class="sxs-lookup"><span data-stu-id="58d8f-114">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="58d8f-115">Egyes alkalmazások esetében a feladatok egymástól függetlenek, és párhuzamosan futtathatók.</span><span class="sxs-lookup"><span data-stu-id="58d8f-115">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="58d8f-116">Más esetekben a feladatok szorosan kapcsolódnak egymáshoz, azaz kommunikálniuk kell és meg kell osztaniuk egymással a köztes eredményeket.</span><span class="sxs-lookup"><span data-stu-id="58d8f-116">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="58d8f-117">Ebben az esetben érdemes olyan nagy sebességű hálózatkezelési technológiákat használni, mint az InfiniBand, és a távoli közvetlen memória-hozzáférés (RDMA).</span><span class="sxs-lookup"><span data-stu-id="58d8f-117">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span>
- <span data-ttu-id="58d8f-118">A számítási feladattól függően nagy számítási igényekre szabott virtuálisgép-méretek is használhatók (H16r, H16mr és A9).</span><span class="sxs-lookup"><span data-stu-id="58d8f-118">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="58d8f-119">Mikor érdemes ezt az architektúrát használni?</span><span class="sxs-lookup"><span data-stu-id="58d8f-119">When to use this architecture</span></span>

- <span data-ttu-id="58d8f-120">Nagy számítási igényű műveletek esetében, mint a szimuláció és a számokkal való munka.</span><span class="sxs-lookup"><span data-stu-id="58d8f-120">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="58d8f-121">Nagy számítási igényű szimulációk esetében, amelyeket fel kell osztani több számítógép és több CPU között (tíztől több ezres nagyságrendig).</span><span class="sxs-lookup"><span data-stu-id="58d8f-121">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="58d8f-122">Olyan szimulációk esetében, amelyeknek túl nagy a memóriaigénye egyetlen számítógép számára, ezért fel kell őket osztani több számítógép között.</span><span class="sxs-lookup"><span data-stu-id="58d8f-122">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="58d8f-123">Hosszan futó számítások esetében, amelyeket egyetlen számítógépen túl sokáig tartana elvégezni.</span><span class="sxs-lookup"><span data-stu-id="58d8f-123">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="58d8f-124">Kisebb számítások esetében, amelyeket több száz vagy több ezer alkalommal el kell végezni, ilyenek például a Monte Carlo-szimulációk.</span><span class="sxs-lookup"><span data-stu-id="58d8f-124">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="58d8f-125">Előnyök</span><span class="sxs-lookup"><span data-stu-id="58d8f-125">Benefits</span></span>

- <span data-ttu-id="58d8f-126">Nagy teljesítmény és „[zavaróan párhuzamos][embarrassingly-parallel]” feldolgozás.</span><span class="sxs-lookup"><span data-stu-id="58d8f-126">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="58d8f-127">Egyszerre több száz vagy több ezer számítógépmag hasznosítása a nagyobb problémák gyorsabb megoldásához.</span><span class="sxs-lookup"><span data-stu-id="58d8f-127">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="58d8f-128">Hozzáférés speciális nagy teljesítményű hardverekhez dedikált nagy sebességű InfiniBand-hálózatokkal.</span><span class="sxs-lookup"><span data-stu-id="58d8f-128">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="58d8f-129">A virtuális gépek igény szerint helyezhetők üzembe és építhetők le.</span><span class="sxs-lookup"><span data-stu-id="58d8f-129">You can provision VMs as needed to do work, and then tear them down.</span></span>

## <a name="challenges"></a><span data-ttu-id="58d8f-130">Problémák</span><span class="sxs-lookup"><span data-stu-id="58d8f-130">Challenges</span></span>

- <span data-ttu-id="58d8f-131">A virtuálisgép-infrastruktúra kezelése.</span><span class="sxs-lookup"><span data-stu-id="58d8f-131">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="58d8f-132">Számokkal mennyiségének kezelése</span><span class="sxs-lookup"><span data-stu-id="58d8f-132">Managing the volume of number crunching</span></span>
- <span data-ttu-id="58d8f-133">Többezer mag időben történő kiépítése.</span><span class="sxs-lookup"><span data-stu-id="58d8f-133">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="58d8f-134">Szorosan összekapcsolt feladatok esetén több mag hozzáadása ronthatja a teljesítményt.</span><span class="sxs-lookup"><span data-stu-id="58d8f-134">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="58d8f-135">A magok megfelelő számának megtalálása lehet, hogy kísérletezést igényel.</span><span class="sxs-lookup"><span data-stu-id="58d8f-135">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="58d8f-136">Big Compute az Azure Batch használatával</span><span class="sxs-lookup"><span data-stu-id="58d8f-136">Big compute using Azure Batch</span></span>

<span data-ttu-id="58d8f-137">Az [Azure Batch][batch] egy felügyelt szolgáltatás, amellyel nagyméretű és nagy teljesítményű feldolgozási (HPC) alkalmazásokat futtathat.</span><span class="sxs-lookup"><span data-stu-id="58d8f-137">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="58d8f-138">Az Azure Batch használatakor konfigurálnia kell egy VM-készletet, és fel kell töltenie az alkalmazásokat és adatfájlokat.</span><span class="sxs-lookup"><span data-stu-id="58d8f-138">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="58d8f-139">Ezután a Batch szolgáltatás kiosztja a virtuális gépeket, feladatokat rendel azokhoz, futtatja a feladatokat, és monitorozza az állapotot.</span><span class="sxs-lookup"><span data-stu-id="58d8f-139">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="58d8f-140">A Batch a számítási feladat igényeinek megfelelően automatikusan méretezi a virtuális gépeket.</span><span class="sxs-lookup"><span data-stu-id="58d8f-140">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="58d8f-141">A Batch emellett a feladatütemezésről is gondoskodik.</span><span class="sxs-lookup"><span data-stu-id="58d8f-141">Batch also provides job scheduling.</span></span>

![A big compute az Azure Batch segítségével ábrája](./images/big-compute-batch.png)

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="58d8f-143">Big Compute a virtuális gépeken</span><span class="sxs-lookup"><span data-stu-id="58d8f-143">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="58d8f-144">A [Microsoft HPC Packkel][hpc-pack] felügyelhet egy virtuális gépekből álló fürtöt, valamint ütemezheti és monitorozhatja a HPC-feladatokat.</span><span class="sxs-lookup"><span data-stu-id="58d8f-144">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="58d8f-145">Ennél a módszernél az Ön feladata a virtuális gépek és a hálózati infrastruktúra kiépítése és felügyelete.</span><span class="sxs-lookup"><span data-stu-id="58d8f-145">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="58d8f-146">A módszert akkor érdemes használni, ha a meglévő HPC számítási feladatait vagy azok egy részét át szeretné helyezni az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="58d8f-146">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="58d8f-147">Áthelyezheti a teljes HPC-fürtöt az Azure-ba, vagy megtarthatja a fürtöt a helyszínen és az Azure-t használhatja a kapacitáscsúcsok esetén.</span><span class="sxs-lookup"><span data-stu-id="58d8f-147">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="58d8f-148">További információkért lásd: [Batch- és HPC-megoldások nagyméretű számítási feladatokhoz][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="58d8f-148">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="58d8f-149">Az Azure-ban telepített HPC Pack</span><span class="sxs-lookup"><span data-stu-id="58d8f-149">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="58d8f-150">Ebben a forgatókönyvben a HPC-fürtöt teljes egészében az Azure-ban hozzuk létre.</span><span class="sxs-lookup"><span data-stu-id="58d8f-150">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![Az Azure-ban telepített HPC Pack ábrája](./images/big-compute-iaas.png)

<span data-ttu-id="58d8f-152">Az átjárócsomópont felügyeleti és feladatütemezési szolgáltatásokat nyújt a fürthöz.</span><span class="sxs-lookup"><span data-stu-id="58d8f-152">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="58d8f-153">Szorosan összekapcsolt feladatokhoz használjon egy RDMA-hálózatot, amely rendkívül nagy sávszélességű, kis késleltetésű kommunikációt biztosít a virtuális gépek között.</span><span class="sxs-lookup"><span data-stu-id="58d8f-153">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="58d8f-154">További információkat a [HPC Pack 2016-fürt az Azure-ban való üzembe helyezését][deploy-hpc-azure] ismertető cikkben olvashat.</span><span class="sxs-lookup"><span data-stu-id="58d8f-154">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="58d8f-155">HPC-fürt löketszerű átvitele az Azure-ba</span><span class="sxs-lookup"><span data-stu-id="58d8f-155">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="58d8f-156">Ebben a forgatókönyvben a vállalat a helyszínen futtatja a HPC Packet, az Azure-beli virtuális gépeket pedig a kapacitáscsúcsok esetén használja.</span><span class="sxs-lookup"><span data-stu-id="58d8f-156">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="58d8f-157">A fürt helyszíni átjárócsomóponttal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="58d8f-157">The cluster head node is on-premises.</span></span> <span data-ttu-id="58d8f-158">A helyszíni hálózat ExpressRoute vagy VPN Gateway használatával csatlakozik az Azure VNethez.</span><span class="sxs-lookup"><span data-stu-id="58d8f-158">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![Hibrid big compute-fürt ábrája](./images/big-compute-hybrid.png)

<!-- links -->

[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029
