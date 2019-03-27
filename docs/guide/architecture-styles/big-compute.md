---
title: A Big Compute architektúrastílus
titleSuffix: Azure Application Architecture Guide
description: Előnyeit, kihívásait és ajánlott eljárások a Big Compute-architektúrák ismerteti az Azure-ban.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19, HPC
ms.openlocfilehash: 56bd2ce010b56880e769ada4c6397391a73bbd1e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299045"
---
# <a name="big-compute-architecture-style"></a>A Big Compute architektúrastílus

A *Big Compute* kifejezés olyan nagyméretű számítási feladatokat jelent, amelyekhez nagy mennyiségű, akár a több száz vagy több ezer mag szükséges. Ilyen forgatókönyv például a képrenderelés, a folyadékdinamika, a pénzügyi kockázatok modellezése, az olajfeltárás, a gyógyszerfejlesztés vagy a műszaki feszültségelemzés.

![Big compute architektúrastílus logikai diagramja](./images/big-compute-logical.png)

A Big Compute-alkalmazások néhány jellemzője:

- A munka felosztható önálló feladatokra, amelyek egyidejűleg futtathatók több magon.
- Mindegyik feladat véges. Bemeneti adatokat igényel, feldolgozást végez és létrehoz egy kimenetet. A teljes alkalmazás véges ideig fut (ez néhány perctől napokig terjedhet). Gyakori minta nagy mennyiségű mag kiépítése egyszerre, majd az alkalmazás befejeződésével a magok számának nullára csökkentése.
- Az alkalmazásnak nem kell folyamatosan futnia. Azonban a rendszernek tudnia kell kezelni a csomóponthibákat és az alkalmazás-összeomlásokat.
- Egyes alkalmazások esetében a feladatok egymástól függetlenek, és párhuzamosan futtathatók. Más esetekben a feladatok szorosan kapcsolódnak egymáshoz, azaz kommunikálniuk kell és meg kell osztaniuk egymással a köztes eredményeket. Ebben az esetben érdemes olyan nagy sebességű hálózatkezelési technológiákat használni, mint az InfiniBand, és a távoli közvetlen memória-hozzáférés (RDMA).
- A számítási feladattól függően nagy számítási igényekre szabott virtuálisgép-méretek is használhatók (H16r, H16mr és A9).

## <a name="when-to-use-this-architecture"></a>Mikor érdemes ezt az architektúrát használni?

- Nagy számítási igényű műveletek esetében, mint a szimuláció és a számokkal való munka.
- Nagy számítási igényű szimulációk esetében, amelyeket fel kell osztani több számítógép és több CPU között (tíztől több ezres nagyságrendig).
- Olyan szimulációk esetében, amelyeknek túl nagy a memóriaigénye egyetlen számítógép számára, ezért fel kell őket osztani több számítógép között.
- Hosszan futó számítások esetében, amelyeket egyetlen számítógépen túl sokáig tartana elvégezni.
- Kisebb számítások esetében, amelyeket több száz vagy több ezer alkalommal el kell végezni, ilyenek például a Monte Carlo-szimulációk.

## <a name="benefits"></a>Előnyök

- Nagy teljesítmény és „[zavaróan párhuzamos][embarrassingly-parallel]” feldolgozás.
- Egyszerre több száz vagy több ezer számítógépmag hasznosítása a nagyobb problémák gyorsabb megoldásához.
- Hozzáférés speciális nagy teljesítményű hardverekhez dedikált nagy sebességű InfiniBand-hálózatokkal.
- A virtuális gépek igény szerint helyezhetők üzembe és építhetők le.

## <a name="challenges"></a>Problémák

- A virtuálisgép-infrastruktúra kezelése.
- Számokkal mennyiségének kezelése
- Többezer mag időben történő kiépítése.
- Szorosan összekapcsolt feladatok esetén több mag hozzáadása ronthatja a teljesítményt. A magok megfelelő számának megtalálása lehet, hogy kísérletezést igényel.

## <a name="big-compute-using-azure-batch"></a>Big Compute az Azure Batch használatával

Az [Azure Batch][batch] egy felügyelt szolgáltatás, amellyel nagyméretű és nagy teljesítményű feldolgozási (HPC) alkalmazásokat futtathat.

Az Azure Batch használatakor konfigurálnia kell egy VM-készletet, és fel kell töltenie az alkalmazásokat és adatfájlokat. Ezután a Batch szolgáltatás kiosztja a virtuális gépeket, feladatokat rendel azokhoz, futtatja a feladatokat, és monitorozza az állapotot. A Batch a számítási feladat igényeinek megfelelően automatikusan méretezi a virtuális gépeket. A Batch emellett a feladatütemezésről is gondoskodik.

![A big compute az Azure Batch segítségével ábrája](./images/big-compute-batch.png)

## <a name="big-compute-running-on-virtual-machines"></a>Big Compute a virtuális gépeken

A [Microsoft HPC Packkel][hpc-pack] felügyelhet egy virtuális gépekből álló fürtöt, valamint ütemezheti és monitorozhatja a HPC-feladatokat. Ennél a módszernél az Ön feladata a virtuális gépek és a hálózati infrastruktúra kiépítése és felügyelete. A módszert akkor érdemes használni, ha a meglévő HPC számítási feladatait vagy azok egy részét át szeretné helyezni az Azure-ba. Áthelyezheti a teljes HPC-fürtöt az Azure-ba, vagy megtarthatja a fürtöt a helyszínen és az Azure-t használhatja a kapacitáscsúcsok esetén. További információkért lásd: [Batch- és HPC-megoldások nagyméretű számítási feladatokhoz][batch-hpc-solutions].

### <a name="hpc-pack-deployed-to-azure"></a>Az Azure-ban telepített HPC Pack

Ebben a forgatókönyvben a HPC-fürtöt teljes egészében az Azure-ban hozzuk létre.

![Az Azure-ban telepített HPC Pack ábrája](./images/big-compute-iaas.png)

Az átjárócsomópont felügyeleti és feladatütemezési szolgáltatásokat nyújt a fürthöz. Szorosan összekapcsolt feladatokhoz használjon egy RDMA-hálózatot, amely rendkívül nagy sávszélességű, kis késleltetésű kommunikációt biztosít a virtuális gépek között. További információkat a [HPC Pack 2016-fürt az Azure-ban való üzembe helyezését][deploy-hpc-azure] ismertető cikkben olvashat.

### <a name="burst-an-hpc-cluster-to-azure"></a>HPC-fürt löketszerű átvitele az Azure-ba

Ebben a forgatókönyvben a vállalat a helyszínen futtatja a HPC Packet, az Azure-beli virtuális gépeket pedig a kapacitáscsúcsok esetén használja. A fürt helyszíni átjárócsomóponttal rendelkezik. A helyszíni hálózat ExpressRoute vagy VPN Gateway használatával csatlakozik az Azure VNethez.

![Hibrid big compute-fürt ábrája](./images/big-compute-hybrid.png)

<!-- links -->

[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029
