---
title: A mélytanulási modellek kötegelt kiértékelése
titleSuffix: Azure Reference Architectures
description: Ez a referenciaarchitektúra bemutatja, hogyan Neurális stílus átviteli alkalmazandó egy videót, az Azure Machine Learning segítségével.
author: jiata
ms.date: 02/06/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 3459f7895b7b57833da5853a77b2641dc7c85a9e
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639716"
---
# <a name="batch-scoring-of-deep-learning-models-on-azure"></a>Batch-pontozás deep learning-modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan Neurális stílus átviteli alkalmazandó egy videót, az Azure Machine Learning segítségével. *Átviteli stílus* deep learning-módszer, amely egy másik képet stílusát egy meglévő kép composes. Ez az architektúra bármilyen forgatókönyvhöz kötegelt pontozási használatával deep learninget használó is általánosítva. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Az Azure Machine Learning használatával deep learning-modellek architektúra ábrája](./_images/aml-scoring-deep-learning.png)

**A forgatókönyv**: Egy media szervezete egy videót, amelynek stílusát, így jelenik meg szeretne, például egy adott festmény. A szervezet szeretne tudni a stílus alkalmazása időben, és automatizált módon zajlik a videó minden keretet. További ismereteket a Neurális stílus átviteli algoritmusok, lásd: [kép stílus átvitel használata Konvolúciós Neurális hálózatokkal] [ image-style-transfer] (PDF).

<!-- markdownlint-disable MD033 -->

| Style-lemezképet: | A videó bemeneti/tartalom: | Kimeneti videót: |
|--------|--------|---------|
| <img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/style_image.jpg" width="300"> | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video.mp4 "A videó bemeneti") *kattintson ide a megtekintéshez videó* | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video.mp4 "A videó kimeneti") *kattintson ide a megtekintéshez videó* |

<!-- markdownlint-enable MD033 -->

Ez a referenciaarchitektúra a számítási feladatokat az Azure storage-ban új adathordozó jelenléte által aktivált lett tervezve.

Feldolgozás az alábbi lépésekből áll:

1. Videofájl feltöltése a tárolóba.
1. A videó a fájlalapú eseményindítók egy logikai alkalmazás egy kérést küldhet az Azure Machine Learning közzétett végpont folyamat.
1. A folyamat dolgozza fel a videót, MPI-stílus átviteli vonatkozik, és a videó postprocesses.
1. A kimenet vissza a blob storage-ban a folyamat befejezése után a rendszer menti.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő összetevőkből áll.

### <a name="compute"></a>Compute

**[Az Azure Machine Learning szolgáltatás] [ amls]**  Azure Machine Learning-folyamatokat hozhat létre reprodukálható és könnyen kezelhető feladatütemezések számítási használ. Emellett egy felügyelt számítási célnak (amelyen folyamat számítást futtatható) nevű [Azure Machine Learning Compute] [ aml-compute] képzés, üzembe helyezése és pontozás a machine learning-modellek.

### <a name="storage"></a>Storage

**[A BLOB storage-] [ blob-storage]**  (a bemeneti képekhez, a stílus lemezképek és a kimeneti képek) az összes rendszerkép tárolására szolgál. Az Azure Machine Learning szolgáltatás integrálható a Blob storage, hogy a felhasználók nem rendelkeznek a kézi számítási platform és a Blob storage között az adatok áthelyezéséhez. A BLOB storage akkor is, amely a számítási feladat igényli a teljesítmény rendkívül költséghatékony.

### <a name="trigger--scheduling"></a>Trigger / ütemezése

**[Az Azure Logic Apps] [ logic-apps]**  a munkafolyamat elindítására szolgál. Amikor a logikai alkalmazás észleli, hogy hozzáadta-e egy blobot a tárolóhoz, az Azure Machine Learning folyamat elindítja. A Logic Apps egy jó észleli a módosításokat a blob storage-ban könnyedén, mert ez a referenciaarchitektúra alkalmas, és egy egyszerű folyamatot biztosít az eseményindító módosítása.

### <a name="preprocessing-and-postprocessing-our-data"></a>Előfeldolgozási és az adatok postprocessing

Ez a referenciaarchitektúra egy fa a Képanyag-orangutan használja. Letöltheti a Képanyag a [Itt][source-video].

1. Használat [FFmpeg] [ ffmpeg] a hangfájl kibontani a videoanyagokat, úgy, hogy a hangfájl is lehet stitched a kimeneti videót be újra később.
1. A videó felosztása egyes keretek a FFmpeg segítségével. A keretek egymástól függetlenül, párhuzamosan kerül feldolgozásra.
1. Ezen a ponton azt is alkalmazhat Neurális stílus átviteli minden egyes keret párhuzamosan.
1. Egy egyes keret feldolgozása megtörtént, igazolnia kell a keretek vissza együtt restitch FFmpeg segítségével.
1. Végül azt csatlakoztassa újból a hangfájl a restitched képanyag.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

### <a name="gpu-versus-cpu"></a>Processzor és GPU

