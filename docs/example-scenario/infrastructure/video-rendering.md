---
title: 3D-s videórenderelés
titleSuffix: Azure Example Scenarios
description: Natív HPC számítási feladatokat futtathat az Azure-ban az Azure Batch szolgáltatás használatával.
author: adamboeglin
ms.date: 07/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-video-rendering.png
ms.openlocfilehash: 39b411eacaa1e0f400b59a099e9aa21b1159c921
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908448"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="3b797-103">3D-s videórenderelés az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="3b797-103">3D video rendering on Azure</span></span>

<span data-ttu-id="3b797-104">Videó 3D renderelési az időigényes, amely jelentős mennyiségű Processzor időtartama igényel.</span><span class="sxs-lookup"><span data-stu-id="3b797-104">3D video rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span> <span data-ttu-id="3b797-105">Egyetlen gépen, létrehozni bloberőforrásokhoz videofájl származó statikus objektumokat is órákig vagy akár napokig eltarthat hossza és a videót, akkor állít elő összetettségétől függően.</span><span class="sxs-lookup"><span data-stu-id="3b797-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span> <span data-ttu-id="3b797-106">Számos vállalat vagy költséges, csúcskategóriás fogja megvásárolni ezeket a feladatokat, vagy be, amely is küldhetők be feladatok a nagy renderelő farmokat asztali számítógépeket.</span><span class="sxs-lookup"><span data-stu-id="3b797-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span> <span data-ttu-id="3b797-107">Azonban az Azure Batch előnyeit kihasználva Ez a teljesítmény érhetőek el, ha szükséges, és leállításakor magát, ha nem, akkor minden tőkebefektetés nélkül.</span><span class="sxs-lookup"><span data-stu-id="3b797-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="3b797-108">A Batch lehetővé teszi egy egységes felügyeleti élmény és a feladatütemezésben választja, a Windows Server vagy Linux számítási csomópontok.</span><span class="sxs-lookup"><span data-stu-id="3b797-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="3b797-109">A Batch-Csel használhatja a meglévő Windows vagy Linux alkalmazásai, például AutoDesk Maya és a Blender, nagy léptékű futtatásához renderelési feladatok az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3b797-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="3b797-110">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="3b797-110">Relevant use cases</span></span>

<span data-ttu-id="3b797-111">Egyéb alkalmazási helyzetek a következők:</span><span class="sxs-lookup"><span data-stu-id="3b797-111">Other relevant use cases include:</span></span>

- <span data-ttu-id="3b797-112">3D modellezés</span><span class="sxs-lookup"><span data-stu-id="3b797-112">3D modeling</span></span>
- <span data-ttu-id="3b797-113">Vizuális FX (VFX) megjelenítési</span><span class="sxs-lookup"><span data-stu-id="3b797-113">Visual FX (VFX) rendering</span></span>
- <span data-ttu-id="3b797-114">Videó átkódolása</span><span class="sxs-lookup"><span data-stu-id="3b797-114">Video transcoding</span></span>
- <span data-ttu-id="3b797-115">Képfeldolgozás, szín javítása és átméretezése</span><span class="sxs-lookup"><span data-stu-id="3b797-115">Image processing, color correction, and resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="3b797-116">Architektúra</span><span class="sxs-lookup"><span data-stu-id="3b797-116">Architecture</span></span>

![A következő összetevők kapnak szerepet egy Azure Batch segítségével a Felhőbeli natív HPC-megoldást architektúrájának áttekintése][architecture]

<span data-ttu-id="3b797-118">Ebben a forgatókönyvben egy munkafolyamatot, amely használja az Azure Batch jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="3b797-118">This scenario shows a workflow that uses Azure Batch.</span></span> <span data-ttu-id="3b797-119">Az adatok folyamatok alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="3b797-119">The data flows as follows:</span></span>

1. <span data-ttu-id="3b797-120">Töltse fel a bemeneti fájlok és feldolgozása ezeket a fájlokat az Azure Storage-fiókot az alkalmazásokkal.</span><span class="sxs-lookup"><span data-stu-id="3b797-120">Upload input files and the applications to process those files to your Azure Storage account.</span></span>
2. <span data-ttu-id="3b797-121">Hozzon létre Batch-készlet, számítási csomópontok Batch-fiókban, egy feladatot, amely a készlethez, valamint a feladat a számítási feladatok futtatásához.</span><span class="sxs-lookup"><span data-stu-id="3b797-121">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="3b797-122">Töltse le a bemeneti fájlok és a Batch az alkalmazásokat.</span><span class="sxs-lookup"><span data-stu-id="3b797-122">Download input files and the applications to Batch.</span></span>
4. <span data-ttu-id="3b797-123">Figyelheti a feladat a végrehajtás.</span><span class="sxs-lookup"><span data-stu-id="3b797-123">Monitor task execution.</span></span>
5. <span data-ttu-id="3b797-124">Tevékenység kimenetének feltöltése.</span><span class="sxs-lookup"><span data-stu-id="3b797-124">Upload task output.</span></span>
6. <span data-ttu-id="3b797-125">Kimeneti fájlok letöltése.</span><span class="sxs-lookup"><span data-stu-id="3b797-125">Download output files.</span></span>

