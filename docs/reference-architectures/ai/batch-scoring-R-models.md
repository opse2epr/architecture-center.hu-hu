---
title: Kötegelt pontozási az R-modellek az Azure-ban
description: Hajtsa végre a kötegelt pontozási az R-modellek használata az Azure Batch és a egy adatkészlet kiskereskedelmi store értékesítési előrejelzés alapján.
author: njray
ms.date: 03/29/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 4fa57168c337b01c8e7d0fc86ba54fee59a7ae47
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887986"
---
# <a name="batch-scoring-of-r-machine-learning-models-on-azure"></a>Batch-pontozás R gépi tanulási modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan hajtsa végre a kötegelt pontozási az R-modellek Azure Batch használatával. A forgatókönyv kiskereskedelmi store értékesítési előrejelzés alapján, de ez az architektúra bármilyen forgatókönyvhöz alapján történő egy R-modellek használatával nagy scaler generációja igénylő is általánosítva van. Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![Architektúradiagram][0]

**A forgatókönyv**: A supermarket lánc termékek értékesítés-előrejelzést a soron következő negyedév keresztül kell. Az előrejelzés lehetővé teszi, hogy a vállalat az ellátási lánc jobban kezelheti, és győződjön meg arról, megfelel-e a tárolók minden egyes termékek iránti igény. A vállalat frissíti az előrejelzéseket minden héten elérhetővé válik az előző hét új értékesítési adatait, és a termék értékesítési stratégia a következő negyedév van beállítva. Ha meg szeretné becsülni az egyes értékesítési előrejelzések a bizonytalanság ki osztóérték előrejelzések jönnek létre.

Feldolgozás az alábbi lépésekből áll:

1. Az Azure Logic Apps az előrejelzési kódgenerálási folyamat hetente egyszer aktiválódik.

1. A logikai alkalmazás elindul egy Azure-Tárolópéldányon futtatja az ütemezőt Docker-tároló, amely elindítja a pontozási feladatokat a Batch-fürtön.

1. Pontozó feladatok párhuzamos futtatása a Batch-fürt csomópontjai között. Minden egyes csomópont:

    1. Lekéri a feldolgozó Docker-rendszerképet a Docker Hubból, és elindít egy tárolót.

    1. A bemeneti adatokat olvasó és előre betanított az R-modellek az Azure Blob storage-ból.

    1. Az előrejelzések betanítják pontszámmodell.

    1. Az előrejelzési eredmények ír a blob storage-bA.

Az alábbi ábrán látható négy termékek (SKU) az előre jelzett értékesítési egy tárolóban. A fekete vonal jelzi az értékesítési előzmények, a szaggatott vonal előrejelzési medián (q50), a rózsaszín sávon kívüli jelöli az erdőtopológia-ötödik és hetvenötödik percentilisei, és a kék sáv az ötödik és kilencven-ötödik percentilisei jelöl.

![Értékesítési előrejelzések][1]

## <a name="architecture"></a>Architektúra

Az architektúra az alábbi összetevőkből áll.

[Az Azure Batch] [ batch] előrejelzési generációs feladatok párhuzamosan futtathatók a fürt a virtuális gépek szolgál. Előrejelzés végrehajtott előre betanított gépi tanulási modellek r Azure Batch megvalósított automatikusan méretezheti az virtuális gépek a fürt számára küldött számítási feladatok száma alapján. Minden egyes csomóponton az R-szkriptek pontszámot rendelni az adatok és előrejelzések készítése a Docker-tároló belül fut.

[Az Azure Blob Storage] [ blob] a bemeneti adatokat, az előre betanított machine learning-modellek és az előrejelzési eredményeket tárolja. Kínál rendkívül költséghatékony tárolási megoldás, amely szükséges a számítási feladat teljesítményét.

[Az Azure Container Instances] [ aci] adja meg a kiszolgáló nélküli számítási igény. Ebben az esetben a tárolópéldány indításához az előrejelzések készítése a Batch-feladatok ütemezett helyezünk üzembe. A Batch-feladatok egy R parancsfájlt az aktivált a [doAzureParallel] [ doAzureParallel] csomagot. A tárolópéldány automatikusan leáll, miután a feladat befejezve.

[Az Azure Logic Apps] [ logic-apps] a teljes munkafolyamat-trigger ütemezés szerint a tárolópéldányok üzembe helyezésével. Egy Azure Container Instances-összekötőt a Logic Apps lehetővé teszi, hogy számos olyan kiváltó események üzembe helyezni egy példányt.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

### <a name="containerized-deployment"></a>Tárolóalapú üzembe helyezés

