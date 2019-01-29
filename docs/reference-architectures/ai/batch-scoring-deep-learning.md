---
title: A mélytanulási modellek kötegelt kiértékelése
titleSuffix: Azure Reference Architectures
description: Ez a referenciaarchitektúra bemutatja, hogyan Neurális stílus átviteli alkalmazandó egy videót, az Azure Batch AI segítségével.
author: jiata
ms.date: 10/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 27975b42179e87f4520186778610159943a93090
ms.sourcegitcommit: 40f3561cc94f721eca50d33f2d75dc974cb6f92b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/29/2019
ms.locfileid: "55147246"
---
# <a name="batch-scoring-on-azure-for-deep-learning-models"></a>Kötegelt pontozási az Azure-on deep learning-modellek

Ez a referenciaarchitektúra bemutatja, hogyan Neurális stílus átviteli alkalmazandó egy videót, az Azure Batch AI segítségével. *Átviteli stílus* deep learning-módszer, amely egy másik képet stílusát egy meglévő kép composes. Ez az architektúra bármilyen forgatókönyvhöz kötegelt pontozási használatával deep learninget használó is általánosítva. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Az Azure Batch AI segítségével mély tanulási modelleket architektúra diagramja](./_images/batch-ai-deep-learning.png)

**A forgatókönyv**: Egy media szervezete egy videót, amelynek stílusát, így jelenik meg szeretne, például egy adott festmény. A szervezet szeretne tudni a stílus alkalmazása időben, és automatizált módon zajlik a videó minden keretet. További ismereteket a Neurális stílus átviteli algoritmusok, lásd: [kép stílus átvitel használata Konvolúciós Neurális hálózatokkal] [ image-style-transfer] (PDF).

| Style-lemezképet: | A videó bemeneti/tartalom: | Kimeneti videót: |
|--------|--------|---------|
| <img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/style_image.jpg" width="300"> | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/input_video.mp4 "A videó bemeneti") *kattintson ide a megtekintéshez videó* | [<img src="https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video_image_0.jpg" width="300" height="300">](https://happypathspublic.blob.core.windows.net/assets/batch_scoring_for_dl/output_video.mp4 "A videó kimeneti") *kattintson ide a megtekintéshez videó* |

Ez a referenciaarchitektúra a számítási feladatokat az Azure storage-ban új adathordozó jelenléte által aktivált lett tervezve. Feldolgozás az alábbi lépésekből áll:

1. Töltsön fel egy kijelölt stílus képet (például egy Van Gogh festmény) és a egy stílust parancsfájl átvitele a Blob Storage.
1. Hozzon létre egy automatikus skálázást egy Batch AI-fürtöt, amely megkezdheti a munkát véve.
1. Ossza fel a videófájlt egyes keretek, és azokat bármely feltöltése a Blob Storage-bA.
1. Az összes keretek feltöltése után eseményindító fájl feltöltése a Blob Storage.
1. Ez a fájl egy logikai alkalmazást, amely létrehoz egy tárolót az Azure Container Instances szolgáltatásban futó aktivál.
1. A tároló fut egy parancsfájl, amely a Batch AI-feladatot hoz létre. Minden egyes feladat a Neurális stílus átviteli párhuzamosan a Batch AI-fürt csomópontjai között vonatkozik.
1. Miután a képek jönnek létre, a rendszer menti a Blob Storage-ba való.
1. Töltse le a létrehozott keretek, és vissza a képek, videó több.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő összetevőkből áll.

### <a name="compute"></a>Compute

**[Az Azure Batch AI] [ batch-ai]**  a Neurális stílus átviteli algoritmus futtatására szolgál. A Batch AI támogatja a deep learning számítási feladatok azáltal, hogy előre konfigurált deep learning keretrendszerek GPU-kompatibilis virtuális gépeken a tárolóalapú környezetekben. A Batch AI a számítási fürt is csatlakozhat a Blob storage.

