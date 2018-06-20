---
title: Küllős hálózati topológia implementálása az Azure-ban
description: Küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 4ebb0d4df3e1907662537516cae1f077e68e47b4
ms.sourcegitcommit: f7418f8bdabc8f5ec33ae3551e3fbb466782caa5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36209576"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Küllős hálózati topológia implementálása az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan lehetséges a küllős hálózati topológia implementálása az Azure-ban. Az *agy* egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz. A *küllők* az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók. A helyszíni adatközpont és az agy közötti forgalom egy ExpressRoute- vagy VPN-átjárókapcsolaton keresztül halad át.  [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

*Töltse le az ][visio-download]architektúra [Visio-fájlját*


A topológia előnyei a következők:

* **Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.
* **Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.
* **Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).

Az architektúra gyakori használati módjai többek között a következők:

* A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS). A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.
* Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.
* Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

* **Helyszíni hálózat**. A cégen belül futó helyi magánhálózat.

* **VPN-eszköz**. A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás. A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS). A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.

* **VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**. A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára. További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.

> [!NOTE]
> A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.

* **Agyi virtuális hálózat**. Egy Azure virtuális hálózat agyként a küllős hálózati topológiában. Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.

* **Átjáró-alhálózat**. A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.

* **Küllő virtuális hálózatok**. Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában. A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek. Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

* **Virtuális társhálózatok létesítése**. Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni. A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok. A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között. Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.

> [!NOTE]
> Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz. Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.

## <a name="recommendations"></a>Ajánlatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="resource-groups"></a>Erőforráscsoportok

Az agyi és minden küllő virtuális hálózat implementálható különböző erőforráscsoportokban és eltérő előfizetésekben is, ha mind ugyanahhoz az Azure Active Directory (Azure AD) bérlőhöz tartozik, ugyanabban az Azure-régióban. Ez lehetővé teszi minden számítási feladat decentralizált felügyeletét, miközben a szolgáltatások megosztása változatlan marad az agyi virtuális hálózatban.

### <a name="vnet-and-gatewaysubnet"></a>Virtuális hálózat és GatewaySubnet

Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal. Ez az alhálózat a virtuális hálózat átjárójához szükséges. Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben.

Az átjáró felállításáról további információkat az alábbi referenciaarchitektúrákban talál (a kapcsolattípustól függően):

- [Hibrid hálózat ExpressRoute használatával][guidance-expressroute]
- [Hibrid hálózat VPN-átjáró használatával][guidance-vpn]

A magasabb rendelkezésre álláshoz használhat ExpressRoute-ot és egy VPN-t feladatátvétel esetére. További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával][hybrid-ha].

Küllős topológiát használhat átjáró nélkül is, ha nincs szükség a helyszíni hálózattal való kapcsolatra. 

### <a name="vnet-peering"></a>Társviszony-létesítés virtuális hálózatok között

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

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo]. Használja a virtuális gépek minden egyes virtuális kapcsolatának tesztelése. Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.

A központi telepítés az előfizetésében hoz létre a következő erőforrás-csoportok:

- hub-nva-rg
- hub-vnet-rg
- a helyi üzemeltetésű-jb-rg
- a helyi üzemeltetésű-vnet-rg
- spoke1-vnet-rg
- spoke2-nyílás-rg

A sablonfájlokat paraméter tekintse meg ezeket a neveket, így módosítja őket, ha a paraméter fájlok frissítése az egyeztetéshez.

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.

2. Telepítés [Azure CLI 2.0][azure-cli-2].

3. Telepítse [az Azure építőelemei][azbb] npm-csomagot.

4. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével.

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>A szimulált helyszíni adatközpont üzembe helyezése

A szimulált olyan helyszíni adatközpontban egy Azure virtuális hálózatot, központi telepítéséhez kövesse az alábbi lépéseket:

1. Keresse meg a `hybrid-networking/hub-spoke` mappába a referencia architektúra tárház.

