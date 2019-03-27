---
title: A gépi tanulási technológia kiválasztása
description: Hasonlítsa össze a létrehozását, üzembe helyezése és kezelése a gépi tanulási modellek lehetőségeit. Döntse el, melyik Microsoft-termékek kiválasztása a megoldását.
author: MikeWasson
ms.date: 03/06/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 69391133f11445283b9835cc0742bde6f2bc334d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299224"
---
# <a name="what-are-the-machine-learning-products-at-microsoft"></a>Mik azok a machine learning-termékek, a Microsoft?

A Machine Learning egy olyan adatelemzési módszer, amely lehetővé teszi, hogy a számítógépek a meglévő adatokból tanulva jövőbeni viselkedéseket, kimeneteket és trendeket jelezhessenek előre. Machine learning segítségével számítógépek információ nélkül tanulhatnak.

Gépi tanulási megoldások iteratív épülnek, és különböző fázisokból áll:

- Adatok előkészítése
- Kísérletezés és a modellek betanítása
- Betanított modellek üzembe helyezése
- Modellek kezelése üzembe helyezve.

A Microsoft számos előkészítési, elkészítheti, telepítheti és kezelheti a gépi tanulási modellek a termék lehetőséget biztosít. A termékek összehasonlításával kiválaszthatja, melyikkel lesz a leghatékonyabb a gépi tanuláson alapuló megoldásának fejlesztése.

## <a name="cloud-based-options"></a>Felhőalapú beállításai

Az alábbi lehetőségek érhetők el a machine learning az Azure-felhőben.

