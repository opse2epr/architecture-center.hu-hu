---
title: Az SAP futtatása az Oracle-adatbázishoz az Azure éles környezetben
description: Egy forgatókönyv példáján megjelenítése egy SAP éles üzembe helyezés az Azure-ban Oracle-adatbázishoz.
author: DharmeshBhagat
ms.date: 9/12/2018
ms.openlocfilehash: 4e61b46a316d3332d3dae6438312a077ed40edb0
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534585"
---
# <a name="running-sap-in-production-using-an-oracle-database-on-azure"></a>Az SAP futtatása az Oracle-adatbázishoz az Azure éles környezetben

Kritikus fontosságú üzleti alkalmazások futtatásához használt SAP-rendszereinket. Bármilyen kimaradásról címmódosítás megszakítja a kulcsfontosságú folyamatokat, és a nagyobb költségek és elveszített bevétel okozhat. Ezen eredmények elkerülve szükséges egy SAP-infrastruktúra, amely magas rendelkezésre állású és rugalmas hibák esetén.

Egy magas rendelkezésre állású SAP-környezet létrehozása szükséges, távolítsa el a rendszer-architektúra és a folyamatok hibáinak hibaérzékeny pontokat. A hibaérzékeny pontokat hely hibák, a rendszer összetevőinek hiba vagy akár emberi hiba okozhatja.

A példaforgatókönyv azt szemlélteti, Windows vagy Linux rendszerű virtuális gépek (VM) az Azure-ban egy magas rendelkezésre állású (HA) Oracle database és a egy SAP üzemelő példányt.  Az SAP üzembe helyezés igényei alapján különböző méretű virtuális gépeket is használhatja.

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Vegye figyelembe az ebben a példában a következő forgatókönyvekhez:

* SAP futó kritikus fontosságú számítási feladatokat.
* A nem kritikus fontosságú SAP számítási feladatokhoz.
* Az SAP, amely egy magas rendelkezésre állású környezet szimulálása tesztelési környezetek.

## <a name="architecture"></a>Architektúra

![Az Azure-beli SAP éles környezetben architektúrájának áttekintése][architecture]

Ebben a példában egy magas rendelkezésre állású konfiguráció egy Oracle database, az SAP central services és a több, különböző virtuális gépeken futó SAP-alkalmazáskiszolgálókhoz tartalmazza. Az Azure-hálózatot használ a [és-küllős topológia](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) biztonsági okokból. Az adatok a következőképpen folyamatok a megoldáson keresztül:

1. A felhasználók elérhetik az SAP-rendszerhez, az SAP felhasználói felülete, egy webes böngésző vagy egyéb ügyféleszközök elől, például a Microsoft Excel használatával. Az ExpressRoute-kapcsolatok biztosít hozzáférést a szervezet helyszíni hálózatból az Azure-ban futó erőforrásokat.
2. Az ExpressRoute naplósorszámig az Azure-beli virtuális hálózat (VNet) az ExpressRoute-átjárót. Hálózati forgalom egy átjáró-alhálózatot az agyi virtuális hálózat létrehozása az ExpressRoute-átjárón keresztül irányítja a rendszer.
3. Az agyi virtuális hálózat, a küllő virtuális hálózat társviszonyban áll. Az alkalmazás szint alhálózatáról üzemelteti a virtuális gépek futnak, az SAP egy rendelkezésre állási csoportban.
4. Az identitás felügyeleti kiszolgálók hitelesítési szolgáltatásokat megoldás biztosítanak.
5. A jump boxon használják a rendszergazdák biztonságosan kezelheti az Azure-ban üzembe helyezett erőforrásokhoz.

### <a name="components"></a>Összetevők

* [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) ebben a forgatókönyvben az Azure-beli virtuális Központ-küllő topológia létrehozásához használt.
* [Virtuális gépek](/azure/virtual-machines/windows/overview) adja meg azokat a számítási erőforrásokat a megoldás minden egyes szinthez. A virtuális gépek minden egyes fürt van konfigurálva, egy [rendelkezésre állási csoport](/azure/virtual-machines/windows/regions-and-availability#availability-sets).
* [Express Route](/azure/expressroute/expressroute-introduction) kiterjeszti a helyszíni hálózatot a kapcsolatszolgáltató által létrehozott egy privát kapcsolaton keresztül a Microsoft-felhőbe.
* [Hálózati biztonsági csoportok (NSG)](/azure/virtual-network/security-overview) egy virtuális hálózatban lévő erőforrásokra irányuló hálózati hozzáférés korlátozásához. Hálózati biztonsági csoportok biztonsági szabályokat, amelyek engedélyezik vagy megtagadják a hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján listáját tartalmazza. 
* [Erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) logikai tárolóként szolgálnak az Azure-erőforrásokhoz.

### <a name="alternatives"></a>Alternatív megoldások

Az SAP az operációs rendszer, az adatbázis-kezelő rendszer és az Azure-környezet virtuális gépek típusai különböző kombinációit rugalmas lehetőségeket nyújt. További információkért lásd: [SAP-jegyzetnek 1928533](https://launchpad.support.sap.com/#/notes/1928533), "SAP alkalmazások az Azure: támogatott termékek és Azure-Virtuálisgéptípusok".

## <a name="considerations"></a>Megfontolandó szempontok

Ajánlott eljárások az Azure-ban magas rendelkezésre állású SAP környezetek kiépítéséhez vannak meghatározva. További információkért lásd: [magas rendelkezésre állású architektúra és forgatókönyvek esetében az SAP NetWeaver](/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios).
További információkért lásd: [magas rendelkezésre állás az SAP-alkalmazások Azure-beli virtuális gépeken](/azure/virtual-machines/workloads/sap/high-availability-guide).
* Oracle-adatbázisokat is rendelkezik ajánlott eljárások az Azure-hoz. További információkért lásd: [tervezése és implementálása az Oracle-adatbázishoz az Azure-ban](/azure/virtual-machines/workloads/oracle/oracle-design). 
* Oracle Data Guard segítségével kiküszöbölése az üzletmenet szempontjából kritikus Oracle-adatbázisok a hibaérzékeny pontokat. További információkért lásd: [Oracle Data Guard megvalósítása az Azure-ban Linux rendszerű virtuális gépen](/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard).

Az Azure infrastruktúra-szolgáltatások üzembe helyezéséhez SAP-termékekhez, az Oracle-adatbázishoz használható kínál. További információkért lásd: [egy SAP számítási feladatok üzembe helyezése egy Azure-on Oracle DBMS](/azure/virtual-machines/workloads/sap/dbms_guide_oracle).

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben futtatásával járó költségeket felfedezheti érdekében a szolgáltatások összes előre konfigurált, a következő költséget számító példa. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg négy példa költség profilok kaphat forgalom mennyisége alapján:

|Méret|A SAP|Virtuális gép adatbázistípus|Adatbázis-tárolóval|(A) SCS VIRTUÁLIS GÉP|(A) SCS-tároló|Alkalmazás VM-típus|Adattárolás|Azure díjkalkulátor|
|----|----|-------|-------|-----|---|---|--------|---------------|
|Kicsi|30000|DS13_v2|4xP20, 1xP20|DS11_v2|1 x P10|DS13_v2|1 x P10|[Kis](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c)|
|Közepes|70000|DS14_v2|6xP20, 1xP20|DS11_v2|1 x P10|4 x DS13_v2|1 x P10|[Közepes](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a)|
Nagy|180000|E32s_v3|5xP30, 1xP20|DS11_v2|1 x P10|6 x DS14_v2|1 x P10|[Nagy méretű](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42)|
Extra nagy|250000|M64s|6xP30, 1xP30|DS11_v2|1 x P10|10 x DS14_v2|1 x P10|[Extra nagy](https://azure.com/e/58c636922cf94faf9650f583ff35e97b)|

Megjegyzés: egy útmutató, díjszabás, és csak azt jelzi, hogy a virtuális gépek és tárolási költségeit. Nem tartalmazza a hálózati, biztonsági mentési tár, és a bejövő/kimenő adatforgalom díját.

* [Kis](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c): az adatbázis-kiszolgáló 8 x vCPUs, 56 GB RAM és 112 GB-os ideiglenes tárterület,-továbbá öt 512 GB-os prémium szintű tárolólemezeket a VM-típus DS13_v2 egy kis rendszer áll. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. Egy egyetlen VM-típus DS13_v2 az SAP-alkalmazáskiszolgáló 8 x vCPUs, 56 GB RAM és 400-GB ideiglenes tárhely, továbbá egy 128 GB-os prémium szintű storage lemezzel.

* [Közepes](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a): az adatbázis-kiszolgáló 16 x vCPUs, 112 GB RAM és 800 GB-os ideiglenes tárterület,-emellett hét 512 GB-os prémium szintű tárolólemezeket a VM-típus DS14_v2 egy közepes méretű rendszer áll. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. Az SAP-alkalmazáskiszolgáló 8 x vCPUs, 56 GB RAM és 400-GB ideiglenes tárhely, továbbá egy 128 GB-os prémium szintű storage lemezzel DS13_v2 írja be a négy virtuális Gépet.

* [Nagy](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42): egy nagy méretű rendszer áll VM-típus E32s_v3 32 x vCPUs, 256 GB RAM és ideiglenes tárterület 800 GB-os, az adatbázis-kiszolgáló emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. Az SAP-alkalmazáskiszolgálók 16 x vCPUs, 112 GB RAM és 224 GB ideiglenes tárhely, továbbá hat 128 GB-os prémium szintű storage lemezzel DS14_v2 írja be a hat virtuális Gépet.

* [Extra nagy](https://azure.com/e/58c636922cf94faf9650f583ff35e97b): 64 x vCPUs, 1024 GB RAM és 2000 GB ideiglenes tárhely,-emellett hét 1024 GB-os prémium szintű tárolólemezeket az adatbázis-kiszolgáló a M64s VM-típus egy extra nagy méretű rendszer áll. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. 10 virtuális gép DS14_v2 lemezzel 16 x vCPUs, 112 GB RAM és 224 GB ideiglenes tárhely, továbbá tíz 128 GB-os prémium szintű storage, az SAP-alkalmazáskiszolgálók írja be.

## <a name="deployment"></a>Környezet

A következő hivatkozás használatával ebben a forgatókönyvben az alapul szolgáló infrastruktúra üzembe helyezése.

<a
href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-3tier-distributed-ora%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> SAP és Oracle nincs telepítve a központi telepítés során. Ezek az összetevők telepítését külön-külön kell.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Más SAP éles környezetben használja az Azure-ban fut, tekintse át a következő SAP referenciaarchitektúrákat:
* [SAP NetWeaver a AnyDB](/azure/architecture/reference-architectures/sap/sap-netweaver) 
* [SAP S/4hana-t](/azure/architecture/reference-architectures/sap/sap-s4hana)
* [Nagyméretű Azure-példányokon futó SAP](/azure/architecture/reference-architectures/sap/hana-large-instances)

<!-- links -->
[architecture]: media/architecture-diagram-sap-production.png
