---
title: Gépi tanulás nagy léptékben
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 962e03f78015904aba4a97e9a39a662857f149ad
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113533"
---
# <a name="machine-learning-at-scale"></a>Gépi tanulás nagy léptékben

Machine learning (gépi tanulás) a technikával matematikai algoritmusokon alapuló prediktív modelleket taníthat be. Machine learning elemzi az ismeretlen értékeinek datová Pole közötti kapcsolatokat.

A következő iteratív folyamat létrehozása és telepítése a machine learning-modellek:

- Az adatszakértők, ismerje meg az adatforrás közötti kapcsolatok meghatározása *funkciók* és előrejelzett *címkék*.
- Az adatszakértők betanítása és alapján keresse meg az optimális előrejelzéshez megfelelő algoritmusokat modellek ellenőrzése.
- Az optimális modell üzembe lett helyezve, éles környezetben, egy webszolgáltatás, vagy valamilyen egyéb beágyazott függvényt.
- Ahogy gyűlnek az adatok új, akkor a modell rendszeres időközönként retrained, a hatékonyság javítása érdekében.

Gépi tanulás nagy mennyiségű címek két különböző méretezhetőségi problémákat. Az első van képzés a modell betanításához egy fürt horizontális felskálázhatóságát igénylő nagy méretű adatkészleteken. A második központok operationalizating a megismert modellt úgy, hogy a videólejátszást az alkalmazások által felhasználható, rugalmasan méretezheti. Általában ez történik a prediktív képességeket, majd kiterjeszthető webszolgáltatásként üzembe helyezésével.

A Machine learning ipari méretekben az előny, hogy azt is tudott hatékony, prediktív képességeket, hatékonyabb modelleket általában több adatot eredménye, mert rendelkezik. Miután a modell tanítása, egy állapot nélküli,-nagy teljesítményű, horizontális felskálázás webszolgáltatás telepíthető.

## <a name="model-preparation-and-training"></a>A modellekre előkészítési és -képzés

A modell előkészítése és képzési fázis során az adatszakértők ismerje meg az adatokat interaktív módon a például a Python és az R nyelv használatával:

- Bontsa ki a minták nagy mennyiségű adattárakból származó.
- Keresse meg és kezelje a kiugró értékek, az ismétlődések és a hiányzó értékek törölje.
- Összefüggéseket és az adatokat statisztikai elemzéssel és vizualizációs kapcsolatokat határozzák meg.
- Hozzon létre új számított funkciók, amelyek javítják a predictiveness statisztikai kapcsolatok.
- Train gépi Tanulási modellek alapján prediktív algoritmusait.
- Ellenőrizze, hogy a betanítás során lett adókról adatok felhasználásával betanított modellek.

Támogatja a interaktív elemzés és modellezés fázis, a data platform engedélyezni kell az adatszakértők adatfeltárás segítségével számos különféle eszközzel. Ezenkívül egy összetett gépi tanulási modell a betanítási lehet szükség nagy mennyiségű adat, az intenzív feldolgozási rengeteg, így elegendő erőforrás a modell betanítása bővítési elengedhetetlen.

## <a name="model-deployment-and-consumption"></a>Modell üzembe helyezés és használat

Amikor egy modell telepítésre kész, beágyazott webszolgáltatásként, és a felhőben, az edge-eszközöket, vagy egy vállalati gépi Tanulási végrehajtási környezetben telepített. Ez a telepítési folyamat operacionalizálás nevezzük.

## <a name="challenges"></a>Problémák

A Machine learning ipari méretekben néhány kihívásokat hoz létre:

- Rendszerint nagy mennyiségű adatot egy modell, különösen a deep learning-modellek betanításához.
- Ezen big data jellegű adatkészletek előkészítése a modell még megkezdése előtt kell.
- A modell a betanítási fázisban a big data-tárakat kell elérniük. Szokás hajtsa végre a modell képzést nyújt a big Data típusú adatok fürtön, például a Spark-, adat-előkészítési szolgálja ki.
- Forgatókönyvek, például a mély tanulás nem csak szüksége lesz egy fürtöt, amely képes biztosítani a processzorok horizontális felskálázás, de a fürt kell GPU-kompatibilis csomópontok állnak.

## <a name="machine-learning-at-scale-in-azure"></a>A Machine learning ipari méretekben az Azure-ban

Mielőtt mellett dönt, amelyek Machine Learning services képzési és operacionalizálás, fontolja meg a akár a modell betanításához minden van szüksége, vagy ha egy előre létrehozott modellt is megfelelnek az elvárásainak. Sok esetben egy előre elkészített modellel: mindössze egy webes szolgáltatás hívása, vagy egy meglévő modell betöltése egy gépi Tanulási kódtár használatával. Egyes lehetőségek a következők:

- A Microsoft Cognitive Services által kínált webes szolgáltatásait használják.
- A Cognitive Toolkit nyújtotta Neurális hálózat imagenet modellek használata.
- Ágyazza be a szerializált modell egy iOS-alkalmazások alapvető gépi tanulás által biztosított.

Előre összeállított modellek az adatok vagy a forgatókönyv nem fér el, ha az Azure-ban lehet például az Azure Machine Learning, HDInsight Spark MLlib és MMLSpark, az Azure Databricks, a Cognitive Toolkit és SQL Machine Learning-szolgáltatások. Ha úgy dönt, hogy a egyéni modell, egy folyamatot, amely tartalmazza a modell betanítása és operacionalizálás kell alakítja ki.

![Modell beállításai az Azure-ban](./images/machine-learning-model-training-and-deployment.png)

Az Azure-ban gépi Tanulási technológia választási listája a következő témakörökben talál:

- [A cognitive services technológia kiválasztása](../technology-choices/cognitive-services.md)
- [A gépi tanulási technológia kiválasztása](../technology-choices/data-science-and-machine-learning.md)
- [A természetes nyelvi feldolgozási technológia kiválasztása](../technology-choices/natural-language-processing.md)

## <a name="next-steps"></a>További lépések

A következő referenciaarchitektúrákat bemutatják a machine learning-forgatókönyvek az Azure-ban:

- [Kötegelt pontozási az Azure-on deep learning-modellek](../../reference-architectures/ai/batch-scoring-deep-learning.md)
- [Valós idejű pontozási Python Scikit-ismerje meg, és a Deep Learning-modellek az Azure-ban](../../reference-architectures/ai/realtime-scoring-python.md)