---
title: Python scikit-képzés – ismerje meg, a modellek az Azure-ban
description: Ez a referenciaarchitektúra bemutatja az ajánlott eljárásokat, a hangolási a hiperparaméterek (képzési paraméterek), a scikit-ismerje meg a Python-modell.
author: njray
ms.date: 03/07/19
ms.custom: azcat-ai
ms.openlocfilehash: 23689ff7423b681c6cf5aeb713920a98f658044f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298569"
---
# <a name="training-of-python-scikit-learn-models-on-azure"></a>Python scikit-képzés – ismerje meg, a modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, ajánlott eljárások a hiperparaméterek (képzési paraméterek) a finomhangoláshoz egy [scikit-további] [ scikit] Python-modell. Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![Architektúradiagram][0]

**Forgatókönyv:** Az itt tárgyalt problémát – gyakori kérdések (GYIK) egyező. Ebben a forgatókönyvben az adatok egy részét Stack Overflow kérdés, amely tartalmazza az eredeti kérdések, JavaScript, az ismétlődő kérdések és a válaszok címkével használ. Ez hangolja a scikit-ismerje meg a folyamat használatával előrejelzi a valószínűségét, hogy egy ismétlődő kérdés megegyezik az eredeti kérdések egyike.

Feldolgozási ebben a forgatókönyvben az alábbi lépésekből áll:

1. A Python-szkriptet képzés elküldve a a [Azure Machine Learning szolgáltatás][aml].

1. A parancsfájl minden egyes csomóponton létrehozott Docker-tárolókban fut.

1. A szkript lekéri a betanításhoz és teszteléshez származó adatok [Azure Storage][storage].

1. A parancsfájl megtanul egy modellt a betanítási adatokból. használjon a betanítási paraméterek kombinációja.

1. A modell teljesítményét értékeli ki a tesztelési adatokon, és Azure Storage tárterületre írt.

1. A legjobban teljesítő modellt regisztrálva van az Azure Machine Learning szolgáltatásban.

További tájékoztatás képzési szempontjai [deep learning-modellek] [ training-deep-learning] gpu-k használatával.

## <a name="architecture"></a>Architektúra

Ez az architektúra számos Azure-felhőszolgáltatásokat, amely a méretezési csoport szükség szerint erőforrás áll. A szolgáltatások és azok szerepköreivel ebben a megoldásban alább ismertetjük.

[A Microsoft adatelemző virtuális gép] [ dsvm] (DSVM) egy Virtuálisgép-rendszerképet, adatelemzési és gépi tanulási eszközökkel konfigurálva. Windows Server és Linux-verziók érhetők el. A központi telepítés Ubuntu 16.04 LTS rendszeren a DSVM Linux verzióján használja.

[Az Azure Machine Learning szolgáltatás] [ aml] betanításához, üzembe helyezése, automatizálhatja és felügyelheti a machine learning-modellek felhőszinten szolgál. Használatban van ebben az architektúrában a kiosztási és az alábbiakban ismertetett Azure-erőforrások kezeléséhez.

[Az Azure Machine Learning Compute] [ aml-compute] , hogy az erőforrás, betanítását és tesztelése a machine learning és a méretezett AI-modellek az Azure-ban. A [számítási célt] [ target] ebben a forgatókönyvben a fürtben kiosztott igény szerint az automatikus alapján [skálázás] [ scaling] lehetőséget. Minden egyes csomópont a betanítási feladat egy adott futtató virtuális gép [hiperparaméter] [ hyperparameter] beállítása.

[Az Azure Container Registry] [ acr] tárolja a lemezképeket a Docker-tárolók üzembe helyezésének összes típusára vonatkozóan. Ezek a tárolók létrehozása minden egyes csomóponton és a képzési Python-szkript futtatásához használt. A lemezképet, a Machine Learning COMPUTE számítási fürtben használt a Machine Learning szolgáltatás a helyi Futtatás és a hiperparaméter finomhangolása notebookok létrejön, és ezután van Container Registrybe leküldött.

[Az Azure Blob] [ blob] tárolási a tanítási és tesztelési adatkészletek kap a Machine Learning szolgáltatás a képzés Python-szkript által használt. Tárolási van csatlakoztatva, egy virtuális meghajtó alakzatot a Machine Learning COMPUTE számítási fürt minden csomópontjának. 

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Minden készlete [hiperparaméterek] [ hyperparameter] a Machine Learning egyik csomópontján fut [számítási célt][target]. A jelen architektúra esetében minden egyes csomópont egy Standard D4 v2 virtuális gép, amely négy magot rendelkezik. Ez az architektúra használ egy [LightGBM] [ lightgbm] osztályozó a machine learning, kiemelési keretrendszer átmenetekhez. Ez a szoftver futtathatja összes négy magot egyszerre, gyorsabbá a legfeljebb négy szorosára minden egyes futtatásához. Ezzel a módszerrel akár az egész hiperparaméter finomhangolása futtatása szükséges a mennyi időt vesz igénybe egynegyede volna egy Machine Learning COMPUTE számítási célt Standard D1 v2 virtuális gép, amely csak egy minden egyes maghoz alapján futtathatók.

A Machine Learning Compute-csomópontok maximális száma befolyásolja a teljes futási idő. A csomópontok ajánlott minimális száma értéke nulla. Ezzel a beállítással egy feladat indításához szükséges idő a fürtbe való néhány percig, automatikus méretezést legalább egy csomópont tartalmaz. Ha a hiperparaméter finomhangolása rövid ideig fut, azonban a feladat vertikális felskálázása ad hozzá a terhelés. Például egy feladat futtatható öt perc alatt, de legfeljebb egy csomópont skálázási egy másik öt percig is eltarthat. Ebben az esetben a minimális értékre egy csomópont időt takaríthat meg, de ad hozzá a költségeket.

## <a name="monitoring-and-logging-considerations"></a>Figyelés és naplózás kapcsolatos szempontok

Küldje el a [HyperDrive] [ hyperparameter] figyelése a Futtatás folyamatban van egy Futtatás objektum használatra visszaadandó futtatási konfigurációt.

### <a name="rundetails-jupyter-widget"></a>RunDetails Jupyter Widget

A Futtatás objektum használata a RunDetails [Jupyter widget] [ jupyter] kényelmesen a az üzenetsor-kezelési, és annak gyermekeihez futó feladatok előrehaladásának figyeléséhez. Azt is bemutatja az elsődleges statisztika, amely valós időben bejelentkeznek értékeit.

### <a name="azure-portal"></a>Azure Portal

Nyomtassa ki egy Futtatás objektumot jeleníti meg a hivatkozást a Futtatás az Azure Portalon ehhez hasonló:

![Futtassa az objektum][1]

Ezen a lapon a Futtatás és annak gyermek futtatások állapotának monitorozásához. Minden egyes gyermek tartozik egy illesztőprogram-napló kimenetét a tanítási szkriptet futott.

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

A hiperparaméter finomhangolása Futtatás költségének Machine Learning Compute Virtuálisgép-méretet, hogy alacsony prioritású csomópontok használhatók és engedélyezett a fürtben található csomópontok maximális száma a kiválasztott lineárisan függ.

Folyamatban lévő költségeit, ha a fürt nem használja a fürt szükséges csomópontok minimális száma függenek. A fürt automatikus méretezés a rendszer automatikusan a megfelelő feladatok száma a megengedett maximális csomópontokat ad hozzá, és azok már nincs rá szükség, majd eltávolítja a csomópontok le a kért minimális. A fürt automatikus skálázási nullára a csomópontok le is, ha azt nem kerül Ha nem használja.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="restrict-access-to-azure-blob-storage"></a>Az Azure Blob Storage-hozzáférés korlátozása

Ez az architektúra használ [tárfiókkulcsok] [ storage-security] a blobtároló eléréséhez. További ellenőrzésére és védelmére fontolja meg egy közös hozzáférésű jogosultságkód (SAS) helyett. Ez a tároló objektumok korlátozott hozzáférést biztosít anélkül, hogy rögzítse szoftveresen a fiókkulcsok, vagy mentse őket az egyszerű szöveges. SAS használatával is biztosítható, hogy a tárfiók rendelkezik a megfelelő irányítás, és, hogy célja, hogy hozzáférhessenek a mobileszközeiken csak a személyeknek.

Forgatókönyvek a több bizalmas adatok győződjön meg arról, hogy a kulcsok összes védettek, mivel ezek a kulcsok teljes hozzáférési jogot az összes bemeneti és kimeneti adatok a munkaterhelési.

### <a name="encrypt-data-at-rest-and-in-motion"></a>Inaktív és a mozgásban lévő adatok titkosításához

Olyan esetekben, olyan bizalmas adatokat, az inaktív adatok titkosítása – storage-ban, hogy az adatokat. SSL használatával minden egyes idő adatok helyeződnek át egyik helyről a másikra, az adatátvitel biztonságos. További információkért lásd: a [Azure Storage biztonsági útmutatóját][storage-security].

### <a name="secure-data-in-a-virtual-network"></a>Biztonságos adattárolás a virtuális hálózat

Éles környezetekben üzemelő példányok fontolja meg a megadott virtuális hálózat egy alhálózatában-fürt üzembe helyezésekor. Ez lehetővé teszi a számítási csomópontok, a fürt más virtuális gépekkel vagy egy helyszíni hálózattal biztonságosan kommunikál. Is [szolgáltatásvégpontokat] [ endpoints] a blob storage-hozzáférés biztosítása virtuális hálózatról, vagy a virtuális hálózaton belül egy egycsomópontos NFS használata az Azure Machine Learning szolgáltatásban.

## <a name="deployment"></a>Környezet

A referenciaimplementációt, az architektúra üzembe helyezéséhez kövesse a [GitHub] [ github] adattárat.

[0]: ./_images//training-python-models.png
[1]: ./_images/run-object.png
[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml]: /azure/machine-learning/service/overview-what-is-azure-ml
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[blob]: /azure/storage/blobs/storage-blobs-introduction
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[github]: https://github.com/Microsoft/MLHyperparameterTuning
[hyperparameter]: /azure/machine-learning/service/how-to-tune-hyperparameters
[jupyter]: http://jupyter.org/widgets
[lightgbm]: https://github.com/Microsoft/LightGBM
[scaling]: /azure/virtual-machine-scale-sets/overview
[scikit]: https://pypi.org/project/scikit-learn/
[storage]: /azure/storage/common/storage-introduction
[storage-security]: /azure/storage/common/storage-security-guide
[target]: /azure/machine-learning/service/how-to-auto-train-remote
[training-deep-learning]: /azure/architecture/reference-architectures/ai/training-deep-learning