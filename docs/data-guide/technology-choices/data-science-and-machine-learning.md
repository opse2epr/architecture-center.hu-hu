---
title: A gépi tanulási technológiával kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
ms.locfileid: "29291963"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="e059e-102">A gépi tanulási technológiával az Azure-ban kiválasztása</span><span class="sxs-lookup"><span data-stu-id="e059e-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="e059e-103">Adattudomány és a gépi tanulás egy munkaterhelés –, amelyek adatszakértőkön általában végzi.</span><span class="sxs-lookup"><span data-stu-id="e059e-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="e059e-104">Szükségel specialistája eszközök sok kifejezetten az adatok interaktív áttekintését és feladatokat, végezze el a adatok tudósok modellezési típusú készültek.</span><span class="sxs-lookup"><span data-stu-id="e059e-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="e059e-105">Machine learning megoldások ismételt épülnek, és két különböző fázisokból áll:</span><span class="sxs-lookup"><span data-stu-id="e059e-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>
* <span data-ttu-id="e059e-106">Adatok előkészítése és a modellezési.</span><span class="sxs-lookup"><span data-stu-id="e059e-106">Data preparation and modeling.</span></span>
* <span data-ttu-id="e059e-107">Központi telepítés és használat prediktív szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="e059e-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="e059e-108">Eszközök és adatok előkészítése és a modellezési szolgáltatás</span><span class="sxs-lookup"><span data-stu-id="e059e-108">Tools and services for data preparation and modeling</span></span>
<span data-ttu-id="e059e-109">Adatszakértőkön általában inkább a Python vagy r nyelven írt egyéni kód használatával adatok Ez a kód képi megjelenítések és annak meghatározásához, a kapcsolatokkal statisztikák létrehozása segítségével lekérdezéséhez és az adatokba, adatszakértőkön interaktív módon, általában futtatásakor.</span><span class="sxs-lookup"><span data-stu-id="e059e-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="e059e-110">Nincsenek R és Python adatszakértőkön használó sok interaktív környezetben.</span><span class="sxs-lookup"><span data-stu-id="e059e-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="e059e-111">Egy adott kedvenc van **Jupyter notebookok** egy böngészőalapú felület, amely lehetővé teszi az adatszakértőkön át a létrehozása, amely biztosít *notebook* R vagy Python kódját és markdown szöveget tartalmazó fájlokat.</span><span class="sxs-lookup"><span data-stu-id="e059e-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="e059e-112">Ez megosztása és dokumentálásához kód együttműködni hatékony módot, és egyetlen dokumentum eredményez.</span><span class="sxs-lookup"><span data-stu-id="e059e-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="e059e-113">A gyakran használt eszközöket a következők:</span><span class="sxs-lookup"><span data-stu-id="e059e-113">Other commonly used tools include:</span></span>
* <span data-ttu-id="e059e-114">**Spyder**: A Python, az Anaconda Python elosztási biztosított interaktív fejlesztőkörnyezet (IDE).</span><span class="sxs-lookup"><span data-stu-id="e059e-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
* <span data-ttu-id="e059e-115">**R Studio**: egy IDE az r programozási nyelv.</span><span class="sxs-lookup"><span data-stu-id="e059e-115">**R Studio**: An IDE for the R programming language.</span></span>
* <span data-ttu-id="e059e-116">**A Visual Studio Code**: egy egyszerűsített, platformfüggetlen kódolási környezetben, amely támogatja a Python, valamint a gyakran használt keretrendszerek gépi tanulási és AI fejlesztési.</span><span class="sxs-lookup"><span data-stu-id="e059e-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="e059e-117">Ezek az eszközök mellett adatszakértőkön használhatják az Azure-szolgáltatások egyszerűbbé teheti a kód és a modell kezelését.</span><span class="sxs-lookup"><span data-stu-id="e059e-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="e059e-118">Azure Notebooks</span><span class="sxs-lookup"><span data-stu-id="e059e-118">Azure Notebooks</span></span>
<span data-ttu-id="e059e-119">Azure notebookok egy online Jupyter notebookok szolgáltatás, amely lehetővé teszi az adatszakértőkön át a létrehozása, futtatása és a felhő alapú tárak Jupyter notebookok megosztásához.</span><span class="sxs-lookup"><span data-stu-id="e059e-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="e059e-120">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-120">Key benefits:</span></span>

