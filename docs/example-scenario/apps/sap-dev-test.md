---
title: SAP fejlesztési/tesztelési feladatokhoz
description: Fejlesztési-tesztelési környezet SAP-forgatókönyv
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: d0f266e40969cf4782e69041889a686387499722
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389179"
---
# <a name="sap-for-devtest-workloads"></a>SAP fejlesztési/tesztelési feladatokhoz

Ebben a példában egy fejlesztési-tesztelési megvalósítását SAP NetWeaver futtatása Windows vagy Linux-alapú környezetben az Azure-ban útmutatást nyújt. A használt adatbázisa AnyDB, az SAP kifejezés bármely támogatott adatbázis-kezelő (amely nem az SAP HANA). Ez az architektúra nem éles környezetekhez lett kialakítva, mert csak egyetlen virtuális géphez (VM) üzemel, és annak méretét is módosítható a szervezet igényeinek megfelelően.

Éles környezetben használja esetben tekintse át az SAP referenciaarchitektúrák alatt érhető el:

* [SAP netweaver a AnyDB][sap-netweaver]
* [SAP S/4hana-t][sap-hana]
* [Nagyméretű Azure-példányokon futó SAP][sap-large]

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

* A nem kritikus SAP nem termelési számítási feladatokhoz (védőfal, fejlesztési, tesztelési, minőségbiztosítási)
* A nem kritikus fontosságú SAP business one számítási feladatok

## <a name="architecture"></a>Architektúra

![Ábra](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

Ebben a forgatókönyvben egy SAP-rendszer önálló adatbázis és a SAP alkalmazás kiszolgáló egyetlen virtuális gépen, az adatfolyam-gyűjteményre, a forgatókönyv segítségével a következőképpen mutatja be:

1. A bemutatási szint-ügyfelek a helyszíni saját SAP grafikus felhasználói Felülettel vagy más felhasználói felületek (az Internet Explorer, az Excel vagy más webes alkalmazás) eléréséhez az Azure-alapú SAP-rendszerhez használja.
2. Kapcsolat egy meglévő Expressroute használatával biztosítja. Az Azure-ban, az Expressroute-átjárója az Express Route-kapcsolat megszakadt. Hálózati forgalmi útvonalak az Express Route-átjárón keresztül az átjáró-alhálózat és az átjáró-alhálózat az alkalmazás szinten küllő alhálózathoz (lásd a [küllős] [ hub-spoke] minta) és a egy hálózati biztonsági keresztül Átjáró, a SAP alkalmazás virtuális géphez.
3. Az identitás felügyeleti kiszolgálók hitelesítési szolgáltatásokat nyújtanak.
4. A jump boxon helyi felügyeleti képességeket biztosít.

### <a name="components"></a>Összetevők

* [Erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) Azure-erőforrások logikai tárolója.
* [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) az alapja az Azure-on belüli hálózati kommunikáció
* [Virtuális gép](/azure/virtual-machines/windows/overview) Azure Virtual Machines biztosít az igény szerinti, nagy léptékben méretezhető, biztonságos és virtualizált infrastruktúra Windows vagy Linux-kiszolgáló használatával
* [Express Route](/azure/expressroute/expressroute-introduction) lehetővé teszi, kiterjesztheti helyszíni hálózatait a Microsoft cloud, amelyet egy kapcsolatszolgáltató biztosít egy privát kapcsolaton keresztül.
* [Hálózati biztonsági csoport](/azure/virtual-network/security-overview) lehetővé teszi a hálózati virtuális hálózatban lévő erőforrásokra irányuló forgalmat. A hálózati biztonsági csoport egy biztonsági szabályokból álló listát tartalmaz, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy a cél IP-címe, illetve portok vagy protokollok alapján. 

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

 A Microsoft kínál a szolgáltatásiszint-szerződés (SLA) egyetlen Virtuálisgép-példányokhoz. További információ a Microsoft Azure szolgáltatásiszint-szerződés virtuális gépek [SLA-t a virtuális gépek](https://azure.microsoft.com/support/legal/sla/virtual-machines)

### <a name="scalability"></a>Méretezhetőség

Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ismerje meg a forgatókönyv futtatásával járó költségeket, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált.  Tekintse meg, hogyan díjszabását szeretné módosítani az adott használatra esetben módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg beolvasni a várt forgalom mennyisége alapján négy példa költség profilok:

|Méret|A SAP|Virtuális gép típusa|Storage|Azure díjkalkulátor|
|----|----|-------|-------|---------------|
|Kicsi|8000|D8s_v3|2xP20, 1xP10|[Kis](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|Közepes|16000|D16s_v3|3xP20, 1xP10|[Közepes](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
Nagy|32000|E32s_v3|3xP20, 1xP10|[Nagy méretű](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
Extra nagy|64000|M64s|4xP20, 1xP10|[Extra nagy](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

Megjegyzés: egy útmutató, díjszabás, és csak azt jelzi, hogy a virtuális gépek és tárolási költségeit (kivéve, hálózati, biztonsági mentési tár, és a bejövő/kimenő adatforgalom díját).

* [Kis](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): egy kis rendszer áll VM-típus D8s_v3 8 x vCPUs, 32 GB RAM-MAL és 200 GB ideiglenes tárhely, emellett két 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.
* [Közepes](https://azure.com/e/465bd07047d148baab032b2f461550cd): egy közepes méretű rendszer áll VM-típus D16s_v3 16 x vCPUs, 64 GB RAM-MAL és 400 GB ideiglenes tárhely, emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.
* [Nagy](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): egy nagy méretű rendszer áll VM-típus E32s_v3 32 x vCPUs, 256 GB RAM-MAL és 512 GB ideiglenes tárhely, emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.
* [Extra nagy](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): egy extra nagy méretű rendszer áll típusú, a virtuális gép M64s 64 x vCPUs, 1024 GB RAM-MAL és 2000 GB ideiglenes tárhely, továbbá négy 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.

## <a name="deployment"></a>Környezet

A fenti forgatókönyv hasonlít az alapul szolgáló infrastruktúra üzembe helyezéséhez használja az üzembe helyezés gombra

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

\* SAP nem lesznek automatikusan telepítve van, manuálisan telepítheti után az infrastruktúra állították össze.

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke