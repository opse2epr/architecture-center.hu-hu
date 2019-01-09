---
title: Batch-pontozás Python-modellek az Azure-ban
description: Hozzon létre egy méretezhető megoldás, a kötegelt pontozási modellek az Azure Batch AI segítségével párhuzamosan ütemezés szerint.
author: njray
ms.date: 12/13/2018
ms.custom: azcat-ai
ms.openlocfilehash: 4c43a3dadab11cb8dcf163cf63618795299283ad
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111051"
---
# <a name="batch-scoring-of-python-models-on-azure"></a>Batch-pontozás Python-modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan hozhat létre egy méretezhető megoldás, a kötegelt pontozási számos modellt az Azure Batch AI segítségével párhuzamosan ütemezés szerint. A megoldás sablonként is használható, és különböző problémákat generalize is.

Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![Batch-pontozás Python-modellek az Azure-ban](./_images/batch-scoring-python.png)

**A forgatókönyv**: Ez a megoldás egy nagy számú egy IoT beállításban, ahol a minden egyes eszköz által érzékelőinek folyamatos működését figyeli. Minden eszköz rendelkezik előre betanított rendellenességek észlelése, a modellek, hogy kell képes előre jelezni az e sorozata mértékegysége, amelyek egy előre meghatározott idő alatt összesítjük, ha meg szeretné anomáliát vagy nem veszi alapul. A valós életből vett példák ennek oka lehet egy adatfolyam érzékelőinek adatai, amelyeket a szűrt és összesítve van használatban a képzés és a valós idejű pontozási előtt kell. Az egyszerűség kedvéért a megoldást ugyanazon adatok fájlt használja a pontozási feladat végrehajtása közben.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő összetevőkből áll:

[Az Azure Event Hubs][event-hubs]. Az üzenet-feldolgozó szolgáltatás fogadására képes akár több millió eseményt üzenetek / másodperc. Ebben az architektúrában az érzékelők adatfolyamot küldeni az event hubs.

[Az Azure Stream Analytics][stream-analytics]. Egy eseményfeldolgozó motor. Stream Analytics-feladat beolvassa az adatokat az eseményközpontból érkező adatfolyamok, és elvégzi a adatfolyam-feldolgozás.

[Az Azure Batch AI][batch-ai]. Ez az elosztott számítási motor szolgál taníthat vagy tesztelhet a machine learning és a méretezett AI-modellek az Azure-ban. A Batch AI és az automatikus méretezési lehetőség, ahol a Batch AI-fürt minden csomópontján fut-e egy adott érzékelő pontozási feladat igény szerinti virtuális gépeket hoz létre. A pontozó Python [parancsfájl] [ python-script] fut, a fürt, ahol a megfelelő érzékelőktől kapott adatok beolvasása, állít elő, előrejelzéseket és a Blob storage-ban tárolja azokat minden egyes csomóponton létrehozott Docker-tárolókat.

[Az Azure Blob Storage][storage]. BLOB-tárolók a pretrained modellek, az adatok és a kimeneti előrejelzéseket tárolására szolgálnak. A modellek töltődnek fel a Blob storage-ban a [létrehozása\_resources.ipynb] [ create-resources] notebookot. Ezek [egy szintű SVM] [ one-class-svm] modellek képzett különböző eszközök eltérő érzékelők értéket jelölő adatokon. Ez a megoldás feltételezi, hogy az adatértékek vannak időszakra vonatkozó összesített érték egy rögzített idő.

[Az Azure Logic Apps][logic-apps]. Ez a megoldás létrehoz egy logikai alkalmazást, amely a Batch AI-feladatok óránként fut. A Logic Apps segítségével egyszerűen hozhat létre a munkafolyamat futásidejű és az ütemezés a megoldás. A Batch AI-feladatok elküldése a Python használatával [parancsfájl] [ script] futtató Docker-tárolóban.

[Az Azure Container Registry][acr]. Docker-rendszerképek a Batch AI és a Logic Apps használja, és hozza létre a rendszer a [létrehozása\_resources.ipynb] [ create-resources] jegyzetfüzet, majd leküldte a tárolójegyzékbe. Ez a képek üzemeltethet, és hozza létre a tárolók más Azure-szolgáltatásokon keresztül kényelmes módot biztosít – a Logic Apps és a Batch AI ebben a megoldásban.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Standard Python-modellek általánosan elfogadott, hogy a processzorok elegendőek a számítási feladatok kezelésére. Ez az architektúra processzorokat használ. Azonban a [mély tanulási célú számítási feladatokhoz][deep], gpu-k általában teljesítményben felülmúlják CPU jelentős szerint – processzorok marketingre fürt általában szükség hasonló teljesítmény eléréséhez.

### <a name="parallelizing-across-vms-vs-cores"></a>Virtuális gépek és magok közötti párhuzamosan futtatni

Futó folyamatok sok modellek pontozása kötegelt módban, a feladatok kell példánylista méretezésnek megfelelően. Két módszer is lehetséges:

* Hozzon létre egy nagyobb fürt alacsony költségű virtuális gépek használatával.

* Hozzon létre egy kisebb fürtöt magas végez az egyes elérhető több maggal rendelkező virtuális gépeket.

Általánosságban véve standard Python-modell pontozása nem az erőforrás-igényű, deep learning-modellek pontozása, és egy kisebb fürtöt tudja hatékonyan kezelni a sorban álló modellek nagy számú kell lennie. Növelheti a fürt csomópontok méretei dataset növekedésének megfelelően.