* <span data-ttu-id="e059e-121">Ingyenes service&mdash;nincs Azure-előfizetés szükséges.</span><span class="sxs-lookup"><span data-stu-id="e059e-121">Free service&mdash;no Azure subscription required.</span></span>
* <span data-ttu-id="e059e-122">Nem szükséges telepíteni a Jupyter és a támogató R vagy Python felosztás helyileg&mdash;használja a böngészőben.</span><span class="sxs-lookup"><span data-stu-id="e059e-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
* <span data-ttu-id="e059e-123">A saját online tárak, és azokat bármilyen eszközön keresztüli elérését.</span><span class="sxs-lookup"><span data-stu-id="e059e-123">Manage your own online libraries and access them from any device.</span></span>
* <span data-ttu-id="e059e-124">A notebookok megosztása közreműködők.</span><span class="sxs-lookup"><span data-stu-id="e059e-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="e059e-125">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-125">Considerations:</span></span>

* <span data-ttu-id="e059e-126">Nem érhető el kapcsolat nélküli módban jegyzetfüzetek fogjuk.</span><span class="sxs-lookup"><span data-stu-id="e059e-126">You will be unable to access your notebooks when offline.</span></span>
* <span data-ttu-id="e059e-127">A szabad notebook szolgáltatás korlátozott képességek nem lehet elég nagy vagy összetett modell betanításához.</span><span class="sxs-lookup"><span data-stu-id="e059e-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="e059e-128">Adatok tudományos virtuális gép</span><span class="sxs-lookup"><span data-stu-id="e059e-128">Data science virtual machine</span></span>
<span data-ttu-id="e059e-129">Az adatok tudományos virtuális gép egy Azure virtuálisgép-lemezkép, amely tartalmazza az eszközök és keretrendszerek adatszakértőkön, beleértve az R, Python, Jupyter notebookok, Visual Studio Code és a gépi tanulás modellezési például a könyvtárak gyakran használják a Microsoft kognitív eszközkészlet.</span><span class="sxs-lookup"><span data-stu-id="e059e-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="e059e-130">Ezek az eszközök összetett és időigényes, és sok egymástól függő szolgáltatásainak, így gyakran verziójú felügyeleti problémák tartalmaznak a lehet.</span><span class="sxs-lookup"><span data-stu-id="e059e-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="e059e-131">Előre telepített lemezkép rendelkező csökkentheti a adatszakértőkön kiosztására szánt problémamegoldást környezet lehetővé teheti, hogy az adatok feltárása összpontosítani idejét, és modellezési feladatok elvégzéséhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="e059e-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="e059e-132">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-132">Key benefits:</span></span>
* <span data-ttu-id="e059e-133">Kevesebb idő telepítése, kezelése és hibaelhárítását adatok tudományos eszközök és -keretrendszerek számára.</span><span class="sxs-lookup"><span data-stu-id="e059e-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
* <span data-ttu-id="e059e-134">A legújabb verzióját az összes használt eszközök és érhetők el keretrendszerek.</span><span class="sxs-lookup"><span data-stu-id="e059e-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
* <span data-ttu-id="e059e-135">Magas szinten méretezhető képek a GPU intenzív adatok modellezési kezelésére képes a virtuális gép lehetőségek között.</span><span class="sxs-lookup"><span data-stu-id="e059e-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="e059e-136">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-136">Considerations:</span></span>
* <span data-ttu-id="e059e-137">A virtuális gép nem érhető el, ha a kapcsolat nélküli.</span><span class="sxs-lookup"><span data-stu-id="e059e-137">The virtual machine cannot be accessed when offline.</span></span>
* <span data-ttu-id="e059e-138">Egy virtuális gépet futtat terhel Azure, ezért meg kell ügyeljen arra, hogy csak szükség esetén futnak.</span><span class="sxs-lookup"><span data-stu-id="e059e-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="e059e-139">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="e059e-139">Azure Machine Learning</span></span>

