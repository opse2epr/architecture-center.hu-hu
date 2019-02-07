---
title: Batch-pontozás Python-modellek az Azure-ban
description: Hozzon létre egy méretezhető megoldás, a kötegelt pontozási modellek Azure Machine Learning szolgáltatás használatával párhuzamosan ütemezés szerint.
author: njray
ms.date: 01/30/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai, AI
ms.openlocfilehash: 81dc353735eaa6573c72d9e588c949fe96a329ef
ms.sourcegitcommit: eee3a35dd5a5a2f0dc117fa1c30f16d6db213ba2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/06/2019
ms.locfileid: "55782013"
---
# <a name="batch-scoring-of-python-models-on-azure"></a>Batch-pontozás Python-modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan hozhat létre egy méretezhető megoldás, a kötegelt pontozási számos modellek Azure Machine Learning szolgáltatás használatával párhuzamosan ütemezés szerint. A megoldás sablonként is használható, és különböző problémákat generalize is.

Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![Batch-pontozás Python-modellek az Azure-ban](./_images/batch-scoring-python.png)

**A forgatókönyv**: Ez a megoldás egy nagy számú egy IoT beállításban, ahol a minden egyes eszköz által érzékelőinek folyamatos működését figyeli. Minden egyes eszköz feltételezhető, imagenet rendellenességek észlelése modellek, amely képes előre jelezni kell társítani kell-e egy sorozatát mértékegysége, amelyek egy előre meghatározott idő alatt összesítjük, felelnek meg az anomáliadetektálási vagy nem. A valós életből vett példák ennek oka lehet egy adatfolyam érzékelőinek adatai, amelyeket a szűrt és összesítve van használatban a képzés és a valós idejű pontozási előtt kell. Az egyszerűség kedvéért ez a megoldás ugyanazon adatok fájlt használja a pontozási feladat végrehajtása közben.

Ez a referenciaarchitektúra a számítási feladatok ütemezett kiváltó lett tervezve. Feldolgozás az alábbi lépésekből áll:
1.  Az Azure Event Hubs küldése érzékelőinek támogatunk.
2.  Hajtsa végre az adatfolyam-feldolgozás és a nyers adatok tárolásához.
3.  Az adatok elküldése egy megkezdheti a munkát véve Machine Learning-fürtön. A fürt minden csomópontján fut egy adott érzékelő pontozási feladat. 
4.  Hajtsa végre a pontozási folyamatot, amely a Machine Learning Python-szkriptekkel párhuzamosan fut a pontozási feladatok. A folyamat létrehozását, közzé és idő előre meghatározott időközönként ütemezve.
5.  Hozzon létre előrejelzések, és tárolja őket Blob Storage-későbbi felhasználásra.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő összetevőkből áll:

[Az Azure Event Hubs][event-hubs]. Az üzenet-feldolgozó szolgáltatás fogadására képes akár több millió eseményt üzenetek / másodperc. Ebben az architektúrában az érzékelők adatfolyamot küldeni az event hubs.

[Az Azure Stream Analytics][stream-analytics]. Egy eseményfeldolgozó motor. Stream Analytics-feladat beolvassa az adatokat az eseményközpontból érkező adatfolyamok, és elvégzi a adatfolyam-feldolgozás.

[Az Azure SQL Database][sql-database]. A érzékelőinek az adatok betöltése az SQL Database-be. Az SQL olyan jól ismert módon (ami táblázatos strukturált és strukturálatlan) feldolgozott, adatfolyamként továbbított adatok tárolására, de más adattárakban is használható.

[Az Azure Machine Learning szolgáltatás][amls]. A Machine Learning szolgáltatás egy felhőalapú szolgáltatás betanítási, pontozási, telepítésére és felügyeletére gépi tanulási modellek ipari méretekben. A kötegelt pontozási kontextusában a Machine Learning egy olyan fürtjét, virtuális gépek igény szerint egy automatikus skálázási beállítást, ahol a fürt minden csomópontján fut-e egy adott érzékelő pontozási feladat hoz létre. A pontozó feladatok várólistára és felügyeli a Machine Learning Python-szkript lépések végrehajtása párhuzamosan. Ezeket a lépéseket a Machine Learning létrehozott, közzétett, és a egy előre meghatározott időközönként idő futtatott, ütemezett folyamat részét képezik.

[Az Azure Blob Storage][storage]. BLOB-tárolók a pretrained modellek, az adatok és a kimeneti előrejelzéseket tárolására szolgálnak. A modellek töltődnek fel a Blob storage-ban a [01_create_resources.ipynb] [ create-resources] notebookot. Ezek [egy szintű SVM] [ one-class-svm] modellek képzett különböző eszközök eltérő érzékelők értéket jelölő adatokon. Ez a megoldás feltételezi, hogy az adatértékek vannak időszakra vonatkozó összesített érték egy rögzített idő.

[Az Azure Container Registry][acr]. A pontozó Python [parancsfájl] [ pyscript] fut, a fürt, ahol a megfelelő érzékelőktől kapott adatok beolvasása, állít elő, előrejelzéseket és a Blob storage-ban tárolja azokat minden egyes csomóponton létrehozott Docker-tárolókat.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Standard Python-modellek általánosan elfogadott, hogy a processzorok elegendőek a számítási feladatok kezelésére. Ez az architektúra processzorokat használ. Azonban a [mély tanulási célú számítási feladatokhoz][deep], gpu-k általában teljesítményben felülmúlják CPU jelentős szerint – processzorok marketingre fürt általában szükség hasonló teljesítmény eléréséhez.