2. Nyissa meg az `onprem.json` fájlt. Cserélje le a értékeinek `adminUsername` és `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (Választható) Linux üzembe helyezés esetén állítsa be `osType` való `Linux`.

4. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítés létrehoz egy virtuális hálózathoz, a virtuális gép és a VPN-átjáró. A VPN-átjáró létrehozásához körülbelül 40 percet is igénybe vehet.

### <a name="deploy-the-hub-vnet"></a>A virtuális hálózat központi telepítése

A virtuális hálózat központi telepítéséhez hajtsa végre az alábbi lépéseket.

1. Nyissa meg az `hub-vnet.json` fájlt. Cserélje le a értékeinek `adminUsername` és `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Választható) Linux üzembe helyezés esetén állítsa be `osType` való `Linux`.

3. A `sharedKey`, adjon meg egy megosztott kulcsot, a VPN-kapcsolathoz. 

    ```bash
    "sharedKey": "",
    ```

4. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró kapcsolat.  A VPN-átjáró létrehozásához körülbelül 40 percet is igénybe vehet.

### <a name="test-connectivity-with-the-hub"></a>A hubbal kapcsolatának tesztelése

A szimulált a helyszíni környezetből szeretne az elosztóhoz VNet kapcsolat tesztelése.

**Windows központi telepítése**

1. Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.

2. Kattintson a `Connect` számára a virtuális gép távoli asztali munkamenetet nyit meg. A megadott jelszó használata a `onprem.json` paraméterfájl.

3. Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag futtatásával ellenőrizze, hogy tud-e csatlakozni a virtuális gép jumpbox VNet központban.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
A kimenet az alábbihoz hasonló kell kinéznie:

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> Alapértelmezés szerint a Windows Server virtuális gépen nem teszik lehetővé az ICMP-válaszokat az Azure-ban. Ha a használni kívánt `ping` kapcsolat tesztelése, az egyes virtuális gépek ICMP-forgalmat a Windows speciális tűzfalon engedélyezni kell.

**Linux-telepítés**

1. Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.

2. Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon. 

3. A Linux parancssorból futtassa `ssh` a szimulált helyszíni környezetben való kapcsolódáshoz. A megadott jelszó használata a `onprem.json` paraméterfájl.

4. Használja a `ping` tesztelése a virtuális gép jumpbox VNet központban parancsot:

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>A Vnetek küllős telepítése

A Vnetek küllős telepítéséhez hajtsa végre az alábbi lépéseket.

1. Nyissa meg az `spoke1.json` fájlt. Cserélje le a értékeinek `adminUsername` és `adminPassword`.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (Választható) Linux üzembe helyezés esetén állítsa be `osType` való `Linux`.

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. Ismételje meg az 1-2 a `spoke2.json` fájlt.

5. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>Kapcsolat tesztelése

A szimulált a helyszíni környezetből a Vnetek küllős való kapcsolat teszteléséhez.

**Windows központi telepítése**

1. Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.

2. Kattintson a `Connect` számára a virtuális gép távoli asztali munkamenetet nyit meg. A megadott jelszó használata a `onprem.json` paraméterfájl.

3. Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag segítségével győződjön meg arról, hogy a virtuális gépek jumpbox a Vnetek küllős a csatlakozhat.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Linux-telepítés**

A szimulált a helyszíni környezetből a küllős Vnetek Linux virtuális gépek használata a kapcsolat, hajtsa végre az alábbi lépéseket:

1. Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.

2. Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon. 

3. A Linux parancssorból futtassa `ssh` a szimulált helyszíni környezetben való kapcsolódáshoz. A megadott jelszó használata a `onprem.json` paraméterfájl.

5. Használja a `ping` a virtuális gépek jumpbox kapcsolat tesztelése az egyes küllős parancs:

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>Kapcsolat hozzáadása küllők között

Ez a lépés nem kötelező. Küllők egymáshoz való csatlakozáshoz engedélyezni szeretné, ha egy newtwork virtuális készülék (NVA) használja a virtuális hálózat központban útválasztóként, és az kényszerítése forgalom a küllők az útválasztó egy másik küllős való kapcsolódási kísérlet során. Egy alapszintű egyetlen virtuális NVA mintaalkalmazás telepítését, felhasználó által definiált útvonalak (udr-EK) engedélyezéséhez a kettő együtt küllő Vnetek csatlakozni, hajtsa végre a következő lépéseket:

1. Nyissa meg az `hub-nva.json` fájlt. Cserélje le a értékeinek `adminUsername` és `adminPassword`.

    ```bash
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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Küllős topológia az Azure-ban"
[1]: ./images/hub-spoke-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással, NVA használatával"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
