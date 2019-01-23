---
title: Az SAP számítási feladatok futtatása az Oracle-adatbázis használatával
titleSuffix: Azure Example Scenarios
description: Éles SAP üzemelő példányt futtathat az Azure-ban egy Oracle-adatbázis használatával.
author: DharmeshBhagat
ms.date: 09/12/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, SAP, Windows, Linux
ms.openlocfilehash: 0f96f173d5db682ccc719869aaa22225345cb3f0
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487474"
---
# <a name="running-sap-production-workloads-using-an-oracle-database-on-azure"></a>Az SAP számítási feladatok futtatása Azure-beli Oracle-adatbázis használata

Kritikus fontosságú üzleti alkalmazások futtatásához használt SAP-rendszereinket. Bármilyen kimaradásról címmódosítás megszakítja a kulcsfontosságú folyamatokat, és a nagyobb költségek és elveszített bevétel okozhat. Ezen eredmények elkerülve szükséges egy SAP-infrastruktúra, amely magas rendelkezésre állású és rugalmas hibák esetén.

Egy magas rendelkezésre állású SAP-környezet létrehozása szükséges, távolítsa el a rendszer-architektúra és a folyamatok hibáinak hibaérzékeny pontokat. A hibaérzékeny pontokat hely hibák, a rendszer összetevőinek hiba vagy akár emberi hiba okozhatja.

A példaforgatókönyv azt szemlélteti, Windows vagy Linux rendszerű virtuális gépek (VM) az Azure-ban egy magas rendelkezésre állású (HA) Oracle database és a egy SAP üzemelő példányt. Az SAP üzembe helyezés igényei alapján különböző méretű virtuális gépeket is használhatja.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- SAP futó kritikus fontosságú számítási feladatokat.
- A nem kritikus fontosságú SAP számítási feladatokhoz.
- Az SAP, amely egy magas rendelkezésre állású környezet szimulálása tesztelési környezetek.

## <a name="architecture"></a>Architektúra

![Az Azure-beli SAP éles környezetben architektúrájának áttekintése][architecture]

Ebben a példában egy magas rendelkezésre állású konfiguráció egy Oracle database, az SAP central services és a több, különböző virtuális gépeken futó SAP-alkalmazáskiszolgálókhoz tartalmazza. Az Azure-hálózatot használ a [és-küllős topológia](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) biztonsági okokból. Az adatok a következőképpen folyamatok a megoldáson keresztül:

1. A felhasználók elérhetik az SAP-rendszerhez, az SAP felhasználói felülete, egy webes böngésző vagy egyéb ügyféleszközök elől, például a Microsoft Excel használatával. Az ExpressRoute-kapcsolatok biztosít hozzáférést a szervezet helyszíni hálózatból az Azure-ban futó erőforrásokat.
2. Az ExpressRoute naplósorszámig az Azure-beli virtuális hálózat (VNet) az ExpressRoute-átjárót. Hálózati forgalom egy átjáró-alhálózatot az agyi virtuális hálózat létrehozása az ExpressRoute-átjárón keresztül irányítja a rendszer.
3. Az agyi virtuális hálózat, a küllő virtuális hálózat társviszonyban áll. Az alkalmazás szint alhálózatáról üzemelteti a virtuális gépek futnak, az SAP egy rendelkezésre állási csoportban.
4. Az identitás felügyeleti kiszolgálók hitelesítési szolgáltatásokat megoldás biztosítanak.
5. A jump boxon használják a rendszergazdák biztonságosan kezelheti az Azure-ban üzembe helyezett erőforrásokhoz.

### <a name="components"></a>Összetevők

- [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) ebben a forgatókönyvben az Azure-beli virtuális Központ-küllő topológia létrehozásához használt.