Ebben az architektúrában az összes R-szkriptek futtatása belül [Docker](https://www.docker.com/) tárolók. Ez biztosítja, hogy a parancsfájlok futtatásához konzisztens környezetben, az R ugyanazon verzióját és a csomagok verzióit, minden alkalommal. Önálló Docker-rendszerképek szolgálnak az ütemezőt és a feldolgozó tárolók, mert mindegyik különböző rendelkezik R csomag függősége.

Az Azure Container Instances a scheduler tároló futtatásához egy kiszolgáló nélküli környezetet biztosít. A scheduler tároló futtatja az R-szkriptet, amely egy Azure Batch-fürtön futtatott egyéni pontozási feladatok.

A Batch-fürt minden csomópontján fut, a feldolgozó tároló, amely végrehajtja a pontozó szkript.

### <a name="parallelizing-the-workload"></a>A számítási feladatot párhuzamosan futtatni

Amikor kötegelt pontozási R-modellek az adatokat, gondolja át, hogyan párhuzamosíthatja a számítási feladatok. A bemeneti adatok valamilyen módon kell particionálni a úgy, hogy a pontozási műveletet a fürt csomópontjai között elosztható. Próbálja meg más megközelítést Fedezze fel a számítási feladatok elosztásához a legjobb választás. Eseti alapon vegye figyelembe a következőket:

- Könnyen betölthetők és egyetlen csomópont, a memória a feldolgozott adatok mennyiségét.
- Minden egyes batch-feladat indítása járó többletterhelést.
- Az R-modellek betöltése járó többletterhelést.

Az ehhez a példához használja a forgatókönyvben a modell objektumait nagy, és az egyes termékek Előrejelzés létrehozásához csak néhány másodpercet vesz igénybe. Ebből kifolyólag a termékek csoportban, és a csomópontonként egy egyetlen Batch-feladat végrehajtása. Belül minden egyes feladat hurkot állít elő a termékekre vonatkozó előrejelzések egymás után. Ez a módszer elemről kiderül, hogy a leghatékonyabb párhuzamosíthatja az adott számítási feladatot is. Ezzel elkerülheti a számos kisebb a Batch-feladat indítása és az R-modellek ismételten betöltése járó többletterhelést.

Egy alternatív módszer is termékenként egy Batch-feladat indításához. Az Azure Batch automatikusan képezi a feladatok várólistáját, és elküldi őket a fürtön kell végrehajtani, amint elérhetővé válnak a csomópontok. Használat [automatikus skálázást] [ autoscale] , állítsa be a feladatok számától függően a fürtben található csomópontok számát. Ez a megközelítés több értelme, ha minden egyes pontozási művelet, a modell objektumait újraépítésével és a feladatok kezdési járó többletterhelést igazoló egy viszonylag hosszú ideig tart. Ez a módszer emellett megvalósítása egyszerűbb, és rugalmasságot biztosít, automatikus méretezés használata – Ha a teljes terhelés mérete nem ismert előre fontos szempont.

## <a name="monitoring-and-logging-considerations"></a>Figyelés és naplózás kapcsolatos szempontok

### <a name="monitoring-azure-batch-jobs"></a>Azure Batch-feladatok figyelése

Figyelheti, és a Batch-feladatok leállítása a **feladatok** az Azure Portalon a Batch-fiók panelén. A batch-fürt, beleértve az egyes csomópontok állapotát a monitorozására a **készletek** ablaktáblán.

### <a name="logging-with-doazureparallel"></a>A doAzureParallel naplózása

A doAzureParallel csomag automatikusan gyűjt minden feladat az Azure Batch elküldve az összes stdout/stderr-naplóit. A telepítéskor jönnek létre storage-fiókban találhatók. Megtekintheti őket, használjon egy tárolási navigációs eszköz például [Azure Storage Explorer] [ storage-explorer] vagy az Azure Portalon.

Gyors fejlesztés során a Batch-feladatok hibakereséséhez, nyomtassa ki a helyi R munkamenetet használ a naplók a [getJobFiles] [ getJobFiles] doAzureParallel funkcióját.

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Ez a referenciaarchitektúra a használt számítási erőforrások összetevői a legtöbb költséges. Ebben a forgatókönyvben egy fix méretű fürt jön létre, amikor a feladat által aktivált, majd állítsa le a feladat befejezése után. Költsége akkor lesz felszámítva, csak a fürt csomópontjai indítása, futtatása vagy leállítása közben. Ez a megközelítés egy forgatókönyvet, ahol az előrejelzések készítése szükséges számítási erőforrások továbbra is viszonylag állandó feladat feladat ideális.

Olyan esetekben, ahol a feladat végrehajtásához szükséges számítási nem ismert előre lehet megfelelő automatikus méretezés használata. Ezzel a módszerrel a fürt méretét erőforrások méretezése pedig ettől felfelé vagy lefelé a projekt méretétől függően. Az Azure Batch számos meghatározásakor a fürt segítségével beállíthatók úgy automatikus méretezési képletek a [doAzureParallel] [ doAzureParallel] API-t.

Bizonyos esetekben a feladatok között eltelt idő lehet túl rövid, állítsa le és indítsa el a fürtöt. Ezekben az esetekben tartsa meg a fürtön futó feladatok között, ha szükséges.

Az Azure Batch- és a doAzureParallel támogatja az alacsony prioritású virtuális gépek. Ezek a virtuális gépek egy jelentős kedvezménnyel, de más magasabb prioritású számítási feladatok által éppen sajátíthatja kockázati kapható. Ezek a virtuális gépek használatát ezért használata nem ajánlott a kritikus fontosságú éles számítási feladatokhoz. Azonban azok nagyon hasznos a kísérleti vagy fejlesztési számítási feladatokhoz.

## <a name="deployment"></a>Környezet

Ez a referenciaarchitektúra üzembe helyezéséhez kövesse az ismertetett lépéseket a [GitHub] [ github] adattárat.


[0]: ./_images/batch-scoring-r-models.png
[1]: ./_images/sales-forecasts.png
[aci]: /azure/container-instances/container-instances-overview
[autoscale]: /azure/batch/batch-automatic-scaling
[batch]: /azure/batch/batch-technical-overview
[blob]: /azure/storage/blobs/storage-blobs-introduction
[doAzureParallel]: https://github.com/Azure/doAzureParallel/blob/master/docs/32-autoscale.md
[getJobFiles]: /azure/machine-learning/service/how-to-train-ml-models
[github]: https://github.com/Azure/RBatchScoring
[logic-apps]: /azure/logic-apps/logic-apps-overview
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows