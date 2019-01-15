---
title: Elosztott-betanítás deep learning-modellek az Azure-ban
description: Ez a referenciaarchitektúra bemutatja, hogyan elosztott-betanítás deep learning-modellek között GPU-kompatibilis virtuális gépek fürtjeinek elvégzésére az Azure Batch AI használatával.
author: njray
ms.date: 01/14/19
ms.custom: azcat-ai
ms.openlocfilehash: 800defeb851f5a31dc730038c3699e1a3d54b923
ms.sourcegitcommit: d5ea427c25f9f7799cc859b99f328739ca2d8c1c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/15/2019
ms.locfileid: "54307837"
---
# <a name="distributed-training-of-deep-learning-models-on-azure"></a>Elosztott-betanítás deep learning-modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan végezhet elosztott-betanítás deep learning-modellek között GPU-kompatibilis virtuális gépek fürtjeinek. A forgatókönyv képbesorolás, de a megoldás is általánosítható más deep learning-forgatókönyvek például a képszegmentáláshoz és objektumfelismeréshez.

Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![Mély tanulás elosztott architektúra][0]

**A forgatókönyv**: Képek besorolása olyan széles körben alkalmazott módszer a számítógépes látástechnológiai, konvolúciós Neurális hálózat (CNN) képzési által gyakran megoldásra. Nagy adatkészletekkel különösen nagyméretű modellek esetében a betanítási folyamat eltarthat hetek vagy hónapok egyetlen gpu-alapú. Bizonyos esetekben a modellek rendkívül nagy méretűek, hogy azt ne legyen lehetséges, hogy ésszerű köteg méretek a GPU-kiszolgálóra is. Ezekben a helyzetekben elosztott képzési használatával lerövidítheti a képzési időt.

Ez az adott esetben egy [ResNet50 CNN modell] [ resnet] tanítása az [Horovod] [ horovod] a a [épít adatkészlet] [ imagenet] és a szintetikus adatok. A referenciaimplementáció ennek a feladatnak három a népszerű deep learning-keretrendszerek használatával mutatja be: Tensorflow-hoz, Keras és PyTorch.

Többféleképpen is egy elosztott módon, beleértve az adat- és modell megközelítések szinkron vagy aszinkron frissítések alapján deep learning-modell betanításához. A leggyakoribb eset jelenleg az adatok párhuzamos szinkron frissítésekkel. Ezt a módszert a legegyszerűbb megvalósításához, és használja a legtöbb esetben elegendő.

Adat-párhuzamos elosztott szinkron frissítésekkel képzés, a modell replikál a rendszer *n* hardvereszközök. Egy mini batch képzési minták oszlik *n* micro-kötegek. Minden egyes eszköz az előre és visszafelé irányuló pass micro – batch hajt végre. Amikor az eszköz befejezi a folyamatot, és a más eszközök közös a frissítéseket. Ezekkel az értékekkel a teljes mini köteg frissített súlyozású kiszámításához, és a modellek között szinkronizált végpontkészletben. Ebben a forgatókönyvben tárgyalja a [GitHub] [ github] tárház.

![Adatok párhuzamos, elosztott betanítás][1]

Ez az architektúra a modell-párhuzamos és aszinkron frissítések is használható. Modell-párhuzamos elosztott képzés, a modell között oszlik *n* hardvereszközök, minden eszközön, a modell egy részének tárolására. A legegyszerűbb megvalósítás minden egyes eszköz rendelkezhet egy réteget, a hálózat, és információkat eszköz alatt az előre és visszafelé között átadott pass. Ezzel a módszerrel nagyobb Neurális hálózatokat is betanított, de cserébe teljesítmény, mivel eszközök folyamatosan várnak egymással végrehajtásához vagy az előre, vagy visszamenőleges adja át. Próbálja meg részben szintetikus átmenetekhez használatával a probléma enyhítése néhány speciális módszerekig.

A képzési lépések a következők:

1. Hozzon létre a parancsprogramokat, amelyek fog futtatunk a fürtön a modell betanítását, majd átviszi a file storage.

1. A Blob Storage szeretne adatokat írni.

1. Hozzon létre egy Batch AI-fájlkiszolgáló, és töltse le az adatokat Blob Storage-ból rá.

1. Hozzon létre minden egyes deep learning keretrendszer Docker-tárolót, és helyezze át a tároló-beállításjegyzék (Docker Hub).

1. Hozzon létre egy Batch AI-készlet, amely a Batch AI-fájlkiszolgáló is csatlakoztatja.

1. Feladatok elküldéséhez. Minden egyes kér le a megfelelő Docker-rendszerképet és szkriptekben.

1. Ha a feladat befejeződött, az összes eredmény írni fájlok tárolására.

## <a name="architecture"></a>Architektúra

Az architektúra az alábbi összetevőkből áll.

**[Az Azure Batch AI] [ batch-ai]**  ebben az architektúrában a központi szerepet játszik az erőforrások skálázása felfelé és lefelé megfelelően szükség szerint. A Batch AI egy szolgáltatása, amely segít üzembe helyezése és kezelése virtuális gépek fürtjeinek, feladatok ütemezése, gyűjtse össze az eredményeket, méretezni az erőforrásokat, hibáinak a kezelése és létrehozása a megfelelő tárolási. Deep learning számítási feladatokhoz GPU-kompatibilis virtuális gépeket támogatja. A Python SDK-t és a egy parancssori felület (CLI) a Batch AI érhető el.

> [!NOTE]
> Kivonás alatt áll az Azure Batch AI szolgáltatás márciusi 2019, és az ipari méretekben képzés és pontozás képességek érhetők el mostantól [Azure Machine Learning szolgáltatás][amls]. Ez a referenciaarchitektúra hamarosan frissül majd használni a Machine Learning, így az úgynevezett felügyelt számítási célt [Azure Machine Learning Compute] [ aml-compute] képzés, üzembe helyezése és pontozás a machine tanulási modelleket.

**[A BLOB storage-] [ azure-blob]**  szolgál az adatok előkészítéséhez. Ezek az adatok betanítás során letölti a Batch AI-fájlkiszolgáló.

**[Az Azure Files] [ files]**  a parancsfájlok, naplókat és az oktatás, a végső eredmények tárolására szolgál. File storage esetén működik hatékonyan tárolására naplózza és parancsfájlok, de nem legjobb teljesítménnyel működhessen, Blob Storage-ban, így nem használható a data-igényes feladatokhoz.

**[Batch AI-fájlkiszolgáló] [ batch-ai-files]**  egy egycsomópontos NFS-megosztás ebben az architektúrában a betanítási adatok tárolására használt. A Batch AI az NFS-megosztások hoz létre, és csatlakoztatja, a fürtön. Batch AI-fájlkiszolgálók ajánljuk a Előkészíthetők az adatok a fürthöz a szükséges átviteli sebességgel.

**[A docker Hub] [ docker]**  a Docker-rendszerképet, amelyet a Batch AI használ a betanítási Futtatás tárolására szolgál. A docker Hub az architektúra lett választva, mert egyszerűen használható, és az alapértelmezett lemezkép adattár a Docker-felhasználók. [Az Azure Container Registry] [ acr] is használható.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Az Azure biztosít a négy [GPU-kompatibilis virtuális gépek típusai] [ gpu] megfelelő deep learning-modellek betanításához. Ezek tartománya az ár, és gyorsabb alacsony, magas a következő:

| **Az Azure Virtuálisgép-sorozatok** | **NVIDIA GPU** |
|---------------------|----------------|
| NC                  | K80            |
| ND                  | P40            |
| NCv2                | P100           |
| NCv3                | V100           |

Azt javasoljuk, hogy a tanítási horizontális felskálázása előtt vertikális felskálázása. Például próbálja meg egyetlen V100 K80s fürtben kísérlet előtt.

A következő diagram bemutatja a teljesítménybeli különbséget különböző GPU-típusok alapján [teljesítménytesztek eredménye] [ benchmark] tensorflow-hoz és Horovod használata a Batch AI végzett. A grafikonon látható átviteli 32 GPU-fürtök különböző modellek különböző GPU-és MPI-verziók között. Modellek TensorFlow 1.9 lettek megvalósítva

![A GPU-fürtökön TensorFlow-modellek átvitelisebesség-eredmény][2]

