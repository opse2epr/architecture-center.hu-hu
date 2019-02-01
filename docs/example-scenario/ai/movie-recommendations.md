---
title: Filmajánló az Azure-on
description: Gépi tanulás használata film- és termékajánlók, illetve egyéb javaslatok automatizálására Azure-beli modell betanítására szolgáló gépi tanulási megoldások és Azure Data Science Virtual Machine (DSVM) használatával.
author: njray
ms.date: 1/9/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: 9387ab7989695df29df53d7aa4a437010cdd9fdf
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/31/2019
ms.locfileid: "55482994"
---
# <a name="movie-recommendations-on-azure"></a><span data-ttu-id="26b32-103">Filmajánló az Azure-on</span><span class="sxs-lookup"><span data-stu-id="26b32-103">Movie recommendations on Azure</span></span>

<span data-ttu-id="26b32-104">A példa forgatókönyv bemutatja, hogyan egy üzleti használhatja a machine learning-ügyfelek számára, akik a termékekre vonatkozó javaslatok automatizálásához.</span><span class="sxs-lookup"><span data-stu-id="26b32-104">This example scenario shows how a business can use machine learning to automate product recommendations for their customers.</span></span> <span data-ttu-id="26b32-105">Az Azure adatelemzési virtuális gép (DSVM) betanítja a modellt az Azure-on, filmek javasolja a felhasználók számára, amely filmek kapott minősítések alapján történik.</span><span class="sxs-lookup"><span data-stu-id="26b32-105">An Azure Data Science Virtual Machine (DSVM) is used to train a model on Azure that recommends movies to users based on ratings that have been given to movies.</span></span>

<span data-ttu-id="26b32-106">Javaslatok a kiskereskedelmi media hírek a különböző iparágakban hasznos lehet.</span><span class="sxs-lookup"><span data-stu-id="26b32-106">Recommendations can be useful in various industries from retail to news to media.</span></span> <span data-ttu-id="26b32-107">A lehetséges alkalmazások termékekre vonatkozó javaslatok virtuális tárolja, híreket biztosít, vagy ossza meg javaslatokat nyújtó, vagy zene javaslatokat nyújtó tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="26b32-107">Potential applications include providing  product recommendations in a virtual store, providing news or post recommendations, or providing music recommendations.</span></span> <span data-ttu-id="26b32-108">Hagyományosan vállalkozások felvétel kész, és az ügyfelek számára személyre szabott ajánlásokat asszisztensek kellett.</span><span class="sxs-lookup"><span data-stu-id="26b32-108">Traditionally, businesses had to hire and train assistants to make personalized recommendations to customers.</span></span> <span data-ttu-id="26b32-109">Még ma felügyelniük Azure-ügyfél beállítások megértéséhez modelleket taníthat be a nagy mennyiségű személyre szabott ajánlásokat is biztosítunk.</span><span class="sxs-lookup"><span data-stu-id="26b32-109">Today, we can  provide customized recommendations at scale by utilizing Azure to train models to understand customer preferences.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="26b32-110">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="26b32-110">Relevant use cases</span></span>

<span data-ttu-id="26b32-111">Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="26b32-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="26b32-112">Film javaslatok-webhelyen.</span><span class="sxs-lookup"><span data-stu-id="26b32-112">Movie recommendations on a website.</span></span>
* <span data-ttu-id="26b32-113">Fogyasztói termékekre vonatkozó javaslatok a mobilalkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="26b32-113">Consumer product recommendations in a mobile app.</span></span>
* <span data-ttu-id="26b32-114">Hírek javaslattal médiafolyamokhoz.</span><span class="sxs-lookup"><span data-stu-id="26b32-114">News recommendations on streaming media.</span></span>

## <a name="architecture"></a><span data-ttu-id="26b32-115">Architektúra</span><span class="sxs-lookup"><span data-stu-id="26b32-115">Architecture</span></span>

![A gépi tanulási modell betanításához filmjánlásokat architektúrája][architecture]

