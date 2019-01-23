---
title: Interaktív adatfeltárás
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: d1912799c79271a585a941c91a2fa21c50787dcf
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482935"
---
# <a name="interactive-data-exploration"></a>Interaktív adatfeltárás

A sok vállalati üzleti intelligencia (BI-) megoldások, jelentések és szemantikai modellek BI szakemberei által létrehozott és központilag kezelhetők. Egyre azonban szervezetek szeretné engedélyezni az adatvezérelt döntések felhasználók. Ezenkívül egy egyre több szervezet is mindegyik *adatszakértők* vagy *adatelemzők*, amelynek feladata az adatok interaktív feltárása és statisztikai modelleket és analitikai módszerek a alkalmazni Keresse meg a trendek és mintázatok az adatok. Interaktív adatfeltárás szükséges eszközöket és platformokat, amelyek közel valós idejű feldolgozás az ad hoc lekérdezéseket és az adatok képi megjelenítésekhez.

![Interaktív adatfeltárás](./images/data-exploration.png)

## <a name="self-service-bi"></a>Self-service BI

Az önkiszolgáló BI egy egy adott üzleti döntések hatékonyságát, amelyben a felhasználók jogosultak keresése, ismerje meg és elemezheti az adatait megosztása a vállalaton belül modern megközelítését név. Ehhez a data-megoldás támogatnia kell több követelmény:

- A data catalog révén az üzleti adatforrások felderítését.
- Adatok entitás definíciók és-értékek konzisztencia biztosításához törzsadat-kezeléssel.
- Interaktív modellezési és vizualizációs eszközök, üzleti felhasználók számára.

Egy önkiszolgáló BI-megoldás, az üzleti felhasználók általában keresse meg és felhasználhat adatforrásokat, amelyek a saját üzleti meghatározott területét, és intuitív eszközöket és a hatékonyságnövelő alkalmazásokkal segítségével meghatározhatja a személyes adatok modelleket és jelentéseket, amelyek meg is oszthatják a munkatársaival.

Kapcsolódó Azure-szolgáltatások:

