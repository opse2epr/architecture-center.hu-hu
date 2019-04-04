---
title: Nagy teljesítményű feldolgozás (HPC) az Azure-ban
description: Útmutató Azure-on futó HPC számítási feladatok készítéséhez
author: adamboeglin
ms.date: 2/4/2019
ms.openlocfilehash: 5263dd3a06e5244bf804df4be6ec57d789574f76
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489198"
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a><span data-ttu-id="fdfbc-103">Nagy teljesítményű feldolgozás (HPC) az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-103">High Performance Computing (HPC) on Azure</span></span>

## <a name="introduction-to-hpc"></a><span data-ttu-id="fdfbc-104">A HPC bemutatása</span><span class="sxs-lookup"><span data-stu-id="fdfbc-104">Introduction to HPC</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="fdfbc-105">A nagy teljesítményű feldolgozás (HPC), más néven „Big Compute” számos CPU- vagy GPU-alapú számítógépet használ összetett matematikai feladatok megoldására.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-105">High Performance Computing (HPC), also called "Big Compute", uses a large number of CPU or GPU-based computers to solve complex mathematical tasks.</span></span>

<span data-ttu-id="fdfbc-106">Számos iparág használ HPC-t a legnehezebb problémák megoldásához.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-106">Many industries use HPC to solve some of their most difficult problems.</span></span>  <span data-ttu-id="fdfbc-107">Ilyenek többek között a következő számítási feladatok:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-107">These include workloads such as:</span></span>

- <span data-ttu-id="fdfbc-108">Genomics</span><span class="sxs-lookup"><span data-stu-id="fdfbc-108">Genomics</span></span>
- <span data-ttu-id="fdfbc-109">Olaj- és gázszimulációk</span><span class="sxs-lookup"><span data-stu-id="fdfbc-109">Oil and gas simulations</span></span>
- <span data-ttu-id="fdfbc-110">Pénzügy</span><span class="sxs-lookup"><span data-stu-id="fdfbc-110">Finance</span></span>
- <span data-ttu-id="fdfbc-111">Félvezetők tervezése</span><span class="sxs-lookup"><span data-stu-id="fdfbc-111">Semiconductor design</span></span>
- <span data-ttu-id="fdfbc-112">Mérnöki tevékenységek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-112">Engineering</span></span>
- <span data-ttu-id="fdfbc-113">Időjárás-modellezés</span><span class="sxs-lookup"><span data-stu-id="fdfbc-113">Weather modeling</span></span>

### <a name="how-is-hpc-different-on-the-cloud"></a><span data-ttu-id="fdfbc-114">Miben más a felhőbeli HPC?</span><span class="sxs-lookup"><span data-stu-id="fdfbc-114">How is HPC different on the cloud?</span></span>

<span data-ttu-id="fdfbc-115">A helyszíni és a felhőbeli HPC-rendszerek közötti egyik legfontosabb különbség, hogy az erőforrások igény szerint, dinamikusan adhatók hozzá vagy távolíthatók el.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-115">One of the primary differences between an on-premise HPC system and one in the cloud is the ability for resources to dynamically be added and removed as they're needed.</span></span>  <span data-ttu-id="fdfbc-116">A dinamikus skálázás megszünteti a számítási kapacitás okozta szűk keresztmetszetet, és így lehetővé teszi, hogy az ügyfelek a feladataik követelményeinek megfelelően igazítsák az infrastruktúrájuk méretét.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-116">Dynamic scaling removes compute capacity as a bottleneck and instead allow customers to right size their infrastructure for the requirements of their jobs.</span></span>

<span data-ttu-id="fdfbc-117">A következő cikk további részleteket tartalmaz a dinamikus méretezési képességről.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-117">The following articles provide more detail about this dynamic scaling capability.</span></span>

