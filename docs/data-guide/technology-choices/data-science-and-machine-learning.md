---
title: A gépi tanulási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: e4b8ecb64afbacc8a4e24e9c6274455db0451d62
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112125"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="80c4f-102">Kiválasztása a machine learning-technológiák az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="80c4f-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="80c4f-103">Adattudományi és machine learning, illetve egy folyamat, amely általában végzi az adatszakértők.</span><span class="sxs-lookup"><span data-stu-id="80c4f-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="80c4f-104">Szakértő eszközök, számos, amely kifejezetten interaktív adatfeltárás és modellezés adatszakértő kell végrehajtó tevékenységek típusú készültek van szükség.</span><span class="sxs-lookup"><span data-stu-id="80c4f-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="80c4f-105">Gépi tanulási megoldások iteratív épülnek, és két fázisban van:</span><span class="sxs-lookup"><span data-stu-id="80c4f-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>

- <span data-ttu-id="80c4f-106">Adat-előkészítési és a modellezés.</span><span class="sxs-lookup"><span data-stu-id="80c4f-106">Data preparation and modeling.</span></span>
- <span data-ttu-id="80c4f-107">Üzembe helyezési és prediktív-szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="80c4f-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="80c4f-108">Eszközöket és szolgáltatásokat az adat-előkészítési és a modellezés</span><span class="sxs-lookup"><span data-stu-id="80c4f-108">Tools and services for data preparation and modeling</span></span>

<span data-ttu-id="80c4f-109">Az adatszakértők általában inkább olyan adatok, Python vagy r nyelven írt egyéni kód használatával Ez a kód generálása a képi megjelenítés és statisztikai annak meghatározásához, a kapcsolatokat, és a lekérdezésre és vizsgálódásra az adatokat, az adatszakértők, általában futását.</span><span class="sxs-lookup"><span data-stu-id="80c4f-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="80c4f-110">Nincsenek számos R és Python, amellyel az adatelemzők interaktív környezeteket.</span><span class="sxs-lookup"><span data-stu-id="80c4f-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="80c4f-111">Egy adott kedvenc van **Jupyter notebookok** , amely egy böngészőalapú rendszerhéj, amely lehetővé teszi az adatszakértők létrehozásához biztosít *notebook* R vagy Python kódját és markdown-szöveget tartalmazó fájlok.</span><span class="sxs-lookup"><span data-stu-id="80c4f-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="80c4f-112">Ez a megosztási és dokumentálja a kód működhet hatékony módja, és egyetlen dokumentum eredményez.</span><span class="sxs-lookup"><span data-stu-id="80c4f-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="80c4f-113">Más gyakran használt eszközök a következők:</span><span class="sxs-lookup"><span data-stu-id="80c4f-113">Other commonly used tools include:</span></span>

- <span data-ttu-id="80c4f-114">**Spyder**: Interaktív fejlesztői környezet (IDE) a Anaconda Python elosztási biztosított Python-hez.</span><span class="sxs-lookup"><span data-stu-id="80c4f-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
- <span data-ttu-id="80c4f-115">**Az R Studio**: Egy ide-vel az R programozási nyelv.</span><span class="sxs-lookup"><span data-stu-id="80c4f-115">**R Studio**: An IDE for the R programming language.</span></span>
- <span data-ttu-id="80c4f-116">**A Visual Studio Code**: Egy egyszerűsített, platformfüggetlen programozási környezetet, amely támogatja a Python, valamint az elterjedt keretrendszerek gépi tanulási és AI-fejlesztést.</span><span class="sxs-lookup"><span data-stu-id="80c4f-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="80c4f-117">Ezek az eszközök mellett adatszakértők használhatják a Azure-szolgáltatások kódot és a modell kezelésének egyszerűsítéséhez.</span><span class="sxs-lookup"><span data-stu-id="80c4f-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="80c4f-118">Azure Notebooks</span><span class="sxs-lookup"><span data-stu-id="80c4f-118">Azure Notebooks</span></span>

