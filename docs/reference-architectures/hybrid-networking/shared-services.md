---
title: "A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban"
description: "Hogyan megvalósításához a hub-küllős hálózati topológia megosztott szolgáltatással az Azure-ban."
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: f6c50dedc03f8a5ea926a0deee02b8c3e583685b
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/17/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a>A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban

A referencia-architektúrában épít, a [hub-küllős] [ guidance-hub-spoke] architektúrával, amelyek közé tartozik a megosztott szolgáltatások összes küllők által felhasználható központban hivatkozik. Első lépésként a datacenter áttelepítése a felhőbe, és felépítése felé egy [virtuális datacenter], az első szolgáltatásokat kell megosztania identitás- és biztonsági. A hivatkozás archiecture bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyszíni adatközpontját a Azure-ba, és hozzáadása a hálózati virtuális készülék (NVA), amely működhet, és a tűzfalon, hub-küllős topológiában.  [**A megoldás üzembe helyezése.**](#deploy-the-solution)

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

* **Megosztott szolgáltatások alhálózata**. Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.

* **DMZ alhálózati**. A központban alhálózat virtuális hálózat, amely működhet, és biztonsági készülékek, például a tűzfalaknak NVAs tárolására szolgálnak.

* **Küllő virtuális hálózatok**. Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában. A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek. Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

* **Virtuális társhálózatok létesítése**. Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni. A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok. A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között. Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.

> [!NOTE]
> Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz. Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.

## <a name="recommendations"></a>Javaslatok

Kapcsolatos ajánlások a [hub-küllős] [ guidance-hub-spoke] referencia-architektúrában is alkalmazhat a megosztott szolgáltatások referencia-architektúrában. 

Az alábbi javaslatokat is, a legtöbb esetben a megosztott szolgáltatások alkalmazni. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="identity"></a>Identitás

A legtöbb vállalati szervezet egy Active Directory Directory Services (ADDS) környezettel rendelkezik, azok a helyszíni adatközpontban. Lehetővé teszi a felügyeleti eszközök áthelyezése az Azure a helyszíni hálózatból ADDS függő, javasoljuk, hogy állomás ADDS tartományvezérlők az Azure-ban.

Ha a csoportházirend-objektumokat, külön-külön szabályozhatja az Azure és a helyszíni környezetben szeretné használni a különböző AD helyet használja minden Azure-régiót. A tartományvezérlők tegyen függő munkaterhelések hozzáférő központi VNet (hub).

### <a name="security"></a>Biztonság

Amikor munkaterhelések a helyszíni környezetből Azure-ba, az ilyen terhelések némelyike el kell futhat virtuális gépeken. Megfelelőségi okokból szükség lehet kényszeríteni azokat a munkaterheléseket áthaladó forgalom korlátozásait. 

Az Azure szolgáltatásban az állomás különböző biztonsági és teljesítménynövelő szolgáltatások hálózati virtuális készülékek (NVAs) is használhatja. Ha ismeri a helyszíni készülékek adott halmazát ma, javasoljuk, hogy az azonos virtuális készülékek használata Azure adott esetben.

> [!NOTE]
> Az üzembe helyezési parancsfájlok a referenciaarchitektúra IP-továbbítás enbaled Ubuntu virtuális gép használatával a hálózati virtuális készülék utánozzák.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="overcoming-vnet-peering-limits"></a>A virtuális társhálózatok létesítési korlátjának kiküszöbölése

Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban. Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik. Az alábbi diagram ezt a megközelítést mutatja be.

![[3]][3]

Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon. Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén. Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo]. Ez minden virtuális hálózatban Ubuntu virtuális gépeket használ a kapcsolat tesztelésére. Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.

### <a name="prerequisites"></a>Előfeltételek

Mielőtt üzembe helyezhetné saját előfizetésében a referenciaarchitektúrát, az alábbi lépéseket kell elvégeznie.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve van a számítógépén. Információk a CLI telepítésével kapcsolatban: [Az Azure CLI 2.0 telepítése][azure-cli-2].

