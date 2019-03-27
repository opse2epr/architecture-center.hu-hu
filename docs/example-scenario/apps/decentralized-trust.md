---
title: Bankok közötti nem központosított megbízhatósági kapcsolatok
titleSuffix: Azure Example Scenarios
description: Megbízható kommunikációs és adatmegosztó környezetet hozhat létre anélkül, hogy egy központosított adatbázisra kellene hagyatkoznia.
author: vitoc
ms.date: 09/09/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: csa-team
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-decentralized-trust.png
ms.openlocfilehash: a3c497f91b3861bf02f05981ee92e578a22a14ca
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299260"
---
# <a name="decentralized-trust-between-banks-on-azure"></a>Bankok közötti nem központosított megbízhatósági kapcsolatok az Azure-ban

Ebben a példaforgatókönyvben hasznos bankok vagy bármely más intézményekkel, az információk megosztásához egy központi adatbázisba menthetők nélkül megbízható környezetet kíván. Jelen példában bemutatunk néhányat kredit pontszám információk között bankok karbantartása kontextusában a forgatókönyvet, de az architektúra alkalmazható a szervezetek konzorcium, ahová az egyik ellenőrzött információk megosztása bármilyen forgatókönyvhöz egy központi rendszer használata nélkül egy másik futott, egy egyetlen fél.

Bankok, pénzügyi rendszeren belül hagyományosan, központosított források kredit ütemezéseiket információ egy személy kredit pontszám és előzmények támaszkodnak. Egy központi módszer a működési kockázatokat, és néha egy szükségtelen külső koncentráció mutat be.

DLTs (elosztott Főkönyv technológiával), a bankok konzorcium hozhatnak létre, amelyek hatékonyabban, kevesebb ki van téve a támadásokkal szemben, és erre a célra egy új platformra, innovatív struktúrákat megoldásához hagyományos implementálhatók decentralizált rendszer az adatvédelmi, a sebesség és a költség kihívásokat.

Ez a példa bemutatja, hogyan Azure szolgáltatásokat, mint a virtuálisgép-méretezési csoportok, virtuális hálózat, a Key Vault, a tárolás, a Load Balancer, és a figyelő is gyorsan üzembe helyezhető egy hatékony privát Ethereum PoA blockchain központi telepítésére vonatkozóan ahol tag bankok is a saját csomópontok létrehozásához.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- A lefoglalt költségvetése multinacionális Corporation különböző üzleti egységek közötti áthelyezését
- Határokon átnyúló
- Kereskedelmi pénzügyi forgatókönyvek
- Használata esetén a különböző vállalatok hűségprogramok használatán keresztül rendszerek
- Ellátási lánccal kapcsolatos ökoszisztémáknál

## <a name="architecture"></a>Architektúra

![Decentralizált Bank megbízhatósági architektúra ábrája](./media/architecture-decentralized-trust.png)

Ebben a forgatókönyvben a háttér-összetevőket, amelyek szükségesek a magánhálózatot hozhat létre méretezhető, biztonságos és felügyelt, nagyvállalati blockchain belül két vagy több tagok konzorcium ismerteti. Ezek az összetevők kiépített hogyan részleteit (azt jelenti, különböző előfizetésekhez és erőforráscsoportokhoz belül), valamint a hálózati kapcsolati követelményeinek (vagyis a VPN- vagy ExpressRoute) a mérnökcsapatnak a szervezeti házirend alapján van hátra követelmények. A következő adatfolyamok:

1. Bank A frissítések egy személy kredit rekord egy tranzakciónak küld a blockchain-hálózat JSON-RPC-n keresztül.
2. Az adatfolyamok Bank egy személyes application server, az Azure terheléselosztó, és ezt követően a virtuálisgép-méretezési csoport a virtuális gép érvényesítése csomópont beállítása.
3. A Ethereum PoA hálózati blokkot hoz létre egy előre beállított időpontban (ebben a forgatókönyvben 2 másodperc).
4. A tranzakció a létrehozott blokk be kötegelve és érvényesíteni a blockchain-hálózaton keresztül.
5. Banki B olvashatja a saját node hasonlóan a JSON-RPC keresztül kommunikál bank A által létrehozott kredit rekord.

### <a name="components"></a>Összetevők

- Virtuális gépek virtuálisgép-méretezési csoportokon belül biztosít az igény szerinti számítási létesítmény az a blockchain érvényesítő folyamatok üzemeltetésére
- A Key Vault titkos kulcsai minden érvényesítő része lesz a biztonságos tárolási létesítmény
- Terheléselosztó osztja el a távoli Eljáráshívás társviszony-létesítéshez, és irányítási DApp kérelmek
- Állandó üzemeltető tárolót a hálózati információkat és a koordináló bérlési
- Az Operations Management Suite (képezte néhány Azure-szolgáltatások) elérhető csomópont, tranzakció / perc és consortium tagok betekintést nyújt.

### <a name="alternatives"></a>Alternatív megoldások

Az Ethereum PoA megközelítés választja ebben a példában egy megfelelő belépési pontját konzorcium szervezetek, amelyek olyan környezetet, ahol információkat kicserélt és osztottak meg egy másik egyszerűen egy megbízható, decentralizált és egyszerűen hozhat létre, mert Ismerje meg, így. Az elérhető Azure-megoldás sablonok is biztosítanak a gyors és kényelmes módszert, nem csak egy Ethereum PoA blockchain elindításához consortium vezető, hanem tag cégek számára a saját erőforráscsoporton belül a saját Azure-erőforrások üzembe helyezése a consortium és az előfizetés egy meglévő hálózathoz csatlakozni.