<span data-ttu-id="80c4f-119">Az Azure notebookok online Jupyter notebookok szolgáltatás, amely lehetővé teszi az adatszakértők, létrehozása, futtatása és megosztása a felhő alapú tárak Jupyter-Notebookjait.</span><span class="sxs-lookup"><span data-stu-id="80c4f-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="80c4f-120">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-120">Key benefits:</span></span>

- <span data-ttu-id="80c4f-121">Az ingyenes szolgáltatás&mdash;nem Azure-előfizetés szükséges.</span><span class="sxs-lookup"><span data-stu-id="80c4f-121">Free service&mdash;no Azure subscription required.</span></span>
- <span data-ttu-id="80c4f-122">Nem kell telepítenie a Jupyter és a támogató R vagy Python disztribúciók helyileg&mdash;használja egy böngészőben.</span><span class="sxs-lookup"><span data-stu-id="80c4f-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
- <span data-ttu-id="80c4f-123">A saját online tárak, és azok bármely eszközről elérhetők.</span><span class="sxs-lookup"><span data-stu-id="80c4f-123">Manage your own online libraries and access them from any device.</span></span>
- <span data-ttu-id="80c4f-124">Ossza meg a notebookok közreműködőkkel együtt.</span><span class="sxs-lookup"><span data-stu-id="80c4f-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="80c4f-125">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-125">Considerations:</span></span>

- <span data-ttu-id="80c4f-126">Ön nem fér hozzá a notebookok offline állapotban lesz.</span><span class="sxs-lookup"><span data-stu-id="80c4f-126">You will be unable to access your notebooks when offline.</span></span>
- <span data-ttu-id="80c4f-127">Az ingyenes notebook szolgáltatás korlátozott feldolgozási képességek nem lehet elég nagy vagy összetett modelleket taníthat be.</span><span class="sxs-lookup"><span data-stu-id="80c4f-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="80c4f-128">Adatelemző virtuális gép</span><span class="sxs-lookup"><span data-stu-id="80c4f-128">Data science virtual machine</span></span>

<span data-ttu-id="80c4f-129">Az adatelemző virtuális gép az Azure-beli virtuálisgép-lemezkép, amely tartalmazza az eszközöket és keretrendszereket gyakran használják az adatszakértők, többek között az R, Python, a Jupyter Notebooks, a Visual Studio Code és gépi tanulási modellezését, például a könyvtárak a A Microsoft Cognitive Toolkit.</span><span class="sxs-lookup"><span data-stu-id="80c4f-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="80c4f-130">Ezek az eszközök lehetnek összetett és időigényes, és számos fennálló függőségeket, amelyek gyakran verziójú felügyeleti problémákat okozhat tartalmaznak.</span><span class="sxs-lookup"><span data-stu-id="80c4f-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="80c4f-131">Egy előre telepített lemezképet kellene csökkentheti a idejét az adatszakértők hibaelhárítási környezetben előforduló problémák megoldására, lehetővé téve számukra az összpontosíthat az adatfeltárás és modellezés feladatok elvégzéséhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="80c4f-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="80c4f-132">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-132">Key benefits:</span></span>

- <span data-ttu-id="80c4f-133">Kevesebb idő telepítése, kezelése és beépített adatelemzési eszközzel, és a keretrendszereket.</span><span class="sxs-lookup"><span data-stu-id="80c4f-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
- <span data-ttu-id="80c4f-134">A legújabb, az összes gyakran használt eszközök és keretrendszerek részét képezik.</span><span class="sxs-lookup"><span data-stu-id="80c4f-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
- <span data-ttu-id="80c4f-135">Virtuális gép beállításai az intenzív adatmodellezés GPU-funkciókkal rendelkező nagy mértékben skálázható képeket is tartalmaznak.</span><span class="sxs-lookup"><span data-stu-id="80c4f-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="80c4f-136">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-136">Considerations:</span></span>

