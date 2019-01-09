---
title: Idősorozat-adatok
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 80dc797e6df16bff73fb9c5116ccce34d7d846f0
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111937"
---
# <a name="time-series-solutions"></a>Idősorozattal kapcsolatos megoldások

Idősorozat-adatok az idő szerint rendezve értékek egy halmazát. Idősorozat-adatok belefoglalása érzékelőadatokat, tőzsdei árfolyamok idő kattintson a streamadatokat, és Alkalmazáshasználati. Idősorozat-adatok elemezhetők a korábbi trendek, valós idejű riasztások és prediktív modellezés céljára.

![Time Series Insights](./images/time-series-insights.png)

Az idősorozat-adatok mutatják be, hogy hogyan változik az idők során egy adategység vagy folyamat. Az adatok rendelkezik időbélyeget, de ami még fontosabb, ideje a legtöbb jelentéssel bíró tengely megjelenítése vagy az adatok elemzése. Idősorozat-adatok általában időrendben érkezik, és általában az adatbázis-frissítés helyett egy insert számít. Emiatt a változás az idő múlásával mérése lehetővé téve keresse ki az előző verziókkal való és előre jelezni a jövőbeli módosítása. Idősorozat-adatok a pontdiagram- vagy vonaldiagramon diagramok, legjobb vizualizálja.

![Vonaldiagram vizualizálja idősorozat-adatok](./images/time-series-chart.png)

Néhány példa idősorozat-adatok a következők:

- Megfigyelheti a szabályszerűségeket az idő múlásával rögzíti a tőzsdei árfolyamok.
- Kiszolgáló teljesítményt, például a CPU kihasználtsága, i/o terhelés, memóriahasználat és hálózati sávszélesség-használatot.
- Ipari berendezés, amely függőben lévő berendezések meghibásodása és az eseményindító riasztási értesítések észleléséhez használható érzékelő telemetriáját.
- Valós idejű autó telemetriai adatokat, többek között a sebesség, fékezés és gyorsítás időkeretben előállításához egy összesített kockázati pontszám az illesztőprogramhoz.

Az egyes ezekben az esetekben láthatja, hogyan ideje egy tengelyként lényeges. Kulcsfontosságú jellemzője az idősorozat-adatok, az események megjelenítése abban a sorrendben, amelyben érkeztek nem, mert van egy természetes historikus rendezése. Ez eltér a normál OLTP adatfolyamatokat állíthat össze az adatokat is lehet a megadott bármilyen sorrendben, és bármikor frissíteni rögzített adatokat.

## <a name="when-to-use-this-solution"></a>Ez a megoldás használata

Válassza ki egy time series megoldás kell kiolvasni az adatokat, amelyek stratégiai értéke egy időszakon belül módosítások köré szerveződik, és Ön elsősorban az új adatok beszúrása, és csak ritkán frissítése, ha egyáltalán. Ez az információ segítségével rendellenességek észlelése, a trendek képi megjelenítését, és összehasonlíthatja az aktuális adatokat előzményadatok, többek között. Az ilyen típusú architektúra azért is prediktív modellezés és előrejelzési eredményeivel, a legmegfelelőbb változások Előzményrekord alkalmazhatók bármely előrejelzési modellek száma idővel rendelkezik.

Időbeli adatsorok használatával az alábbi előnyökkel jár:

- Egyértelműen jelöli, hogy egy eszköz vagy folyamat változik az idő múlásával.
- Segítségével gyorsan észlelheti a kapcsolódó források értékre változik, így a rendellenességeket és újonnan megjelenő trendeket egyértelműen kitűnjön.
- A legalkalmasabb prediktív modellezési és előrejelzését.

### <a name="internet-of-things-iot"></a>Eszközök internetes hálózata (IoT)

IoT-eszközök által gyűjtött adatok a természetesen illeszkednek a time series tárolási és elemzése. A bejövő adatok beszúrásakor és ritkán, ha minden eddiginél frissítve. Az adatok az idő megjelölve, és beilleszti érkezett a sorrendben, és ezeket az adatokat általában trendek, azonosíthatja a rendellenességeket, és használja a információkat prediktív elemzési felhasználók időrendi sorrendben jelennek meg.