> [!NOTE]
> Kivonás alatt áll az Azure Batch AI szolgáltatás márciusi 2019, és az ipari méretekben képzés és pontozás képességek érhetők el mostantól [Azure Machine Learning szolgáltatás][amls]. Ez a referenciaarchitektúra hamarosan frissül majd használni a Machine Learning, így az úgynevezett felügyelt számítási célt [Azure Machine Learning Compute] [ aml-compute] képzés, üzembe helyezése és pontozás a machine tanulási modelleket.

### <a name="storage"></a>Storage

**[A BLOB storage-] [ blob-storage]**  tárolására szolgál az összes rendszerkép (a bemeneti képekhez, a stílus lemezképek és a kimeneti képek), a Batch AI előállított továbbá naplókat. A Batch AI segítségével integrálja a BLOB storage [blobfuse][blobfuse], egy nyílt forráskódú virtuális fájlrendszer, amely a Blob storage használatával. A BLOB storage akkor is, amely a számítási feladat igényli a teljesítmény rendkívül költséghatékony.

### <a name="trigger--scheduling"></a>Trigger / ütemezése

**[Az Azure Logic Apps] [ logic-apps]**  a munkafolyamat elindítására szolgál. Amikor a logikai alkalmazás észleli, hogy hozzáadta-e egy blobot a tárolóhoz, a Batch AI-folyamat elindítja. A Logic Apps egy jó észleli a módosításokat a blob storage-ban könnyedén, mert ez a referenciaarchitektúra alkalmas, és egy egyszerű folyamatot biztosít az eseményindító módosítása.

**[Az Azure Container Instances] [ container-instances]**  , amely a Batch AI-feladatok létrehozása a Python-szkriptek futtatására szolgál. Ezek a szkriptek egy Docker-tárolóban futó kényelmes módja a igény szerinti futtatásához. Ebben az architektúrában használ Container Instances, mert egy előre elkészített logikai alkalmazás-összekötő, amely lehetővé teszi, hogy a logikai alkalmazás aktiválásához a Batch AI-feladat. Container Instances szolgáltatásban hozhatók létre újabb webpéldányok be állapot nélküli folyamatok gyors.

**[DockerHub] [ dockerhub]**  tárolja a Docker-rendszerképet, amely a Container Instances segítségével hajtsa végre a feladat létrehozását. Az architektúra DockerHub választott, mert könnyen használható és az alapértelmezett lemezkép adattár a Docker-felhasználók. [Az Azure Container Registry] [ container-registry] is használható.

### <a name="data-preparation"></a>Adatok előkészítése

Ez a referenciaarchitektúra egy fa a Képanyag-orangutan használja. Letöltheti a Képanyag a [Itt] [ source-video] és feldolgozása a munkafolyamat az alábbi lépéseket:

1. Használat [AzCopy] [ azcopy] töltheti le a videót a nyilvános blob.
2. Használat [FFmpeg] [ ffmpeg] csomagolnia hang, úgy, hogy a hangfájl is lehet stitched a kimeneti videót be újra később.
3. A videó felosztása egyes keretek a FFmpeg segítségével. A keretek egymástól függetlenül, párhuzamosan kerül feldolgozásra.
4. Az AzCopy használatával az egyes keretek másolhatja a blobtárolóba.

Ezen a ponton a videoanyagokat Neurális stílus adatátvitelhez használható formátumban van.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

### <a name="gpu-vs-cpu"></a>GPU-és CPU

A deep learning számítási feladatokhoz gpu-k általánosan élekről végez, és processzorokat sávszélességétől, az processzorok marketingre fürt általában szükség hasonló teljesítmény eléréséhez. Bár a lehetőség csak processzorokat használata ebben az architektúrában, gpu-k egy sokkal jobb költség/teljesítmény profilt biztosít. Azt javasoljuk, hogy a legújabb [NCv3 sorozat] virtuálisgép-méretek – gpu használatával GPU-optimalizált virtuális gépek.

