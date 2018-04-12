---
title: Bővített analitika
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 31ba357fe37b1de35a6eea324d2d1d6766e172e5
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
---
# <a name="advanced-analytics"></a>Bővített analitika

Speciális elemzésekre túllép a korábbi jelentéskészítési és a hagyományos üzleti intelligenciával és matematikai, használja a probabilisztikus és statisztikai modellezési módszereket adatösszesítés prediktív feldolgozási és automatizált révén engedélyezése .

Speciális elemzési megoldások általában tartalmaz, amely az alábbi munkaterhelések:

* Adatok interaktív áttekintését és -megjelenítésre
* Gépi tanulási modell betanítási
* Valós idejű vagy prediktív kötegfeldolgozási

A legtöbb haladó analytics architektúrák vagy azok egy részét az alábbi összetevőket tartalmazza:

* **Adattárolás**. Speciális elemzési megoldásokat szükséges adatok machine learning modellek betanítása. Adatszakértőkön kell általában az adatokba a prediktív szolgáltatásai és a statisztikai kapcsolatát azokat az értékeket, akkor előre jelezni (más néven a címke). Az előre jelzett címke mennyiségi érték, például valamit a jövőben vagy repülési késleltetés időtartama percben pénzügyi értéke lehet. Akkor jelenthet kategorikus osztály, például a "true" vagy "false", "repülési késleltetés" vagy "repülési késleltetés", vagy kategóriák, például a "alacsony kockázat," vagy "közepes kockázat" vagy "magas kockázatú."

* **Kötegfeldolgozási**. A gépi tanulási modell betanításához általában kell feldolgozni a betanítási adatok nagy mennyiségű. A modell betanítása (terabájtok rendelése óra perc) hosszabb időt is igénybe vehet. Ez a képzés például Python vagy R nyelven íródott parancsfájlok használatával végezheti el, és elosztott feldolgozási platformokon, például az Apache Spark on HDInsight vagy egy Docker-tároló üzemeltetett használatával képzési idő csökkentése érdekében kiterjeszthető.

* **Valós idejű üzenet adatfeldolgozást**. Éles számos speciális elemzés valós idejű adatfolyamokat hírcsatorna a prediktív modell, amely egy webszolgáltatás közzé lett téve. A bejövő adatfolyam általában bekerül az várólista valamilyen és egy adatfolyam program lekéri az adatokat, ebből a várólistából, és az előrejelzés vonatkozik a bemeneti adatok közel valós időben.  

* **Az adatfolyam feldolgozása**. Ha elvégezte a modell betanítását, előrejelzés (vagy pontozási) általában egy nagyon gyorsan (terabájtok rendelése ezredmásodperc) egy adott objektumcsoporthoz funkcióját. Valós idejű üzenetek rögzítésével, miután a megfelelő funkció értékek átadhatók a prediktív szolgáltatás egy előre jelzett címke létrehozásához.

* **Analitikai adatokat tároló**. Bizonyos esetekben az előre jelzett címkeértéket az analitikus adatok áruházban reporting írt és jövőbeni elemzés.

* **Elemzési és jelentéskészítési**. Javasol a neve, mint a speciális elemzési megoldásokat általában eredményez valamilyen jelentés vagy analitikai adatcsatornára előre jelzett adatok értékeket tartalmaz. Gyakran előre jelzett címkeértéket feltöltésére használatos olyan valós idejű irányítópultokat.

* **Vezénylési**. Bár a kezdeti adatok feltárása és modellezési interaktív módon hajtja végre adatszakértőkön, számos speciális elemzési megoldásokat rendszeres időközönként új adatokkal újra betanítása modellek &mdash; folyamatosan szűkebbé tenni a modell pontosságát. A átképezési automatizálható az orchestrated munkafolyamat használatával.

## <a name="machine-learning"></a>Gépi tanulás
Machine learning a prediktív modell betanításához használandó matematikai modellezési technika. Az általános elvet statisztikai algoritmus előzményadatoknak tartalmaz a mezők közötti kapcsolatok felfedésére nagy adatkészletre alkalmazandó.

Machine learning modellezési általában adatszakértőkön, akik alaposan vizsgálatát, és készítse elő az adatok egy modell betanítása előtt végzi el. A kutatási funkciójával és előkészítése általában foglal magába interaktív adatelemzés és a képi megjelenítés nagy mennyiségű &mdash; általában használja például a Python és az R nyelv és interaktív eszközök, amelyeket kifejezetten ehhez a feladathoz.

Egyes esetekben esetleg használandó [modellek pretrained](/machine-learning-server/install/microsoftml-install-pretrained-models) , amely adatokat kapott, és a Microsoft által kifejlesztett betanítása kapható. A pretrained modellek előnye, hogy pontozása, és új tartalom besorolására azonnal, még akkor is, ha nem rendelkezik a szükséges betanítási adatok, az erőforrások kezelése nagy adatkészletek vagy összetett modellek betanítása.

Gépi tanulás széles két kategóriába sorolhatók:

* **Felügyelt tanítással**. Felügyelt tanítás során a leggyakrabban használt gépi tanulás által alkalmazott megközelítéshez. Egy felügyelt tanítás során a modellben az adatok olyan készlete áll *szolgáltatás* egy vagy több matematikai kapcsolatban álló adatmezők *címke* adatmezőket. Gépi tanulási folyamat képzési fázisában az adatkészlet funkciók és ismert címkék is tartalmaz, és algoritmus alkalmazott fér el a megfelelő címke előrejelzéseket kiszámításához szolgáltatásoktól függvénybe. Általában egy részét a képzés dataset tárolt vissza, és a betanított modell teljesítményétől érvényesítéséhez használt. Ha a modell még betanítva, azt is telepítése éles környezetben, és ismeretlen értékeinek használt. 

* **Felügyelt tanítást**. Az felügyeletlen tanulási modell a tanítási adatokat nem tartalmaz ismert címkeértéket. Ehelyett az algoritmus lehetővé teszi az előrejelzés első kitéve a adatok alapján. Az általános űrlap felügyeletlen tanulás *Fürtszolgáltatás*, ahol az algoritmus meghatározza, hogy a legjobb módszer az adatok felosztása a szolgáltatások statisztikai Hasonlóságok alapján egy megadott számát. A fürtszolgáltatás, előre jelzett eredménye a fürt számát, amely a bemeneti szolgáltatások tartoznak. Amíg néha felhasználásuk hasznos előrejelzéseket, például egy adatbázis több ügyfelek, a felhasználók azonosítására a Fürtszolgáltatás segítségével közvetlenül létrehozásához felügyeletlen tanulási megközelítések gyakrabban azonosítására szolgálnak mely adatokra leghasznosabb, ha szeretne biztosítani egy a modell betanítása a felügyelt tanulási algoritmus.

Kapcsolódó Azure-szolgáltatások:

- [Azure Machine Learning](/azure/machine-learning/)
- [Számítógép-kiszolgáló (K) a HDInsight tanulási](/azure/hdinsight/r-server/r-server-overview)

## <a name="deep-learning"></a>A részletes tanulás

Matematikai módszerek, például a lineáris vagy logisztikai regresszió alapuló gépi tanulási modellek egy kis ideig volt elérhető. Használatát újabban *mély tanulási* Neurális hálózatokat alapján technikák növekedett. Ennek célja a részben magas szinten méretezhető feldolgozási rendszerek, amelyekkel csökkenthető a mennyi időt vesz igénybe bonyolult modellek betanítása rendelkezésre állását. Is a nagyobb elterjedtségének a big data megkönnyíti a különböző tartományok mély tanulási modellek betanítása.

A speciális elemzésekre architektúra tervezésekor érdemes mély tanulási modellek nagyméretű feldolgozásához szükséges. Ezek a keresztül elosztott feldolgozási platformokon, például az Apache Spark és a legújabb generációját, virtuális gépek, amelyek tartalmazzák a GPU-hardveres eléréséhez megadható.

Kapcsolódó Azure-szolgáltatások:

- [A részletes tanulási virtuális gép](/azure/machine-learning/data-science-virtual-machine/deep-learning-dsvm-overview)
- [Az Apache Spark on hdinsight](/azure/hdinsight/spark/apache-spark-overview)

## <a name="artificial-intelligence"></a>Mesterséges intelligencia

Mesterséges intelligencia (AI) forgatókönyvekben, ahol a gép utánozza emberi minds, például a tanulási és problémák megoldására társított kognitív függvények hivatkozik. AI kihasználja a gépi tanulási algoritmusok, mert azt egy tartozó kifejezés tekinteni. A legtöbb AI megoldás prediktív szolgáltatások végrehajtása gyakran webszolgáltatások, valamint a természetes nyelvű felületek, például chatbots, amely kommunikálhatnak a továbbítókon keresztül szöveg vagy beszéd-, hogy a mobileszközökön és egyéb ügyfelek futtató AI alkalmazások jelennek kombinációja támaszkodnak. Bizonyos esetekben a gépi tanulási modell a AI alkalmazással van beágyazva. 

## <a name="model-deployment"></a>Telepítési modell

A prediktív AI alkalmazásokat támogató szolgáltatások egyéni machine learning modellek vagy off-the-shelf kognitív szolgáltatások pretrained modellek elérését lehetővé tevő előfordulhat, hogy kihasználja. Egyéni modellek éles környezetben üzembe helyezésének folyamata az úgynevezett operationalization, ahol az azonos AI modellek betanítása és a feldolgozási környezetben tesztelt szerializált és érhetik el a külső alkalmazások és szolgáltatások köteg vagy önkiszolgáló előrejelzéseket. A modell prediktív funkció azt deszerializálni van, és segítségével a machine learning tár, amely tartalmazza a modell betanításához az elsőként használt algoritmust. Ezt a szalagtárat prediktív funkciókat biztosít (gyakran nevezik pontszám vagy előrejelzése), amely a modell és szolgáltatások bemeneti adatként, és térjen vissza az előrejelzés. A működési elvet ezt követően az alkalmazás közvetlenül meghívhatja, vagy egy webszolgáltatás elérhetővé tehető függvényből. 

Kapcsolódó Azure-szolgáltatások:

- [Azure Machine Learning](/azure/machine-learning/)
- [Számítógép-kiszolgáló (K) a HDInsight tanulási](/azure/hdinsight/r-server/r-server-overview)


## <a name="see-also"></a>Lásd még

- [Egy kognitív technológia kiválasztása](../technology-choices/cognitive-services.md)
- [A gépi tanulási technológiával kiválasztása](../technology-choices/data-science-and-machine-learning.md)
