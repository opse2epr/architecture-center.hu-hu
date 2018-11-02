---
title: Egy küllős hálózati topológia közös szolgáltatásokkal megvalósítása az Azure-ban
description: Hogyan lehet közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 10/09/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 209791950c79760ea8aaafc77ff779d6207410ce
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916278"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>Közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban

Ez a referenciaarchitektúra épül, amely a [küllős] [ guidance-hub-spoke] referenciaarchitektúra, amely minden küllő tudják használni az agyban megosztott szolgáltatások tartalmazza. A felhőbe való migrálás egy adatközpontban, és létrehozását az első lépés egy [virtuális adatközpont], meg kell osztania az első szolgáltatások az identitások és a biztonság. Ez a referenciaarchitektúra bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyi adatközpontból az Azure, és hogyan adhat hozzá egy hálózati virtuális készüléket (NVA) működő, a tűzfal egy küllős topológiában.  [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download]*

Ez a topológia előnyei a következők:

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

* **DMZ-alhálózat**. Az agyi virtuális hálózat a gazdagép nva-kkal működő, biztonsági eszközökkel, például a tűzfalak használt alhálózat.

* **Küllő virtuális hálózatok**. Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában. A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek. Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

* **Virtuális társhálózatok létesítése**. Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni. A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok. A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között. Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.

> [!NOTE]
> Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz. Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.

## <a name="recommendations"></a>Javaslatok

Az összes javaslatot a [küllős] [ guidance-hub-spoke] referenciaarchitektúra is alkalmazni kell a megosztott szolgáltatások referenciaarchitektúra. 

Emellett az alábbi javaslatok a a megosztott szolgáltatások legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="identity"></a>Identitás

A legtöbb vállalati szervezetek Active Directory Directory Services (ADDS) környezet rendelkezik a helyszíni adatközpontját. Felügyeleti eszközök a helyszíni hálózatból az Azure-bA áthelyezni, ettől az ADDS elősegítése érdekében ajánlott gazdagép ADDS-tartományvezérlőkhöz az Azure-ban.

Ha használja a csoportházirend-objektumok, külön-külön szabályozhatja az Azure és a helyszíni környezetben, a kívánt használja egy másik Active Directory-hely az egyes Azure-régióban. Helyezze a tartományvezérlők egy központi virtuális hálózatban (hub) függő számítási feladatokhoz férhet hozzá.

### <a name="security"></a>Biztonság

Helyváltoztatáskor számítási feladatokat a helyszíni környezetből az Azure-ba, néhány ilyen számítási feladat kell virtuális gépeken üzemeltetni. Megfelelőségi okokból szükség lehet kikényszerítheti forgalomról azokat a munkaterheléseket. 

Az Azure biztonságát és teljesítményét szolgáltatások különböző típusú gazdagép-hálózati virtuális berendezések (nva-k) is használhatja. Ha ismeri a helyszíni berendezések adott halmazát még ma, ajánlott az azonos virtuális berendezések használata az Azure-ban, ha vannak ilyenek.

> [!NOTE]
> Ez a referenciaarchitektúra üzembe helyezési szkriptjei egy Ubuntu virtuális gép IP-továbbítás engedélyezve van egy hálózati virtuális berendezésen utánzására használja.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="overcoming-vnet-peering-limits"></a>A virtuális társhálózatok létesítési korlátjának kiküszöbölése

Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban. Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik. Az alábbi diagram ezt a megközelítést mutatja be.

![[3]][3]

Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon. Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén. Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo]. Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:

- eseményközpont-ad-rg
- eseményközpont-nva-rg
- eseményközpont-vnet-rg
- rendszert-vnet-rg
- spoke1-vnet-rg
- spoke2-művele-rg

A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>Üzembe helyezése az azbb használatával szimulált helyszíni adatközpont

Ez a lépés telepíti a szimulált helyszíni adatközpont Azure virtuális hálózatként.

1. Keresse meg a `hybrid-networking\shared-services-stack\` mappájában, a GitHub-adattárban.

2. Nyissa meg az `onprem.json` fájlt. 

3. Keresse meg az összes példányát `UserName`, `adminUserName`,`Password`, és `adminPassword`. Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt. 

4. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítéshez létrehoz egy virtuális hálózatot, a Windows és a egy VPN-átjárót futtató virtuális gépet. Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.

### <a name="deploy-the-hub-vnet"></a>Az agyi virtuális hálózat üzembe helyezése

Ebben a lépésben helyez üzembe az agyi virtuális hálózat, és csatlakoztatja a szimulált helyszíni virtuális hálózathoz.

1. Nyissa meg az `hub-vnet.json` fájlt. 

2. Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek. 

3. Keresse meg az összes példányát `sharedKey` és a egy megosztott kulcsot adjon meg egy értéket. Mentse a fájlt.

   ```bash
   "sharedKey": "abc123",
   ```

4. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítés egy virtuális hálózatot, egy virtuális gépet, a VPN-átjáró és az előző szakaszban létrehozott átjáróhoz való kapcsolatot hoz létre. A VPN-átjáró több mint 40 percet is igénybe vehet.

### <a name="deploy-ad-ds-in-azure"></a>Az Azure AD DS üzembe helyezése

Ebben a lépésben üzembe helyezi az AD DS-tartományvezérlők az Azure-ban.

1. Nyissa meg az `hub-adds.json` fájlt.

2. Keresse meg az összes példányát `Password` és `adminPassword`. Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt. 

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
Ebben a lépésben üzembe helyezés eltarthat néhány percig, mert a két virtuális gép csatlakoztatja a szimulált helyszíni adatközpont tartományhoz, és telepíti őket az AD DS.

### <a name="deploy-the-spoke-vnets"></a>A küllő virtuális hálózatok üzembe helyezése

Ebben a lépésben üzembe helyezi a küllő virtuális hálózatokhoz.

1. Nyissa meg az `spoke1.json` fájlt.

2. Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek. 

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. Ismételje meg a 1. és 2 fájl `spoke2.json`.

5. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a>A küllő virtuális hálózatokhoz az agyi virtuális hálózat társviszonyba állítása

Az agyi virtuális hálózat közötti társviszony-létesítési kapcsolat létrehozásához a küllő virtuális hálózatokhoz, futtassa a következő parancsot:

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a>Az nva-t üzembe helyezése

Ebben a lépésben üzembe helyezi az NVA a `dmz` alhálózat.

1. Nyissa meg az `hub-nva.json` fájlt.

2. Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek. 

3. Futtassa az alábbi parancsot:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a>Kapcsolat tesztelése 

Az agyi virtuális hálózat, a szimulált helyszíni környezetből conectivity teszteléséhez.

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

Ismételje meg a sames a küllő virtuális hálózatokhoz csatlakozni:

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
[virtuális adatközpont]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Megosztott szolgáltatások topológia az Azure-ban"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