Az előző táblázatban szereplő minden egyes Virtuálisgép-sorozatok, InfiniBand használatával egy konfigurációt tartalmaz. Az InfiniBand-konfigurációk használata elosztott futtatásakor képzésekről, a gyorsabb kommunikáció a csomópontok között. InfiniBand is növeli a méretezési hatékonyságát az oktatás, amelyek kihasználhatják a azt a keretrendszereket. További információkért lásd: az Infiniband [összehasonlító becslésére][benchmark].

Bár a Batch AI a Blob storage használatával csatlakoztathatja a [blobfuse] [ blobfuse] adapter, nem javasoljuk a Blob Storage ezzel a módszerrel elosztott képzéshez, mert a teljesítmény nem elég jó kezeléséhez a szükséges átviteli sebességet. Helyezze át az adatokat egy Batch AI-fájlkiszolgáló helyette, ahogyan az az architektúra diagramját.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az elosztott képzési méretezési hatékonyságát mindig kevesebb, mint 100 %-os hálózati terhelés miatt &mdash; szinkronizálása eszközök között a teljes modell szűk keresztmetszetté válik. Elosztott képzési ezért legjobban megfelel nagy modellek, amelyek nem egy ésszerű kötegmérettel egyetlen gpu-alapú kell képezni, illetve egy egyszerű, párhuzamos módon osztja meg a modell nem javított problémák.

Elosztott képzési hiperparaméter keresések futtatása nem ajánlott. A méretezési hatékonyságát befolyásolja a teljesítményt, és elosztott kevésbé hatékony, mint a több modell konfiguráció külön-külön képzési teszi.

Egy méretezési hatékonyság növelése módja a köteg méretének növeléséhez. Amely kell elvégezni, óvatosan, azonban, mert a köteg méretének növelése nélkül a többi paraméter módosításával összeállítása hátrányosan befolyásolhatja a modell végső teljesítményét.

## <a name="storage-considerations"></a>A tárterülettel kapcsolatos szempontok

Deep learning-modellek betanításakor gyakran kihagyott felmérésére, az adatok tárolására. Ha a tároló a GPU-k igényeinek kielégítésére túl lassú, a képzési teljesítmény csökken.

A Batch AI számos tárolási megoldásokat támogatja. Ez az architektúra egy Batch AI-fájlkiszolgáló használja, mert biztosítja a legjobb egyensúlyt a könnyű használat és a teljesítmény között. A legjobb teljesítmény érdekében az adatok helyi betölteni. Azonban ez nehézkes lehet, mert minden csomópont le kell töltenie az adatok Blob Storage-ból, és az épít adatkészlettel, ez órát is igénybe vehet. [Prémium szintű Azure Blob Storage] [ blob] (korlátozott nyilvános előzetes verzió) egy másik megfontolandó jó lehetőség.

Csatlakoztatható, Blobok és fájlok tárolási adattárak, az elosztott képzési. Túl lassú, és akadályozzák a betanítási teljesítményét.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="restrict-access-to-azure-blob-storage"></a>Az Azure Blob Storage-hozzáférés korlátozása

Ez az architektúra használ [tárfiókkulcsok] [ security-guide] a blobtároló eléréséhez. További ellenőrzésére és védelmére fontolja meg egy közös hozzáférésű jogosultságkód (SAS) helyett. Ez a tároló objektumok korlátozott hozzáférést biztosít anélkül, hogy rögzítse szoftveresen a fiókkulcsok, vagy mentse őket az egyszerű szöveges. SAS használatával is biztosítható, hogy a tárfiók rendelkezik a megfelelő irányítás, és, hogy célja, hogy hozzáférhessenek a mobileszközeiken csak a személyeknek.

Forgatókönyvek a több bizalmas adatok győződjön meg arról, hogy a kulcsok összes védettek, mivel ezek a kulcsok teljes hozzáférési jogot az összes bemeneti és kimeneti adatok a munkaterhelési.

### <a name="encrypt-data-at-rest-and-in-motion"></a>Inaktív és a mozgásban lévő adatok titkosításához

Olyan esetekben, olyan bizalmas adatokat, az inaktív adatok titkosítása &mdash; storage-ban, hogy az adatokat. SSL használatával minden egyes idő adatok helyeződnek át egyik helyről a másikra, az adatátvitel biztonságos. További információkért lásd: a [Azure Storage biztonsági útmutatóját][security-guide].

### <a name="secure-data-in-a-virtual-network"></a>Biztonságos adattárolás a virtuális hálózat

Éles környezetekben üzemelő példányok fontolja meg a megadott virtuális hálózat egy alhálózatában Batch AI-fürt üzembe helyezésekor. Ez lehetővé teszi a számítási csomópontok, a fürt más virtuális gépekkel vagy egy helyszíni hálózattal biztonságosan kommunikál. Is [szolgáltatásvégpontokat] [ endpoints] hozzáférést biztosít egy virtuális hálózatból, vagy a virtuális hálózaton belül egy egycsomópontos NFS használata a Batch AI blob-tárolóval.

## <a name="monitoring-considerations"></a>Felügyeleti szempontok

A feladat futtatásakor fontos nyomon követheti, és győződjön meg arról, hogy a dolgok várt módon működik. Azonban ez több aktív csomópontból álló fürtben figyelése kihívást jelenthet.

Az Azure Portalon kezelheti a Batch AI-fájlkiszolgálók vagy, ha a [Azure CLI-vel] [ cli] és a Python SDK-t. Megtapasztalhatja, a fürt általános állapotát, navigáljon a **Batch AI** a fürtcsomópontok állapotának vizsgálata az Azure Portalon. Ha egy csomópont nem aktív, vagy egy feladat sikertelen volt, a hibanaplókat blob storage-bA lesznek mentve, és a is elérhetők az Azure portál **feladatok**.

Naplók való csatlakozással figyelési bővítését [Azure Application Insights] [ ai] vagy különálló, amely lekérdezi a Batch AI-fürtöt, és a feladatok állapotát a folyamat futtatásával.

A Batch AI minden stdout/stderr automatikusan bejelentkezik a Blob storage-fiók társítása. Például használja a tárolók navigációs eszköz [Azure Storage Explorer] [ storage-explorer] és a naplófájlok navigáláskor könnyebben szolgáltatással.

Akkor is a stream minden egyes feladathoz a naplókat. Ezt a beállítást, lásd a fejlesztési lépéseket a [GitHub][github].

## <a name="deployment"></a>Környezet

A referenciaimplementációt a jelen architektúra érhető el az [GitHub][github]. Elosztott-betanítás deep learning-modellek között GPU-kompatibilis virtuális gépek fürtjeinek elvégzésére van ismertetett lépéseket követve.

## <a name="next-steps"></a>További lépések

Ez az architektúra a kimenete a betanított modell mentett blob storage-bA. Ez a modell, valós idejű pontozási vagy kötegelt pontozási is üzembe helyezheti. További információkért tekintse meg a következő referenciaarchitektúrákat:

- [Valós idejű pontozási Python Scikit-ismerje meg, és a deep learning-modellek az Azure-ban][real-time-scoring]
- [Kötegelt pontozási az Azure-on deep learning-modellek][batch-scoring]

[0]: ./_images/distributed_dl_architecture.png
[1]: ./_images/distributed_dl_flow.png
[2]: ./_images/distributed_dl_tests.png
[acr]: /azure/container-registry/container-registry-intro
[ai]: /azure/application-insights/app-insights-overview
[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[azure-blob]: /azure/storage/blobs/storage-blobs-introduction
[batch-ai]: /azure/batch-ai/overview
[batch-ai-files]: /azure/batch-ai/resource-concepts#file-server
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-deep-learning
[benchmark]: https://github.com/msalvaris/BatchAIHorovodBenchmark
[blob]: https://azure.microsoft.com/en-gb/blog/introducing-azure-premium-blob-storage-limited-public-preview/
[blobfuse]: https://github.com/Azure/azure-storage-fuse
[cli]: https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md
[docker]: https://hub.docker.com/
[endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[files]: /azure/storage/files/storage-files-introduction
[github]: https://github.com/Azure/DistributedDeepLearning/
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[horovod]: https://github.com/uber/horovod
[imagenet]: http://www.image-net.org/
[real-time-scoring]: /azure/architecture/reference-architectures/ai/realtime-scoring-python
[resnet]: https://arxiv.org/abs/1512.03385
[security-guide]: /azure/storage/common/storage-security-guide
[storage-explorer]: /azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows
[tutorial]: https://github.com/Azure/DistributedDeepLearning