A deep learning számítási feladatokhoz gpu-k általánosan élekről végez, és processzorokat sávszélességétől, az processzorok marketingre fürt általában szükség hasonló teljesítmény eléréséhez. Bár a lehetőség csak processzorokat használata ebben az architektúrában, gpu-k egy sokkal jobb költség/teljesítmény profilt biztosít. Azt javasoljuk, hogy a legújabb [NCv3 sorozat] virtuálisgép-méretek – gpu használatával GPU-optimalizált virtuális gépek.

Az összes régióban alapértelmezés szerint nem engedélyezettek a GPU-kkal. Ellenőrizze, hogy válasszon ki egy régiót a GPU-k engedélyezve van. Emellett az előfizetések nulla magok alapértelmezett kvóta tartozik GPU-optimalizált virtuális gépek számára. Nyissa meg a támogatási kérelmet a kvóta is növelheti. Győződjön meg arról, hogy előfizetése rendelkezik-e elegendő kvótával a számítási feladatok futtatásához.

### <a name="parallelizing-across-vms-versus-cores"></a>Magok és a virtuális gépen párhuzamosan futtatni

Batch-feladat egy stílust átviteli folyamat fut, amikor a feladatok elsősorban a GPU-n futó virtuális gépek között méretezésnek megfelelően kell. Két módszer is lehetséges: Egyetlen GPU rendelkező virtuális gépeken nagyobb méretű fürtöt létrehozni, vagy hozzon létre egy kisebb fürtöt használó virtuális gépek sok gpu-kkal.

Ilyen számítási feladatok esetében két lesz hasonló teljesítményt. Virtuális gépenként több gpu-kkal kevesebb virtuális gépek használatával segíthet csökkenteni az adatok áthelyezését. A számítási feladatonként adatmennyiség viszont nem nagyon nagy, ezért nem vizsgálja meg, mennyi szabályozás blob Storage.

### <a name="mpi-step"></a>MPI lépés

Amikor létrehoz egy folyamatot az Azure Machine Learning, a párhuzamos számítások végrehajtásához használt lépéseket egyik az MPI-lépés. Az MPI-lépés segít az adatok egyenletes felosztása elérhető csomópontok között. Az MPI-lépés a rendszer nem hajtotta végre, amíg készen áll a kért csomópontok. Kell egy csomópont sikertelen, vagy a felfüggesztett beolvasása (ha az alacsony prioritású virtuális gép), az MPI lépést kell újra futtatni.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="restricting-access-to-azure-blob-storage"></a>Az Azure blob storage-ba való hozzáférés korlátozása

Ez a referenciaarchitektúra az Azure blob storage a fő tárolási összetevő, amely a védendő. A GitHub-adattárat látható az alapkonfigurációk azon központi telepítését tárfiókkulcsok segítségével a blobtároló eléréséhez. További ellenőrzésére és védelmére fontolja meg egy közös hozzáférésű jogosultságkód (SAS) helyett. Ez a tároló objektumok korlátozott hozzáférést biztosít a anélkül, hogy keményen a fiókkulcsok code, vagy egyszerű szövegként menti. Ez a módszer különösen hasznos, mert fiókkulcsok láthatók a sima szöveges Logic App Tervező felületen belül. SAS használatával is biztosítható, hogy a tárfiók rendelkezik a megfelelő irányítás, és, hogy célja, hogy hozzáférhessenek a mobileszközeiken csak a személyeknek.

Forgatókönyvek a több bizalmas adatok győződjön meg arról, hogy a kulcsok összes védettek, mivel ezek a kulcsok teljes hozzáférési jogot az összes bemeneti és kimeneti adatok a munkaterhelési.

### <a name="data-encryption-and-data-movement"></a>Adattitkosítás és -adatok áthelyezése

Ez a referenciaarchitektúra stílus átvitelt használ példaként szolgál a kötegelt pontozási a folyamatot. További adatok bizalmas forgatókönyvek esetén az adatok a storage-ban kell lennie titkosítása. Minden alkalommal adat áthelyezik az egyik helyről a következő, az SSL használatával biztonságossá tétele az adatátvitel. További információkért lásd: [Azure Storage biztonsági útmutatóját][storage-security].

### <a name="securing-your-computation-in-a-virtual-network"></a>A számítási virtuális hálózat biztonságossá tétele

A Machine Learning számítási fürt üzembe helyezésekor, konfigurálhatja a fürt egy alhálózatán belül üzembe helyezhető egy [virtuális hálózat][virtual-network]. Ez lehetővé teszi a számítási csomópontok, a fürt más virtuális gépek biztonságos kommunikációhoz.

### <a name="protecting-against-malicious-activity"></a>Rosszindulatú tevékenység elleni védelme

A forgatókönyvek, ahol több felhasználó van, győződjön meg arról, hogy kártékony tevékenységek ellen védett-e a bizalmas adatokat. Ha más felhasználók kapjanak hozzáférést ehhez a központi telepítéshez a bemeneti adatok testreszabása, jegyezze fel a következő óvintézkedéseket és szempontok:

- Az RBAC használatával csak a számukra szükséges erőforrásokhoz való felhasználói hozzáférés korlátozására.
- Két különböző storage-fiókok kiépítéséhez. Az első fiók Store a bemeneti és kimeneti adatokat. A külső felhasználók is hozzáférést kell biztosítani ehhez a fiókhoz. Store végrehajtható parancsfájlokat, és a kimeneti naplófájlokat a fiókban. A külső felhasználók nem kell a hozzáférést a fiókjához. Ez biztosítja, hogy a külső felhasználók nem módosítható a végrehajtható fájl (a kártevő kód beszúrása), és nem fér hozzá a logfiles, amelyek bizalmas információkat tartalmazhat.
- Rosszindulatú felhasználók DDOS a feladat-várólistában vagy beszúrása a feladat-várólistában lévő helytelen formátumú ártalmas üzenetek okoz a rendszer zárolja, vagy üzenetmozgatót hibákat okoz.

## <a name="monitoring-and-logging"></a>Monitorozás és naplózás

### <a name="monitoring-batch-jobs"></a>Batch-feladatok figyelése

A feladat futtatásakor fontos nyomon követheti, és győződjön meg arról, hogy a dolgok várt módon működik. Azonban ez több aktív csomópontból álló fürtben figyelése kihívást jelenthet.

A fürt általános állapotáról kaphat, nyissa meg a Machine Learning panel az Azure Portal, a csomópontok a fürt állapotának vizsgálata. Ha egy csomópont nem aktív, vagy egy feladat sikertelen volt, a hibanaplókat blob storage-bA lesznek mentve, és az Azure Portalon is elérhetők.

Figyelés is lehet további bővített naplók csatlakozik az Application Insights vagy a fürt és a feladatok állapotát a lekérdezéséhez külön folyamatokba futtatja.

### <a name="logging-with-azure-machine-learning"></a>Az Azure Machine Learning-naplózás

Az Azure Machine Learning automatikusan projekttárolóba összes stdout/stderr társítása a blob storage-fiókhoz. Ellenkező értelmű rendelkezés az Azure Machine Learning-munkaterület automatikusan kiépíteni a tárfiókot, és a naplók dump bele. A tárolók navigációs eszköz, például a Storage Explorer, amely egy sokkal egyszerűbb megoldást biztosít a naplófájlok navigáláshoz használhatja.

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

A storage és az ütemezési összetevők képest, ez a referenciaarchitektúra a messze használt számítási erőforrások uralja költségei tekintetében. Az egyik fő kihívása van hatékonyan párhuzamosan futtatni a munkahelyi GPU-kompatibilis gép egy fürtön.

A Machine Learning COMPUTE számítási fürt mérete automatikusan méretezhetők felfelé és lefelé a várólistában a feladatok függően. Automatikus vertikális felskálázás programozott módon engedélyezheti a minimális és maximális csomópontok beállításával.

És nem igényel azonnali feldolgozási munka automatikus méretezés beállítása nulla csomópontból álló fürtben tehát az alapértelmezett állapot (legalább). Ezzel a konfigurációval a fürt nullára a csomópontok kezdődik, és ha a várólistán lévő feladatok csak felskálázással. Ha a kötegelt pontozási folyamat csak néhány alkalommal naponta történik vagy annál kisebb, ez a beállítás lehetővé teszi, hogy jelentős költségmegtakarítást.

Automatikus méretezés nem lehet megfelelő kötegelt feladatok számához egymáshoz közel túl fordulhat elő. A fürt számára le üzemeltethet a idejét is költségkezelési, így ha egy batch számítási feladatot csak néhány percet az előző feladat befejezése után kezdődik, költséghatékonyabb tartani a fürtön futó feladatok között lehet.

Machine Learning Compute is támogatja az alacsony prioritású virtuális gépekre. Ez lehetővé teszi, hogy a számítási futtatni, akkor előfordulhat, hogy lehet pre-empted bármikor csoportosítani kedvezményes virtuális gépeken. Alacsony prioritású virtuális gépek ideálisak a nem kritikus fontosságú kötegelt pontozási a számítási feladatokat.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra üzembe helyezéséhez kövesse az ismertetett lépéseket a [GitHub-adattárat][deployment].

> [!NOTE]
> A kötegelt pontozási architektúra deep learning-modellek az Azure Kubernetes Service használatával is telepítheti. A jelen ismertetett lépéseket követve [Github-adattárat][deployment2].

<!-- links -->

[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[container-instances]: /azure/container-instances/
[container-registry]: /azure/container-registry/
[deployment]: https://github.com/Azure/Batch-Scoring-Deep-Learning-Models-With-AML
[deployment2]: https://github.com/Azure/Batch-Scoring-Deep-Learning-Models-With-AKS
[ffmpeg]: https://www.ffmpeg.org/
[image-style-transfer]: https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf
[logic-apps]: /azure/logic-apps/
[source-video]: https://happypathspublic.blob.core.windows.net/videos/orangutan.mp4
[storage-security]: /azure/storage/common/storage-security-guide
[vm-sizes-gpu]: /azure/virtual-machines/windows/sizes-gpu
[virtual-network]: /azure/machine-learning/service/how-to-enable-virtual-network
