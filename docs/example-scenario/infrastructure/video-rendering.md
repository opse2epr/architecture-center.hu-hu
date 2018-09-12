---
title: 3D videó megjelenítése az Azure-ban
description: Natív HPC számítási feladatok futtatása az Azure-ban, az Azure Batch szolgáltatással
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: 723d437671c52dc9f717bef9641663d0e7a8fbc4
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389349"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="246f3-103">3D videó megjelenítése az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="246f3-103">3D video rendering on Azure</span></span>

<span data-ttu-id="246f3-104">3D renderelési az időigényes, amely a CPU-idő co teljes jelentős mennyiségű igényel.</span><span class="sxs-lookup"><span data-stu-id="246f3-104">3D rendering is a time consuming process that requires a significant amount of CPU time co complete.</span></span>  <span data-ttu-id="246f3-105">Egyetlen gépen, létrehozni bloberőforrásokhoz videofájl származó statikus objektumokat is órákig vagy akár napokig eltarthat hossza és a videót, akkor állít elő összetettségétől függően.</span><span class="sxs-lookup"><span data-stu-id="246f3-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span>  <span data-ttu-id="246f3-106">Számos vállalat vagy költséges, csúcskategóriás fogja megvásárolni ezeket a feladatokat, vagy be, amely is küldhetők be feladatok a nagy renderelő farmokat asztali számítógépeket.</span><span class="sxs-lookup"><span data-stu-id="246f3-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span>  <span data-ttu-id="246f3-107">Azonban az Azure Batch előnyeit kihasználva Ez a teljesítmény érhetőek el, ha szükséges, és leállításakor magát, ha nem, akkor minden tőkebefektetés nélkül.</span><span class="sxs-lookup"><span data-stu-id="246f3-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="246f3-108">A Batch lehetővé teszi egy egységes felügyeleti élmény és a feladatütemezésben választja, a Windows Server vagy Linux számítási csomópontok.</span><span class="sxs-lookup"><span data-stu-id="246f3-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="246f3-109">A Batch-Csel használhatja a meglévő Windows vagy Linux alkalmazásai, például AutoDesk Maya és a Blender, nagy léptékű futtatásához renderelési feladatok az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="246f3-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="246f3-110">Kapcsolódó alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="246f3-110">Related use cases</span></span>

<span data-ttu-id="246f3-111">Ebben a forgatókönyvben ezek hasonló használati esetek, vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="246f3-111">Consider this scenario for these similar use cases:</span></span>

* <span data-ttu-id="246f3-112">3D modellezés</span><span class="sxs-lookup"><span data-stu-id="246f3-112">3D Modeling</span></span>
* <span data-ttu-id="246f3-113">Vizuális FX (VFX) megjelenítési</span><span class="sxs-lookup"><span data-stu-id="246f3-113">Visual FX (VFX) Rendering</span></span>
* <span data-ttu-id="246f3-114">Videó átkódolása</span><span class="sxs-lookup"><span data-stu-id="246f3-114">Video transcoding</span></span>
* <span data-ttu-id="246f3-115">Képfeldolgozás, szín javítása és átméretezése</span><span class="sxs-lookup"><span data-stu-id="246f3-115">Image processing, color correction, & resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="246f3-116">Architektúra</span><span class="sxs-lookup"><span data-stu-id="246f3-116">Architecture</span></span>

![A következő összetevők kapnak szerepet egy Azure Batch segítségével a Felhőbeli natív HPC-megoldást architektúrájának áttekintése][architecture]

<span data-ttu-id="246f3-118">Ezen forgatókönyv esetén a munkafolyamatot mutatja be, amikor Azure Batch, a data-adatfolyamok használatával az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="246f3-118">This sample scenario covers the workflow when using Azure Batch, the data flows as follows:</span></span>

1. <span data-ttu-id="246f3-119">Töltse fel a bemeneti fájlok és feldolgozása ezeket a fájlokat az Azure Storage-fiókot az alkalmazásokkal</span><span class="sxs-lookup"><span data-stu-id="246f3-119">Upload input files and the applications to process those files to your Azure Storage account</span></span>
2. <span data-ttu-id="246f3-120">Hozzon létre Batch-készlet, számítási csomópontok Batch-fiókban, egy feladatot, amely a készlethez, valamint a feladat a számítási feladatok futtatásához.</span><span class="sxs-lookup"><span data-stu-id="246f3-120">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="246f3-121">Töltse le a bemeneti fájlok és az alkalmazásokat a Batch</span><span class="sxs-lookup"><span data-stu-id="246f3-121">Download input files and the applications to Batch</span></span>
4. <span data-ttu-id="246f3-122">Tevékenységek végrehajtásának figyelése</span><span class="sxs-lookup"><span data-stu-id="246f3-122">Monitor task execution</span></span>
5. <span data-ttu-id="246f3-123">Tevékenység kimenetének feltöltése</span><span class="sxs-lookup"><span data-stu-id="246f3-123">Upload task output</span></span>
6. <span data-ttu-id="246f3-124">Kimeneti fájlok letöltése</span><span class="sxs-lookup"><span data-stu-id="246f3-124">Download output files</span></span>