- [Virtuális gépek](/azure/virtual-machines/windows/overview) adja meg azokat a számítási erőforrásokat a megoldás minden egyes szinthez. A virtuális gépek minden egyes fürt van konfigurálva, egy [rendelkezésre állási csoport](/azure/virtual-machines/windows/regions-and-availability#availability-sets).

- [Az ExpressRoute](/azure/expressroute/expressroute-introduction) kiterjeszti a helyszíni hálózatot a kapcsolatszolgáltató által létrehozott egy privát kapcsolaton keresztül a Microsoft-felhőbe.

- [Hálózati biztonsági csoportok (NSG)](/azure/virtual-network/security-overview) egy virtuális hálózatban lévő erőforrásokra irányuló hálózati hozzáférés korlátozásához. Hálózati biztonsági csoportok biztonsági szabályokat, amelyek engedélyezik vagy megtagadják a hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján listáját tartalmazza.

- [Erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) logikai tárolóként szolgálnak az Azure-erőforrásokhoz.

### <a name="alternatives"></a>Alternatív megoldások

Az SAP az operációs rendszer, az adatbázis-kezelő rendszer és az Azure-környezet virtuális gépek típusai különböző kombinációit rugalmas lehetőségeket nyújt. További információkért lásd: [SAP-jegyzetnek 1928533](https://launchpad.support.sap.com/#/notes/1928533), "Azure-beli SAP-alkalmazások: Támogatott termékek és Azure Virtuálisgép-típusok".

## <a name="considerations"></a>Megfontolandó szempontok

- Ajánlott eljárások az Azure-ban magas rendelkezésre állású SAP környezetek kiépítéséhez vannak meghatározva. További információkért lásd: [magas rendelkezésre állású architektúra és forgatókönyvek esetében az SAP NetWeaver](/azure/virtual-machines/workloads/sap/sap-high-availability-architecture-scenarios). További tájékoztatás [magas rendelkezésre állás az SAP-alkalmazások Azure-beli virtuális gépeken](/azure/virtual-machines/workloads/sap/high-availability-guide).

- Oracle-adatbázisokat is rendelkezik ajánlott eljárások az Azure-hoz. További információkért lásd: [tervezése és implementálása az Oracle-adatbázishoz az Azure-ban](/azure/virtual-machines/workloads/oracle/oracle-design).

- Oracle Data Guard segítségével kiküszöbölése az üzletmenet szempontjából kritikus Oracle-adatbázisok a hibaérzékeny pontokat. További információkért lásd: [Oracle Data Guard megvalósítása az Azure-ban Linux rendszerű virtuális gépen](/azure/virtual-machines/workloads/oracle/configure-oracle-dataguard).

- Az Azure infrastruktúra-szolgáltatások üzembe helyezéséhez SAP-termékekhez, az Oracle-adatbázishoz használható kínál. További információkért lásd: [egy SAP számítási feladatok üzembe helyezése egy Azure-on Oracle DBMS](/azure/virtual-machines/workloads/sap/dbms_guide_oracle).

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben futtatásával járó költségeket felfedezheti érdekében a szolgáltatások összes előre konfigurált, a következő költséget számító példa. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg négy példa költség profilok kaphat forgalom mennyisége alapján:

|Méret|A SAP|Virtuális gép adatbázistípus|Adatbázis-tárolóval|(A)SCS VM|(A)SCS Storage|Alkalmazás VM-típus|App Storage|Azure díjkalkulátor|
|----|----|-------|-------|-----|---|---|--------|---------------|
|Kis méretű|30000|DS13_v2|4xP20, 1xP20|DS11_v2|1x P10|DS13_v2|1x P10|[Kis](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c)|
|Közepes|70000|DS14_v2|6xP20, 1xP20|DS11_v2|1x P10|4x DS13_v2|1x P10|[Közepes](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a)|
Nagy méretű|180000|E32s_v3|5xP30, 1xP20|DS11_v2|1x P10|6x DS14_v2|1x P10|[Nagy méretű](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42)|
Extra nagy|250000|M64s|6xP30, 1xP30|DS11_v2|1x P10|10x DS14_v2|1x P10|[Extra nagy](https://azure.com/e/58c636922cf94faf9650f583ff35e97b)|

> [!NOTE]
> A díjszabás egy útmutató, és csak azt jelzi, hogy a virtuális gépek és tárolási költségeit. Nem tartalmazza a hálózati, biztonsági mentési tár, és a bejövő/kimenő adatforgalom díját.

- [Kis](https://azure.com/e/45880ba0bfdf47d497851a7cf2650c7c): VM-típus DS13_v2 8 x vCPUs, 56 GB RAM és 112 GB-os ideiglenes tárterület, továbbá öt 512 GB-os prémium szintű tárolólemezeket az adatbázis-kiszolgáló egy kis rendszer áll. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. Egy egyetlen VM-típus DS13_v2 az SAP-alkalmazáskiszolgáló 8 x vCPUs, 56 GB RAM és 400-GB ideiglenes tárhely, továbbá egy 128 GB-os prémium szintű storage lemezzel.

- [Közepes](https://azure.com/e/9a523f79591347ca9a48c3aaa1406f8a): Az adatbázis-kiszolgáló 16 x vCPUs, 112 GB RAM és 800 GB-os ideiglenes tárterület,-emellett hét 512 GB-os prémium szintű tárolólemezeket a VM-típus DS14_v2 egy közepes méretű rendszer áll. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. Az SAP-alkalmazáskiszolgáló 8 x vCPUs, 56 GB RAM és 400-GB ideiglenes tárhely, továbbá egy 128 GB-os prémium szintű storage lemezzel DS13_v2 írja be a négy virtuális Gépet.

- [Nagy](https://azure.com/e/f70fccf571e948c4b37d4fecc07cbf42): A nagy méretű rendszer VM-típus E32s_v3 32 x vCPUs, 256 GB RAM és ideiglenes tárterület 800 GB-os, az adatbázis-kiszolgáló emellett áll három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. Az SAP-alkalmazáskiszolgálók 16 x vCPUs, 112 GB RAM és 224 GB ideiglenes tárhely, továbbá hat 128 GB-os prémium szintű storage lemezzel DS14_v2 írja be a hat virtuális Gépet.

- [Extra nagy](https://azure.com/e/58c636922cf94faf9650f583ff35e97b): A 64 x vCPUs, 1024 GB RAM-MAL és 2000 GB ideiglenes tárhely,-emellett hét 1024 GB-os prémium szintű tárolólemezeket az adatbázis-kiszolgáló M64s VM-típus egy extra nagy méretű rendszer áll. Az SAP-példány központi server DS11_v2 virtuálisgéptípusok 2 x vCPUs 14 GB RAM-MAL és 28-GB ideiglenes tárhely. 10 virtuális gép DS14_v2 lemezzel 16 x vCPUs, 112 GB RAM és 224 GB ideiglenes tárhely, továbbá tíz 128 GB-os prémium szintű storage, az SAP-alkalmazáskiszolgálók írja be.

## <a name="deployment"></a>Üzemelő példány

A következő hivatkozás használatával ebben a forgatókönyvben az alapul szolgáló infrastruktúra üzembe helyezése.

<!-- markdownlint-disable MD033 -->

<a
href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-3tier-distributed-ora%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> SAP és Oracle nincs telepítve a központi telepítés során. Ezek az összetevők telepítését külön-külön kell.

## <a name="related-resources"></a>Kapcsolódó erőforrások

Egyéb SAP számítási feladatok futtatása az Azure-ban kapcsolatos információkért tekintse át a következő referenciaarchitektúrákat:

- [SAP NetWeaver a AnyDB](/azure/architecture/reference-architectures/sap/sap-netweaver)
- [SAP S/4HANA](/azure/architecture/reference-architectures/sap/sap-s4hana)
- [SAP HANA nagyméretű példányok](/azure/architecture/reference-architectures/sap/hana-large-instances)

<!-- links -->
[architecture]: media/architecture-sap-production.png