<span data-ttu-id="3b797-126">A folyamat leegyszerűsítése érdekében is használhatja a [Batch beépülő modulok, a Maya és a 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="3b797-126">To simplify this process, you could also use the [Batch Plugins for Maya and 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="3b797-127">Összetevők</span><span class="sxs-lookup"><span data-stu-id="3b797-127">Components</span></span>

<span data-ttu-id="3b797-128">Az Azure Batch-listaalkalmazásra épül a következő Azure-technológiák:</span><span class="sxs-lookup"><span data-stu-id="3b797-128">Azure Batch builds upon the following Azure technologies:</span></span>

- <span data-ttu-id="3b797-129">[Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) a fő csomópontot és a számítási erőforrásokat is használhatók.</span><span class="sxs-lookup"><span data-stu-id="3b797-129">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are used for both the head node and the compute resources.</span></span>
- <span data-ttu-id="3b797-130">[Az Azure Storage-fiókok](/azure/storage/common/storage-introduction) szinkronizálás és az adatmegőrzést szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="3b797-130">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
- <span data-ttu-id="3b797-131">[Virtual Machine Scale Sets] [ vmss] CycleCloud számítási erőforrásokat használ.</span><span class="sxs-lookup"><span data-stu-id="3b797-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="3b797-132">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="3b797-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="3b797-133">Gépek méretei az Azure Batch számára elérhető</span><span class="sxs-lookup"><span data-stu-id="3b797-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="3b797-134">Renderelési a legtöbb ügyfél fog válassza ki a magas CPU-teljesítmény-erőforrásokat, miközben más számítási feladatok, a virtual machine scale sets használatával eltérően előfordulhat, hogy válassza ki virtuális gépeket, és számos tényezőtől függ:</span><span class="sxs-lookup"><span data-stu-id="3b797-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

- <span data-ttu-id="3b797-135">Van kötve a memória a futtatni az alkalmazást?</span><span class="sxs-lookup"><span data-stu-id="3b797-135">Is the application being run memory bound?</span></span>
- <span data-ttu-id="3b797-136">Az alkalmazás gpu-k használatához nem kell?</span><span class="sxs-lookup"><span data-stu-id="3b797-136">Does the application need to use GPUs?</span></span>
- <span data-ttu-id="3b797-137">A feladat típusok zavaróan párhuzamos vagy infiniband kapcsolatot igényelnek a szorosan összekapcsolt feladatokhoz?</span><span class="sxs-lookup"><span data-stu-id="3b797-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
- <span data-ttu-id="3b797-138">Gyors i/o számítási csomópontok tároló eléréséhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="3b797-138">Require fast I/O to access storage on the compute Nodes.</span></span>

<span data-ttu-id="3b797-139">Egy számos különböző Virtuálisgép-méretek, amely az alkalmazás a fenti követelmények minden egy az Azure rendelkezik, néhány csak az adott HPC, de még a legkisebb méretű, amellyel egy hatékony rács implementáció:</span><span class="sxs-lookup"><span data-stu-id="3b797-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

- <span data-ttu-id="3b797-140">[HPC-Virtuálisgép-méretek] [ compute-hpc] renderelési kötött CPU jellege miatt általában javasolt az Azure H-sorozatú virtuális gépek.</span><span class="sxs-lookup"><span data-stu-id="3b797-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span> <span data-ttu-id="3b797-141">Az ilyen típusú virtuális gép kifejezetten a magas szintű számítási igényekre épül, 8 és 16 magos vCPU-mérettel érhető el, és szolgáltatások esetében DDR4 memóriával, ideiglenes SSD-tárolóval és a technológia az Intel Haswell E5 rendelkeznek.</span><span class="sxs-lookup"><span data-stu-id="3b797-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
- <span data-ttu-id="3b797-142">[GPU-Virtuálisgép-méretek] [ compute-gpu] GPU-optimalizált virtuális gépek méretek a következők specializált virtuális gépek egy vagy több NVIDIA gpu-k használatával érhető el.</span><span class="sxs-lookup"><span data-stu-id="3b797-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="3b797-143">Ezeket a méreteket képi megjelenítés, nagy számítási igényű és magas grafikai igényű számítási feladatokhoz tervezték.</span><span class="sxs-lookup"><span data-stu-id="3b797-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
- <span data-ttu-id="3b797-144">Hálózati vezérlő, NCv2, az NCv3 és ND méretek nagy számítási és hálózatigényű alkalmazásokra és algoritmusokra, beleértve a CUDA és OpenCL-alapú alkalmazásokat és szimulációkat, mesterséges Intelligencia és a Deep Learning vannak optimalizálva.</span><span class="sxs-lookup"><span data-stu-id="3b797-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="3b797-145">NV-méretek vannak kialakítva és optimalizálva távoli képi megjelenítés, streamelési, játék, kódolási és VDI-forgatókönyvekhez OpenGL, DirectX és hasonló keretrendszereket.</span><span class="sxs-lookup"><span data-stu-id="3b797-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
- <span data-ttu-id="3b797-146">[Memóriahasználatra optimalizált Virtuálisgép-méretek] [ compute-memory] több memóriára szükség, amikor a memóriahasználatra optimalizált Virtuálisgép-méretek kínál a nagyobb memória – Processzor memóriaarányt.</span><span class="sxs-lookup"><span data-stu-id="3b797-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
- <span data-ttu-id="3b797-147">[Általános célú virtuális gépek méreteit] [ compute-general] General-purpose Virtuálisgép-méretek is elérhetők, és adja meg a kiegyensúlyozott Processzor-memória arány.</span><span class="sxs-lookup"><span data-stu-id="3b797-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="3b797-148">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="3b797-148">Alternatives</span></span>

<span data-ttu-id="3b797-149">Ha jobban szabályozhatja az Azure-ban a renderelési környezet igényelnek, vagy egy hibrid megoldás kell, majd CycleCloud számítástechnika segítségével összehangolhatja a felhőben egy IaaS rács.</span><span class="sxs-lookup"><span data-stu-id="3b797-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="3b797-150">Azure Batch a mögöttes Azure technológiákat használ, így létrehozását és karbantartását az IaaS rács hatékonyabbá teszi a folyamatot.</span><span class="sxs-lookup"><span data-stu-id="3b797-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="3b797-151">További, és ismerje meg a tervezési alapelveket használja a következő hivatkozásra:</span><span class="sxs-lookup"><span data-stu-id="3b797-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="3b797-152">Az Azure-ban elérhető összes HPC-megoldások teljes körű áttekintést lásd a cikk [HPC, Batch és Big Compute megoldások az Azure virtuális gépekkel][hpc-alt-solutions]</span><span class="sxs-lookup"><span data-stu-id="3b797-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="3b797-153">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="3b797-153">Availability</span></span>

<span data-ttu-id="3b797-154">Az Azure Batch-összetevők figyelésének, szolgáltatások, eszközök és API-k széles keresztül érhető el.</span><span class="sxs-lookup"><span data-stu-id="3b797-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="3b797-155">Figyelés a következő cikkben található a [figyelő Batch-megoldások] [ batch-monitor] cikk.</span><span class="sxs-lookup"><span data-stu-id="3b797-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="3b797-156">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="3b797-156">Scalability</span></span>

<span data-ttu-id="3b797-157">Belül egy fiók is, vagy a méretezési csoport manuális intézkedés révén, vagy egy képlet alapján az Azure Batch-mérőszámok, az Azure Batch Pools automatikusan skálázhatók.</span><span class="sxs-lookup"><span data-stu-id="3b797-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="3b797-158">Méretezhetőség további információkért tekintse meg a cikket [hozzon létre egy Batch-készletben lévő csomópontok méretezése egy automatikus méretezési képlet][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="3b797-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="3b797-159">Biztonság</span><span class="sxs-lookup"><span data-stu-id="3b797-159">Security</span></span>

<span data-ttu-id="3b797-160">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="3b797-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="3b797-161">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="3b797-161">Resiliency</span></span>

<span data-ttu-id="3b797-162">Bár az Azure Batch szolgáltatásban jelenleg nem nincs feladatátvételi képességének, javasolt egy nem tervezett kimaradás esetén a rendelkezésre állás biztosítása érdekében az alábbi lépéseket követve:</span><span class="sxs-lookup"><span data-stu-id="3b797-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

- <span data-ttu-id="3b797-163">Egy másik Azure-beli helyen, egy másik tárfiókkal az Azure Batch-fiók létrehozása</span><span class="sxs-lookup"><span data-stu-id="3b797-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
- <span data-ttu-id="3b797-164">Az azonos csomópontkészletek ugyanazzal a névvel, nulla lefoglalt csomópontok létrehozása</span><span class="sxs-lookup"><span data-stu-id="3b797-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
- <span data-ttu-id="3b797-165">Győződjön meg, hogy az alkalmazások létrehozása és a másodlagos tárfiók frissítése</span><span class="sxs-lookup"><span data-stu-id="3b797-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
- <span data-ttu-id="3b797-166">Bemeneti fájlok feltöltése és a másodlagos Azure-Batch-fiók-feladatok elküldése</span><span class="sxs-lookup"><span data-stu-id="3b797-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="3b797-167">A forgatókönyv megvalósításához</span><span class="sxs-lookup"><span data-stu-id="3b797-167">Deploy the scenario</span></span>

### <a name="create-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="3b797-168">Azure Batch-fiók és -készletek létrehozása manuálisan</span><span class="sxs-lookup"><span data-stu-id="3b797-168">Create an Azure Batch account and pools manually</span></span>

<span data-ttu-id="3b797-169">Ebben a forgatókönyvben be az Azure Batch működése közben az Azure Batch Labs példaként Szolgáltatottszoftver-megoldás, amely az ügyfelek fejlesztette ki, amely azt mutatja be:</span><span class="sxs-lookup"><span data-stu-id="3b797-169">This scenario demonstrates how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="3b797-170">[Az Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="3b797-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploy-the-components"></a><span data-ttu-id="3b797-171">Az összetevők üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3b797-171">Deploy the components</span></span>

<span data-ttu-id="3b797-172">A sablon telepíti:</span><span class="sxs-lookup"><span data-stu-id="3b797-172">The template will deploy:</span></span>

- <span data-ttu-id="3b797-173">Egy új Azure Batch-fiók</span><span class="sxs-lookup"><span data-stu-id="3b797-173">A new Azure Batch account</span></span>
- <span data-ttu-id="3b797-174">Storage-fiók</span><span class="sxs-lookup"><span data-stu-id="3b797-174">A storage account</span></span>
- <span data-ttu-id="3b797-175">A batch-fiókhoz társított csomópontkészletek</span><span class="sxs-lookup"><span data-stu-id="3b797-175">A node pool associated with the batch account</span></span>
- <span data-ttu-id="3b797-176">A csomópont készlethez Canonical Ubuntu-lemezképekkel A2 v2 virtuális gép használatára lesz konfigurálva</span><span class="sxs-lookup"><span data-stu-id="3b797-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
- <span data-ttu-id="3b797-177">A csomópont készlethez virtuális gépek kezdetben tartalmazni fog, és el kell azt manuálisan hozzáadni a virtuális gépek méretezési</span><span class="sxs-lookup"><span data-stu-id="3b797-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>
<!-- markdownlint-enable MD033 -->

<span data-ttu-id="3b797-178">[További tudnivalók a Resource Manager-sablonok][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="3b797-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="3b797-179">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="3b797-179">Pricing</span></span>

<span data-ttu-id="3b797-180">Azure Batch használatának költségét a készleteket, és mennyi ideig vannak lefoglalva a virtuális gépek által használt Virtuálisgép-méretek függ, és ott futtatja, nem egy Azure Batch-fiók létrehozása társított költségek nélkül.</span><span class="sxs-lookup"><span data-stu-id="3b797-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="3b797-181">Tárolás és adatkezelés kimenő figyelembe kell venni, mivel ezek további költségekkel is érvényes lesz.</span><span class="sxs-lookup"><span data-stu-id="3b797-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="3b797-182">Az alábbi parancsok példák, amelyek használatával a kiszolgálókat különböző számú 8 órán belül befejeződik egy feladat sikerült felmerülő költségek:</span><span class="sxs-lookup"><span data-stu-id="3b797-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

- <span data-ttu-id="3b797-183">Nagy teljesítményű CPU 100 virtuális gépet: [Költségbecslés][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="3b797-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="3b797-184">100 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom</span><span class="sxs-lookup"><span data-stu-id="3b797-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

- <span data-ttu-id="3b797-185">Nagy teljesítményű CPU 50 virtuális gépek: [Költségbecslés][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="3b797-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="3b797-186">50 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom</span><span class="sxs-lookup"><span data-stu-id="3b797-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

- <span data-ttu-id="3b797-187">Nagy teljesítményű CPU 10 virtuális gépek: [Költségbecslés][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="3b797-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>

  <span data-ttu-id="3b797-188">10 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom</span><span class="sxs-lookup"><span data-stu-id="3b797-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="pricing-for-low-priority-vms"></a><span data-ttu-id="3b797-189">Alacsony prioritású virtuális gépek díjszabása</span><span class="sxs-lookup"><span data-stu-id="3b797-189">Pricing for low-priority VMs</span></span>

<span data-ttu-id="3b797-190">Az Azure Batch alacsony prioritású virtuális gépek használatát is támogatja a potenciálisan megadhat egy jelentős megtakarítás csomópontkészletek.</span><span class="sxs-lookup"><span data-stu-id="3b797-190">Azure Batch also supports the use of low-priority VMs in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="3b797-191">További információk, beleértve a standard szintű virtuális gépek és az alacsony prioritású virtuális gépek, ár összehasonlítását: [Azure Batch szolgáltatás díjszabása][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="3b797-191">For more information, including a price comparison between standard VMs and low-priority VMs, see [Azure Batch Pricing][batch-pricing].</span></span>

> [!NOTE]
> <span data-ttu-id="3b797-192">Alacsony prioritású virtuális gépek csak olyan, megfelelő az egyes alkalmazások és számítási feladatok.</span><span class="sxs-lookup"><span data-stu-id="3b797-192">Low-priority VMs are only suitable for certain applications and workloads.</span></span>

## <a name="related-resources"></a><span data-ttu-id="3b797-193">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="3b797-193">Related resources</span></span>

<span data-ttu-id="3b797-194">[Az Azure Batch áttekintése][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="3b797-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="3b797-195">[Az Azure Batch – dokumentáció][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="3b797-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="3b797-196">[Tárolók az Azure Batch használatával][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="3b797-196">[Using containers on Azure Batch][batch-containers]</span></span>

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