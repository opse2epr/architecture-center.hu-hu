---
title: A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban
description: Hogyan megvalósításához a hub-küllős hálózati topológia megosztott szolgáltatással az Azure-ban.
author: telmosampaio
ms.date: 06/19/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 5e5029dd7de78c6953229364f9e8ae2789c2b348
ms.sourcegitcommit: f7418f8bdabc8f5ec33ae3551e3fbb466782caa5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36209559"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban

A referencia-architektúrában épít, a [hub-küllős] [ guidance-hub-spoke] architektúrával, amelyek közé tartozik a megosztott szolgáltatások összes küllők által felhasználható központban hivatkozik. Első lépésként a datacenter áttelepítése a felhőbe, és felépítése felé egy [virtuális datacenter], az első szolgáltatásokat kell megosztania identitás- és biztonsági. A referencia-architektúrában bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyszíni adatközpontját a Azure-ba, és hozzáadása a hálózati virtuális készülék (NVA), amely működhet, és a tűzfalon, hub-küllős topológiában.  [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

*Töltse le az ][visio-download]architektúra [Visio-fájlját*

Ez a topológia a következő előnyöket nyújtja:

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

* **Megosztott szolgáltatások alhálózata**. Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.

* **DMZ alhálózati**. A központban alhálózat virtuális hálózat, amely működhet, és biztonsági készülékek, például a tűzfalaknak NVAs tárolására szolgálnak.

* **Küllő virtuális hálózatok**. Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában. A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek. Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

* **Virtuális társhálózatok létesítése**. Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni. A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok. A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között. Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.

> [!NOTE]
> Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz. Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.

## <a name="recommendations"></a>Ajánlatok

Kapcsolatos ajánlások a [hub-küllős] [ guidance-hub-spoke] referencia-architektúrában is alkalmazhat a megosztott szolgáltatások referencia-architektúrában. 

Az alábbi javaslatokat is, a legtöbb esetben a megosztott szolgáltatások alkalmazni. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="identity"></a>Identitáskezelés

A legtöbb vállalati szervezet egy Active Directory Directory Services (ADDS) környezettel rendelkezik, azok a helyszíni adatközpontban. Lehetővé teszi a felügyeleti eszközök áthelyezése az Azure a helyszíni hálózatból ADDS függő, javasoljuk, hogy állomás ADDS tartományvezérlők az Azure-ban.

Ha a csoportházirend-objektumokat, külön-külön szabályozhatja az Azure és a helyszíni környezetben szeretné használni a különböző AD helyet használja minden Azure-régiót. A tartományvezérlők tegyen függő munkaterhelések hozzáférő központi VNet (hub).

### <a name="security"></a>Biztonság

Amikor munkaterhelések a helyszíni környezetből Azure-ba, az ilyen terhelések némelyike el kell futhat virtuális gépeken. Megfelelőségi okokból szükség lehet kényszeríteni azokat a munkaterheléseket áthaladó forgalom korlátozásait. 

Az Azure szolgáltatásban az állomás különböző biztonsági és teljesítménynövelő szolgáltatások hálózati virtuális készülékek (NVAs) is használhatja. Ha ismeri a helyszíni készülékek adott halmazát ma, javasoljuk, hogy az azonos virtuális készülékek használata Azure adott esetben.

> [!NOTE]
> Az üzembe helyezési parancsfájlok a referenciaarchitektúra Ubuntu virtuális gép IP-továbbítás engedélyezve van, hogy a hálózati virtuális készülék utánozzák használja.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="overcoming-vnet-peering-limits"></a>A virtuális társhálózatok létesítési korlátjának kiküszöbölése

Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban. Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik. Az alábbi diagram ezt a megközelítést mutatja be.

![[3]][3]

Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon. Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén. Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo]. A központi telepítés az előfizetésében hoz létre a következő erőforrás-csoportok:

- hub hozzáadása rg
- hub-nva-rg
- hub-vnet-rg
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

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>A szimulált olyan helyszíni adatközpontban azbb használatával telepítése

Ez a lépés a szimulált helyszíni adatközpontját telepíti, egy Azure virtuális hálózatot.

1. Keresse meg a `hybrid-networking\shared-services-stack\` mappában található a GitHub-tárházban.

2. Nyissa meg az `onprem.json` fájlt. 

3. Keresse meg az összes példányát `Password` és `adminPassword`. Adja meg a felhasználónevet és jelszót a paraméterek értékeit, és mentse a fájlt. 

4. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítés létrehoz egy virtuális hálózatot, Windows és a VPN-átjáró egy olyan virtuális géphez. Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.

### <a name="deploy-the-hub-vnet"></a>A virtuális hálózat központi telepítése

Ez a lépés a hub VNet telepíti, és csatlakozik a a szimulált helyszíni virtuális hálózat.

1. Nyissa meg az `hub-vnet.json` fájlt. 

2. Keresse meg `adminPassword` , és írja be egy felhasználónevet és jelszót a paraméterek. 

3. Keresse meg az összes példányát `sharedKey` és egy megosztott kulcsot adjon meg egy értéket. Mentse a fájlt.

   ```bash
   "sharedKey": "abc123",
   ```

4. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró, az előző szakaszban létrehozott kapcsolat. A VPN-átjáró legfeljebb 40 percet is igénybe vehet.

### <a name="deploy-ad-ds-in-azure"></a>Az Azure Active Directory tartományi szolgáltatások telepítése

Ez a lépés telepíti az Azure Active Directory tartományi szolgáltatások tartományvezérlő.

1. Nyissa meg az `hub-adds.json` fájlt.

2. Keresse meg az összes példányát `Password` és `adminPassword`. Adja meg a felhasználónevet és jelszót a paraméterek értékeit, és mentse a fájlt. 

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
A központi telepítési lépés eltarthat néhány percig, mert ez a két virtuális gép csatlakozik a tartományhoz, a szimulált helyszíni adatközpontban futtatott, és telepíti az Active Directory tartományi szolgáltatások rajtuk.

### <a name="deploy-the-spoke-vnets"></a>A Vnetek küllős telepítése

Ez a lépés telepíti a Vnetek küllős.

1. Nyissa meg az `spoke1.json` fájlt.

2. Keresse meg `adminPassword` , és írja be egy felhasználónevet és jelszót a paraméterek. 

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. Ismételje meg a 1. és 2 fájl `spoke2.json`.

5. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a>A virtuális hálózat hubot a Vnetek küllős partnert

Társviszony-létesítési kapcsolatot létesíthet a virtuális hálózat központi és a Vnetek küllős, futtassa a következő parancsot:

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a>Az NVA telepítése

Ez a lépés telepíti az NVA a `dmz` alhálózati.

1. Nyissa meg az `hub-nva.json` fájlt.

2. Keresse meg `adminPassword` , és írja be egy felhasználónevet és jelszót a paraméterek. 

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a>Kapcsolat tesztelése 

A szimulált a helyszíni környezetből szeretne az elosztóhoz VNet kapcsolat tesztelése.

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

Ismételje meg a sames a Vnetek küllős kapcsolat teszteléséhez:

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```


<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[virtuális datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "A megosztott szolgáltatások topológia az Azure-ban"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
