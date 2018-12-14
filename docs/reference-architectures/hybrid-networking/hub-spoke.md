---
title: Küllős hálózati topológia implementálása az Azure-ban
titleSuffix: Azure Reference Architectures
description: Küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 10/08/2018
ms.custom: seodec18
ms.openlocfilehash: fe56630b621f02fe71b864642b75688ba1965862
ms.sourcegitcommit: 8d951fd7e9534054b160be48a1881ae0857561ef
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/13/2018
ms.locfileid: "53329432"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Küllős hálózati topológia implementálása az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan lehetséges a küllős hálózati topológia implementálása az Azure-ban. Az *agy* egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz. A *küllők* az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók. A helyszíni adatközpont és az agy közötti forgalom egy ExpressRoute- vagy VPN-átjárókapcsolaton keresztül halad át. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download]*

A topológia előnyei a következők:

- **Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.
- **Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.
- **Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).

Az architektúra gyakori használati módjai többek között a következők:

- A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS). A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.
- Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.
- Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

- **Helyszíni hálózat**. A cégen belül futó helyi magánhálózat.

- **VPN-eszköz**. A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás. A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS). A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.

- **VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**. A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára. További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.

> [!NOTE]
> A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.

- **Agyi virtuális hálózat**. Egy Azure virtuális hálózat agyként a küllős hálózati topológiában. Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.

- **Átjáró-alhálózat**. A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.

- **Küllő virtuális hálózatok**. Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában. A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek. Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

- **Virtuális társhálózatok létesítése**. Két virtuális hálózat hitelesítéssel lehet csatlakozni egy [társviszonykapcsolat][vnet-peering]. A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok. A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között. Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni. Az ugyanabban a régióban, vagy eltérő régiókban lévő virtuális hálózatok társviszonyt. További információkért lásd: [-követelmények és korlátozások][vnet-peering-requirements].

> [!NOTE]
> Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz. Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="resource-groups"></a>Erőforráscsoportok

Az agyi virtuális hálózat, és minden küllő virtuális hálózat implementálható különböző erőforráscsoportokban és eltérő előfizetésekben is. Különböző előfizetésekben található virtuális hálózatok társviszonyba állítása akkor, ha mindkét előfizetés társíthatók az azonos vagy eltérő Azure Active Directory-bérlő. Ez lehetővé teszi minden számítási feladat decentralizált felügyeletét, miközben a szolgáltatások megosztása változatlan marad az agyi virtuális hálózatban.

### <a name="vnet-and-gatewaysubnet"></a>Virtuális hálózat és GatewaySubnet

Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal. Ez az alhálózat a virtuális hálózat átjárójához szükséges. Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben.

Az átjáró felállításáról további információkat az alábbi referenciaarchitektúrákban talál (a kapcsolattípustól függően):

- [Hibrid hálózat ExpressRoute használatával][guidance-expressroute]
- [Hibrid hálózat VPN-átjáró használatával][guidance-vpn]

A magasabb rendelkezésre álláshoz használhat ExpressRoute-ot és egy VPN-t feladatátvétel esetére. További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával][hybrid-ha].

Küllős topológiát használhat átjáró nélkül is, ha nincs szükség a helyszíni hálózattal való kapcsolatra.

### <a name="vnet-peering"></a>Társviszony létesítése virtuális hálózatok között

A virtuális társhálózatok létesítése nem tranzitív kapcsolat két virtuális hálózat között. Ha két küllőt szeretne egymáshoz csatlakoztatni, vegye fontolóra egy külön társviszonykapcsolat hozzáadását az adott küllők között.

Ha azonban több küllőt is szeretne egymáshoz csatlakoztatni, gyorsan el fognak fogyni a rendelkezésre álló társviszonykapcsolatok a [virtuális hálózatonként létesíthető társviszonykapcsolatok számának korlátozása][vnet-peering-limit] miatt. Ilyenkor vegye fontolóra a felhasználó által megadott útvonalak (UDR-ek) használatát. Ezekkel kényszerítheti, hogy az egy adott küllőre irányuló forgalmat a rendszer egy NVA-ra küldje, amely útválasztóként működik az agyi virtuális hálózaton. Ez lehetővé teszi, hogy a küllők kapcsolódjanak egymáshoz.

Ezenkívül konfigurálhatja is a küllőket, hogy használják az agyi virtuális hálózatot a távoli hálózatokkal való kommunikációra. Ahhoz, hogy az átjáróforgalom működjön a küllő és az agy között, a távoli hálózatok pedig kapcsolódjanak, a következők szükségesek:

- Konfigurálja a virtuális társhálózat-kapcsolatot az agyban az **átjáróforgalom engedélyezéséhez**.
- Konfigurálja a virtuális társhálózat-kapcsolatot minden egyes küllőben a **távoli átjárók** használatához.
- Konfiguráljon minden virtuális társhálózat-kapcsolatot a **továbbított forgalom engedélyezéséhez**.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="spoke-connectivity"></a>Küllőkapcsolat

Ha küllők között szeretne kapcsolatot létrehozni, fontolja meg egy NVA implementálását az agyban történő útválasztáshoz, illetve az UDR-ek használatát a küllőkben a forgalom továbbítása érdekében az agyhoz.

![[2]][2]

Ilyenkor konfigurálnia kell a társviszony-kapcsolatokat a **továbbított forgalom engedélyezéséhez**.

