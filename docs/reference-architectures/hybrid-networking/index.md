---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz
titleSuffix: Azure Reference Architectures
description: Referenciaarchitektúrák összehasonlítása egy helyszíni hálózat az Azure-hoz való csatlakoztatásához.
author: telmosampaio
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 6172866b08197b0ca1cd3aabb3c14c01b4f06f9c
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486849"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Válasszon egy megoldást a helyszíni hálózat Azure-hoz való csatlakoztatásához

Ez a cikk a helyszíni hálózat egy Azure-beli virtuális hálózathoz (VNethez) történő csatlakoztatásának lehetőségeit hasonlítja össze. Mindegyik lehetőséghez elérhető egy referenciaarchitektúra.

## <a name="vpn-connection"></a>VPN-kapcsolat

A [VPN-átjáró](/azure/vpn-gateway/vpn-gateway-about-vpngateways) a virtuális hálózati átjárók egyik típusa, amely titkosított adatforgalmat továbbít egy Azure-beli virtuális hálózat és egy helyszíni hely között. A titkosított adatforgalom a nyilvános interneten halad át.

Ez az architektúra olyan hibrid alkalmazások esetében megfelelő, ahol a helyszíni hardver és a felhő közötti forgalom várhatóan alacsony, illetve ha a felhő nyújtotta rugalmasságért és feldolgozási kapacitásért cserébe elfogadhatónak tart némileg nagyobb késést.

### <a name="benefits"></a>Előnyök

- Egyszerűen konfigurálható.

### <a name="challenges"></a>Problémák

- Helyszíni VPN-eszközt igényel.
- Bár a Microsoft 99,9%-os rendelkezésre állást biztosít minden VPN-átjáróhoz, az [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) csak a VPN-átjáróra vonatkozik, az átjáróval létesített hálózati kapcsolatra nem.
- Az Azure VPN Gatewayen keresztül létesített VPN-kapcsolat jelenleg legfeljebb 1,25 Gbps sávszélességet támogat. Előfordulhat, hogy az Azure virtuális hálózatot particionálnia kell több VPN-kapcsolat között, ha várhatóan át fogja lépni ezt az átviteli sebességet.

### <a name="reference-architecture"></a>Referenciaarchitektúra

- [Hibrid hálózat VPN-átjáróval](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute-kapcsolat

Az [ExpressRoute](/azure/expressroute/)-kapcsolatok dedikált, privát kapcsolatot használnak egy külső kapcsolatszolgáltatón keresztül. A privát kapcsolat kiterjeszti a helyszíni hálózatot az Azure-ba.

Ez az architektúra megfelelő az olyan nagy méretű, kritikus fontosságú számítási feladatokat futtató hibrid alkalmazásokhoz, amelyek nagyfokú méretezhetőséget igényelnek.

### <a name="benefits"></a>Előnyök

- Sokkal nagyobb sávszélesség áll rendelkezésre; a kapcsolatszolgáltatótól függően legfeljebb 10 Gbps.
- Támogatja a dinamikus sávszélesség-méretezést, amellyel alacsony igénybevétel esetén csökkenthetők a költségek. Ezt a lehetőséget azonban nem minden kapcsolatszolgáltató támogatja.
- A kapcsolat szolgáltatójától függően lehetővé teheti a vállalat számára az országos felhőkhöz való hozzáférést.
- 99,9%-os rendelkezésre állási SLA a teljes kapcsolatra vonatkozóan.

### <a name="challenges"></a>Problémák

- Beállítása bonyolult lehet. ExpressRoute-kapcsolat létrehozásához külső kapcsolatszolgáltatóra van szükség. A hálózati kapcsolat kiépítése a szolgáltató feladata.
- Nagy sebességű helyszíni útválasztókat igényel.

### <a name="reference-architecture"></a>Referenciaarchitektúra

- [Hibrid hálózat ExpressRoute-tal](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute VPN-alapú feladatátvétellel

Ez a lehetőség lényegében az előző kettő kombinációja. Normál körülmények között ExpressRoute-ot használ, ha azonban az ExpressRoute-kapcsolatcsoportban megszakad a kapcsolat, akkor egy VPN-kapcsolat veszi át a feladatokat.

Ez az architektúra olyan hibrid alkalmazásokhoz megfelelő, amelyek az ExpressRoute által nyújtott nagyobb sávszélességet és a magas rendelkezésre állású hálózati kapcsolatot is igénylik.

### <a name="benefits"></a>Előnyök

- Magas rendelkezésre állás az ExpressRoute-kapcsolatcsoport meghibásodása esetén, azonban a tartalék kapcsolat egy alacsonyabb sávszélességű hálózaton fut.

### <a name="challenges"></a>Problémák

- Konfigurálása bonyolult. Egy VPN-kapcsolatot és egy ExpressRoute-kapcsolatcsoportot is be kell állítania.
- Redundáns hardvert (VPN-berendezéseket) igényel, valamint redundáns Azure VPN Gateway-kapcsolatot is, amelynek külön díja van.

### <a name="reference-architecture"></a>Referenciaarchitektúra

- [Hibrid hálózat ExpressRoute-tal és VPN-feladatátvétellel](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a>Küllős hálózati topológia

A küllős hálózati topológiával elkülöníthetők a számítási feladatok, ugyanakkor megoszthatók az olyan szolgáltatások, mint az identitáskezelés és a biztonság. Az agy egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz. A küllők az agyhoz kapcsolódó virtuális hálózatok. A közös szolgáltatások üzembe helyezése az agyon történik, az egyes számítási feladatokat a küllőkön vannak.

### <a name="reference-architectures"></a>Referenciaarchitektúrák

- [Küllős topológia](./hub-spoke.md)
- [Küllős topológia közös szolgáltatásokkal](./shared-services.md)
