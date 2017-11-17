---
title: "Egy helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével"
description: "Hogyan egy biztonságos pont-pont hálózati architektúra, amely egy Azure virtuális hálózatra és az Azure ExpressRoute használatával csatlakoztatott helyszíni hálózat kiterjedő végrehajtásához."
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 671be5118faaefab5ba5348de81642d8a8124b59
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>Egy helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével

A referencia-architektúrában bemutatja, hogyan csatlakozzon egy helyi hálózati Azure, a virtuális hálózatok használatával [Azure ExpressRoute][expressroute-introduction]. ExpressRoute-kapcsolatot egy külső kapcsolatot közvetítésével saját, dedikált kapcsolatot használja. A magánhálózati kapcsolat az Azure a helyszíni hálózat kiterjesztése. [**Ez a megoldás üzembe helyezéséhez**.](#deploy-the-solution)

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőkből áll.

* **Helyszíni vállalati hálózat**. Helyi magánhálózat fut egy szervezeten belül.

* **ExpressRoute-kapcsolatcsoportot**. A 2. vagy 3 rétegbeli áramkör által biztosított a kapcsolat szolgáltatójánál, hogy a peremhálózati útválasztók az Azure-ral csatlakozik a helyi hálózaton. A kapcsolatcsoport a hardver-infrastruktúrát kezeli a kapcsolat szolgáltatóját használja.

* **Helyi peremhálózati útválasztók**. A kapcsolatcsoport a szolgáltató által kezelt-hez a helyi hálózaton útválasztók. Attól függően, hogy a kapcsolat ki van építve szükség lehet arra, hogy az útválasztók a nyilvános IP-címeit.
* **A Microsoft edge útválasztók**. Két útválasztók aktív-aktív magas rendelkezésre állású konfigurációban. Az útválasztókat engedélyezése a kapcsolatok közvetlen csatlakoztatása az adatközpontjában, a kapcsolat szolgáltatóját. Attól függően, hogy a kapcsolat ki van építve szükség lehet arra, hogy az útválasztók a nyilvános IP-címeit.

* **Az Azure virtuális hálózatokról (Vnetekről)**. Minden virtuális hálózat egyetlen Azure régióban található, és több alkalmazásrétegek üzemeltethet. Alkalmazás rétegeket is szegmentált, minden egyes virtuális alhálózatok használatával.

* **Az Azure nyilvános szolgáltatások**. Azure-szolgáltatásokhoz használhat egy hibrid alkalmazáson belül. Ezek a szolgáltatások is elérhetőek az interneten keresztül, de elérése őket az ExpressRoute-kapcsolatcsoportot biztosít alacsony késéssel és több kiszámítható teljesítményt, mert forgalom nem halad az interneten keresztül. Kapcsolatok végrehajtása használatával történik [nyilvános társviszony][expressroute-peering], a szervezete tulajdonában lévő vagy a kapcsolat szolgáltatójánál által biztosított-címekkel.

* **Az Office 365 szolgáltatások**. Az Office 365 nyilvánosan elérhető alkalmazások és a Microsoft által nyújtott szolgáltatások. Kapcsolatok végrehajtása használatával történik [Microsoft társviszony-létesítés][expressroute-peering], a szervezete tulajdonában lévő vagy a kapcsolat szolgáltatójánál által biztosított-címekkel. Keresztül is csatlakozhat közvetlenül a Microsoft CRM Online-hoz a Microsoft társviszony-létesítés.

* **Kapcsolat szolgáltatók** (nincs ábrázolva). Szolgáltató vállalatok kapcsolat segítségével réteg 2 vagy 3 rétegbeli kapcsolatot az adatközpontban, és egy Azure-adatközpontban között.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="connectivity-providers"></a>Kapcsolat-szolgáltatók

Válassza ki a megfelelő ExpressRoute kapcsolat szolgáltatójánál helyéhez. A helyen érhető el kapcsolat szolgáltatók listájának lekéréséhez használja a következő Azure PowerShell-parancsot:

```powershell
Get-AzureRmExpressRouteServiceProvider
```

ExpressRoute-kapcsolat szolgáltató az Adatközpont csatlakoztatni a Microsoft az alábbi módokon:

* **A felhőalapú exchange közös helyen**. Ha most közös elhelyezésű egy helyen az olyan felhőalapú exchange, rendezheti virtuális kereszt-kapcsolatok az Azure-bA a közös elhelyezés szolgáltató Ethernet exchange keresztül. Közös elhelyezés szolgáltatók kínálhat, 2. réteg kereszt-kapcsolatokat, vagy a felügyelt réteg 3 közötti kapcsolatok között a a közös elhelyezés létesítmény és az Azure-infrastruktúra.
* **Point-to-Point protokoll Ethernet kapcsolatok**. Csatlakozhat a helyszíni adatközpontját/használt Azure Point-to-Point protokoll Ethernet-kapcsolaton keresztül. Point-to-Point protokoll Ethernet szolgáltatók kínálhat, 2. réteg kapcsolatok vagy a hely és az Azure közötti felügyelt réteg 3 kapcsolatok.
* **Bármely elem közöttiként (IPVPN) hálózatok**. A nagy kiterjedésű hálózati (WAN) integrálható az Azure-ral. Internet protocol virtuális magánhálózati (IPVPN) (általában a többprotokollos címkék váltás VPN) szolgáltatók bármely közöttiként kapcsolat a fiókirodák és adatközpontok között. Azure csakúgy, mint bármely más fiókiroda megjelenítést a WAN össze kell kötni. Az esetek többségében WAN szolgáltatók által felügyelt 3 rétegbeli kapcsolatot.

Kapcsolat szolgáltatókról további információkat talál a [ExpressRoute bemutatása][expressroute-introduction].

### <a name="expressroute-circuit"></a>ExpressRoute-kapcsolatcsoportot

Győződjön meg arról, hogy a szervezet megfelelt a [ExpressRoute előfeltételként szükséges követelmények] [ expressroute-prereqs] Azure való kapcsolódáshoz.

Ha még nem tette meg, adjon hozzá egy nevű alhálózatot `GatewaySubnet` az Azure virtuális hálózat számára, és hozzon létre egy ExpressRoute virtuális hálózati átjárónak, az Azure VPN-átjáró szolgáltatás segítségével. További információ erről a folyamatról: [kiépítésével és a kapcsolatcsoport állapotok az ExpressRoute-munkafolyamatokat][ExpressRoute-provisioning].

ExpressRoute-kapcsolatcsoportot létrehozni az alábbiak szerint:

1. Futtassa a következő PowerShell-parancsot:
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Küldjön a `ServiceKey` a szolgáltatóval való új kapcsolat.

3. Várja meg a szolgáltatót, amelyet a kapcsolatcsoport kiépítéséhez. A kiépítési állapot a kapcsolat ellenőrzéséhez futtassa a következő PowerShell-parancsot:
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    A `Provisioning state` mező mellett a `Service Provider` kimenet állapotúról `NotProvisioned` való `Provisioned` amikor készen áll-e a kapcsolat.

    > [!NOTE]
    > Ha 3 rétegbeli kapcsolatot használ, a a szolgáltató kell konfigurálása és kezelése a útválasztási meg. Megadja a szolgáltatót, amelyet a megfelelő útvonalak megvalósításához szükséges információkat.
    > 
    > 

4. A 2. réteg kapcsolat használata:

    1. Két foglalási/30 alhálózatok össze az egyes érvényes nyilvános IP-címek a társviszony-létesítés szeretné végrehajtani. Ezek/30 alhálózatok történik meg az IP-címeket az útválasztók a kapcsolatcsoport használatos. Ha saját, a nyilvános és a Microsoft társviszony-létesítés készül olyanokat, 6/30-as alhálózat érvényes nyilvános IP-címekkel rendelkező lesz szüksége.     

    2. Az ExpressRoute-kör útválasztás konfigurálása. Futtassa a következő PowerShell-parancsok az egyes társviszony-létesítés (magánfelhő, nyilvános, és a Microsoft) konfigurálni szeretné. További információkért lásd: [létrehozása és módosítása az ExpressRoute-kapcsolatcsoportot útválasztási][configure-expressroute-routing].
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Foglaljon le egy másik érvényes nyilvános IP-címkészletet használandó hálózati címfordítás (NAT) a nyilvános és a Microsoft társviszony-létesítés. Javasoljuk, hogy rendelkezik, és különböző a minden társviszony-létesítéshez. Adja meg a kapcsolat szolgáltatójánál, hogy a készlet, így konfigurálhatják a border gateway protocol (BGP) hirdetmények ezeket a tartományokat.

5. Futtassa a következő PowerShell-parancsokat a személyes VNet(s) csatolása az ExpressRoute-kapcsolatcsoportot. További információkért lásd: [virtuális hálózat csatolása ExpressRoute-kapcsolatcsoportot][link-vnet-to-expressroute].

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

Az azonos ExpressRoute-kapcsolatcsoportot a különböző régiókban található több Vnetek is elérheti, mindaddig, amíg az összes virtuális hálózatokat és az ExpressRoute-kapcsolatcsoport geopolitikai régión belül találhatók.

### <a name="troubleshooting"></a>Hibaelhárítás 

Ha egy korábban működő ExpressRoute-kapcsolatcsoportot most nem csatlakozik, bármilyen konfigurációs módosításokat a helyszínen vagy a titkos virtuális hálózat nincs szükség lehet kapcsolódni a kapcsolat szolgáltatóját, és azokat a probléma elhárításához. A következő Powershell-parancsok segítségével győződjön meg arról, hogy az ExpressRoute-kapcsolatcsoport van kiépítve:

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Ez a parancs kimenetét mutatja a kapcsolatcsoport több tulajdonságainak beleértve `ProvisioningState`, `CircuitProvisioningState`, és `ServiceProviderProvisioningState` alább látható módon.

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

Ha a `ProvisioningState` értéke nem `Succeeded` után hozzon létre egy új kapcsolat próbált, távolítsa el a kapcsolatcsoport az alábbi parancs használatával, és próbálja meg újra létrehozni.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

Ha a szolgáltató már volt megadva a kapcsolatcsoport, és a `ProvisioningState` értéke `Failed`, vagy a `CircuitProvisioningState` nem `Enabled`, további segítségért forduljon a szolgáltatóhoz.

## <a name="scalability-considerations"></a>Méretezési szempontok

ExpressRoute-Kapcsolatcsoportok adja meg a nagy sávszélesség elérési hálózatok között. Általában a nagyobb sávszélességet minél nagyobb a költségeket. 

ExpressRoute biztosít két [díjszabások] [ expressroute-pricing] legyenek az ügyfelek, egy mért használatú terv és egy adatforgalmi. Díjak kapcsolatcsoport sávszélessége függvényében. Rendelkezésre álló sávszélesség valószínűleg eltérőek a provider szolgáltató. Használja a `Get-AzureRmExpressRouteServiceProvider` parancsmag használatával ellenőrizheti a területi és a sávszélesség kínálnak, rendelkezésre álló szolgáltatót.
 
Egy ExpressRoute-kapcsolatcsoportot bizonyos számú esetében, valamint a virtuális hálózat mutató hivatkozásokat is támogatja. Lásd: [ExpressRoute korlátok](/azure/azure-subscription-service-limits) további információt.

Az ExpressRoute prémium szintű bővítmény külön díj, néhány további lehetőség biztosít:

* Nagyobb útvonal korlátok a nyilvános és magánhálózati társviszony-létesítéshez. 
* Nagyobb száma VNet egy ExpressRoute-kapcsolatcsoportot. 
* Globális kapcsolódás a szolgáltatásokhoz.

Lásd: [ExpressRoute árképzési] [ expressroute-pricing] részleteiről. 

ExpressRoute-Kapcsolatcsoportok úgy tervezték, hogy engedélyezi az átmeneti hálózati felszakadásáig legfeljebb kétszer a sávszélességre vonatkozó korlátját, akiktől a további költség nélkül. A redundáns hivatkozások használatával érhető el. Nem minden kapcsolat szolgáltatók azonban támogatja ezt a szolgáltatást. Győződjön meg arról, hogy a kapcsolat szolgáltatójánál lehetővé teszi, hogy ez a szolgáltatás előtt attól függően, hogy azt.

Bár egyes szolgáltatók lehetővé teszik a sávszélesség megváltoztatását, győződjön meg arról, válasszon egy kezdeti sávszélesség meghaladja az igényeknek és a hely marad. Ha a jövőben növelésére sávszélességet van szüksége, marad, két lehetőség közül választhat:

- Növelik a sávszélességet. Ez a beállítás lehetőség szerint kerülje el, és nem minden szolgáltatók lehetővé teszik dinamikusan növelésére sávszélességet. De ha nincs szükség a sávszélesség növelését, ellenőrizze azok támogatják a változó ExpressRoute sávszélesség tulajdonságok Powershell-parancsok segítségével ellenőrizheti a szolgáltatónál. Ha így tesznek, futtassa az alábbi parancsok.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    A sávszélesség, a kapcsolat megszakadása nélkül növelhető. Alacsonyabb verziójúra változtatása a sávszélességet eredményez megszűnésének a kapcsolat, mert törölnie kell a kapcsolatcsoport, majd hozza létre újra az új konfigurációval.

- Módosítsa a tarifacsomagot, és/vagy váltson prémium. Ehhez futtassa a következő parancsokat. A `Sku.Tier` tulajdonság lehet `Standard` vagy `Premium`; a `Sku.Name` tulajdonság lehet `MeteredData` vagy `UnlimitedData`.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Győződjön meg arról, hogy a `Sku.Name` tulajdonság megegyezik a `Sku.Tier` és `Sku.Family`. Ha módosítja a családja és szint, de nem a név, a kapcsolat letiltásra kerül.
    > 
    > 

    Frissítheti a Termékváltozat megszakítása nélkül, de nem lehet átállítani a korlátlan árképzési tervből mértre. Amikor alacsonyabb verziójúra változtatása a Termékváltozat, a sávszélesség-felhasználást a standard Termékváltozat alapértelmezett keretein belül kell maradnia.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

ExpressRoute nem támogatja az útválasztó redundancia protokollok-pl.: meleg készenléti útválasztási protokollt (HSRP) és virtuális útválasztó redundancia protokoll megvalósítása a magas rendelkezésre állású (VRRP). Ehelyett egy redundáns párt a BGP társviszony-létesítés munkamenetek használ. Lehetővé teszi a hálózati kapcsolatok magas rendelkezésre állású, Azure látja el, a két redundáns portokkal aktív-aktív konfigurációban két útválasztókon (a Microsoft edge része).

Alapértelmezés szerint a BGP-munkamenetek használja az üresjárati időtúllépés értéke 60 másodperc. Ha a munkamenet időkorlátja lejár háromszor (teljes 180 másodperc), az útválasztó jelölése szerint nem érhető el, és minden forgalmat a rendszer átirányítja a fennmaradó útválasztó. Előfordulhat, hogy a 180 másodperces időtúllépési túl hosszú, és kulcsfontosságú alkalmazások kiszolgálására. Ha igen, módosíthatja a BGP időtúllépési beállításainak a helyi útválasztó kisebb értékre.

Az Azure-kapcsolat a magas rendelkezésre állású használata szolgáltató típusa mellett, és az ExpressRoute-Kapcsolatcsoportok és hajlandó konfigurálása most a virtuális hálózati átjáró-kapcsolatok száma függően különböző módon állíthatja be. Az alábbiakban összefoglalást láthat a rendelkezésre állási beállítások:

* A 2. réteg kapcsolat használata, telepítse a helyszíni hálózat aktív-aktív konfigurációban útválasztókat. Az elsődleges kapcsolat csatlakozhatnak egy útválasztó és a többi másodlagos kör. Ez biztosítja a magas rendelkezésre állású kapcsolat, a kapcsolat mindkét végén. Ez azért szükséges, ha szüksége van a ExpressRoute szolgáltatásiszint-szerződéssel (SLA). Lásd: [Azure ExpressRoute SLA] [ sla-for-expressroute] részleteiről.

    Az alábbi ábrán látható, a redundáns helyi útválasztó konfigurációja az elsődleges és másodlagos Kapcsolatcsoportok csatlakozik. A kapcsolatok kezeli a forgalmat a nyilvános társviszony-létesítés, és a magánhálózati társviszony-létesítés (/ 30-as két kijelölt minden társviszony-címtereket, az előző szakaszban leírtak szerint).

    ![[1]][1]

* Ha a 3 rétegbeli kapcsolatot használ, ellenőrizze a redundáns BGP biztosítja, amelyek kezelik a rendelkezésre állási meg a munkamenetek.

* Csatlakoztassa a virtuális hálózat több ExpressRoute-Kapcsolatcsoportok, különböző szolgáltatók által biztosított. Ezt a stratégiát további magas rendelkezésre állású és vész helyreállítási lehetőségeket kínál.

* A telephelyek közötti VPN elérési útnak feladatátvétel konfigurálása ExpressRoute. Ez a beállítás kapcsolatban bővebben lásd: [egy a helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével a VPN-feladatátvételi][highly-available-network-architecture].
 Ez a beállítás csak érvényes magánhálózati társviszony-létesítés. Azure és az Office 365 szolgáltatáshoz az internethez az egyetlen feladatátvevő útvonal. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Használhatja a [Azure kapcsolat Toolkit (AzureCT)] [ azurect] figyelheti a helyszíni adatközpontját és az Azure közötti kapcsolatot. 

## <a name="security-considerations"></a>Biztonsági szempontok

Biztonsági beállításokkal megadhatja az Azure-kapcsolat különböző módokon, attól függően, hogy a biztonsági problémáit és megfelelőségi igényeit. 

ExpressRoute 3. rétegben működik. A hálózati biztonsági készülékek, amely korlátozza a forgalmat jogos erőforrások használatával megakadályozása az alkalmazási rétegre fenyegetések ellen. Továbbá használatával a nyilvános társviszony ExpressRoute-kapcsolatok csak kezdeményezheti a helyszíni. Ez megakadályozza, hogy egy engedélyezetlen szolgáltatás eléréséhez és az internetről a helyszíni adatok veszélyeztetése.

Biztonság maximalizálása érdekében vegye fel a hálózati biztonsági készülékek a helyszíni hálózat és a szolgáltató peremhálózati útválasztók között. Ez segít a jogosulatlan forgalom vízátfolyás a VNet korlátozhatja, hogy:

![[2]][2]

Naplózás vagy megfelelőségi okokból lehet szükség a közvetlen hozzáférés tiltása az internethez, és alkalmazzon a Vneten belül futó összetevőkből [kényszerített bújtatás][forced-tuneling]. Ebben a helyzetben internetes forgalom vissza helyileg futó, ahol naplózható proxyn keresztül kell irányítani. A proxy beállítható úgy, hogy blokkolni, jogosulatlan forgalom továbbítására ki, és a potenciálisan kártevő bejövő forgalmat.

![[3]][3]

A biztonság fokozása nem egy nyilvános IP-cím engedélyezése a virtuális géphez, és az NSG-k segítségével győződjön meg arról, hogy a virtuális gépeken nem nyilvánosan elérhető. Virtuális gépek csak a belső IP-cím elérhetőnek kell lennie. Ezeket a címeket is elérhetővé válik a ExpressRoute hálózati konfiguráció vagy karbantartás végrehajtásához a helyi DevOps személyzet engedélyezése.

Ha egy külső hálózathoz, fel kell fednie felügyeleti végpontok virtuális gépekhez, az NSG-k használata, vagy hozzáférés-vezérlési listák korlátozása egy hálózatok és IP-címek engedélyezési listája látható-e ezeket a portokat.

> [!NOTE]
> Alapértelmezés szerint az Azure portálon keresztül telepített Azure virtuális gépek tartalmaznak egy nyilvános IP-címet, amely hozzáférést biztosít.  
> 
> 


## <a name="deploy-the-solution"></a>A megoldás üzembe helyezéséhez

**Prequisites.** Rendelkeznie kell egy meglévő helyi infrastruktúra már be van állítva a megfelelő hálózati berendezések.

A megoldás üzembe helyezéséhez, a következő lépésekkel.

1. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd kövesse az alábbi lépéseket:
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-er-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd kövesse az alábbi lépéseket:
   * Válassza ki **használata meglévő** a a **erőforráscsoport** szakaszt, és adja meg `ra-hybrid-er-rg` a szövegmezőben.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
6. Várjon, amíg az üzembe helyezés befejeződik.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute/v1_0/
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Hibrid hálózati architektúra Azure ExpressRoute segítségével"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Az ExpressRoute-Kapcsolatcsoportok elsődleges és másodlagos útválasztókat használatával"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Biztonsági eszközök hozzáadása a helyszíni hálózat"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Használja a kényszerített bújtatás az internetre irányuló forgalomnak a naplózandó"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "Az ExpressRoute-kapcsolatcsoportot ServiceKey keresése"  