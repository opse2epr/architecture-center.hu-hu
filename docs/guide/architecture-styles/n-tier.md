---
title: N szintű architektúra stílus
description: Előnyeit, kihívást és ajánlott eljárások az N szintű architektúrák ismerteti az Azure-on
author: MikeWasson
ms.openlocfilehash: 8333b789e03a9da2b021abe7d7c193cd2af8d6bf
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540377"
---
# <a name="n-tier-architecture-style"></a>N szintű architektúra stílus

Az N szintű architektúrát osztja a az alkalmazás **logikai rétegek** és **fizikai rétegek**. 

![](./images/n-tier-logical.svg)

Rétegek, amelyek egy külön feladatkörök, és függőségeinek kezelése. Minden réteget egy adott felelősséggel tartozik. Egy magasabb réteg akkor használhatják a szolgáltatásokat az alsóbb rétegben, de nem fordítva. 

Rétegek fizikailag elkülönített, külön történő futtatása. A réteg hívja közvetlenül egy másik réteghez, vagy aszinkron üzenetkezelési (üzenet-várólista) használja. Bár az egyes rétegekhez adó saját rétegben, amely nem szükséges. Több azonos tartozó előfordulhat, hogy futhat. A rétegek fizikailag elválasztó fokozza a méretezhetőség és a rugalmasság, de is hozzáadja a további hálózati kommunikáció a késés. 

A hagyományos háromrétegű alkalmazásnak a bemutatási szint, egy középső és adatbázis-rétegből. A középső réteg nem kötelező megadni. Összetettebb alkalmazásokat rendelkezhet több mint három réteg. A fenti ábra mutatja a középső réteg két, különböző területei funkcióit és az alkalmazás. 

N szintű alkalmazás rendelkezhet egy **architektúra lezárt** vagy egy **nyissa meg a architektúra**:

- Lezárt réteg architektúra esetén a réteg csak meghívhatja a következő réteg azonnal. 
- Egy megnyitott rétegű architektúra réteg meghívhatja a rétegek alatta bármelyikét. 

Lezárt architektúra korlátozza a rétegek közötti függőségek. Azonban, előfordulhat, hogy hozzon létre szükségtelen hálózati forgalmat, ha egy réteg egyszerűen továbbítja a kérelmeket a következő réteggel. 

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

N szintű architektúrák általában megvalósítása szolgáltatásként infrastruktúra (IaaS) alkalmazások, és minden egyes réteg futó virtuális gépek külön készletét. Tiszta IaaS kell azonban N szintű kérelmet nem szükséges. Gyakran, célszerű a architektúrára épülő bizonyos részeihez felügyelt szolgáltatásokat használja, különösen gyorsítótárazás, üzenetküldés és az adatokat.

Vegye figyelembe az N szintű architektúrája:

- Egyszerű webes alkalmazásokhoz. 
- A minimális újrabontása Azure a helyszíni alkalmazások áttelepítése.
- Egyesített fejlesztése a helyszíni és felhőalapú alkalmazásokhoz.

N szintű architektúrák gyakran előfordul, a hagyományos helyszínen alkalmazások, ezért természetes alkalmasnak áttelepítése meglévő alkalmazások az Azure-bA.

## <a name="benefits"></a>Előnyök

- Hordozhatósága, valamint felhőalapú és helyszíni között felhőplatformokkal.
- A legtöbb fejlesztőknek kevesebb tanulási görbére.
- A hagyományos alkalmazás modellből természetes alakulása.
- Nyissa meg a heterogén környezetben (Windows/Linux)

## <a name="challenges"></a>Kihívásai

- Végül egy középső, amelyet csak CRUD műveletek az adatbázis extra késés hozzáadása nélkül hasznos munka során könnyebben. 
- Egységes tervezési megakadályozza, hogy a szolgáltatások független telepítését.
- Több munkát csak használó alkalmazások által felügyelt szolgáltatások, mint egy infrastruktúra-szolgáltatási alkalmazás kezelése. 
- Hálózati biztonság kezeléséhez nagy rendszer nehézkes lehet.

## <a name="best-practices"></a>Ajánlott eljárások

- Használja az automatikus skálázás módosítások a terhelés kezelésére. Lásd: [automatikus skálázás gyakorlati tanácsok][autoscaling].
- Aszinkron üzenetkezelési segítségével rétegek használata leválasztja.
- Gyorsítótár félig statikus adatok. Lásd: [gyakorlati tanácsok gyorsítótárazás][caching].
- Konfigurálja a magas rendelkezésre állás, olyan megoldást használni, mint az adatbázis-rétegből [SQL Server Always On rendelkezésre állási csoportok][sql-always-on].
- Helyezze a webalkalmazási tűzfal (waf-ot) az előtér- és az Internet között.
- Minden egyes réteg saját alhálózatba helyezze el, és a biztonsági határ alhálózatok használja. 
- Hozzáférés korlátozása az adatréteg, így csak a középső tier(s) érkező kérelmeket.

## <a name="n-tier-architecture-on-virtual-machines"></a>A virtuális gépek N szintű architektúra

Ez a szakasz ismerteti a virtuális gépeken futó ajánlott N szintű architektúra. 

![](./images/n-tier-physical.png)

Két vagy több virtuális gép, egy rendelkezésre állási csoport vagy a Virtuálisgép-méretezési készlet minden egyes réteg áll. Több virtuális gép biztosít rugalmasságot, abban az esetben, ha egy virtuális gép nem sikerül. A terheléselosztók segítségével kérelmek szétosztását a virtuális gépeket, a réteg. A számítógépréteg bővíthető vízszintesen további virtuális gépek hozzáadásával a készlethez. 

Minden egyes réteg is helyezhetők saját alhálózatba, ami azt jelenti, a belső IP-címek azonos cím tartományba esik. Amely megkönnyíti a hálózati biztonsági csoport (NSG) szabályok alkalmazása és az egyes rétegek útvonaltáblát.

A web és business szintek olyan állapot nélküli. A virtuális gép képes kezelni az adott réteg kérésének. Az adatréteg replikált adatbázis kell állnia. Windows SQL Server Always On rendelkezésre állási csoportok használatával a magas rendelkezésre állás érdekében javasolt. A Linux válasszon, amely támogatja a replikációt, például az Apache Cassandra adatbázist. 

Hálózati biztonsági csoportokkal (NSG-k) korlátozhatja a hozzáférést az egyes rétegek. Az adatbázis-rétegből például csak az üzleti szint való hozzáférést lehetővé teszi.

További részletekért és telepíthető Resource Manager-sablon tekintse meg a következő hivatkozás architektúrák:

- [Futtassa a Windows virtuális gépek N szintű alkalmazás][n-tier-windows]
- [Linux virtuális gépek futtatásához N szintű alkalmazások][n-tier-linux]

### <a name="additional-considerations"></a>Néhány fontos megjegyzés

- N szintű architektúrák három réteg nem korlátozódnak. Összetett alkalmazások gyakori, hogy olyan további rétegek. Ebben az esetben érdemes lehet egy adott szint kérelmek útválasztási réteg – 7.

- Rétegek a méretezhetőség, megbízhatóság és biztonsági határ. Érdemes külön rétegek eltérő követelmények vonatkoznak a szolgáltatások azokon a területeken.

- Virtuálisgép-méretezési készlet használja az automatikus skálázást.

- Keresse meg az architektúra, ahol ugyanúgy használhatók egy felügyelt szolgáltatás nélkül jelentős újrabontása helyek. Különösen tekintse meg gyorsítótárazás, üzenetküldés, a tároló és az adatbázisok. 

- A nagyobb biztonság érdekében helyezze el a hálózati DMZ alkalmazása előtt. A Szegélyhálózaton a hálózati virtuális készülékek (NVAs), amelyek megvalósítják a biztonsági funkciók, például tűzfalak és Csomagvizsgálat tartalmazza. További információkért lásd: [hálózati DMZ referencia-architektúrában][dmz].

- A magas rendelkezésre állás érdekében helyezze két, vagy egy rendelkezésre állási további NVAs internetes kérelmek szét a példányok külső terheléselosztással állítsa be. További információkért lásd: [telepíteni a magas rendelkezésre állású hálózati virtuális készülékek][ha-nva].

- Nincs engedélyezve a közvetlen RDP és az SSH hozzáférés alkalmazáskód futtató virtuális gépekhez. Ehelyett operátorok kell bejelentkezni egy jumpbox, más néven a megerősített gazdagépet. Ez a virtuális gép, amely a rendszergazdák csatlakozhat a virtuális gépeket a hálózaton. A jumpbox rendelkezik, amely lehetővé teszi az RDP és az SSH csak a jóváhagyott nyilvános IP-címek egy NSG.

- A helyszíni hálózatra a pont-pont virtuális magánhálózat (VPN) vagy Azure ExpressRoute kiterjesztheti az Azure virtuális hálózat. További információkért lásd: [hibrid hálózati referencia-architektúrában][hybrid-network].

- Ha a szervezet Active Directoryt identitás kezelése, érdemes lehet terjessze ki az Active Directory-környezet az Azure virtuális hálózatot. További információkért lásd: [Identity management referencia-architektúrában][identity].

- Ha kell, mint a virtuális gépek az Azure garantált szolgáltatási szintje itt magasabb rendelkezésre állását, az alkalmazás replikálása két régióban is, és a feladatátvételre használni az Azure Traffic Manager. További információkért lásd: [futtassa a Windows virtuális gépek több régióba] [ multiregion-windows] vagy [futtatása a Linux virtuális gépek több régióba][multiregion-linux].

[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[dmz]: ../../reference-architectures/dmz/index.md
[ha-nva]: ../../reference-architectures/dmz/nva-ha.md
[hybrid-network]: ../../reference-architectures/hybrid-networking/index.md
[identity]: ../../reference-architectures/identity/index.md
[multiregion-linux]: ../../reference-architectures/virtual-machines-linux/multi-region-application.md
[multiregion-windows]: ../../reference-architectures/virtual-machines-windows/multi-region-application.md
[n-tier-linux]: ../../reference-architectures/virtual-machines-linux/n-tier.md
[n-tier-windows]: ../../reference-architectures/virtual-machines-windows/n-tier.md
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server