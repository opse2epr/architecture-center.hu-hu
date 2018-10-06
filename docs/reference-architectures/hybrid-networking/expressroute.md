---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz ExpressRoute használatával
description: A jelen cikk azt ismerteti, hogyan lehet kiépíteni egy helyek közötti biztonságos hálózati architektúrát, amely az Azure ExpressRoute használatával összekapcsolt Azure-beli virtuális hálózatból és helyszíni hálózatból áll.
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 754542b37988e0cd2694ae84eb6b03d68c147243
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819176"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>Helyszíni hálózat csatlakoztatása az Azure-hoz ExpressRoute használatával

Ez a referenciaarchitektúra azt mutatja be, hogyan kapcsolható össze egy helyszíni hálózat az Azure virtuális hálózataival az [Azure ExpressRoute][expressroute-introduction] segítségével. Az ExpressRoute-kapcsolatok dedikált, privát kapcsolatot használnak egy külső kapcsolatszolgáltatón keresztül. A privát kapcsolat kiterjeszti a helyszíni hálózatot az Azure-ba. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

* **Helyszíni vállalati hálózat**. A cégen belül futó helyi magánhálózat.

* **ExpressRoute-kapcsolatcsoport**. A kapcsolatszolgáltató által biztosított 2. vagy 3. rétegbeli kapcsolatcsoport, amely a peremhálózati útválasztókon keresztül kapcsolja össze a helyszíni hálózatot az Azure-ral. A kapcsolatcsoport a kapcsolatszolgáltató által felügyelt hardveres infrastruktúrát használja.

* **Helyi peremhálózati útválasztók**. A helyszíni hálózatot a szolgáltató által kezelt kapcsolatcsoporttal összekapcsoló útválasztók. A kapcsolat kiépítésének módjától függően előfordulhat, hogy meg kell adnia az útválasztók által használt nyilvános IP-címeket.
* **A Microsoft peremhálózati útválasztói**. Két útválasztó egy aktív-aktív, magas rendelkezésre állású konfigurációban. Ezek az útválasztók lehetővé teszik a kapcsolatszolgáltató számára a kapcsolatcsoportok közvetlen csatlakoztatását az adatközpontjaikhoz. A kapcsolat kiépítésének módjától függően előfordulhat, hogy meg kell adnia az útválasztók által használt nyilvános IP-címeket.

* **Azure-beli virtuális hálózatok**. Mindegyik virtuális hálózat egy adott Azure-régióban található, és több alkalmazásréteget is üzemeltethet. Az alkalmazásrétegek alhálózatokkal szegmentálhatók az egyes virtuális hálózatokban.

* **Nyilvános Azure-szolgáltatások**. A hibrid alkalmazásokban használható Azure-szolgáltatások. Ezek a szolgáltatások az interneten keresztül is elérhetők, ExpressRoute-kapcsolatcsoporttal történő használatuk azonban alacsony késést és kiszámíthatóbb teljesítményt biztosít, mert a forgalomnak nem kell áthaladnia az interneten. A kapcsolatok [nyilvános társviszony-létesítéssel][expressroute-peering] jönnek létre, és a vállalat tulajdonában álló vagy a kapcsolatszolgáltató által biztosított címeket használják.

* **Office 365-szolgáltatások**. A Microsoft által biztosított, nyilvánosan elérhető Office 365-alkalmazások és -szolgáltatások. A kapcsolatok [Microsoft társviszony-létesítéssel][expressroute-peering] jönnek létre, és a vállalat tulajdonában álló vagy a kapcsolatszolgáltató által biztosított címeket használják. Microsoft társviszony-létesítéssel közvetlenül kapcsolódhat a Microsoft CRM Online szolgáltatáshoz is.

* **Kapcsolatszolgáltatók** (nem láthatók). Olyan vállalatok, amelyek 2. vagy 3. rétegbeli kapcsolattal biztosítják az Ön adatközpontja és egy Azure-adatközpont közötti csatlakozást.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="connectivity-providers"></a>Kapcsolatszolgáltatók

Az adott helynek megfelelő ExpressRoute-kapcsolatszolgáltatót válasszon. Az Ön helyén elérhető kapcsolatszolgáltatók listájának a lekéréséhez használja az alábbi Azure PowerShell-parancsot:

```powershell
Get-AzureRmExpressRouteServiceProvider
```

Az ExpressRoute-kapcsolatszolgáltatók az alábbi módokon létesítenek kapcsolatot az Ön adatközpontja és a Microsoft között:

* **Közös elhelyezés felhőalapú adatcsere keretében**. Ha közösen van elhelyezve egy felhőalapú adatcserével rendelkező létesítményben, virtuális keresztkapcsolatokat rendelhet az Azure-hoz a közös környezet szolgáltatójának Ethernet-adatcserélőjén keresztül. A közöskörnyezet-szolgáltatók 2. rétegbeli keresztkapcsolatokat vagy felügyelt, 3. rétegbeli keresztkapcsolatokat kínálnak a közös elhelyezési létesítményben lévő infrastruktúra és az Azure között.
* **Pontok közötti Ethernet-kapcsolatok**. A helyszíni adatközpontokat/irodákat pontok közötti Ethernet-hivatkozásokon keresztül is csatlakoztathatja az Azure-hoz. A pontok közötti Ethernet-szolgáltatók 2. rétegbeli kapcsolatokat vagy felügyelt, 3. rétegbeli kapcsolatokat kínálnak az adott hely és az Azure között.
* **Bármely elemek közötti (IPVPN-) hálózatok**. Nagykiterjedésű hálózatát (wide area network, WAN) is integrálhatja az Azure-ral. Az IPVPN-szolgáltatók (jellemzően egy MPLS VPN) bármilyen elemek között létesíthető kapcsolatot kínálnak a fiókirodák és az adatközpontok közötti kapcsolatokhoz. Az Azure összekapcsolható a WAN hálózattal, így ugyanúgy jelenik meg, mint bármely másik fiókiroda. A WAN-szolgáltatók jellemzően 3. rétegbeli felügyelt kapcsolatokat kínálnak.

További információ a kapcsolatszolgáltatókról: [ExpressRoute – áttekintés][expressroute-introduction].

### <a name="expressroute-circuit"></a>ExpressRoute-kapcsolatcsoport

Gondoskodjon róla, hogy a cég megfeleljen az [ExpressRoute előfeltételeinek][expressroute-prereqs], mert ezek hiányában nem tud az Azure-hoz csatlakozni.

Ha még nem tette meg, vegyen fel egy `GatewaySubnet` nevű alhálózatot az Azure-beli virtuális hálózatba, és hozzon létre egy ExpressRoute virtuális hálózati átjárót az Azure VPN Gateway szolgáltatással. További információ a folyamattal kapcsolatban: [Az ExpressRoute kapcsolatcsoport-kiépítési munkafolyamatai és a kapcsolatcsoportok állapotai][ExpressRoute-provisioning].

Hozzon létre egy ExpressRoute-kapcsolatcsoportot az alábbi lépéseket követve:

1. Futtassa az alábbi PowerShell-parancsot:
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Küldje el az új kapcsolatcsoport `ServiceKey` kulcsát a szolgáltatónak.

3. Várjon, amíg a szolgáltató üzembe helyezi a kapcsolatcsoportot. A kapcsolatcsoport üzembehelyezési állapotának az ellenőrzéséhez futtassa az alábbi PowerShell-parancsot:
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    A kimenet `Service Provider` szakaszának `Provisioning state` mezőjében az állapot `NotProvisioned` állapotról `Provisioned` állapotra változik, ha a kapcsolatcsoport készen áll.

    > [!NOTE]
    > Ha 3. rétegbeli kapcsolatot használ, a szolgáltatónak kell konfigurálnia és felügyelnie az útválasztást Ön helyett. Ehhez biztosítania kell a megfelelő útválasztás megvalósításához szükséges információkat a szolgáltatónak.
    > 
    > 

4. Ha 2. rétegbeli kapcsolatot használ:

    1. Foglaljon le két, érvényes nyilvános IP-címekből álló /30 alhálózatot a megvalósítani kívánt társviszony-létesítések minden típusa számára. Ezen /30 alhálózatok segítségével biztosíthatók az IP-címek a kapcsolatcsoporthoz használt útválasztóknak. Ha magánhálózati, nyilvános és Microsoft társviszony-létesítést valósít meg, összesen 6, érvényes nyilvános IP-címekkel rendelkező /30 alhálózatra lesz szüksége.     

    2. Konfigurálja az útválasztást az ExpressRoute-kapcsolatcsoporthoz. Futtassa az alábbi PowerShell-parancsokat a konfigurálni kívánt társviszony-létesítések mindegyik típusára (magánhálózati, nyilvános és Microsoft) vonatkozóan. További információt az [ExpressRoute-kapcsolatcsoport útválasztásának létrehozásával és módosításával][configure-expressroute-routing] foglalkozó cikkben talál.
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Foglalja le az érvényes nyilvános IP-címek egy másik készletét, amelyet a nyilvános és a Microsoft társviszony-létesítés hálózati címfordításához (network address translation, NAT) fog használni. Ajánlott eltérő készleteket használni mindegyik társviszony-létesítéshez. Adja meg a készletet a kapcsolatszolgáltatónak, hogy az konfigurálhassa a Border Gateway Protocol- (BGP-) hirdetéseket az adott tartományok számára.

5. Futtassa az alábbi PowerShell-parancsokat, hogy a virtuális magánhálózatot vagy -hálózatokat az ExpressRoute-kapcsolatcsoporthoz csatlakoztassa. További információért tekintése át a [virtuális hálózat ExpressRoute-kapcsolatcsoporttal történő összekapcsolásával][link-vnet-to-expressroute] foglalkozó cikket.

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

Különböző régiókban található virtuális hálózatokat is csatlakoztathat ugyanahhoz az ExpressRoute-kapcsolatcsoporthoz, feltéve, hogy az ExpressRoute-kapcsolatcsoport és a virtuális hálózatok mindegyike ugyanazon geopolitikai régióban található.

### <a name="troubleshooting"></a>Hibaelhárítás 

Ha egy korábban működő ExpressRoute-kapcsolatcsoport többé nem kapcsolódik, és semmilyen konfigurációs módosítást nem hajtott végre a helyszíni vagy a virtuális magánhálózaton, előfordulhat, hogy a kapcsolatszolgáltatóhoz kell fordulnia, és vele együttműködve kell elhárítania a problémát. Az alábbi PowerShell-parancsokkal ellenőrizheti, hogy sor került-e az ExpressRoute-kapcsolatcsoport üzembe helyezésére:

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

A parancs kimenete a kapcsolatcsoport számos tulajdonságát megjeleníti, ilyen például az alább látható `ProvisioningState`, `CircuitProvisioningState` és `ServiceProviderProvisioningState`.

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

Ha a `ProvisioningState` értéke nem `Succeeded` lesz, miután megpróbált létrehozni egy új kapcsolatcsoportot, távolítsa el a kapcsolatcsoportot az alábbi paranccsal, és próbálkozzon újra a létrehozásával.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Ha a szolgáltató már üzembe helyezte a kapcsolatcsoportot, és a `ProvisioningState` értéke `Failed`, vagy a `CircuitProvisioningState` értéke nem `Enabled`, a szolgáltatótól kérhet további segítséget.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az ExpressRoute-kapcsolatcsoportok nagy sávszélességű útvonalat biztosítanak a hálózatok között. Általánosságban elmondható, hogy minél nagyobb a sávszélesség, annál nagyobb a költség. 

Az ExpressRoute két [díjszabási csomagot][expressroute-pricing] kínál, amelyek közül az egyik forgalmi díjas, a másik pedig korlátlan adatforgalmú. A díjak a kapcsolatcsoport sávszélességétől függően változnak. A rendelkezésre álló sávszélesség nagy valószínűséggel szolgáltatónként eltérő lesz. A `Get-AzureRmExpressRouteServiceProvider` parancsmaggal tekintheti át az adott régióban elérhető szolgáltatókat és az általuk kínált sávszélességet.
 
Egy ExpressRoute-kapcsolatcsoport adott számú társviszony-létesítés és virtuális hálózati kapcsolat támogatására képes. További információt az [ExpressRoute-ra vonatkozó korlátozásokkal](/azure/azure-subscription-service-limits) foglalkozó cikkben talál.

Felár ellenében az ExpressRoute Prémium bővítménye további képességeket biztosít:

* Magasabb útvonalkorlát, amely a nyilvános és a magánhálózati társviszony-létesítésekre is vonatkozik. 
* Több virtuális hálózati kapcsolat ExpressRoute-kapcsolatcsoportonként. 
* Globális kapcsolódás a szolgáltatásokhoz.

További információ: [ExpressRoute – díjszabás][expressroute-pricing]. 

Az ExpressRoute-kapcsolatcsoportok úgy vannak kialakítva, hogy felár nélkül is lehetővé tegyék a hálózati forgalom átmeneti növelését a megvásárolt sávszélesség legfeljebb kétszereséig. Ez redundáns kapcsolatok használatával érhető el, de nem mindegyik kapcsolatszolgáltató támogatja ezt a funkciót. Ha úgy gondolja, hogy szüksége lenne rá, mindenképpen ellenőrizze, hogy a kapcsolatszolgáltató engedélyezi-e a funkció használatát.

Jóllehet egyes szolgáltatók engedélyezik a sávszélesség módosítását, mindenképpen olyan kezdeti sávszélességet válasszon, amely meghaladja az igényeit, és teret hagy a növekedésre. Ha a jövőben növelnie kell a sávszélességet, mindössze két lehetőség áll a rendelkezésére:

- Növelheti a sávszélességet. Lehetőség szerint kerülje ezt a megoldást, mert nem minden szolgáltató teszi lehetővé a sávszélesség dinamikus növelését. Ha mindenképpen szüksége van a növelésre, egyeztessen a szolgáltatóval, hogy támogatja-e az ExpressRoute-sávszélesség módosítását PowerShell-parancsokkal. Ha támogatja, akkor futtassa az alábbi parancsokat.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    A sávszélességet a kapcsolat megszakadása nélkül növelheti. A sávszélesség csökkentése azonban a kapcsolat megszakadását eredményezi, mert törölnie kell a kapcsolatcsoportot, és újra létre kell hoznia az új konfigurációval.

- Módosíthatja a díjcsomagot és/vagy Prémium csomagra válthat. Ehhez futtassa az alábbi parancsokat. Az `Sku.Tier` tulajdonság `Standard` vagy `Premium` lehet, az `Sku.Name` tulajdonság pedig `MeteredData` vagy `UnlimitedData`.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Az `Sku.Name` tulajdonságnak egyeznie kell az `Sku.Tier` és az `Sku.Family` tulajdonsággal. Ha módosítja a családot és a szintet, a nevet azonban nem, a rendszer letiltja a kapcsolatot.
    > 
    > 

    A termékváltozat a kapcsolat megszakadása nélkül módosítható magasabb szintűre, de a korlátlan díjcsomagot nem módosíthatja forgalmi díjas csomagra. Ha a termékváltozatot alacsonyabb szintre módosítja, a sávszélesség-használatnak a standard termékváltozat alapértelmezett korlátozásán belül kell maradnia.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az ExpressRoute nem támogatja az útválasztó-redundancia protokollokat (amilyen például a Hot Standby Routing Protocol (HSRP) és a Virtual Router Redundancy Protocol (VRRP)) a magas rendelkezésre állás megvalósításához. Helyettük a társviszonyonként meglévő BGP-munkamenetek redundáns párját használja. A hálózattal létesített magas rendelkezésre állású kapcsolatok elősegítése érdekében az Azure két redundáns portot biztosít két útválasztón (amelyek a Microsoft-peremhálózat részei) egy aktív-aktív konfigurációban.

Alapértelmezés szerint a BGP-munkamenetek üresjárati időkorlátja 60 másodperc. Ha egy munkamenet háromszor túllépi az időkorlátot (összesen 180 másodperc), a rendszer rendelkezésre nem állóként jelöli meg az útválasztót, és minden forgalmat a fennmaradó útválasztóra irányít át. Előfordulhat, hogy ez a 180 másodperces időkorlát túl hosszú, ha kritikus alkalmazásokról van szó. Ilyenkor módosíthatja a BGP üresjárati beállításait a helyszíni útválasztón egy kisebb értékre.

Az Azure-kapcsolat magas rendelkezésre állását különböző módokon konfigurálhatja, a használt szolgáltató típusától, valamint a konfigurálni kívánt ExpressRoute-kapcsolatcsoportok és virtuális hálózati átjárókapcsolatok számától függően. Az alábbiakban a rendelkezésre állással kapcsolatos lehetőségeket foglaljuk össze:

* Ha 2. rétegbeli kapcsolatot használ, helyezzen üzembe redundáns útválasztókat a helyszíni hálózaton egy aktív-aktív konfigurációban. Csatlakoztassa az elsődleges kapcsolatcsoportot az egyik útválasztóhoz, a másodlagos kapcsolatcsoportot pedig a másikhoz. Így magas rendelkezésre állású kapcsolattal fog rendelkezni a kapcsolat mindkét végén. Erre akkor van szükség, ha ExpressRoute szolgáltatói szerződést (SLA) igényel. További információt az [Azure ExpressRoute-ra vonatkozó szolgáltatói szerződést][sla-for-expressroute] ismertető cikkben talál.

    Az alábbi ábrán az a konfiguráció látható, amely elsődleges és a másodlagos kapcsolatcsoporthoz csatlakoztatott redundáns helyszíni útválasztókból áll. Az egyes kapcsolatcsoportok egy nyilvános és egy magánhálózati társviszony-létesítés forgalmát kezelik (mindegyik társviszony-létesítés számára ki lett jelölve egy pár /30 címtér az előző szakaszban leírtak szerint).

    ![[1]][1]

* Ha 3. rétegbeli kapcsolatot használ, ellenőrizze, hogy az biztosít-e a rendelkezésre állást Ön helyett kezelő redundáns BGP-munkameneteket.

