---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz
titleSuffix: Azure Reference Architectures
description: Az Azure virtuális hálózat és a egy helyszíni hálózat VPN-átjáróval feladatátvételt biztosító ExpressRoute használatával összekapcsolt kiterjedő magas rendelkezésre állású és biztonságos helyek közötti hálózati architektúra megvalósítása.
author: telmosampaio
ms.date: 10/22/2017
ms.custom: seodec18
ms.openlocfilehash: d44c046f2351d6103a01108574e0295302f0ba11
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119948"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával

Ez a referenciaarchitektúra azt mutatja be, hogy hogyan kapcsolható össze egy helyszíni hálózat egy Azure-beli virtuális hálózattal az ExpressRoute segítségével, amely helyek közötti virtuális magánhálózatot (VPN) biztosít feladatátvételi kapcsolatként. A helyszíni hálózat és az Azure-beli virtuális hálózat közötti forgalom egy ExpressRoute-kapcsolaton keresztül halad át. Ha az ExpressRoute-kapcsolatcsoportban megszakad a kapcsolat, a forgalmat egy IPSec VPN-alagúton keresztül továbbítja a rendszer. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

Vegye figyelembe, hogy ha az ExpressRoute-kapcsolatcsoport nem áll rendelkezésre, a VPN-útválasztás csak a magánhálózati társviszony-létesítésen alapuló kapcsolatokat fogja kezelni. A nyilvános és a Microsoft társviszony-létesítésen alapuló kapcsolatok az interneten fognak továbbítódni.

![Referenciaarchitektúra az ExpressRoute és VPN-átjáró használatával magas rendelkezésre állású hibrid hálózati architektúra](./images/expressroute-vpn-failover.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

- **Helyszíni hálózat**. A cégen belül futó helyi magánhálózat.

- **VPN-berendezés**. A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás. A VPN-berendezés lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS). A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.

- **ExpressRoute-kapcsolatcsoport**. A kapcsolatszolgáltató által biztosított 2. vagy 3. rétegbeli kapcsolatcsoport, amely a peremhálózati útválasztókon keresztül kapcsolja össze a helyszíni hálózatot az Azure-ral. A kapcsolatcsoport a kapcsolatszolgáltató által felügyelt hardveres infrastruktúrát használja.

- **ExpressRoute virtuális hálózati átjáró**. Az ExpressRoute virtuális hálózati átjáró lehetővé teszi, hogy a virtuális hálózat csatlakozni tudjon a helyszíni hálózattal létesített kapcsolathoz használt ExpressRoute-kapcsolatcsoporthoz.

- **VPN virtuális hálózati átjáró**. A VPN virtuális hálózati átjáró a helyszíni hálózaton található VPN-berendezéshez való csatlakozást teszi lehetővé a virtuális hálózat számára. A VPN virtuális hálózati átjáró úgy van konfigurálva, hogy kizárólag a VPN-berendezésen keresztül fogadjon el kéréseket a helyszíni hálózattól. További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.

- **VPN-kapcsolat**. A kapcsolat olyan tulajdonságokkal rendelkezik, amelyek megadják a kapcsolat típusát (IPSec) és a helyszíni VPN-berendezéssel megosztott kulcsot a forgalom titkosításához.

- **Azure Virtual Network (VNet)**. Mindegyik virtuális hálózat egy adott Azure-régióban található, és több alkalmazásréteget is üzemeltethet. Az alkalmazásrétegek alhálózatokkal szegmentálhatók az egyes virtuális hálózatokban.

- **Átjáró-alhálózat**. A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.

- **Felhőalkalmazás**. Az Azure-ban üzemeltetett alkalmazás. Több réteget is foglalhat magában, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="vnet-and-gatewaysubnet"></a>Virtuális hálózat és GatewaySubnet

Az ExpressRoute és a VPN virtuális hálózati átjárót ugyanazon a virtuális hálózaton hozza létre. Ez azt jelenti, hogy ugyanazon a *GatewaySubnet* nevű alhálózaton kell osztozniuk.

Ha a virtuális hálózat már tartalmaz egy *GatewaySubnet* nevű alhálózatot, gondoskodjon róla, hogy az /27 vagy nagyobb címtérrel rendelkezzen. Ha a meglévő alhálózat túl kicsi, távolítsa el az alábbi PowerShell-paranccsal:

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

Ha a virtuális hálózat nem tartalmaz **GatewaySubnet** nevű alhálózatot, hozzon létre egy újat az alábbi PowerShell-paranccsal:

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>VPN- és ExpressRoute-átjárók

Gondoskodjon róla, hogy a cég megfeleljen az [ExpressRoute előfeltételeinek][expressroute-prereq], mert ezek hiányában nem tud az Azure-hoz csatlakozni.

Ha már rendelkezik VPN virtuális hálózati átjáróval az Azure-beli virtuális hálózaton, távolítsa el az alábbi PowerShell-paranccsal:

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

Az ExpressRoute-kapcsolat létrehozásához kövesse a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][implementing-expressroute] szóló cikk utasításait.

A VPN virtuális hálózati átjárókapcsolat létrehozásához kövesse a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][implementing-vpn] szóló cikk utasításait.

A virtuális hálózati átjárókapcsolatok létrehozását követően tesztelje a környezetet az alábbi lépéseket követve:

1. Ellenőrizze, hogy tud-e kapcsolódni a helyszíni hálózatból az Azure-beli virtuális hálózathoz.
2. Kérje meg a szolgáltatót az ExpressRoute-kapcsolat leállítására tesztelés céljából.
3. Ellenőrizze, hogy továbbra is tud-e kapcsolódni a helyszíni hálózatból az Azure-beli virtuális hálózathoz a VPN virtuális hálózati átjárókapcsolat használatával.
4. Kérje meg a szolgáltatót az ExpressRoute-kapcsolat újbóli létrehozására.

## <a name="considerations"></a>Megfontolandó szempontok

Az ExpressRoute-tal kapcsolatos szempontokat a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][guidance-expressroute] szóló cikkben tekintheti át.

A helyek közötti VPN-nel kapcsolatos szempontokat a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][guidance-vpn] szóló cikkben tekintheti át.

Az Azure-biztonsággal kapcsolatos általános szempontokat [a Microsoft Cloud Services szolgáltatásokkal és a hálózati biztonsággal][best-practices-security] foglalkozó cikkben találja.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

**Előfeltételek**. Rendelkeznie kell egy megfelelő hálózati berendezéssel konfigurált meglévő helyszíni infrastruktúrával.

A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

<!-- markdownlint-disable MD033 -->

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

2. Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:
   - Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-er-rg` karakterláncot.
   - Válassza ki a régiót a **Hely** legördülő listából.
   - Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   - Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   - Kattintson a **Vásárlás** gombra.

3. Várjon, amíg az üzembe helyezés befejeződik.

4. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

5. Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:
   - Válassza az **Erőforráscsoport** szakasz **Meglévő használata** elemét, és írja be az `ra-hybrid-vpn-er-rg` karakterláncot a szövegmezőbe.
   - Válassza ki a régiót a **Hely** legördülő listából.
   - Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   - Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   - Kattintson a **Vásárlás** gombra.

<!-- markdownlint-enable MD033 -->

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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