- [Az Azure Data Catalog](/azure/data-catalog/data-catalog-what-is-data-catalog)
- [Microsoft Power BI](https://powerbi.microsoft.com/)

## <a name="data-science-experimentation"></a>Data science kísérletezés

Amikor egy szervezet igényli a magas szintű elemzések és prediktív modellezés, a kezdeti előkészítési munkahelyi általában végzett szakértő adatszakértők. Adatszakértő felderíti az adatokat, és alkalmazza a statisztikai analitikai módszerek található adatok közötti kapcsolatok *funkciók* és a kívánt előre jelzett *címkék*. Adatfeltárás általában történik, amely natív módon támogatja a statisztikai modellezési és vizualizációs például Python vagy R programozási nyelvet használ. A parancsfájlok segítségével az adatok általában speciális környezetekben, például a Jupyter Notebooks üzemel. Ezek az eszközök az adatok programozott módon dokumentálásáért és megtalálják az elemzések megosztása során fedezheti fel az adatszakértők engedélyezése.

Kapcsolódó Azure-szolgáltatások:

- [Azure Notebooks](https://notebooks.azure.com/)
- [Az Azure Machine Learning Studióban](/azure/machine-learning/studio/what-is-ml-studio)
- [Az Azure Machine Learning-Kísérletezési szolgáltatás](/azure/machine-learning/preview/experimentation-service-configuration)
- [Az adatelemző virtuális gép](/azure/machine-learning/data-science-virtual-machine/overview)

## <a name="challenges"></a>Problémák

- **Adatok adatvédelmi megfelelőségi**. Legyen óvatos a személyes adatok elérhetővé tétele a felhasználók számára az önkiszolgáló elemzés és jelentéskészítés kell. Nincsenek valószínűleg megfelelőségi szempontok miatt a szervezeti szabályzatok, és szabályozási problémákat.

- **Adatmennyiség**. A teljes adatforráshoz való hozzáférés engedélyezése a felhasználók számára hasznos lehet, amíg azt eredményezhet nagyon hosszú ideig futó Excel- vagy Power BI-műveletek, vagy a Spark SQL jellemező a fürterőforrások lekérdezések.

- **Felhasználói Tudásbázis**. A felhasználók a saját lekérdezéseket és az aggregációhoz tájékoztatja az üzleti döntéseket hozhasson létre. Ön abban, hogy a felhasználók rendelkeznek a szükséges elemzési és a lekérdezés képességek a pontos eredmények eléréséhez?

- **Eredmények megosztása**. Előfordulhatnak biztonsági szempontok, ha a felhasználó létrehozhat és megoszthat jelentéseket és adatvizualizációkat.

## <a name="architecture"></a>Architektúra

Bár ebben a forgatókönyvben az a célja, hogy támogatja az interaktív adatelemzés, az adattisztító mintavétel, valamint elvégzendő feladatokat az adatok strukturálásához adatelemzési gyakran tartalmaz hosszú futású folyamatok. Így egy [kötegelt feldolgozás](../big-data/batch-processing.md) megfelelő architektúrát.

## <a name="technology-choices"></a>Technológiai lehetőségek

A következő technológiákat az Azure-ban interaktív adatfeltárás választási használata ajánlott.

### <a name="data-storage"></a>Adattárolás

- **Az Azure Storage-Blobtárolók** vagy **Azure Data Lake Store**. Az adatszakértők a nyers forrásadatok, annak érdekében, hogy ők is hozzáférhetnek az összes lehetséges szolgáltatások, a kiugró értékek és a hibák az adatok általában együttműködve. A big Data típusú adatok esetben ezeket az adatokat általában eltart fájlok formájában a tárolóban.

További információkért lásd: [adattárolás](../technology-choices/data-storage.md).

### <a name="batch-processing"></a>Kötegelt feldolgozás

- **Az R Server** vagy **Spark**. Az adatszakértők a legtöbb programozási nyelvek erős támogató használata matematikai és statisztikai csomagok, például az R vagy Python. Ha nagy mennyiségű adatot dolgozik, csökkentheti a késést platformokat, amelyek lehetővé teszik az elosztott feldolgozási használandó nyelvek használatával. Az R Server használható önállóan, vagy a Spark együtt horizontális felskálázási R feldolgozási funkciók, és a Spark natív módon támogatja a Python a hasonló horizontális felskálázhatóságát azon a nyelven.
- **Hive-**. Hive a megfelelő választás az olyan SQL-szerű szemantika használatával az adatok átalakítása. Felhasználók létrehozása és a HiveQL utasításokkal, amelyek szemantikailag hasonlóak az SQL-táblák betöltése.

További információkért lásd: [kötegelt feldolgozás](../technology-choices/batch-processing.md).

### <a name="analytical-data-store"></a>Analitikus adatok Store

- **A Spark SQL**. Spark SQL API-k, Spark, amely támogatja a dataframes és táblákat, amelyek az SQL-szintaxissal lekérdezhetők épülő. Függetlenül, hogy az elemezni kívánt adatfájlokat nyers forrásfájlok vagy új fájlokat, amelyek tisztítás és a egy kötegelt folyamat által készített meghatározhatják a Spark SQL-táblák rajtuk a további lekérdezéséhez az elemzés.

- **Hive-**. Mellett a batch a nyers adatok feldolgozása a Hive használatával egy Hive-adatbázis, amely tartalmazza a Hive-táblák és nézetek alapján az adatokat tároló mappák engedélyezése elemzéshez interaktív lekérdezések és a jelentéskészítés is létrehozhat. HDInsight magában foglalja az interaktív Hive-fürt típusa, amely Hive-lekérdezés válaszidők csökkentése érdekében használja a memórián belüli gyorsítótárazáshoz. Felhasználók, akik jól ismerik az SQL-szerű szintaxist használatával interaktív Hive-adatok megismerése.

További információkért lásd: [analitikus adattárak](../technology-choices/analytical-data-stores.md).

### <a name="analytics-and-reporting"></a>Elemzések és jelentéskészítés

- **Jupyter**. A Jupyter Notebooks böngészőalapú felületet biztosít a futó kód például az R, Python vagy a Scala nyelvek. Adatok kötegelt feldolgozásához R Server vagy a Spark használata esetén, vagy a táblák lekérdezése a sémát a Spark SQL használatával, a Jupyter lehet megfelelő választás az olyan, az adatok lekérdezéséhez. A Spark használata esetén a szabványos Spark dataframe API vagy a Spark SQL API-t, valamint beágyazott SQL-utasítások használhatja az adatok lekérdezése és állítja elő a Vizualizációk.

- **Részletezés**. Ha szeretne végezni az ad-hoc adatfeltárás, [Apache Drill](https://drill.apache.org/) egy séma nélküli SQL-lekérdezési motor. A séma nincs szükség, mert a számos különféle adatforrásból származó adatokat lekérdezheti, és a motor automatikusan tisztában van az adatok struktúráját.  Az Azure Blob Storage-tárolókat, részletezés is használhatja a [Azure Blob Storage beépülő modul](https://drill.apache.org/docs/azure-blob-storage-plugin/). Ez lehetővé teszi a Blob Storage szolgáltatásban tárolt adatok irányuló lekérdezések futtatása az adatok áthelyezése nélkül.

- **Interaktív Hive-ügyfelek**. Ha egy interaktív Hive-fürtön az adatok lekérdezéséhez, használhatja a Hive-nézet az Ambari fürt irányítópultja, a Beeline parancssori eszköz vagy egy ODBC-alapú eszközt (a Hive ODBC illesztőprogram segítségével), például a Microsoft Excel- vagy Power bi-ban.

További információkért lásd: [adatelemzési és jelentéskészítési technológia](../technology-choices/analysis-visualizations-reporting.md).