* Csatlakoztassa a virtuális hálózatot több ExpressRoute-kapcsolatcsoporthoz, amelyet különböző szolgáltatók biztosítanak. Ez a stratégia fokozza a magas rendelkezésre állást, és vészhelyreállítási képességeket is biztosít.

* Konfiguráljon helyek közötti VPN-t feladatátvételi útvonalként az ExpressRoute számára. Ezzel a lehetőséggel az a cikk foglalkozik bővebben, amely a [helyszíni hálózat és az Azure VPN-feladatátvételt biztosító ExpressRoute használatával történő csatlakoztatását][highly-available-network-architecture] ismerteti.
 A lehetőség csak magánhálózati társviszony-létesítések esetén érhető el. Azure- és Office 365-szolgáltatások esetében az internet az egyedüli feladatátvételi útvonal. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Az [Azure Connectivity Toolkit (AzureCT)][azurect] eszközzel monitorozhatja a helyszíni adatközpont és az Azure közötti kapcsolatot. 

## <a name="security-considerations"></a>Biztonsági szempontok

Az Azure-kapcsolat biztonsági beállításait különböző módokon konfigurálhatja a biztonsági megfontolásoktól és a megfelelőségi igényektől függően. 

Az ExpressRoute a 3. rétegben működik. Az alkalmazásréteget fenyegető kockázatok egy olyan hálózati biztonsági berendezéssel előzhetők meg, amely megbízható erőforrásokra korlátozza a forgalmat. A nyilvános társviszony-létesítést használó ExpressRoute-kapcsolatok emellett csak a helyszínről kezdeményezhetők. Ez meggátolja a rosszindulatú szolgáltatásokat abban, hogy az internetről hozzáférhessenek a helyszíni adatokhoz, és jogtalanul felhasználják azokat.

A biztonság maximalizálása érdekében a hálózati biztonsági berendezéseket a helyszíni hálózat és a szolgáltató peremhálózati útválasztói között helyezze üzembe. Így könnyebben korlátozható a virtuális hálózatról érkező jogosulatlan forgalom:

![[2]][2]

Naplózási vagy megfelelőségi okokból szükség lehet a virtuális hálózaton futó összetevők közvetlen internet-hozzáférésének a letiltására, valamint a [kényszerített bújtatás][forced-tuneling] megvalósítására. Ilyen helyzetekben az internetes forgalmat át kell irányítani a helyszínen futó proxyra, ahol naplózhatóvá válik. A proxy konfigurálható a nem engedélyezett forgalom kijutásának a megakadályozására és az esetlegesen rosszindulatú bejövő forgalom szűrésére.

![[3]][3]

A biztonság maximalizálása érdekében ne engedélyezze nyilvános IP-címek használatát a virtuális gépek számára, és NSG-k használatával biztosítsa, hogy a virtuális gépek ne lehessenek nyilvánosan elérhetők. Gondoskodni kell arról, hogy a virtuális gépeket csak a belső IP-címekkel lehessen elérni. Ezek a címek az ExpressRoute-hálózaton keresztül tehetők elérhetővé, ami lehetővé teszi a helyszíni fejlesztési és üzemeltetési csapat számára a konfigurálást vagy a karbantartást.

Ha a virtuális gépek felügyeleti végpontjait egy külső hálózaton kell közzétennie, NSG-k vagy hozzáférés-vezérlési listák használatával korlátozhatja az érintett portok láthatóságát az engedélyezett IP-címekre vagy hálózatokra.

> [!NOTE]
> Az Azure Portal használatával üzembe helyezett Azure-beli virtuális gépek alapértelmezés szerint egy nyilvános IP-címet tartalmaznak, amely lehetővé teszi a bejelentkezésen alapuló hozzáférést.  
> 
> 


## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

**Előfeltételek.** Rendelkeznie kell egy megfelelő hálózati berendezéssel konfigurált meglévő helyszíni infrastruktúrával.

A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-er-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:
   * Válassza az **Erőforráscsoport** szakasz **Meglévő használata** elemét, és írja be az `ra-hybrid-er-rg` karakterláncot a szövegmezőbe.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
6. Várjon, amíg az üzembe helyezés befejeződik.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Hibrid hálózati architektúra az Azure ExpressRoute használatával"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Redundáns útválasztók használata elsődleges és másodlagos ExpressRoute-kapcsolatcsoportokkal"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Biztonsági eszközök hozzáadása a helyszíni hálózathoz"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Kényszerített bújtatás használata az internetes forgalom naplózására"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "ExpressRoute-kapcsolatcsoport ServiceKey kulcsának megkeresése"  