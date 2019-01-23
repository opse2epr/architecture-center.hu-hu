---
title: Az SAP-feladatokat a fejlesztési és tesztelési környezetek
titleSuffix: Azure Example Scenarios
description: Fejlesztési és tesztelési környezetet hozhat létre SAP számítási feladatokhoz.
author: AndrewDibbins
ms.date: 07/11/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, SAP
ms.openlocfilehash: c06e81f72e264b545aef4bca2c795bb662a16a13
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488205"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a>Fejlesztési és tesztelési környezetek SAP számítási feladatokhoz az Azure-ban

Ez a példa bemutatja, hogyan lehet létrehozni egy fejlesztési-tesztelési környezetet az SAP NetWeaver az olyan Windows vagy Linux-környezet az Azure-ban. A használt adatbázisa AnyDB, az SAP kifejezés bármely támogatott adatbázis-kezelő (amely nem az SAP HANA). Ez az architektúra nem éles környezetekhez lett kialakítva, mert csak egyetlen virtuális géphez (VM) üzemel, és annak méretét is módosítható a szervezet igényeinek megfelelően.

Éles környezetben használja esetben tekintse át az SAP referenciaarchitektúrák alatt érhető el:

- [SAP NetWeaver a AnyDB][sap-netweaver]
- [SAP S/4HANA-T][sap-hana]
- [Nagyméretű Azure-példányokon futó SAP][sap-large]

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- A nem kritikus SAP nem termelési számítási feladatokhoz (védőfal, fejlesztési, tesztelési, minőségbiztosítási)
- A nem kritikus fontosságú SAP számítási feladata

## <a name="architecture"></a>Architektúra

![Az SAP-feladatokat a fejlesztési és tesztelési környezetek architektúra diagramja](./media/architecture-sap-dev-test.png)

Ez a forgatókönyv azt mutatja be, az SAP-rendszer önálló adatbázis és SAP-alkalmazáskiszolgáló az egyetlen virtuális gép üzembe helyezése. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. Az SAP felhasználói felület vagy más ügyféleszközök (az Excel, egy webes böngésző vagy egyéb webalkalmazás) az Azure-alapú SAP-rendszer elérésére használják.
2. Kapcsolat egy meglévő ExpressRoute használatával biztosítja. Az ExpressRoute-kapcsolat leállítása az Azure-ban, az ExpressRoute-átjárót. A hálózati forgalom az ExpressRoute-átjárót az átjáró-alhálózathoz, valamint az átjáró-alhálózat az alkalmazásrétegek küllő alhálózathoz útvonalakat (lásd a [küllős hálózati topológia][hub-spoke]) és a egy hálózaton keresztül Biztonsági átjáró a SAP alkalmazás virtuális géphez.
3. Az identitás felügyeleti kiszolgálók hitelesítési szolgáltatásokat nyújtanak.
4. A jump boxon helyi felügyeleti képességeket biztosít.

### <a name="components"></a>Összetevők

- [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) Azure-on belüli hálózati kommunikáció alapját képezi.
- [Virtuális gép](/azure/virtual-machines/windows/overview) Azure virtuális gépek igény szerinti, nagy léptékben méretezhető, biztonságos és virtualizált infrastruktúra Windows vagy Linux-kiszolgáló használatával biztosít.
- [Az ExpressRoute](/azure/expressroute/expressroute-introduction) lehetővé teszi, kiterjesztheti helyszíni hálózatait a Microsoft cloud, amelyet egy kapcsolatszolgáltató biztosít egy privát kapcsolaton keresztül.
- [Hálózati biztonsági csoport](/azure/virtual-network/security-overview) lehetővé teszi a hálózati virtuális hálózatban lévő erőforrásokra irányuló forgalmat. A hálózati biztonsági csoport egy biztonsági szabályokból álló listát tartalmaz, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy a cél IP-címe, illetve portok vagy protokollok alapján.
- [Erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) logikai tárolóként szolgálnak az Azure-erőforrásokhoz.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

A Microsoft kínál a szolgáltatásiszint-szerződés (SLA) egyetlen Virtuálisgép-példányokhoz. További információ a Microsoft Azure szolgáltatásiszint-szerződés virtuális gépek [SLA-t a virtuális gépek](https://azure.microsoft.com/support/legal/sla/virtual-machines)

### <a name="scalability"></a>Méretezhetőség

Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonsági

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben futtatásával járó költségeket felfedezheti érdekében a szolgáltatások összes előre konfigurált, a következő költséget számító példa. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg négy példa költség profilok kaphat forgalom mennyisége alapján:

|Méret|A SAP|Virtuális gép típusa|Tárhely|Azure díjkalkulátor|
|----|----|-------|-------|---------------|
|Kis méretű|8000|D8s_v3|2xP20, 1xP10|[Kis](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Közepes|16000|D16s_v3|3xP20, 1xP10|[Közepes](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
Nagy méretű|32000|E32s_v3|3xP20, 1xP10|[Nagy méretű](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Extra nagy|64000|M64s|4xP20, 1xP10|[Extra nagy](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> A díjszabás is egy útmutatót, amely csak azt jelzi, hogy a virtuális gépek és tárolási költségeit. Nem tartalmazza a hálózati, biztonsági mentési tár, és a bejövő/kimenő adatforgalom díját.

- [Kis](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): Egy kis rendszer VM-típus D8s_v3 8 x vCPUs, 32 GB RAM-MAL és 200 GB ideiglenes tárhely, továbbá áll két 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.
- [Közepes](https://azure.com/e/465bd07047d148baab032b2f461550cd): Egy közepes méretű rendszer VM-típus D16s_v3 16 x vCPUs, 64 GB RAM-MAL és 400 GB ideiglenes tárhely, továbbá áll három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.
- [Nagy](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): Egy nagy méretű rendszer VM-típus E32s_v3 32 x vCPUs, 256 GB RAM-MAL és 512 GB ideiglenes tárhely, továbbá áll három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.
- [Extra nagy](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): Az extra nagy méretű rendszer áll típusú, a virtuális gép M64s 64 x vCPUs, 1024 GB RAM-MAL és 2000 GB ideiglenes tárhely, továbbá négy 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.

## <a name="deployment"></a>Üzemelő példány

Ebben a forgatókönyvben az alapul szolgáló infrastruktúra telepítéséhez kattintson ide.

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> SAP és Oracle nincs telepítve a központi telepítés során. Ezek az összetevők telepítését külön-külön kell.

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
