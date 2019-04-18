---
title: Teljesítménnyel kapcsolatos hibaelhárítás az Azure Databricksbe az Azure Monitor használatával
titleSuffix: ''
description: Grafana irányítópultok használatával kapcsolatos problémák elhárítása az Azure Databricksben
author: petertaylor9999
ms.date: 04/02/2019
ms.topic: ''
ms.service: ''
ms.subservice: ''
ms.openlocfilehash: 49ec63d0c45ab388ca83b3ab0562428327539619
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740374"
---
# <a name="troubleshoot-performance-bottlenecks-in-azure-databricks"></a>Az Azure Databricksben a teljesítmény szűk hibaelhárítása

Ez a cikk ismerteti a figyelési irányítópult használata a teljesítmény szűk megkereséséhez az Azure Databricks Spark-feladatok.

[Az Azure Databricks](/azure/azure-databricks/) van egy [Apache Spark](https://spark.apache.org/)– elemzési szolgáltatás, amely megkönnyíti az gyors fejlesztése és üzembe helyezése a big data koncepción alapuló. Megfigyelés és hibaelhárítás a teljesítménnyel kapcsolatos problémák esetén egy kritikus fontosságú Azure Databricks éles üzemi. Gyakori teljesítményproblémák azonosításához, hasznos lehet a telemetriai adatok alapján figyelési Vizualizációk használatára.

## <a name="prerequisites"></a>Előfeltételek

Ebben a cikkben látható a Grafana az irányítópultok beállításához:

- Konfigurálja a Databricks-fürtön, hogy telemetriát küldjön egy Log Analytics-munkaterületet, az Azure Databricks-figyelő könyvtár használatával. További információkért lásd: [konfigurálása az Azure Databricks mérőszámok küldése az Azure monitornak](./configure-cluster.md).

- Egy virtuális gépet üzembe helyezni a Grafana. Lásd: [irányítópultok használatával jelenítheti meg az Azure Databricks-metrikák](./dashboards.md).

Az üzembe helyezett lesz a Grafana irányítópultja egy idősorozat-vizualizációkat tartalmaz. Az egyes egy Apache Spark-feladat kapcsolatos metrikákat a feladatokat és a tevékenységeket, minden egyes szakaszhoz alkotó fázisa az idősorozat-diagram.

## <a name="azure-databricks-performance-overview"></a>Az Azure Databricks teljesítmény áttekintése

Az Azure Databricks az Apache Spark egy általános célú elosztott számítási rendszer alapul. Alkalmazáskód, más néven egy **feladat**, végrehajtja az Apache Spark-fürt, a kezelő feladata. Egy feladat általában a legmagasabb szintű számítási egységek. Egy feladat jelöli a Spark-alkalmazás által végrehajtott művelet befejezéséhez. Végrehajtott egy szokásos műveletet tartalmaz az adatok olvasása a forrásból, adatátalakítások alkalmazása és az eredmények olvasást a tárolón vagy a másik célhelyet.

Feladatok részletezhetők **szakaszok**. A feladat aknázhatja nyújtotta fejlett lehetőségeket a szakaszok keresztül egymás után, ami azt jelenti, hogy a későbbi szakaszokra kell várnia a korábbi fázisokban végrehajtásához. Szakaszok tartalmazzák az azonos csoportok **feladatok** , amely a Spark-fürtben több csomóponton párhuzamosan hajtható végre. Feladatok olyan végrehajtási zajló az adatok egy részét a legrészletesebb egysége.

A következő szakaszokban néhány irányítópult-Vizualizációk, amelyek hasznosak a teljesítménnyel kapcsolatos hibaelhárítás.

## <a name="job-and-stage-latency"></a>Feladat vagy előkészítési fázisban késés

Feldolgozás késéssel az az időtartam, a feladat végrehajtása a, amíg be nem fejezi indításakor. Akkor jelenik meg. percentilisei / fürt és -Azonosítóját, a feladat végrehajtása, hogy a Vizualizáció a kiugró értékek. A következő diagram bemutatja a Feladatelőzmény egy feladat, a 90. százalékértékre elérte a 50 másodperc, annak ellenére, hogy az 50. percentilis következetesen körülbelül 10 másodpercig volt.

![Feldolgozás késéssel ábrázoló grafikon](./_images/grafana-job-latency.png)

Keresse meg a feladat végrehajtása a fürt és az alkalmazások, a késés az adatforgalmi csúcsokhoz keres. Fürtökkel és alkalmazásokkal rendelkező nagy késleltetésű azonosítás után folytassa a szakasz késés vizsgálatára.

Fázis késés is jelenik meg. percentilisei, hogy a Vizualizáció a kiugró értékeket. Fázis várakozási ideje szerint a fürt, az alkalmazás és a fázis neve van szétbontva. A meghatározásához, hogy milyen feladatokat vissza a fázis befejezése után tárolnak a grafikon a feladat késés azonosítsa az adatforgalmi csúcsokhoz.

![A grafikont fázis késés](./_images/grafana-stage-latency.png)

A fürt átviteli graph feladatok szakaszok és percenként elvégzett feladatok számát jeleníti meg. Ez segít, hogy jobban megismerhesse a munkaterhelés-szakaszok és műveletek feladatonként relatív számának tekintetében. Itt láthatja, hogy a feladatok percenkénti számát címtartományok 2. és 6 közötti szintjeinek száma pedig körülbelül 12 &ndash; 24 / perc.

![Graph megjelenítő fürt átviteli sebesség](./_images/grafana-cluster-throughput.png)

## <a name="sum-of-task-execution-latency"></a>A feladat-végrehajtási késés összege

Ezt a vizualizációt gazdagépenként egy fürtön futó feladat végrehajtási késés az összegét mutatja. Futó feladatok lassan lelassítja a gazdagép miatt a fürt vagy a feladatokat végrehajtó kiszolgálónként képtelenségében észleléséhez használni ehhez a diagramhoz. Az alábbi diagramon az állomások a legtöbb rendelkezik körülbelül 30 másodpercet összege. Azonban két gazdagép rendelkezik, amely körülbelül 10 percet vigye összegeket. A gazdagépek lassan futnak, vagy a végrehajtó / feladatok számát misallocated van.

![A feladat a végrehajtás gazdagépenként Graph ábrázoló összege](./_images/grafana-sum-task-exec.png)

A végrehajtó / feladatok számát mutatja, hogy két végrehajtóval rendelt feladatok, aránytalanul nagy számú szűk keresztmetszetet okoz.

![A grafikont feladatokat végrehajtó kiszolgálónként](./_images/grafana-tasks-per-exec.png)

## <a name="task-metrics-per-stage"></a>A feladat metrika. fázis

A feladat metrikák Vizualizáció egy feladat a végrehajtás megtalálható a költségek részletezése. Ezzel a relatív időt töltött lásd a feladatokat, köztük a szerializálást és deszerializálást. Ezek az adatok optimalizálása a lehetőségek előfordulhat, hogy megjelenítése &mdash; használatával például [változók szórási](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#broadcast-variables) szállítási adatok elkerülése érdekében. A feladat metrikák is megjelenítheti az shuffle mérete egy feladat, és a shuffle írási és olvasási alkalommal. Ha ezeket az értékeket magas, azt jelenti, hogy nagy mennyiségű adatot helyez át a hálózaton keresztül.

Egy másik feladat metrika a scheduler késleltetés, amely méri, hogy mennyi ideig tart a feladat ütemezését. Ideális esetben ez az érték legyen, valójában a feladat végrehajtása végrehajtó számítási időért képest, amely az az idő töltött.

Az alábbi grafikonon látható scheduler késleltetési idő (3.7 s), amely meghaladja a végrehajtó számítási idő (1.1-es s). Ez azt jelenti, hogy több időt vesz igénybe, mint a tényleges munkát végző ütemezett tevékenységek.

![Grafikont a feladat metrika. fázis](./_images/grafana-metrics-per-stage.png)

Ebben az esetben a probléma okozta túl sok partíció, amely számos olyan ráfordításokat okozta. A scheduler késleltetési időt a partíciók számának csökkentését csökken. A következő diagram azt mutatja be, hogy a legtöbbször a rendszer tölt a feladat végrehajtása.

![Grafikont a feladat metrika. fázis](./_images/grafana-metrics-per-stage2.png)

## <a name="streaming-throughput-and-latency"></a>Folyamatos átviteli teljesítménye és adatelérési sebessége

A strukturált streamelésbe közvetlenül kapcsolódik a folyamatos átviteli sebességet. Nincsenek társított streamelés sebességének két fontos metrikákat: A bemeneti sorok száma másodpercenként a második és a feldolgozott sorokat. Ha másodpercenkénti bemeneti sorok outpaces másodpercenként feldolgozott sorok, az azt jelenti, a streamfeldolgozó rendszer elmaradásban van. Is ha a bemeneti adatokat az Event Hubs és a Kafka származik, majd másodpercenkénti bemeneti sorok kell tartani az előtér adatok feldolgozási sebességét.

Két feladatot hasonló fürt átviteli sebesség, de különböző streamelési metrikák is rendelkezhet. Az alábbi képernyőfelvételen két különböző számítási feladatokhoz. Azok a hasonló tekintetében fürt átviteli sebesség (feladatok, szakaszok és feladatok / perc). De a második Futtatás dolgozza fel a 12 000 sor/mp és 4000 sor/mp-ben.

![Streamelés sebességének Graph-megjelenítő](./_images/grafana-streaming-throughput.png)

Streamelési átviteli sebesség gyakran egy jobb üzleti mérőszám, mint a fürt átviteli sebesség, mert azt feldolgozott adatfelderítési rekordok számát méri.

## <a name="resource-consumption-per-executor"></a>Erőforrás-használatot végrehajtó

A metrikák segítenek megérteni a munkát, amely minden egyes végrehajtó hajt végre.

**Százalékos metrikák** mérheti, hogy mennyi idő végrehajtó foglalkozik a különböző dolgokat kifejezve egy általános végrehajtó számítási idő töltött idő aránya. A mérőszámok a következők:

- A(z) % szerializálni idő
- A(z) % deszerializálási ideje
- % CPU-végrehajtó idő
- Idő %-ban JVM

Ezek a Vizualizációk megjelenítése mennyi egyes metrikák hozzájárul általános végrehajtó feldolgozása.

![A grafikont százalékos metrikák](./_images/grafana-percentage.png)

**Metrikák shuffle** metrikák kapcsolódó adatok újbóli felosztás a végrehajtóval között.

- Shuffle i/o
- Shuffle memória
- Fájlrendszerhasználat
- Lemezhasználat

## <a name="common-performance-bottlenecks"></a>Gyakran a teljesítmény szűk

Két gyakori teljesítmény szűk keresztmetszeteinek megszüntetéséhez Spark vannak *elvetésének feladat* és a egy *partíciók száma nem optimális shuffle*.

### <a name="task-stragglers"></a>A feladat elvetésének

Egy feladat szakaszai megadott sorrendben hajtja végre blokkolja a későbbi szakaszokra korábbi szakaszaiban. Egy tevékenység más feladatok lassabban végrehajtja a shuffle partíció, ha a fürt összes feladatot a lassú feladat fejezheti be a fázis előtt olvasásra kell várni. Ez akkor fordulhat elő, a következő okok miatt:

1. Egy gazdagép vagy gazdagépek nincs lassan futnak. Tünetek: Magas feladat, a szakaszban vagy feldolgozás késéssel és a fürt alacsony átviteli sebesség. A feladatok késések gazdagépenként háttéradatok nem lehet egyenletesen oszlik el. Azonban erőforrás-használat egyenletesen között szétosztani végrehajtóval.

1. Feladatok végrehajtásához (adatok eltorzítják) költséges összesítést rendelkeznek. Tünetek: Magas feladat, a fázis, vagy a feladat és a fürt alacsony átviteli sebesség, de az késleltetések gazdagépenként háttéradatok egyenletesen vannak elosztva. Erőforrás-használat végrehajtóval között egyenlően szétosztani.

1. Ha a partíciók nem egyenlő méretű, egy nagyobb partíció egyenetlen feladat a végrehajtás (partíció eltorzítják) vezethet. Tünetek: Erőforrás-használat végrehajtó túl magas a fürtben futó más végrehajtóval képest. A végrehajtó futó összes feladat lassú végrehajtását, és a szakasz végrehajtása tartsa a folyamat. Szakaszokban vannak szólt kell *korlátok előkészítése*.

### <a name="non-optimal-shuffle-partition-count"></a>A nem optimális shuffle partíciók száma

Strukturált streamelési lekérdezés alatt egy tevékenység-végrehajtó történő, hozzárendelését egy erőforrás-igényes művelet, a fürt számára. A shuffle adatok nem optimális mérete, ha a feladat késleltetése mennyisége negatív hatással átviteli sebességgel és késéssel. Ha túl kevés partíciókat, a Processzormagok száma a fürtben fog kell eredményeztek ennek eredményeként elégtelenségekkel feldolgozása. Ezzel szemben ha túl sok partíció van, nincs terhelés kevés számú feladat-kezelés nagy fokú.

Az erőforrás-felhasználási mérőszámok használatával a partíció eltorzítják és végrehajtóval a fürtön, képtelenségében hibaelhárítása. Ha egy partíció torzítja van, akkor végrehajtó erőforrásokat a fürtben futó más végrehajtóval támogatáshoz képest korlátozottabb emelt szintű.

Például a következő diagram azt mutatja, hogy az első két végrehajtóval az újbóli felosztás által felhasznált memória X nagyobb, mint a többi végrehajtóval 90:

![A grafikont százalékos metrikák](./_images/grafana-shuffle-memory.png)