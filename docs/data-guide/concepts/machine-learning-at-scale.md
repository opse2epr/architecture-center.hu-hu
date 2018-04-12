---
title: Gépi tanulás nagy léptékben
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: a92060008f90f43f71869bd1ad251af150b4a9db
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="machine-learning-at-scale"></a>Gépi tanulás nagy léptékben

Gépi tanulás (ML) matematikai algoritmusok alapuló prediktív modellek betanítása használt módszer. Gépi tanulás elemzi az ismeretlen értékeinek adatmezők közötti kapcsolatokat.

A következő iteratív folyamat létrehozása és telepítése a gépi tanulási modell:

* Adatszakértőkön adatokba a forrás közötti kapcsolatok meghatározásához *szolgáltatások* és előre jelzett *címkék*.
* Az adatelemzők betanítása, és keresse meg az optimális előrejelzés megfelelő algoritmusokat alapuló modellek ellenőrzése.
* Az optimális modell éles környezetben, a rendszer egy webes szolgáltatás, vagy valamilyen egyéb beágyazott függvény.
* Új adatokat kell összegyűjteni, mert a modell rendszeresen retrained javítására van hatékonyságát.

Gépi tanulás léptékű címek két különböző méretezhetőség vonatkozik. Az első van betanítása a modell tanítását kell még betanítani a fürt kibővített képességeit igényelnek nagy méretű adatkészletekhez. A második központok operationalizating oly módon, hogy méretezhető, az azt használó alkalmazások követelményeinek megismert modell. Általában ez érhető el, a prediktív képességeit, majd kiterjeszthető webszolgáltatásként üzembe helyezésével.

Gépi tanulási léptékű az előnye, hogy azt is létrehozhat hatékony, prediktív képességek, mert jobb modellek általában több adatot eredménye van. Ha egy modell betanítása, azt egy állapotmentes, magas-performant, kibővített webszolgáltatás telepíthető központilag. 

## <a name="model-preparation-and-training"></a>Minta előkészítése és a képzési

A modell előkészítése és a képzési fázis során adatszakértőkön ismerheti meg az adatokat, például a Python és az R nyelv interaktív használatával:

* Bontsa ki a minták áruházakból nagy mennyiségű adat.
* Keresse meg és kezeli a kiugró, az ismétlődések és a hiányzó értékeket adatot.
* Korrelációk és az adatokat statisztikai elemzések és a képi megjelenítés lévő kapcsolatok meghatározásához.
* Új számított funkciókat, amelyek javítják a predictiveness statisztikai kapcsolatok létrehozása.
* Vonat ML modellek prediktív algoritmusok alapján.
* Ellenőrizze a betanítás során lett futtatásában adatokkal betanított modellek.

Az interaktív elemzések elvégzéséhez és a modellezési fázis támogatásához a adatplatform engedélyeznie kell az adatszakértőkön át a különböző eszközök használatával adatokba. Emellett egy összetett gépi tanulási modell a tanítási megkövetelheti az sok nagy mennyiségű adatot, intenzív feldolgozása elegendő erőforrás a modell betanítási kiterjesztése az alapvető fontosságú.

## <a name="model-deployment-and-consumption"></a>Minta központi telepítés és használat

Amikor egy modell telepítésre kész, egy webszolgáltatás be vannak ágyazva, és a felhőbe, a peremhálózati eszköz, vagy egy vállalati ML végrehajtási környezeten belül telepíteni. Ez a telepítési folyamat operationalization nevezzük.

## <a name="challenges"></a>Problémák

Gépi tanulási léptékű néhány kihívást hoz létre:

- Általában szüksége nagy mennyiségű adatot kell még betanítani a modellt, különösen a részletes tanulási modellek.
- Elő kell készíteni a big data készletek a modell betanítása még megkezdése előtt.
- A modell betanítási fázisban kell hozzáférést, a big Data típusú adatok. Esetében gyakori, a modell betanítási ugyanazon big Data típusú adatok fürt, például a Spark, adatok előkészítése használt használatával végrehajtani. 
- Ahhoz hasonló forgatókönyvek esetén mély tanulási nem csak szükség van-e a fürt, amely képes biztosítani a CPU-n a horizontális, de a fürtnek szüksége lesz a GPU-engedélyezett csomópontok állnak.

## <a name="machine-learning-at-scale-in-azure"></a>Gépi tanulási léptékű az Azure-ban

Előtt annak eldöntése, amely a ML szolgáltatások használandó képzési és operationalization, ha a modell betanításához minden szüksége van, vagy ha egy előre elkészített modell képes igényeinek fontolja meg. Sok esetben egy előre elkészített modell használata csak egy függetlenül attól, hogy egy webes szolgáltatás hívása, vagy egy meglévő modell betöltése az ML-tár használatával. Néhány lehetőségek a következők: 

- Microsoft kognitív szolgáltatás által biztosított webes szolgáltatásait használják.
- A kognitív eszközkészlet által biztosított pretrained Neurális hálózat modellek használata.
- Az iOS-alkalmazások Core gépi tanulás által biztosított szerializált modellek beágyazása. 

Egy előre elkészített modell nem felelnek meg az adatok vagy adott esetben, ha az Azure-ban a választható lehetőségek Azure Machine Learning, HDInsight Spark MLlib és MMLSpark, kognitív eszközkészlet és SQL Machine Learning szolgáltatás. Ha úgy dönt, hogy egy egyéni modellt használja, egy folyamatot, amely tartalmazza a modell betanítási és operationalization kell tervezésekor. 

![Az Azure-ban modell beállításai](./images/machine-learning-model-training-and-deployment.png)

Az Azure ML technológia lehetőségek listáját az alábbi témakörökben található:

- [Egy kognitív technológia kiválasztása](../technology-choices/cognitive-services.md)
- [A gépi tanulási technológiával kiválasztása](../technology-choices/data-science-and-machine-learning.md)
- [Technológia feldolgozása természetes nyelv kiválasztása](../technology-choices/natural-language-processing.md)