### <a name="parallelizing-across-vms-vs-cores"></a>Virtuális gépek és magok közötti párhuzamosan futtatni

Futó folyamatok sok modellek pontozása kötegelt módban, a feladatok kell példánylista méretezésnek megfelelően. Két módszer is lehetséges:

* Hozzon létre egy nagyobb fürt alacsony költségű virtuális gépek használatával.

* Hozzon létre egy kisebb fürtöt magas végez az egyes elérhető több maggal rendelkező virtuális gépeket.

Általánosságban véve standard Python-modell pontozása nem az erőforrás-igényű, deep learning-modellek pontozása, és egy kisebb fürtöt tudja hatékonyan kezelni a sorban álló modellek nagy számú kell lennie. Növelheti a fürt csomópontok méretei dataset növekedésének megfelelően.

Az ebben a forgatókönyvben az egyszerűség kedvéért egy pontozási tevékenység belül küldte el gépi tanulási folyamat egyetlen lépésben. Azonban érdemes lehet hatékonyabb, ha az azonos folyamat lépés belül több adattömbök pontozása. Ezekben az esetekben több adatkészlet olvasása és azok számára, egy egylépéses végrehajtása során a pontozó szkript végrehajtása egyéni kód írása.

## <a name="management-considerations"></a>Eszközkezeléssel kapcsolatos szempontok

- **Feladatok figyelése**. Fontos, hogy a futó feladatok előrehaladásának figyeléséhez, de azt az aktív csomópontból álló fürtben figyelése kihívást jelenthet. Vizsgálja meg a fürt csomópontjainak állapotát, használja a [az Azure Portal] [ portal] kezelheti a [machine learning-munkaterület][ml-workspace]. Ha egy csomópont nem aktív, vagy egy feladat sikertelen volt, a hibanaplókat blob storage-bA lesznek mentve, és a folyamatok szakaszban is elérhetők. Gazdagabb figyelés csatlakozzon a naplók [Application Insights][app-insights], vagy a fürt és a feladatok állapotát a lekérdezéséhez külön folyamatok futtatásához.
-   **Naplózás**. Machine Learning szolgáltatás a társított Azure Storage-fiók összes stdout/stderr naplóz. A naplófájlok egyszerűen megtekintéséhez használja a tárolók navigációs eszköz például [Azure Storage Explorer][explorer].

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Ez a referenciaarchitektúra a használt legköltségesebb összetevői a számítási erőforrásokat. Attól függően, a várólistában a feladatok felfelé és lefelé méretezi a számítási fürt mérete. Automatikus skálázás engedélyezése programozott módon a Python SDK-n keresztül a számítási kiépítési konfigurációjának módosításával. Vagy használja a [Azure CLI-vel] [ cli] a fürt automatikus skálázási paramétereinek a beállításához.

És nem igényel azonnali feldolgozási munka konfigurálja az automatikus skálázási képletet, hogy az alapértelmezett állapot (minimum) az nulla csomópontból álló fürtben. Ezzel a konfigurációval a fürt nullára a csomópontok kezdődik, és ha a várólistán lévő feladatok csak felskálázással. Ha a kötegelt pontozási folyamat naponta csak néhány alkalommal történik, vagy kevesebb, mint ez a beállítás lehetővé teszi a jelentős költségmegtakarítást.

Az automatikus skálázás nem lehet megfelelő, a kötegelt feladatok számához egymáshoz közel túl fordulhat elő. A fürt számára le üzemeltethet a idejét is felmerülő költség, így ha egy batch számítási feladatot csak néhány percet az előző feladat befejezése után kezdődik, költséghatékonyabb tartani a fürtön futó feladatok között lehet. E pontozási folyamatokat beütemezve egy nagy gyakoriságú (például minden órában), vagy ritkábban függ (például havonta egyszer).


## <a name="deployment"></a>Környezet

Ez a referenciaarchitektúra üzembe helyezéséhez kövesse az ismertetett lépéseket a [GitHub-adattárat][github].

[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[automatic-scaling]: /azure/batch/batch-automatic-scaling
[azure-files]: /azure/storage/files/storage-files-introduction
[cli]: https://docs.microsoft.com/en-us/cli/azure
[create-resources]: https://github.com/Microsoft/AMLBatchScoringPipeline/blob/master/01_create_resources.ipynb
[deep]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[event-hubs]: /azure/event-hubs/event-hubs-geo-dr
[explorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
[github]: https://github.com/Microsoft/AMLBatchScoringPipeline
[one-class-svm]: http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html
[portal]: https://portal.azure.com
[ml-workspace]: https://docs.microsoft.com/en-us/azure/machine-learning/studio/create-workspace
[python-script]: https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py
[pyscript]: https://github.com/Microsoft/AMLBatchScoringPipeline/blob/master/scripts/predict.py
[storage]: /azure/storage/blobs/storage-blobs-overview
[stream-analytics]: /azure/stream-analytics/
[sql-database]: https://docs.microsoft.com/en-us/azure/sql-database/
[app-insights]: https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview
