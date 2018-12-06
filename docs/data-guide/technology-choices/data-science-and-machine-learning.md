---
title: A gépi tanulási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 507d343ca7bc0a3f161602c50e6b96e921276374
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902361"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Kiválasztása a machine learning-technológiák az Azure-ban

Adattudományi és machine learning, illetve egy folyamat, amely általában végzi az adatszakértők. Szakértő eszközök, számos, amely kifejezetten interaktív adatfeltárás és modellezés adatszakértő kell végrehajtó tevékenységek típusú készültek van szükség.

Gépi tanulási megoldások iteratív épülnek, és két fázisban van:
* Adat-előkészítési és a modellezés.
* Üzembe helyezési és prediktív-szolgáltatások.

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>Eszközöket és szolgáltatásokat az adat-előkészítési és a modellezés
Az adatszakértők általában inkább olyan adatok, Python vagy r nyelven írt egyéni kód használatával Ez a kód generálása a képi megjelenítés és statisztikai annak meghatározásához, a kapcsolatokat, és a lekérdezésre és vizsgálódásra az adatokat, az adatszakértők, általában futását. Nincsenek számos R és Python, amellyel az adatelemzők interaktív környezeteket. Egy adott kedvenc van **Jupyter notebookok** , amely egy böngészőalapú rendszerhéj, amely lehetővé teszi az adatszakértők létrehozásához biztosít *notebook* R vagy Python kódját és markdown-szöveget tartalmazó fájlok. Ez a megosztási és dokumentálja a kód működhet hatékony módja, és egyetlen dokumentum eredményez.

Más gyakran használt eszközök a következők:
* **Spyder**: A Python, az Anaconda Python elosztási megadott interaktív fejlesztőkörnyezet (IDE).
* **Az R Studio**: egy ide-vel az R programozási nyelv.
* **A Visual Studio Code**: egy egyszerűsített, platformfüggetlen programozási környezetet, amely támogatja a Python, valamint a gyakran használt keretrendszerek gépi tanulási és AI-fejlesztést.

Ezek az eszközök mellett adatszakértők használhatják a Azure-szolgáltatások kódot és a modell kezelésének egyszerűsítéséhez.

### <a name="azure-notebooks"></a>Azure Notebooks
Az Azure notebookok online Jupyter notebookok szolgáltatás, amely lehetővé teszi az adatszakértők, létrehozása, futtatása és megosztása a felhő alapú tárak Jupyter-Notebookjait.

Fő előnyök:

* Az ingyenes szolgáltatás&mdash;nem Azure-előfizetés szükséges.
* Nem kell telepítenie a Jupyter és a támogató R vagy Python disztribúciók helyileg&mdash;használja egy böngészőben.
* A saját online tárak, és azok bármely eszközről elérhetők.
* Ossza meg a notebookok közreműködőkkel együtt.

Szempontok:

* Ön nem fér hozzá a notebookok offline állapotban lesz.
* Az ingyenes notebook szolgáltatás korlátozott feldolgozási képességek nem lehet elég nagy vagy összetett modelleket taníthat be.

### <a name="data-science-virtual-machine"></a>Adatelemző virtuális gép
Az adatelemző virtuális gép az Azure-beli virtuálisgép-lemezkép, amely tartalmazza az eszközöket és keretrendszereket gyakran használják az adatszakértők, többek között az R, Python, a Jupyter Notebooks, a Visual Studio Code és gépi tanulási modellezését, például a könyvtárak a A Microsoft Cognitive Toolkit. Ezek az eszközök lehetnek összetett és időigényes, és számos fennálló függőségeket, amelyek gyakran verziójú felügyeleti problémákat okozhat tartalmaznak. Egy előre telepített lemezképet kellene csökkentheti a idejét az adatszakértők hibaelhárítási környezetben előforduló problémák megoldására, lehetővé téve számukra az összpontosíthat az adatfeltárás és modellezés feladatok elvégzéséhez szükséges.

Fő előnyök:
* Kevesebb idő telepítése, kezelése és beépített adatelemzési eszközzel, és a keretrendszereket.
* A legújabb, az összes gyakran használt eszközök és keretrendszerek részét képezik.
* Virtuális gép beállításai az intenzív adatmodellezés GPU-funkciókkal rendelkező nagy mértékben skálázható képeket is tartalmaznak.