| Felhőalapú&nbsp;beállításai | Mi ez? | Mik a lehetőségei |
|-|-|-|
| [Azure Machine Learning szolgáltatás](#azure-machine-learning-service) | Felügyelt felhőbeli szolgáltatás a machine Learning  | Modellek betanítása, üzembe helyezése és kezelése Python és CLI használatával |
| [Az Azure Machine Learning Studióban](#azure-machine-learning-studio) | A csomóponthúzási&ndash;és&ndash;dobja el a vizuális felhasználói felületet, a machine Learning | Modellek létrehozása és üzembe helyezése, illetve tanulási kísérletek futtatása előre konfigurált algoritmusok használatával |

Ha előre elkészített mesterséges Intelligencia és gépi tanulási modelleket, a használni kívánt [Azure Cognitive Services](#azure-cognitive-services) lehetővé teszi, hogy az alkalmazások egyszerűen hozzáadhat intelligens funkciókat.

## <a name="on-premises-options"></a>Helyszíni beállítások

Az alábbi lehetőségek érhetők el a machine learning a helyszínen. A helyszíni kiszolgálók egy virtuális gépen a felhőben is futtathatja.

| A helyszíni&nbsp;beállításai | Mi ez? | Mik a lehetőségei |
|-|-|-|
| [SQL Server Machine Learning-szolgáltatások](#sql-server-machine-learning-services) | SQL-be ágyazott elemzési motor | Modellek létrehozása és üzembe helyezése SQL Serveren belül |
| [Microsoft Machine Learning Server](#microsoft-machine-learning-server) | Különálló nagyvállalati kiszolgáló prediktív elemzésekhez | Hozhat létre és helyezhet üzembe modelleket előre feldolgozott adatok |

## <a name="development-platforms-and-tools"></a>Fejlesztési platformok és eszközök

Az alábbi fejlesztési platformok és eszközök érhetők el a machine Learning szolgáltatáshoz.

| Platformok és eszközök | Mi ez? | Mik a lehetőségei |
|-|-|-|
| [Azure Data Science Virtual Machine](#azure-data-science-virtual-machine) | Virtuális gép előre telepített adatelemzési eszközökkel | Egy előre konfigurált környezet gépi tanulási megoldások fejlesztése |
| [Azure Databricks](#azure-databricks) | Spark-alapú elemzési platform | Modellek és adat-munkafolyamatok létrehozása és üzembe helyezése |
| [ML.NET](#mlnet) | Nyílt forráskódú, platformfüggetlen machine learning-SDK | Gépi tanulási megoldások .NET-alkalmazások fejlesztése |
| [Windows ML](#windows-ml) | Windows 10-es gépi tanulási platform | Betanított modellek értékelése Windows 10-es eszközökön |

## <a name="azure-machine-learning-service"></a>Azure Machine Learning szolgáltatás

[Az Azure Machine Learning szolgáltatás](/azure/machine-learning/service/overview-what-is-azure-ml.md) egy teljes körűen felügyelt felhőszolgáltatás, betanítását, üzembe helyezése és kezelése a machine learning-modellek ipari méretekben használt. Teljes körűen támogatja a nyílt forráskódú technológiákat, így több tízezer nyílt forráskódú Python-csomaggal, többek között TensorFlow-val, PyTorch-csal és scikit-learnnel is használható. Gazdag eszközök is elérhetők, például [Azure notebookok](https://notebooks.azure.com/), [Jupyter notebookok](http://jupyter.org), vagy a [a Visual Studio Code az Azure Machine Learning](https://aka.ms/vscodetoolsforai) bővítmény megkönnyíti a felfedezése és alakíthat át adatokat, és ezután betanítása és modellek üzembe helyezése. Az Azure Machine Learning szolgáltatás funkcióival a modellek generálása és finomhangolása könnyedén, hatékonyan és pontosan automatizálható.

Az Azure Machine Learning szolgáltatás használatával betanításához, üzembe helyezése és kezelése a machine learning-modellek felhőszinten Python és a parancssori felület használatával.

Próbálja ki a [Azure Machine Learning szolgáltatás ingyenes vagy fizetős verzióját](http://aka.ms/AMLFree).

|||
|-|-|
|**Típus**                   |Felhőalapú machine learning-megoldását|
|**Támogatott nyelvek**    |Python|
|**Machine learning fázisai**|Adatok előkészítése<br>Modell betanítása<br>Környezet<br>Kezelés|
|**Fő előnyei**           |Központi felügyeleti parancsfájlokat és a futtatási előzményei, így könnyen modell verziójának összehasonlítása.<br/><br/>Könnyű üzembe helyezés és kezelés modellek a felhőben és a peremhálózati eszközökre.|
|**Megfontolások**         |A modell felügyeleti modell bizonyos fokú ismeretét igényli.|

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Az [Azure Machine Learning Studio](/azure/machine-learning/studio/what-is-ml-studio.md) interaktív, grafikus munkaterületet nyújt, amelyen az előre összeállított gépi tanuláson alapuló algoritmusok használatával könnyedén hozhat létre, tesztelhet és helyezhet üzembe modelleket. A Machine Learning Studio a modelleket webszolgáltatásként teszi közzé, amelyeket az egyéni alkalmazások vagy az Excel és más üzletiintelligencia-eszközök egyszerűen felhasználhatnak.
Nincs szükség programozásra – a gépi tanuláson alapuló modellek létrehozásához elegendő az adathalmazokat és az elemzési modulokat egy interaktív vásznon összekapcsolni, ezt követően pedig a modell már néhány kattintással üzembe is helyezhető.

Akkor érdemes a Machine Learning Studiót használnia, ha programkód írása nélkül szeretne modelleket fejleszteni és üzembe helyezni.

Próbálja ki [Azure Machine Learning Studio](https://studio.azureml.net/?selectAccess=true&o=2&target=_blank), fizetős vagy ingyenes lehetőségek az elérhető.

|||
|-|-|
|**Típus**                   |Felhőalapú, fogd és vidd machine learning-megoldását|
|**Támogatott nyelvek**    |Python, R|
|**Machine learning fázisai**|Adatok előkészítése<br>Modell betanítása<br>Környezet<br>Kezelés|
|**Fő előnyei**           |Interaktív vizuális felületen lehetővé teszi, hogy a minimális programozással való modellezés, gépi tanulás.<br/><br/>Beépített Jupyter notebookok adatok feltárásához.<br/><br/>Közvetlen üzembe helyezés a betanított modellek Azure webszolgáltatásként.|
|**Megfontolások**         |Korlátozott a méretezhetősége. Betanítási adatkészletet maximális mérete 10 GB-ot.<br/><br/>Csak online. Nincs kapcsolat nélküli fejlesztési környezet.|

## <a name="azure-cognitive-services"></a>Azure Cognitive Services

Az [Azure Cognitive Services](/azure/cognitive-services/welcome) egy olyan API-készlet, amely lehetővé teszi a természetes kommunikációs módszereket használó alkalmazások összeállítását. Az API-k csupán néhány sornyi kód alapján lehetővé teszik az alkalmazások számára, hogy lássanak, halljanak, beszéljenek, megértsék és értelmezzék a felhasználó szükségleteit. Egyszerűen hozzáadhat intelligens funkciókat az alkalmazáshoz, például a következőket:

- Érzelemfelismerés
- Látvány- és beszédfelismerés
- Language understanding (LUIS)
- Ismeretek és keresés

A Cognitive Services különböző eszközökre és platformokra történő alkalmazásfejlesztésre használható. Az API-k egyre fejlettebbek lesznek, és könnyű őket beállítani.

|||
|-|-|
|**Típus**                   |API-khoz, intelligens alkalmazások készítéséhez|
|**Támogatott nyelvek**    |a szolgáltatástól függően számos lehetőség|
|**Machine learning fázisai**|Környezet|
|**Fő előnyei**           |Beépíti a gépi tanulási funkciók az alkalmazásokban előre betanított modellek használatával.<br/><br/>Számos látás- és a természetes kommunikációs módszereket a modellt.|
|**Megfontolások**         |Modellek előre betanított és nem testre szabható.|

## <a name="sql-server-machine-learning-services"></a>SQL Server Machine Learning-szolgáltatások

Az [SQL Server Microsoft Machine Learning-szolgáltatás](https://docs.microsoft.com/sql/advanced-analytics/r/r-services) az SQL Server-adatbázisok relációs adatait statisztikai elemzéssel, adatvizualizációval, illetve R- és Python-kódot használó prediktív elemzéssel egészíti ki. R és Python-kódtárakat a Microsoft SQL Server speciális modellezési és gépi tanulási algoritmusokat, ami a párhuzamos és nagy méretekben futtathatja, vegye fel.

Akkor érdemes az SQL Server Machine Learning-szolgáltatásokat használnia, ha egy előre létrehozott mesterséges intelligenciára és prediktív elemzésre van szüksége az SQL Server relációs adataihoz.

|||
|-|-|
|**Típus**                   |A helyszíni prediktív elemzési relációs adatok|
|**Támogatott nyelvek**    |Python, R|
|**Machine learning fázisai**|Adatok előkészítése<br>Modell betanítása<br>Környezet|
|**Fő előnyei**           |Egy adatbázis függvényben, így könnyen felvenni az adatrétegbeli logikai prediktív logikai magába foglalja.|
|**Megfontolások**         |Feltételezi, hogy egy SQL Server-adatbázis, az adatréteg az alkalmazáshoz.|

## <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning-kiszolgáló

A [Microsoft Machine Learning-kiszolgáló](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server) egy vállalati kiszolgáló, amely R- és Python-folyamatok párhuzamos és elosztott munkaterheléseinek futtatására és kezelésére szolgál. A Microsoft Machine Learning-kiszolgáló Linux és Windows rendszeren, illetve Hadoopon és Apache Sparkon fut, valamint elérhető a [HDInsight-on](https://azure.microsoft.com/services/hdinsight/r-server/) is. Végrehajtómotorokat biztosít a [RecoScaleR-](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler), a [revoscalepy-](https://docs.microsoft.com/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) és a [Microsoft Machine Learning-csomagokkal](https://docs.microsoft.com/r-server/r/concept-what-is-the-microsoftml-package) összeállított megoldásokhoz, és kiterjeszti a nyílt forráskódú R és Python támogatását a nagy teljesítményű elemzésre, a statisztikai elemzésre, a gépi tanulásra és a nagyméretű adatkészletekre. Ezt a funkciót a kiszolgálóval telepített jogvédett csomagok biztosítják. Fejlesztéshez olyan IDE-ket használhat, mint például az [R Tools for Visual Studio](https://www.visualstudio.com/vs/rtvs/) és a [Python Tools for Visual Studio](https://www.visualstudio.com/vs/python/).

Akkor érdemes a Microsoft Machine Learning-kiszolgálót használnia, ha egy kiszolgálón R vagy Python használatával szeretne modelleket létrehozni, vagy így készült modelleket szeretne üzembe helyezni, illetve ha R- vagy Python-betanítást szeretne nagy méretben kiosztani egy Hadoop- vagy Spark-fürtön

|||
|-|-|
|**Típus**                   |A helyszíni vállalati kiszolgáló, a prediktív elemzés|
|**Támogatott nyelvek**    |Python, R|
|**Machine learning fázisai**|Modell betanítása<br>Környezet|
|**Fő előnyei**           |Magas skálázhatóság.|
|**Megfontolások**         |Üzembe helyezése és kezelése a Machine Learning-kiszolgáló a vállalat kell.|

## <a name="azure-data-science-virtual-machine"></a>Azure Data Science Virtual Machine

Az [Azure Data Science Virtual Machine](/azure/machine-learning/data-science-virtual-machine/overview.md) egy személyre szabott virtuálisgép-környezet a Microsoft Azure-felhőben, amelyet kifejezetten adatelemzésre hoztak létre. Számos népszerű adatelemzési és egyéb eszköz található meg rajta előre telepítve és konfigurálva, amelyek jelentősen felgyorsítják az intelligens alkalmazások fejlett elemzésekhez történő összeállítását.

Azure Machine Learning szolgáltatás célként támogatja a Data Science Virtual Machine-t.
Windows és Linux Ubuntu (az Azure Machine Learning szolgáltatás nem támogatott Linux CentOS) verzió érhető el.
Egy adott verzió információiért és a benne található funkciók listájáért lásd a [Azure Data Science Virtual Machine (adatelemző virtuális gép) bemutatása](/azure/machine-learning/data-science-virtual-machine/overview.md) című témakört.

Akkor érdemes az Adatelemzési virtuális gépet használnia, ha egyetlen csomóponton kell futtatnia a feladatait. Illetve ha távolról kell virtuálisan felskáláznia a feldolgozást egyetlen gépen.

|||
|-|-|
|**Típus**                   |Testre szabott virtuálisgép-környezet adatelemzéshez|
|**Fő előnyei**           |Kevesebb idő telepítése, kezelése és beépített adatelemzési eszközzel, és a keretrendszereket.<br/><br/>A legújabb, az összes gyakran használt eszközök és keretrendszerek részét képezik.<br/><br/>Virtuális gép beállításai az intenzív adatmodellezés GPU-funkciókkal rendelkező nagy mértékben skálázható képeket is tartalmaznak.|
|**Megfontolások**         |A virtuális gép nem érhető el, ha offline állapotban van.<br/><br/>Azure-szolgáltatások díjait, virtuális gépek tekintetében, így kell lennie arra, hogy csak szükség esetén futnak.|

## <a name="azure-databricks"></a>Azure Databricks

Az [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) a Microsoft Azure Cloud Services platformra optimalizált Apache Spark-alapú elemzési platform. A Databricks integrálva van az Azure-ral, így egyetlen kattintással beállítható, zökkenőmentes munkafolyamatokat tesz lehetővé, valamint olyan interaktív munkaterületet biztosít, amellyel lehetővé válik az adatszakértők, az adatmérnökök és az üzleti elemzők közötti együttműködés.
Python-, R-, Scala- és SQL-kód használatával adatokat kérdezhet le, vizualizálhat és modellezhet webes jegyzetfüzetekben.

Akkor érdemes a Databrickset használnia, ha az Apache Sparkon másokkal együttműködve szeretne gépi tanuláson alapuló megoldásokat létrehozni.

|||
|-|-|
|**Típus**                   |Apache Spark-alapú elemzési platform|
|**Támogatott nyelvek**    |Python, R, Scala, SQL|
|**Machine learning fázisai**|Adatlekérdezés<br>Modell betanítása|

## <a name="mlnet"></a>ML.NET

Az [ML-NET](https://docs.microsoft.com/dotnet/machine-learning/) egy ingyenes, nyílt forráskódú, többplatformos gépi tanulási keretrendszer, amellyel gépi tanuláson alapuló egyéni megoldásokat hozhat létre, majd integrálhat .NET-alkalmazásokban.

Akkor érdemes az ML.NET-et használnia, ha gépi tanuláson alapuló megoldásokat szeretne integrálni a .NET-alkalmazásaiba.

|||
|-|-|
|**Típus**                   |Nyílt forráskódú keretrendszer egyéni gépi tanulásra képes alkalmazások fejlesztéséhez|
|**Támogatott nyelvek**    |.NET|

## <a name="windows-ml"></a>Windows ML

[Windows-ML](https://docs.microsoft.com/windows/uwp/machine-learning/) következtetésekhez motor lehetővé teszi, hogy betanított machine learning-modellek az alkalmazásokban, betanított modellek helyileg, a Windows 10-es eszközök kiértékelése.

Akkor érdemes a Windows ML-t használni, ha gépi tanuláson alapuló, már betanított modelleket szeretne használni a Windows-alkalmazásaiban.

|||
|-|-|
|**Típus**                   |Következtetésekhez motor betanított modellek Windows-eszközök esetén|
|**Támogatott nyelvek**    |C#/C++, JavaScript|

## <a name="next-steps"></a>További lépések

- A Microsoftnál elérhető összes mesterségesintelligencia-fejlesztési termékről a [Microsoft AI-platformon](https://www.microsoft.com/ai) olvashat.
- Az AI-megoldások fejlesztésével kapcsolatos képzésről a [Microsoft AI School](https://aischool.microsoft.com/learning-paths) oldalán tájékozódhat.
