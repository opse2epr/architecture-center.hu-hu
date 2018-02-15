---
title: "Adatsorozat időadatok"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ceb8f34d4fd950e5270edfea05945a824c4492f0
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="time-series-solutions"></a>Idő adatsorozat megoldások

Adatsorozat időadatok olyan értékek idő szerint vannak rendezve. Idő adatsorozat közé tartoznak az érzékelő adatokat, a rendszer a díjak, kattintson a adatfolyam adatok és telemetriai. Idő adatsorozat adatok elemzése a múltbeli trendek, a valós idejű riasztásokat vagy a prediktív modellezési.

![Time Series Insights](./images/time-series-insights.png) 

Adatsorozat időadatok jelöli, hogy egy eszköz vagy folyamat idővel hogyan változik. Az adatok egy Timestamp típusú, de ennél is fontosabb, idő megjelenítése, vagy az adatok elemzése a lényeges tengely. Adatsorozat időadatok általában idő szerinti sorrendben megérkeznek, és általában a rendszer az adatbázis frissítése helyett egy INSERT utasítás. Ebből kifolyólag módosítása mérése történik adott idő alatt, amely lehetővé teszi, az előző verziókkal való kereséséhez és előre jelezni jövőbeli változás. Ilyen adatsorozat időadatok legjobb formájában jelenik meg pont vagy sor diagramok.

![Egy sor diagramon ábrázolt adatsorozat időadatok](./images/time-series-chart.png)

Néhány példa a idő adatsor a következők:

- A rendszer árak rögzített időbeli trendeket észleléséhez.
- Kiszolgáló teljesítményét, például CPU használati, i/o terhelés, memóriahasználat és a hálózati sávszélességet.
- Az érzékelők ipari berendezés, amely függőben lévő berendezések hibákhoz és eseményindító riasztási értesítések észleléséhez használható a telemetriai adatokat.
- Az illesztőprogram pontszám valós idejű car telemetriai adatokat, többek között a sebességét, fékezés és gyorsítás keresztül egy olyan időkeretet, egy összesített kockázat létrehozásához.

Az egyes ezekben az esetekben láthatja, hogyan idő egy tengely, lényeges. Az események megjelenítése a ahhoz, azok érkező jellemzője, kulcs idő adatsorozat adatainak, amíg a természetes historikus rendezést. Ez eltér olyan szabványos online Tranzakciófeldolgozási adatok adatcsatorna esetén, ahol adatok bármilyen sorrendben megadott, illetve bármikor frissíteni összegyűjtött adatokat.

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Idő adatsorozat megoldás választása kell betöltik az adatokat, amelyek stratégiai értéke egy meghatározott időtartamra vonatkozóan módosítások köré szerveződik, és elsősorban az új adatok beszúrása és ritkán frissítése, ha egyáltalán. Ez az információ segítségével rendellenességek észlelését, megjelenítheti a trendeket, és hasonlítsa össze az előzményadatok, többek között az aktuális adatok. Ez a architektúra típus is legmegfelelőbb prediktív modellezési és előrejelzési eredményeivel, mert a korábbi módosítások rekord adott idő alatt, amely tetszőleges számú előrejelzési modellek alkalmazhatók. 

A time series használja az alábbi előnyökkel jár:

* Egy eszköz vagy folyamat idővel hogyan változik egyértelműen jelöli.
* A segítségével gyorsan észlelni a kapcsolódó források értékre változik, így rendellenességek észlelését, és a trendek feltörekvő egyértelműen kiemeléséhez.
* A legalkalmasabb prediktív modellezési és előrejelzését.

### <a name="internet-of-things-iot"></a>Eszközök internetes hálózata (IoT)

Az IoT-eszközök által összegyűjtött adatok természetes alkalmasnak idő adatsorozat tárolási és elemzése. A bejövő adatokat beszúrni, és ritkán, ha valaha is, frissítve. Az adatok megjelölve, és szúrja be a sorrendben érkezett, és ezeket az adatokat általában a időrendben sorolja fel, hogy a felhasználók felderítéséhez a trendeket, helyszíni rendellenességek észlelését, és olvassa el a prediktív elemzési jelenik meg.

További információkért lásd: [az eszközök internetes hálózatát](../concepts/big-data.md#internet-of-things-iot).

### <a name="real-time-analytics"></a>Valós idejű elemzések

Adatok legtöbbször a Time series idő-és nagybetűket &mdash; Ez azt jelenti, hogy azt kell lennie, amelyre a gyors, valós idejű követni a trendeket, vagy hozzon létre riasztást. Ezekben az esetekben insights késedelmes okozhat állásidő és üzleti hatást. Emellett gyakran szükség van, különböző forrásokból, például az érzékelők különböző adatai közötti összefüggések keresésére.

Ideális esetben kellene lennie, amely a bejövő adatok valós idejű kezelésének és az összes magas pontosság és nagy részletességű feldolgozza az adatfolyam feldolgozása réteg. Ez nem mindig lehetséges, attól függően, hogy az adatfolyam-továbbítási architektúra és az adatfolyam pufferelés és az adatfolyam feldolgozása rétegek összetevői. Szükség lehet a idő adatsorok pontossági feláldozása csökkenti azt. Feldolgozási idő windows (néhány másodpercig, például), a késleltetett ehhez a feldolgozási réteg számításokhoz időben engedélyezése. Előfordulhat, hogy felbontásának kell, és az adatok összesítő hosszabb ideig, például több hónapon keresztül összegyűjtött adatokat a megjelenítendő nagyításához megjelenítésekor.

## <a name="challenges"></a>Kihívásai

* Adatsorozat időadatok legtöbbször nagyon nagy mennyiségű, különösen az IoT-forgatókönyvek esetén. Tárolására, indexelő, lekérdezése, elemzése és idő adatsor megjelenítése kihívást jelenthet. 
* Nagy sebességű tárolási megfelelő kombinációja keressenek meg és hatékony számítási műveletek kezelésére valós idejű elemzés, ugyanakkor minimalizálja a piacra és a teljes idő komoly kihívást beruházási költség.

## <a name="architecture"></a>Architektúra

Számos forgatókönyvben, például az adatsorozat időadatok, IoT, például az adatok rögzített valós időben. Például egy [valós idejű feldolgozással](./real-time-processing.md) architektúrájának megfelelő. 

Egy vagy több adatforrás adatait a réteg által pufferelés adatfolyamba van okozhatnak [IoT-központ](/azure/iot-hub/), [Event Hubs](/azure/event-hubs/), vagy [hdinsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction). Ezt követően az adatok a rétegben adatfolyam feldolgozása, amelyek igény szerint is kiosztják a machine learning a prediktív elemzési szolgáltatás feldolgozott adatokat dolgoz fel. A feldolgozott adatok tárolása az analitikus adatok, például a [HBase](/azure/hdinsight/hbase/apache-hbase-overview), [Azure Cosmos DB](/azure/cosmos-db/), Azure Data Lake, vagy a Blob Storage tárolóban. Az elemzés és a reporting alkalmazást vagy szolgáltatást, mint a Power bi-ban vagy OpenTSDB (Ha a HBase-ben tárolt) segítségével elemzési idő adatsorozat adatainak megjelenítéséhez.

Egy másik lehetőség [Azure idő adatsorozat Insights](/azure/time-series-insights/). Idő adatsorozat Insights egy olyan teljes körűen felügyelt szolgáltatás idő adatsorozat adatok. Ebben az architektúrában idő adatsorozat elemzések a szerepköröket, az adatfolyam feldolgozása, adattároló, és az elemzések és jelentéskészítési hajt végre. Adatfolyam-adatokat az IoT-központ vagy az Event Hubs, és tárolja, a folyamatok, elemzi, és közel valós időben jeleníti meg az adatokat fogad. Előre nem összesíti az adatokat, de a nyers események tárolja.

Idő adatsorozat Insights egy séma adaptív, ami azt jelenti, hogy nem kell minden adatok előkészítése való származtatás insights indításához tegye. Ez lehetővé teszi vizsgálatát, hasonlítsa össze és adatforrások különböző zökkenőmentesen összefüggéseket. Emellett biztosítja az SQL-szerű szűrők és az összesítések, képes létrehozni, megjelenítheti, hasonlítsa össze, és különböző idő adatsorozat kombinációját, tűz leképezések és mentése és megosztása lekérdezések képes átfedő. 

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Data Storage](../technology-choices/data-storage.md)
- [Elemzés, képi megjelenítések és jelentéskészítés](../technology-choices/analysis-visualizations-reporting.md)
- [Analitikai adatokat tároló](../technology-choices/analytical-data-stores.md)
- [Az adatfolyam feldolgozása](../technology-choices/stream-processing.md)