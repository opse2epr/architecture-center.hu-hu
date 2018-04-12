---
title: Interaktív adatfeltárás
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 20740a8fe912a63526c847416b832941f4ac33ec
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
---
# <a name="interactive-data-exploration"></a>Interaktív adatfeltárás

A sok vállalati üzleti intelligencia (BI) megoldások, jelentések és szemantikai modellek létrehozása BI szakemberei és központilag kezelhetők. Egyre azonban a szervezetek szeretné engedélyezni az adatvezérelt döntések felhasználók. Emellett a szervezetek egyre több bérbeadása vannak *adatszakértőkön* vagy *adatelemzők*, feladatokkal, hogy adatokba interaktív módon, és alkalmazza a statisztikai modellek és analitikai technikák a trendek és mintázatok található adatokat. Adatok interaktív áttekintését szükséges eszközök, és adja meg a kis késleltetésű feldolgozása ad hoc lekérdezéseket és az adatok vizuális-platformokat.

![](./images/data-exploration.png)

## <a name="self-service-bi"></a>Self-service BI

Az önkiszolgáló üzleti Intelligencia egy adott üzleti révén, amelyben a felhasználók jogosultak keresése, vizsgálatát, és megoszthatja az információkat kaphat a adatok a vállalati modern megközelítése név. Ennek érdekében az adatok megoldásnak támogatnia számos követelményt:

* Az üzleti adatforrások keresztül egy adatkatalógus felderítése.
* Törzsadatokat felügyeleti adatok entitás definíciók és értékek konzisztencia biztosításához.
* Interaktív adatok modellezési és a képi megjelenítés eszközök üzleti felhasználók számára.

A BI önkiszolgáló megoldás üzleti felhasználók általában, és felhasználhatják az adatforrások, amely kapcsolódik az adott területre az üzleti, intuitív eszközök és az alkalmazást személyes adatok modellek és megoszthatják-jelentések a munkatársaival.

Kapcsolódó Azure-szolgáltatások:

- [Az Azure Data Catalog](/azure/data-catalog/data-catalog-what-is-data-catalog)
- [Microsoft Power BI](https://powerbi.microsoft.com/)

## <a name="data-science-experimentation"></a>Adatok tudományos kísérletezés
Amikor egy szervezet megköveteli a speciális elemzés és prediktív modellezési, akkor a kezdeti előkészítéséhez munkahelyi általában specialistája adatszakértőkön végzi. Egy adatok tudósok felderíti az adatokat, és alkalmazza a statisztikai analitikai módszerek található adatok közötti kapcsolatok *szolgáltatások* és a kívánt előre jelezni *címkék*. Az adatok feltárása szokás programozási nyelvek például Python vagy R statisztikai modellezési és a képi megjelenítés natív módon támogató használatával. A parancsfájlok segítségével feltérképezheti az adatokat általában speciális környezetekben, például Jupyter notebookok üzemelteti. Ezek az eszközök engedélyezése adatszakértőkön ásna az adatokban programozott módon dokumentálásáért és találnak feltárása megosztása során.

Kapcsolódó Azure-szolgáltatások:

- [Azure Notebooks](https://notebooks.azure.com/)
- [Az Azure Machine Learning Studióban](/azure/machine-learning/studio/what-is-ml-studio)
- [Az Azure gépi tanulás kísérletezhet szolgáltatások](/azure/machine-learning/preview/experimentation-service-configuration)
- [Az adatok tudományos virtuális gép](/azure/machine-learning/data-science-virtual-machine/overview)

## <a name="challenges"></a>Problémák

- **Adatvédelmi megfelelőségi adatokat.** Személyes adatok elérhetővé tétele a felhasználók számára az önkiszolgáló elemzési és jelentéskészítési körültekintően kell. Nincsenek valószínűleg megfelelőségi szempontok miatt szervezeti házirendeket, és kapcsolatos szabályozási problémákat. 

- **Adatmennyiség.** Az teljes adatforráshoz való hozzáférés engedélyezése a felhasználók számára hasznos lehet, amíg eredményezhet nagyon hosszú futású Excel vagy a Power BI műveleteket vagy a Spark SQL lekérdezések, amelyek nagy mennyiségű fürt erőforrásait.

- **Felhasználói ismerete.** Felhasználók a saját lekérdezések és az aggregációhoz hozzon létre, és értesíti az üzleti döntéseket hozhat. Biztos benne, hogy a felhasználó rendelkezik a szükséges analitikai és lekérdező képességek pontos eredményekhez juthat?

- **Eredmények megosztása.** Előfordulhatnak biztonsági szempontok, ha a felhasználók létrehozhat és megoszthat a jelentéseket vagy adatokat képi megjelenítések.

## <a name="architecture"></a>Architektúra

Bár ebben a forgatókönyvben az a célja, hogy interaktív adatelemzés támogatja, a tisztítás, mintavételi, és a feladatok elvégzésében adatok szerkezetének kialakítása tudományos gyakran tartalmaznia hosszú futású folyamatokat. Ennek köszönhetően a [kötegfeldolgozási](../big-data/batch-processing.md) architektúrájának megfelelő.

## <a name="technology-choices"></a>Technológiai lehetőségek

A következő technológiákat kell az Azure-ban adatok interaktív áttekintését használata ajánlott.

### <a name="data-storage"></a>Adattárolás

- **Az Azure Storage-Blob tárolók** vagy **az Azure Data Lake Store**. Adatszakértőkön általában együttműködve nyers forrásadatok férhet hozzá, az összes lehetséges szolgáltatások, a kiugró és a hibák az adatok biztosításához. Big Data típusú adatok esetén ezeket az adatokat általában formájában történik a fájlok a tárolóban.

További információkért lásd: [adattárolás](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Kötegelt feldolgozás

- **R Server** vagy **Spark**. A legtöbb adatszakértőkön programozási nyelvet használják erős-támogatással rendelkező matematikai és statisztikai csomagok, például az R vagy Python. Ha nagy mennyiségű adatot, platformokat, amelyek lehetővé teszik az ezeken a nyelveken elosztott feldolgozási használandó használatával csökkentése érdekében késés. R Server használható önállóan, vagy a Spark együtt kiterjesztése az R feldolgozási funkciók, és Spark natív módon támogatja a Python hasonló kibővített képességeinek az adott nyelveken.
- **Hive**. Hive jó választás az SQL-szerű szemantika használatával adatok átalakítása. Felhasználók létrehozása és használati HiveQL utasításokat, amelyek szemantikai szempontból hasonlóak SQL táblák betölteni.

További információkért lásd: [kötegfeldolgozási](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Analitikai adattár

- **Spark SQL**. Spark SQL az API-k, amely támogatja a létrehozása dataframes és -táblázatot, amely az SQL-szintaxis használatával lehet lekérdezni a Spark épül. Függetlenül, hogy az elemezni kívánt adatfájlok nyers forrásfájlok vagy tisztítás és a batch-folyamat által készített új fájlokat meghatározhatják a Spark SQL-táblák rajtuk további lekérdezése az elemzést. 
- **Hive**. Mellett kötegelt nyers adatok feldolgozása a Hive eszközzel a Hive-adatbázist, amelyben a Hive táblák és nézetek alapján az adatokat tároló mappák elemzéshez interaktív lekérdezések engedélyezése és a jelentéskészítés is létrehozhat. HDInsight tartalmaz egy interaktív Hive fürt által használt típusa a memóriában történő gyorsítótárazás Hive lekérdezés a válaszhoz szükséges idő csökkentése érdekében. Felhasználók, akik SQL-szerű szintaxis a Feladatkezelő segítségével interaktív Hive adatokba.

További információkért lásd: [analitikai adattárolókhoz](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Elemzések és jelentéskészítés

- **Jupyter**. Jupyter notebookok webböngésző-alapú felületet biztosít például R, Python vagy Scala nyelvű kód futtatásával. R Server vagy a Spark kötegelt adatfeldolgozásra történő használata esetén, vagy ha a Spark SQL használatával definiálhat egy sémát táblák lekérdezése, Jupyter lehet az adatok lekérdezése érdemes választani. Ha Spark, a standard előidézett dataframe API vagy a Spark SQL API-t, valamint beágyazott SQL-utasítások használhatja a lekérdezést, valamint képi megjelenítéseket készíthet.
- **Interaktív Hive ügyfelek**. Ha a lekérdezést egy interaktív Hive-fürtöt használ, a Hive nézete az Ambari fürt irányítópultot, az Beeline parancssori eszköz vagy bármely ODBC-alapú eszköz (a Hive ODBC-illesztőprogram használatával), például a Microsoft Excel alkalmazást vagy a Power BI is használhatja.

További információkért lásd: [adatelemzés és jelentéskészítési technológia](../technology-choices/analysis-visualizations-reporting.md).