---
title: Nagy számítási architektúra stílus
description: Előnyeit, kihívást és ajánlott eljárások az architektúrák nagy számítási ismerteti az Azure-on
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="big-compute-architecture-style"></a>Nagy számítási architektúra stílus

A kifejezés *nagy számítási* nagyméretű mag, gyakran száma akár a több száz vagy ezer a nagy számú igénylő munkaterhelések ismerteti. Forgatókönyvek például a kép megjelenítési, folyadékból dynamics, pénzügyi kockázat modellezési, olaj feltárása, gyógyszerfejlesztés és mérnöki magas terhelés elemző, többek között.

![](./images/big-compute-logical.png)

Az alábbiakban néhány tipikus nagy számítási alkalmazások jellemzői:

- A munkahelyi felosztható diszkrét feladatokat, melyekhez egyidejűleg több mag között futtatható.
- Minden feladat azonban véges. Az egyes bemenetből fogad adatokat, nem az egyes feldolgozási, és kimenetet. A teljes alkalmazás futtatása véges időtartama (perc nap). A megszokott mintát követi, hogy osszon ki a továbbítás magok nagy mennyiségű, és le egyszer nulla léptetéses az alkalmazás végrehajtja. 
- Az alkalmazás nem kell mentése 24/7 maradnak. A rendszer azonban csomópontok hibáit vagy az alkalmazás összeomlik kell kezelni.
- Az egyes alkalmazásokra a feladatok független, és a párhuzamosan futtatható. Más esetekben feladatok szorosan összekapcsolt, tehát kell módosításának vagy köztes eredmények exchange. Ebben az esetben érdemes lehet a nagy sebességű hálózatkezelési technológiákat, például az InfiniBand és a távoli közvetlen memória-hozzáférés (RDMA). 
- Attól függően, hogy a munkaterhelés használhatja a számítási igényű Virtuálisgép-méretek (H16r H16mr és a9-es).

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

- Számítástechnikai számításigényes műveletek, például szimuláció és szám crunching.
- Számítástechnikai intenzív és több számítógépet (10-1000 egység) a processzorok között kell osztani szimulációja.
- Túl sok memóriát igényelnek egy számítógép, amely több számítógép között meg kell osztani szimulációja.
- Hosszan futó számítások, az adott túl sokáig tartott egyetlen számítógépen.
- Kisebb számítások, amelyek 100-as egység vagy idő szerint, például a Monte Carlo szimulációja 1000 egység kell futtatni.

## <a name="benefits"></a>Előnyök

- Nagy teljesítményt és az "[embarrassingly párhuzamos][embarrassingly-parallel]" feldolgozása.
- Is hasznosítására több száz vagy ezer számítógép magok gyorsabban nagy problémák megoldására.
- Speciális nagy teljesítményű hardverre, dedikált nagy sebességű InfiniBand hálózatokkal való hozzáférést.
- Virtuális gépek építhető megfelelően működik, és szakadjon, majd el őket. 

## <a name="challenges"></a>Kihívásai

- A virtuális gép infrastruktúra kezelése.
- A szám crunching mennyisége kezelése. 
- Magok ezer kiépítése időben.
- Szorosan összekapcsolt feladatokhoz több maggal is jár diminishing értéket ad vissza. Szükség lehet az optimális magok száma található kipróbálásához.

## <a name="big-compute-using-azure-batch"></a>Nagy számítási az Azure Batch használatával

[Az Azure Batch] [ batch] egy felügyelt szolgáltatás nagy méretű nagy teljesítményű számítástechnikai rendszerek (HPC)-alkalmazások futtatására.

Azure Batch használatával, konfigurálja a VM-készletben, és az alkalmazások és adatok fájlok feltöltése. Majd a Batch szolgáltatás kiosztja a virtuális gépek, feladatokat rendelhet a virtuális gépek, futtatja a feladatok és kíséri. Kötegelt automatikusan lehet terjeszteni a virtuális gépeket, a munkaterhelés adott válaszként. Kötegelt is biztosít a feladatütemezés.

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a>Nagy virtuális gépeken futó számítási

Használhat [Microsoft HPC Pack] [ hpc-pack] virtuális gépek, a fürt felügyeletéhez és ütemezéséhez és figyelése a HPC feladatokat. Ezt a módszert kell kiépíteni, és a virtuális gépek és a hálózati infrastruktúra kezeléséhez. Fontolja meg ezt a módszert használja, ha a meglévő HPC munkaterhelésekkel rendelkeznek, és szeretne áttérni a tanúsítványszolgáltatás néhány vagy az összes informatikai az Azure-bA. A teljes HPC-fürt áthelyezése az Azure-ba, vagy tartsa a HPC fürt a helyszínen, de Azure kapacitásnövelés kapacitás használja. További információkért lásd: [számítási munkaterhelés nagy méretű kötegelt és HPC megoldások][batch-hpc-solutions].

### <a name="hpc-pack-deployed-to-azure"></a>Az Azure rendszerbe telepítendő HPC Pack

Ebben a forgatókönyvben a HPC-fürt egészében Azure jön létre.

![](./images/big-compute-iaas.png) 
 
Az átjárócsomópont kezelést és a feladatütemezés a fürt szolgáltatásokat biztosítja. Szorosan összekapcsolt feladatokhoz használja az RDMA hálózati nagyon nagy sávszélesség, virtuális gépek kis késleltetésű kommunikációját biztosító. További információ: [HPC Pack 2016-fürt üzembe helyezése az Azure-ban][deploy-hpc-azure].

### <a name="burst-an-hpc-cluster-to-azure"></a>HPC-fürt löketszerű átvitele az Azure-ba

Ebben a forgatókönyvben a szervezet helyszíni HPC Pack fut, és használja az Azure virtuális gépek kapacitásnövelés kapacitást. A fürt átjárócsomópontjába helyszíni. Expressroute-on vagy VPN-átjárót a helyszíni hálózat csatlakozik az Azure virtuális hálózatot.

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
