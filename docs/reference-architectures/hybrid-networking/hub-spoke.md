---
title: "A hálózati hub-küllős topológia végrehajtása az Azure-ban"
description: "Hogyan egy hub-küllős hálózati topológia végrehajtásához az Azure-ban."
author: telmosampaio
ms.date: 02/14/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: c03ecd4ba5ddbe50cfb17e56d75c18102b751cfb
ms.sourcegitcommit: 475064f0a3c2fac23e1286ba159aaded287eec86
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/19/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>A hálózati hub-küllős topológia végrehajtása az Azure-ban

A referencia-architektúrában bemutatja, hogyan egy hub-küllős topológia végrehajtásához az Azure-ban. A *hub* van egy virtuális hálózatot (VNet), amely különbséglemezként funkcionál középpontja a helyszíni hálózatot az Azure-ban. A *küllők* társkapcsolatot létesíteni a központ, amely segítségével a feladatok elkülönítése céljából Vnetek vannak. A forgalom a helyszíni adatközpontját és ExpressRoute- vagy VPN-átjáró kapcsolaton keresztül a központ között.  [**Ez a megoldás üzembe helyezéséhez**](#deploy-the-solution).

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra*


A toplogy előnyei a következők:

* **Költségmegtakarításhoz** központosításával szolgáltatásokról, amelyek több munkaterhelés, például a hálózati virtuális készülékek (NVAs) és a DNS-kiszolgálók, egyetlen helyen megoszthatók.
* **Előfizetések korlátok kapcsolatos** társviszony-létesítési Vnetekhez különböző előfizetésből a központi hub által.
* **Aggályokat elkülönítése** közötti központi informatikai (SecOps, InfraOps) és a munkaterhelések (DevOps).

Ez az architektúra a jellemző használati többek között:

* Különböző környezetekben, például a fejlesztési, tesztelési és éles, üzembe helyezett munkaterhelések tekintetében, amely esetében a megosztott szolgáltatások, például a DNS, az Azonosítók, az NTP vagy az AD DS. Megosztott szolgáltatások kerülnek a hub VNet, amíg minden környezetben telepít egy küllős elkülönítési fenntartásához.
* Egymással kapcsolatot nem igényelnek, de a megosztott szolgáltatások elérését igénylő munkaterhelések.
* A vállalatok, amelyek központi ellenőrzése alatt tartja a biztonsági szempontjait, például egy tűzfal DMZ, mint a központban igényelnek, és minden küllős lévő munkaterhelések felügyeleti elkülönített.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

* **A helyszíni hálózat**. Helyi magánhálózat fut egy szervezeten belül.

* **VPN-eszköz**. Egy eszköz vagy a helyszíni hálózathoz külső kapcsolatot biztosító szolgáltatás. A VPN-eszköz lehet olyan hardvereszköz, vagy egy szoftveres megoldás, mint például az Útválasztás és távelérés szolgáltatás (RRAS) a Windows Server 2012-ben. Támogatott VPN-készülék és konfigurálásáról a kiválasztott VPN készülékek Azure történő kapcsolódáshoz listájáért lásd: [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok][vpn-appliance].

* **VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**. A virtuális hálózati átjáró lehetővé teszi, hogy a virtuális hálózaton a VPN-eszköz vagy ExpressRoute-kapcsolatcsoportot, a helyszíni hálózati kapcsolattal használt való csatlakozáshoz. További információkért lásd: [egy a helyszíni hálózathoz csatlakozni a Microsoft Azure virtuális hálózat][connect-to-an-Azure-vnet].

> [!NOTE]
> Az üzembe helyezési parancsfájlok a referenciaarchitektúra használható VPN-átjáró a hálózati kapcsolatot, és egy Vnetet az Azure-ban szimulációjára a helyszíni hálózat.

* **Hub VNet**. A központ-küllős topológia hub használt Azure virtuális hálózat. A központ középpontja a helyszíni hálózathoz csatlakoznak, és a hely, a különböző terhelésekhez a Vnetek küllős üzemeltetett által felhasználható gazdaszolgáltatásokat.

* **Átjáró alhálózati**. A virtuális hálózati átjárók ugyanazon az alhálózaton van használatban.

* **A megosztott szolgáltatások alhálózati**. A virtuális hálózat összes küllők, például a DNS- vagy Active Directory tartományi szolgáltatások között megosztható gazdaszolgáltatásokat használt központban alhálózat.

* **Vnetek küllő**. Egy vagy több Azure Vnetekhez, amely a hub-küllős topológia küllők használatosak. Különítse el a saját Vnetek, függetlenül a más küllők felügyelt munkaterhelésekre küllők használható. Egyes alkalmazások és szolgáltatások közé tartozik a többrétegű konfigurációk –, több alhálózattal Azure load Balancer terheléselosztók keresztül csatlakoznak. Az alkalmazás-infrastruktúrával kapcsolatos további információkért lásd: [futó Windows virtuális gép munkaterhelések] [ windows-vm-ra] és [futó Linux virtuális gép munkaterhelések] [ linux-vm-ra].

* **Vnetben társviszony-létesítés**. Két azonos Azure-régióban vnetek használatával egy [társviszony-létesítési kapcsolat][vnet-peering]. Társviszony-létesítési kapcsolatok nem tranzitív, alacsony késésű kapcsolatok Vnetek között. Ha nincsenek társviszonyban, a Vnetek exchange-forgalom Azure gerincét, nincs szükség az útválasztó használatával. A hálózati hub-küllős topológiában használhatja a központ kapcsolódni minden egyes küllős hálózatokból.

> [!NOTE]
> Ez a cikk csak magában foglalja az [erőforrás-kezelő](/azure/azure-resource-manager/resource-group-overview) központi telepítések, de is csatlakozhatnak a klasszikus virtuális hálózatot egy erőforrás-kezelő virtuális hálózat ugyanabban az előfizetésben. Ily módon a küllők lehet üzemeltetni a klasszikus üzembe helyezés-továbbra is igénybe központban megosztott szolgáltatások.


## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="resource-groups"></a>Erőforráscsoportok

A virtuális hálózat, küllős minden VNet megvalósítható másik erőforráscsoport-sablonok, és akár más előfizetések mindaddig, amíg az azonos Azure Active Directory (Azure AD) bérlői azonos Azure-régióban tartoznak. Ez lehetővé teszi az egyes munkaterhelések decentralizált felügyeleti szolgáltatások a virtuális hálózat központban kezelt megosztása során.

### <a name="vnet-and-gatewaysubnet"></a>Virtuális hálózat és a GatewaySubnet

Hozzon létre egy nevű alhálózatot *GatewaySubnet*, a /27 címtartományt. Ez az alhálózat által a virtuális hálózati átjáró szükséges. Lefoglalásával megelőzése érdekében az alhálózathoz 32 cím segítségével átjáró méretkorlátai a jövőben elérése.

Az átjáró beállításával kapcsolatos további információkért tekintse meg a következő hivatkozás architektúrák, attól függően, hogy a kapcsolat típusa:

- [Hibrid hálózati ExpressRoute segítségével][guidance-expressroute]
- [Hibrid hálózathoz egy VPN-átjáró][guidance-vpn]

Magas rendelkezésre állás érdekében használhatja az ExpressRoute- és VPN-en a feladatátvételre. Lásd: [egy a helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével a VPN-feladatátvételi][hybrid-ha].

A központ-küllős topológia is nélkül használható lesz átjárót, ha nem szükséges kapcsolat a helyszíni hálózattal. 

### <a name="vnet-peering"></a>Társviszony létesítése virtuális hálózatok között

VNet-társviszony létesítése – két Vnetek között nem tranzitív kapcsolat. Ha csatlakozni egymáshoz küllők van szüksége, fontolja meg, ezek küllők külön társviszony-létesítési kapcsolatát.

Azonban ha több küllők egymással csatlakoztatni, futtatja kívül lehetséges társviszony-létesítési kapcsolatok nagyon gyorsan miatt a [tartozó Vnetek esetében egy virtuális hálózat számára a korlátozás][vnet-peering-limit]. Ebben az esetben fontolja meg az felhasználó által megadott útvonalakat (udr-EK) használata egy, a központ VNet útválasztóként NVA küldendő küllős irányuló forgalom kényszerítése. Ez lehetővé teszi a küllők csatlakozni egymáshoz.

Beállíthatja úgy is küllők kommunikálni a távoli hálózatokhoz a központ virtuális hálózatot átjáró használatára. Az átjáró forgalmat engedélyezi a küllős hubhoz, és a távoli hálózatokhoz csatlakozhat, meg kell:

  - A Vnetben társviszony-létesítési kapcsolat konfigurálása a központban **átjáró átvitel engedélyezése**.
  - A Vnetben társviszony-létesítési kapcsolat konfigurálása az egyes küllős **távoli átjárókat használ**.
  - Minden Vnetben társviszony-létesítési létesített kapcsolatok konfigurálására **lehetővé a továbbított forgalom**.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="spoke-connectivity"></a>Kapcsolat küllő

Ha küllők közötti kapcsolat van szüksége, vegye fontolóra NVA központban útválasztáshoz, és a küllős udr-EK segítségével szeretne az elosztóhoz forgalom továbbítására.

![[2]][2]

Ebben az esetben konfigurálnia kell a társviszony-létesítési kapcsolatok **lehetővé a továbbított forgalom**.

### <a name="overcoming-vnet-peering-limits"></a>Hibás a Vnetben társviszony-létesítési korlátok

Mindenképpen vegye figyelembe a [tartozó Vnetek esetében egy virtuális hálózat számára a korlátozás] [ vnet-peering-limit] az Azure-ban. Ha úgy dönt, mint a korlát lehetővé teszi több küllők van szüksége, fontolja meg egy hub-küllős-hub-küllős topológia, ahol az első szintű küllők is összekötőként hubok létrehozása. Az alábbi ábrán látható, ezt a módszert használja.

![[3]][3]

Figyelembe venni az milyen szolgáltatásokat megosztott központban, annak érdekében, a központ méretezi küllők nagyobb számú. Például ha a központ tűzfal szolgáltatásokat nyújt, fontolja meg a sávszélesség korlátja a tűzfal megoldás több küllők hozzáadásakor. Érdemes a megosztott szolgáltatások hubok a második szintű át.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez az architektúra telepítésének érhető el a [GitHub][ref-arch-repo]. Kapcsolat tesztelése Ubuntu virtuális gépeket használ minden egyes virtuális. Nincsenek tárolva tényleges szolgáltatások a **megosztott szolgáltatások** alhálózatának a **hub VNet**.

### <a name="prerequisites"></a>Előfeltételek

Mielőtt a saját előfizetésének telepítése a referencia-architektúrában, a következő lépésekkel kell azt.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [AzureCAT referencia architektúra] [ ref-arch-repo] GitHub-tárházban.

2. Ha szeretné használni az Azure parancssori felület, győződjön meg arról, hogy az Azure CLI 2.0 telepítve a számítógépre. A parancssori felület telepítéséhez kövesse az utasításokat a [Azure CLI 2.0 telepítése][azure-cli-2].

3. Ha jobban szeret PowerShell segítségével, győződjön meg arról, hogy a legújabb PowerShell-modult az Azure-bA telepítve, a számítógépen. A legújabb Azure PowerShell-modul telepítéséhez kövesse a [az Azure PowerShell telepítése][azure-powershell].

4. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával használja az alábbi parancsok egyikét, és kövesse az utasításokat.

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>A szimulált olyan helyszíni adatközpontban telepítése

A szimulált olyan helyszíni adatközpontban, egy Azure virtuális hálózatot telepítéséhez hajtsa végre az alábbi lépéseket.

1. Keresse meg a `hybrid-networking\hub-spoke\onprem` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `onprem.vm.parameters.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 11 és 12 közötti alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Futtassa a bash vagy a PowerShell-parancsot az alábbi történő telepítését a szimulált helyszíni környezetben egy Vnetet az Azure-ban. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-onprem-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.

4. Várjon, amíg a telepítés befejezéséhez. A telepítéshez létrehoz egy virtuális hálózatot, Ubuntu és a VPN-átjáró egy olyan virtuális géphez. A VPN-átjáró létrehozása több mint 40 percet is igénybe vehet.

### <a name="azure-hub-vnet"></a>Az Azure hub virtuális hálózat

A virtuális hálózat központi telepítése, és a fentiekben létrehozott szimulált helyszíni VNet csatlakoznak, hajtsa végre az alábbi lépéseket.

1. Keresse meg a `hybrid-networking\hub-spoke\hub` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `hub.vm.parameters.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 11 és 12 közötti alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Nyissa meg a `hub.gateway.parameters.json` fájlt, és adjon meg egy megosztott kulcsot sorban 23, az idézőjelek között alább látható módon, majd mentse a fájlt. Jegyezze fel ezt az értéket megtartásához később a központi telepítésben lévő használandó szüksége lesz.

  ```bash
  "sharedKey": "",
  ```

4. Futtassa a bash vagy a PowerShell-parancsot az alábbi történő telepítését a szimulált helyszíni környezetben egy Vnetet az Azure-ban. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-hub-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.

5. Várjon, amíg a telepítés befejezéséhez. A telepítéshez létrehoz egy virtuális hálózatot, Ubuntu, a VPN-átjáró és a kapcsolatot az átjáró, az előző szakaszban létrehozott futtató virtuális gép. A VPN-átjáró létrehozása több mint 40 percet is igénybe vehet.

### <a name="connection-from-on-premises-to-the-hub"></a>Kapcsolat a helyszíni és a központ

Csatlakoztassa a szimulált helyszíni adatközpontját a hub VNet, hajtsa végre az alábbi lépéseket.

1. Keresse meg a `hybrid-networking\hub-spoke\onprem` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `onprem.connection.parameters.json` fájlt, és adjon meg egy megosztott kulcsot sorban 9, az idézőjelek között alább látható módon, majd mentse a fájlt. A megosztott kulcs értéke az a korábban már telepítette a helyszíni átjáró használt azonosnak kell lennie.

  ```bash
  "sharedKey": "",
  ```

3. Futtassa a bash vagy a PowerShell-parancsot az alábbi történő telepítését a szimulált helyszíni környezetben egy Vnetet az Azure-ban. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-onprem-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.

4. Várjon, amíg a telepítés befejezéséhez. A központi telepítés szimulálása egy helyszíni adatközpontot, és a virtuális hálózat hub használt hálózatok közötti kapcsolatot hoz létre.

### <a name="azure-spoke-vnets"></a>Az Azure küllős Vnetek

A Vnetek küllős telepíteni, és a fentiekben létrehozott virtuális hálózat központi csatlakozás, hajtsa végre az alábbi lépéseket.

1. Váltás a `hybrid-networking\hub-spoke\spokes` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `spoke1.web.parameters.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 53 és 54 közötti alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. Ismételje meg az előző lépésben a fájl `spoke2.web.parameters.json`.

4. Futtassa a bash vagy telepítése az első küllős, és szeretne az elosztóhoz csatakoztatni az alábbi PowerShell-parancsot. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-spoke1-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.

5. Várjon, amíg a telepítés befejezéséhez. A központi telepítés létrehoz egy virtuális hálózatot, a terheléselosztó három Ubuntu és Apache és egy virtuális hálózatot szeretne az elosztóhoz Vnetben társviszony-létesítési kapcsolat az előző szakaszban létrehozott futtató virtuális gépet. A telepítés több mint 20 percig is eltarthat.

6. Futtassa a bash vagy telepítése az első küllős, és szeretne az elosztóhoz csatakoztatni az alábbi PowerShell-parancsot. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-spoke2-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.

5. Várjon, amíg a telepítés befejezéséhez. A központi telepítés létrehoz egy virtuális hálózatot, a terheléselosztó három Ubuntu és Apache és egy virtuális hálózatot szeretne az elosztóhoz Vnetben társviszony-létesítési kapcsolat az előző szakaszban létrehozott futtató virtuális gépet. A telepítés több mint 20 percig is eltarthat.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>Azure hub Vnetben társviszony-létesítést úgy küllős Vnetek

A virtuális hálózat társviszony-létesítési kapcsolatok VNet központ telepítéséhez hajtsa végre az alábbi lépéseket.

1. Váltás a `hybrid-networking\hub-spoke\hub` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Futtassa a bash vagy a PowerShell-parancsot a társviszony-létesítési kapcsolat telepítéséhez az első küllős. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. Futtassa a bash vagy a PowerShell-parancsot a második küllős központi telepítése a társviszony-létesítési kapcsolat. Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>Kapcsolat tesztelése

Győződjön meg arról, hogy a hub-küllős topológia egy helyszíni adatközpontbeli dolgozott, kövesse az alábbi lépéseket.

1. Azure-portálról [] [portal] csatlakozás az előfizetéshez, és keresse meg a `ra-onprem-vm1` a virtuális gép a `ra-onprem-rg` erőforráscsoportot.

2. Az a `Overview` panelen, vegye figyelembe a `Public IP address` a virtuális gép számára.

3. Csatlakozhat egy SSH-ügyfél az IP-cím, a felhasználónév és a telepítés során megadott jelszóval fent leírt eseteket.

4. A virtuális Gépen a parancssorból csatlakoznia, az az alábbi parancs futtatásával ellenőrizze a Spoke1 VNet és a helyszíni virtuális hálózat közötti kapcsolatot.

  ```bash
  ping 10.1.1.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Az Azure-ban hub-küllős topológia"
[1]: ./images/hub-spoke-gateway-routing.svg "Tranzitív útválasztási az Azure-ban hub-küllős topológia"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Tranzitív útválasztási NVA használata az Azure-ban hub-küllős topológia"
[3]: ./images/hub-spokehub-spoke.svg "Az Azure-ban hub-küllős-hub-küllős topológia"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