<span data-ttu-id="e059e-140">Az Azure Machine Learning egy olyan felhőalapú szolgáltatás, gépi tanulási kísérletekhez és modellek kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="e059e-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="e059e-141">Ez magában foglalja egy kísérleti szolgáltatás, amely nyomon követi az adatok előkészítése és a modellezési képzési parancsfájlok fenntartása minden végrehajtások előzményeit modell teljesítmény összehasonlítás ismétlési között.</span><span class="sxs-lookup"><span data-stu-id="e059e-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="e059e-142">Egy Azure Machine Learning-munkaterület nevű platformfüggetlen ügyfél eszköz központi kezelőfelületet biztosít parancsfájl-kezelési és előzményeit, továbbra is engedélyezze az adatszakértőkön át a választott, például a Jupyter notebookok vagy a Visual Studio Code eszköz parancsfájlokat hozhatnak létre.</span><span class="sxs-lookup"><span data-stu-id="e059e-142">A cross-platform client tool named Azure Machine Learning Workbench provides a central interface for script management and history, while still enabling data scientists to create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code.</span></span>

<span data-ttu-id="e059e-143">Az Azure Machine Learning-munkaterület a interaktív adatok előkészítése eszközök segítségével egyszerűsítheti a közös adatok átalakítása feladatok, és beállíthatja, hogy a parancsfájl végrehajtási környezetet helyileg, a modell betanítási parancsfájlok futtatásához méretezhető Docker-tároló vagy a Spark.</span><span class="sxs-lookup"><span data-stu-id="e059e-143">From Azure Machine Learning Workbench, you can use the interactive data preparation tools to simplify common data transformation tasks, and you can configure the script execution environment to run model training scripts locally, in a scalable Docker container, or in Spark.</span></span>

<span data-ttu-id="e059e-144">Ha készen áll a modell rendszerbe állítása, a munkaterület környezet segítségével a modell csomagot, és telepítse azt egy Docker-tároló, a Spark on Azure HDinsight, Microsoft Machine Learning Server vagy SQL Server webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="e059e-144">When you are ready to deploy your model, use the Workbench environment to package the model and deploy it as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="e059e-145">Az Azure Machine Learning modell Management szolgáltatás majd lehetővé teszi, hogy nyomon követhető, és a felhőben, a peremhálózati eszköz, vagy a vállalaton belül a modell központi telepítések felügyeletéhez szükséges.</span><span class="sxs-lookup"><span data-stu-id="e059e-145">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="e059e-146">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-146">Key benefits:</span></span>

* <span data-ttu-id="e059e-147">Központi felügyeleti parancsfájlok és futtatási előzményei, így könnyen modellverziók összehasonlítani.</span><span class="sxs-lookup"><span data-stu-id="e059e-147">Central management of scripts and run history, making it easy to compare model versions.</span></span>
* <span data-ttu-id="e059e-148">Interaktív adatok átalakítása visual szerkesztő keresztül.</span><span class="sxs-lookup"><span data-stu-id="e059e-148">Interactive data transformation through a visual editor.</span></span>
* <span data-ttu-id="e059e-149">Egyszerű telepítés és a felhő vagy a peremhálózati eszköz számára modellek kezelése.</span><span class="sxs-lookup"><span data-stu-id="e059e-149">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="e059e-150">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-150">Considerations:</span></span>
* <span data-ttu-id="e059e-151">A modell felügyeleti modellt és a munkaterületet üzemeltető eszköz környezetet bizonyos fokú ismeretét igényli.</span><span class="sxs-lookup"><span data-stu-id="e059e-151">Requires some familiarity with the model management model and Workbench tool environment.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="e059e-152">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="e059e-152">Azure Batch AI</span></span>