Az összes régióban alapértelmezés szerint nem engedélyezettek a GPU-kkal. Ellenőrizze, hogy válasszon ki egy régiót a GPU-k engedélyezve van. Emellett az előfizetések nulla magok alapértelmezett kvóta tartozik GPU-optimalizált virtuális gépek számára. Nyissa meg a támogatási kérelmet a kvóta is növelheti. Győződjön meg arról, hogy előfizetése rendelkezik-e elegendő kvótával a számítási feladatok futtatásához.

### <a name="parallelizing-across-vms-vs-cores"></a>Virtuális gépek és magok közötti párhuzamosan futtatni

Batch-feladat egy stílust átviteli folyamat fut, amikor a feladatok elsősorban a GPU-n futó virtuális gépek között méretezésnek megfelelően kell. Két módszer is lehetséges: Egyetlen GPU rendelkező virtuális gépeken nagyobb méretű fürtöt létrehozni, vagy hozzon létre egy kisebb fürtöt használó virtuális gépek sok gpu-kkal.

Ilyen számítási feladatok esetében két lesz hasonló teljesítményt. Virtuális gépenként több gpu-kkal kevesebb virtuális gépek használatával segíthet csökkenteni az adatok áthelyezését. A számítási feladatonként adatmennyiség viszont nem nagyon nagy, ezért nem vizsgálja meg, mennyi szabályozás blob Storage.

### <a name="images-batch-size-per-batch-ai-job"></a>Képek kötegméret a Batch AI feladatonként

Egy másik, amelyet be kell állítani paramétere a Batch AI feladatonként feldolgozni képek számát. Másrészről szeretné győződjön meg arról, hogy munkahelyi széles körben között megoszlik a csomópontok és, hogy ha egy feladat sikertelen volt, nem kell újra túl sok lemezképek. Több Batch AI-feladatok, és így feladatonként feldolgozni lemezképek száma kevés, amely mutat. Másrészről Ha túl kevés lemezképek feladatonként feldolgozása, a telepítő/indítási idejének válik aránytalanul nagy. Beállíthatja, hogy a fürtben található csomópontok maximális száma egyenlő feladatok számát. Ez lesz a legjobb teljesítménnyel rendelkezőt feltételezve, hogy nincs feladat nem sikerült, mert minimalizálja a telepítő/indítási költséget. Előfordulhat azonban, ha egy feladat sikertelen volt, egy képeket nagy számú kell futását újra.

### <a name="file-servers"></a>Fájlkiszolgálók

Batch AI használata esetén kiválaszthatja a forgatókönyvhöz szükséges átviteli függően többféle tárolási lehetőség. Az alacsony átviteli sebességet megkövetelő számítási feladatokhoz (keresztül blobfuse) a blob storage-dzsal legyen elegendő. Azt is megteheti a Batch AI is támogatja a Batch AI-fájlkiszolgáló, egy felügyelt egycsomópontos NFS, amely a feladatok egy központilag elérhető tárhelyet biztosít a fürtcsomópontokon automatikusan csatlakoztathatók. A legtöbb esetben csak egy fájlkiszolgáló van szükség a munkaterületen, és meg is szét az adatokat a betanítási feladatokhoz más könyvtárban. Ha egy egycsomópontos NFS nem megfelelő, a számítási feladatokhoz, Batch AI támogatja a más tárolási lehetőségeket, beleértve az Azure Files és az egyéni megoldások, például egy Gluster vagy Lustre fájlrendszer.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="restricting-access-to-azure-blob-storage"></a>Az Azure blob storage-ba való hozzáférés korlátozása

Ez a referenciaarchitektúra az Azure blob storage a fő tárolási összetevő, amely a védendő. A GitHub-adattárat látható az alapkonfigurációk azon központi telepítését tárfiókkulcsok segítségével a blobtároló eléréséhez. További ellenőrzésére és védelmére fontolja meg egy közös hozzáférésű jogosultságkód (SAS) helyett. Ez a tároló objektumok korlátozott hozzáférést biztosít a anélkül, hogy keményen a fiókkulcsok code, vagy egyszerű szövegként menti. Ez a módszer különösen hasznos, mert fiókkulcsok láthatók a sima szöveges Logic App Tervező felületen belül. SAS használatával is biztosítható, hogy a tárfiók rendelkezik a megfelelő irányítás, és, hogy célja, hogy hozzáférhessenek a mobileszközeiken csak a személyeknek.

Forgatókönyvek a több bizalmas adatok győződjön meg arról, hogy a kulcsok összes védettek, mivel ezek a kulcsok teljes hozzáférési jogot az összes bemeneti és kimeneti adatok a munkaterhelési.

### <a name="data-encryption-and-data-movement"></a>Adattitkosítás és -adatok áthelyezése

Ez a referenciaarchitektúra stílus átvitelt használ példaként szolgál a kötegelt pontozási a folyamatot. További adatok bizalmas forgatókönyvek esetén az adatok a storage-ban kell lennie titkosítása. Minden alkalommal adat áthelyezik az egyik helyről a következő, az SSL használatával biztonságossá tétele az adatátvitel. További információkért lásd: [Azure Storage biztonsági útmutatóját][storage-security].

### <a name="securing-data-in-a-virtual-network"></a>A virtuális hálózatban az adatok védelme

A Batch AI-fürt üzembe helyezésekor, konfigurálhatja a fürt egy virtuális hálózat alhálózatán belül ki kell építeni. Ez lehetővé teszi a számítási csomópontok, a fürt más virtuális gépekkel, vagy egy helyszíni hálózat mellett is biztonságosan kommunikál. Is [szolgáltatásvégpontokat] [ service-endpoints] a blob storage-hozzáférés biztosítása virtuális hálózatról, vagy a virtuális hálózaton belül egy egycsomópontos NFS használata a Batch AI, győződjön meg arról, hogy az adatok mindig védve van.

### <a name="protecting-against-malicious-activity"></a>Rosszindulatú tevékenység elleni védelme

A forgatókönyvek, ahol több felhasználó van, győződjön meg arról, hogy kártékony tevékenységek ellen védett-e a bizalmas adatokat. Ha más felhasználók kapjanak hozzáférést ehhez a központi telepítéshez a bemeneti adatok testreszabása, jegyezze fel a következő óvintézkedéseket és szempontok:

- Az RBAC használatával csak a számukra szükséges erőforrásokhoz való felhasználói hozzáférés korlátozására.
- Két különböző storage-fiókok kiépítéséhez. Az első fiók Store a bemeneti és kimeneti adatokat. A külső felhasználók is hozzáférést kell biztosítani ehhez a fiókhoz. Store végrehajtható parancsfájlokat, és a kimeneti naplófájlokat a fiókban. A külső felhasználók nem kell a hozzáférést a fiókjához. Ez biztosítja, hogy a külső felhasználók nem módosítható a végrehajtható fájl (a kártevő kód beszúrása), és nem fér hozzá a logfiles, amelyek bizalmas információkat tartalmazhat.
- Rosszindulatú felhasználók DDOS a feladat-várólistában vagy beszúrása a feladat-várólistában lévő helytelen formátumú ártalmas üzenetek okoz a rendszer zárolja, vagy üzenetmozgatót hibákat okoz.

## <a name="monitoring-and-logging"></a>Monitorozás és naplózás

### <a name="monitoring-batch-ai-jobs"></a>Batch AI-feladatok figyelése

A feladat futtatásakor fontos nyomon követheti, és győződjön meg arról, hogy a dolgok várt módon működik. Azonban ez több aktív csomópontból álló fürtben figyelése kihívást jelenthet.

Megtapasztalhatja, a fürt általános állapotát, lépjen a Batch AI panel az Azure Portal vizsgálhatja meg a fürt csomópontjainak állapotát. Ha egy csomópont nem aktív, vagy egy feladat sikertelen volt, a hibanaplókat blob storage-bA lesznek mentve, és is elérhetők az Azure portálon a feladatok panelen.

Figyelés is lehet további bővített naplók csatlakozik az Application Insights vagy a Batch AI-fürtöt és a feladatok állapotának lekérdezéséhez külön folyamatokba futtatja.

### <a name="logging-in-batch-ai"></a>A Batch AI-naplózás

A Batch AI automatikusan projekttárolóba összes stdout/stderr társítása a blob storage-fiókhoz. Egy tárolási navigációs eszköz, például a Storage Explorer használatával való navigáláshoz naplófájlokat egy sokkal könnyebb élményt biztosít.

Ez a referenciaarchitektúra üzembe helyezési lépései is bemutatja, hogyan állíthat be egy több egyszerű naplózási rendszerben, hogy a különböző feladatok között a naplók ugyanabba a könyvtárba a blobtárolókban található lesznek mentve, ahogy az alábbi. Ezek a naplók segítségével figyelheti, mennyi ideig tart minden egyes rendszerképek és feldolgozni. Ekkor kap egy jobban érti, hogyan optimalizálható a folyamat tovább.

![Bejelentkezés az Azure Batch AI szolgáltatással képernyőképe](./_images/batch-ai-logging.png)

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

A storage és az ütemezési összetevők képest, ez a referenciaarchitektúra a messze használt számítási erőforrások uralja költségei tekintetében. Az egyik fő kihívása van hatékonyan párhuzamosan futtatni a munkahelyi GPU-kompatibilis gép egy fürtön.

A Batch AI-fürt mérete automatikusan méretezhetők felfelé és lefelé a várólistában a feladatok függően. Automatikus méretezés a Batch AI és a két módszer egyikével engedélyezhető. Megteheti szoftveresen, amely konfigurálható a `.env` fájlt, amely része a [üzembe helyezési lépések][deployment], vagy módosíthatja a méretezési képletet közvetlenül a portálon, miután a fürt létrehozott.

És nem igényel azonnali feldolgozási munka konfigurálja az automatikus méretezési képletet, hogy az alapértelmezett állapot (legalább) az nulla csomópontból álló fürtben. Ezzel a konfigurációval a fürt nullára a csomópontok kezdődik, és ha a várólistán lévő feladatok csak felskálázással. Ha a kötegelt pontozási folyamat csak néhány alkalommal naponta történik vagy annál kisebb, ez a beállítás lehetővé teszi, hogy jelentős költségmegtakarítást.

Automatikus méretezés nem lehet megfelelő kötegelt feladatok számához egymáshoz közel túl fordulhat elő. A fürt számára le üzemeltethet a idejét is költségkezelési, így ha egy batch számítási feladatot csak néhány percet az előző feladat befejezése után kezdődik, költséghatékonyabb tartani a fürtön futó feladatok között lehet.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra üzembe helyezéséhez kövesse az ismertetett lépéseket a [GitHub-adattárat][deployment].

<!-- links -->

[aml-compute]: /azure/machine-learning/service/how-to-set-up-training-targets#amlcompute
[amls]: /azure/machine-learning/service/overview-what-is-azure-ml
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[batch-ai]: /azure/batch-ai/
[blobfuse]: https://github.com/Azure/azure-storage-fuse
[blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[container-instances]: /azure/container-instances/
[container-registry]: /azure/container-registry/
[deployment]: https://github.com/Azure/batch-scoring-for-dl-models
[dockerhub]: https://hub.docker.com/
[ffmpeg]: https://www.ffmpeg.org/
[image-style-transfer]: https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf
[logic-apps]: /azure/logic-apps/
[service-endpoints]: /azure/storage/common/storage-network-security?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network
[source-video]: https://happypathspublic.blob.core.windows.net/videos/orangutan.mp4
[storage-security]: /azure/storage/common/storage-security-guide
[vm-sizes-gpu]: /azure/virtual-machines/windows/sizes-gpu
