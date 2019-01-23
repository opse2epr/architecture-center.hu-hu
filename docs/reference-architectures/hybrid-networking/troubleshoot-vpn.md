---
title: Hibrid VPN-kapcsolat hibaelhárítása
titleSuffix: Azure Reference Architectures
description: Végezzen hibaelhárítást a VPN gateway kapcsolati percalapú egy a helyszíni hálózat és az Azure.
author: telmosampaio
ms.date: 10/08/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 49dfcf444665ef526e76cac0f4ad13104a142840
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485927"
---
# <a name="troubleshoot-a-hybrid-vpn-connection"></a>Hibrid VPN-kapcsolat hibaelhárítása

Ez a cikk egy VPN-átjárókapcsolat a helyszíni hálózat és az Azure közötti hibaelhárítási tippek biztosít. Általános információk a VPN-nel kapcsolatos gyakori hibák elhárításáról: [VPN-nel kapcsolatos gyakori hibák elhárítása][troubleshooting-vpn-errors].

## <a name="verify-the-vpn-appliance-is-functioning-correctly"></a>Ellenőrizze, hogy a VPN-berendezés megfelelően működik-e

Az alábbi javaslatok segítenek meghatározni, hogy a helyszíni VPN-berendezés megfelelően működik-e.

**Ellenőrizze a VPN-berendezés által létrehozott naplófájlokat hibák és meghibásodások vonatkozásában.** Ez segít meghatározni, hogy a VPN-berendezés helyesen működik-e. Ennek az információnak a helye a berendezéstől függően változik. Ha például a Windows Server 2012 RRAS szolgáltatását használja, az alábbi PowerShell-paranccsal jelenítheti meg az RRAS szolgáltatással kapcsolatos hibaeseményekre vonatkozó információkat:

```PowerShell
Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
```

Az egyes bejegyzések *Üzenet* tulajdonsága tartalmaz hibaleírást. Néhány gyakori példa:

- Hogy nem tud csatlakozni, valószínűleg helytelen az Azure VPN gateway az RRAS VPN hálózatiadapter-konfigurációjában megadott IP-cím miatt.

  ```console
  EventID            : 20111
  MachineName        : on-prem-vm
  Data               : {41, 3, 0, 0}
  Index              : 14231
  Category           : (0)
  CategoryNumber     : 0
  EntryType          : Error
  Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                          interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                          successfully because of the  following error: The network connection between your computer and
                          the VPN server could not be established because the remote server is not responding. This could
                          be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                          and the remote server is not configured to allow VPN connections. Please contact your
                          Administrator or your service provider to determine which device may be causing the problem.
  Source             : RemoteAccess
  ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                          your computer and the VPN server could not be established because the remote server is not
                          responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                          between your computer and the remote server is not configured to allow VPN connections. Please
                          contact your Administrator or your service provider to determine which device may be causing the
                          problem.}
  InstanceId         : 20111
  TimeGenerated      : 3/18/2016 1:26:02 PM
  TimeWritten        : 3/18/2016 1:26:02 PM
  UserName           :
  Site               :
  Container          :
  ```

- Az RRAS VPN hálózatiadapter-konfigurációjában megadott helytelen megosztott kulcsot.

  ```console
  EventID            : 20111
  MachineName        : on-prem-vm
  Data               : {233, 53, 0, 0}
  Index              : 14245
  Category           : (0)
  CategoryNumber     : 0
  EntryType          : Error
  Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                          interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                          successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

  Source             : RemoteAccess
  ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                          unacceptable.
                          }
  InstanceId         : 20111
  TimeGenerated      : 3/18/2016 1:34:22 PM
  TimeWritten        : 3/18/2016 1:34:22 PM
  UserName           :
  Site               :
  Container          :
  ```

Az alábbi PowerShell-paranccsal lekérheti a RRAS szolgáltatáson keresztül megkísérelt csatlakozásra vonatkozó eseménynapló-adatokat is:

```powershell
Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
```

Ha nem sikerül a csatlakozás, ez a napló az alábbihoz hasonló hibákat fog tartalmazni:

```console
EventID            : 20227
MachineName        : on-prem-vm
Data               : {}
Index              : 4203
Category           : (0)
CategoryNumber     : 0
EntryType          : Error
Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                        AzureGateway that has failed. The error code returned on failure is 809.
Source             : RasClient
ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
InstanceId         : 20227
TimeGenerated      : 3/18/2016 1:29:21 PM
TimeWritten        : 3/18/2016 1:29:21 PM
UserName           :
Site               :
Container          :
```

## <a name="verify-connectivity"></a>Ellenőrizze a kapcsolatot

**Ellenőrizze a kapcsolatot és az útválasztást a VPN-átjárón.** Előfordulhat, hogy a VPN-berendezés nem megfelelően irányítja a forgalmat az Azure VPN Gatewayen keresztül. Egy [PsPing][psping] eszközhöz hasonló eszközzel ellenőrizheti a kapcsolatot és az útválasztást a VPN-átjárón. Ha például tesztelni szeretné a csatlakozást egy helyszíni gép és egy, a virtuális hálózaton található webkiszolgáló között, futtassa az alábbi parancsot (a `<<web-server-address>>` helyére írja a webkiszolgáló címét):

```console
PsPing -t <<web-server-address>>:80
```

Ha a helyszíni gép át tudja irányítani a forgalmat a webkiszolgálóra, az alábbihoz hasonló kimenetnek kell megjelennie:

```console
D:\PSTools>psping -t 10.20.0.5:80

PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
Copyright (C) 2012-2014 Mark Russinovich
Sysinternals - www.sysinternals.com

TCP connect to 10.20.0.5:80:
Infinite iterations (warmup 1) connecting test:
Connecting to 10.20.0.5:80 (warmup): 6.21ms
Connecting to 10.20.0.5:80: 3.79ms
Connecting to 10.20.0.5:80: 3.44ms
Connecting to 10.20.0.5:80: 4.81ms

    Sent = 3, Received = 3, Lost = 0 (0% loss),
    Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
```

Ha a helyi számítógép nem tud kommunikálni a megadott céllal, az alábbihoz hasonló üzenetek jelennek meg:

```console
D:\PSTools>psping -t 10.20.1.6:80

PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
Copyright (C) 2012-2014 Mark Russinovich
Sysinternals - www.sysinternals.com

TCP connect to 10.20.1.6:80:
Infinite iterations (warmup 1) connecting test:
Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
Connecting to 10.20.1.6:80:
    Sent = 3, Received = 0, Lost = 3 (100% loss),
    Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
```

**Ellenőrizze, hogy a helyszíni tűzfal engedélyezi-e a VPN-forgalom áthaladását, és hogy a megfelelő portok vannak-e megnyitva.**

**Győződjön meg arról, hogy a helyszíni VPN-berendezés, amely az Azure VPN gateway kompatibilis titkosítási módszert használja.** A házirendalapú útválasztás esetében az Azure VPN-átjáró az AES256, az AES128 és a 3DES titkosítási algoritmust támogatja. Az útvonalalapú átjárók az AES256 és 3DES titkosítási algoritmust támogatják. További információkért lásd: [tudnivalók a VPN-eszközök és IPsec/IKE-paramétereinek Site-to-Site VPN Gateway-kapcsolatok][vpn-appliance].

## <a name="check-for-problems-with-the-azure-vpn-gateway"></a>Az Azure VPN-átjáróval kapcsolatos problémák ellenőrzése

Az alábbi javaslatok segítenek megállapítani, van-e probléma az Azure VPN-átjáróval:

**Vizsgálja meg az Azure VPN gateway diagnosztikai naplóinak potenciális problémákat.** Lásd: [lépésről lépésre: Az Azure Resource Manager virtuális hálózati átjáró diagnosztikai naplók rögzítése][gateway-diagnostic-logs].

**Ellenőrizze, hogy az Azure VPN-átjáró és a helyszíni VPN-berendezés ugyanazt a megosztott hitelesítési kulcsot használja-e.** Az Azure VPN-átjáró által tárolt megosztott kulcsot az alábbi Azure CLI-paranccsal tekintheti meg:

```azurecli
azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
```

Használja a helyszíni VPN-berendezéshez megfelelő parancsot a berendezéshez konfigurált megosztott kulcs megjelenítéséhez.

Győződjön meg arról, hogy az Azure VPN-átjárónak helyet adó *GatewaySubnet* alhálózat nincs NSG-hez társítva.

Az alhálózati adatokat a következő Azure CLI-paranccsal tekintheti meg:

```azurecli
azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
```

Biztosítására ne legyen nevű adatmező *hálózati biztonsági csoportjának azonosítója*. Az alábbi példa bemutatja egy olyan *GatewaySubnet*-példány eredményeit, amelyhez hozzá van rendelve egy NSG (*VPN-Gateway-Group*). Emiatt előfordulhat, hogy az átjáró hibásan működik, ha vannak szabályok meghatározva az NSG-hez.

```console
C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
    info:    Executing command network vnet subnet show
    + Looking up virtual network "profx-vnet"
    + Looking up the subnet "GatewaySubnet"
    data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
    data:    Name                            : GatewaySubnet
    data:    Provisioning state              : Succeeded
    data:    Address prefix                  : 10.20.3.0/27
    data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
    info:    network vnet subnet show command OK
```