<span data-ttu-id="e059e-153">Azure Batch AI lehetővé teszi a machine learning kísérleteket párhuzamos futtatása, és hajtsa végre a modell betanítási léptékű Feldolgozóegységekkel rendelkező virtuális gépek fürtön belül.</span><span class="sxs-lookup"><span data-stu-id="e059e-153">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="e059e-154">Kötegelt AI képzési horizontális felskálázás mély tanulási feladatok fürtözött Feldolgozóegységekkel keresztül, például kognitív eszközkészlet Caffe, Chainer vagy TensorFlow keretrendszerek használatával teszi lehetővé.</span><span class="sxs-lookup"><span data-stu-id="e059e-154">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span> 

<span data-ttu-id="e059e-155">Modell kezelése az Azure Machine Learning segítségével kötegelt AI igénybe modellek betanítása központi telepítéséhez, kezeléséhez, és figyelheti azokat.</span><span class="sxs-lookup"><span data-stu-id="e059e-155">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span> 

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="e059e-156">Azure Machine Learning Studio</span><span class="sxs-lookup"><span data-stu-id="e059e-156">Azure Machine Learning Studio</span></span>

<span data-ttu-id="e059e-157">Az Azure Machine Learning Studio egy felhőalapú, visual fejlesztőkörnyezet adatok kísérletek létrehozása, a machine learning modellek betanítása és a közzé őket az Azure-ban webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="e059e-157">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="e059e-158">A vizuális fogd és vidd felület lehetővé teszi, hogy a adatszakértőkön és a kiemelt felhasználók machine learning megoldások gyors létrehozásához, miközben egyéni R és Python logika, a meghatározott statisztikai algoritmusok és technikák számos támogatja a machine Learning modellezési feladatok , és a Jupyter notebookok beépített támogatása.</span><span class="sxs-lookup"><span data-stu-id="e059e-158">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="e059e-159">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-159">Key benefits:</span></span>

* <span data-ttu-id="e059e-160">Interaktív vizuális felület lehetővé teszi, hogy a gépi tanulás modellezési minimális kóddal.</span><span class="sxs-lookup"><span data-stu-id="e059e-160">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
* <span data-ttu-id="e059e-161">Jupyter notebookok beépített az adatok feltárása.</span><span class="sxs-lookup"><span data-stu-id="e059e-161">Built-in Jupyter Notebooks for data exploration.</span></span>
* <span data-ttu-id="e059e-162">Közvetlen telepítése betanított modellek Azure webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="e059e-162">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="e059e-163">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-163">Considerations:</span></span>

* <span data-ttu-id="e059e-164">Korlátozott méretezhetőséget.</span><span class="sxs-lookup"><span data-stu-id="e059e-164">Limited scalability.</span></span> <span data-ttu-id="e059e-165">A képzési dataset maximális mérete 10 GB-os.</span><span class="sxs-lookup"><span data-stu-id="e059e-165">The maximum size of a training dataset is 10 GB.</span></span>
* <span data-ttu-id="e059e-166">Csak online.</span><span class="sxs-lookup"><span data-stu-id="e059e-166">Online only.</span></span> <span data-ttu-id="e059e-167">Nincs kapcsolat nélküli fejlesztési környezet.</span><span class="sxs-lookup"><span data-stu-id="e059e-167">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="e059e-168">Eszközöket és szolgáltatásokat a machine learning modellek telepítéséről</span><span class="sxs-lookup"><span data-stu-id="e059e-168">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="e059e-169">Egy adatok tudósok létre gépi tanulási modell, általában akkor telepítheti, és az alkalmazásokból, és a többi használni, azt.</span><span class="sxs-lookup"><span data-stu-id="e059e-169">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="e059e-170">Számos lehetséges telepítési célok gépi tanulási modellek.</span><span class="sxs-lookup"><span data-stu-id="e059e-170">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="e059e-171">Az Azure HDInsighthoz készült Spark</span><span class="sxs-lookup"><span data-stu-id="e059e-171">Spark on Azure HDInsight</span></span>