- <span data-ttu-id="80c4f-137">A virtuális gép nem érhető el, ha offline állapotban van.</span><span class="sxs-lookup"><span data-stu-id="80c4f-137">The virtual machine cannot be accessed when offline.</span></span>
- <span data-ttu-id="80c4f-138">Azure-szolgáltatások díjait, virtuális gépek tekintetében, így kell lennie arra, hogy csak szükség esetén futnak.</span><span class="sxs-lookup"><span data-stu-id="80c4f-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="80c4f-139">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="80c4f-139">Azure Machine Learning</span></span>

<span data-ttu-id="80c4f-140">Az Azure Machine Learning egy olyan felhőalapú szolgáltatás, machine learning-kísérletek és a modellek felügyeletére.</span><span class="sxs-lookup"><span data-stu-id="80c4f-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="80c4f-141">Ez magában foglalja egy Kísérletezési szolgáltatás, amely nyomon követi az adat-előkészítési és modellezés a betanítási szkriptekhez, az összes végrehajtás előzményeit fenntartása, így összehasonlíthatja a modellek teljesítményének ismétléseinek között.</span><span class="sxs-lookup"><span data-stu-id="80c4f-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="80c4f-142">Az adatszakértők a kívánt eszközben dolgozhat, például a Jupyter notebookok vagy a Visual Studio Code-ban alatt létrehozhatják a parancsfájlok és majd telepítheti a számos különböző [számítási erőforrások](/azure/machine-learning/service/how-to-set-up-training-targets) az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="80c4f-142">Data scientists can create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code, and then deploy to a variety of different [compute resources](/azure/machine-learning/service/how-to-set-up-training-targets) in Azure.</span></span>

<span data-ttu-id="80c4f-143">Modelleket webszolgáltatásként, amely egy Docker-tárolóba, a Spark on Azure HDinsight, a Microsoft Machine Learning-kiszolgáló vagy az SQL Server is telepíthető.</span><span class="sxs-lookup"><span data-stu-id="80c4f-143">Models can be deployed as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="80c4f-144">Az Azure Machine Learning Modellkezelés szolgáltatás majd lehetővé teszi, hogy nyomon követheti és kezelheti a modell-üzembehelyezések a felhőben, a peremhálózati eszközökön, vagy a vállalaton belül.</span><span class="sxs-lookup"><span data-stu-id="80c4f-144">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="80c4f-145">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-145">Key benefits:</span></span>

- <span data-ttu-id="80c4f-146">Központi felügyeleti parancsfájlokat és a futtatási előzményei, így könnyen modell verziójának összehasonlítása.</span><span class="sxs-lookup"><span data-stu-id="80c4f-146">Central management of scripts and run history, making it easy to compare model versions.</span></span>
- <span data-ttu-id="80c4f-147">Interaktív adatok átalakítása egy vizuális szerkesztő.</span><span class="sxs-lookup"><span data-stu-id="80c4f-147">Interactive data transformation through a visual editor.</span></span>
- <span data-ttu-id="80c4f-148">Könnyű üzembe helyezés és kezelés modellek a felhőben és a peremhálózati eszközökre.</span><span class="sxs-lookup"><span data-stu-id="80c4f-148">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="80c4f-149">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-149">Considerations:</span></span>

- <span data-ttu-id="80c4f-150">A modell felügyeleti modell bizonyos fokú ismeretét igényli.</span><span class="sxs-lookup"><span data-stu-id="80c4f-150">Requires some familiarity with the model management model.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="80c4f-151">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="80c4f-151">Azure Batch AI</span></span>

<span data-ttu-id="80c4f-152">Az Azure Batch AI lehetővé teszi a machine learning-kísérletek párhuzamos futtatása, és a virtuális gépek a GPU-fürtön belül hajtsa végre a modell betanítást szeretne nagy méretben.</span><span class="sxs-lookup"><span data-stu-id="80c4f-152">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="80c4f-153">A Batch AI training lehetővé teszi a horizontális felskálázás mély tanulási feladatok csoportosított gpu-kon keresztül, keretrendszerekkel, mint például a Cognitive Toolkit, Caffe, Chainer és tensorflow-hoz.</span><span class="sxs-lookup"><span data-stu-id="80c4f-153">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span>

