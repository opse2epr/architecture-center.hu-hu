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
# <a name="choosing-a-machine-learning-technology-in-azure"></a>A gépi tanulási technológiával az Azure-ban kiválasztása

Adattudomány és a gépi tanulás egy munkaterhelés –, amelyek adatszakértőkön általában végzi. Szükségel specialistája eszközök sok kifejezetten az adatok interaktív áttekintését és feladatokat, végezze el a adatok tudósok modellezési típusú készültek.

Machine learning megoldások ismételt épülnek, és két különböző fázisokból áll:
* Adatok előkészítése és a modellezési.
* Központi telepítés és használat prediktív szolgáltatások.

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>Eszközök és adatok előkészítése és a modellezési szolgáltatás
Adatszakértőkön általában inkább a Python vagy r nyelven írt egyéni kód használatával adatok Ez a kód képi megjelenítések és annak meghatározásához, a kapcsolatokkal statisztikák létrehozása segítségével lekérdezéséhez és az adatokba, adatszakértőkön interaktív módon, általában futtatásakor. Nincsenek R és Python adatszakértőkön használó sok interaktív környezetben. Egy adott kedvenc van **Jupyter notebookok** egy böngészőalapú felület, amely lehetővé teszi az adatszakértőkön át a létrehozása, amely biztosít *notebook* R vagy Python kódját és markdown szöveget tartalmazó fájlokat. Ez megosztása és dokumentálásához kód együttműködni hatékony módot, és egyetlen dokumentum eredményez.

A gyakran használt eszközöket a következők:
* **Spyder**: A Python, az Anaconda Python elosztási biztosított interaktív fejlesztőkörnyezet (IDE).
* **R Studio**: egy IDE az r programozási nyelv.
* **A Visual Studio Code**: egy egyszerűsített, platformfüggetlen kódolási környezetben, amely támogatja a Python, valamint a gyakran használt keretrendszerek gépi tanulási és AI fejlesztési.

Ezek az eszközök mellett adatszakértőkön használhatják az Azure-szolgáltatások egyszerűbbé teheti a kód és a modell kezelését.

### <a name="azure-notebooks"></a>Azure Notebooks
Azure notebookok egy online Jupyter notebookok szolgáltatás, amely lehetővé teszi az adatszakértőkön át a létrehozása, futtatása és a felhő alapú tárak Jupyter notebookok megosztásához.

Főbb előnyei:

* Ingyenes service&mdash;nincs Azure-előfizetés szükséges.
* Nem szükséges telepíteni a Jupyter és a támogató R vagy Python felosztás helyileg&mdash;használja a böngészőben.
* A saját online tárak, és azokat bármilyen eszközön keresztüli elérését.
* A notebookok megosztása közreműködők.

Szempontok:

* Nem érhető el kapcsolat nélküli módban jegyzetfüzetek fogjuk.
* A szabad notebook szolgáltatás korlátozott képességek nem lehet elég nagy vagy összetett modell betanításához.

### <a name="data-science-virtual-machine"></a>Adatok tudományos virtuális gép
Az adatok tudományos virtuális gép egy Azure virtuálisgép-lemezkép, amely tartalmazza az eszközök és keretrendszerek adatszakértőkön, beleértve az R, Python, Jupyter notebookok, Visual Studio Code és a gépi tanulás modellezési például a könyvtárak gyakran használják a Microsoft kognitív eszközkészlet. Ezek az eszközök összetett és időigényes, és sok egymástól függő szolgáltatásainak, így gyakran verziójú felügyeleti problémák tartalmaznak a lehet. Előre telepített lemezkép rendelkező csökkentheti a adatszakértőkön kiosztására szánt problémamegoldást környezet lehetővé teheti, hogy az adatok feltárása összpontosítani idejét, és modellezési feladatok elvégzéséhez szükséges.

Főbb előnyei:
* Kevesebb idő telepítése, kezelése és hibaelhárítását adatok tudományos eszközök és -keretrendszerek számára.
* A legújabb verzióját az összes használt eszközök és érhetők el keretrendszerek.
* Magas szinten méretezhető képek a GPU intenzív adatok modellezési kezelésére képes a virtuális gép lehetőségek között.

