---
title: Válasszon egy megoldást a helyszíni hálózat Azure-hoz való csatlakoztatásához
description: Referenciaarchitektúrákat hasonlít össze egy helyszíni hálózat az Azure-hoz való csatlakoztatásához.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: 0cc07d3b7d45accf9f99ce32914b0ef065d62f32
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987478"
---
# <a name="connect-an-on-premises-network-to-azure"></a>Helyszíni hálózat csatlakoztatása az Azure-hoz

Ez a cikk a helyszíni hálózat egy Azure-beli virtuális hálózathoz (VNethez) történő csatlakoztatásának lehetőségeit hasonlítja össze. Mindegyik lehetőséghez elérhető egy referenciaarchitektúra.

## <a name="vpn-connection"></a>VPN-kapcsolat

A [VPN-átjáró](/azure/vpn-gateway/vpn-gateway-about-vpngateways) a virtuális hálózati átjárók egyik típusa, amely titkosított adatforgalmat továbbít egy Azure-beli virtuális hálózat és egy helyszíni hely között. A titkosított adatforgalom a nyilvános interneten halad át.

Ez az architektúra olyan hibrid alkalmazások esetében megfelelő, ahol a helyszíni hardver és a felhő közötti forgalom várhatóan alacsony, illetve ha a felhő nyújtotta rugalmasságért és feldolgozási kapacitásért cserébe elfogadhatónak tart némileg nagyobb késést.

**Előnyök**

- Egyszerűen konfigurálható.

**Problémák**

- Helyszíni VPN-eszközt igényel.
- Bár a Microsoft 99,9%-os rendelkezésre állást biztosít minden VPN-átjáróhoz, az [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) csak a VPN-átjáróra vonatkozik, az átjáróval létesített hálózati kapcsolatra nem.
- Azure VPN Gateway-en keresztül létesített VPN-kapcsolat jelenleg legfeljebb 200 Mbps sávszélességet támogat. Előfordulhat, hogy az Azure virtuális hálózatot particionálnia kell több VPN-kapcsolat között, ha várhatóan át fogja lépni ezt az átviteli sebességet.

**Referenciaarchitektúra**

- [Hibrid hálózat VPN-átjáróval](./vpn.md)

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute-kapcsolat

Az [ExpressRoute](/azure/expressroute/)-kapcsolatok dedikált, privát kapcsolatot használnak egy külső kapcsolatszolgáltatón keresztül. A privát kapcsolat kiterjeszti a helyszíni hálózatot az Azure-ba. 

Ez az architektúra megfelelő az olyan nagy méretű, kritikus fontosságú számítási feladatokat futtató hibrid alkalmazásokhoz, amelyek nagyfokú méretezhetőséget igényelnek. 

**Előnyök**

- Sokkal nagyobb sávszélesség áll rendelkezésre; a kapcsolatszolgáltatótól függően legfeljebb 10 Gbps.
- Támogatja a dinamikus sávszélesség-méretezést, amellyel alacsony igénybevétel esetén csökkenthetők a költségek. Ezt a lehetőséget azonban nem minden kapcsolatszolgáltató támogatja.
- A kapcsolat szolgáltatójától függően lehetővé teheti a vállalat számára az országos felhőkhöz való hozzáférést.
- 99,9%-os rendelkezésre állási SLA a teljes kapcsolatra vonatkozóan.

**Problémák**

- Beállítása bonyolult lehet. ExpressRoute-kapcsolat létrehozásához külső kapcsolatszolgáltatóra van szükség. A hálózati kapcsolat kiépítése a szolgáltató feladata.
- Nagy sebességű helyszíni útválasztókat igényel.

**Referenciaarchitektúra**

- [Hibrid hálózat ExpressRoute-tal](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a>ExpressRoute VPN-alapú feladatátvétellel

Ez a lehetőség lényegében az előző kettő kombinációja. Normál körülmények között ExpressRoute-ot használ, ha azonban az ExpressRoute-kapcsolatcsoportban megszakad a kapcsolat, akkor egy VPN-kapcsolat veszi át a feladatokat.

Ez az architektúra olyan hibrid alkalmazásokhoz megfelelő, amelyek az ExpressRoute által nyújtott nagyobb sávszélességet és a magas rendelkezésre állású hálózati kapcsolatot is igénylik. 

**Előnyök**

- Magas rendelkezésre állás az ExpressRoute-kapcsolatcsoport meghibásodása esetén, azonban a tartalék kapcsolat egy alacsonyabb sávszélességű hálózaton fut.

**Problémák**

- Konfigurálása bonyolult. Egy VPN-kapcsolatot és egy ExpressRoute-kapcsolatcsoportot is be kell állítania.
- Redundáns hardvert (VPN-berendezéseket) igényel, valamint redundáns Azure VPN Gateway-kapcsolatot is, amelynek külön díja van.

**Referenciaarchitektúra**

- [Hibrid hálózat ExpressRoute-tal és VPN-feladatátvétellel](./expressroute-vpn-failover.md)


## <a name="hub-spoke-network-topology"></a>Küllős hálózati topológia

A küllős hálózati topológiával elkülöníthetők a számítási feladatok, ugyanakkor megoszthatók az olyan szolgáltatások, mint az identitáskezelés és a biztonság. Az agy egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz. A küllők az agyhoz kapcsolódó virtuális hálózatok. A közös szolgáltatások üzembe helyezése az agyon történik, az egyes számítási feladatokat a küllőkön vannak.


**Referenciaarchitektúrák**

- [Küllős topológia](./hub-spoke.md)
- [Küllős topológia közös szolgáltatásokkal](./shared-services.md)
