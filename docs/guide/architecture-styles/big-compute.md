---
title: Nagy számítási architektúra stílus
description: Előnyeit, kihívást és ajánlott eljárások az architektúrák nagy számítási ismerteti az Azure-on
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24539785"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="09424-103">Nagy számítási architektúra stílus</span><span class="sxs-lookup"><span data-stu-id="09424-103">Big compute architecture style</span></span>

<span data-ttu-id="09424-104">A kifejezés *nagy számítási* nagyméretű mag, gyakran száma akár a több száz vagy ezer a nagy számú igénylő munkaterhelések ismerteti.</span><span class="sxs-lookup"><span data-stu-id="09424-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="09424-105">Forgatókönyvek például a kép megjelenítési, folyadékból dynamics, pénzügyi kockázat modellezési, olaj feltárása, gyógyszerfejlesztés és mérnöki magas terhelés elemző, többek között.</span><span class="sxs-lookup"><span data-stu-id="09424-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="09424-106">Az alábbiakban néhány tipikus nagy számítási alkalmazások jellemzői:</span><span class="sxs-lookup"><span data-stu-id="09424-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="09424-107">A munkahelyi felosztható diszkrét feladatokat, melyekhez egyidejűleg több mag között futtatható.</span><span class="sxs-lookup"><span data-stu-id="09424-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="09424-108">Minden feladat azonban véges.</span><span class="sxs-lookup"><span data-stu-id="09424-108">Each task is finite.</span></span> <span data-ttu-id="09424-109">Az egyes bemenetből fogad adatokat, nem az egyes feldolgozási, és kimenetet.</span><span class="sxs-lookup"><span data-stu-id="09424-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="09424-110">A teljes alkalmazás futtatása véges időtartama (perc nap).</span><span class="sxs-lookup"><span data-stu-id="09424-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="09424-111">A megszokott mintát követi, hogy osszon ki a továbbítás magok nagy mennyiségű, és le egyszer nulla léptetéses az alkalmazás végrehajtja.</span><span class="sxs-lookup"><span data-stu-id="09424-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="09424-112">Az alkalmazás nem kell mentése 24/7 maradnak.</span><span class="sxs-lookup"><span data-stu-id="09424-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="09424-113">A rendszer azonban csomópontok hibáit vagy az alkalmazás összeomlik kell kezelni.</span><span class="sxs-lookup"><span data-stu-id="09424-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="09424-114">Az egyes alkalmazásokra a feladatok független, és a párhuzamosan futtatható.</span><span class="sxs-lookup"><span data-stu-id="09424-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="09424-115">Más esetekben feladatok szorosan összekapcsolt, tehát kell módosításának vagy köztes eredmények exchange.</span><span class="sxs-lookup"><span data-stu-id="09424-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="09424-116">Ebben az esetben érdemes lehet a nagy sebességű hálózatkezelési technológiákat, például az InfiniBand és a távoli közvetlen memória-hozzáférés (RDMA).</span><span class="sxs-lookup"><span data-stu-id="09424-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="09424-117">Attól függően, hogy a munkaterhelés használhatja a számítási igényű Virtuálisgép-méretek (H16r H16mr és a9-es).</span><span class="sxs-lookup"><span data-stu-id="09424-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="09424-118">Mikor érdemes használni, ez az architektúra</span><span class="sxs-lookup"><span data-stu-id="09424-118">When to use this architecture</span></span>

- <span data-ttu-id="09424-119">Számítástechnikai számításigényes műveletek, például szimuláció és szám crunching.</span><span class="sxs-lookup"><span data-stu-id="09424-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="09424-120">Számítástechnikai intenzív és több számítógépet (10-1000 egység) a processzorok között kell osztani szimulációja.</span><span class="sxs-lookup"><span data-stu-id="09424-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="09424-121">Túl sok memóriát igényelnek egy számítógép, amely több számítógép között meg kell osztani szimulációja.</span><span class="sxs-lookup"><span data-stu-id="09424-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="09424-122">Hosszan futó számítások, az adott túl sokáig tartott egyetlen számítógépen.</span><span class="sxs-lookup"><span data-stu-id="09424-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="09424-123">Kisebb számítások, amelyek 100-as egység vagy idő szerint, például a Monte Carlo szimulációja 1000 egység kell futtatni.</span><span class="sxs-lookup"><span data-stu-id="09424-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="09424-124">Előnyök</span><span class="sxs-lookup"><span data-stu-id="09424-124">Benefits</span></span>

- <span data-ttu-id="09424-125">Nagy teljesítményt és az "[embarrassingly párhuzamos][embarrassingly-parallel]" feldolgozása.</span><span class="sxs-lookup"><span data-stu-id="09424-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="09424-126">Is hasznosítására több száz vagy ezer számítógép magok gyorsabban nagy problémák megoldására.</span><span class="sxs-lookup"><span data-stu-id="09424-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="09424-127">Speciális nagy teljesítményű hardverre, dedikált nagy sebességű InfiniBand hálózatokkal való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="09424-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="09424-128">Virtuális gépek építhető megfelelően működik, és szakadjon, majd el őket.</span><span class="sxs-lookup"><span data-stu-id="09424-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="09424-129">Kihívásai</span><span class="sxs-lookup"><span data-stu-id="09424-129">Challenges</span></span>

- <span data-ttu-id="09424-130">A virtuális gép infrastruktúra kezelése.</span><span class="sxs-lookup"><span data-stu-id="09424-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="09424-131">A szám crunching mennyisége kezelése.</span><span class="sxs-lookup"><span data-stu-id="09424-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="09424-132">Magok ezer kiépítése időben.</span><span class="sxs-lookup"><span data-stu-id="09424-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="09424-133">Szorosan összekapcsolt feladatokhoz több maggal is jár diminishing értéket ad vissza.</span><span class="sxs-lookup"><span data-stu-id="09424-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="09424-134">Szükség lehet az optimális magok száma található kipróbálásához.</span><span class="sxs-lookup"><span data-stu-id="09424-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="09424-135">Nagy számítási az Azure Batch használatával</span><span class="sxs-lookup"><span data-stu-id="09424-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="09424-136">[Az Azure Batch] [ batch] egy felügyelt szolgáltatás nagy méretű nagy teljesítményű számítástechnikai rendszerek (HPC)-alkalmazások futtatására.</span><span class="sxs-lookup"><span data-stu-id="09424-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="09424-137">Azure Batch használatával, konfigurálja a VM-készletben, és az alkalmazások és adatok fájlok feltöltése.</span><span class="sxs-lookup"><span data-stu-id="09424-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="09424-138">Majd a Batch szolgáltatás kiosztja a virtuális gépek, feladatokat rendelhet a virtuális gépek, futtatja a feladatok és kíséri.</span><span class="sxs-lookup"><span data-stu-id="09424-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="09424-139">Kötegelt automatikusan lehet terjeszteni a virtuális gépeket, a munkaterhelés adott válaszként.</span><span class="sxs-lookup"><span data-stu-id="09424-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="09424-140">Kötegelt is biztosít a feladatütemezés.</span><span class="sxs-lookup"><span data-stu-id="09424-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="09424-141">Nagy virtuális gépeken futó számítási</span><span class="sxs-lookup"><span data-stu-id="09424-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="09424-142">Használhat [Microsoft HPC Pack] [ hpc-pack] virtuális gépek, a fürt felügyeletéhez és ütemezéséhez és figyelése a HPC feladatokat.</span><span class="sxs-lookup"><span data-stu-id="09424-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="09424-143">Ezt a módszert kell kiépíteni, és a virtuális gépek és a hálózati infrastruktúra kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="09424-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="09424-144">Fontolja meg ezt a módszert használja, ha a meglévő HPC munkaterhelésekkel rendelkeznek, és szeretne áttérni a tanúsítványszolgáltatás néhány vagy az összes informatikai az Azure-bA.</span><span class="sxs-lookup"><span data-stu-id="09424-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="09424-145">A teljes HPC-fürt áthelyezése az Azure-ba, vagy tartsa a HPC fürt a helyszínen, de Azure kapacitásnövelés kapacitás használja.</span><span class="sxs-lookup"><span data-stu-id="09424-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="09424-146">További információkért lásd: [számítási munkaterhelés nagy méretű kötegelt és HPC megoldások][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="09424-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="09424-147">Az Azure rendszerbe telepítendő HPC Pack</span><span class="sxs-lookup"><span data-stu-id="09424-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="09424-148">Ebben a forgatókönyvben a HPC-fürt egészében Azure jön létre.</span><span class="sxs-lookup"><span data-stu-id="09424-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="09424-149">Az átjárócsomópont kezelést és a feladatütemezés a fürt szolgáltatásokat biztosítja.</span><span class="sxs-lookup"><span data-stu-id="09424-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="09424-150">Szorosan összekapcsolt feladatokhoz használja az RDMA hálózati nagyon nagy sávszélesség, virtuális gépek kis késleltetésű kommunikációját biztosító.</span><span class="sxs-lookup"><span data-stu-id="09424-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="09424-151">További információ: [HPC Pack 2016-fürt üzembe helyezése az Azure-ban][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="09424-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="09424-152">HPC-fürt löketszerű átvitele az Azure-ba</span><span class="sxs-lookup"><span data-stu-id="09424-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="09424-153">Ebben a forgatókönyvben a szervezet helyszíni HPC Pack fut, és használja az Azure virtuális gépek kapacitásnövelés kapacitást.</span><span class="sxs-lookup"><span data-stu-id="09424-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="09424-154">A fürt átjárócsomópontjába helyszíni.</span><span class="sxs-lookup"><span data-stu-id="09424-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="09424-155">Expressroute-on vagy VPN-átjárót a helyszíni hálózat csatlakozik az Azure virtuális hálózatot.</span><span class="sxs-lookup"><span data-stu-id="09424-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