Szempontok:
* A virtuális gép nem érhető el, ha a kapcsolat nélküli.
* Egy virtuális gépet futtat terhel Azure, ezért meg kell ügyeljen arra, hogy csak szükség esetén futnak.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Az Azure Machine Learning egy olyan felhőalapú szolgáltatás, gépi tanulási kísérletekhez és modellek kezeléséhez. Ez magában foglalja egy kísérleti szolgáltatás, amely nyomon követi az adatok előkészítése és a modellezési képzési parancsfájlok fenntartása minden végrehajtások előzményeit modell teljesítmény összehasonlítás ismétlési között. Egy Azure Machine Learning-munkaterület nevű platformfüggetlen ügyfél eszköz központi kezelőfelületet biztosít parancsfájl-kezelési és előzményeit, továbbra is engedélyezze az adatszakértőkön át a választott, például a Jupyter notebookok vagy a Visual Studio Code eszköz parancsfájlokat hozhatnak létre.

Az Azure Machine Learning-munkaterület a interaktív adatok előkészítése eszközök segítségével egyszerűsítheti a közös adatok átalakítása feladatok, és beállíthatja, hogy a parancsfájl végrehajtási környezetet helyileg, a modell betanítási parancsfájlok futtatásához méretezhető Docker-tároló vagy a Spark.

Ha készen áll a modell rendszerbe állítása, a munkaterület környezet segítségével a modell csomagot, és telepítse azt egy Docker-tároló, a Spark on Azure HDinsight, Microsoft Machine Learning Server vagy SQL Server webszolgáltatásként. Az Azure Machine Learning modell Management szolgáltatás majd lehetővé teszi, hogy nyomon követhető, és a felhőben, a peremhálózati eszköz, vagy a vállalaton belül a modell központi telepítések felügyeletéhez szükséges.

Főbb előnyei:

* Központi felügyeleti parancsfájlok és futtatási előzményei, így könnyen modellverziók összehasonlítani.
* Interaktív adatok átalakítása visual szerkesztő keresztül.
* Egyszerű telepítés és a felhő vagy a peremhálózati eszköz számára modellek kezelése.

Szempontok:
* A modell felügyeleti modellt és a munkaterületet üzemeltető eszköz környezetet bizonyos fokú ismeretét igényli.

### <a name="azure-batch-ai"></a>Azure Batch AI

Azure Batch AI lehetővé teszi a machine learning kísérleteket párhuzamos futtatása, és hajtsa végre a modell betanítási léptékű Feldolgozóegységekkel rendelkező virtuális gépek fürtön belül. Kötegelt AI képzési horizontális felskálázás mély tanulási feladatok fürtözött Feldolgozóegységekkel keresztül, például kognitív eszközkészlet Caffe, Chainer vagy TensorFlow keretrendszerek használatával teszi lehetővé. 

Modell kezelése az Azure Machine Learning segítségével kötegelt AI igénybe modellek betanítása központi telepítéséhez, kezeléséhez, és figyelheti azokat. 

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Az Azure Machine Learning Studio egy felhőalapú, visual fejlesztőkörnyezet adatok kísérletek létrehozása, a machine learning modellek betanítása és a közzé őket az Azure-ban webszolgáltatásként. A vizuális fogd és vidd felület lehetővé teszi, hogy a adatszakértőkön és a kiemelt felhasználók machine learning megoldások gyors létrehozásához, miközben egyéni R és Python logika, a meghatározott statisztikai algoritmusok és technikák számos támogatja a machine Learning modellezési feladatok , és a Jupyter notebookok beépített támogatása.

Főbb előnyei:

* Interaktív vizuális felület lehetővé teszi, hogy a gépi tanulás modellezési minimális kóddal.
* Jupyter notebookok beépített az adatok feltárása.
* Közvetlen telepítése betanított modellek Azure webszolgáltatásként.

Szempontok:

* Korlátozott méretezhetőséget. A képzési dataset maximális mérete 10 GB-os.
* Csak online. Nincs kapcsolat nélküli fejlesztési környezet.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Eszközöket és szolgáltatásokat a machine learning modellek telepítéséről

Egy adatok tudósok létre gépi tanulási modell, általában akkor telepítheti, és az alkalmazásokból, és a többi használni, azt. Számos lehetséges telepítési célok gépi tanulási modellek.

### <a name="spark-on-azure-hdinsight"></a>Az Azure HDInsighthoz készült Spark

Az Apache Spark on Spark MLlib, a keretrendszer és a könyvtár a gépi tanulási modellek tartalmazza. A Microsoft Machine Learning könyvtárban Spark (MMLSpark) is biztosít a mély tanulási algoritmus támogatás a Spark prediktív modellek esetén.

Főbb előnyei:

* A Spark az elosztott platformot kínál a nagy mennyiségű gépi tanulási a folyamatok méretezhetőségét.
* Modellek közvetlenül telepíteni a Spark on HDinsight az Azure Machine Learning-munkaterület, és kezelheti azokat az Azure Machine Learning modell felügyeleti szolgáltatás használatával.

Szempontok:

* Spark költségeket terhel a teljes idő, fut-e HDinsght fürtben futtatja. A machine learning szolgáltatás csak alkalmanként használható, ha emiatt előfordulhat, hogy a felesleges költségek.

### <a name="web-service-in-a-container"></a>A tárolóban lévő webszolgáltatás

A gépi tanulási modellek telepíthet egy Docker-tároló Python webszolgáltatásként. A modell telepítheti az Azure-bA vagy egy peremhálózati eszköz, ahol használható helyileg az adatokat, amelyen működik.

Fő értékek:

* Tárolók egy egyszerűsített, amelyek általában költséghatékony csomag és a szolgáltatások telepítéséhez.
* Lehetővé teszi a peremhálózati eszköz telepítését lehetővé teszi a prediktív logika közelebb áthelyezése az adatokat.
* Egy tároló közvetlenül az Azure Machine Learning-munkaterület telepítheti.

Szempontok:

* A telepítési modell a Docker-tároló, alapul, ismernie kell a technológiával így webes szolgáltatás telepítése előtt.

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning-kiszolgáló

Machine Learning Server (korábbi nevén Microsoft R Server) szolgáltatás egy méretezhető platform, az R és Python kódok, kifejezetten a machine learning forgatókönyvek esetén.

Főbb előnyei:

* Méretezhetőségét.
* Közvetlen telepítését az Azure Machine Learning-munkaterület.

Szempontok:

* Központi telepítése és kezelése a Machine Learning-kiszolgáló a vállalat kell.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server

Microsoft SQL Server támogatja R és Python natív módon, amely lehetővé teszi a machine learning modellek foglalják magukban a beépített ezeken a nyelveken értelmezett Transact-SQL-adatbázisban.

Főbb előnyei:

* Egy adatbázis funkciót, így könnyen adatrétegbeli logika szerepeljenek a prediktív logikai foglalják magukban.

Szempontok:

* Azt feltételezi, hogy az SQL Server-adatbázis, az alkalmazás az adatok rétegként.

### <a name="azure-machine-learning-web-service"></a>Az Azure Machine Learning webszolgáltatás

Amikor a gépi tanulási modell Azure Machine Learning Studio használatával hoz létre, telepíthet egy webszolgáltatás. Ez lehet majd használni egy REST-felület a bármely ügyfél alkalmazásokat, amelyek képesek a HTTP-en keresztül.

Főbb előnyei:

* A könnyű fejlesztés és üzembe helyezés.
* Webes felügyeleti portál az alapvető figyelési metrikákat.
* Azure Machine Learning webszolgáltatások hívja az Azure Data Lake Analytics, az Azure Data Factory és az Azure Stream Analytics beépített támogatása.

Szempontok:

* Csak Azure Machine Learning Studio használatával készített modellek esetén érhető el.
* Webes hozzáférést csak, betanított modellek nem futtatható, a helyszíni vagy offline állapotú.