Az ebben a forgatókönyvben az egyszerűség kedvéért egy pontozási tevékenység belül küldte el egy Batch AI-feladat. Azonban érdemes lehet hatékonyabb, ha egy Batch AI feladaton belül több adattömbök pontozása. Ezekben az esetekben több adatkészlet olvasása és azok számára egy egy Batch AI-feladat végrehajtása során, a pontozó szkript végrehajtása egyéni kód írása.

### <a name="file-servers"></a>Fájlkiszolgálók

Batch AI használata esetén kiválaszthatja a forgatókönyvhöz szükséges átviteli függően többféle tárolási lehetőség. Az alacsony átviteli sebességet megkövetelő számítási feladatokhoz a blob storage használatával kell lennie elegendő. Másik lehetőségként a Batch AI is támogatja a [Batch AI-fájlkiszolgáló][bai-file-server], egy felügyelt, egycsomópontos NFS, amely automatikusan csatlakoztathatók a fürtcsomópontokon, adja meg a központilag elérhető tárolási helye feladatok. A legtöbb esetben csak egy fájlkiszolgáló van szükség a munkaterületen, és meg is szét az adatokat a betanítási feladatokhoz más könyvtárban.

Ha egy egycsomópontos NFS nem megfelelő, a számítási feladatokhoz, Batch AI támogatja-e más tárolási lehetőségeket, ideértve [Azure Files] [ azure-files] és egyéni megoldások, például egy Gluster vagy Lustre fájlrendszer.

## <a name="management-considerations"></a>Eszközkezeléssel kapcsolatos szempontok

### <a name="monitoring-batch-ai-jobs"></a>Batch AI-feladatok figyelése

Fontos, hogy a futó feladatok előrehaladásának figyeléséhez, de azt az aktív csomópontból álló fürtben figyelése kihívást jelenthet. Megtapasztalhatja, a fürt általános állapotát, nyissa meg a **Batch AI** paneljén a [az Azure Portal] [ portal] vizsgálhatja meg a fürt csomópontjainak állapotát. Ha egy csomópont nem aktív, vagy egy feladat sikertelen volt, a hibanaplókat menti, és a blob storage-, és szintén elérhető az **feladatok** a portál panelén.

Gazdagabb figyelés csatlakozzon a naplók [Application Insights][ai], vagy a Batch AI-fürtöt és a feladatok állapotának lekérdezéséhez külön folyamatok futtatásához.

### <a name="logging-in-batch-ai"></a>A Batch AI-naplózás

A Batch AI minden stdout/stderr naplók társított Azure storage-fiókhoz. Egyszerűen megtalálható a naplófájlok, használja a tárolók navigációs eszköz például [Azure Storage Explorer][explorer].

Ez a referenciaarchitektúra telepítésekor lehetősége van egy egyszerűbb naplózási rendszer beállításához. Ezzel a beállítással a különböző feladatok között a naplók ugyanabba a könyvtárba, a blob-tárolóban, ahogy az alábbi lesznek mentve. Ezek a naplók segítségével figyelheti, mennyi ideig tart minden egyes rendszerképek és feldolgozni, így jobban érti, hogyan optimalizálható a folyamat.

![Azure Storage Explorer](./_images/batch-scoring-python-monitor.png)

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Ez a referenciaarchitektúra a használt legköltségesebb összetevői a számítási erőforrásokat.

A Batch AI-fürt mérete méretezhető felfelé és lefelé a várólistában a feladatok függően. Engedélyezheti a [automatikus skálázást] [ automatic-scaling] a Batch AI és a két módszer egyikével. Megteheti szoftveresen, amely konfigurálható a .env fájlban, amely része a [üzembe helyezési lépések][github], vagy a fürt létrehozása után módosíthatja a méretezési képletet közvetlenül a portálon.

És nem igényel azonnali feldolgozási munka konfigurálja az automatikus skálázási képletet, hogy az alapértelmezett állapot (minimum) az nulla csomópontból álló fürtben. Ezzel a konfigurációval a fürt nullára a csomópontok kezdődik, és ha a várólistán lévő feladatok csak felskálázással. Ha a kötegelt pontozási folyamat csak néhány alkalommal naponta történik vagy annál kisebb, ez a beállítás lehetővé teszi, hogy jelentős költségmegtakarítást.

Az automatikus skálázás nem lehet megfelelő, a kötegelt feladatok számához egymáshoz közel túl fordulhat elő. A fürt számára le üzemeltethet a idejét is költségkezelési, így ha egy batch számítási feladatot csak néhány percet az előző feladat befejezése után kezdődik, költséghatékonyabb tartani a fürtön futó feladatok között lehet. E pontozási folyamatokat beütemezve egy nagy gyakoriságú (például minden órában), vagy ritkábban függ (például havonta egyszer).

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A referenciaimplementációt a jelen architektúra érhető el az [GitHub][github]. Kövesse a beállítási lépéseket nem hozhat létre egy méretezhető megoldás, sok modellek pontozó a Batch AI segítségével párhuzamosan.

[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[automatic-scaling]: /azure/batch/batch-automatic-scaling
[azure-files]: /azure/storage/files/storage-files-introduction
[batch-ai]: /azure/batch-ai/
[bai-file-server]: /azure/batch-ai/resource-concepts#file-server
[create-resources]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/create_resources.ipynb
[deep]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[event-hubs]: /azure/event-hubs/event-hubs-geo-dr
[explorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
[github]: https://github.com/Azure/BatchAIAnomalyDetection
[logic-apps]: /azure/logic-apps/logic-apps-overview
[one-class-svm]: http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html
[portal]: https://portal.azure.com
[python-script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py
[script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/sched/submit_jobs.py
[storage]: /azure/storage/blobs/storage-blobs-overview
[stream-analytics]: /azure/stream-analytics/