Más kiterjesztett vagy más esetekben például tranzakciós adatvédelmi aggályokat fordulhatnak elő. Például egy értékpapírok forgatókönyv szerint konzorcium tagjai nem szeretné a tranzakciók, még akkor is, a más tagok láthatják. Más, Ethereum PoA alternatíva megoldó ezek a problémák, a saját módon:

- Corda
- Kvórum
- Hyperledger

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

[Az Azure Monitor] [ monitor] folyamatosan az a blockchain hálózati problémák rendelkezésre állásának figyelésére használható. Az Azure Monitor alapuló egyéni figyelési irányítópult mutató hivatkozást küld, a sikeres telepítés a blockchain-megoldás ebben a forgatókönyvben használt sablon. Az irányítópult megjeleníti a szívverés az elmúlt 30 percben, valamint más hasznos adatokat jelentő csomópontok.

Rendelkezésre állási témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

A blockchain népszerű feladata a tranzakciók, amelyek a blockchain tartalmazhatnak egy előre beállított időn belül. Ebben a forgatókönyvben koncepció jogosultság használ, ahol ilyen méretezhetőség jobban kezelheti, mint a koncepció-az-végzett munka. A koncepció jogosultság&ndash;alapján hálózatok, a consensus résztvevők ismertek és kezelt, alkalmassá téve azt több privát blockchain konzorcium szervezet számára, hogy tudja, hogy egy másik. Például average paraméterek idő, tranzakció / perc letiltása, és a számítási erőforrás-használat az egyéni Irányítópult segítségével egyszerűen figyelhető. Erőforrások is kell beállítani, hogy ennek megfelelően a méretezési követelmények alapján.

Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

[Az Azure Key Vault] [ vault] könnyedén tárolhatja és érvényesítőket a titkos kulcsok kezeléséhez. Ebben a példában az alapértelmezett üzembe helyezés létrehoz egy blockchain-hálózat, amely az interneten keresztül elérhető. Az éles forgatókönyvet, ahol van szükség a magánhálózaton tagok csatlakozhatnak egymáshoz virtuális hálózatok közötti VPN gateway-kapcsolatok keresztül. A kapcsolódó erőforrások az alábbi szakasz a VPN konfigurálásának lépéseit tartalmazza.

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Az Ethereum PoA blockchain is maga biztosít bizonyos fokú rugalmasság az érvényesítési csomópontot is üzembe helyezhetők különböző régiókban. Az Azure rendelkezik üzembe helyezéseket több mint 54 régiókban világszerte. A blockchain, mint ahogyan az ebben a forgatókönyvben a rugalmasság növelésére együttműködés egyedi és frissítése lehetőségeket biztosít. Az ezzel járó rugalmasságot, a hálózat nem csupán biztosít az egyetlen központosított fél, de minden tagjainak. A koncepció jogosultság&ndash;alapján blockchain lehetővé teszi, hogy a hálózat rugalmasságának biztosításával, hogy még a tervezett és szándékos.

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változók várt teljesítmény- és elérhetőségi igényeinek megfelelő.

A méretezési csoport Virtuálisgép-példányain az alkalmazások (a példányok különböző régiókban is lehetnek) futtató száma alapján három példa költség profilok adtunk meg.

- [Kis][small-pricing]: a díjszabási példa 2 virtuális gépen havonta utal. monitorozással ki van kapcsolva
- [Közepes][medium-pricing]: a díjszabási példa 7 virtuális gép havonta utal. figyelési bekapcsolva
- [Nagy][large-pricing]: a díjszabási példa kapcsolva figyeléssel havonta 15 virtuális gépekhez utal.

A fenti díjszabás van egy consortium tag indítása és csatlakozás a blockchain-hálózathoz. Általában a egy consortium, amelyekben több vállalat vagy szervezet vesz részt, minden egyes tagja megkapja a saját Azure-előfizetést.

## <a name="next-steps"></a>További lépések

Ebben a forgatókönyvben egy példát látni, helyezzen üzembe a [Ethereum PoA blockchain bemutató alkalmazás] [ deploy] az Azure-ban. Tekintse át a [a forgatókönyv a forráskód információs][source].

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

További információ az Ethereum koncepció jogosultság megoldássablon használatát az Azure-ban, tekintse át ezt [– használati útmutató][guide].

<!-- links -->
[small-pricing]: https://azure.com/e/4e429d721eb54adc9a1558fae3e67990
[medium-pricing]: https://azure.com/e/bb42cd77437744be8ed7064403bfe2ef
[large-pricing]: https://azure.com/e/e205b443de3e4adfadf4e09ffee30c56
[guide]: /azure/blockchain-workbench/ethereum-poa-deployment
[deploy]: https://portal.azure.com/?pub_source=email&pub_status=success#create/microsoft-azure-blockchain.azure-blockchain-ethereumethereum-poa-consortium
[source]: https://github.com/vitoc/creditscoreblockchain
[monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/
[vault]: https://azure.microsoft.com/services/key-vault/