3. Telepítse [az Azure építőelemei][azbb] npm-csomagot.

4. Egy parancs parancssori futtatásával, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével bash, és kövesse az utasításokat.

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a>A szimulált olyan helyszíni adatközpontban azbb használatával telepítése

A szimulált olyan helyszíni adatközpontban egy Azure virtuális hálózatot, központi telepítéséhez kövesse az alábbi lépéseket:

1. Navigáljon az előfeltételekre vonatkozó előző lépésekben letöltött adattár `hybrid-networking\shared-services-stack\` mappájához.

2. Nyissa meg a `onprem.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sor 45 és 46, alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Futtatás `azbb` a szimulált helyi üzemeltetésű környezet telepítése a lent látható módon.

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `onprem-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

4. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítés létrehoz egy virtuális hálózatot, Windows és a VPN-átjáró egy olyan virtuális géphez. Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.

### <a name="azure-hub-vnet"></a>Azure agyi virtuális hálózat

Az agyi virtuális hálózat üzembe helyezéséhez és a korábban létrehozott, szimulált helyszíni virtuális hálózathoz való kapcsolódáshoz kövesse az alábbi lépéseket.

1. Nyissa meg a `hub-vnet.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sor 50 és 51, alább látható módon.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. A sor 52, a `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.

3. Adjon meg egy megosztott kulcsot sorban 83, az idézőjelek között alább látható módon, majd mentse a fájlt.

  ```bash
  "sharedKey": "",
  ```

4. Futtatás `azbb` a szimulált helyi üzemeltetésű környezet telepítése a lent látható módon.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

5. Várjon, amíg az üzembe helyezés befejeződik. A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró, az előző szakaszban létrehozott kapcsolat. Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.

### <a name="adds-in-azure"></a>Az Azure ad hozzá

A tartományvezérlőket az ADDS az Azure-ban, hajtsa végre az alábbi lépéseket.

1. Nyissa meg a `hub-adds.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sorok 14 és 15 közé alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. Futtatás `azbb` a ADDS tartományvezérlők telepítése a lent látható módon.

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-adds-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

  > [!NOTE]
  > Ez a központi telepítés részét eltarthat néhány percig, mivel a két virtuális gép csatlakoztatása a tartományhoz, a szimulált helyszíni adatközpontban, majd az AD DS telepítése rajtuk futó igényel.

### <a name="nva"></a>NVA

Az NVA telepítéséhez a `dmz` alhálózati, hajtsa végre a következő lépéseket:

1. Nyissa meg a `hub-nva.json` fájlt, és adjon meg egy felhasználónevet és jelszót az 14., 13 sorok idézőjelek között alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. Futtatás `azbb` központi telepítése az NVA virtuális gép és a felhasználó által megadott útvonalak.

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-nva-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

### <a name="azure-spoke-vnets"></a>Azure küllő virtuális hálózatok

A Vnetek küllős telepítéséhez hajtsa végre az alábbi lépéseket.

1. Nyissa meg a `spoke1.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sorok 52 és 53, alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. A sor 54, a `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.

3. Futtatás `azbb` központi telepítése az első küllős VNet környezet alább látható módon.

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `spoke1-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

3. Ismételje meg az 1-fájl `spoke2.json`.

4. Futtatás `azbb` központi telepítése a második küllős VNet környezet alább látható módon.

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `spoke2-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Azure agyi virtuális társhálózatok létesítése küllő virtuális hálózatokhoz

A központ virtuális hálózat számára a Vnetek küllős társviszony-létesítési kapcsolat létrehozásához hajtsa végre az alábbi lépéseket.

1. Nyissa meg a `hub-vnet-peering.json` fájlt, és ellenőrizze, hogy az egyes a virtuális hálózati társviszony sor 29 kezdve az erőforráscsoport nevét, és a virtuális hálózat neve helyesen-e.

2. Futtatás `azbb` központi telepítése az első küllős VNet környezet alább látható módon.

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.

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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "A megosztott szolgáltatások topológia az Azure-ban"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