- [<span data-ttu-id="fdfbc-118">A Big Compute architektúrastílus</span><span class="sxs-lookup"><span data-stu-id="fdfbc-118">Big Compute Architecture Style</span></span>](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-119">Automatikus skálázással kapcsolatos ajánlott eljárások</span><span class="sxs-lookup"><span data-stu-id="fdfbc-119">Autoscaling best practices</span></span>](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a><span data-ttu-id="fdfbc-120">Megvalósítási ellenőrzőlista</span><span class="sxs-lookup"><span data-stu-id="fdfbc-120">Implementation checklist</span></span>

<span data-ttu-id="fdfbc-121">Amikor saját HPC-megoldást tervez megvalósítani az Azure-on, mindenképpen tekintse át a következő témaköröket:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-121">As you're looking to implement your own HPC solution on Azure, ensure you're reviewed the following topics:</span></span>

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - <span data-ttu-id="fdfbc-122">A követelményei alapján válassza ki a megfelelő [architektúrát](#infrastructure)</span><span class="sxs-lookup"><span data-stu-id="fdfbc-122">Choose the appropriate [architecture](#infrastructure) based on your requirements</span></span>
> - <span data-ttu-id="fdfbc-123">Legyen tisztában azzal, hogy melyik [számítási](#compute) lehetőségek felelnek meg számítási feladataihoz</span><span class="sxs-lookup"><span data-stu-id="fdfbc-123">Know which [compute](#compute) options is right for your workload</span></span>
> - <span data-ttu-id="fdfbc-124">Azonosítsa az igényeinek megfelelő [tárolási](#storage) megoldást</span><span class="sxs-lookup"><span data-stu-id="fdfbc-124">Identify the right [storage](#storage) solution that meets your needs</span></span>
> - <span data-ttu-id="fdfbc-125">Döntse el, hogyan fogja [kezelni](#management) az összes erőforrást</span><span class="sxs-lookup"><span data-stu-id="fdfbc-125">Decide how you're going to [manage](#management) all your resources</span></span>
> - <span data-ttu-id="fdfbc-126">Optimalizálja [alkalmazását](#hpc-applications) a felhőhöz</span><span class="sxs-lookup"><span data-stu-id="fdfbc-126">Optimize your [application](#hpc-applications) for the cloud</span></span>
> - <span data-ttu-id="fdfbc-127">Gondoskodjon az infrastruktúra [védelméről](#security)</span><span class="sxs-lookup"><span data-stu-id="fdfbc-127">[Secure](#security) your Infrastructure</span></span>

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a><span data-ttu-id="fdfbc-128">Infrastruktúra</span><span class="sxs-lookup"><span data-stu-id="fdfbc-128">Infrastructure</span></span>

<span data-ttu-id="fdfbc-129">Számos infrastruktúra-összetevőre van szükség egy HPC-rendszer felépítéséhez.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-129">There are a number of infrastructure components necessary to build an HPC system.</span></span>  <span data-ttu-id="fdfbc-130">A HPC számítási feladatok választott kezelési módjától függetlenül a Compute, a Storage és a Networking adja meg az alapvető összetevőket.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-130">Compute, Storage, and Networking provide the underlying components, no matter how you choose to manage your HPC workloads.</span></span>

### <a name="example-hpc-architectures"></a><span data-ttu-id="fdfbc-131">Példák HPC-architektúrákra</span><span class="sxs-lookup"><span data-stu-id="fdfbc-131">Example HPC architectures</span></span>

<span data-ttu-id="fdfbc-132">Számos különböző módon tervezheti és valósíthatja meg HPC-architektúráját az Azure-on.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-132">There are a number of different ways to design and implement your HPC architecture on Azure.</span></span>  <span data-ttu-id="fdfbc-133">A HPC-alkalmazások több ezer számítási magra is felskálázhatók, kiterjeszthetik a helyszíni fürtöket, de 100%-ban natív felhőalapú megoldásként is futtathatók.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-133">HPC applications can scale to thousands of compute cores, extend on-premises clusters, or run as a 100% cloud native solution.</span></span>

<span data-ttu-id="fdfbc-134">A következő forgatókönyvek a HPC-megoldások felépítésének néhány gyakori módját írják le.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-134">The following scenarios outline a few of the common ways HPC solutions are built.</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-135">CAE-szolgáltatások az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-135">Computer-aided engineering services on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-136">SaaS-platformot biztosíthat CAE-projektekhez az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-136">Provide a software-as-a-service (SaaS) platform for computer-aided engineering (CAE) on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-137">Számítási folyadékdinamikai (CFD) szimulációk az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-137">Computational fluid dynamics (CFD) simulations on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-138">Számítási folyadékdinamikai (CFD) szimulációkat hajthat végre az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-138">Execute computational fluid dynamics (CFD) simulations on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-139">3D-s videórenderelés az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-139">3D video rendering on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-140">Natív HPC számítási feladatok futtatása az Azure-ban az Azure Batch szolgáltatás használatával</span><span class="sxs-lookup"><span data-stu-id="fdfbc-140">Run native HPC workloads in Azure using the Azure Batch service</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a><span data-ttu-id="fdfbc-141">Compute</span><span class="sxs-lookup"><span data-stu-id="fdfbc-141">Compute</span></span>

<span data-ttu-id="fdfbc-142">Az Azure számos, nagy CPU- és GPU-igényű számítási feladatokhoz optimalizált méretet nyújt.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-142">Azure offers a range of sizes that are optimized for both CPU & GPU intensive workloads.</span></span>

#### <a name="cpu-based-virtual-machines"></a><span data-ttu-id="fdfbc-143">CPU-alapú virtuális gépek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-143">CPU-based virtual machines</span></span>

- [<span data-ttu-id="fdfbc-144">Linux rendszerű virtuális gépek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-144">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="fdfbc-145">[Windows rendszerű virtuális gépek](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) virtuális gépei</span><span class="sxs-lookup"><span data-stu-id="fdfbc-145">[Windows VM's](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) VMs</span></span>
  
#### <a name="gpu-enabled-virtual-machines"></a><span data-ttu-id="fdfbc-146">GPU-kompatibilis virtuális gépek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-146">GPU-enabled virtual machines</span></span>

<span data-ttu-id="fdfbc-147">Az N-sorozatú virtuális gépeken NVIDIA GPU-k találhatók, amelyek nagy számítási igényű vagy nagy grafikai igényű alkalmazásokhoz vannak tervezve, például mesterséges intelligencia (AI) tanításához és vizualizációhoz.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-147">N-series VMs feature NVIDIA GPUs designed for compute-intensive or graphics-intensive applications including artificial intelligence (AI) learning and visualization.</span></span>

- [<span data-ttu-id="fdfbc-148">Linux rendszerű virtuális gépek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-148">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-149">Windows rendszerű virtuális gépek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-149">Windows VMs</span></span>](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a><span data-ttu-id="fdfbc-150">Storage</span><span class="sxs-lookup"><span data-stu-id="fdfbc-150">Storage</span></span>

<span data-ttu-id="fdfbc-151">A nagy méretű Batch és HPC számítási feladatok olyan adattárolási és hozzáférési igényekkel rendelkeznek, amelyek meghaladják a hagyományos felhőalapú fájlrendszerek képességeit.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-151">Large-scale Batch and HPC workloads have demands for data storage and access that exceed the capabilities of traditional cloud file systems.</span></span>  <span data-ttu-id="fdfbc-152">Számos megoldás létezik az Azure-ban futó HPC-alkalmazások sebesség- és kapacitásigényeinek kezelésére</span><span class="sxs-lookup"><span data-stu-id="fdfbc-152">There are a number of solutions to manage both the speed and capacity needs of HPC applications on Azure</span></span>

- <span data-ttu-id="fdfbc-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) – gyorsabb, hozzáférhetőbb adattárolás a peremhálózaton végzett nagy teljesítményű számításokhoz</span><span class="sxs-lookup"><span data-stu-id="fdfbc-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) for faster, more accessible data storage for high-performance computing at the edge</span></span>
- [<span data-ttu-id="fdfbc-154">BeeGFS</span><span class="sxs-lookup"><span data-stu-id="fdfbc-154">BeeGFS</span></span>](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [<span data-ttu-id="fdfbc-155">Tárolásra optimalizált virtuális gépek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-155">Storage Optimized Virtual Machines</span></span>](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-156">Blob, tábla és Queue Storage</span><span class="sxs-lookup"><span data-stu-id="fdfbc-156">Blob, table, and queue storage</span></span>](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-157">Azure SMB fájltároló</span><span class="sxs-lookup"><span data-stu-id="fdfbc-157">Azure SMB File storage</span></span>](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-158">Intel Cloud Edition Lustre</span><span class="sxs-lookup"><span data-stu-id="fdfbc-158">Intel Cloud Edition Lustre</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

<span data-ttu-id="fdfbc-159">Az Azure-on használt Lustre, GlusterFS és BeeGFS összehasonlításáért tekintse meg [a párhuzamos fájlrendszerekkel foglalkozó Azure e-könyvet](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/).</span><span class="sxs-lookup"><span data-stu-id="fdfbc-159">For more information comparing Lustre, GlusterFS, and BeeGFS on Azure, review the [Parallel Files Systems on Azure eBook](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)</span></span>

### <a name="networking"></a><span data-ttu-id="fdfbc-160">Hálózat</span><span class="sxs-lookup"><span data-stu-id="fdfbc-160">Networking</span></span>

<span data-ttu-id="fdfbc-161">A H16r, H16mr, A8 és A9 virtuális gépek magas teljesítményű, háttérben futó RDMA-hálózathoz csatlakozhatnak.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-161">H16r, H16mr, A8, and A9 VMs can connect to a high throughput back-end RDMA network.</span></span> <span data-ttu-id="fdfbc-162">Ez a hálózat javíthatja a Microsoft MPI vagy az Intel MPI alatt futó szorosan összekapcsolt párhuzamos alkalmazások teljesítményét.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-162">This network can improve the performance of tightly coupled parallel applications running under Microsoft MPI or Intel MPI.</span></span>

- [<span data-ttu-id="fdfbc-163">RDMA-kompatibilis példányok</span><span class="sxs-lookup"><span data-stu-id="fdfbc-163">RDMA Capable Instances</span></span>](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [<span data-ttu-id="fdfbc-164">Virtual Network</span><span class="sxs-lookup"><span data-stu-id="fdfbc-164">Virtual Network</span></span>](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-165">ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="fdfbc-165">ExpressRoute</span></span>](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a><span data-ttu-id="fdfbc-166">Kezelés</span><span class="sxs-lookup"><span data-stu-id="fdfbc-166">Management</span></span>

### <a name="do-it-yourself"></a><span data-ttu-id="fdfbc-167">Önkiszolgáló</span><span class="sxs-lookup"><span data-stu-id="fdfbc-167">Do-it-yourself</span></span>

<span data-ttu-id="fdfbc-168">A HPC-rendszerek az alapoktól való felépítése az Azure-ban jelentős mértékű rugalmasságot nyújt, de gyakran nagyon sok karbantartást igényel.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-168">Building an HPC system from scratch on Azure offers a significant amount of flexibility, but is often very maintenance intensive.</span></span>  

1. <span data-ttu-id="fdfbc-169">Állítsa be saját fürtkörnyezetét az Azure-beli virtuális gépeken vagy a [virtuálisgép-méretezési csoportokban](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="fdfbc-169">Set up your own cluster environment in Azure virtual machines or [virtual machine scale sets](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>
2. <span data-ttu-id="fdfbc-170">Helyezzen üzembe vezető [számításifeladat-kezelőket](#workload-managers), infrastruktúrát és [alkalmazásokat](#hpc-applications) Azure Resource Manager-sablonok segítségével.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-170">Use Azure Resource Manager templates to deploy leading [workload managers](#workload-managers), infrastructure, and [applications](#hpc-applications).</span></span>
3. <span data-ttu-id="fdfbc-171">Válasszon HPC és GPU [virtuálisgép-méreteket](#compute), amelyek speciális hardvert és hálózati kapcsolatokat tartalmaznak az MPI és GPU számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-171">Choose HPC and GPU [VM sizes](#compute) that include specialized hardware and network connections for MPI or GPU workloads.</span></span>
4. <span data-ttu-id="fdfbc-172">Adjon hozzá [nagy teljesítményű tárolót](#storage) az I/O-igényes számítási feladatok számára.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-172">Add [high performance storage](#storage) for I/O-intensive workloads.</span></span>

### <a name="hybrid-and-cloud-bursting"></a><span data-ttu-id="fdfbc-173">Hibrid és felhőalapú teljesítménynövelés</span><span class="sxs-lookup"><span data-stu-id="fdfbc-173">Hybrid and cloud Bursting</span></span>

<span data-ttu-id="fdfbc-174">Ha meglévő helyszíni HPC rendszere van, amelyet az Azure-hoz szeretne csatlakoztatni, számos erőforrás segíthet ennek megkezdésében.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-174">If you have an existing on-premise HPC system that you'd like to connect to Azure, there are a number of resources to help get you started.</span></span>

<span data-ttu-id="fdfbc-175">Először tekintse át a dokumentáció [A helyszíni hálózat Azure-hoz való csatlakoztatásának lehetőségei](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) című cikkét.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-175">First, review the [Options for connecting an on-premises network to Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) article in the documentation.</span></span>  <span data-ttu-id="fdfbc-176">Ezután a következő csatlakoztatási lehetőségekről szerezhet információt:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-176">From there, you may want information on these connectivity options:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-177">Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-átjáró használatával</span><span class="sxs-lookup"><span data-stu-id="fdfbc-177">Connect an on-premises network to Azure using a VPN gateway</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-178">Ez a referenciaarchitektúra bemutatja, hogyan lehet kibővíteni a helyszíni hálózatot az Azure-ra helyek közötti virtuális magánhálózat (VPN) használatával.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-178">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-179">Helyszíni hálózat csatlakoztatása az Azure-hoz ExpressRoute használatával</span><span class="sxs-lookup"><span data-stu-id="fdfbc-179">Connect an on-premises network to Azure using ExpressRoute</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-180">Az ExpressRoute-kapcsolatok dedikált, privát kapcsolatot használnak egy külső kapcsolatszolgáltatón keresztül.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-180">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="fdfbc-181">A privát kapcsolat kiterjeszti a helyszíni hálózatot az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-181">The private connection extends your on-premises network into Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-182">Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával</span><span class="sxs-lookup"><span data-stu-id="fdfbc-182">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-183">Egy helyek közötti, magas rendelkezésre állású és biztonságos hálózati architektúrát építhet ki, amely a VPN-átjáróval feladatátvételt biztosító ExpressRoute használatával összekapcsolt Azure-beli virtuális hálózatból és helyszíni hálózatból áll.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-183">Implement a highly available and secure site-to-site network architecture that spans an Azure virtual network and an on-premises network connected using ExpressRoute with VPN gateway failover.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

<span data-ttu-id="fdfbc-184">A hálózati kapcsolat biztonságos létrejötte után megkezdheti a felhőalapú számítási erőforrások igény szerinti használatát a meglévő [számításifeladat-kezelő](#workload-managers) teljesítménynövelési képességeivel.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-184">Once network connectivity is securely established, you can start using cloud compute resources on-demand with the bursting capabilities of your existing [workload manager](#workload-managers).</span></span>

### <a name="marketplace-solutions"></a><span data-ttu-id="fdfbc-185">A Marketplace-ről származó megoldások</span><span class="sxs-lookup"><span data-stu-id="fdfbc-185">Marketplace solutions</span></span>

<span data-ttu-id="fdfbc-186">Az [Azure Marketplace-en](https://azuremarketplace.microsoft.com/marketplace/) számos számításifeladat-kezelő érhető el.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-186">There are a number of workload managers offered in the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/).</span></span>

- [<span data-ttu-id="fdfbc-187">RogueWave CentOS-alapú HPC</span><span class="sxs-lookup"><span data-stu-id="fdfbc-187">RogueWave CentOS-based HPC</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [<span data-ttu-id="fdfbc-188">SUSE Linux Enterprise Server HPC-hez</span><span class="sxs-lookup"><span data-stu-id="fdfbc-188">SUSE Linux Enterprise Server for HPC</span></span>](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [<span data-ttu-id="fdfbc-189">TIBCO Grid Server motor</span><span class="sxs-lookup"><span data-stu-id="fdfbc-189">TIBCO Grid Server Engine</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [<span data-ttu-id="fdfbc-190">Windows és Linux rendszerhez való Azure Data Science VM</span><span class="sxs-lookup"><span data-stu-id="fdfbc-190">Azure Data Science VM for Windows and Linux</span></span>](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-191">D3View</span><span class="sxs-lookup"><span data-stu-id="fdfbc-191">D3View</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [<span data-ttu-id="fdfbc-192">UberCloud</span><span class="sxs-lookup"><span data-stu-id="fdfbc-192">UberCloud</span></span>](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a><span data-ttu-id="fdfbc-193">Azure Batch</span><span class="sxs-lookup"><span data-stu-id="fdfbc-193">Azure Batch</span></span>

<span data-ttu-id="fdfbc-194">Az [Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) platformszolgáltatás lehetővé teszi, hogy hatékonyan futtasson nagyméretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat a felhőben.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) is a platform service for running large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="fdfbc-195">Az Azure Batch számításigényes munkák futtatását ütemezi virtuális gépek felügyelt készletében, és automatikusan képes méretezni a számítási erőforrásokat a feladatok igényeinek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-195">Azure Batch schedules compute-intensive work to run on a managed pool of virtual machines, and can automatically scale compute resources to meet the needs of your jobs.</span></span>

<span data-ttu-id="fdfbc-196">A SaaS-szolgáltatók vagy -fejlesztők a Batch SDK-kal és -eszközökkel HPC-alkalmazásokat vagy tárolókhoz kapcsolódó számítási feladatokat integrálhatnak az Azure-ral, adatokat bocsáthatnak rendelkezésre az Azure-ba, valamint feladat-végrehajtási folyamatokat hozhatnak létre.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-196">SaaS providers or developers can use the Batch SDKs and tools to integrate HPC applications or container workloads with Azure, stage data to Azure, and build job execution pipelines.</span></span>

### <a name="azure-cyclecloud"></a><span data-ttu-id="fdfbc-197">Azure CycleCloud</span><span class="sxs-lookup"><span data-stu-id="fdfbc-197">Azure CycleCloud</span></span>

<span data-ttu-id="fdfbc-198">Az [Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) nyújtja a legegyszerűbb módszert a HPC számítási feladatok bármilyen ütemezővel (például Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro vagy Symphony) történő kezelésére az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) Provides the simplest way to manage HPC workloads using any scheduler (like Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro, or Symphony), on Azure</span></span>

<span data-ttu-id="fdfbc-199">A CycleCloud a következőket teszi lehetővé:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-199">CycleCloud allows you to:</span></span>

- <span data-ttu-id="fdfbc-200">Teljes fürtök és más erőforrások (ütemező, számítási virtuális gépek, tárterület, hálózat és gyorsítótár) üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="fdfbc-200">Deploy full clusters and other resources, including scheduler, compute VMs, storage, networking, and cache</span></span>
- <span data-ttu-id="fdfbc-201">A feladatok, az adatok és a felhő munkafolyamatainak összehangolása</span><span class="sxs-lookup"><span data-stu-id="fdfbc-201">Orchestrate job, data, and cloud workflows</span></span>
- <span data-ttu-id="fdfbc-202">Teljes körű szabályozás biztosítása a rendszergazdáknak afölött, hogy melyik felhasználók futtathatnak feladatokat, illetve hol és milyen költségek mellett tehetik meg ezt.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-202">Give admins full control over which users can run jobs, as well as where and at what cost</span></span>
- <span data-ttu-id="fdfbc-203">Fürtök testreszabása és optimalizálása speciális szabályzatok és irányítási funkciók, például költségvezérlés, Active Directory-integráció, monitorozás és jelentéskészítés segítségével</span><span class="sxs-lookup"><span data-stu-id="fdfbc-203">Customize and optimize clusters through advanced policy and governance features, including cost controls, Active Directory integration, monitoring, and reporting</span></span>
- <span data-ttu-id="fdfbc-204">A jelenlegi feladatütemező és alkalmazások használata módosítás nélkül</span><span class="sxs-lookup"><span data-stu-id="fdfbc-204">Use your current job scheduler and applications without modification</span></span>
- <span data-ttu-id="fdfbc-205">Beépített, automatikus skálázási funkció és élesben kipróbált referenciaarchitektúrák számos HPC számítási feladathoz és iparághoz</span><span class="sxs-lookup"><span data-stu-id="fdfbc-205">Take advantage of built-in autoscaling and battle-tested reference architectures for a wide range of HPC workloads and industries</span></span>

### <a name="workload-managers"></a><span data-ttu-id="fdfbc-206">Számításifeladat-kezelők</span><span class="sxs-lookup"><span data-stu-id="fdfbc-206">Workload managers</span></span>

<span data-ttu-id="fdfbc-207">Az alábbiakban néhány példa látható az Azure-infrastruktúrában futtatható fürt- és számításifeladat-kezelőkre.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-207">The following are examples of cluster and workload managers that can run in Azure infrastructure.</span></span> <span data-ttu-id="fdfbc-208">Önálló fürtöket hozhat létre az Azure-beli virtuális gépeken, illetve adatlöketet vihet át rájuk egy helyszíni fürtről.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-208">Create stand-alone clusters in Azure VMs or burst to Azure VMs from an on-premises cluster.</span></span>

- [<span data-ttu-id="fdfbc-209">Alces Flight Compute</span><span class="sxs-lookup"><span data-stu-id="fdfbc-209">Alces Flight Compute</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [<span data-ttu-id="fdfbc-210">TIBCO DataSynapse GridServer</span><span class="sxs-lookup"><span data-stu-id="fdfbc-210">TIBCO DataSynapse GridServer</span></span>](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [<span data-ttu-id="fdfbc-211">Bright Cluster Manager</span><span class="sxs-lookup"><span data-stu-id="fdfbc-211">Bright Cluster Manager</span></span>](http://www.brightcomputing.com/technology-partners/microsoft)
- [<span data-ttu-id="fdfbc-212">IBM Spectrum Symphony és Symphony LSF</span><span class="sxs-lookup"><span data-stu-id="fdfbc-212">IBM Spectrum Symphony and Symphony LSF</span></span>](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [<span data-ttu-id="fdfbc-213">PBS Pro</span><span class="sxs-lookup"><span data-stu-id="fdfbc-213">PBS Pro</span></span>](http://pbspro.org)
- [<span data-ttu-id="fdfbc-214">Altair</span><span class="sxs-lookup"><span data-stu-id="fdfbc-214">Altair</span></span>](http://www.altair.com/)
- [<span data-ttu-id="fdfbc-215">Rescale</span><span class="sxs-lookup"><span data-stu-id="fdfbc-215">Rescale</span></span>](https://www.rescale.com/azure/)
- [<span data-ttu-id="fdfbc-216">Microsoft HPC Pack</span><span class="sxs-lookup"><span data-stu-id="fdfbc-216">Microsoft HPC Pack</span></span>](https://technet.microsoft.com/library/mt744885.aspx)
  - [<span data-ttu-id="fdfbc-217">Windows rendszerhez készült HPC Pack</span><span class="sxs-lookup"><span data-stu-id="fdfbc-217">HPC Pack for Windows</span></span>](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [<span data-ttu-id="fdfbc-218">Linux rendszerhez készült HPC Pack</span><span class="sxs-lookup"><span data-stu-id="fdfbc-218">HPC Pack for Linux</span></span>](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a><span data-ttu-id="fdfbc-219">Containers</span><span class="sxs-lookup"><span data-stu-id="fdfbc-219">Containers</span></span>

<span data-ttu-id="fdfbc-220">Néhány HPC számítási feladat kezeléséhez tárolók is használhatók.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-220">Containers can also be used to manage some HPC workloads.</span></span>  <span data-ttu-id="fdfbc-221">Az Azure Kubernetes Service-hez (AKS) hasonló szolgáltatásokkal egyszerűen helyezhetők üzembe a felügyelt Kubernetes-fürtök az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-221">Services like the Azure Kubernetes Service (AKS) makes it simple to deploy a managed Kubernetes cluster in Azure.</span></span>

- [<span data-ttu-id="fdfbc-222">Azure Kubernetes Service (AKS)</span><span class="sxs-lookup"><span data-stu-id="fdfbc-222">Azure Kubernetes Service (AKS)</span></span>](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-223">Container Registry</span><span class="sxs-lookup"><span data-stu-id="fdfbc-223">Container Registry</span></span>](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a><span data-ttu-id="fdfbc-224">Cost Management</span><span class="sxs-lookup"><span data-stu-id="fdfbc-224">Cost management</span></span>

<span data-ttu-id="fdfbc-225">A HPC költségeinek kezelése az Azure-on különböző módokon végezhető el.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-225">Managing your HPC cost on Azure can be done through a few different ways.</span></span>  <span data-ttu-id="fdfbc-226">Mindenképpen tekintse át az [Azure vásárlási lehetőségeit](https://azure.microsoft.com/pricing/purchase-options/) a cégének legmegfelelőbb módszer kiválasztásához.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-226">Ensure you've reviewed the [Azure purchasing options](https://azure.microsoft.com/pricing/purchase-options/) to find the method that works best for your organization.</span></span>

<span data-ttu-id="fdfbc-227">Az [alacsony prioritású virtuális gépekkel](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) jelentős költségmegtakarítás mellett használhatja ki a nem használt kapacitásunkat.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-227">[Low priority VMs](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) allow you to take advantage of our unutilized capacity at a significant cost savings.</span></span>

## <a name="security"></a><span data-ttu-id="fdfbc-228">Biztonság</span><span class="sxs-lookup"><span data-stu-id="fdfbc-228">Security</span></span>

<span data-ttu-id="fdfbc-229">Az Azure ajánlott biztonsági eljárásait az [Azure Security dokumentációjában](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) találja.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-229">For an overview of security best practices on Azure, review the [Azure Security Documentation](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>  

<span data-ttu-id="fdfbc-230">A [Felhőalapú teljesítménynövelés](#hybrid-and-cloud-bursting) szakaszban elérhető hálózati konfigurációk mellett érdemes lehet megvalósítani egy küllős konfigurációt a hálózati erőforrások elkülönítése érdekében:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-230">In addition to the network configurations available in the [Cloud Bursting](#hybrid-and-cloud-bursting) section, you may want to implement a hub/spoke configuration to isolate your compute resources:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-231">Küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-231">Implement a hub-spoke network topology in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-232">Az agy egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-232">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="fdfbc-233">A küllők az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-233">The spokes are VNets that peer with the hub, and can be used to isolate workloads.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-234">Küllős hálózati topológia implementálása megosztott szolgáltatásokkal az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-234">Implement a hub-spoke network topology with shared services in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-235">Ez a referenciaarchitektúra a küllős referenciaarchitektúrára épít, hogy az összes küllő által használható megosztott szolgáltatásokat lehessen foglalni a központba.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-235">This reference architecture builds on the hub-spoke reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a><span data-ttu-id="fdfbc-236">HPC-alkalmazások</span><span class="sxs-lookup"><span data-stu-id="fdfbc-236">HPC applications</span></span>

<span data-ttu-id="fdfbc-237">Egyéni vagy kereskedelmi HPC-alkalmazások futtatása az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-237">Run custom or commercial HPC applications in Azure.</span></span> <span data-ttu-id="fdfbc-238">Az ebben a szakaszban szereplő számos példa tesztelve lett, hogy hatékonyan lehessen méretezni további virtuális gépekkel vagy számítási magokkal.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-238">Several examples in this section are benchmarked to scale efficiently with additional VMs or compute cores.</span></span> <span data-ttu-id="fdfbc-239">Az [Azure Marketplace-en](https://azuremarketplace.microsoft.com/marketplace) üzembe helyezésre kész megoldásokat talál.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-239">Visit the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) for ready-to-deploy solutions.</span></span>

> [!NOTE]
> <span data-ttu-id="fdfbc-240">Minden kereskedelmi alkalmazás szállítójánál érdeklődjön a felhőbeli futtatásra vonatkozó licencelési vagy egyéb korlátozásokról.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-240">Check with the vendor of any commercial application for licensing or other restrictions for running in the cloud.</span></span> <span data-ttu-id="fdfbc-241">Nem minden szállító kínál használatalapú licencet.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-241">Not all vendors offer pay-as-you-go licensing.</span></span> <span data-ttu-id="fdfbc-242">Lehet, hogy a megoldásához licenckiszolgálót kell használnia a felhőben, vagy helyszíni licenckiszolgálóhoz kell csatlakoznia.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-242">You might need a licensing server in the cloud for your solution, or connect to an on-premises license server.</span></span>

### <a name="engineering-applications"></a><span data-ttu-id="fdfbc-243">Mérnöki alkalmazások</span><span class="sxs-lookup"><span data-stu-id="fdfbc-243">Engineering applications</span></span>

- [<span data-ttu-id="fdfbc-244">Altair RADIOSS</span><span class="sxs-lookup"><span data-stu-id="fdfbc-244">Altair RADIOSS</span></span>](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [<span data-ttu-id="fdfbc-245">ANSYS CFD</span><span class="sxs-lookup"><span data-stu-id="fdfbc-245">ANSYS CFD</span></span>](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [<span data-ttu-id="fdfbc-246">MATLAB Distributed Computing Server</span><span class="sxs-lookup"><span data-stu-id="fdfbc-246">MATLAB Distributed Computing Server</span></span>](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-247">StarCCM+</span><span class="sxs-lookup"><span data-stu-id="fdfbc-247">StarCCM+</span></span>](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [<span data-ttu-id="fdfbc-248">OpenFOAM</span><span class="sxs-lookup"><span data-stu-id="fdfbc-248">OpenFOAM</span></span>](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a><span data-ttu-id="fdfbc-249">Grafika és renderelés</span><span class="sxs-lookup"><span data-stu-id="fdfbc-249">Graphics and rendering</span></span>

- <span data-ttu-id="fdfbc-250">[Autodesk Maya, 3ds Max és Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) az Azure Batch-en</span><span class="sxs-lookup"><span data-stu-id="fdfbc-250">[Autodesk Maya, 3ds Max, and Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) on Azure Batch</span></span>

### <a name="ai-and-deep-learning"></a><span data-ttu-id="fdfbc-251">AI és mély tanulás</span><span class="sxs-lookup"><span data-stu-id="fdfbc-251">AI and deep learning</span></span>

- [<span data-ttu-id="fdfbc-252">Microsoft Cognitive Toolkit</span><span class="sxs-lookup"><span data-stu-id="fdfbc-252">Microsoft Cognitive Toolkit</span></span>](/cognitive-toolkit/cntk-on-azure)
- [<span data-ttu-id="fdfbc-253">Deep Learning VM</span><span class="sxs-lookup"><span data-stu-id="fdfbc-253">Deep Learning VM</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [<span data-ttu-id="fdfbc-254">Batch Shipyard mélytanulási módszerek</span><span class="sxs-lookup"><span data-stu-id="fdfbc-254">Batch Shipyard recipes for deep learning</span></span>](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a><span data-ttu-id="fdfbc-255">MPI-szolgáltatók</span><span class="sxs-lookup"><span data-stu-id="fdfbc-255">MPI Providers</span></span>

- [<span data-ttu-id="fdfbc-256">Microsoft MPI</span><span class="sxs-lookup"><span data-stu-id="fdfbc-256">Microsoft MPI</span></span>](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a><span data-ttu-id="fdfbc-257">Távoli vizualizáció</span><span class="sxs-lookup"><span data-stu-id="fdfbc-257">Remote visualization</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="fdfbc-258">Linux rendszerű virtuális asztali környezetek a Citrix-szel</span><span class="sxs-lookup"><span data-stu-id="fdfbc-258">Linux virtual desktops with Citrix</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="fdfbc-259">VDI-környezetet hozhat létre Linux rendszerű asztali környezetekhez a Citrix használatával az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-259">Build a VDI environment for Linux Desktops using Citrix on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a><span data-ttu-id="fdfbc-260">Teljesítménymérések</span><span class="sxs-lookup"><span data-stu-id="fdfbc-260">Performance Benchmarks</span></span>

- [<span data-ttu-id="fdfbc-261">Számításiteljesítmény-mérések</span><span class="sxs-lookup"><span data-stu-id="fdfbc-261">Compute benchmarks</span></span>](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a><span data-ttu-id="fdfbc-262">Ügyfelek történetei</span><span class="sxs-lookup"><span data-stu-id="fdfbc-262">Customer stories</span></span>

<span data-ttu-id="fdfbc-263">Sok ügyfél ért el hatalmas sikereket az Azure HPC számítási feladatokhoz való használata során.</span><span class="sxs-lookup"><span data-stu-id="fdfbc-263">There are a number of customers who have seen great success by using Azure for their HPC workloads.</span></span>  <span data-ttu-id="fdfbc-264">Az alábbiakban néhány ilyen ügyfél esettanulmányát láthatja:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-264">You can find a few of these customer case studies below:</span></span>

- [<span data-ttu-id="fdfbc-265">ANEO</span><span class="sxs-lookup"><span data-stu-id="fdfbc-265">ANEO</span></span>](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [<span data-ttu-id="fdfbc-266">AXA Global P&C</span><span class="sxs-lookup"><span data-stu-id="fdfbc-266">AXA Global P&C</span></span>](https://customers.microsoft.com/story/axa-global-p-and-c)
- [<span data-ttu-id="fdfbc-267">Axioma</span><span class="sxs-lookup"><span data-stu-id="fdfbc-267">Axioma</span></span>](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [<span data-ttu-id="fdfbc-268">d3View</span><span class="sxs-lookup"><span data-stu-id="fdfbc-268">d3View</span></span>](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [<span data-ttu-id="fdfbc-269">EFS</span><span class="sxs-lookup"><span data-stu-id="fdfbc-269">EFS</span></span>](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [<span data-ttu-id="fdfbc-270">Hymans Robertson</span><span class="sxs-lookup"><span data-stu-id="fdfbc-270">Hymans Robertson</span></span>](https://customers.microsoft.com/story/hymans-robertson)
- [<span data-ttu-id="fdfbc-271">MetLife</span><span class="sxs-lookup"><span data-stu-id="fdfbc-271">MetLife</span></span>](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [<span data-ttu-id="fdfbc-272">Microsoft Research</span><span class="sxs-lookup"><span data-stu-id="fdfbc-272">Microsoft Research</span></span>](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [<span data-ttu-id="fdfbc-273">Milliman</span><span class="sxs-lookup"><span data-stu-id="fdfbc-273">Milliman</span></span>](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [<span data-ttu-id="fdfbc-274">Mitsubishi UFJ Securities International</span><span class="sxs-lookup"><span data-stu-id="fdfbc-274">Mitsubishi UFJ Securities International</span></span>](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [<span data-ttu-id="fdfbc-275">NeuroInitiative</span><span class="sxs-lookup"><span data-stu-id="fdfbc-275">NeuroInitiative</span></span>](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [<span data-ttu-id="fdfbc-276">Schlumberger</span><span class="sxs-lookup"><span data-stu-id="fdfbc-276">Schlumberger</span></span>](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [<span data-ttu-id="fdfbc-277">Towers Watson</span><span class="sxs-lookup"><span data-stu-id="fdfbc-277">Towers Watson</span></span>](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a><span data-ttu-id="fdfbc-278">További fontos információk</span><span class="sxs-lookup"><span data-stu-id="fdfbc-278">Other important information</span></span>

- <span data-ttu-id="fdfbc-279">A nagy méretű számítási feladatok futtatásának megkísérlése előtt győződjön meg arról, hogy megnövelte a [vCPU-kvótát](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="fdfbc-279">Ensure your [vCPU quota](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) has been increased before attempting to run large-scale workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fdfbc-280">További lépések</span><span class="sxs-lookup"><span data-stu-id="fdfbc-280">Next steps</span></span>

<span data-ttu-id="fdfbc-281">A legújabb bejelentéseket itt találja:</span><span class="sxs-lookup"><span data-stu-id="fdfbc-281">For the latest announcements, see:</span></span>

- [<span data-ttu-id="fdfbc-282">A Microsoft HPC és Batch csapatának blogja</span><span class="sxs-lookup"><span data-stu-id="fdfbc-282">Microsoft HPC and Batch team blog</span></span>](http://blogs.technet.com/b/windowshpc/)
- <span data-ttu-id="fdfbc-283">Látogasson el az [Azure blogra](https://azure.microsoft.com/blog/tag/hpc/).</span><span class="sxs-lookup"><span data-stu-id="fdfbc-283">Visit the [Azure blog](https://azure.microsoft.com/blog/tag/hpc/).</span></span>

### <a name="microsoft-batch-examples"></a><span data-ttu-id="fdfbc-284">Microsoft Batch – példák</span><span class="sxs-lookup"><span data-stu-id="fdfbc-284">Microsoft Batch Examples</span></span>

<span data-ttu-id="fdfbc-285">Ezek az oktatóanyagok az alkalmazások Microsoft Batch-en való futtatásának részleteit tartalmazzák</span><span class="sxs-lookup"><span data-stu-id="fdfbc-285">These tutorials will provide you with details on running applications on Microsoft Batch</span></span>

- [<span data-ttu-id="fdfbc-286">A fejlesztés megkezdése a Batch használatával</span><span class="sxs-lookup"><span data-stu-id="fdfbc-286">Get started developing with Batch</span></span>](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-287">Az Azure Batch kódmintáinak használata</span><span class="sxs-lookup"><span data-stu-id="fdfbc-287">Use Azure Batch code samples</span></span>](https://github.com/Azure/azure-batch-samples)
- [<span data-ttu-id="fdfbc-288">Alacsony prioritású virtuális gépek használata a Batch szolgáltatással</span><span class="sxs-lookup"><span data-stu-id="fdfbc-288">Use low-priority VMs with Batch</span></span>](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="fdfbc-289">Tárolóalapú HPC számítási feladatok futtatása a Batch Shipyard használatával</span><span class="sxs-lookup"><span data-stu-id="fdfbc-289">Run containerized HPC workloads with Batch Shipyard</span></span>](https://github.com/Azure/batch-shipyard)
- [<span data-ttu-id="fdfbc-290">Párhuzamos R számítási feladatok futtatása a Batch szolgáltatásban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-290">Run parallel R workloads on Batch</span></span>](https://github.com/Azure/doAzureParallel)
- [<span data-ttu-id="fdfbc-291">Igény szerinti Spark-feladatok futtatása a Batch szolgáltatásban</span><span class="sxs-lookup"><span data-stu-id="fdfbc-291">Run on-demand Spark jobs on Batch</span></span>](https://github.com/Azure/aztk)
- [<span data-ttu-id="fdfbc-292">Nagy számítási igényű virtuális gépek használata Batch-készletekben</span><span class="sxs-lookup"><span data-stu-id="fdfbc-292">Use compute-intensive VMs in Batch pools</span></span>](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)