További információkért lásd: [IOT-](../big-data/index.md#internet-of-things-iot).

### <a name="real-time-analytics"></a>Valós idejű elemzések

Idősorozat-adatok gyakran az idő-és nagybetűket &mdash; , vagyis azt kell lennie, amelyre a gyors, valós idejű felhasználva azonosíthatja a trendeket, vagy riasztást. Ezekben az esetekben insights az késleltetések okozhat állásidő és üzleti hatás. Emellett gyakran szükség van egy számos különböző forrásokból, például érzékelőktől származó adatok korrelációját.

Ideális esetben egy adatfolyam-feldolgozási réteg, amely képes kezelni a bejövő adatokat valós időben, és feldolgozza az összes, a nagy pontosságú és nagy pontossággal kell. Ez nem mindig lehetséges, a streamelési architektúra és a streampufferelésnek és adatfolyam-feldolgozás rétegek összetevői függően. Szükség lehet néhány az idősorozat-adatok pontosságát lecsökken csökkentésével azt. Ehhez feldolgozása a késleltetett idő windows (néhány másodpercig, a példában), lehetővé téve a feldolgozási réteg időben számítások végrehajtásához. Előfordulhat, hogy felbontásának kell, és az adatok összesítésének hosszabb ideig, például a több hónapon keresztül rögzített adatok megjelenítéséhez nagyításához megjelenítésekor.

## <a name="challenges"></a>Problémák

- Idősorozat-adatok gyakran rendkívül nagy mennyiségű, különösen az IoT-forgatókönyveket. Tárolására, az indexelés, lekérdezés, elemzése és jelenítenek idősorozat-adatok kihívást jelenthet.

- Ez kihívást jelenthet, keresse meg a nagy sebességű tárolási kombinációjával és hatékony kezelésére valós idejű elemzési, ugyanakkor minimalizálja a piacra és a teljes idő műveletek beruházási költség.

## <a name="architecture"></a>Architektúra

Az idősorozat-adatok, például az IoT-érintő számos forgatókönyv az adatok rögzített valós időben. Ennek megfelelően egy [valós idejű feldolgozás](../big-data/real-time-processing.md) architektúra alkalmas.

Egy vagy több adatforrásokból származó adatok betöltődnek az adatfolyam-réteg által pufferelés [az IoT Hub](/azure/iot-hub/), [az Event Hubs](/azure/event-hubs/), vagy [a HDInsight alatt futó Kafka](/azure/hdinsight/kafka/apache-kafka-introduction). Ezután az adatok feldolgozása, amelyek igény szerint is kiosztják a feldolgozott adatokat egy gépi tanulási prediktív elemzési szolgáltatás feldolgozási streamrétegében. A feldolgozott adatok tárolva van egy analitikai adattár, például [HBase](/azure/hdinsight/hbase/apache-hbase-overview), [Azure Cosmos DB](/azure/cosmos-db/), az Azure Data Lake, vagy a Blob Storage. Az elemzések és jelentéskészítés az alkalmazás vagy szolgáltatás, mint a Power bi-ban vagy az OpenTSDB, (ha az a HBase tárolja) az elemzéshez idősoros adatainak megjelenítéséhez használható.

Egy másik lehetőség [Azure Time Series Insights](/azure/time-series-insights/). A Time Series Insights egy teljes körűen felügyelt szolgáltatás, az idősorozat-adatok. Ebben az architektúrában a Time Series Insights a szerepkörök adatfolyam-feldolgozás, adattár, és az elemzési és jelentéskészítési hajt végre. Streamelési adatok az IoT Hub vagy az Event Hubs, és tárolja, a folyamatokat, elemzi, és közel valós időben jeleníti meg az adatokat fogad. Előre nem összesíti az adatokat, de a nyers események tárolja.

A Time Series Insights séma, az adaptív, ami azt jelenti, hogy nem kell tennie minden olyan adat-előkészítés indítása a Származtatás insights. Ez lehetővé teszi, hogy ismerje meg, összehasonlítása és a egy számos különféle adatforrásból zökkenőmentes összekapcsolását. SQL-szerű szűrőket is biztosít, és összesítések, képes létrehozni, megjelenítése, hasonlítsa össze, és átfedő különböző time series minták, fűtés térképek és mentése és megosztása a lekérdezések lehetővé teszi.

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Adattárolás](../technology-choices/data-storage.md)
- [Elemzés, a Vizualizációk és a jelentéskészítés](../technology-choices/analysis-visualizations-reporting.md)
- [Analitikus adattárak](../technology-choices/analytical-data-stores.md)
- [Az adatfolyam feldolgozása](../technology-choices/stream-processing.md)