Szempontok:
* A virtuális gép nem érhető el, ha offline állapotban van.
* Azure-szolgáltatások díjait, virtuális gépek tekintetében, így kell lennie arra, hogy csak szükség esetén futnak.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Az Azure Machine Learning egy olyan felhőalapú szolgáltatás, machine learning-kísérletek és a modellek felügyeletére. Ez magában foglalja egy Kísérletezési szolgáltatás, amely nyomon követi az adat-előkészítési és modellezés a betanítási szkriptekhez, az összes végrehajtás előzményeit fenntartása, így összehasonlíthatja a modellek teljesítményének ismétléseinek között. Egy platformfüggetlen ügyféleszköz Azure Machine Learning Workbench nevű felületet biztosít a központi parancsfájl-kezelési és előzményeit, miközben továbbra is az adatszakértők a kívánt eszközben dolgozhat, például a Jupyter notebookok vagy a Visual Studio Code parancsfájlok létrehozására.

Az Azure Machine Learning Workbench a interaktív adatelőkészítési eszközök segítségével egyszerűsítheti a leggyakoribb Adatátalakítási feladatokhoz, és beállíthatja, hogy a parancsfájl végrehajtási környezet helyileg, a modell betanítási szkriptekhez méretezhető Docker-tárolóban vagy a Spark.

Ha készen áll a modell üzembe helyezése, használja a munkaterület-környezet a modell csomagolása és üzembe webszolgáltatásként, amely egy Docker-tárolóba, a Spark on Azure HDinsight, a Microsoft Machine Learning-kiszolgáló vagy az SQL Server. Az Azure Machine Learning Modellkezelés szolgáltatás majd lehetővé teszi, hogy nyomon követheti és kezelheti a modell-üzembehelyezések a felhőben, a peremhálózati eszközökön, vagy a vállalaton belül.

Fő előnyök:

* Központi felügyeleti parancsfájlokat és a futtatási előzményei, így könnyen modell verziójának összehasonlítása.
* Interaktív adatok átalakítása egy vizuális szerkesztő.
* Könnyű üzembe helyezés és kezelés modellek a felhőben és a peremhálózati eszközökre.

Szempontok:
* A modell felügyeleti modell és a Workbench eszköz környezet bizonyos fokú ismeretét igényli.

### <a name="azure-batch-ai"></a>Azure Batch AI

Az Azure Batch AI lehetővé teszi a machine learning-kísérletek párhuzamos futtatása, és a virtuális gépek a GPU-fürtön belül hajtsa végre a modell betanítást szeretne nagy méretben. A Batch AI training lehetővé teszi a horizontális felskálázás mély tanulási feladatok csoportosított gpu-kon keresztül, keretrendszerekkel, mint például a Cognitive Toolkit, Caffe, Chainer és tensorflow-hoz. 

Az Azure Machine Learning Modellkezelés használható modellek átvételére a Batch AI képzési üzembe helyezéséhez és kezeléséhez, és megfigyelheti őket. 

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Az Azure Machine Learning Studióban egy felhőalapú, visual fejlesztési környezetet, a data-kísérletek létrehozása machine learning-modellek betanítása és ezek közzététele webszolgáltatásként az Azure-ban. A vizuális fogd és vidd felület lehetővé teszi, hogy az adatszakértők és a kiemelt felhasználók gépi tanulási megoldások gyors létrehozásához, miközben támogatja az egyéni R és Python logika, a létrehozott statisztikai algoritmusok és technikák számos machine Learning-tevékenységek modellezése , és a Jupyter notebookok beépített támogatását.

Fő előnyök:

* Interaktív vizuális felületen lehetővé teszi, hogy a minimális programozással való modellezés, gépi tanulás.
* Beépített Jupyter notebookok adatok feltárásához.
* Közvetlen üzembe helyezés a betanított modellek Azure webszolgáltatásként.

Szempontok:

* Korlátozott a méretezhetősége. Betanítási adatkészletet maximális mérete 10 GB-ot.
* Csak online. Nincs kapcsolat nélküli fejlesztési környezet.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Eszközök és szolgáltatások üzembe helyezéséhez a machine learning-modellek

Adatszakértő rendelkezik egy machine learning-modellek létrehozása után általában kell üzembe helyezni és felhasználását, alkalmazások vagy más adatok folyamatokban. Számos lehetséges telepítési céljainak gépi tanulási modelleket.