### <a name="overcoming-vnet-peering-limits"></a>A virtuális társhálózatok létesítési korlátjának kiküszöbölése

Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban. Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik. Az alábbi diagram ezt a megközelítést mutatja be.

![[3]][3]

Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon. Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén. Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo]. Használ a virtuális gépeket az egyes virtuális hálózatok kapcsolatának tesztelése. Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.

Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:

- eseményközpont-nva-rg
- eseményközpont-vnet-rg
- rendszert-jb-rg
- rendszert-vnet-rg
- spoke1-vnet-rg
- spoke2-művele-rg

A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>A szimulált helyszíni adatközpont üzembe helyezése

A szimulált helyszíni adatközpont Azure virtuális hálózatként üzembe helyezéséhez kövesse az alábbi lépéseket:

1. Keresse meg a `hybrid-networking/hub-spoke` mappába a referencia-architektúrák tárház.

2. Nyissa meg az `onprem.json` fájlt. Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (Nem kötelező) Állítsa be a Linux-telepítéshez `osType` való `Linux`.

4. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. Várjon, amíg az üzembe helyezés befejeződik. Ehhez a központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép és egy VPN-átjárót. A VPN-átjáró létrehozása körülbelül 40 percet is igénybe vehet.

### <a name="deploy-the-hub-vnet"></a>Az agyi virtuális hálózat üzembe helyezése

Az agyi virtuális hálózat üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Nyissa meg az `hub-vnet.json` fájlt. Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Nem kötelező) Állítsa be a Linux-telepítéshez `osType` való `Linux`.

3. Keresse meg a mindkét példányát `sharedKey` , és adjon meg egy megosztott kulcsot a VPN-kapcsolat. Az értékeknek meg kell egyezniük.

    ```json
    "sharedKey": "",
    ```

4. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítés egy virtuális hálózatot, egy virtuális gépet, a VPN-átjáró és az átjáró kapcsolatot hoz létre.  A VPN-átjáró létrehozása körülbelül 40 percet is igénybe vehet.

### <a name="test-connectivity-with-the-hub"></a>A hub-kapcsolat tesztelése

Az agyi virtuális hálózat, a szimulált helyszíni környezetből conectivity teszteléséhez.

**Windows központi telepítési**

1. Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.

2. Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához. Használja az `onprem.json` paraméterfájlban megadott jelszót.

3. Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális Géphez az agyi virtuális hálózat.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

A kimenetnek a következőképpen kell kinéznie:

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> Alapértelmezés szerint Windows Serveres virtuális gépek nem teszik lehetővé az ICMP-válaszok az Azure-ban. Ha a használni kívánt `ping` kapcsolat teszteléséhez, engedélyeznie kell a fokozott biztonságú Windows tűzfal az ICMP-forgalmat az egyes virtuális Gépekhez.

**Linux-telepítés**

1. Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.

2. Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon. 

3. A Linux. Ehhez futtassa `ssh` csatlakozhat a szimulált helyszíni környezetet. Használja az `onprem.json` paraméterfájlban megadott jelszót.

4. Használja a `ping` parancsot az agyi virtuális hálózat a jumpbox virtuális Géphez való kapcsolódás tesztelése:

   ```shell
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>A küllő virtuális hálózatok üzembe helyezése

A küllő virtuális hálózatok üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Nyissa meg az `spoke1.json` fájlt. Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Nem kötelező) Állítsa be a Linux-telepítéshez `osType` való `Linux`.

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. Ismételje meg az 1 – 2. lépéseket a `spoke2.json` fájlt.

5. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>Kapcsolat tesztelése

Teszt conectivity a szimulált helyszíni környezetből a küllő virtuális hálózatokhoz.

**Windows központi telepítési**

1. Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.

2. Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához. Használja az `onprem.json` paraméterfájlban megadott jelszót.

3. Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális gépeket a küllő virtuális hálózatok a.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Linux-telepítés**

A küllő virtuális hálózatokhoz, Linux rendszerű virtuális gépek használata a szimulált helyszíni környezetből conectivity teszteléséhez hajtsa végre az alábbi lépéseket:

1. Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.

2. Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon.

3. A Linux. Ehhez futtassa `ssh` csatlakozhat a szimulált helyszíni környezetet. Használja az `onprem.json` paraméterfájlban megadott jelszót.

4. Használja a `ping` parancsot minden egyes küllőben a jumpbox virtuális gépek kapcsolatának teszteléséhez:

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Kapcsolat hozzáadása küllők között

Ez a lépés nem kötelező. Ha azt szeretné, hogy a küllők kapcsolódjanak egymáshoz, használja az agyi virtuális hálózat útválasztó egy hálózati virtuális készüléket (NVA), és az kényszerít ki forgalmat a küllők az útválasztó egy másik küllőhöz kapcsolódási kísérlet során. Egy alapszintű példa nva-t, mint egyetlen virtuális gép üzembe helyezéséhez a felhasználó által megadott útvonalak (udr-EK), hogy a kettő együtt küllő a virtuális hálózatokhoz való csatlakozás, hajtsa végre az alábbi lépéseket:

1. Nyissa meg az `hub-nva.json` fájlt. Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures

[0]: ./images/hub-spoke.png "Küllős topológia az Azure-ban"
[1]: ./images/hub-spoke-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással, NVA használatával"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