<span data-ttu-id="26b32-117">Ez a forgatókönyv ismerteti, a képzés és a Spark használatával gépi tanulási modell kiértékelése [legkisebb négyzetek váltakozó] [ als] (ALS) algoritmus egy adatkészleten, film minősítése.</span><span class="sxs-lookup"><span data-stu-id="26b32-117">This scenario covers the training and evaluating of the machine learning model using the Spark [alternating least squares][als] (ALS) algorithm on a dataset of movie ratings.</span></span> <span data-ttu-id="26b32-118">Ez a forgatókönyv lépései a következők következő:</span><span class="sxs-lookup"><span data-stu-id="26b32-118">The steps for this scenario are as following:</span></span>

1. <span data-ttu-id="26b32-119">Az előtér-webhely vagy az app service felhasználói-movie kölcsönhatások, felhasználó, az elem és a numerikus minősítés rekordokat tartalmazó tábláját szereplő korábbi adatokat gyűjti össze.</span><span class="sxs-lookup"><span data-stu-id="26b32-119">The front-end website or app service collects historical data of user-movie interactions, which are represented in a table of user, item, and numerical rating tuples.</span></span>

2. <span data-ttu-id="26b32-120">A begyűjtött korábbi adatok egy blob storage-ban tárolt.</span><span class="sxs-lookup"><span data-stu-id="26b32-120">The collected historical data is stored in a blob storage.</span></span>

3. <span data-ttu-id="26b32-121">A DSVM kísérletezhet velük, a Spark ALS ajánló modell productize gyakran használják.</span><span class="sxs-lookup"><span data-stu-id="26b32-121">A DSVM is often used to experiment with or productize a Spark ALS recommender model.</span></span> <span data-ttu-id="26b32-122">A ALS modell tanítása az betanítási adatkészletet, amely a megfelelő adatok felosztása stratégia alkalmazásával a teljes adatkészlet előállítása.</span><span class="sxs-lookup"><span data-stu-id="26b32-122">The ALS model is trained using a training dataset, which is produced from the overall dataset by applying the appropriate data splitting strategy.</span></span> <span data-ttu-id="26b32-123">Például az adatkészlet felosztása csoportok véletlenszerűen, időrendi sorrendben, is, vagy műanyaggal rétegezett, attól függően, az üzleti követelményt.</span><span class="sxs-lookup"><span data-stu-id="26b32-123">For example, the dataset can be split into sets randomly, chronologically, or stratified, depending on the business requirement.</span></span> <span data-ttu-id="26b32-124">Más gépi tanulási feladatok hasonlóan egy ajánló érvényesítési értékelési mérőszámok használatával (például a pontosság\@*k*, visszaírási\@*k*, [térkép] [ map], [nDCG\@k][ndcg]).</span><span class="sxs-lookup"><span data-stu-id="26b32-124">Similar to other machine learning tasks, a recommender is validated by using evaluation metrics (for example, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span></span>

4. <span data-ttu-id="26b32-125">Az Azure Machine Learning szolgáltatás a koordinációs a kísérleti, például a hiperparaméter kezdik és a modellkezelési szolgál.</span><span class="sxs-lookup"><span data-stu-id="26b32-125">Azure Machine Learning service is used for coordinating the experimentation, such as hyperparameter sweeping and model management.</span></span>

5. <span data-ttu-id="26b32-126">Betanított modell megőrződik az Azure Cosmos DB, amely ezután alkalmazható felső megközelítéssel *k* filmek az adott felhasználó számára.</span><span class="sxs-lookup"><span data-stu-id="26b32-126">A trained model is preserved on Azure Cosmos DB, which can then be applied for recommending the top *k* movies for a given user.</span></span>

6. <span data-ttu-id="26b32-127">A modell majd Azure Container Instances szolgáltatásban vagy az Azure Kubernetes Service használatával telepít egy webes vagy az app service-be.</span><span class="sxs-lookup"><span data-stu-id="26b32-127">The model is then deployed onto a web or app service by using Azure Container Instances or Azure Kubernetes Service.</span></span>

<span data-ttu-id="26b32-128">Részletes útmutató és ajánló-szolgáltatás méretezése, lásd: [valós idejű ajánlás API létrehozása az Azure-ban][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="26b32-128">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span>

### <a name="components"></a><span data-ttu-id="26b32-129">Összetevők</span><span class="sxs-lookup"><span data-stu-id="26b32-129">Components</span></span>

* <span data-ttu-id="26b32-130">[Adatelemző virtuális gép] [ dsvm] (DSVM) egy Azure virtuális gépen, a deep learning-keretrendszerek és eszközök a machine learning és az adattudomány.</span><span class="sxs-lookup"><span data-stu-id="26b32-130">[Data Science Virtual Machine][dsvm] (DSVM) is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="26b32-131">A DSVM rendelkezik egy önálló a Spark környezet ALS futtatásához használható.</span><span class="sxs-lookup"><span data-stu-id="26b32-131">The DSVM has a standalone Spark environment that can be used to run ALS.</span></span>

* <span data-ttu-id="26b32-132">[Az Azure Blob storage] [ blob] filmjánlásokat adatkészletének tárolja.</span><span class="sxs-lookup"><span data-stu-id="26b32-132">[Azure Blob storage][blob] stores the dataset for movie recommendations.</span></span>

* <span data-ttu-id="26b32-133">[Az Azure Machine Learning szolgáltatás] [ mls] gyorsabb létrehozását, kezelését és a machine learning-modellek üzembe helyezéséhez használt.</span><span class="sxs-lookup"><span data-stu-id="26b32-133">[Azure Machine Learning service][mls] is used to accelerate the building, managing, and deploying of machine learning models.</span></span>

* <span data-ttu-id="26b32-134">[Az Azure Cosmos DB] [ cosmosdb] lehetővé teszi, hogy a globálisan elosztott és többmodelles adatbázis-tárolás.</span><span class="sxs-lookup"><span data-stu-id="26b32-134">[Azure Cosmos DB][cosmosdb] enables globally distributed and multi-model database storage.</span></span>

* <span data-ttu-id="26b32-135">[Az Azure Container Instances] [ aci] szolgál a betanított modellek üzembe helyezése web- vagy app serviceshez, igény szerint segítségével [Azure Kubernetes Service][aks].</span><span class="sxs-lookup"><span data-stu-id="26b32-135">[Azure Container Instances][aci] is used to deploy the trained models to web or app services, optionally using [Azure Kubernetes Service][aks].</span></span>

### <a name="alternatives"></a><span data-ttu-id="26b32-136">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="26b32-136">Alternatives</span></span>

<span data-ttu-id="26b32-137">[Az Azure Databricks] [ databricks] egy felügyelt Spark fürt, ahol a modell betanítása és kiértékelése történik.</span><span class="sxs-lookup"><span data-stu-id="26b32-137">[Azure Databricks][databricks] is a managed Spark cluster where model training and evaluating is performed.</span></span> <span data-ttu-id="26b32-138">Beállíthat egy felügyelt Spark-környezetben, percek és [automatikus skálázási] [ autoscale] felfelé és lefelé, hogy az erőforrások és a fürtök manuális skálázás társított költségek csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="26b32-138">You can set up a managed Spark environment in minutes, and [autoscale][autoscale] up and down to help reduce the resources and costs associated with scaling clusters manually.</span></span> <span data-ttu-id="26b32-139">Egy másik erőforrás-megtakarítási lehetőség, inaktív konfigurálása [fürtök] [ clusters] automatikusan leáll.</span><span class="sxs-lookup"><span data-stu-id="26b32-139">Another resource-saving option is to configure inactive [clusters][clusters] to terminate automatically.</span></span>

## <a name="considerations"></a><span data-ttu-id="26b32-140">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="26b32-140">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="26b32-141">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="26b32-141">Availability</span></span>

<span data-ttu-id="26b32-142">Machine learning-beépített alkalmazások osztották két erőforrás-összetevőt: képzési erőforrásokat és a kiszolgáló forrásokat.</span><span class="sxs-lookup"><span data-stu-id="26b32-142">Machine-learning-built apps are split into two resource components: resources for training, and resources for serving.</span></span> <span data-ttu-id="26b32-143">Képzés általában szükséges erőforrásokat éles kérések közvetlenül éri el ezeket az erőforrásokat, nem kell magas rendelkezésre állás érdekében.</span><span class="sxs-lookup"><span data-stu-id="26b32-143">Resources required for training generally do not need  high availability, as live production requests do not directly hit these resources.</span></span> <span data-ttu-id="26b32-144">Kiszolgáló számára szükséges erőforrások magas rendelkezésre állású ügyfél kérelmek kiszolgálására van szükség.</span><span class="sxs-lookup"><span data-stu-id="26b32-144">Resources required for serving need to have high availability to serve customer requests.</span></span>

<span data-ttu-id="26b32-145">A képzés, a DSVM érhető el a [több régióban] [ regions] szerte a világon, és megfelel-e a [szolgáltatói szerződést] [ sla] (SLA) számára virtuális gépek.</span><span class="sxs-lookup"><span data-stu-id="26b32-145">For training, the DSVM is available in [multiple regions][regions] around the globe and meets the [service level agreement][sla] (SLA) for virtual machines.</span></span> <span data-ttu-id="26b32-146">A kiszolgáló, az Azure Kubernetes Service biztosít egy [magas rendelkezésre állású] [ ha] infrastruktúra.</span><span class="sxs-lookup"><span data-stu-id="26b32-146">For serving, Azure Kubernetes Service provides a [highly available][ha] infrastructure.</span></span> <span data-ttu-id="26b32-147">Ügynökcsomópontok is hajtsa végre a [SLA] [ sla-aks] virtuális gépek számára.</span><span class="sxs-lookup"><span data-stu-id="26b32-147">Agent nodes also follow the [SLA][sla-aks] for virtual machines.</span></span>

### <a name="scalability"></a><span data-ttu-id="26b32-148">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="26b32-148">Scalability</span></span>

<span data-ttu-id="26b32-149">Ha nagy mennyiségű adat méret, méretezhető képzési idő lerövidítheti a dsvm-hez.</span><span class="sxs-lookup"><span data-stu-id="26b32-149">If you have a large data size, you can scale your DSVM to shorten training time.</span></span> <span data-ttu-id="26b32-150">Méretezheti a virtuális gép felfelé vagy lefelé módosításával a [Virtuálisgép-méret][vm-size].</span><span class="sxs-lookup"><span data-stu-id="26b32-150">You can scale a VM up or down by changing the [VM size][vm-size].</span></span> <span data-ttu-id="26b32-151">Válassza ki, a memória méretét elég nagy az adatkészlet a memóriában, és a egy nagyobb vCPU-számot idő csökkentése érdekében, hogy képzési vesz igénybe.</span><span class="sxs-lookup"><span data-stu-id="26b32-151">Choose a memory size large enough to fit your dataset in-memory and a higher vCPU count in order to decrease the amount of time that training takes.</span></span>

### <a name="security"></a><span data-ttu-id="26b32-152">Biztonság</span><span class="sxs-lookup"><span data-stu-id="26b32-152">Security</span></span>

<span data-ttu-id="26b32-153">Ebben a forgatókönyvben az Azure Active Directory használatával hitelesítheti a felhasználókat a [a dsvm-hez való hozzáférés][dsvm-id], amely tartalmazza a kódot, modelleket és (memóriabeli) adatait.</span><span class="sxs-lookup"><span data-stu-id="26b32-153">This scenario can use Azure Active Directory to authenticate users for [access to the DSVM][dsvm-id], which contains your code, models, and (in-memory) data.</span></span> <span data-ttu-id="26b32-154">Adatok az Azure Storage tárolja a dsvm-hez, a betöltése előtt, ha automatikusan titkosítva használatával [a Storage Service Encryption][storage-security].</span><span class="sxs-lookup"><span data-stu-id="26b32-154">Data is stored in Azure Storage prior to being loaded on a DSVM, where it is automatically encrypted using [Storage Service Encryption][storage-security].</span></span> <span data-ttu-id="26b32-155">Engedélyek az Azure Active Directory-hitelesítés vagy a szerepköralapú hozzáférés-vezérlés használatával kezelhetők.</span><span class="sxs-lookup"><span data-stu-id="26b32-155">Permissions can be managed via Azure Active Directory authentication or role-based access control.</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="26b32-156">Ez a forgatókönyv megvalósítható</span><span class="sxs-lookup"><span data-stu-id="26b32-156">Deploy this scenario</span></span>

<span data-ttu-id="26b32-157">**Előfeltételek**: Meglévő Azure-fiókkal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="26b32-157">**Prerequisites**: You must have an existing Azure account.</span></span> <span data-ttu-id="26b32-158">Ha nem rendelkezik Azure-előfizetéssel, első lépésként létrehozhat egy [ingyenes][free] fiókot.</span><span class="sxs-lookup"><span data-stu-id="26b32-158">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="26b32-159">Ebben a forgatókönyvben a kód érhető el a [Microsoft Recommenders tárház][github].</span><span class="sxs-lookup"><span data-stu-id="26b32-159">All the code for this scenario is available in the [Microsoft Recommenders repository][github].</span></span>

<span data-ttu-id="26b32-160">Futtassa a következő lépésekkel a [ALS rövid notebook][notebook]:</span><span class="sxs-lookup"><span data-stu-id="26b32-160">Follow these steps to run the [ALS quickstart notebook][notebook]:</span></span>

1. <span data-ttu-id="26b32-161">[Hozzon létre egy DSVM] [ dsvm-ubuntu] az Azure Portalról.</span><span class="sxs-lookup"><span data-stu-id="26b32-161">[Create a DSVM][dsvm-ubuntu] from the Azure portal.</span></span>

2. <span data-ttu-id="26b32-162">Klónozza az adattárat a notebookok mappában:</span><span class="sxs-lookup"><span data-stu-id="26b32-162">Clone the repo in the Notebooks folder:</span></span>

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. <span data-ttu-id="26b32-163">Telepítse a következő témakörben ismertetett conda-függőségeket a [SETUP.md] [ setup] fájlt.</span><span class="sxs-lookup"><span data-stu-id="26b32-163">Install the conda dependencies following the steps described in the [SETUP.md][setup] file.</span></span>

4. <span data-ttu-id="26b32-164">Egy böngészőben nyissa meg a virtuális gép jupyterlab, és navigáljon a `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span><span class="sxs-lookup"><span data-stu-id="26b32-164">In a browser, go to your jupyterlab VM and navigate to `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span></span>

5. <span data-ttu-id="26b32-165">Hajtsa végre a jegyzetfüzetet.</span><span class="sxs-lookup"><span data-stu-id="26b32-165">Execute the notebook.</span></span>

## <a name="related-resources"></a><span data-ttu-id="26b32-166">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="26b32-166">Related resources</span></span>

<span data-ttu-id="26b32-167">Részletes útmutató és ajánló-szolgáltatás méretezése, lásd: [valós idejű ajánlás API létrehozása az Azure-ban][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="26b32-167">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span> <span data-ttu-id="26b32-168">Oktatóanyagok és példák a javaslattételi rendszerek: [Microsoft Recommenders tárház][github].</span><span class="sxs-lookup"><span data-stu-id="26b32-168">For tutorials and examples of recommendation systems, see [Microsoft Recommenders repository][github].</span></span>

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-id]: /azure/machine-learning/data-science-virtual-machine/dsvm-common-identity
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