<span data-ttu-id="80c4f-154">Az Azure Machine Learning Modellkezelés használható modellek átvételére a Batch AI képzési üzembe helyezéséhez és kezeléséhez, és megfigyelheti őket.</span><span class="sxs-lookup"><span data-stu-id="80c4f-154">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span>

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="80c4f-155">Azure Machine Learning Studio</span><span class="sxs-lookup"><span data-stu-id="80c4f-155">Azure Machine Learning Studio</span></span>

<span data-ttu-id="80c4f-156">Az Azure Machine Learning Studióban egy felhőalapú, visual fejlesztési környezetet, a data-kísérletek létrehozása machine learning-modellek betanítása és ezek közzététele webszolgáltatásként az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="80c4f-156">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="80c4f-157">A vizuális fogd és vidd felület lehetővé teszi, hogy az adatszakértők és a kiemelt felhasználók gépi tanulási megoldások gyors létrehozásához, miközben támogatja az egyéni R és Python logika, a létrehozott statisztikai algoritmusok és technikák számos machine Learning-tevékenységek modellezése , és a Jupyter notebookok beépített támogatását.</span><span class="sxs-lookup"><span data-stu-id="80c4f-157">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="80c4f-158">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-158">Key benefits:</span></span>

- <span data-ttu-id="80c4f-159">Interaktív vizuális felületen lehetővé teszi, hogy a minimális programozással való modellezés, gépi tanulás.</span><span class="sxs-lookup"><span data-stu-id="80c4f-159">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
- <span data-ttu-id="80c4f-160">Beépített Jupyter notebookok adatok feltárásához.</span><span class="sxs-lookup"><span data-stu-id="80c4f-160">Built-in Jupyter Notebooks for data exploration.</span></span>
- <span data-ttu-id="80c4f-161">Közvetlen üzembe helyezés a betanított modellek Azure webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="80c4f-161">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="80c4f-162">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-162">Considerations:</span></span>

- <span data-ttu-id="80c4f-163">Korlátozott a méretezhetősége.</span><span class="sxs-lookup"><span data-stu-id="80c4f-163">Limited scalability.</span></span> <span data-ttu-id="80c4f-164">Betanítási adatkészletet maximális mérete 10 GB-ot.</span><span class="sxs-lookup"><span data-stu-id="80c4f-164">The maximum size of a training dataset is 10 GB.</span></span>
- <span data-ttu-id="80c4f-165">Csak online.</span><span class="sxs-lookup"><span data-stu-id="80c4f-165">Online only.</span></span> <span data-ttu-id="80c4f-166">Nincs kapcsolat nélküli fejlesztési környezet.</span><span class="sxs-lookup"><span data-stu-id="80c4f-166">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="80c4f-167">Eszközök és szolgáltatások üzembe helyezéséhez a machine learning-modellek</span><span class="sxs-lookup"><span data-stu-id="80c4f-167">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="80c4f-168">Adatszakértő rendelkezik egy machine learning-modellek létrehozása után általában kell üzembe helyezni és felhasználását, alkalmazások vagy más adatok folyamatokban.</span><span class="sxs-lookup"><span data-stu-id="80c4f-168">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="80c4f-169">Számos lehetséges telepítési céljainak gépi tanulási modelleket.</span><span class="sxs-lookup"><span data-stu-id="80c4f-169">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="80c4f-170">Az Azure HDInsighthoz készült Spark</span><span class="sxs-lookup"><span data-stu-id="80c4f-170">Spark on Azure HDInsight</span></span>

<span data-ttu-id="80c4f-171">Az Apache Spark tartalmazza a Spark MLlib, keretrendszer és gépi tanulási modelleket könyvtárában.</span><span class="sxs-lookup"><span data-stu-id="80c4f-171">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="80c4f-172">A Microsoft Machine Learning library for Spark (MMLSpark) is biztosít a deep learning-prediktív modelleket a Spark algoritmus támogatása.</span><span class="sxs-lookup"><span data-stu-id="80c4f-172">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="80c4f-173">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-173">Key benefits:</span></span>

- <span data-ttu-id="80c4f-174">A Spark, elosztott platformot, amely nagy mennyiségű machine learning-folyamatokat a nagy mértékű skálázhatóságot biztosít.</span><span class="sxs-lookup"><span data-stu-id="80c4f-174">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
- <span data-ttu-id="80c4f-175">Modellek üzembe helyezése közvetlenül a Spark on HDinsight, és kezelheti azokat az Azure Machine Learning Modellkezelés szolgáltatás használatával.</span><span class="sxs-lookup"><span data-stu-id="80c4f-175">You can deploy models directly to Spark in HDinsight and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="80c4f-176">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-176">Considerations:</span></span>

- <span data-ttu-id="80c4f-177">A Spark Hdinsight-fürtök, hogy fut-e a teljes idő költségeket terhel futtatja.</span><span class="sxs-lookup"><span data-stu-id="80c4f-177">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="80c4f-178">A machine learning-szolgáltatás csak alkalmanként használható, ha emiatt előfordulhat, hogy a felesleges költségek.</span><span class="sxs-lookup"><span data-stu-id="80c4f-178">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="azure-databricks"></a><span data-ttu-id="80c4f-179">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="80c4f-179">Azure Databricks</span></span>

<span data-ttu-id="80c4f-180">[Az Azure Databricks](/azure/azure-databricks/) egy Apache Spark-alapú elemzési platform.</span><span class="sxs-lookup"><span data-stu-id="80c4f-180">[Azure Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform.</span></span> <span data-ttu-id="80c4f-181">Felfoghatók úgy, mint "Spark szolgáltatás."</span><span class="sxs-lookup"><span data-stu-id="80c4f-181">You can think of it as "Spark as a service."</span></span> <span data-ttu-id="80c4f-182">A Spark használata az Azure platformon legegyszerűbb módja.</span><span class="sxs-lookup"><span data-stu-id="80c4f-182">It's the easiest way to use Spark on the Azure platform.</span></span> <span data-ttu-id="80c4f-183">Machine Learning szolgáltatás használható [MLFlow](https://www.mlflow.org/), [Databricks futtatókörnyezet ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib és mások.</span><span class="sxs-lookup"><span data-stu-id="80c4f-183">For machine learning, you can use [MLFlow](https://www.mlflow.org/), [Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib, and others.</span></span> <span data-ttu-id="80c4f-184">További információkért lásd: [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span><span class="sxs-lookup"><span data-stu-id="80c4f-184">For more information, see [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="80c4f-185">Webszolgáltatás-tárolóban</span><span class="sxs-lookup"><span data-stu-id="80c4f-185">Web service in a container</span></span>

<span data-ttu-id="80c4f-186">Telepíthet egy gépi tanulási modellt a Docker-tárolóban Python webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="80c4f-186">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="80c4f-187">Is üzembe a modellt, az Azure-bA vagy egy edge-eszközön, ahol használat helyben az adatokat, amelyen működik.</span><span class="sxs-lookup"><span data-stu-id="80c4f-187">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="80c4f-188">Fő értékek:</span><span class="sxs-lookup"><span data-stu-id="80c4f-188">Key Benefits:</span></span>

- <span data-ttu-id="80c4f-189">Tárolók csomagolása és üzembe helyezése a szolgáltatások egy könnyen használható és általánosan költséghatékony módon.</span><span class="sxs-lookup"><span data-stu-id="80c4f-189">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
- <span data-ttu-id="80c4f-190">Edge-eszköz üzembe helyezése lehetővé teszi lehetővé teszi, hogy közelebb a prediktív logikai áthelyezheti az adatokat.</span><span class="sxs-lookup"><span data-stu-id="80c4f-190">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>

<span data-ttu-id="80c4f-191">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-191">Considerations:</span></span>

- <span data-ttu-id="80c4f-192">Ez a telepítési modell alapuló Docker-tárolókat, ismernie kell a technológia az ezzel a módszerrel a web Service szolgáltatásának telepítése előtt.</span><span class="sxs-lookup"><span data-stu-id="80c4f-192">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="80c4f-193">Microsoft Machine Learning-kiszolgáló</span><span class="sxs-lookup"><span data-stu-id="80c4f-193">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="80c4f-194">Machine Learning-kiszolgáló (korábbi nevén Microsoft R Server) az R és Python kódok, kifejezetten a machine learning-forgatókönyvekhez készült méretezhető platform.</span><span class="sxs-lookup"><span data-stu-id="80c4f-194">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="80c4f-195">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-195">Key benefits:</span></span>

- <span data-ttu-id="80c4f-196">Magas skálázhatóság.</span><span class="sxs-lookup"><span data-stu-id="80c4f-196">High scalability.</span></span>

<span data-ttu-id="80c4f-197">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-197">Considerations:</span></span>

- <span data-ttu-id="80c4f-198">Üzembe helyezése és kezelése a Machine Learning-kiszolgáló a vállalat kell.</span><span class="sxs-lookup"><span data-stu-id="80c4f-198">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="80c4f-199">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="80c4f-199">Microsoft SQL Server</span></span>

<span data-ttu-id="80c4f-200">A Microsoft SQL Server támogatja az R és Python natív módon, az ezeken a nyelveken Transact-SQL függvényeket egy adatbázisban, amely lehetővé teszi, hogy magába foglalja a machine learning-modellek beépített.</span><span class="sxs-lookup"><span data-stu-id="80c4f-200">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="80c4f-201">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-201">Key benefits:</span></span>

- <span data-ttu-id="80c4f-202">Egy adatbázis függvényben, így könnyen felvenni az adatrétegbeli logikai prediktív logikai magába foglalja.</span><span class="sxs-lookup"><span data-stu-id="80c4f-202">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="80c4f-203">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-203">Considerations:</span></span>

- <span data-ttu-id="80c4f-204">Feltételezi, hogy egy SQL Server-adatbázis, az adatréteg az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="80c4f-204">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="80c4f-205">Az Azure Machine Learning-webszolgáltatás</span><span class="sxs-lookup"><span data-stu-id="80c4f-205">Azure Machine Learning web service</span></span>

<span data-ttu-id="80c4f-206">Amikor egy gépi tanulási modellt az Azure Machine Learning Studio használatával hoz létre, telepíthet webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="80c4f-206">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="80c4f-207">Ez majd használhatók fel a HTTP-alapú kommunikáció kompatibilis ügyfélalkalmazások REST-interfészen keresztül.</span><span class="sxs-lookup"><span data-stu-id="80c4f-207">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="80c4f-208">Fő előnyök:</span><span class="sxs-lookup"><span data-stu-id="80c4f-208">Key benefits:</span></span>

- <span data-ttu-id="80c4f-209">Egyszerű fejlesztéséhez és üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="80c4f-209">Ease of development and deployment.</span></span>
- <span data-ttu-id="80c4f-210">Webes szolgáltatás felügyeleti portáljára alapszintű figyelési metrikákat.</span><span class="sxs-lookup"><span data-stu-id="80c4f-210">Web service management portal with basic monitoring metrics.</span></span>
- <span data-ttu-id="80c4f-211">Az Azure Machine Learning webszolgáltatások hívása az Azure Data Lake Analytics, Azure Data Factory és az Azure Stream Analytics beépített támogatása.</span><span class="sxs-lookup"><span data-stu-id="80c4f-211">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="80c4f-212">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="80c4f-212">Considerations:</span></span>

- <span data-ttu-id="80c4f-213">Csak a modellek Azure Machine Learning Studio használatával érhető el.</span><span class="sxs-lookup"><span data-stu-id="80c4f-213">Only available for models built using Azure Machine Learning Studio.</span></span>
- <span data-ttu-id="80c4f-214">Webes hozzáférés csak, betanított modellek nem futtatható helyi vagy offline állapotú.</span><span class="sxs-lookup"><span data-stu-id="80c4f-214">Web-based access only, trained models cannot run on-premises or offline.</span></span>