### <a name="spark-on-azure-hdinsight"></a>Az Azure HDInsighthoz készült Spark

Az Apache Spark tartalmazza a Spark MLlib, keretrendszer és gépi tanulási modelleket könyvtárában. A Microsoft Machine Learning library for Spark (MMLSpark) is biztosít a deep learning-prediktív modelleket a Spark algoritmus támogatása.

Fő előnyök:

* A Spark, elosztott platformot, amely nagy mennyiségű machine learning-folyamatokat a nagy mértékű skálázhatóságot biztosít.
* Modellek üzembe helyezése közvetlenül a Spark on HDinsight az Azure Machine Learning Workbench, és kezelheti azokat az Azure Machine Learning Modellkezelés szolgáltatás használatával.

Szempontok:

* A Spark Hdinsight-fürtök, hogy fut-e a teljes idő költségeket terhel futtatja. A machine learning-szolgáltatás csak alkalmanként használható, ha emiatt előfordulhat, hogy a felesleges költségek.

### <a name="azure-databricks"></a>Azure Databricks

[Az Azure Databricks](/azure/azure-databricks/) egy Apache Spark-alapú elemzési platform. Felfoghatók úgy, mint "Spark szolgáltatás." A Spark használata az Azure platformon legegyszerűbb módja. Machine Learning szolgáltatás használható [MLFlow](https://www.mlflow.org/), [Databricks futtatókörnyezet ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib és mások. További információkért lásd: [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html). 

### <a name="web-service-in-a-container"></a>Webszolgáltatás-tárolóban

Telepíthet egy gépi tanulási modellt a Docker-tárolóban Python webszolgáltatásként. Is üzembe a modellt, az Azure-bA vagy egy edge-eszközön, ahol használat helyben az adatokat, amelyen működik.

Fő értékek:

* Tárolók csomagolása és üzembe helyezése a szolgáltatások egy könnyen használható és általánosan költséghatékony módon.
* Edge-eszköz üzembe helyezése lehetővé teszi lehetővé teszi, hogy közelebb a prediktív logikai áthelyezheti az adatokat.
* Egy tárolóba közvetlenül az Azure Machine Learning Workbench telepítése.

Szempontok:

* Ez a telepítési modell alapuló Docker-tárolókat, ismernie kell a technológia az ezzel a módszerrel a web Service szolgáltatásának telepítése előtt.

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning-kiszolgáló

Machine Learning-kiszolgáló (korábbi nevén Microsoft R Server) az R és Python kódok, kifejezetten a machine learning-forgatókönyvekhez készült méretezhető platform.

Fő előnyök:

* Magas skálázhatóság.
* Közvetlen üzembe helyezés az Azure Machine Learning Workbench alkalmazásban.

Szempontok:

* Üzembe helyezése és kezelése a Machine Learning-kiszolgáló a vállalat kell.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

A Microsoft SQL Server támogatja az R és Python natív módon, az ezeken a nyelveken Transact-SQL függvényeket egy adatbázisban, amely lehetővé teszi, hogy magába foglalja a machine learning-modellek beépített.

Fő előnyök:

* Egy adatbázis függvényben, így könnyen felvenni az adatrétegbeli logikai prediktív logikai magába foglalja.

Szempontok:

* Feltételezi, hogy egy SQL Server-adatbázis, az adatréteg az alkalmazáshoz.

### <a name="azure-machine-learning-web-service"></a>Az Azure Machine Learning-webszolgáltatás

Amikor egy gépi tanulási modellt az Azure Machine Learning Studio használatával hoz létre, telepíthet webszolgáltatásként. Ez majd használhatók fel a HTTP-alapú kommunikáció kompatibilis ügyfélalkalmazások REST-interfészen keresztül.

Fő előnyök:

* Egyszerű fejlesztéséhez és üzembe helyezéséhez.
* Webes szolgáltatás felügyeleti portáljára alapszintű figyelési metrikákat.
* Az Azure Machine Learning webszolgáltatások hívása az Azure Data Lake Analytics, Azure Data Factory és az Azure Stream Analytics beépített támogatása.

Szempontok:

* Csak a modellek Azure Machine Learning Studio használatával érhető el.
* Webes hozzáférés csak, betanított modellek nem futtatható helyi vagy offline állapotú.