<span data-ttu-id="246f3-125">A folyamat leegyszerűsítése érdekében is használhatja a [Batch beépülő modulok, a Maya és a 3ds Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="246f3-125">To simplify this process, you could also use the [Batch Plugins for Maya & 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="246f3-126">Összetevők</span><span class="sxs-lookup"><span data-stu-id="246f3-126">Components</span></span>

<span data-ttu-id="246f3-127">Az Azure Batch-listaalkalmazásra épül a következő Azure-technológiák:</span><span class="sxs-lookup"><span data-stu-id="246f3-127">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="246f3-128">[Erőforráscsoportok] [ resource-groups] Azure-erőforrások logikai tárolója.</span><span class="sxs-lookup"><span data-stu-id="246f3-128">[Resource Groups][resource-groups] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="246f3-129">[Virtuális hálózatok] [ vnet] szolgálnak a fő csomópont és a számítási erőforrások</span><span class="sxs-lookup"><span data-stu-id="246f3-129">[Virtual Networks][vnet] are used to for both the Head Node and Compute resources</span></span>
* <span data-ttu-id="246f3-130">[Tárolási] [ storage] fiókok használhatók a szinkronizálás és adatmegőrzés</span><span class="sxs-lookup"><span data-stu-id="246f3-130">[Storage][storage] accounts are used for the synchronization and data retention</span></span>
* <span data-ttu-id="246f3-131">[Virtual Machine Scale Sets] [ vmss] számítási erőforrások CycleCloud által használt</span><span class="sxs-lookup"><span data-stu-id="246f3-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources</span></span>

## <a name="considerations"></a><span data-ttu-id="246f3-132">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="246f3-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="246f3-133">Gépek méretei az Azure Batch számára elérhető</span><span class="sxs-lookup"><span data-stu-id="246f3-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="246f3-134">Renderelési a legtöbb ügyfél fog válassza ki a magas CPU-teljesítmény-erőforrásokat, miközben más számítási feladatok, a virtual machine scale sets használatával eltérően előfordulhat, hogy válassza ki virtuális gépeket, és számos tényezőtől függ:</span><span class="sxs-lookup"><span data-stu-id="246f3-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

* <span data-ttu-id="246f3-135">Van kötve a memória a futtatni az alkalmazást?</span><span class="sxs-lookup"><span data-stu-id="246f3-135">Is the application being run memory bound?</span></span>
* <span data-ttu-id="246f3-136">Az alkalmazás gpu-k használatához nem kell?</span><span class="sxs-lookup"><span data-stu-id="246f3-136">Does the application need to use GPUs?</span></span> 
* <span data-ttu-id="246f3-137">A feladat típusok zavaróan párhuzamos vagy infiniband kapcsolatot igényelnek a szorosan összekapcsolt feladatokhoz?</span><span class="sxs-lookup"><span data-stu-id="246f3-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
* <span data-ttu-id="246f3-138">Gyors i/o-tárhely a számítási csomópontok megkövetelése</span><span class="sxs-lookup"><span data-stu-id="246f3-138">Require fast I/O to storage on the compute Nodes</span></span>

<span data-ttu-id="246f3-139">Egy számos különböző Virtuálisgép-méretek, amely az alkalmazás a fenti követelmények minden egy az Azure rendelkezik, néhány csak az adott HPC, de még a legkisebb méretű, amellyel egy hatékony rács implementáció:</span><span class="sxs-lookup"><span data-stu-id="246f3-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

* <span data-ttu-id="246f3-140">[HPC-Virtuálisgép-méretek] [ compute-hpc] renderelési kötött CPU jellege miatt általában javasolt az Azure H-sorozatú virtuális gépek.</span><span class="sxs-lookup"><span data-stu-id="246f3-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span>  <span data-ttu-id="246f3-141">Az ilyen típusú virtuális gép kifejezetten a magas szintű számítási igényekre épül, 8 és 16 magos vCPU-mérettel érhető el, és szolgáltatások esetében DDR4 memóriával, ideiglenes SSD-tárolóval és a technológia az Intel Haswell E5 rendelkeznek.</span><span class="sxs-lookup"><span data-stu-id="246f3-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
* <span data-ttu-id="246f3-142">[GPU-Virtuálisgép-méretek] [ compute-gpu] GPU-optimalizált virtuális gépek méretek a következők specializált virtuális gépek egy vagy több NVIDIA gpu-k használatával érhető el.</span><span class="sxs-lookup"><span data-stu-id="246f3-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="246f3-143">Ezeket a méreteket képi megjelenítés, nagy számítási igényű és magas grafikai igényű számítási feladatokhoz tervezték.</span><span class="sxs-lookup"><span data-stu-id="246f3-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
* <span data-ttu-id="246f3-144">Hálózati vezérlő, NCv2, az NCv3 és ND méretek nagy számítási és hálózatigényű alkalmazásokra és algoritmusokra, beleértve a CUDA és OpenCL-alapú alkalmazásokat és szimulációkat, mesterséges Intelligencia és a Deep Learning vannak optimalizálva.</span><span class="sxs-lookup"><span data-stu-id="246f3-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="246f3-145">NV-méretek vannak kialakítva és optimalizálva távoli képi megjelenítés, streamelési, játék, kódolási és VDI-forgatókönyvekhez OpenGL, DirectX és hasonló keretrendszereket.</span><span class="sxs-lookup"><span data-stu-id="246f3-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
* <span data-ttu-id="246f3-146">[Memóriahasználatra optimalizált Virtuálisgép-méretek] [ compute-memory] több memóriára szükség, amikor a memóriahasználatra optimalizált Virtuálisgép-méretek kínál a nagyobb memória – Processzor memóriaarányt.</span><span class="sxs-lookup"><span data-stu-id="246f3-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
* <span data-ttu-id="246f3-147">[Általános célú virtuális gépek méreteit] [ compute-general] General-purpose Virtuálisgép-méretek is elérhetők, és adja meg a kiegyensúlyozott Processzor-memória arány.</span><span class="sxs-lookup"><span data-stu-id="246f3-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="246f3-148">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="246f3-148">Alternatives</span></span>

<span data-ttu-id="246f3-149">Ha jobban szabályozhatja az Azure-ban a renderelési környezet igényelnek, vagy egy hibrid megoldás kell, majd CycleCloud számítástechnika segítségével összehangolhatja a felhőben egy IaaS rács.</span><span class="sxs-lookup"><span data-stu-id="246f3-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="246f3-150">Azure Batch a mögöttes Azure technológiákat használ, így létrehozását és karbantartását az IaaS rács hatékonyabbá teszi a folyamatot.</span><span class="sxs-lookup"><span data-stu-id="246f3-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="246f3-151">További, és ismerje meg a tervezési alapelveket használja a következő hivatkozásra:</span><span class="sxs-lookup"><span data-stu-id="246f3-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="246f3-152">Az Azure-ban elérhető összes HPC-megoldások teljes körű áttekintést lásd a cikk [HPC, Batch és Big Compute megoldások az Azure virtuális gépekkel][hpc-alt-solutions]</span><span class="sxs-lookup"><span data-stu-id="246f3-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="246f3-153">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="246f3-153">Availability</span></span>

<span data-ttu-id="246f3-154">Az Azure Batch-összetevők figyelésének, szolgáltatások, eszközök és API-k széles keresztül érhető el.</span><span class="sxs-lookup"><span data-stu-id="246f3-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="246f3-155">Figyelés a következő cikkben található a [figyelő Batch-megoldások] [ batch-monitor] cikk.</span><span class="sxs-lookup"><span data-stu-id="246f3-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="246f3-156">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="246f3-156">Scalability</span></span>

<span data-ttu-id="246f3-157">Belül egy fiók is, vagy a méretezési csoport manuális intézkedés révén, vagy egy képlet alapján az Azure Batch-mérőszámok, az Azure Batch Pools automatikusan skálázhatók.</span><span class="sxs-lookup"><span data-stu-id="246f3-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="246f3-158">Méretezhetőség további információkért tekintse meg a cikket [hozzon létre egy Batch-készletben lévő csomópontok méretezése egy automatikus méretezési képlet][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="246f3-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="246f3-159">Biztonság</span><span class="sxs-lookup"><span data-stu-id="246f3-159">Security</span></span>

<span data-ttu-id="246f3-160">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="246f3-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="246f3-161">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="246f3-161">Resiliency</span></span>

<span data-ttu-id="246f3-162">Bár az Azure Batch szolgáltatásban jelenleg nem nincs feladatátvételi képességének, javasolt egy nem tervezett kimaradás esetén a rendelkezésre állás biztosítása érdekében az alábbi lépéseket követve:</span><span class="sxs-lookup"><span data-stu-id="246f3-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

* <span data-ttu-id="246f3-163">Egy másik Azure-beli helyen, egy másik tárfiókkal az Azure Batch-fiók létrehozása</span><span class="sxs-lookup"><span data-stu-id="246f3-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="246f3-164">Az azonos csomópontkészletek ugyanazzal a névvel, nulla lefoglalt csomópontok létrehozása</span><span class="sxs-lookup"><span data-stu-id="246f3-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
* <span data-ttu-id="246f3-165">Győződjön meg, hogy az alkalmazások létrehozása és a másodlagos tárfiók frissítése</span><span class="sxs-lookup"><span data-stu-id="246f3-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="246f3-166">Bemeneti fájlok feltöltése és a másodlagos Azure-Batch-fiók-feladatok elküldése</span><span class="sxs-lookup"><span data-stu-id="246f3-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="246f3-167">Ez a forgatókönyv megvalósítható</span><span class="sxs-lookup"><span data-stu-id="246f3-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="246f3-168">Azure Batch-fiók és -készletek manuális létrehozása</span><span class="sxs-lookup"><span data-stu-id="246f3-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="246f3-169">Ez a mintaforgatókönyv segíti a tanulási Azure Batch működése közben az Azure Batch Labs példaként Szolgáltatottszoftver-megoldás, amely az ügyfelek fejlesztette ki, amely azt mutatja be:</span><span class="sxs-lookup"><span data-stu-id="246f3-169">This sample scenario helps in learning how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="246f3-170">[Az Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="246f3-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-template"></a><span data-ttu-id="246f3-171">Az Azure Resource Manager-sablon használatával mintaforgatókönyv üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="246f3-171">Deploying the sample scenario using an Azure Resource Manager template</span></span>

<span data-ttu-id="246f3-172">A sablon telepíti:</span><span class="sxs-lookup"><span data-stu-id="246f3-172">The template will deploy:</span></span>

* <span data-ttu-id="246f3-173">Egy új Azure Batch-fiók</span><span class="sxs-lookup"><span data-stu-id="246f3-173">A new Azure Batch account</span></span>
* <span data-ttu-id="246f3-174">Storage-fiók</span><span class="sxs-lookup"><span data-stu-id="246f3-174">A storage account</span></span>
* <span data-ttu-id="246f3-175">A batch-fiókhoz társított csomópontkészletek</span><span class="sxs-lookup"><span data-stu-id="246f3-175">A node pool associated with the batch account</span></span>
* <span data-ttu-id="246f3-176">A csomópont készlethez Canonical Ubuntu-lemezképekkel A2 v2 virtuális gép használatára lesz konfigurálva</span><span class="sxs-lookup"><span data-stu-id="246f3-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
* <span data-ttu-id="246f3-177">A csomópont készlethez virtuális gépek kezdetben tartalmazni fog, és el kell azt manuálisan hozzáadni a virtuális gépek méretezési</span><span class="sxs-lookup"><span data-stu-id="246f3-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="246f3-178">[További tudnivalók a Resource Manager-sablonok][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="246f3-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="246f3-179">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="246f3-179">Pricing</span></span>

<span data-ttu-id="246f3-180">Azure Batch használatának költségét a készleteket, és mennyi ideig vannak lefoglalva a virtuális gépek által használt Virtuálisgép-méretek függ, és ott futtatja, nem egy Azure Batch-fiók létrehozása társított költségek nélkül.</span><span class="sxs-lookup"><span data-stu-id="246f3-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="246f3-181">Tárolás és adatkezelés kimenő figyelembe kell venni, mivel ezek további költségekkel is érvényes lesz.</span><span class="sxs-lookup"><span data-stu-id="246f3-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="246f3-182">Az alábbi parancsok példák, amelyek használatával a kiszolgálókat különböző számú 8 órán belül befejeződik egy feladat sikerült felmerülő költségek:</span><span class="sxs-lookup"><span data-stu-id="246f3-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

* <span data-ttu-id="246f3-183">Nagy teljesítményű CPU 100 virtuális: [Költségbecslés][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="246f3-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="246f3-184">100 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom</span><span class="sxs-lookup"><span data-stu-id="246f3-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="246f3-185">50 nagy teljesítményű Processzorral virtuális: [Költségbecslés][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="246f3-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="246f3-186">50 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom</span><span class="sxs-lookup"><span data-stu-id="246f3-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

* <span data-ttu-id="246f3-187">10, nagy teljesítményű CPU-alapú virtuális gépből: [Költségbecslés][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="246f3-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>
  
  <span data-ttu-id="246f3-188">10 x H16m (16 mag, 225 GB RAM, prémium szintű Storage 512 GB), 2 TB-os Blobtárhelyet, 1 TB-os forgalom</span><span class="sxs-lookup"><span data-stu-id="246f3-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="low-priority-vm-pricing"></a><span data-ttu-id="246f3-189">Alacsony prioritású virtuális gépek díjszabása</span><span class="sxs-lookup"><span data-stu-id="246f3-189">Low-Priority VM Pricing</span></span>

<span data-ttu-id="246f3-190">Az Azure Batch a potenciálisan megadhat egy jelentős megtakarítás csomópontkészletek, alacsony prioritású virtuális gépek \* használatát is támogatja.</span><span class="sxs-lookup"><span data-stu-id="246f3-190">Azure Batch also supports the use of Low-Priority VMs\* in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="246f3-191">Standard szintű virtuális gépek és az alacsony prioritású virtuális gépek között, és a további információk az alacsony prioritású virtuális gépek árát összehasonlításáért lásd: [Batch díjszabása][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="246f3-191">For a price comparison between standard VMs and Low-Priority VMs, and to find out more about Low-Priority VMs, see [Batch Pricing][batch-pricing].</span></span>

<span data-ttu-id="246f3-192">\* Vegye figyelembe, hogy csak bizonyos alkalmazások és munkaterhelések alkalmas lesz az alacsony prioritású virtuális gépen való futtatáshoz.</span><span class="sxs-lookup"><span data-stu-id="246f3-192">\* Please note that only certain applications and workloads will be suitable to run on Low-Priority VMs.</span></span>

## <a name="related-resources"></a><span data-ttu-id="246f3-193">Kapcsolódó erőforrások</span><span class="sxs-lookup"><span data-stu-id="246f3-193">Related Resources</span></span>

<span data-ttu-id="246f3-194">[Az Azure Batch áttekintése][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="246f3-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="246f3-195">[Az Azure Batch – dokumentáció][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="246f3-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="246f3-196">[Tárolók az Azure Batch használatával][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="246f3-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
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
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job