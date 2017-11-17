---
title: "Egy magas rendelkezésre állású hibrid hálózati architektúra megvalósítása"
description: "Hogyan egy biztonságos pont-pont hálózati architektúra, amely egy Azure virtuális hálózatra és egy a helyszíni hálózathoz csatlakoztatott ExpressRoute a VPN-átjáró feladatátvételi kiterjedő végrehajtásához."
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 4c101f17e5e91085b61178f9efb2bc5acb61189c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>Egy helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével a VPN-feladatátvételi

A referencia-architektúrában bemutatja, hogyan csatlakozzon egy helyi hálózati egy Azure virtuális hálózatot (VNet) ExpressRoute, pont-pont virtuális magánhálózat (VPN), a feladatátvételi kapcsolat használatával. A helyszíni hálózat és az ExpressRoute-kapcsolaton keresztül az Azure VNet közötti forgalmat. Ha a kapcsolat az ExpressRoute-kapcsolatcsoport leállását, továbbítódik az IPSec VPN-alagúton keresztül. [**Ez a megoldás üzembe helyezéséhez**.](#deploy-the-solution)

Vegye figyelembe, hogy az ExpressRoute-kapcsolatcsoport nem érhető el, ha a VPN-útvonal csak kezelnek magánhálózati társviszony-létesítési kapcsolatok. Nyilvános társviszony-létesítést és a Microsoft társviszony-létesítés kapcsolatok az interneten keresztül továbbítani fogja. 

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra 

Az architektúra a következő összetevőkből áll.

* **A helyszíni hálózat**. Helyi magánhálózat fut egy szervezeten belül.

* **VPN-készülék**. Egy eszköz vagy a helyszíni hálózathoz külső kapcsolatot biztosító szolgáltatás. Lehet, hogy a VPN-készülék olyan hardvereszköz, vagy egy szoftveres megoldás, mint például az Útválasztás és távelérés szolgáltatás (RRAS) a Windows Server 2012-ben. Támogatott VPN-készülék és konfigurálásáról a kiválasztott VPN készülékek Azure történő kapcsolódáshoz listájáért lásd: [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok][vpn-appliance].

* **ExpressRoute-kapcsolatcsoportot**. A 2. vagy 3 rétegbeli áramkör által biztosított a kapcsolat szolgáltatójánál, hogy a peremhálózati útválasztók az Azure-ral csatlakozik a helyi hálózaton. A kapcsolatcsoport a hardver-infrastruktúrát kezeli a kapcsolat szolgáltatóját használja.

* **ExpressRoute virtuális hálózati átjáró**. Az ExpressRoute virtuális hálózati átjáró lehetővé teszi, hogy a kapcsolat a helyszíni hálózaton használt ExpressRoute-kapcsolatcsoportot csatlakozni a virtuális hálózat.

* **VPN virtuális hálózati átjáró**. A VPN-virtuális hálózati átjáró lehetővé teszi, hogy a helyi hálózaton a VPN-készülék csatlakozni a virtuális hálózat. A VPN-virtuális hálózati átjárót a helyszíni hálózat kéréseket fogad csak a VPN-készülék keresztül van konfigurálva. További információkért lásd: [egy a helyszíni hálózathoz csatlakozni a Microsoft Azure virtuális hálózat][connect-to-an-Azure-vnet].

* **VPN-kapcsolat**. A kapcsolat tulajdonságai adja meg a kapcsolat típusát (IPSec) és a kulcsot, a helyszíni VPN-készülék megosztott forgalom titkosításához.

* **Az Azure Virtual Network (VNet)**. Minden virtuális hálózat egyetlen Azure régióban található, és több alkalmazásrétegek üzemeltethet. Alkalmazás rétegeket is szegmentált, minden egyes virtuális alhálózatok használatával.

* **Átjáró alhálózati**. A virtuális hálózati átjárók ugyanazon az alhálózaton van használatban.

* **A felhő alkalmazás**. Az alkalmazást az Azure-ban. A többrétegű konfigurációk – Azure load Balancer terheléselosztók keresztül kapcsolódik, több alhálózattal rendelkező tartalmazhat. Az alkalmazás-infrastruktúrával kapcsolatos további információkért lásd: [futó Windows virtuális gép munkaterhelések] [ windows-vm-ra] és [futó Linux virtuális gép munkaterhelések] [ linux-vm-ra].

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="vnet-and-gatewaysubnet"></a>Virtuális hálózat és a GatewaySubnet

Az ExpressRoute virtuális hálózati átjáró és a VPN-virtuális hálózati átjáró létrehozása ugyanazt a virtuális hálózatot. Ez azt jelenti, hogy ugyanazon az alhálózaton nevű kell közös *GatewaySubnet*.

Ha a virtuális hálózat már tartalmaz egy nevű alhálózat *GatewaySubnet*, győződjön meg arról, hogy egy /27 vagy nagyobb címtartományt. Ha a meglévő alhálózati túl kicsi, a következő PowerShell-parancs segítségével távolítsa el az alhálózatot: 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

Ha a virtuális hálózat nem tartalmaz nevű alhálózat **GatewaySubnet**, hozzon létre egy újat a következő Powershell-parancsot:

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>VPN- és az ExpressRoute-átjáró

Győződjön meg arról, hogy megfelel-e a szervezet a [ExpressRoute előfeltételként szükséges követelmények] [ expressroute-prereq] Azure való kapcsolódáshoz.

Ha az Azure virtuális hálózat már rendelkezik egy VPN-virtuális hálózati átjáró, a következő Powershell-parancs segítségével távolítsa el:

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

Kövesse az utasításokat a [egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási] [ implementing-expressroute] az ExpressRoute-kapcsolatot létesíteni.

Kövesse az utasításokat a [egy hibrid hálózati architektúra az Azure és a helyszíni VPN-végrehajtási] [ implementing-vpn] a VPN-virtuális hálózati átjáró kapcsolat létrehozásához.

Miután a virtuális hálózati átjáró-kapcsolatok létesítése, tesztkörnyezetben a következőképpen:

1. Győződjön meg arról, hogy a helyi hálózaton is elérheti az Azure virtuális hálózat.
2. Lépjen kapcsolatba a szolgáltatót, hogy állítsa le az ExpressRoute-kapcsolatot a teszteléshez.
3. Győződjön meg arról, hogy továbbra is csatlakozhat a helyszíni hálózatból az Azure virtuális hálózat a virtuális hálózati átjáró VPN-kapcsolat használatával.
4. Kérje a szolgáltató újbóli ExpressRoute-kapcsolatot.

## <a name="considerations"></a>Megfontolandó szempontok

ExpressRoute szempontok, tekintse meg a [egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási] [ guidance-expressroute] útmutatást.

Pont-pont VPN szempontok, tekintse meg a [egy hibrid hálózati architektúra az Azure és a helyszíni VPN-végrehajtási] [ guidance-vpn] útmutatást.

Általános Azure biztonsági szempontokat lásd: [Microsoft cloud services és a hálózati biztonság][best-practices-security].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezéséhez

**Prequisites.** Rendelkeznie kell egy meglévő helyi infrastruktúra már be van állítva a megfelelő hálózati berendezések.

A megoldás üzembe helyezéséhez, a következő lépésekkel.

1. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd kövesse az alábbi lépéseket:   
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-er-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd adja meg, majd kövesse az alábbi lépéseket:
   * Válassza ki **használata meglévő** a a **erőforráscsoport** szakaszt, és adja meg `ra-hybrid-vpn-er-rg` a szövegmezőben.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "Egy magas rendelkezésre állású hibrid hálózati architektúra ExpressRoute- és VPN-átjáró architektúrája"
