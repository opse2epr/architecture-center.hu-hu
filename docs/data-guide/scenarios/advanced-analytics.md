---
title: Bővített analitika
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: cba2efe04e9588874d1dab7f96f39b6b35fbe8d8
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52901694"
---
# <a name="advanced-analytics"></a>Bővített analitika

Bővített analitika megkezdte a korábbi jelentéskészítési és adatok összesítésére a hagyományos üzleti intelligenciára épülő (BI), és matematikai, használja a valószínűségi és statisztikai modellezési eljárások prediktív feldolgozási és automatizált döntéshozatal engedélyezése .

Fejlett elemzési megoldásokat általában a következő számítási feladatokkal jár:

* Interaktív adatfeltárás és vizualizációs
* Machine Learning-modell betanítása
* Valós idejű és prediktív kötegelt feldolgozás

A legtöbb haladó analytics architektúrák vagy azok egy részét a következő összetevőket tartalmazza:

* **Adattárolás**. Fejlett analitikai megoldások machine learning-modellek betanításához szükséges. Az adatszakértők általában kell az adatfeltárás, prediktív funkcióit és statisztikai kapcsolatai őket és az értékek azok előrejelzése (más néven címkék). Az előre jelzett címke lehet mennyiségi értékének, például valamit, a jövőben vagy repülési késleltetés időtartama percben pénzügyi értékét. Vagy, előfordulhat, hogy egy kategorikus osztályt, például "true" vagy "false", "eszközmegfelelőségre késleltetés" vagy "repülési késleltetés nélkül" vagy hasonló "alacsony kockázat," kategóriák "Közepes kockázatot jelentő" vagy "magas kockázatú."

* **Kötegelt feldolgozás**. Machine learning-modell betanításához általában kell feldolgozni a betanítási adatok nagy mennyiségű. A modell tanítása (sorrendjéről óra, perc) hosszabb időt is igénybe vehet. Ez a képzés is elvégezhető, például a Python vagy R nyelven írt szkriptekkel, és elosztott feldolgozási platformokon, például az Apache Spark HDInsight- vagy Docker-tárolóban lévő üzemeltetett használatával képzési idő csökkentése érdekében kiterjeszthető.

* **Valós idejű üzenetbetöltés**. Éles környezetben számos speciális analitikai hírcsatorna valós idejű adatstreamek egy prediktív modellt, amely webszolgáltatásként, amely közzé lett téve. A bejövő adatfolyam általában bekerül az üzenetsorba valamilyen és a egy adatfolyam-feldolgozó motor lekéri az adatokat az adott üzenetsorban, és a bemeneti adatok közel valós időben az előrejelzés vonatkozik.  

* **Streamfeldolgozás**. Miután a betanított modell, előrejelzési (vagy pontozási) általában egy nagyon gyors művelet (sorrendjéről milliszekundum) az adott szolgáltatást. Valós idejű üzenetek rögzítése után a szolgáltatás megfelelő értékek a prediktív szolgáltatás hozza létre előre jelzett címke adható át.

* **Analitikus adattár**. Bizonyos esetekben a címke előre jelzett értékek írt jelentéskészítési analitikai adattár és a jövőbeli elemzések.

* **Elemzés és jelentéskészítés**. Ahogy a neve is sugallja, fejlett elemzési megoldásokat általában termék valamilyen jelentést vagy analitikai csatorna, amely tartalmazza az előre jelzett értékek. Gyakran címke előre jelzett értékek használhatók valós idejű irányítópultok feltöltéséhez.

* **Vezénylés**. Bár a kezdeti adatáttekintés és modellezés interaktív módon történik az adatszakértők, számos fejlett analitikai megoldások rendszeres időközönként új adatokat modellek újbóli betanítása &mdash; folyamatosan Finomítás a modell pontosságát. Ez átképezési automatizálható egy előkészített munkafolyamat használatával.

## <a name="machine-learning"></a>Gépi tanulás
A Machine learning szolgáltatás egy matematikai modellezési módszer, amellyel betanítunk egy prediktív modellt. Az általános elvet, hogy egy nagy méretű adathalmazt tartalmaz a mezők közötti kapcsolatok felfedezhető előzményadatok száma egy statisztikai algoritmussal vonatkoznak.

Machine learning modellezési általában az adatelemzők, akik alaposan vizsgálata, és mielőtt a modell tanítása az adatok előkészítése történik. A feltárás és -előkészítés általában magában foglalja az interaktív adatelemzés és vizualizációs nagy fokú &mdash; általában ebben a feladatban a interaktív eszközökkel és környezeteket, amelyek kifejezetten a például a Python és az R nyelv használatával.

Bizonyos esetekben előfordulhat, hogy kell használni [modellek imagenet](/machine-learning-server/install/microsoftml-install-pretrained-models) , amely a betanítási adatok kapott, és a Microsoft által kifejlesztett kapható. A modellek imagenet előnye, hogy pontozását, majd új tartalom azonnal, besorolását akkor is, ha nem rendelkezik a szükséges betanítási adatok, az erőforrások kezelése a nagyméretű adathalmazok vagy összetettebb modelleket taníthat be.

Machine Learning két tág kategóriába sorolhatók:

* **Felügyelt tanítással**. Felügyelt tanítás a gépi tanulás által leggyakrabban használt módszer. A felügyelt tanítási modell, a forrásadatok egy készlete áll *funkció* datová Pole egy vagy több matematikai kapcsolattal rendelkező *címke* adatmezőket. A képzési fázisban a gépi tanulási folyamat az adatkészlet a funkciók és ismert címkéket is tartalmaz, és az algoritmus alkalmazva van a függvény, amely az a funkciók alapján számítja ki a megfelelő címke előrejelzések működik megfelelően. Általában egy részét a betanítási adatkészletet visszatarthatóak és a teljesítmény a betanított modell érvényesítéséhez használt. Ha a modell rendelkezik betanítva, azt telepíthető éles környezetben, és képes előre jelezni az ismeretlen értékeket. 

* **Tanítást**. Az felügyeletlen tanulási modell a betanítási adatok nem tartalmazza a ismert címkeértéket. Ehelyett az algoritmus lehetővé teszi az előrejelzéseket, az adatok első kitéve a alapján. A leggyakoribb formája, felügyeletlen learning *Fürtszolgáltatás*, ahol az algoritmus határozza meg a legjobb módszer az adatok felosztása a megadott számú fürtök a szolgáltatások statisztikai Hasonlóságok alapján. A fürtözés előre jelzett eredménye a fürt szám, amely a bemeneti funkciói tartoznak. Néha felhasználásuk közvetlenül a hasznos előrejelzéseket, például a Fürtszolgáltatás használatával azonosíthatja a felhasználók, akik egy adatbázisban csoportjait készítése közben felügyeletlen learning megközelítések gyakrabban azonosítására szolgálnak mely adatokat biztosít a leghasznosabb egy felügyelt tanítás algoritmus a modell betanítása.

Kapcsolódó Azure-szolgáltatások:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning-kiszolgáló (az R Server), a HDInsight](/azure/hdinsight/r-server/r-server-overview)

## <a name="deep-learning"></a>Deep learning

Machine learning-modellek lineáris vagy logisztikai regressziós matematikai eljárások alapján egy kis ideig elérhető törölték. Újabban a használatának *deep learning* növekedett a Neurális hálózatokat alapuló technikákkal. Ez hajtja, részben rugalmasan méretezhető feldolgozási rendszerek, amelyek csökkentik a mennyi ideig tart összetettebb modelleket taníthat be rendelkezésre állását. Ezenkívül big Data típusú adatok nagyobb gyakoriságának megkönnyíti tartományok különböző mély tanulási modelleket taníthat be.

A fejlett analitikai architektúra tervezésekor fontolja meg a deep learning-modellek nagy léptékű feldolgozásával van szükség. Ezek az elosztott feldolgozási platformokon, például az Apache Spark és a legújabb generációs virtuális gépek, amely hozzáférést biztosít a GPU-hardveres keresztül adható meg.

Kapcsolódó Azure-szolgáltatások:

- [Deep Learning virtuális gép](/azure/machine-learning/data-science-virtual-machine/deep-learning-dsvm-overview)
- [Az Apache Spark on HDInsight](/azure/hdinsight/spark/apache-spark-overview)

## <a name="artificial-intelligence"></a>Mesterséges intelligencia

Mesterséges intelligencia (AI) forgatókönyvek, ahol a gép az emberi értesülhet például learninggel és a probléma megoldására társított cognitive függvények utánozza hivatkozik. Mesterséges Intelligencia gépi tanulási algoritmusokat használja, mert egy összevonó kifejezésként tekinthetők meg. A legtöbb AI-megoldások támaszkodnak prediktív szolgáltatás gyakran megvalósítása web services és a természetes nyelvi felületek, például a látás, a szöveges vagy beszéd útján kommunikálhassanak, amely a mobileszközökön és egyéb ügyfelek futó AI-alkalmazások jelennek meg. Bizonyos esetekben a gépi tanulási modellt az AI-alkalmazással van beágyazva. 

## <a name="model-deployment"></a>Modell-üzembehelyezés

A prediktív szolgáltatások, amelyek támogatják az AI-alkalmazások egyedi gépi tanulási modellek vagy off-the-shelf cognitive services imagenet modellek elérését biztosító is használhatja. Egyéni modellek éles környezetbe való üzembe helyezés folyamata az úgynevezett operacionalizálás, amennyiben az ugyanazon AI-modellek, amelyek betanított és a feldolgozási környezetben tesztelt szerializált és elérhetővé a külső alkalmazások és szolgáltatások a batch vagy önkiszolgáló előrejelzéseket. A prediktív modell funkció azt van deszerializálását és az azonos gépi tanulási kódtár, amely tartalmazza az algoritmus a modell betanításához az elsőként használt tölthetők be. Ebben a könyvtárban olyan prediktív függvények (vagy más néven a pontszám vagy előrejelzése), amely bemenetként a modell és a szolgáltatások igénybe, és az előrejelzési adja vissza. A logikai ezt követően a függvény, amely egy alkalmazás hívhatja közvetlenül, vagy egy webszolgáltatás elérhetővé tehető. 

Kapcsolódó Azure-szolgáltatások:

- [Azure Machine Learning](/azure/machine-learning/)
- [Machine Learning-kiszolgáló (az R Server), a HDInsight](/azure/hdinsight/r-server/r-server-overview)


## <a name="see-also"></a>Lásd még

- [A cognitive services technológia kiválasztása](../technology-choices/cognitive-services.md)
- [A gépi tanulási technológia kiválasztása](../technology-choices/data-science-and-machine-learning.md)