**Ellenőrizze, hogy az Azure virtuális hálózat virtuális gépeinek konfigurációja engedélyezi-e a virtuális hálózaton kívülről érkező forgalmat.** Ellenőrizze az összes, a virtuális gépeket tartalmazó alhálózatokhoz társított NSG-szabályt. Az alábbi Azure CLI-paranccsal megtekintheti az összes NSG-szabályt:

```azurecli
azure network nsg show -g <<resource-group>> -n <<nsg-name>>
```

**Ellenőrizze, hogy csatlakoztatva van-e az Azure VPN-átjáró.** A következő Azure PowerShell-paranccsal ellenőrizheti az Azure VPN-kapcsolat aktuális állapotát. A `<<connection-name>>` paraméter a virtuális hálózati átjárót és a helyi átjárót társító Azure VPN-kapcsolat neve.

```powershell
Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
```

Az alábbi kódrészletek kiemelik azt a kimenetet, amely az átjáró csatlakoztatásakor (első példa) és leválasztásakor (második példa) jön létre:

```powershell
PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

AuthorizationKey           :
VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
VirtualNetworkGateway2     :
LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
Peer                       :
ConnectionType             : IPsec
RoutingWeight              : 0
SharedKey                  : ####################################
ConnectionStatus           : Connected
EgressBytesTransferred     : 55254803
IngressBytesTransferred    : 32227221
ProvisioningState          : Succeeded
...
```

```powershell
PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

AuthorizationKey           :
VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
VirtualNetworkGateway2     :
LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
Peer                       :
ConnectionType             : IPsec
RoutingWeight              : 0
SharedKey                  : ####################################
ConnectionStatus           : NotConnected
EgressBytesTransferred     : 0
IngressBytesTransferred    : 0
ProvisioningState          : Succeeded
...
```

## <a name="miscellaneous-issues"></a>Egyéb problémák

Az alábbi javaslatok segítenek megállapítani, hogy van-e probléma a virtuális gazdagép konfigurációjával, a hálózati sávszélesség felhasználásával vagy az alkalmazás teljesítményével:

**Tűzfal-konfiguráció ellenőrzése.** Győződjön meg arról, hogy az alhálózat az Azure virtuális gépeken futó vendég operációs rendszerben a tűzfal megfelelően van konfigurálva, hogy a helyi IP-címtartományokkal engedélyezett forgalmat.

**Ellenőrizze, hogy a forgalom mennyisége nem közelít-e az Azure VPN-átjárón rendelkezésre álló sávszélesség korlátjához.** Ennek az ellenőrzési módja a helyszínen futó VPN-berendezéstől függ. Ha például a Windows Server 2012 RRAS szolgáltatását használja, a teljesítményfigyelő segítségével nyomon követheti a VPN-kapcsolaton keresztül fogadott és küldött adatok mennyiségét. A *Távelérés (RAS) áttekintése* objektum segítségével válassza ki a *Fogadott bájtok/mp* és a *Küldési sebesség (bájt/s)* számlálókat:

![Teljesítményszámlálók a VPN-hálózati forgalom figyelésére](../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png)

Meg kell az eredményeket hasonlítsa össze a rendelkezésre álló sávszélesség VPN-átjáróhoz (a 100 MB/s VpnGw3 termékváltozat 1,25 GB/s, az alapszintű Termékváltozat esetén):

![Példa VPN hálózat teljesítménye ábra](../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png)

**Győződjön meg arról, hogy az alkalmazásterheléshez megfelelő számban és méretben telepített virtuális gépeket.** Állapítsa meg, hogy nem lassúak-e az Azure virtuális hálózat virtuális gépei. Ha igen, akkor lehet, hogy túl vannak terhelve. Elképzelhető, hogy túl kevés gép van a terhelés kezeléséhez, vagy nincsenek megfelelően konfigurálva a terheléselosztók. Ennek meghatározásához, [rögzítse és elemezze a diagnosztikai adatokat][azure-vm-diagnostics]. Az eredményeket megvizsgálhatja az Azure Portal segítségével, de számos külső féltől származó, a teljesítményadatok részleteibe bepillantást engedő eszköz is a rendelkezésére áll.

**Ellenőrizze, hogy az alkalmazás hatékonyan használja-e a felhőerőforrásokat.** Az egyes virtuális gépeken futó kialakítási alkalmazáskód, amelynek feladata meghatározni, hogy az alkalmazások a lehető legjobban kihasználják-e az erőforrásokat. Használhatja például az [Application Insights][application-insights] eszközt.

<!-- links -->

[application-insights]: /azure/application-insights/app-insights-overview-usage
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