<span data-ttu-id="e059e-172">Az Apache Spark on Spark MLlib, a keretrendszer és a könyvtár a gépi tanulási modellek tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="e059e-172">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="e059e-173">A Microsoft Machine Learning könyvtárban Spark (MMLSpark) is biztosít a mély tanulási algoritmus támogatás a Spark prediktív modellek esetén.</span><span class="sxs-lookup"><span data-stu-id="e059e-173">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="e059e-174">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-174">Key benefits:</span></span>

* <span data-ttu-id="e059e-175">A Spark az elosztott platformot kínál a nagy mennyiségű gépi tanulási a folyamatok méretezhetőségét.</span><span class="sxs-lookup"><span data-stu-id="e059e-175">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
* <span data-ttu-id="e059e-176">Modellek közvetlenül telepíteni a Spark on HDinsight az Azure Machine Learning-munkaterület, és kezelheti azokat az Azure Machine Learning modell felügyeleti szolgáltatás használatával.</span><span class="sxs-lookup"><span data-stu-id="e059e-176">You can deploy models directly to Spark in HDinsight from Azure Machine Learning Workbench, and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="e059e-177">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-177">Considerations:</span></span>

* <span data-ttu-id="e059e-178">Spark költségeket terhel a teljes idő, fut-e HDinsght fürtben futtatja.</span><span class="sxs-lookup"><span data-stu-id="e059e-178">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="e059e-179">A machine learning szolgáltatás csak alkalmanként használható, ha emiatt előfordulhat, hogy a felesleges költségek.</span><span class="sxs-lookup"><span data-stu-id="e059e-179">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="e059e-180">A tárolóban lévő webszolgáltatás</span><span class="sxs-lookup"><span data-stu-id="e059e-180">Web service in a container</span></span>

<span data-ttu-id="e059e-181">A gépi tanulási modellek telepíthet egy Docker-tároló Python webszolgáltatásként.</span><span class="sxs-lookup"><span data-stu-id="e059e-181">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="e059e-182">A modell telepítheti az Azure-bA vagy egy peremhálózati eszköz, ahol használható helyileg az adatokat, amelyen működik.</span><span class="sxs-lookup"><span data-stu-id="e059e-182">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="e059e-183">Fő értékek:</span><span class="sxs-lookup"><span data-stu-id="e059e-183">Key Benefits:</span></span>

* <span data-ttu-id="e059e-184">Tárolók egy egyszerűsített, amelyek általában költséghatékony csomag és a szolgáltatások telepítéséhez.</span><span class="sxs-lookup"><span data-stu-id="e059e-184">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
* <span data-ttu-id="e059e-185">Lehetővé teszi a peremhálózati eszköz telepítését lehetővé teszi a prediktív logika közelebb áthelyezése az adatokat.</span><span class="sxs-lookup"><span data-stu-id="e059e-185">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>
* <span data-ttu-id="e059e-186">Egy tároló közvetlenül az Azure Machine Learning-munkaterület telepítheti.</span><span class="sxs-lookup"><span data-stu-id="e059e-186">You can deploy to a container directly from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="e059e-187">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-187">Considerations:</span></span>

* <span data-ttu-id="e059e-188">A telepítési modell a Docker-tároló, alapul, ismernie kell a technológiával így webes szolgáltatás telepítése előtt.</span><span class="sxs-lookup"><span data-stu-id="e059e-188">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="e059e-189">Microsoft Machine Learning-kiszolgáló</span><span class="sxs-lookup"><span data-stu-id="e059e-189">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="e059e-190">Machine Learning Server (korábbi nevén Microsoft R Server) szolgáltatás egy méretezhető platform, az R és Python kódok, kifejezetten a machine learning forgatókönyvek esetén.</span><span class="sxs-lookup"><span data-stu-id="e059e-190">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="e059e-191">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-191">Key benefits:</span></span>

* <span data-ttu-id="e059e-192">Méretezhetőségét.</span><span class="sxs-lookup"><span data-stu-id="e059e-192">High scalability.</span></span>
* <span data-ttu-id="e059e-193">Közvetlen telepítését az Azure Machine Learning-munkaterület.</span><span class="sxs-lookup"><span data-stu-id="e059e-193">Direct deployment from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="e059e-194">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-194">Considerations:</span></span>

* <span data-ttu-id="e059e-195">Központi telepítése és kezelése a Machine Learning-kiszolgáló a vállalat kell.</span><span class="sxs-lookup"><span data-stu-id="e059e-195">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="e059e-196">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="e059e-196">Microsoft SQL Server</span></span>

<span data-ttu-id="e059e-197">Microsoft SQL Server támogatja R és Python natív módon, amely lehetővé teszi a machine learning modellek foglalják magukban a beépített ezeken a nyelveken értelmezett Transact-SQL-adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="e059e-197">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="e059e-198">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-198">Key benefits:</span></span>

* <span data-ttu-id="e059e-199">Egy adatbázis funkciót, így könnyen adatrétegbeli logika szerepeljenek a prediktív logikai foglalják magukban.</span><span class="sxs-lookup"><span data-stu-id="e059e-199">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="e059e-200">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-200">Considerations:</span></span>

* <span data-ttu-id="e059e-201">Azt feltételezi, hogy az SQL Server-adatbázis, az alkalmazás az adatok rétegként.</span><span class="sxs-lookup"><span data-stu-id="e059e-201">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="e059e-202">Az Azure Machine Learning webszolgáltatás</span><span class="sxs-lookup"><span data-stu-id="e059e-202">Azure Machine Learning web service</span></span>

<span data-ttu-id="e059e-203">Amikor a gépi tanulási modell Azure Machine Learning Studio használatával hoz létre, telepíthet egy webszolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="e059e-203">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="e059e-204">Ez lehet majd használni egy REST-felület a bármely ügyfél alkalmazásokat, amelyek képesek a HTTP-en keresztül.</span><span class="sxs-lookup"><span data-stu-id="e059e-204">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="e059e-205">Főbb előnyei:</span><span class="sxs-lookup"><span data-stu-id="e059e-205">Key benefits:</span></span>

* <span data-ttu-id="e059e-206">A könnyű fejlesztés és üzembe helyezés.</span><span class="sxs-lookup"><span data-stu-id="e059e-206">Ease of development and deployment.</span></span>
* <span data-ttu-id="e059e-207">Webes felügyeleti portál az alapvető figyelési metrikákat.</span><span class="sxs-lookup"><span data-stu-id="e059e-207">Web service management portal with basic monitoring metrics.</span></span>
* <span data-ttu-id="e059e-208">Azure Machine Learning webszolgáltatások hívja az Azure Data Lake Analytics, az Azure Data Factory és az Azure Stream Analytics beépített támogatása.</span><span class="sxs-lookup"><span data-stu-id="e059e-208">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="e059e-209">Szempontok:</span><span class="sxs-lookup"><span data-stu-id="e059e-209">Considerations:</span></span>

* <span data-ttu-id="e059e-210">Csak Azure Machine Learning Studio használatával készített modellek esetén érhető el.</span><span class="sxs-lookup"><span data-stu-id="e059e-210">Only available for models built using Azure Machine Learning Studio.</span></span>
* <span data-ttu-id="e059e-211">Webes hozzáférést csak, betanított modellek nem futtatható, a helyszíni vagy offline állapotú.</span><span class="sxs-lookup"><span data-stu-id="e059e-211">Web-based access only, trained models cannot run on-premises or offline.</span></span>

